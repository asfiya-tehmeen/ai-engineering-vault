---
tags: [phase-6, resources]
---
# 12 — Resources & Practice Links (Phase 6)

> ⭐ = top pick. This area changes weekly. I haven't opened every link; if one moves, search the title. arXiv IDs are from memory. For protocol/framework specifics, trust **official docs over blogs** (several 2026 comparison articles had inconsistent versions and dates).

## Must-read engineering essays (start here)
1. ⭐ Anthropic — *Building Effective Agents* (workflows vs agents; 5 patterns; ACI) https://www.anthropic.com/engineering/building-effective-agents
2. ⭐ Anthropic — *Writing Effective Tools for Agents* https://www.anthropic.com/engineering/writing-tools-for-agents
3. ⭐ Anthropic — *Effective Context Engineering for AI Agents* https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
4. ⭐ Anthropic — *How we built our multi-agent research system* https://www.anthropic.com/engineering/multi-agent-research-system
5. Anthropic — *Effective harnesses for long-running agents*; *Code execution with MCP*; *Equipping agents with Agent Skills*; *Demystifying evals for AI agents* — search titles at https://www.anthropic.com/engineering
6. OpenAI — *A practical guide to building agents* (PDF; search the title) · OpenAI Cookbook agent examples https://cookbook.openai.com/
7. Google — Agents whitepaper / Kaggle agents intensive (search "Google agents whitepaper")
8. Lilian Weng — *LLM Powered Autonomous Agents* https://lilianweng.github.io/posts/2023-06-23-agent/
9. Simon Willison — *The lethal trifecta for AI agents* https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/ · prompt-injection tag https://simonwillison.net/tags/prompt-injection/
10. Hamel Husain / Eugene Yan / Chip Huyen — agent & eval write-ups (https://hamel.dev/ · https://eugeneyan.com/ · https://huyenchip.com/)

## MCP
- ⭐ Official site & docs https://modelcontextprotocol.io/ · **Specification (current revision per late-2026 sources: 2026-07-28)** https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization · Spec release posts https://blog.modelcontextprotocol.io/
- SDKs: Python https://github.com/modelcontextprotocol/python-sdk · TypeScript https://github.com/modelcontextprotocol/typescript-sdk · others (Java, Kotlin, C#, Go, Rust, Ruby, Swift) via the org page https://github.com/modelcontextprotocol
- ⭐ **MCP Inspector** (debug servers) https://github.com/modelcontextprotocol/inspector · Reference servers https://github.com/modelcontextprotocol/servers · **Registry** (listed as preview/growing; vet servers) https://registry.modelcontextprotocol.io/
- FastMCP https://gofastmcp.com/ · MCP-use · Spring AI MCP · Cloudflare remote MCP servers docs
- Security: MCP security best practices (in spec docs) · OWASP MCP guidance (genai.owasp.org) · research on tool poisoning/rug pulls (search "MCP tool poisoning")
- Agent-to-agent: **A2A protocol** https://a2a-protocol.org/ · **AG-UI** https://docs.ag-ui.com/ · Agentic AI Foundation (Linux Foundation) announcements

## Agent SDKs & frameworks (official docs)
- **Claude Agent SDK** https://code.claude.com/docs/en/agent-sdk · Claude API tool use & Managed Agents (docs.claude.com → platform docs) · Anthropic Cookbook agent recipes https://github.com/anthropics/anthropic-cookbook
- **OpenAI Agents SDK** https://openai.github.io/openai-agents-python/
- **LangGraph** https://langchain-ai.github.io/langgraph/ · LangSmith https://docs.smith.langchain.com/
- **Google ADK** https://google.github.io/adk-docs/
- **Pydantic AI** https://ai.pydantic.dev/ · Logfire https://pydantic.dev/logfire
- CrewAI https://docs.crewai.com/ · AutoGen/AG2 · LlamaIndex agents https://docs.llamaindex.ai/ · Haystack agents · Mastra (TS) https://mastra.ai/ · Vercel AI SDK https://ai-sdk.dev/ · DSPy https://dspy.ai/ · Semantic Kernel
- Durable execution: Temporal https://docs.temporal.io/ · Inngest https://www.inngest.com/docs · Restate https://docs.restate.dev/
- Memory: Letta https://docs.letta.com/ · Mem0 https://docs.mem0.ai/ · Zep https://help.getzep.com/
- Comparison articles (use with caution — versions/dates vary): Speakeasy, Langfuse, Arize framework comparisons (search "agent framework comparison 2026")

## Sandboxing & computer use
- Sandboxes: E2B https://e2b.dev/docs · Modal sandboxes https://modal.com/docs · Daytona · Cloudflare/Vercel sandboxes · Docker · gVisor https://gvisor.dev/ · Firecracker https://firecracker-microvm.github.io/ · Kata Containers
- Browser automation: Playwright https://playwright.dev/ · Browserbase · browser-use https://github.com/browser-use/browser-use · Stagehand
- Computer use: provider docs (Anthropic computer use, OpenAI computer use, Gemini computer use) — tool versions change; check docs

## Security references
- ⭐ OWASP GenAI Security Project — Top 10 for **Agentic Applications** & Top 10 for LLM Applications https://genai.owasp.org/
- Cloud Security Alliance — *AI Agent Lethal Trifecta* research note (search title) · NIST AI RMF https://www.nist.gov/itl/ai-risk-management-framework · MITRE ATLAS https://atlas.mitre.org/
- Papers: *Not what you've signed up for* (indirect prompt injection; Greshake et al.) https://arxiv.org/abs/2302.12173 · *AgentDojo* https://arxiv.org/abs/2406.13352 · *Design Patterns for Securing LLM Agents against Prompt Injections* (search title) · *CaMeL / Defeating Prompt Injections by Design* (search title) · *Universal and transferable adversarial attacks* https://arxiv.org/abs/2307.15043
- Practice: **Gandalf** https://gandalf.lakera.ai/ · Gray Swan arena https://www.grayswan.ai/ · HackAPrompt · Garak (LLM vulnerability scanner) https://github.com/NVIDIA/garak · PyRIT (Microsoft red-teaming) https://github.com/Azure/PyRIT · promptfoo red-teaming https://www.promptfoo.dev/docs/red-team/

## Papers (agent foundations)
| Topic | Paper | Link |
|---|---|---|
| ReAct | Yao et al. | https://arxiv.org/abs/2210.03629 |
| Tool use | Toolformer | https://arxiv.org/abs/2302.04761 |
| Reflection | Reflexion | https://arxiv.org/abs/2303.11366 |
| Memory | MemGPT | https://arxiv.org/abs/2310.08560 |
| Simulation | Generative Agents | https://arxiv.org/abs/2304.03442 |
| Skills | Voyager | https://arxiv.org/abs/2305.16291 |
| Coding agents | SWE-agent | https://arxiv.org/abs/2405.15793 |
| Code actions | CodeAct | https://arxiv.org/abs/2402.01030 |
| Multi-agent | *Why Do Multi-Agent LLM Systems Fail?* | https://arxiv.org/abs/2503.13657 |
| Multi-agent frameworks | AutoGen | https://arxiv.org/abs/2308.08155 |
| Survey | *The Rise and Potential of LLM-based Agents* | https://arxiv.org/abs/2309.07864 |

## Benchmarks & eval tools
- **Inspect AI** (UK AISI evals framework; agents, sandboxes) https://inspect.aisi.org.uk/ · Braintrust · Langfuse · Phoenix · Promptfoo · DeepEval · OpenAI Evals
- Benchmarks: SWE-bench https://www.swebench.com/ (note 2026 contamination critiques of the Verified subset; look for Pro/newer variants) · **Terminal-Bench** https://www.tbench.ai/ · **τ-bench / τ²-bench** https://github.com/sierra-research/tau-bench (arXiv 2406.12045) · **OSWorld** https://os-world.github.io/ · WebArena https://webarena.dev/ · **BrowseComp** (OpenAI; search title) · GAIA https://huggingface.co/gaia-benchmark · BFCL https://gorilla.cs.berkeley.edu/leaderboard.html · AgentBench https://arxiv.org/abs/2308.03688 · MLE-bench / PaperBench / RE-Bench · METR time-horizon work https://metr.org/
- Research on eval reliability (2026 preprints surfaced in search): benchmark auditing, judge drift ("version-dependent LLM-judge error"), and *Towards a Science of AI Agent Reliability* — search these titles on arXiv.

## Courses & talks
- ⭐ Hugging Face **Agents Course** https://huggingface.co/learn/agents-course
- ⭐ DeepLearning.AI: *AI Agents in LangGraph*, *Agentic AI* (Andrew Ng), *Building Systems with the ChatGPT API*, MCP short course https://www.deeplearning.ai/short-courses/
- Berkeley **LLM Agents** MOOC (CS294/194) https://rdi.berkeley.edu/llm-agents/f24 · Princeton/Stanford agent lectures · AI Engineer conference agent tracks https://www.ai.engineer/
- Karpathy — "Software 3.0"/agents talks (YouTube) · Latent Space agent episodes https://www.latent.space/

## Practice projects & datasets
- Build on **SWE-bench-lite**-style tasks locally (small repos with tests) · Exercism tracks as agent tasks · your own repos' failing tests
- τ-bench retail/airline domains (customer-service agent eval) · WebArena self-hosted sites · MiniWoB · OSWorld VMs
- GAIA validation set (tool-using Q&A) · BrowseComp-style research prompts (create your own)
- Mock enterprise environments: sample CRM/ERP schemas (Chinook, Northwind, AdventureWorks), fake email/calendar servers, GitHub test orgs
- Open-source agent repos to read: OpenHands https://github.com/All-Hands-AI/OpenHands · SWE-agent https://github.com/SWE-agent/SWE-agent · Aider https://github.com/Aider-AI/aider · Cline · Goose https://github.com/block/goose · Open Deep Research implementations · LangGraph examples

## If you only have 1 week
Day 1: *Building Effective Agents* + build the raw loop (Note 09). Day 2: tool design + tool-selection eval. Day 3: your first MCP server + Inspector. Day 4: memory + checkpoint/resume. Day 5: security — trifecta threat model + injection red-team. Day 6: eval suite with pass^k. Day 7: port to LangGraph or an SDK and write a comparison.

Next → [[13-Projects-and-Checkpoint]]
