---
tags: [phase-4, prompting]
---
# 02 — Prompt Engineering

> A prompt is the **program** you give a very capable but literal-minded, context-dependent colleague who has never met you. The craft is: supply the missing context, state the task and success criteria, show examples, and **test**. "Vibes" are not a method — pair every prompt change with an eval ([[../Phase-2-ML-Deep-Learning/04-Evaluation-and-Metrics]], Phase 7).

## 1. Core principles
1. **Be clear and direct.** Say what you want, for whom, in what format, at what length. Avoid ambiguity; ambiguity gets filled with the model's priors.
2. **Give context and the *why*.** "Summarize for a CFO deciding whether to renew the contract; she cares about cost and risk" beats "Summarize." Reasons let the model generalize to edge cases.
3. **Define success.** Acceptance criteria, what to include/exclude, how to handle uncertainty/missing info ("if the answer isn't in the document, say 'not found'").
4. **Show, don't just tell** — 3–5 diverse, representative examples (few-shot). Include edge cases. Keep format identical between examples.
5. **Structure the prompt** — separate instructions, context, examples, and the user input with clear delimiters (XML-style tags work well on Claude; Markdown headers or triple quotes also fine). Put **long documents first, the question last**.
6. **Let it think** on hard tasks — via reasoning/effort settings on reasoning models, or "think step by step before answering" on non-reasoning ones. Ask for the final answer in a delimited field so you can parse it.
7. **Specify output format precisely** — or better, use **structured outputs** ([[03-Structured-Outputs-and-Tool-Use]]).
8. **Iterate empirically** — change one thing, measure on a test set, keep a changelog.
9. **Prefer positive instructions** ("write in plain prose") over long lists of don'ts; explain constraints.
10. **Don't over-prescribe for newer models.** Prompts written for older, weaker models (excessive step-by-step scaffolding, shouted ALL-CAPS rules, redundant reminders, "you are a world-class expert…") are often *counter-productive* on current models and can reduce quality or cause rigid, over-literal behavior. Re-test and **prune** old prompts when you upgrade models ("prompt cruft").

## 2. Anatomy of a strong prompt
```text
<role>You are a claims analyst at a health insurer.</role>

<task>
Decide whether the claim below is covered under the policy excerpt. 
</task>

<policy>
{{policy_text}}
</policy>

<claim>
{{claim_text}}
</claim>

<instructions>
- Base your decision ONLY on the policy text. If the policy is silent, answer "unclear".
- Quote the clause(s) that support your decision.
- Be concise; a claims adjuster will read this.
</instructions>

<output_format>
Return JSON: {"decision": "covered"|"not_covered"|"unclear", "clauses": [string], "rationale": string}
</output_format>
```
- **System vs user:** system = stable role/rules/format/tools policy (cache it); user = per-request data. Data from untrusted sources (users, web, documents) goes in clearly delimited sections and is **treated as data, not instructions** (prompt-injection hygiene — Phase 7).
- Use **variables** (`{{...}}`) via templating (Jinja2/f-strings) and keep prompts in files, not scattered strings.

## 3. Techniques toolbox
| Technique | When | Notes |
|---|---|---|
| **Zero-shot instruction** | simple tasks | start here |
| **Few-shot examples** | format/style/labeling tasks | diverse, representative; order & label balance matter; wrap in `<example>` tags |
| **Chain-of-thought / reasoning** | multi-step logic/math/analysis | reasoning models/effort do this natively; for others "think first, then answer in `<answer>`" |
| **Role/persona** | tone, domain framing | modest effect on accuracy; real value in tone/format |
| **Decomposition / prompt chaining** | complex workflows | split into steps (extract → analyze → draft → critique); each call simpler, testable, cacheable; add code between steps |
| **Self-critique / reflection** | quality-sensitive outputs | generate → critique against rubric → revise; costs 2–3× |
| **Self-consistency / best-of-N** | reasoning accuracy | sample N, vote/judge; N× cost |
| **Retrieval (RAG)** | needs external/up-to-date facts | Phase 5 |
| **Tool use** | calculations, lookups, actions | [[03-Structured-Outputs-and-Tool-Use]] |
| **Constrain with schema** | machine-consumed output | prefer over "please output JSON" |
| **Permission to say "I don't know"** | reduce hallucination | explicitly allow & define abstention |
| **Quote-first** | long-doc QA | "first extract relevant quotes in `<quotes>`, then answer" improves grounding |
| **Meta-prompting** | drafting/improving prompts | ask a model to critique/rewrite your prompt, then **verify with evals** |
| **Prompt generators/optimizers** | at scale | DSPy, console prompt improvers, evolutionary methods (Phase 7) |

## 4. Writing system prompts for products
Include: **who/what** (role & product context), **audience**, **goals & priorities**, **hard rules** (safety, scope, escalation), **tone/style**, **tool-use policy** (when to call what), **output format**, **uncertainty policy**, **examples of edge cases**, and **current date/time & user info** if needed (put volatile items *late* to preserve cache — Note 05).
- Keep it as short as it can be while remaining unambiguous. Every sentence should change behavior in some test.
- **Version it** (`prompts/support_v7.md`), review in PRs, tag with eval scores.
- Avoid contradictions (the model resolves them unpredictably); keep one source of truth.
- **Mid-conversation operator instructions:** newer Claude models accept `{"role":"system"}` messages inside `messages` for mode switches without busting the cached prefix; otherwise append reminders in user turns.

## 5. Output control without prefill
Since assistant prefill is removed on newer Claude models, control format by: (1) schema-constrained outputs; (2) explicit format section + 1–2 examples; (3) XML-tag wrapping of the answer; (4) stop sequences for delimiters; (5) post-validation + retry.

## 6. Reasoning-model prompting (2026 practice)
- Don't tell reasoning models *how* to reason step by step; give the problem, constraints, and success criteria. Over-prescribing can hurt.
- Use `effort` (Claude), reasoning effort (OpenAI), `thinking_level` (Gemini) to trade quality for latency/cost; **tune per route** (classification at low; hard analysis/coding at high).
- Reasoning tokens are billed as output and add latency — measure cost per *completed task*.
- If you need explanations for users, request a *final* concise rationale; don't depend on raw thinking text (it may be hidden/summarized and isn't a faithful audit log).

## 7. Anti-patterns (and fixes)
| Anti-pattern | Fix |
|---|---|
| Vague ask ("make it better") | specify audience, criteria, length, examples |
| Giant instruction dump with contradictions | prioritize; remove duplicates; test |
| Examples that all look alike | add diverse and edge-case examples |
| Telling the model to "be accurate/don't hallucinate" as the only safeguard | provide sources, allow abstention, add verification |
| Parsing free text with regex | use structured outputs |
| Asking for JSON in prose and hoping | schema-constrained output |
| Untrusted text mixed into instructions | delimit as data; privilege separation |
| Prompt lives only in someone's head/notebook | version control + evals |
| One mega-prompt doing 6 jobs | chain, or route to specialized prompts |
| Optimizing on 3 examples | ≥30–100 representative cases with CIs |
| Never re-testing after a model upgrade | re-run evals; prune old cruft |
| Stuffing the entire knowledge base | retrieve; cache stable parts |

## 8. A prompt-development workflow (do this every time)
1. **Write the success criteria** and 20–50 test inputs (include hard/edge cases, adversarial inputs, and "should refuse/abstain" cases).
2. **Baseline** with a simple prompt on the cheapest plausible model. Measure.
3. **Read failures** (error analysis). Categorize.
4. **Change one variable**: add context / example / format / reasoning effort / chain step.
5. **Re-run all tests**; keep a table: prompt version × metric × cost × latency.
6. **Freeze** a golden set; hold out a validation slice to avoid overfitting prompts to examples.
7. **Ship with monitoring**; collect real failures into the test set (flywheel).

## 9. Templates worth memorizing
**Extraction:** "From the text in `<doc>`, extract the fields in the schema. If a field is absent, use null. Do not guess."
**Classification:** "Classify into exactly one of: A (definition), B (definition)… Choose 'other' only if none apply. Return label and a one-sentence reason."
**Summarization:** audience + purpose + length + must-include + must-exclude + format.
**Rewrite/transform:** preserve meaning/facts; change tone/format; list what must not change.
**Critique/judge:** rubric with anchored scale and examples; require evidence quotes before score (Phase 7).
**Planner:** "Break the task into steps; for each, state the tool or info needed and success check."

## 10. Prompt security basics (preview of Phase 7)
- Anything from users, web pages, files, tool outputs can contain **instructions that try to hijack the model** (prompt injection). Delimit and label untrusted content, restrict tool permissions, require human approval for risky actions, validate outputs, and never put secrets in prompts you wouldn't want leaked.
- Practice with Lakera's Gandalf (see [[10-Resources-and-Practice-Links]]).

## Self-check
1. Rewrite "summarize this" into a production-grade prompt. 2. Why put documents before the question? 3. When does few-shot beat instructions? 4. Why can old "be a world-class expert, think step by step!!" prompts hurt new models? 5. How do you control output format without prefill? 6. What belongs in the system prompt vs user message?

Next → [[03-Structured-Outputs-and-Tool-Use]]
