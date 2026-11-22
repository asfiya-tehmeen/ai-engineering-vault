---
tags: [phase-3, tokenization]
---
# 01 — Tokenization

> LLMs never see text. They see integer IDs. Tokenization is the source of many "weird LLM behaviors" and of your **bill** (you pay per token).

## 1. Why tokens?
- **Characters:** tiny vocab, but very long sequences (attention is O(T²)).
- **Words:** huge vocab, OOV problem ("unseen" words), no sharing across inflections.
- **Subwords:** the compromise — frequent words are 1 token; rare words split into pieces. Vocab typically **32k–256k**.
- Pipeline: `text → (normalize) → pre-tokenize → subword algorithm → token IDs → embedding lookup`.

## 2. Algorithms
### Byte-Pair Encoding (BPE) — GPT family, LLaMA 3+, most modern LLMs
Training: start with base symbols (bytes), repeatedly merge the most frequent adjacent pair into a new token until vocab size reached. Record merges in order.
```python
from collections import Counter
def get_stats(ids): return Counter(zip(ids, ids[1:]))
def merge(ids, pair, new_id):
    out, i = [], 0
    while i < len(ids):
        if i < len(ids)-1 and (ids[i], ids[i+1]) == pair: out.append(new_id); i += 2
        else: out.append(ids[i]); i += 1
    return out

def train_bpe(text, vocab_size):
    ids = list(text.encode("utf-8"))                 # byte-level: base vocab = 256
    merges = {}
    for new_id in range(256, vocab_size):
        stats = get_stats(ids)
        if not stats: break
        pair = max(stats, key=stats.get)
        ids = merge(ids, pair, new_id); merges[pair] = new_id
    return merges
```
Encoding applies merges in learned priority order. **Byte-level BPE** guarantees any string is encodable (no `<unk>`).
- **Pre-tokenization regex** (GPT-2/4 style) splits text into chunks (words, digits, punctuation, whitespace) *before* BPE so merges don't cross boundaries awkwardly.
### Others
- **WordPiece** (BERT): likelihood-based merges; `##` continuation pieces.
- **Unigram LM / SentencePiece** (T5, LLaMA-1/2, Gemma): start big, prune by likelihood; language-agnostic, treats whitespace as `▁`.
- **Byte/char-level models** (ByT5, MegaByte, byte latent transformers): no tokenizer, longer sequences; active research (Phase 9).

## 3. Special tokens & chat templates
- `<|endoftext|>`/`</s>` (EOS), `<s>` (BOS), `<pad>`, role markers like `<|im_start|>`, tool-call markers, reasoning markers, image placeholders.
- **Chat template:** the exact string format turning `[{"role":"system",...},{"role":"user",...}]` into one token stream, e.g. `<|im_start|>user\nHi<|im_end|>\n<|im_start|>assistant\n`. **Using the wrong template with an open model silently degrades quality.** Hugging Face: `tokenizer.apply_chat_template(messages, add_generation_prompt=True)`.
- **Prompt injection relevance:** special-token strings in user input must be escaped/neutralized by the tokenizer/API.

## 4. Practical consequences (memorize these)
| Phenomenon | Cause |
|---|---|
| Can't reliably count letters in "strawberry" | the model sees token(s), not characters |
| Weak at reversing strings, spelling, anagrams | same |
| Arithmetic quirks | numbers split into odd chunks (modern tokenizers often split digits individually or in groups of 3 to help) |
| Non-English text is costlier & sometimes worse | fewer merges learned for that language → more tokens per sentence |
| Code/whitespace/indentation tokens | tokenizers tuned for code merge runs of spaces |
| Leading space matters: `"hello"` ≠ `" hello"` | different tokens |
| "SolidGoldMagikarp" glitch tokens | tokens in vocab but almost never seen in training |
| Same text → different token counts across providers | different tokenizers → **always count with the provider's tokenizer/count endpoint** |
Rules of thumb (English): ~4 chars ≈ 1 token; 100 tokens ≈ 75 words. JSON/XML/code and non-Latin scripts are token-heavy.

## 5. Vocabulary size trade-offs
Bigger vocab → shorter sequences (cheaper attention, more context per window) but bigger embedding/unembedding matrices and sparser training for rare tokens. Many frontier-class open models use ~128k–256k vocab.

## 6. Tools
- **tiktoken** (OpenAI; `cl100k_base`, `o200k_base`), **Hugging Face `tokenizers`** (Rust), **SentencePiece**, Karpathy's **minbpe**.
- Visualize: tiktokenizer.vercel.app, OpenAI tokenizer page.
- Provider token-count APIs (Anthropic `count_tokens`, etc.) — use for budgeting.

```python
import tiktoken
enc = tiktoken.get_encoding("o200k_base")
ids = enc.encode("Hello, world! 12345")
print(ids, [enc.decode([i]) for i in ids], len(ids))
```

## 7. Engineering implications
- **Cost & latency scale with tokens** → compress prompts, avoid verbose JSON keys, cache prefixes.
- **Context window is in tokens**, not characters.
- **Chunking for RAG** should be token-aware (Phase 5).
- **Truncation** can cut mid-token-sequence of structured data → invalid JSON; truncate at semantic boundaries.
- **Fine-tuning** requires the exact same tokenizer + template as the base.
- **Logprobs** are per-token; useful for confidence/classification.

## Self-check
1. Hand-run 3 BPE merges on "aaabdaaabac". 2. Why byte-level? 3. Why is `" the"` a different token from `"the"`? 4. Why does a Hindi paragraph cost more than the English equivalent? 5. What breaks if you use the wrong chat template?

Next → [[02-Transformer-Architecture]]
