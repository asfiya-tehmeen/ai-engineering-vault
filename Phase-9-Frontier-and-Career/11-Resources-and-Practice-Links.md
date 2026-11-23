---
tags: [phase-9, resources]
---
# 11 — Resources & Practice Links (Phase 9)

> ⭐ = top pick. Links are from memory or surfaced in search and **not all opened**; search titles if one moves. arXiv IDs are from memory. For the fastest-moving topics, prefer **primary sources** (papers, lab blogs, docs) over summaries.

## Reasoning, RL & test-time compute
- ⭐ Sutton & Barto — *Reinforcement Learning: An Introduction* (free) http://incompleteideas.net/book/the-book.html · OpenAI Spinning Up https://spinningup.openai.com/ · Berkeley CS285 Deep RL http://rail.eecs.berkeley.edu/deeprlcourse/ · Hugging Face Deep RL course https://huggingface.co/learn/deep-rl-course
- ⭐ Nathan Lambert — *RLHF Book* https://rlhfbook.com/ · Interconnects https://www.interconnects.ai/ (see also the 2025 survey *Reinforcement Learning from Human Feedback*, arXiv 2504.12501)
- Papers: CoT 2201.11903 · Self-Consistency 2203.11171 · Tree of Thoughts 2305.10601 · *Let's Verify Step by Step* 2305.20050 · *Scaling LLM Test-Time Compute Optimally* 2408.03314 · *s1: Simple test-time scaling* 2501.19393 · DeepSeekMath/GRPO 2402.03300 · DeepSeek-R1 2501.12948
- Surfaced 2026 sources (commentary/preprints; verify): *The "Thinking" Revolution: Inference-Time Scaling and RLVR* (Substack) https://interestingengineering.substack.com/p/the-thinking-revolution-how-inference · *How Inference Compute Shapes Frontier LLM Evaluation* (arXiv 2606.17930) · *Predictable GRPO: A Closed-Form Model of Training Dynamics* (arXiv 2606.30789) · *Apriel-Reasoner: RL Post-Training* (arXiv 2604.02007)
- Tools: TRL GRPO https://huggingface.co/docs/trl · verl https://github.com/volcengine/verl · OpenRLHF https://github.com/OpenRLHF/OpenRLHF · Unsloth RL notebooks https://docs.unsloth.ai/
- Long-horizon capability measurement: METR https://metr.org/ (task time-horizon work) · Epoch AI https://epoch.ai/

## Efficiency, quantization, distillation, architectures
- ⭐ Horace He — *Making Deep Learning Go Brrrr* https://horace.io/brrr_intro.html · ⭐ *How to Scale Your Model* (JAX/TPU systems book) https://jax-ml.github.io/scaling-book/ · Kipply *Transformer Inference Arithmetic* https://kipply.github.io/blog/2022/transformer_inference_arithmetic/ · EleutherAI *Transformer Math 101* https://blog.eleuther.ai/transformer-math/
- Quantization papers: LLM.int8() 2208.07339 · GPTQ 2210.17323 · AWQ 2306.00978 · SmoothQuant 2211.10438 · FP8 formats 2209.05433 · QLoRA 2305.14314 · **NVFP4 Quantization-Aware Distillation** arXiv 2601.20088 (NVIDIA; surfaced in search) https://arxiv.org/abs/2601.20088 · NVIDIA Nemotron QAD page https://research.nvidia.com/labs/nemotron/nemotron-qad · Red Hat *Accelerating LLMs with NVFP4 quantization* https://developers.redhat.com/articles/2026/02/04/accelerating-large-language-models-nvfp4-quantization (vendor-reported results)
- Tools: llm-compressor (vLLM project) · AutoAWQ · GPTQModel · bitsandbytes · llama.cpp/GGUF · TensorRT Model Optimizer · torchao https://github.com/pytorch/ao
- Distillation: Hinton et al. 1503.02531 · MiniLLM 2306.08543 · GKD (on-policy) 2306.13649 · Sebastian Raschka distillation write-ups
- MoE: Switch Transformer 2101.03961 · Mixtral 2401.04088 · DeepSeekMoE 2401.06066 · DeepSeek-V3 2412.19437 · Raschka *LLM Architecture Gallery* https://sebastianraschka.com/llm-architecture-gallery/ · *The Big LLM Architecture Comparison* https://magazine.sebastianraschka.com/p/the-big-llm-architecture-comparison · *Beyond Standard LLMs* https://magazine.sebastianraschka.com/p/beyond-standard-llms
- Alternatives to softmax attention: Mamba 2312.00752 · Mamba-2 2405.21060 · Gated DeltaNet 2412.06464 · RWKV https://www.rwkv.com/ · TTT layers 2407.04620 · Titans 2501.00663 · LLaDA (diffusion LM) 2502.09992
- Kernels/GPU programming: Triton tutorials https://triton-lang.org/ · CUDA (NVIDIA docs; *Programming Massively Parallel Processors*) · GPU MODE community & lectures https://www.youtube.com/@GPUMODE · LeetGPU https://leetgpu.com/ · Sasha Rush GPU Puzzles https://github.com/srush/GPU-Puzzles · FlashAttention repo https://github.com/Dao-AILab/flash-attention · Karpathy llm.c https://github.com/karpathy/llm.c

## Alignment & safety
- ⭐ AI Alignment Forum https://www.alignmentforum.org/ · LessWrong https://www.lesswrong.com/ · BlueDot Impact courses (AI safety fundamentals) https://bluedot.org/ · 80,000 Hours AI safety career guide https://80000hours.org/
- Papers: Concrete Problems in AI Safety 1606.06565 · Constitutional AI 2212.08073 · Debate 1805.00899 · Weak-to-Strong 2312.09390 · Sleeper Agents 2401.05566 · Alignment Faking 2412.14093 · Emergent Misalignment 2502.17424 · AI Control 2312.06942 · Sycophancy 2310.13548 · *An Approach to Technical AGI Safety and Security* 2504.01849 (surfaced in search)
- Lab frameworks & system cards: Anthropic, OpenAI, Google DeepMind safety/preparedness frameworks and model cards (read the originals)
- Evals/red-team: UK AISI Inspect https://inspect.aisi.org.uk/ · HarmBench https://www.harmbench.org/ · Garak · PyRIT · AgentDojo
- Programs (availability/format change; verify): MATS https://www.matsprogram.org/ · lab fellowships/residencies · SPAR/ERA-style mentorship programs

## Interpretability
- ⭐ Transformer Circuits thread https://transformer-circuits.pub/ — *A Mathematical Framework*, *Toy Models of Superposition* (2209.10652), *Towards/Scaling Monosemanticity*, **attribution-graph methods** https://transformer-circuits.pub/2025/attribution-graphs/methods.html (surfaced in search) and *On the Biology of a Large Language Model*
- ⭐ Neel Nanda — mechanistic interpretability guides & "getting started" https://www.neelnanda.io/mechanistic-interpretability · **ARENA** curriculum https://www.arena.education/ · TransformerLens https://github.com/TransformerLensOrg/TransformerLens · nnsight https://nnsight.net/ · SAELens https://github.com/jbloomAus/SAELens · Neuronpedia https://www.neuronpedia.org/
- Papers: SAEs 2309.08600 · ROME 2202.05262 · Representation Engineering 2310.01405 · Refusal direction 2406.11717 · Logit lens (nostalgebraist) · Tuned lens 2303.08112
- 2026 preprints surfaced in search (verify): *Actionable mechanistic interpretability* survey (arXiv 2601.22594) · *LLMs Can Annotate Attribution Graphs* (arXiv 2608.02632) · work arguing small neuron-level circuits can control benchmark behaviors (arXiv 2601.14004) · *Automatically Identifying Local and Global Circuits with Linear Computation Graphs* (arXiv 2405.13868)

## Data, continual learning, world models, embodied AI
- Data: *Scaling Data-Constrained LMs* 2305.16264 · *Textbooks Are All You Need* 2306.11644 · *The Curse of Recursion / model collapse* 2305.17493 · FineWeb/FineWeb-Edu (Hugging Face) · Dolma · OLMo · Nemotron-CC · surfaced 2026 data-constrained scaling commentary: https://bayesiansapien.github.io/cere-bro/llms-foundation-models/2026-05-09-prescriptive-scaling-laws-data-constrained
- Continual learning: survey 2404.16789 · EWC (1612.00796) · model editing (ROME/MEMIT) · MemGPT 2310.08560 · Letta/Mem0/Zep docs
- World models/multimodal/embodied: Genie 2402.15391 · V-JEPA / JEPA (LeCun) · RT-2 2307.15818 · π0 2410.24164 · LeRobot (Hugging Face) https://github.com/huggingface/lerobot · MuJoCo/Isaac Sim docs
- Forecast/commentary (opinion; weigh accordingly): the mid-2026 predictions review https://www.uxtigers.com/post/2026-predictions-halfway · NextBigFuture world-models post · Medium scaling/reasoning overviews · Epoch AI trend dashboards https://epoch.ai/data

## Reading papers & staying current
- Feeds: arXiv cs.CL/cs.LG/cs.AI/cs.CR https://arxiv.org/ · Hugging Face Daily Papers https://huggingface.co/papers · alphaXiv https://www.alphaxiv.org/ · Semantic Scholar https://www.semanticscholar.org/ · Connected Papers https://www.connectedpapers.com/
- Newsletters/podcasts: Latent Space https://www.latent.space/ · Interconnects · Ahead of AI https://magazine.sebastianraschka.com/ · Import AI https://importai.substack.com/ · The Batch https://www.deeplearning.ai/the-batch/ · Simon Willison https://simonwillison.net/ · AI Engineer talks https://www.ai.engineer/ · ML Street Talk · Dwarkesh Podcast
- How-to-read guides: S. Keshav *How to Read a Paper* (3-pass method; search title) · Andrew Ng's advice on reading papers · Jia-Bin Huang's research habits videos
- Courses to deepen theory: Stanford CS336, CS229, CS224N, CS231n · MIT 6.S191 · DeepLearning.AI specializations · fast.ai Part 2 · Karpathy Zero to Hero
- Research taste/writing: *Principles of Effective Research* (Michael Nielsen) · Jacob Steinhardt's research advice (search) · Neel Nanda's *How to become a mechanistic interpretability researcher* post

## Career & job search
- ✱ Surfaced career sources (blogs/vendor content; treat claims as directional): Interview Query *Demand for AI Engineers in 2026* https://www.interviewquery.com/p/ai-engineers-demand · TripleTen *AI skills to learn in 2026* https://tripleten.com/blog/posts/ai-skills · Boston University *Entry-level AI jobs: how early-career engineers can break in* https://www.bu.edu/online/2026/10/12/entry-level-ai-jobs-how-early-career-engineers-can-break-in/ · Study.com entry-level AI jobs https://study.com/resources/top-entry-level-ai-jobs.html · Consultadd, Nexus IT, Axial Search, CV-Maker AI-engineer market posts (search titles; compare for consistency)
- Resumes & interviews: Tech Interview Handbook https://www.techinterviewhandbook.org/ (resume, negotiation, behavioral) · *Cracking the Coding Interview* · *The Tech Resume Inside-Out* · levels.fyi https://www.levels.fyi/ (compensation data; verify freshness) · Blind (anonymous insights; low-signal caution)
- Networking/personal brand: *Show Your Work!* (Kleon) · Julia Evans (blogging as learning) https://jvns.ca/ · Swyx *Learn in Public* essay · *So Good They Can't Ignore You* (Newport) · *How to Win Friends and Influence People* (classic) · LinkedIn/X communities for AI engineering
- Career design: *Designing Your Life* · *Staff Engineer* (Larson) · *The Pragmatic Engineer* newsletter https://newsletter.pragmaticengineer.com/ · *High Output Management* (Grove) · *The Manager's Path*
- Open-source & programs: GitHub "good first issues" https://github.com/topics/good-first-issue · Google Summer of Code https://summerofcode.withgoogle.com/ · Outreachy https://www.outreachy.org/ · MLH Fellowship https://fellowship.mlh.io/ · hackathons (lablab.ai, Devpost)
- Interview prep cross-links: [[../Phase-8-Forward-Deployed-Engineering/09-FDE-Interview-Prep]] · Q&A notes in Phases 1–8

## Tools for your learning OS
Obsidian plugins: Spaced Repetition, Dataview, Periodic Notes, Templater, Excalidraw (diagrams) · Zotero/Readwise for papers/highlights · Git for versioning · Raindrop/Pocket queue · W&B/MLflow for experiments · Anki (alternative flashcards)

## If you only have 1 week
Day 1: pick target roles + audit portfolio. Day 2: polish one flagship README with metrics. Day 3: three-pass read one paper + read-log entry. Day 4: a small test-time-compute or quantization experiment with CIs. Day 5: resume + LinkedIn update; 10 outreach messages. Day 6: mock interview + debrief. Day 7: weekly review + plan next week.

Next → [[12-Projects-and-Final-Checkpoint]]
