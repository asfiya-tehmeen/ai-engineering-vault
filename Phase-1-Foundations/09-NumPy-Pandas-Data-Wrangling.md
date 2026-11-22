---
tags: [phase-1, numpy, pandas, data]
---
# 09 — NumPy, Pandas & Data Wrangling

> Embeddings are NumPy arrays; datasets for evals are DataFrames; customer data is messy CSV/Excel. This note is your bridge to Phase 2.

## 1. NumPy: n-dimensional arrays
```python
import numpy as np
a = np.array([[1,2,3],[4,5,6]])        # shape (2,3), dtype int64
a.shape, a.ndim, a.dtype
np.zeros((3,3)); np.ones(5); np.arange(0,10,2); np.linspace(0,1,5)
rng = np.random.default_rng(42); rng.normal(size=(1000, 384))
```
- **Vectorization**: operate on whole arrays in C; avoid Python loops. 100× faster.
- **Broadcasting**: shapes align from the right; size-1 dims stretch. `(3,1) + (1,4) → (3,4)`. A top source of shape bugs.
- **Indexing**: slicing `a[:, 1]`, boolean masks `a[a>3]`, fancy indexing `a[[0,2]]`. Views vs copies!
- **Reductions**: `sum/mean/std/max/argmax(axis=…)`. `axis=0` collapses rows, `axis=1` collapses columns.
- **Linear algebra**: `a @ b` (matmul), `.T`, `np.linalg.norm`, `inv`, `svd`, `eig`.
- **Dtypes**: float32 vs float64 vs float16/bfloat16 — memory vs precision.

### Core AI operation: cosine similarity
```python
def cosine_sim(q, M):                    # q: (d,), M: (n,d)
    q = q / np.linalg.norm(q)
    M = M / np.linalg.norm(M, axis=1, keepdims=True)
    return M @ q                         # (n,) similarity scores
top_k = np.argsort(-scores)[:5]
```
This *is* the heart of semantic search (Phase 5). Dot product, cosine, Euclidean — know when each applies (normalized vectors → dot = cosine).

## 2. Pandas: tabular data
```python
import pandas as pd
df = pd.read_csv("orders.csv", parse_dates=["created_at"])
df.head(); df.info(); df.describe(); df.dtypes; df.shape
df["total"].mean(); df["status"].value_counts()
df[df.total > 100]; df.query("status == 'paid' and total > 100")
df.loc[row_label, "col"]; df.iloc[0:5, 1:3]
df["month"] = df.created_at.dt.to_period("M")
df.groupby("user_id").agg(n=("id","count"), revenue=("total","sum")).reset_index()
pd.merge(orders, users, left_on="user_id", right_on="id", how="left")
df.pivot_table(index="month", columns="status", values="total", aggfunc="sum")
df.sort_values("total", ascending=False).head(10)
df.to_parquet("out.parquet"); df.to_json("out.jsonl", orient="records", lines=True)
```
### Cleaning checklist (80% of real work)
- Missing values: `isna().sum()`, `fillna`, `dropna` — decide *why* it's missing.
- Duplicates: `duplicated()`, `drop_duplicates(subset=…)`.
- Types: strings → numbers/dates (`pd.to_numeric(errors="coerce")`), categories.
- Text normalization: strip, lowercase, unicode normalize, regex extract.
- Outliers, inconsistent units/currencies, timezone mismatches, encoding issues.
- **Validate** after cleaning (assertions/Pandera/Great Expectations).

### Performance & modern alternatives
- Avoid `apply(axis=1)` row loops; vectorize. Use `category` dtype, chunked reading (`chunksize`), Parquet.
- **Polars** (fast, Rust, lazy), **DuckDB** (SQL on files/DataFrames, excellent), **PyArrow**. Pandas 2.x supports Arrow-backed dtypes.
- Excel reality: `openpyxl`, merged cells, multiple sheets — FDEs inherit these.

## 3. Visualization
`matplotlib` (base), `seaborn`, `plotly`. Learn to plot distributions, scatter, time series; always label axes. In notebooks use **Jupyter/JupyterLab/VS Code notebooks/Colab**.

## 4. Notebooks vs scripts
Notebooks for exploration; move stable logic into tested modules. Restart-and-run-all before sharing. Keep secrets out of notebooks.

## 5. Statistics you need now
Mean/median/mode, variance/std, percentiles (p50/p95/p99 — latency!), distributions, correlation ≠ causation, sampling, confidence intervals, A/B tests (preview for Phase 7 evals). Resource: *Think Stats* (free).

Next → [[10-Exercises]]
