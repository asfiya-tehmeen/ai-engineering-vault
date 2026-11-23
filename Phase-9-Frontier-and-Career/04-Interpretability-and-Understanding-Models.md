---
tags: [phase-9, interpretability, mechanistic]
---
# 04 — Interpretability & Understanding Models

> **Source note:** my search surfaced Anthropic's 2025 **attribution-graph/circuit-tracing** methods pages (with their own caveat that the "replacement model" is an *indirect* approximation whose mechanisms need validation), a 2026 paper claiming small neuron-level circuits can control a standard agreement benchmark, a 2026 preprint on **LLM-annotated attribution graphs**, and a January 2026 survey on *actionable* interpretability. **No 2026 lab publications were retrieved**, so check transformer-circuits.pub for the latest. Everything here is a research area with **active disagreement**; treat claims as hypotheses.

## 1. Why interpretability matters to engineers
- **Debugging & trust:** understand *why* a model fails (shortcut features, spurious correlations).
- **Safety:** detect deception/hidden goals, verify refusals, audit for dangerous knowledge.
- **Control:** **steer** behavior (add/subtract directions) without retraining.
- **Science:** learn what representations/algorithms training discovers.
- **Practicality check:** today's tools give *partial, expensive, sometimes unreliable* insight; they're not yet routine production monitors — but ideas (probes, steering, feature-level evals) are creeping into practice.

## 2. Toolbox from easy to hard
| Method | What it does | Strength | Limitation |
|---|---|---|---|
| **Behavioral analysis** | prompt variations, counterfactuals | black-box, easy | can't see mechanism |
| **Attention visualization** | which tokens attend to which | intuitive | attention ≠ explanation (not necessarily causal) |
| **Probing** | train a classifier on hidden states to predict a property | finds *decodable* info | decodable ≠ used; probe capacity confound |
| **Logit lens / tuned lens** | decode intermediate residual stream through the unembedding | see how predictions evolve by layer | early layers aren't in "output space"; tuned lens helps |
| **Activation/causal patching** | replace activations from a corrupted run with clean ones (or vice versa) to find components that *cause* behavior | causal evidence | needs careful baselines; combinatorial |
| **Path patching / causal tracing (ROME-style)** | locate where facts are stored/recalled | localization | "localization ≠ editing success" debates |
| **Direct logit attribution** | each component's contribution to the final logits | simple decomposition | linear approximation |
| **Sparse autoencoders (SAEs)** | decompose activations into sparse, interpretable features | tackles superposition | reconstruction error, feature splitting/absorption, dead features, interpretability of labels |
| **Transcoders / crosscoders / replacement models** | learn interpretable stand-ins for MLPs/layers across models/layers | enables circuit-style analysis | approximation fidelity |
| **Attribution graphs / circuit tracing** | trace feature→feature influence for a specific prompt | rich per-prompt "biology" | expensive, partial, human-labor-heavy (LLM annotation research aims to automate), mechanisms need validation |
| **Representation engineering / steering vectors** | find directions (e.g., honesty, refusal) and add/subtract them | control levers | brittle, side effects |
| **Interpretability agents/automation** | LLMs label features, propose hypotheses | scalability | verification burden |

## 3. Core concepts
- **Residual stream as shared bus:** components read/write additively; analysis often decomposes the final logits into contributions from attention heads/MLPs/layers.
- **Superposition:** models represent **more features than dimensions** by using **nearly-orthogonal, sparse** directions; features aren't aligned with neurons → individual neurons are **polysemantic**. (*Toy Models of Superposition* shows sparsity + importance drive superposition.)
- **Features vs circuits:** *features* = directions with meaning; *circuits* = subgraphs of features/components implementing an algorithm (e.g., **induction heads**: previous-token head + induction head → in-context copy; indirect-object-identification circuits; modular arithmetic "Fourier" circuits; "grokking" mechanisms).
- **Linear representation hypothesis:** concepts often correspond to linear directions (useful for probes/steering) — approximately true, with exceptions.
- **MLPs as key–value memories**; attention as information movers; layer-wise **iterative refinement** of predictions.
- **Faithfulness/validation problem:** an explanation (feature label, circuit diagram) must be **tested by intervention** (ablate/patch/steer) to show it's causally relevant; otherwise it's a story.

## 4. Sparse autoencoders (hands-on)
**Architecture:** for activations `x ∈ ℝ^d` (e.g., residual stream or MLP output at one layer): 
```
f = ReLU(W_enc (x − b_dec) + b_enc)       # sparse code, dimension m ≫ d (e.g., 8–64×)
x̂ = W_dec f + b_dec                        # decoder columns = feature directions (often norm-constrained)
loss = ‖x − x̂‖² + λ‖f‖₁                    # reconstruction + sparsity (variants: top-k SAEs, JumpReLU, gated SAEs)
```
```python
import torch, torch.nn as nn, torch.nn.functional as F
class SAE(nn.Module):
    def __init__(s, d, m):
        super().__init__(); s.enc = nn.Linear(d, m); s.dec = nn.Linear(m, d, bias=True)
        with torch.no_grad(): s.dec.weight.copy_(F.normalize(s.dec.weight, dim=0))
    def forward(s, x):
        f = F.relu(s.enc(x - s.dec.bias)); xh = s.dec(f); return xh, f
def sae_loss(x, xh, f, lam=1e-3): return F.mse_loss(xh, x) + lam * f.abs().sum(-1).mean()
# After each step: renormalize decoder columns to unit norm. Train on cached activations from a small LM (e.g., GPT-2 small / your tinyformer).
```
**Evaluate:** reconstruction loss/variance explained, **L0** (avg active features), **loss recovered** when splicing the SAE into the model (cross-entropy delta), % dead features, feature interpretability (top-activating examples; automated labeling with an LLM + scoring), **downstream causal tests** (ablate/steer a feature and check behavior).
**Known issues:** feature splitting (granularity depends on dictionary size), absorption, imperfect reconstruction ("dark matter"), sensitivity to hyperparameters, labels that look plausible but aren't causal.

## 4b. Circuits & attribution graphs, in brief
- **Replacement model:** swap the model's MLPs with **cross-layer transcoders** (sparse features) so a prompt's computation becomes a graph of features; **attribution graphs** show influence paths from input tokens → features → output logit.
- **Findings reported in published work (2025):** multi-step reasoning "in the head," planning (e.g., choosing rhyme targets before writing a line), shared multilingual features, arithmetic via parallel pathways, features related to refusal/hallucination decisions. *These are case studies on specific prompts; generality and faithfulness are debated, and the authors caution about approximation error.*
- **Costs:** heavy compute, manual annotation (now partially automated by LLMs), per-prompt analysis.
- **Alternative view (2026 preprint noted in search):** small neuron-level circuits may suffice for some benchmark behaviors, challenging the "too messy" assumption — one more reason to treat the field as unsettled.

## 5. Practical labs (use TransformerLens/nnsight; verify current APIs)
1. **Logit lens:** run GPT-2 small on a prompt; decode each layer's residual through `ln_final` + unembedding; plot the correct token's rank by layer.
2. **Induction heads:** feed repeated random token sequences; compute per-head induction scores; ablate the top heads and measure the loss increase on the second repetition. Reproduce the phase change during training in a tiny model (your tinyformer from Phase 3).
3. **Activation patching:** on an IOI-style or factual-recall prompt pair (clean vs corrupted), patch residual streams layer-by-layer/position-by-position; produce a heatmap of causal effect.
4. **Probing with controls:** train linear probes for part-of-speech or sentiment by layer; add **control tasks** (random labels) to measure probe capacity; compare to patching evidence.
5. **Train an SAE** (code above) on a layer of GPT-2 small; label 20 features by top examples; run ablation/steering tests; report L0/loss-recovered.
6. **Steering vectors:** compute a mean-difference direction between two contrastive prompt sets (e.g., formal vs casual, honest vs sycophantic); add it at a layer during generation; measure effect and side effects.
7. **Refusal direction (research replication-lite):** find a direction separating refused vs complied prompts in a small open chat model; test ablating it (research context only; handle responsibly).
```python
# TransformerLens-style sketch (verify API): logit lens
from transformer_lens import HookedTransformer
model = HookedTransformer.from_pretrained("gpt2")
toks = model.to_tokens("The Eiffel Tower is located in the city of")
logits, cache = model.run_with_cache(toks)
for l in range(model.cfg.n_layers):
    resid = cache["resid_post", l][0, -1]                       # last position
    layer_logits = model.unembed(model.ln_final(resid[None, None]))[0, 0]
    print(l, model.tokenizer.decode(layer_logits.argmax()))
```

## 6. Interpretability and safety/control
- **Probes for deception/hidden knowledge**, **feature monitors** (e.g., "is the model considering X?"), **auditing for hidden objectives** (blind auditing games), **steering to reduce harmful behavior**, **unlearning evaluation**.
- Current limitations for production monitoring: coverage, false positives, cost, adversarial robustness. Use as a **research-informed complement** to behavioral evals, not a replacement.

## 7. Evaluating interpretability claims (how to be skeptical)
Ask: Is there an **intervention** supporting the claim? Is it **reproduced** across prompts/seeds/models? What's the **baseline** (random directions, shuffled labels)? Are labels **validated** (blind scoring)? What's the **approximation error** of the surrogate? Does the explanation **predict** new behavior (e.g., a steering edit works)? Beware **illusions**: seductive top-k examples, cherry-picked prompts, attention-as-explanation.

## 8. Resources (see Note 11) & how to start
Neel Nanda's guides and **ARENA** curriculum, TransformerLens, nnsight, Neuronpedia (feature browser), transformer-circuits.pub papers, SAELens. Plan: reproduce the **induction-head** result → logit lens/patching → train one SAE → write up what you learned (including failures).

## Self-check
1. Define superposition and polysemanticity. 2. Probing vs patching: what does each establish? 3. Write the SAE objective and name three failure modes. 4. How do attribution graphs differ from SAEs, and what are their caveats? 5. Design an experiment to test whether a feature is causally used. 6. Why is "attention weights as explanation" problematic?

Next → [[05-Data-Continual-Learning-and-Open-Problems]]
