---
tags: [phase-2, ml, sklearn]
---
# 03 — Classical ML Algorithms

> Classical ML still wins on **tabular/structured data** (gradient-boosted trees usually beat deep nets there) and is the backbone of many enterprise FDE engagements: churn, fraud, forecasting, scoring. Also: they are the cleanest way to learn ML concepts.

## 1. scikit-learn essentials
```python
from sklearn.model_selection import train_test_split, cross_val_score, StratifiedKFold, GridSearchCV
from sklearn.pipeline import Pipeline, make_pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report

pre = ColumnTransformer([
    ("num", make_pipeline(SimpleImputer(strategy="median"), StandardScaler()), num_cols),
    ("cat", make_pipeline(SimpleImputer(strategy="most_frequent"), OneHotEncoder(handle_unknown="ignore")), cat_cols),
])
clf = Pipeline([("pre", pre), ("model", LogisticRegression(max_iter=1000))])
Xtr, Xte, ytr, yte = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)
scores = cross_val_score(clf, Xtr, ytr, cv=StratifiedKFold(5, shuffle=True, random_state=0), scoring="f1")
clf.fit(Xtr, ytr); print(classification_report(yte, clf.predict(Xte)))
```
**Estimator API:** `.fit(X, y)`, `.predict(X)`, `.predict_proba(X)`, `.transform(X)`, `.score`. Always wrap preprocessing in a `Pipeline` → **prevents leakage** and makes deployment a single object.

## 2. Linear regression
- Model: `ŷ = wᵀx + b`. Loss: MSE. Solve via normal equation or GD.
- **Assumptions** (for inference): linearity, independence, homoscedasticity, normal errors, no multicollinearity.
- **Ridge** (L2), **Lasso** (L1), **ElasticNet**; regularization strength `α`/`λ` via CV.
- **Polynomial features** + regularization = flexible curves. Beware extrapolation.
- Diagnostics: residual plots, R², adjusted R², VIF for multicollinearity.

## 3. Logistic regression (classification workhorse)
- `p(y=1|x) = σ(wᵀx + b)`, `σ(z)=1/(1+e⁻ᶻ)`. Loss: binary cross-entropy (convex!). Decision boundary: `wᵀx + b = 0` (linear).
- **Interpretation:** `wⱼ` = change in **log-odds** per unit `xⱼ`; `exp(wⱼ)` = odds ratio.
- **Multiclass:** softmax regression (multinomial) = a single-layer neural network.
- **Thresholding:** default 0.5 isn't sacred; pick by cost/PR tradeoff.
- Regularized via `C = 1/λ`. Great baseline; often hard to beat on text with TF-IDF.

## 4. k-Nearest Neighbors
- Predict by majority/mean of `k` closest training points. **No training**, expensive inference; needs **scaling**; suffers in high dimensions. `k` small → high variance, large → high bias.
- Distances: Euclidean, Manhattan, cosine. **Approximate NN (ANN)** — the basis of vector DBs (Phase 5).

## 5. Decision trees
- Recursive axis-aligned splits chosen to maximize **impurity reduction**: Gini `1−Σpₖ²`, entropy `−Σpₖlog pₖ` (information gain), variance for regression.
- Pros: interpretable, handles mixed types/missing (some impls), no scaling, captures interactions. Cons: **high variance**, overfit easily, step-function boundaries, unstable.
- Control with `max_depth`, `min_samples_leaf`, pruning (`ccp_alpha`).

## 6. Ensembles
### Bagging & Random Forest
- **Bootstrap aggregation**: train many trees on resampled data, average → **reduces variance**. **Random forest** adds random feature subsets per split → decorrelates trees. OOB error = free validation. Feature importance (impurity-based is biased; prefer **permutation importance** / SHAP).
### Boosting (usually the tabular champion)
- Build models **sequentially**, each fixing the previous errors → **reduces bias**.
- **AdaBoost** (reweight examples), **Gradient Boosting** (fit trees to the *negative gradient* of the loss — functional gradient descent).
- Libraries: **XGBoost**, **LightGBM** (leaf-wise, fast, histogram-based), **CatBoost** (great categorical handling). Key hyperparameters: `learning_rate`, `n_estimators` (use early stopping), `max_depth/num_leaves`, `subsample`, `colsample_bytree`, `min_child_weight`, `reg_lambda`.
- **Stacking/blending**: meta-model over base models' predictions (use out-of-fold predictions to avoid leakage).

## 7. Support Vector Machines (SVM)
- Maximize the **margin** between classes; only **support vectors** matter. **Soft margin** `C` trades margin vs errors. **Kernel trick** (RBF, polynomial) → nonlinear boundaries without explicit features. Scales poorly (`~n²`–`n³`); mostly historical now, but conceptually important (margins, kernels, duality).

## 8. Naive Bayes
- Bayes rule + conditional independence assumption: `P(y|x) ∝ P(y) Π P(xⱼ|y)`. Fast, strong with little data, text classification baseline (Multinomial NB). Poorly calibrated.

## 9. Clustering (unsupervised)
| Algorithm | Idea | Notes |
|---|---|---|
| **k-means** | iteratively assign to nearest centroid, update centroids | spherical clusters, need `k`, k-means++ init, scale features |
| Hierarchical | agglomerative merging → dendrogram | choose linkage |
| **DBSCAN / HDBSCAN** | density-based, finds arbitrary shapes + noise | no `k`; HDBSCAN robust (used in BERTopic) |
| **Gaussian Mixture (EM)** | soft assignments via mixture of Gaussians | |
| Spectral | graph Laplacian | |
Evaluate: silhouette score, Davies–Bouldin, elbow method, stability, *and domain sense*. Uses: customer segmentation, **clustering embeddings to explore/label text data**, dedup.

## 10. Dimensionality reduction & visualization
- **PCA**: project onto top-variance eigenvectors (from SVD of centered data); `explained_variance_ratio_`; denoising, decorrelation, speedups. Linear.
- **t-SNE, UMAP**: nonlinear 2D/3D visualization of embeddings. Distances between clusters/global structure are **not reliable**; use for exploration only. UMAP faster, preserves more global structure.
- **Autoencoders** (nonlinear; see [[07-Deep-Architectures]]). **Random projections**, **Matryoshka embeddings** (Phase 5).

## 11. Anomaly detection
Isolation Forest, One-Class SVM, Local Outlier Factor, autoencoder reconstruction error, statistical z-score/IQR, robust covariance. Hard part: defining "anomaly" and getting labels.

## 12. Recommenders (brief)
Collaborative filtering, **matrix factorization** (SVD-like), content-based, two-tower neural retrieval, learning-to-rank (LambdaMART), cold start. Two-tower = same idea as dense retrieval in RAG.

## 13. Time series (brief)
Stationarity, trend/seasonality, ACF/PACF, ARIMA/ETS, Prophet, **lag-feature + GBM** approach (often strongest), walk-forward validation, probabilistic forecasting, foundation models for TS (Chronos, TimesFM). **Never random-split time series.**

## 14. Interpretability
- Global: coefficients, feature/permutation importance, partial dependence (PDP), ICE.
- Local: **SHAP** (Shapley values), **LIME**, counterfactuals.
- Interpretable-by-design models; trust & regulatory needs (credit, health) matter in FDE engagements.

## 15. Which algorithm when? (cheat sheet)
| Situation | Start with |
|---|---|
| Tabular, any size | **LightGBM/XGBoost** + good features; logistic regression as baseline |
| Need interpretability | linear/logistic, small tree, GAMs, EBM |
| Tiny data | regularized linear, naive Bayes, kNN, SVM |
| Text | TF-IDF + logistic regression baseline → fine-tuned transformer / embeddings + classifier / LLM |
| Images/audio | CNN/ViT pretrained → fine-tune |
| Unlabeled exploration | PCA/UMAP + k-means/HDBSCAN |
| Forecasting | lag features + GBM, ETS/ARIMA baselines |

Next → [[04-Evaluation-and-Metrics]]
