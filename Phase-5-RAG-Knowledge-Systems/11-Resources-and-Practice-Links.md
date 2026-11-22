---
tags: [phase-5, resources]
---
# 11 — Resources & Practice Links (Phase 5)

> ⭐ = top pick. I haven't opened every link; if one moves, search the title. Embedding/vector-DB rankings change monthly and sources conflict — use live leaderboards **and your own evals**. arXiv IDs below are from memory; search the title if one doesn't resolve.

## Start here
1. ⭐ Lance Martin — *RAG From Scratch* (video series + notebooks) https://github.com/langchain-ai/rag-from-scratch
2. ⭐ Anthropic — *Introducing Contextual Retrieval* (chunk context + BM25 + reranking) https://www.anthropic.com/news/contextual-retrieval
3. ⭐ Jason Liu — *Systematically Improving RAG* & writing https://jxnl.co/writing/ (evals-first RAG, "RAG is more than embeddings")
4. ⭐ Hamel Husain — RAG/eval posts https://hamel.dev/ · Eugene Yan — *Patterns for LLM systems* https://eugeneyan.com/writing/llm-patterns/
5. Pinecone Learning Center (concepts, chunking, hybrid) https://www.pinecone.io/learn/
6. Hugging Face cookbook — *Advanced RAG* recipes https://huggingface.co/learn/cookbook
7. Chip Huyen — *AI Engineering* (book; RAG & evaluation chapters) https://huyenchip.com/

## Papers (read in this order)
| Topic | Paper | Link |
|---|---|---|
| RAG origin | Lewis et al. *RAG for Knowledge-Intensive NLP* | https://arxiv.org/abs/2005.11401 |
| Dense retrieval | DPR (Karpukhin et al.) | https://arxiv.org/abs/2004.04906 |
| Benchmark | BEIR (zero-shot retrieval) | https://arxiv.org/abs/2104.08663 |
| Embedding benchmark | MTEB | https://arxiv.org/abs/2210.07316 |
| Late interaction | ColBERT · ColBERTv2 | https://arxiv.org/abs/2004.12832 · https://arxiv.org/abs/2112.01488 |
| Visual docs | ColPali | https://arxiv.org/abs/2407.01449 |
| Learned sparse | SPLADE | https://arxiv.org/abs/2107.05720 |
| ANN | HNSW (Malkov & Yashunin) | https://arxiv.org/abs/1603.09320 |
| Query expansion | HyDE | https://arxiv.org/abs/2212.10496 |
| Lost in the middle | Liu et al. | https://arxiv.org/abs/2307.03172 |
| Hierarchical | RAPTOR | https://arxiv.org/abs/2401.18059 |
| Corrective/Self | Self-RAG · CRAG | https://arxiv.org/abs/2310.11511 · https://arxiv.org/abs/2401.15884 |
| GraphRAG | *From Local to Global* (Microsoft) | https://arxiv.org/abs/2404.16130 |
| RAG training | RAFT | https://arxiv.org/abs/2403.10131 |
| RAG eval | RAGAS | https://arxiv.org/abs/2309.15217 |
| Surveys | *RAG for LLMs: A Survey* (Gao et al.) | https://arxiv.org/abs/2312.10997 |
| Embeddings | Qwen3 Embedding technical report · Gemini Embedding | https://arxiv.org/abs/2506.05176 · https://arxiv.org/abs/2503.07891 |

## Embedding models & leaderboards
- ⭐ **MTEB leaderboard** (filter by Retrieval, language, size) https://huggingface.co/spaces/mteb/leaderboard — *2026 blog rankings disagree on leaders; verify here and on your data.*
- Open families: Qwen3-Embedding / Reranker https://huggingface.co/Qwen · BGE / BGE-M3 / bge-reranker (FlagEmbedding) https://github.com/FlagOpen/FlagEmbedding · E5 / GTE · NVIDIA NV-Embed/Nemotron embed · Jina https://jina.ai/ · Nomic · mixedbread
- APIs: Voyage AI https://docs.voyageai.com/ · OpenAI embeddings (platform docs) · Google Gemini embeddings (ai.google.dev) · Cohere Embed/Rerank https://docs.cohere.com/
- ⭐ Sentence-Transformers docs (training, evaluation, CrossEncoder) https://sbert.net/
- Matryoshka / quantization guides (Sentence-Transformers & Hugging Face blogs)

## Vector search & databases
- ⭐ pgvector https://github.com/pgvector/pgvector · pgvectorscale https://github.com/timescale/pgvectorscale · ParadeDB (BM25 in Postgres) https://www.paradedb.com/
- Qdrant https://qdrant.tech/documentation/ · Weaviate https://weaviate.io/developers/weaviate · Milvus https://milvus.io/docs · Pinecone https://docs.pinecone.io/ · Chroma https://docs.trychroma.com/ · LanceDB https://lancedb.github.io/lancedb/ · turbopuffer https://turbopuffer.com/docs · Elasticsearch/OpenSearch vector & hybrid docs · Vespa https://docs.vespa.ai/
- Libraries: FAISS https://github.com/facebookresearch/faiss · hnswlib · USearch · ScaNN · Annoy
- Benchmarks (use cautiously; test your own filters): ANN-Benchmarks https://ann-benchmarks.com/ · VectorDBBench https://github.com/zilliztech/VectorDBBench
- Concepts: *Vector DB comparison guides (2026)* — many blogs, conflicting numbers; use decision criteria in [[03-Vector-Search-and-Databases]].

## Parsing, chunking & ingestion tools
- ⭐ Docling https://github.com/docling-project/docling · Unstructured https://docs.unstructured.io/ · MarkItDown https://github.com/microsoft/markitdown · PyMuPDF · pdfplumber · marker https://github.com/VikParuchuri/marker · LlamaParse (LlamaIndex) · cloud OCR/doc AI (Textract, Document AI, Azure Document Intelligence)
- Chunking: LangChain text splitters · LlamaIndex node parsers · Chonkie https://github.com/chonkie-inc/chonkie · "Chunking strategies" posts by Pinecone/Weaviate/Unstructured
- tree-sitter (code parsing) https://tree-sitter.github.io/
- Orchestration: Airflow · Dagster · Prefect · Temporal · Airbyte (connectors) https://docs.airbyte.com/
- PII: Presidio https://microsoft.github.io/presidio/

## Frameworks (learn the raw pipeline first)
- LlamaIndex https://docs.llamaindex.ai/ · LangChain/LangGraph https://python.langchain.com/ · Haystack https://docs.haystack.deepset.ai/ · DSPy https://dspy.ai/ · Pydantic AI · Vercel AI SDK · RAGatouille (ColBERT) https://github.com/AnswerDotAI/RAGatouille
- GraphRAG: Microsoft GraphRAG https://github.com/microsoft/graphrag · LightRAG https://github.com/HKUDS/LightRAG · HippoRAG · Neo4j GraphRAG https://neo4j.com/docs/neo4j-graphrag-python/ · nano-graphrag

## Evaluation tooling
- ⭐ RAGAS https://docs.ragas.io/ · TruLens https://www.trulens.org/ · DeepEval https://docs.confident-ai.com/ · ARES https://github.com/stanford-futuredata/ARES
- Observability/eval platforms: Langfuse https://langfuse.com/docs · LangSmith · Arize Phoenix https://docs.arize.com/phoenix · Braintrust · Promptfoo (RAG evals) https://www.promptfoo.dev/docs/
- IR eval libs: `ranx` https://github.com/AmenRa/ranx · `pytrec_eval` · BEIR toolkit https://github.com/beir-cellar/beir
- LLM-judge reliability: *Can LLMs Be Trusted for Evaluating RAG Systems?* (survey, arXiv 2504.20119) · Hamel's evals writing

## Datasets & benchmarks for practice
- Retrieval: **BEIR** https://github.com/beir-cellar/beir · MS MARCO https://microsoft.github.io/msmarco/ · Natural Questions · TREC RAG track https://trec-rag.github.io/ · MIRACL (multilingual)
- Multi-hop: HotpotQA https://hotpotqa.github.io/ · MuSiQue · 2WikiMultihopQA · MultiHop-RAG · FRAMES (Google)
- Domain: FinanceBench https://github.com/patronus-ai/financebench · CUAD (legal contracts) · SEC EDGAR filings https://www.sec.gov/edgar · PubMedQA / BioASQ · CodeSearchNet / CoIR (code) · DocVQA (visual docs)
- RAG-specific: CRAG benchmark (Meta, KDD Cup) · RAGBench · RGB · LegalBench-RAG
- Corpora to build on: Wikipedia dumps · arXiv abstracts/PDFs · Stack Exchange dumps · your favorite open-source docs (Python, Kubernetes) · government/open-data PDFs

## Courses & talks
- ⭐ DeepLearning.AI: *Retrieval Augmented Generation* (Weaviate/others), *Building and Evaluating Advanced RAG* (LlamaIndex + TruEra), *Knowledge Graphs for RAG* (Neo4j), *Advanced Retrieval for AI with Chroma* https://www.deeplearning.ai/short-courses/
- Full-stack RAG talks from AI Engineer conferences (YouTube) https://www.ai.engineer/
- Stanford CS224N / CS276 (Information Retrieval) lectures · Manning *Introduction to Information Retrieval* (free book) https://nlp.stanford.edu/IR-book/
- Vespa/Elastic/Qdrant/Weaviate blog engineering posts (hybrid search, filtering, quantization)

## Practice challenges
- **Kaggle:** RAG/LLM competitions and notebooks https://www.kaggle.com/ (search "RAG", "retrieval")
- **TREC RAG** & **BEIR** leaderboards (reproduce baselines)
- **MTEB retrieval tasks**: evaluate a small model locally via the `mteb` package https://github.com/embeddings-benchmark/mteb
- **Build vs. benchmark:** reproduce the Contextual Retrieval gains on a public corpus; publish your table.
- Open-source contributions: LlamaIndex/Haystack/Docling/RAGAS issues labeled "good first issue."

## If you only have 1 week
Day 1: RAG From Scratch videos + NumPy search. Day 2: embedding bake-off (3 models) on a tiny golden set. Day 3: pgvector + hybrid + RRF. Day 4: cross-encoder reranker + measure. Day 5: citations + abstention. Day 6: golden-set evals + error analysis. Day 7: write the README with numbers.

Next → [[12-Projects-and-Checkpoint]]
