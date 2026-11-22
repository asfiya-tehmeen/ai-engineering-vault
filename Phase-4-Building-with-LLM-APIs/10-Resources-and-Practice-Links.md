---
tags: [phase-4, resources]
---
# 10 — Resources & Practice Links (Phase 4)

> ⭐ = top pick. **Docs are the source of truth** — API features change monthly. I verified Anthropic details against its SDK reference as of Nov 2026; OpenAI and Gemini details are from search snippets, so check their docs. I haven't opened every link below; if one moves, search the title.

## Official docs (bookmark all)
- ⭐ **Anthropic / Claude**: docs https://docs.claude.com/ (redirects to the platform docs) · API reference & SDKs https://github.com/anthropics/anthropic-sdk-python · Release notes/changelog (check docs site) · Console https://console.anthropic.com/
- ⭐ **OpenAI**: https://platform.openai.com/docs — Responses API, function calling, structured outputs, batch, reasoning · changelog (in docs) · Cookbook https://cookbook.openai.com/
- ⭐ **Google Gemini API**: https://ai.google.dev/gemini-api/docs — Gemini 3 developer guide, thinking, structured output, function calling, caching, batch · changelog https://ai.google.dev/gemini-api/docs/changelog · LLM-friendly docs index https://ai.google.dev/gemini-api/docs/llms.txt
- Cloud platforms: AWS Bedrock https://docs.aws.amazon.com/bedrock/ · Google Vertex AI https://cloud.google.com/vertex-ai/docs · Microsoft Foundry / Azure OpenAI https://learn.microsoft.com/azure/ai-foundry/
- Open-model providers: Together https://docs.together.ai/ · Fireworks https://docs.fireworks.ai/ · Groq https://console.groq.com/docs · OpenRouter https://openrouter.ai/docs · Hugging Face Inference Providers https://huggingface.co/docs/inference-providers

## Prompt & context engineering (read these)
- ⭐ Anthropic — *Building Effective Agents* (workflow vs agent patterns) https://www.anthropic.com/engineering/building-effective-agents
- ⭐ Anthropic — *Effective Context Engineering for AI Agents* https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- ⭐ Anthropic — *Writing Effective Tools for Agents* https://www.anthropic.com/engineering/writing-tools-for-agents
- Anthropic — Prompt engineering overview (in the docs) + **interactive prompt engineering tutorial** https://github.com/anthropics/prompt-eng-interactive-tutorial · Courses repo https://github.com/anthropics/courses
- ⭐ Anthropic Cookbook (runnable examples: tool use, RAG, caching, evals, multimodal) https://github.com/anthropics/anthropic-cookbook
- OpenAI — Prompt engineering guide https://platform.openai.com/docs/guides/prompt-engineering · OpenAI Cookbook https://cookbook.openai.com/
- Google — Prompting strategies (Gemini docs) · Prompt design intro https://ai.google.dev/gemini-api/docs/prompting-intro
- Prompting Guide (community, broad techniques) https://www.promptingguide.ai/
- Lilian Weng — *Prompt Engineering* https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/
- Chip Huyen — *Building A Generative AI Platform*, *AI Engineering* (book, O'Reilly 2025) https://huyenchip.com/
- Eugene Yan — *Patterns for Building LLM-based Systems & Products* https://eugeneyan.com/writing/llm-patterns/
- Hamel Husain — *Your AI Product Needs Evals* & field guides https://hamel.dev/
- Simon Willison's blog (practical LLM tooling, prompt injection) https://simonwillison.net/
- Jason Liu — Instructor library + writing https://python.useinstructor.com/ · https://jxnl.co/

## Structured outputs & tool-use tooling
- Pydantic https://docs.pydantic.dev/ · **Instructor** https://python.useinstructor.com/ · **Pydantic AI** https://ai.pydantic.dev/ · **Outlines** (constrained generation for local models) https://github.com/dottxt-ai/outlines · XGrammar https://github.com/mlc-ai/xgrammar · `jsonschema` https://python-jsonschema.readthedocs.io/ · JSON Schema guide https://json-schema.org/learn/getting-started-step-by-step
- **MCP**: spec & docs https://modelcontextprotocol.io/ · Python SDK https://github.com/modelcontextprotocol/python-sdk · server registry/examples https://github.com/modelcontextprotocol/servers
- Function calling guides: OpenAI (platform docs) · Anthropic tool use (docs) · Gemini function calling (docs)
- `sqlglot` (SQL parsing/validation) https://github.com/tobymao/sqlglot · Chinook sample DB https://github.com/lerocha/chinook-database · Spider/BIRD text-to-SQL benchmarks https://yale-lily.github.io/spider · https://bird-bench.github.io/

## Reliability, gateways, observability
- tenacity (retries) https://tenacity.readthedocs.io/ · respx (mock httpx) https://lundberg.github.io/respx/ · anyio / asyncio docs
- **LiteLLM** (OpenAI-format gateway/SDK) https://docs.litellm.ai/ · **OpenRouter** https://openrouter.ai/ · Portkey https://portkey.ai/docs · Cloudflare AI Gateway https://developers.cloudflare.com/ai-gateway/
- Observability/eval platforms: **Langfuse** https://langfuse.com/docs · **LangSmith** https://docs.smith.langchain.com/ · **Braintrust** https://www.braintrust.dev/docs · **Arize Phoenix** https://docs.arize.com/phoenix · Helicone https://docs.helicone.ai/ · Promptfoo https://www.promptfoo.dev/docs/ · OpenTelemetry GenAI semantic conventions https://opentelemetry.io/docs/specs/semconv/gen-ai/
- Durable execution: Temporal https://docs.temporal.io/ · Inngest https://www.inngest.com/docs · Celery / RQ
- Rate-limit & capacity: provider docs (rate limits pages) · Token-bucket explainers

## Multimodal & documents
- Docling (IBM; document parsing) https://github.com/docling-project/docling · Unstructured https://docs.unstructured.io/ · Microsoft **MarkItDown** https://github.com/microsoft/markitdown · PyMuPDF https://pymupdf.readthedocs.io/ · pdfplumber · Tesseract OCR · cloud: AWS Textract, Google Document AI, Azure Document Intelligence
- Whisper https://github.com/openai/whisper · faster-whisper · Deepgram / AssemblyAI docs · ElevenLabs docs
- Presidio (PII detection/redaction) https://microsoft.github.io/presidio/
- Datasets for practice: FUNSD/CORD/SROIE (receipts/forms), DocVQA https://www.docvqa.org/, RVL-CDIP, SEC EDGAR filings https://www.sec.gov/edgar, CUAD (legal contracts) https://www.atticusprojectai.org/cuad

## Courses (short, practical)
- ⭐ DeepLearning.AI short courses (ChatGPT/Claude prompt engineering for developers, function calling, structured outputs, building agents, evaluating LLM apps) https://www.deeplearning.ai/short-courses/
- ⭐ Hugging Face Agents course / LLM course https://huggingface.co/learn
- Anthropic Academy / courses (see docs & https://github.com/anthropics/courses)
- OpenAI/Google/Microsoft free learning paths (search "Generative AI for Beginners" — Microsoft https://github.com/microsoft/generative-ai-for-beginners)
- Full Stack LLM Bootcamp / Full Stack Deep Learning https://fullstackdeeplearning.com/
- Maven/Hamel & Shreya "AI Evals for Engineers & PMs" (paid; evals preview for Phase 7)

## Practice (hands-on, with links)
- ⭐ **Anthropic prompt-engineering interactive tutorial** (Jupyter, graded exercises) https://github.com/anthropics/prompt-eng-interactive-tutorial
- ⭐ **Gandalf** (prompt-injection game) https://gandalf.lakera.ai/ · Lakera Gandalf agent challenges · Prompt Airlines https://promptairlines.com/ · Gray Swan arena https://www.grayswan.ai/
- **Promptfoo** — build and run eval suites locally https://www.promptfoo.dev/
- **LMArena** — compare models blind https://lmarena.ai/
- Kaggle GenAI intensive / notebooks https://www.kaggle.com/learn-guide/5-day-genai
- **Build-a-thing** idea lists: Anthropic cookbook recipes · OpenAI cookbook examples · "awesome-llm-apps" https://github.com/Shubhamsaboo/awesome-llm-apps
- Open datasets for extraction evals: CORD, FUNSD, SROIE, EDGAR 10-K sections, Enron emails, Spider (SQL), GSM8K (reasoning), IFEval (instruction following) https://github.com/google-research/google-research/tree/master/instruction_following_eval
- Local models for free iteration: Ollama https://ollama.com/ · LM Studio https://lmstudio.ai/ · vLLM https://docs.vllm.ai/
- Hackathons (great for portfolio + networking): lablab.ai, Devpost, Anthropic/OpenAI/Google community hackathons, MLH.

## Keep up (fast-moving; check weekly)
Provider changelogs (Anthropic, OpenAI, Google) · Simon Willison · Latent Space · Interconnects · Hamel/Eugene Yan · r/LocalLLaMA · Hacker News · AI Engineer conference talks (YouTube) https://www.ai.engineer/ · Anthropic engineering blog https://www.anthropic.com/engineering

## If you only have 1 week
Day 1: SDK + raw HTTP + usage logging. Day 2: Anthropic prompt tutorial (ch. 1–5). Day 3: structured outputs with Pydantic. Day 4: hand-written tool loop. Day 5: streaming + async fan-out. Day 6: prompt caching experiments. Day 7: PDF extraction mini-project.

Next → [[11-Projects-and-Checkpoint]]
