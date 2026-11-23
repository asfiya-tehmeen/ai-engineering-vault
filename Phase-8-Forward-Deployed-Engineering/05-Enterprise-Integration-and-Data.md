---
tags: [phase-8, integration, enterprise, sso, data]
---
# 05 — Enterprise Integration & Data

> In enterprise AI, **the model is rarely the hardest part.** The hardest parts are *getting access*, *connecting to legacy systems*, *identity and permissions*, *data quality*, *security approvals*, and *operating inside someone else's environment*. FDEs who handle this fluently ship; those who don't stall for months. (Builds on [[../Phase-1-Foundations/06-HTTP-APIs-JSON]] and [[../Phase-1-Foundations/07-SQL-and-Databases]].)

## 1. The enterprise systems landscape (know the vocabulary)
| Category | Common systems | Typical integration surface |
|---|---|---|
| **ERP / finance** | SAP (S/4HANA, ECC), Oracle (EBS, Fusion), Microsoft Dynamics, NetSuite, Workday Financials | REST/OData/SOAP APIs, IDocs/BAPI (SAP), batch files/SFTP, DB views (read replicas), middleware |
| **CRM / sales & service** | Salesforce, HubSpot, Dynamics 365, Zendesk, Freshdesk | REST/SOAP/Bulk APIs, webhooks, platform events, SOQL/SQL exports |
| **ITSM / workflow** | ServiceNow, Jira Service Management, BMC Remedy | REST Table API, webhooks/flows, MID servers, scripted REST |
| **HRIS** | Workday, SuccessFactors, ADP, BambooHR | REST/SOAP, reports, SCIM (identity) |
| **Document/knowledge** | SharePoint/OneDrive, Confluence, Google Drive, Box, Notion, Documentum, file shares | Graph API, REST, crawlers, export jobs; **permissions inheritance** |
| **Collaboration** | Slack, Microsoft Teams, email (Exchange/Gmail) | bots/apps, Graph/Gmail APIs, webhooks, add-ins |
| **Data platforms** | Snowflake, Databricks, BigQuery, Redshift, Synapse, Oracle/SQL Server/Postgres, Teradata | SQL, connectors, Spark, Delta Sharing, secure views |
| **Healthcare** | Epic, Cerner/Oracle Health, HL7 v2, **FHIR** APIs | FHIR REST, HL7 interfaces, integration engines |
| **Finance/banking** | core banking, payments, KYC/AML systems, SWIFT, ISO 20022 | MQ, SOAP, batch, vendor APIs |
| **Industrial/IoT** | MES, SCADA/historians (OSIsoft PI), PLM, CMMS | OPC-UA, historians, REST, files |
| **Legacy** | mainframes (COBOL/CICS/DB2), AS/400, green-screen terminals, FTP/EDI | MQ, flat files, screen scraping/RPA, middleware |
| **iPaaS/ETL/RPA** | MuleSoft, Boomi, Workato, Zapier, Informatica, Fivetran/Airbyte, UiPath/Automation Anywhere | pre-built connectors; low-code flows |
You don't need to master each; you need to **ask the right questions** ("what's the API surface, auth model, rate limits, sandbox, and who owns it?") and **learn docs quickly** (a skill Palantir-style interviews explicitly test).

## 2. Integration patterns
| Pattern | Use when | Notes |
|---|---|---|
| **Direct REST/GraphQL API calls** | modern systems | auth, pagination, rate limits, retries/idempotency ([[../Phase-1-Foundations/06-HTTP-APIs-JSON]]) |
| **Webhooks / event-driven** | react to changes | verify signatures, ack fast, dedupe, handle retries/out-of-order |
| **Polling** | no webhooks | use cursors/modified-since; backoff |
| **Batch file drop (SFTP/S3/shared drive)** | legacy/batch systems | schema contracts, encoding, partial files, idempotent loads, archive & replay |
| **Database read replica / views / CDC** | direct data access (careful) | read-only roles, avoid prod load; CDC (Debezium) for change streams; **don't write to vendor DBs** |
| **ETL/ELT into warehouse/lake** | analytics + AI over history | dbt, Airflow/Dagster, Fivetran/Airbyte |
| **Message queues/ESB (Kafka, RabbitMQ, IBM MQ, SQS)** | enterprise messaging | consumer groups, DLQ, schema registry |
| **iPaaS/low-code connectors** | customer already uses it | faster but less control; understand limits |
| **RPA / UI automation** | no API available | brittle; last resort; pair with verification |
| **MCP servers / tool wrappers** | expose systems to agents | least privilege, per-user auth, gateway ([[../Phase-6-Agents-Tools-MCP/03-MCP-Model-Context-Protocol]]) |
| **Embedded UI (Teams/Slack bot, browser extension, side panel, plugin)** | meet users where they work | adoption driver |
| **SOAP/XML/WSDL, EDI, HL7 v2, fixed-width files** | legacy/industry standards | use libraries; schema validation; character encodings |
**Principles:** prefer **read-only** first; **idempotent** writes; **contracts** (schemas) with versioning; **retry with backoff**; **dead-letter** queues; **observability** per integration; **isolation** so a customer system failure doesn't cascade; **respect rate limits and maintenance windows**.

## 3. Identity & access (SSO, authorization, provisioning)
- **Authentication protocols:** **SAML 2.0** (older enterprise web SSO), **OIDC/OAuth 2.0** (modern), Kerberos/NTLM (internal Windows), LDAP/AD bind (legacy). **Identity providers (IdPs):** Okta, Microsoft Entra ID (Azure AD), Ping, OneLogin, Google Workspace, Keycloak (self-hosted).
- **Provisioning:** **SCIM** for user/group lifecycle sync (create/update/deactivate); **JIT provisioning** on first login; group-to-role mappings.
- **Authorization models:** RBAC, ABAC, per-document ACLs; enforce **at the data/tool layer**, using the *user's* identity (delegated tokens), not the agent's service account ([[../Phase-6-Agents-Tools-MCP/07-Agent-Reliability-Safety-and-Security]]).
- **Service-to-service:** OAuth client-credentials, mTLS, managed identities/workload identity federation, API keys in secret managers, short-lived tokens.
- **Typical asks:** "Users must log in with Okta," "access must follow SharePoint permissions," "log every access," "deactivate users within X hours of termination."
- **Pitfalls:** group nesting, stale group caches, tenants with multiple IdPs, shared accounts, service accounts with excessive rights, token audience mistakes.

## 4. Networking & deployment environments
- **Where it runs:** vendor SaaS (multi-tenant), **single-tenant/dedicated**, **customer cloud VPC/VNet**, **on-prem**, **air-gapped**, hybrid.
- **Network realities:** firewalls/allow-lists (IP ranges, FQDNs), **egress proxies** with TLS interception (certificate trust issues), VPN/ExpressRoute/Direct Connect, **private endpoints/PrivateLink**, DNS split-horizon, no inbound from internet, restricted outbound to AI providers.
- **Typical blockers:** "can't call the model API from the VPC" → use cloud-native model endpoints (Bedrock/Vertex/Azure) through private links, or self-host open-weight models; "no outbound internet" → offline bundles/mirrors; "can't install packages" → internal artifact repos.
- **Environments:** dev/test/UAT/prod with **separate data and approvals**; ask early about **change-control windows (CAB/ITIL)**, release freezes, and who can deploy.
- **Windows/legacy shells:** don't assume Linux/Docker; PowerShell, restricted laptops, locked-down VDI — plan tooling accordingly.
- **Infrastructure-as-code & deployment:** Terraform/Helm/CloudFormation; deliver as containers + Helm chart/Terraform module; document prerequisites (ports, roles, quotas).
- **Data residency/sovereignty:** region pinning, local processing, subprocessors.

## 5. Security review & procurement (the gauntlet)
**Typical sequence:** intake form → security questionnaire (SIG/CAIQ-style, hundreds of questions) → architecture review → data-protection assessment (DPIA/DPA) → pen-test/attestation requests (SOC 2/ISO 27001 reports) → legal (MSA, DPA, BAA if HIPAA) → procurement/vendor onboarding → IT onboarding (SSO, network). **Weeks to months.**
**How to move faster:** start on **day 1**; provide an **evidence pack** (architecture & data-flow diagrams, data classification, encryption, access control, logging, retention/deletion, subprocessors, model-provider terms incl. retention/ZDR, red-team/guardrail results, incident response, SOC2/ISO attestations — [[../Phase-7-Evals-Observability-LLMOps/08-Governance-Compliance-and-FinOps]]); offer **deployment options** (customer VPC, private endpoints, open-weight on-prem); find the **security champion** and ask what would make them comfortable; answer questions precisely and consistently (keep a Q&A library); **never** guess — "I'll confirm and respond by Friday."
**Common LLM-specific questions you must answer well:** Where does data go? Is it used for training? How long retained? Can you run in our tenant/region? How do you prevent prompt injection/data leakage? How do you enforce our permissions? What if the model hallucinates? Can we audit every output? How do you handle PII? What happens if the provider is down or changes the model?

## 6. Data work in enterprises
- **Access first:** request **read-only**, least-privilege access to *specific* tables/folders; use **service accounts**; get a **sandbox** or sanitized extracts for development; document the data contract (fields, freshness, owner).
- **Data quality:** duplicates, versions, free-text junk, inconsistent IDs, time zones/currencies, encodings, missing fields, scans. Build **quality reports** and share findings with data owners (they usually know).
- **Master data & entity resolution:** same customer/vendor/product with different IDs across systems → build mapping tables; use LLM-assisted matching **with human review**.
- **Semantic layers & glossaries:** "revenue" means three things → capture definitions (metrics layer/dbt/LookML) for text-to-SQL and RAG disambiguation.
- **Governance:** classification (public/internal/confidential/restricted), PII tagging, lineage, retention; obey data-handling rules (don't copy restricted data into unapproved tools).
- **Lakehouse/warehouse integration:** read via governed views; use **secure views/row-level security**; avoid ad-hoc exports; use Delta/Iceberg sharing where available.
- **Unstructured corpora:** SharePoint/Confluence/Drive crawls — handle permissions inheritance, deletions, version history, duplicates, attachments, OCR ([[../Phase-5-RAG-Knowledge-Systems/04-Ingestion-Parsing-and-Chunking]]).
- **Streaming/real-time needs:** confirm if truly required; batch/near-real-time is simpler.

## 7. Getting unblocked (the practical playbook)
1. **Maintain an "access tracker"**: system, owner, request date, status, blocker, escalation path. Review weekly with the sponsor.
2. **Parallelize**: while waiting, build with samples/synthetic/mock services (record/replay API fixtures; local containers).
3. **Escalate with specifics**: "Request #123 for read-only CRM API has been pending 8 days; it blocks the week-3 demo; can you ask IT to prioritize?"
4. **Find the real decision-maker**; many blockers are policy ambiguity, not malice.
5. **Reduce the ask**: narrower scope, read-only, fewer fields, shorter retention, sandbox first.
6. **Propose alternatives**: customer-hosted deployment, private endpoints, a vendor-managed secure enclave, redaction/tokenization.
7. **Document decisions** (Slack/Confluence) so you can point to approvals later.

## 8. Mock enterprise sandbox (practice environment; see Project 2)
`docker compose` with: **Keycloak** (OIDC/SAML IdP + SCIM), **Postgres** (stand-in for ERP DB), a **fake CRM REST API** (FastAPI) with pagination/rate limits/errors, a **fake ITSM** with webhooks, an **SFTP** server for batch files, a **legacy SOAP** service, a **Squid proxy** with egress allow-list, **MinIO** (S3), and **Mailhog** (email). Build integrations against it: SSO login, SCIM sync, CRM read, ticket creation, file ingestion, webhook receiver, proxy-constrained model calls, secrets via Vault.

## 9. Integration quality checklist
- [ ] Auth method agreed; secrets in a vault; tokens short-lived
- [ ] Least-privilege scopes; read-only by default
- [ ] Rate limits/backoff handled; pagination correct; idempotent writes
- [ ] Error handling & DLQ; replay tooling
- [ ] Schema validation + contract tests; versioning strategy
- [ ] Timeouts/circuit breakers; failure isolation
- [ ] Monitoring: success rate, latency, queue depth; alerts to the right owner
- [ ] Audit logging of data access and actions
- [ ] Data classification & retention honored; PII handling
- [ ] Documented runbook and ownership
- [ ] Tested in non-prod; change-control scheduled for prod

## 10. Common failure modes
Underestimating approval timelines · building on unauthorized data copies · prod credentials in notebooks · ignoring permission inheritance (RAG leaks) · assuming APIs behave like docs · hitting vendor rate limits in bulk loads · no idempotency → duplicate records · timezone/encoding bugs · proxy/TLS interception breaking SDKs · "works in my laptop" vs locked-down VDI · single integration point of failure · no monitoring on the integration (silent breakage).

## Self-check
1. For each of ERP, CRM, ITSM, SharePoint, a mainframe: name the likely integration patterns and traps. 2. Explain SAML vs OIDC vs SCIM and where each fits. 3. Design network access for calling a model API from a locked-down VPC. 4. List 8 security-questionnaire questions for an LLM app and your answers. 5. Build an access tracker and escalation plan. 6. How do you enforce SharePoint permissions in a RAG system?

Next → [[06-Stakeholders-Communication-and-Change]]
