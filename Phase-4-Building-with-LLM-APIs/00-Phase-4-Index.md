---
tags: [phase-4, index]
---
# Phase 4 — Building with LLM APIs

> [!abstract] Why this phase exists
> This is where you start *shipping*. Every LLM product — chatbots, extractors, copilots, agents — is built from a small set of API primitives: **messages, prompts, structured outputs, tool calls, streaming, caching, batching, multimodal inputs, and context management**. Master these and Phases 5–8 (RAG, agents, evals, FDE) are compositions of them.

**Duration:** 3–5 weeks · **Prereq:** [[../Phase-1-Foundations/00-Phase-1-Index|Phase 1]] (HTTP, Python, async, Pydantic) and [[../Phase-3-Transformers-LLM-Internals/00-Phase-3-Index|Phase 3]] (tokens, KV cache, decoding).

> [!warning] This is the fastest-changing phase
> API shapes change every few months. Code in this phase was written against **Anthropic's Python SDK 1.x and the Claude 5.x model family as documented in November 2026**; OpenAI and Gemini details come from search results and may lag. **Always cross-check the provider docs linked in [[10-Resources-and-Practice-Links]] before copying code into production.** Where I'm unsure, I say so.

## Topic map
| # | Note | Core ideas |
|---|---|---|
| 01 | [[01-API-Fundamentals]] | messages & roles, stateless calls, request/response anatomy, tokens/usage, stop reasons, errors, auth, SDKs, providers, model selection, parameters that changed |
| 02 | [[02-Prompt-Engineering]] | principles, structure, XML tags, examples, reasoning/effort, system prompts, chaining, anti-patterns, prompt versioning |
| 03 | [[03-Structured-Outputs-and-Tool-Use]] | JSON Schema, `parse`, strict tools, the tool loop, parallel calls, tool design, server tools, validation/repair |
| 04 | [[04-Streaming-Async-Batch-Reliability]] | SSE streaming, async fan-out, Batch API, retries, timeouts, idempotency, fallbacks, rate limits, routing |
| 05 | [[05-Context-Engineering-Caching-Cost]] | context budgets, prompt caching, compaction, context editing, memory, token counting, cost & latency modeling |
| 06 | [[06-Multimodal-and-Documents]] | images, PDFs, Files API, citations, audio/video, document pipelines, web search/fetch/code execution |
| 07 | [[07-Multi-Provider-and-App-Patterns]] | OpenAI/Gemini differences, gateways & abstraction, common app architectures, safety basics, UX patterns |
| 08 | [[08-Exercises]] | 80 exercises |
| 09 | [[09-QA-Interview-Questions]] | 60 Q&A |
| 10 | [[10-Resources-and-Practice-Links]] | docs, courses, cookbooks, practice |
| 11 | [[11-Projects-and-Checkpoint]] | 6 projects + self-assessment |

## Weekly plan (4 weeks)
| Week | Focus | Output |
|---|---|---|
| 1 | Notes 01, 02 | CLI chatbot with history + prompt library; token/cost tracker |
| 2 | Note 03 | schema-validated extractor + a tool-using assistant (hand-written loop) |
| 3 | Notes 04, 05 | async batch processor with retries, caching, cost dashboard |
| 4 | Notes 06, 07 + capstone | multimodal document-intake service (see [[11-Projects-and-Checkpoint]]) |

## Completion checklist
- [ ] Make a raw HTTP call and an SDK call; explain every field of the request and response
- [ ] Explain why chat history is *your* responsibility, and what it costs
- [ ] Write prompts with clear structure; iterate against a test set instead of vibes
- [ ] Extract validated JSON using Pydantic + schema-constrained outputs; handle refusals and truncation
- [ ] Write a complete tool-use loop by hand (parallel calls, errors, `stop_reason`)
- [ ] Stream tokens to a terminal and a web client (SSE)
- [ ] Fan out 100+ calls concurrently with a semaphore, retries, and backoff; use the Batch API for offline work
- [ ] Use prompt caching and prove it works from `usage` fields; compute cost per request
- [ ] Send images and PDFs; get citations
- [ ] Call at least two providers behind a thin abstraction
- [ ] Capstone shipped on GitHub

## Cost tip for learners
Use the **smallest model that works** for experiments (e.g., Claude Haiku 5.5, a Gemini "flash"-tier model, or a small OpenAI model) and a local OpenAI-compatible server (Ollama/vLLM) for free iteration. Set **spend limits** in each provider console on day one, and never run an unbounded loop against a paid API.

Back to [[../00-Roadmap]].
