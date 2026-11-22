---
tags: [phase-2, deep-learning, architectures]
---
# 07 — Deep Learning Architectures (pre-Transformer + ideas that carry over)

> Transformers (Phase 3) now dominate, but you need the history and the **ideas**: inductive bias, weight sharing, embeddings, residuals, transfer learning, self-supervision, generative modeling. They show up in vision-language models, diffusion, and LLM internals.

## 1. Convolutional Neural Networks (CNNs)
- **Idea:** exploit **locality** and **translation equivariance** with small learned filters slid across the image (weight sharing → far fewer params than dense).
- **Conv layer:** `out[c_out,i,j] = Σ_{c_in,u,v} W[c_out,c_in,u,v]·x[c_in,i+u,j+v] + b`. Hyperparams: kernel size, stride, padding, dilation, channels, groups (depthwise-separable).
- **Receptive field** grows with depth. **Pooling** (max/avg) downsamples. Feature maps: edges → textures → parts → objects.
- Output size: `⌊(n + 2p − k)/s⌋ + 1`.
- **Milestones:** LeNet → AlexNet (2012; GPUs + ReLU + dropout) → VGG → GoogLeNet/Inception → **ResNet** (residual connections) → DenseNet → EfficientNet → ConvNeXt. **Vision Transformers (ViT)** treat image patches as tokens; large-scale pretraining beats CNNs; CLIP aligns images & text.
- **Tasks:** classification, detection (YOLO, Faster R-CNN, DETR), segmentation (U-Net, Mask R-CNN, SAM), pose, OCR/document AI.
- **Data augmentation:** flips, crops, color jitter, mixup/cutmix, RandAugment.
- 1D convs for audio/time series; 3D for video.

## 2. Sequence models: RNNs
- **Vanilla RNN:** `hₜ = tanh(W_h hₜ₋₁ + W_x xₜ + b)`; process sequences left-to-right with shared weights; **BPTT** (backprop through time); vanishing/exploding gradients over long spans.
- **LSTM** (cell state + input/forget/output gates) and **GRU** (simplified) mitigate vanishing gradients; gates = learned soft switches.
- **Bidirectional** RNNs; **stacked** RNNs.
- **Seq2seq (encoder–decoder):** encode source to a vector, decode target. Bottleneck → **attention** (Bahdanau 2014): decoder looks back at all encoder states with learned weights — the seed of the Transformer ("Attention Is All You Need", 2017).
- RNN limits: sequential computation (can't parallelize over time), memory fading. Modern revival: **state-space models (Mamba/S4)**, linear attention, RWKV, hybrid models — Phase 9.
- **Teacher forcing**, exposure bias, beam search, greedy decoding.

## 3. Embeddings (the bridge to LLMs)
- Map discrete items (words, tokens, users, products) to dense vectors learned so that "similar" items are close.
- **`nn.Embedding(V, D)`** is just a lookup table (= one-hot × matrix).
- **word2vec** (skip-gram/CBOW with negative sampling): "king − man + woman ≈ queen"; **GloVe**; **fastText** (subwords). Limitation: one static vector per word (no context) → **contextual embeddings** (ELMo → BERT → LLM hidden states).
- **Sentence/document embeddings** (SBERT, E5, BGE, OpenAI/Voyage/Cohere embedding APIs) power semantic search and RAG (Phase 5). Trained **contrastively** (below).
- Evaluate embeddings: similarity benchmarks, MTEB, downstream retrieval.

## 4. Autoencoders & latent-variable models
- **Autoencoder:** encoder `z=E(x)`, decoder `x̂=D(z)`, minimize reconstruction. Bottleneck ⇒ compression/features. Variants: denoising, sparse, contractive. Uses: anomaly detection, pretraining, dimensionality reduction. **Sparse autoencoders** are central to LLM **mechanistic interpretability** (Phase 9).
- **VAE (variational autoencoder):** encoder outputs a distribution `q(z|x)=N(μ,σ²)`; loss = reconstruction + `KL(q(z|x)‖p(z))` (ELBO); **reparameterization trick** `z = μ + σ⊙ε`. Smooth latent space; blurry samples.
- **VQ-VAE / VQ-GAN:** discrete latents (image tokens) → used in image/audio generation.

## 5. Generative models overview
| Family | Idea | Notes |
|---|---|---|
| **Autoregressive** | `P(x)=ΠP(xₜ|x<ₜ)` | **LLMs**, PixelCNN, image/audio AR models; exact likelihood; sequential sampling |
| VAE | latent variable + ELBO | stable, blurry |
| **GAN** | generator vs discriminator minimax | sharp images; unstable; mode collapse |
| **Diffusion** | learn to denoise: forward adds noise over T steps; model predicts noise/score; sampling = iterative denoising | state of the art for images/video/audio (Stable Diffusion, DALL·E, Sora-type video); **latent diffusion** (diffuse in VAE latent space), classifier-free guidance, flow matching / rectified flow, **DiT** (diffusion transformers) |
| Normalizing flows | invertible transforms | exact density |
| Energy-based | | |
Text diffusion/discrete diffusion LMs are an active research area (Phase 9).

## 6. Transfer learning & fine-tuning (the practical superpower)
- **Pretrain** on large generic data → **adapt** to your task with little data.
- **Strategies:** feature extraction (freeze backbone, train a head); **fine-tune** all or last layers with a small LR (discriminative LRs); parameter-efficient fine-tuning (LoRA — Phase 7); **linear probing** to test representation quality.
- Why it works: early features are generic. Risks: **catastrophic forgetting**, domain mismatch, negative transfer.
- **Hugging Face Hub** + `timm` for pretrained models.

## 7. Self-supervised & contrastive learning
- **Self-supervised**: create labels from the data: masked prediction (BERT, MAE), next-token (GPT), rotation prediction, etc. Scales to unlabeled web-scale data.
- **Contrastive learning:** pull positive pairs together, push negatives apart. **InfoNCE loss**: `−log( exp(sim(q,k⁺)/τ) / Σⱼ exp(sim(q,kⱼ)/τ) )`. Examples: SimCLR, MoCo, **CLIP** (image–text), **sentence embedding models**, dense retrievers (DPR). Temperature `τ`, **hard negatives**, in-batch negatives matter.
- **Knowledge distillation:** train a small student to match a large teacher's soft outputs (`KL` on logits, temperature). Used for compact models (Phase 9).

## 8. Multi-task, multimodal, metric learning
- Shared encoders, multi-head outputs. Multimodal: CLIP, vision-language models (LLaVA-style: ViT encoder → projector → LLM), audio (Whisper), speech, video.
- **Metric learning:** triplet loss, ArcFace; **two-tower** models for retrieval.

## 9. Graph neural networks (brief)
Message passing over graph structure (GCN, GAT, GraphSAGE); used in molecules, recommendation, fraud rings; related to knowledge graphs/GraphRAG.

## 10. Reinforcement learning primer (needed for RLHF/RLVR in Phase 3)
- **MDP:** states, actions, rewards, policy `π(a|s)`, return `Σγᵗrₜ`, value `V(s)`, action-value `Q(s,a)`, Bellman equation.
- Methods: tabular Q-learning, DQN, **policy gradients (REINFORCE)** `∇J = E[∇log π(a|s)·A]`, actor–critic, **PPO** (clipped surrogate objective; stable), GRPO (group-relative, no value net), exploration vs exploitation, reward hacking.
- LLM link: policy = model; action = token; reward = preference model or verifiable checker. Covered in Phase 3 & 9.

## 11. Architectural ideas to carry forward
Weight sharing & inductive bias · residual connections · normalization · gating · attention as soft lookup · embeddings as learned representations · pretrain→fine-tune · contrastive & generative self-supervision · bottlenecks · skip connections (U-Net).

Next → [[08-Practical-ML-Engineering]]
