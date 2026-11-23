---
tags: [phase-7, serving, infrastructure, gpu]
---
# 06 — Serving & Infrastructure

> Many FDE engagements end with **"we need to run this inside our environment."** This note covers when to self-host, how to size and operate it, and how to put a gateway and SLOs around any model (hosted or self-hosted). Builds on [[../Phase-3-Transformers-LLM-Internals/05-Inference-and-Decoding]] (KV cache, batching, quantization).

## 1. Managed API vs cloud-hosted vs self-hosted
| Option | Pros | Cons | Choose when |
|---|---|---|---|
| **Provider API** (first-party) | best models, zero ops, newest features | data leaves boundary (check retention/ZDR), rate limits, vendor dependence | default for most apps |
| **Cloud-platform-hosted** (Bedrock/Vertex/Azure/Foundry) | IAM/VPC/private networking, residency, consolidated billing, compliance posture | features lag first-party; different quotas/model IDs/clients | customer standardizes on a cloud |
| **Hosted open-weight** (Together/Fireworks/Groq/etc.) | cheap, fast, many open models | third-party trust; limited customization | open-model cost/latency wins |
| **Self-hosted open-weight** (your GPUs/VPC/on-prem) | full control, data stays inside, customization (LoRA), predictable cost at scale, air-gap | you run GPUs/serving/safety; model quality gap vs frontier on some tasks; staffing | strict privacy/sovereignty, high steady volume, latency control, offline/edge |
**Self-hosting economics:** breakeven = (GPU $/hr ÷ sustained tokens/s × 3600) vs API $/token. **Utilization decides everything** — idle GPUs make self-hosting more expensive than APIs; high steady load (or batch workloads) makes it cheaper. Include engineering/on-call cost, redundancy (≥2 replicas for HA), and upgrade effort.

## 2. Serving engines (verify current features)
| Engine | Notes |
|---|---|
| **vLLM** | most popular open engine: PagedAttention, continuous batching, chunked prefill, prefix caching, quantization (AWQ/GPTQ/FP8), speculative decoding, multi-LoRA, tensor/pipeline parallel, OpenAI-compatible server |
| **SGLang** | strong for agents/structured outputs/repeated context (RadixAttention prefix reuse), fast scheduler |
| **TensorRT-LLM / Triton / NVIDIA Dynamo** | NVIDIA-optimized kernels & disaggregated serving; best raw perf on NVIDIA at the cost of build complexity |
| **TGI (Hugging Face)** | production server; check maintenance status |
| **llama.cpp / Ollama / LM Studio / MLX** | local/edge/CPU/Apple Silicon; GGUF quantization; dev & small deployments |
| **LMDeploy, DeepSpeed-MII, Ray Serve LLM, KServe, BentoML** | frameworks/orchestrators |
Run behind an **OpenAI-compatible API** so apps/gateways swap backends easily.

## 3. Sizing a deployment (worked method)
1. **Choose model & precision:** weights memory = params × bytes (bf16=2, FP8/int8=1, 4-bit≈0.5). 70B bf16 ≈ 140 GB; 4-bit ≈ 35–40 GB.
2. **KV cache per token** = `2 × layers × kv_heads × head_dim × bytes` (e.g., ~128 KB/token for an 8B-class GQA model at bf16). × context × concurrent sequences.
3. **Add overhead:** activations, CUDA graphs, framework buffers (~10–20%).
4. **Fit to GPUs:** per-GPU memory (e.g., 24 GB consumer, 40/80 GB A100, 80–141 GB H100/H200, larger newer parts); use tensor parallel across GPUs if weights+KV exceed one GPU; NVLink matters for TP.
5. **Throughput estimate:** decode is memory-bandwidth-bound → tokens/s per sequence ≈ bandwidth ÷ bytes read per token; **batching** multiplies aggregate throughput until KV memory or compute binds. **Measure with your prompts** (don't trust vendor charts).
6. **Replicas & headroom:** peak load × p95 token demand ÷ per-replica goodput; add N+1 redundancy and autoscaling margin.
7. **Cost per million tokens** = (GPU $/hr × GPUs) ÷ (sustained tokens/hr) × 1e6 — compare to API pricing.
Worked example to practice: *70B at 4-bit, 8k avg context, 40 concurrent users* → weights ≈ 40 GB; KV for GQA-80-layer-class ≈ ~0.3 MB/token ⇒ 8k ≈ 2.5 GB/seq ⇒ 40 seqs ≈ 100 GB → needs multi-GPU or KV quantization/shorter contexts/lower concurrency. (Compute exact numbers from the model's `config.json`.)

## 4. Performance engineering levers
- **Quantization** (weights & KV; AWQ/GPTQ/FP8/FP4/NVFP4 where supported) — validate quality on your evals.
- **Prefix caching** (system prompts/tools/RAG docs shared across requests) + **cache-aware routing** (send same-prefix requests to same replica; llm-d/Dynamo-style routers).
- **Continuous batching, chunked prefill** tuning (`max_num_seqs`, `max_num_batched_tokens`).
- **Speculative decoding / MTP** for latency-sensitive decode.
- **Disaggregated prefill/decode** at scale.
- **Parallelism:** TP for latency/fit, PP for very large models, **expert parallel** for MoE, data-parallel replicas for throughput.
- **Right-size context limits** per route; truncate/compress inputs.
- **Hardware:** match to workload — memory bandwidth/capacity for decode, FLOPs for prefill; consider specialized inference silicon (Groq/Cerebras/TPUs/Trainium) via providers.
- **Scheduler fairness & priorities:** interactive vs batch queues, preemption.

## 5. Production architecture
```
clients → API gateway/LLM gateway (auth, rate limit, routing, cache, logging, budgets, guardrails)
        → router (model aliases, A/B, fallback chains, cache-aware routing)
        → inference pool(s) (vLLM/SGLang replicas on GPU nodes; autoscaled) 
        → model store (weights in object storage; versioned), adapter store (LoRA)
        ↳ observability (metrics/traces/logs), secrets, queues for batch workloads
```
- **LLM gateways/proxies:** LiteLLM, Portkey, Cloudflare AI Gateway, Kong/Envoy AI gateway, cloud-native; features: unified API, key management, budgets, retries/fallbacks, semantic/exact caching, logging, guardrail hooks.
- **Kubernetes:** GPU node pools (NVIDIA device plugin/GPU Operator), pod scheduling by GPU type, autoscaling (HPA/KEDA on queue depth or GPU utilization/tokens-per-second; note slow model-load times → keep warm replicas, use faster weight loading/streaming, pre-pulled images), `KServe`/`Ray Serve`/Helm charts, node affinity, PodDisruptionBudgets, rolling updates with readiness probes (model loaded).
- **Model artifact management:** immutable versioned weights (hash), signed artifacts, license tracking, SBOM; load from fast storage; CPU/GPU memory pinning.
- **Multi-LoRA serving:** one base model + many adapters (per tenant/task) hot-swapped in the same engine → efficient multi-tenant customization.
- **Multi-region/DR:** replicate model stores; health-checked routing; capacity reservations; fallbacks to hosted APIs.
- **Security:** network isolation (private subnets), mTLS, authN/Z at gateway, secrets manager, egress controls, image scanning, patching, audit logs; **no public inference endpoints without auth**.
- **On-prem/air-gapped:** offline artifact mirrors (models, containers, Python wheels), internal registries, GPU supply/lead times, monitoring without SaaS, update/rollback procedures, hardware refresh planning.

## 6. Reliability engineering
- **SLOs:** availability, TTFT p95, tokens/s, error rate, queue wait; **error budgets**.
- **Health checks:** liveness vs readiness (weights loaded, a tiny generation succeeds); canary probes.
- **Overload handling:** admission control, queue limits + `429/503` with `Retry-After`, load shedding by priority, circuit breakers, timeouts; avoid unbounded queues (latency collapse).
- **Graceful degradation:** shorter context/outputs, smaller model, cached answers, hosted-API fallback.
- **Failure modes:** GPU OOM (KV growth), CUDA/driver mismatches, NCCL hangs, slow cold starts, noisy-neighbor contention, long-prompt starvation, memory fragmentation, model-load failures, thermal throttling, hardware faults (plan automated replacement).
- **Chaos/load tests:** kill replicas, saturate queues, send worst-case prompts.

## 7. Load testing & benchmarking
- **Tools:** vLLM benchmark scripts, GuideLLM, NVIDIA GenAI-Perf, k6/Locust/hey with streaming support, custom async clients.
- **Workload realism:** replay production prompt/length distributions (not uniform synthetic), include prefix-sharing patterns, streaming, concurrency ramps, mixed short/long.
- **Report:** TTFT/TPOT/e2e percentiles, throughput, goodput under SLO, error/timeouts, GPU utilization/memory, cost per M tokens, knee of the latency–throughput curve, saturation behavior.
- Compare configs (quantization, TP size, batch limits, speculative decoding, prefix cache on/off) with **same eval quality** checks — a faster config that degrades answers isn't a win.

## 8. Cost optimization (self-hosted & hosted)
Right-size models (route easy → small), quantize, cache prefixes/answers, batch offline work off-peak, spot/preemptible for batch (with checkpointing), reserved/committed capacity for baseline load, autoscale down to zero for dev, consolidate adapters on shared bases, cut context length, optimize prompts, cap output tokens, track **cost per successful task**. (FinOps in [[08-Governance-Compliance-and-FinOps]].)

## 9. Edge & local deployment
Small models (1–8B, quantized) on laptops/phones/devices/browsers (llama.cpp, MLX, ONNX Runtime, WebGPU/WebLLM, ExecuTorch, vendor NPUs). Use for privacy/offline/latency; limited quality; update/distribution concerns; evaluate on target hardware.

## 10. Embeddings/rerankers/vision serving
Different profiles: embeddings (batch, compute-bound, huge throughput; Text Embeddings Inference/Infinity/vLLM embeddings), rerankers (pairwise scoring latency), vision/audio (preprocessing heavy). Co-locate or separate pools; dynamic batching; GPU vs CPU decisions.

## 11. Checklist before go-live
- [ ] Capacity plan with measured goodput at SLO and headroom
- [ ] Autoscaling tested incl. cold-start time
- [ ] Health checks, overload behavior, fallbacks tested
- [ ] AuthN/Z, network isolation, secrets, audit logging
- [ ] Observability: metrics/traces/dashboards/alerts; cost tagging
- [ ] Versioned artifacts, rollback procedure
- [ ] Load/chaos tests; quality parity evals on the served (quantized) model
- [ ] Runbooks and on-call; DR plan

## Self-check
1. Compute break-even between API and self-hosting for a given load. 2. Size 70B-4bit for 40 concurrent 8k-context users. 3. What is cache-aware routing and why does it matter? 4. Design overload handling for an inference service. 5. Why must you re-run quality evals after quantization/engine changes? 6. How does multi-LoRA serving help multi-tenant customization?

Next → [[07-Guardrails-and-Security-Hardening]]
