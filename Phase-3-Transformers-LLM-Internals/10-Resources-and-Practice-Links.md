---
tags: [phase-3, resources]
---
# 10 — Resources & Practice Links (Phase 3)

> ⭐ = top pick. Pages/versions change — I haven't opened every link, so search the title if one moves. For anything "latest" (models, benchmarks), use the **live trackers** section.

## Start here (in this order)
1. ⭐ 3Blue1Brown *Deep Learning* chapters on **transformers/attention** (visual intuition) https://www.3blue1brown.com/topics/neural-networks
2. ⭐ Jay Alammar — *The Illustrated Transformer* https://jalammar.github.io/illustrated-transformer/ · *Illustrated GPT-2* https://jalammar.github.io/illustrated-gpt2/
3. ⭐ Karpathy — *Let's build GPT: from scratch, in code* + *Let's build the GPT Tokenizer* + *Let's reproduce GPT-2* (https://karpathy.ai/zero-to-hero.html)
4. ⭐ *The Annotated Transformer* (Harvard NLP) https://nlp.seas.harvard.edu/annotated-transformer/
5. The paper: *Attention Is All You Need* https://arxiv.org/abs/1706.03762
6. ⭐ Sebastian Raschka — *Build a Large Language Model (From Scratch)* (book) + code https://github.com/rasbt/LLMs-from-scratch

## Courses
- ⭐ **Stanford CS336 — Language Modeling from Scratch** (tokenizer → model → systems → scaling → data → alignment; assignments public) https://stanford-cs336.github.io/
- Stanford **CS224N** (NLP with DL) https://web.stanford.edu/class/cs224n/
- Hugging Face **LLM Course** https://huggingface.co/learn/llm-course · HF **smol course** (post-training) — see https://huggingface.co/learn
- Karpathy — *Intro to Large Language Models* (1-hr talk, YouTube) & *Deep Dive into LLMs like ChatGPT* (3.5 hr, YouTube)
- DeepLearning.AI short courses (e.g., "How Transformer LLMs Work", post-training, quantization) https://www.deeplearning.ai/short-courses/
- CMU *Advanced NLP / LLMs* lectures (YouTube/website), Princeton COS597 *Understanding LLMs*, Berkeley *LLM Agents* (Phase 6)
- Umar Jamil (YouTube): Transformer, LLaMA, Mistral, DPO, RLHF, FlashAttention from scratch

## Core papers (read in this order; 1–2 per week)
| Topic | Paper | Link |
|---|---|---|
| Transformer | Attention Is All You Need | https://arxiv.org/abs/1706.03762 |
| GPT | GPT-2 (Radford 2019) · GPT-3 *Few-Shot Learners* | https://arxiv.org/abs/2005.14165 |
| Scaling | Kaplan et al. *Scaling Laws for Neural LMs* | https://arxiv.org/abs/2001.08361 |
| | Hoffmann et al. *Chinchilla* | https://arxiv.org/abs/2203.15556 |
| Tokenization | Sennrich et al. BPE · SentencePiece | https://arxiv.org/abs/1508.07909 · https://arxiv.org/abs/1808.06226 |
| Positional | RoFormer (RoPE) | https://arxiv.org/abs/2104.09864 |
| Attention variants | MQA (Shazeer) · GQA | https://arxiv.org/abs/1911.02150 · https://arxiv.org/abs/2305.13245 |
| Efficiency | FlashAttention · FlashAttention-2 | https://arxiv.org/abs/2205.14135 · https://arxiv.org/abs/2307.08691 |
| Open models | LLaMA (1/2) · **The Llama 3 Herd of Models** | https://arxiv.org/abs/2307.09288 · https://arxiv.org/abs/2407.21783 |
| MoE | Mixtral of Experts · **DeepSeek-V3 Technical Report** (MLA, MoE, FP8, MTP) | https://arxiv.org/abs/2401.04088 · https://arxiv.org/abs/2412.19437 |
| Alignment | InstructGPT (RLHF) · Constitutional AI · DPO | https://arxiv.org/abs/2203.02155 · https://arxiv.org/abs/2212.08073 · https://arxiv.org/abs/2305.18290 |
| Reasoning RL | DeepSeekMath (GRPO) · **DeepSeek-R1** | https://arxiv.org/abs/2402.03300 · https://arxiv.org/abs/2501.12948 |
| CoT | Chain-of-Thought Prompting (Wei) | https://arxiv.org/abs/2201.11903 |
| Serving | vLLM / PagedAttention · Speculative Decoding | https://arxiv.org/abs/2309.06180 · https://arxiv.org/abs/2211.17192 |
| Context | Lost in the Middle | https://arxiv.org/abs/2307.03172 |
| Data | FineWeb (datasets paper) · Textbooks Are All You Need (Phi) | search arXiv by title |
| Interpretability | A Mathematical Framework for Transformer Circuits | https://transformer-circuits.pub/2021/framework/index.html |
| Emergence | Are Emergent Abilities a Mirage? | https://arxiv.org/abs/2304.15004 |
(arXiv IDs given from memory; if one doesn't resolve, search the title on arxiv.org.)

## Architecture reference (keep up to date)
- ⭐ Raschka — **LLM Architecture Gallery** (side-by-side open-weight model architectures; attention types, MoE, hybrids) https://sebastianraschka.com/llm-architecture-gallery/
- ⭐ Raschka — *The Big LLM Architecture Comparison* https://magazine.sebastianraschka.com/p/the-big-llm-architecture-comparison · *Beyond Standard LLMs* https://magazine.sebastianraschka.com/p/beyond-standard-llms · Ahead of AI newsletter https://magazine.sebastianraschka.com/
- bbycroft *LLM Visualization* (3D walkthrough of a GPT) https://bbycroft.net/llm
- Lilian Weng's blog (attention, RL, agents, hallucination) https://lilianweng.github.io/
- Transformer Circuits thread https://transformer-circuits.pub/

## Back-of-envelope math
- ⭐ EleutherAI *Transformer Math 101* https://blog.eleuther.ai/transformer-math/
- Kipply — *Transformer Inference Arithmetic* https://kipply.github.io/blog/2022/transformer_inference_arithmetic/
- Horace He — *Making Deep Learning Go Brrrr* https://horace.io/brrr_intro.html
- *How to Scale Your Model* (Google DeepMind/JAX scaling book, TPUs/GPUs systems view) https://jax-ml.github.io/scaling-book/
- Hugging Face *Ultra-Scale Playbook* (training at scale) — search title on huggingface.co/spaces
- Hugging Face *Smol Training Playbook* (practical end-to-end LLM training) — search title on huggingface.co/spaces

## Tokenization tools
- tiktoken https://github.com/openai/tiktoken · Tiktokenizer (visual) https://tiktokenizer.vercel.app/ · HF tokenizers docs https://huggingface.co/docs/tokenizers · minbpe https://github.com/karpathy/minbpe · SentencePiece https://github.com/google/sentencepiece

## Post-training tools & docs
- ⭐ Hugging Face **TRL** (SFT, DPO, GRPO, reward modeling) https://huggingface.co/docs/trl · PEFT https://huggingface.co/docs/peft
- **verl** (RL for LLMs) https://github.com/volcengine/verl · **OpenRLHF** https://github.com/OpenRLHF/OpenRLHF · Axolotl https://github.com/axolotl-ai-cloud/axolotl · Unsloth https://github.com/unslothai/unsloth · torchtune https://github.com/pytorch/torchtune · LLaMA-Factory https://github.com/hiyouga/LLaMA-Factory
- Nathan Lambert — *RLHF Book* https://rlhfbook.com/ · Interconnects newsletter https://www.interconnects.ai/
- Chip Huyen — *RLHF: Reinforcement Learning from Human Feedback* https://huyenchip.com/2023/05/02/rlhf.html

## Inference & serving
- ⭐ vLLM docs https://docs.vllm.ai/ · SGLang https://docs.sglang.ai/ · llama.cpp https://github.com/ggml-org/llama.cpp · Ollama https://ollama.com/ · MLX (Apple) https://github.com/ml-explore/mlx · TensorRT-LLM https://github.com/NVIDIA/TensorRT-LLM
- Quantization: bitsandbytes, AutoAWQ, GPTQ, GGUF (llama.cpp) docs · HF quantization guide https://huggingface.co/docs/transformers/main/en/quantization/overview
- FlashAttention repo https://github.com/Dao-AILab/flash-attention
- Outlines (structured generation) https://github.com/dottxt-ai/outlines · XGrammar https://github.com/mlc-ai/xgrammar
- Karpathy **llm.c** (GPT-2 training in C/CUDA) https://github.com/karpathy/llm.c

## Live trackers (for the fast-moving stuff)
- ⭐ **LMArena** (human-preference leaderboard) https://lmarena.ai/
- ⭐ **Artificial Analysis** (independent quality/speed/price) https://artificialanalysis.ai/
- **Epoch AI** (models, compute, trends) https://epoch.ai/
- Hugging Face Open LLM / model hub trending https://huggingface.co/models · HF Papers https://huggingface.co/papers
- SWE-bench https://www.swebench.com/ · LiveCodeBench https://livecodebench.github.io/ · Scale SEAL https://scale.com/leaderboard · vals.ai
- Newsletters/blogs: Latent Space https://www.latent.space/ · Interconnects · Ahead of AI · Simon Willison https://simonwillison.net/ · Import AI · The Batch · Sebastian Ruder · Hamel Husain
- Provider docs for model cards/system cards: Anthropic https://docs.claude.com/ · OpenAI https://platform.openai.com/docs · Google https://ai.google.dev/ · Meta Llama https://www.llama.com/ · DeepSeek, Qwen, Mistral sites

## Practice
- ⭐ **Deep-ML** (attention, softmax, etc.) https://www.deep-ml.com/ · **TorchLeet** https://github.com/Exorust/TorchLeet · **TorchCode** https://huggingface.co/spaces/duoan/TorchCode
- Sasha Rush **Tensor Puzzles** & **GPU Puzzles** https://github.com/srush/Tensor-Puzzles · **LLM Training Puzzles** https://github.com/srush/LLM-Training-Puzzles
- **LeetGPU** https://leetgpu.com/ (CUDA, advanced)
- Karpathy exercises at the end of each video; CS336 assignments (tokenizer, transformer, FlashAttention-Triton, scaling, data, alignment)
- Raschka repo exercises (`LLMs-from-scratch` chapter exercises)
- Reproduce **nanoGPT / modded-nanoGPT speedrun** ideas https://github.com/KellerJordan/modded-nanogpt (advanced optimization tricks)

## GPU / compute for this phase
Colab / Kaggle (free), Lightning Studios, Modal, RunPod, Lambda, Vast.ai; local: Apple Silicon (MLX) or RTX GPU. A 10–50M-param GPT trains on a free Colab/Kaggle GPU; GPT-2 124M reproduction needs a rented A100/H100 for a few hours (~$10–40).

## If you only have 2 weeks
Day 1–2: Illustrated Transformer + 3B1B. Day 3–6: Karpathy "Let's build GPT" typed along. Day 7: tokenizer video. Day 8–9: Transformer Math 101 + KV cache arithmetic. Day 10–12: Karpathy "Deep Dive into LLMs" + TRL SFT/DPO toy. Day 13–14: Raschka's architecture gallery + Q&A drill.

Next → [[11-Projects-and-Checkpoint]]
