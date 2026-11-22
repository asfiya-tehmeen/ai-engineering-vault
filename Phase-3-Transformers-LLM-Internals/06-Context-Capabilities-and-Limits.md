---
tags: [phase-3, context, hallucination, limits]
---
# 06 — Context, Capabilities & Limits

> Understanding *why* LLMs fail is what lets you design reliable systems. This note is the bridge to Phases 4–7.

## 1. The context window
- **Context window** = max tokens (input + output [+ reasoning]) the model can attend to in one call. Modern models span from ~8k to **1M+** tokens (verify per model).
- Cost/latency scale with context: prefill compute O(T²) for attention (mitigated by Flash/sparse), KV cache O(T) memory, API price per input token.
- **Effective vs advertised context:** performance degrades before the limit. Findings: **"Lost in the middle"** (Liu et al. 2023) — information in the middle of long inputs is used worse than at the start/end (improved but not eliminated in newer models); **context rot** — accuracy drifts down as context grows, with distractors worsening it; multi-hop/aggregation over long context is much harder than single-needle retrieval ("needle-in-a-haystack" is too easy). Benchmarks: RULER, NoLiMa, LongBench, HELMET, MRCR, long-context agent benchmarks.
- **Practical rules:** put instructions & key info at start/end; keep context *clean and relevant* (**context engineering**, Phase 4); retrieve rather than stuff when possible; summarize/compact long histories; measure on *your* task.
- Multi-turn drift: models can "lose the thread" in long multi-turn interactions (active research).

## 2. In-context learning (ICL)
- Models adapt from examples in the prompt **without weight updates**: zero-shot, few-shot, instructions, demonstrations. Mechanisms include induction heads/implicit gradient-descent-like computation.
- Sensitive to example order, format, label balance, and recency. More/better examples help up to saturation; for many tasks, good instructions + 3–5 diverse examples suffices.
- **Chain-of-thought (CoT):** asking for step-by-step reasoning improves multi-step tasks by giving the model **more serial computation (tokens) to think with**. Reasoning models do this natively and internally. CoT text may **not faithfully reflect** the true computation — don't treat it as an audit log.

## 3. Capabilities (what LLMs are good at, broadly)
Language understanding/generation, translation, summarization, extraction/classification, coding (generation, debugging, refactoring, agentic SWE), math with reasoning models, tool use/agents, retrieval-grounded QA, multimodal understanding (images, charts, PDFs, audio, video), structured output, role-play/persona, brainstorming. Competence is **"jagged"**: superhuman at some tasks, brittle at trivial ones; skill varies by phrasing and domain.

## 4. Fundamental limitations (design around these)
| Limitation | Why | Mitigation |
|---|---|---|
| **Hallucination (confabulation)** | trained to produce *plausible* continuations; no built-in "I don't know"; rewarded for answering; rare facts under-learned (long-tail) | RAG/grounding with citations, tool use, constrained outputs, abstention training/prompts, verification, evals, human review |
| **Knowledge cutoff & staleness** | static weights | retrieval, web search tools, date in prompt |
| **No true memory across sessions** | stateless API | external memory/stores (Phase 6) |
| **Non-determinism** | sampling, numerics | low T, seeds, schema constraints, evals w/ variance |
| **Prompt sensitivity/brittleness** | distributional | eval-driven prompt iteration, robustness tests |
| **Poor at exact computation, counting, long arithmetic** (without tools) | token-level, no exact algorithms | code execution/calculator tools |
| **Planning/long-horizon failure; compounding errors** | per-step error accumulates | decomposition, verification, checkpoints, human-in-loop |
| **Sycophancy** | preference training | neutral prompting, adversarial evals |
| **Prompt injection / jailbreaks** | instructions & data share one channel | privilege separation, input/output filtering, least-privilege tools (Phase 7) |
| **Bias & fairness issues** | training data | evaluation by slice, guardrails |
| **Confident wrong answers; miscalibration** | RLHF reduces calibration | logprobs/self-consistency, verifiers |
| **Overreliance risk** (humans) | fluency | UX that shows sources/uncertainty |
| **Privacy/memorization** | training data leakage | data handling, no sensitive training, DLP |
| **Cost/latency** | tokens | caching, smaller models, routing |
**Types of hallucination:** factual (wrong facts), faithfulness (contradicts provided context), fabricated citations/URLs/APIs, instruction-violating. **Intrinsic vs extrinsic.** Detection: self-consistency, NLI/entailment vs sources, logprob signals, verification by retrieval, LLM-judge with evidence.

## 5. Reasoning & "thinking" models
- Generate long hidden/visible **reasoning traces** before the answer; trained with RLVR ([[04-Post-Training-and-Alignment]]).
- API controls: thinking **budget / effort** levels; reasoning tokens are billed as output; some providers return summarized or encrypted reasoning.
- Strong at math, code, planning, multi-step analysis; **overthinking** simple tasks wastes cost/latency; sometimes worse at casual chat/format adherence. Choose effort per task; **route** easy queries to fast models.
- **Faithfulness caveat:** displayed reasoning may omit real drivers; monitor outputs + actions, not just CoT.

## 6. Multimodal models
- **Vision-language:** image → ViT/encoder patches → projector → tokens in the LLM; documents/charts/screens/UI understanding; OCR-free doc QA; *token cost scales with image size/detail*.
- **Audio:** speech-to-text (Whisper-class), native audio-in/out models, realtime voice APIs. **Video:** frame sampling + long context.
- **Image/video generation:** diffusion/flow transformers; native image generation inside LLMs.
- Failure modes: counting, spatial reasoning, small text, hallucinated details, OCR errors on tables → verify with structured extraction + validation.

## 7. Open-weight vs closed (proprietary)
| | Closed API | Open-weight |
|---|---|---|
| Quality frontier | usually highest | close behind on many tasks, sometimes leading in specific areas |
| Control/customization | prompt, limited fine-tuning | full fine-tuning, quantization, on-prem |
| Data privacy | vendor terms/zero-retention options | data never leaves your infra |
| Ops burden | none | you run GPUs, serving, security |
| Cost | per token | GPU cost; cheaper at scale/high utilization |
| Latency | provider-dependent | tunable |
| Risks | vendor lock-in, deprecations, rate limits | licensing (read it: Apache-2.0/MIT vs custom community licenses), safety responsibility |
FDE relevance: **regulated/air-gapped customers often require open-weight + on-prem/VPC**; others prefer managed APIs via Bedrock/Vertex/Azure for compliance.

## 8. Benchmarks — how to read them (and not be fooled)
- Knowledge: MMLU/MMLU-Pro, GPQA(-Diamond), HLE; math: AIME, MATH; code: HumanEval (saturated), LiveCodeBench, **SWE-bench Verified/Pro**, Terminal-Bench; agents: τ-bench, BrowseComp, OSWorld, GAIA; instruction following: IFEval; long context: RULER; human preference: **LMArena**; safety: HarmBench.
- Pitfalls: **saturation**, **contamination**, **prompt/harness differences**, cherry-picked settings (pass@k vs pass@1, tool use allowed?), vendor-reported vs independent, benchmark ≠ your task. **Trust your own evals** (Phase 7).
- Independent trackers: LMArena, Artificial Analysis, Epoch AI, Scale SEAL leaderboards, vals.ai.

## 9. Choosing a model (decision checklist)
1. **Task type & difficulty** (extraction vs reasoning vs agentic coding). 2. **Quality on your eval set** (the only real answer). 3. **Latency target** (TTFT/throughput). 4. **Cost** at projected volume. 5. **Context needs.** 6. **Modalities.** 7. **Tool-use & structured-output reliability.** 8. **Privacy/compliance/residency/licensing.** 9. **Rate limits & availability/SLA.** 10. **Fine-tunability.** 11. **Vendor stability & deprecation schedule.**
Strategy: start with the strongest model to establish a quality ceiling → **move down** (smaller/cheaper models, caching, routing, fine-tuning) while holding eval scores. Model **families** typically offer tiers: large/flagship (hard tasks), mid (workhorse), small/fast (high-volume, classification, routing).

## 10. Mental models to carry forward
- LLM = **very large conditional distribution over text** shaped by post-training; powerful *pattern completion + learned procedures*, not a database or a calculator.
- **Everything in context influences output**; garbage in → garbage out; ordering and framing matter.
- **Reliability comes from the system** (retrieval, tools, validation, evals, fallbacks, human review), not from the model alone — the central thesis of Phases 4–8.

## Self-check
1. Why hallucinate? Give 5 mitigations. 2. What's lost-in-the-middle? 3. When would you pick open-weight? 4. Why can a benchmark score mislead? 5. How does CoT help, mechanically? 6. Why shouldn't you trust displayed reasoning as an explanation?

Next → [[07-Build-GPT-From-Scratch]]
