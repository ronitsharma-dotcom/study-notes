# Machine Learning Revision — Part 2: Top 50 Interview Questions

> The 50 most-asked Machine Learning interview questions for AI Engineer / Data Scientist roles, with concise, interview-ready answers. Organized by topic. Master these and you cover the vast majority of what gets asked.

---

## Section A: Core Concepts (8 Questions)

**Q1. What's the difference between AI, ML, and Deep Learning?**

AI is any system that mimics intelligence (includes rule-based systems). ML is a subset where systems LEARN from data instead of being explicitly programmed. DL is a subset of ML using multi-layer neural networks. DL excels at unstructured data (images, text, audio) with lots of data; traditional ML is better for structured/tabular data and smaller datasets.

---

**Q2. What's the difference between supervised, unsupervised, and reinforcement learning?**

Supervised: data has labels (input + correct answer); learns input→output mapping (spam detection, price prediction). Unsupervised: no labels; finds hidden patterns/groups (clustering, anomaly detection). Reinforcement: an agent takes actions in an environment, gets rewards/penalties, and learns an optimal strategy (game playing, robotics).

---

**Q3. Explain the bias-variance tradeoff.**

Bias = error from the model being too SIMPLE (wrong assumptions) → underfitting → bad on BOTH training and test. Variance = error from the model being too SENSITIVE to data → overfitting → great on training but bad on test (big gap). Making a model more complex lowers bias but raises variance, and vice versa. The goal is the sweet spot where total error (bias² + variance + irreducible noise) is lowest. Detect via the train/test gap.

---

**Q4. What is overfitting and underfitting? How do you fix each?**

Overfitting: model memorizes training data (incl. noise) → high train accuracy, low test accuracy. Fix: more data, regularization (L1/L2/dropout), simpler model, early stopping, cross-validation. Underfitting: model too simple → bad on both train and test. Fix: more complex model, more/better features, less regularization, train longer.

---

**Q5. What is regularization? Explain L1 vs L2.**

Regularization adds a penalty for large weights to prevent overfitting. L2 (Ridge): penalty Σw² → shrinks all weights toward zero but never exactly zero → keeps all features. L1 (Lasso): penalty Σ|w| → drives some weights to exactly zero → automatic feature selection. Geometrically, L1's diamond constraint has corners on the axes (→ zeros), L2's circle is smooth (→ small but non-zero). Elastic Net combines both.

---

**Q6. What is feature engineering? Why is it important?**

Creating new features from existing ones to help the model learn better. Examples: date → day_of_week/is_weekend; text → word count/sentiment; price + quantity → total_spend; age → age bins. It's often more impactful than algorithm choice — good features can make a simple model outperform a complex one on bad features.

---

**Q7. What is feature scaling and when is it necessary?**

Transforming features to similar ranges. StandardScaler (mean 0, std 1) or MinMaxScaler ([0,1]). NECESSARY for distance/gradient-based algorithms: KNN, SVM, Linear/Logistic Regression, Neural Nets, PCA, K-Means. NOT necessary for tree-based methods (Decision Tree, Random Forest, XGBoost) since they split on thresholds and don't use distances.

---

**Q8. What is the curse of dimensionality?**

In high-dimensional space, data becomes sparse and all points become roughly equidistant — so "nearest" loses meaning. This hurts distance-based algorithms (KNN, clustering) badly and requires exponentially more data to fill the space. Fixes: dimensionality reduction (PCA), feature selection, or using algorithms less sensitive to it (tree-based).

---

## Section B: Regression & Classification (8 Questions)

**Q9. What are the assumptions of Linear Regression?**

(1) Linearity — features have a linear relationship with the target. (2) No multicollinearity — features aren't highly correlated with each other. (3) Homoscedasticity — constant variance of errors. (4) Normality of residuals — errors are normally distributed. (5) No autocorrelation — residuals are independent. Violations are handled with polynomial features, removing correlated features/Ridge, weighted regression, or time-series methods.

---

**Q10. Why can't we use MSE as the loss for Logistic Regression?**

MSE combined with the sigmoid creates a non-convex loss surface (many local minima) → gradient descent gets stuck. Log loss (binary cross-entropy) is convex with sigmoid → guaranteed global minimum. Log loss also heavily penalizes confident wrong predictions, giving better gradients.

---

**Q11. What is the sigmoid function and why is it used in Logistic Regression?**

Sigmoid: σ(z) = 1/(1+e⁻ᶻ). It squashes any real number into (0,1), interpretable as a probability. Logistic regression computes a linear combination z = wx+b (any value), passes it through sigmoid to get P(class=1), and predicts class 1 if P > 0.5 (threshold adjustable).

---

**Q12. Explain precision, recall, and F1-score. When does each matter?**

Precision = TP/(TP+FP): "of predicted positives, how many are correct?" — matters when false positives are costly (spam filter). Recall = TP/(TP+FN): "of actual positives, how many did we catch?" — matters when false negatives are costly (cancer detection). F1 = harmonic mean of the two — use when you need balance and classes are imbalanced.

---

**Q13. What is AUC-ROC and why is it better than accuracy for imbalanced data?**

ROC plots True Positive Rate vs False Positive Rate across thresholds; AUC is the area under it (1 = perfect, 0.5 = random). It's threshold- and class-balance-independent. With 95% one class, predicting all-majority gives 95% accuracy but is useless — AUC correctly shows ~0.5. So AUC reveals true discriminative ability that accuracy hides.

---

**Q14. What's the difference between Type I and Type II errors?**

Type I (False Positive): predicting positive when it's actually negative — a "false alarm" (healthy diagnosed sick). Type II (False Negative): predicting negative when it's actually positive — a "missed detection" (sick diagnosed healthy). Which is worse depends on context — in cancer screening, Type II is far worse.

---

**Q15. How do you handle class imbalance (e.g., 99% vs 1%)?**

(1) Resampling — SMOTE (oversample minority) or undersample majority. (2) Class weights — penalize minority errors more. (3) Right metrics — precision/recall/F1/AUC-PR, never plain accuracy. (4) Threshold tuning — move off 0.5. (5) Anomaly-detection framing if extreme. (6) Collect more minority data. Choice depends on severity and cost of each error type.

---

**Q16. What is the difference between generative and discriminative models?**

Discriminative (Logistic Regression, SVM, neural nets): model P(class|features) — learn the decision boundary directly. Usually more accurate for classification. Generative (Naive Bayes, GMM): model P(features|class) and P(class), then apply Bayes' theorem. Can generate new data and handle missing features, but often less accurate for pure classification.

---

## Section C: Tree-Based & Ensemble Methods (9 Questions)

**Q17. How does a Decision Tree decide where to split?**

At each node it tries every feature and threshold, measuring how well each split separates classes using Gini impurity or Information Gain (entropy). It picks the split with the biggest impurity reduction, then recurses until leaves are pure or a stopping criterion (max_depth, min_samples) is met. For regression, it minimizes variance/MSE instead.

---

**Q18. Why do Decision Trees overfit and how do you prevent it?**

Trees grow until leaves are pure → they memorize noise. Prevent via: max_depth, min_samples_leaf, min_samples_split, max_features (pre-pruning); post-pruning (grow then cut branches that don't help validation); or use ensembles (Random Forest). 

---

**Q19. Explain Random Forest. Why is it better than a single tree?**

Random Forest = many decision trees, each trained on a random bootstrap sample of data AND a random subset of features at each split, then averaged/voted. Individual trees are high-variance, but averaging cancels their errors → much lower variance with low bias. The random feature selection ensures trees are diverse (not all dominated by one strong feature), which is what makes averaging effective.

---

**Q20. What is the difference between Bagging and Boosting?**

Bagging: trains models INDEPENDENTLY (parallel) on random subsets, combines by averaging → reduces VARIANCE; uses complex base models (e.g., Random Forest). Boosting: trains models SEQUENTIALLY, each fixing the previous one's errors → reduces BIAS; uses weak base models (e.g., XGBoost, AdaBoost). Bagging = strong learners averaged; boosting = weak learners built up.

---

**Q21. Explain AdaBoost step by step.**

(1) Start with equal sample weights. (2) Train a weak learner (stump). (3) Compute its error and its "say" α = 0.5·ln((1−err)/err). (4) Increase weights of misclassified samples, decrease correctly-classified ones. (5) Normalize. (6) Next learner focuses on the now-heavier hard samples. (7) Repeat; final prediction = weighted vote using each learner's α.

---

**Q22. What is Gradient Boosting? Explain the intuition.**

Build models sequentially where each new tree predicts the RESIDUALS (errors) of the current ensemble. Start with a base prediction (e.g., the mean), compute residuals, train a tree on them, add its (scaled) output, recompute smaller residuals, repeat. Each tree corrects the remaining error by a small step (learning rate η). Final = base + η·Tree1 + η·Tree2 + …

---

**Q23. What makes XGBoost better than regular Gradient Boosting?**

(1) Built-in L1+L2 regularization on leaf weights. (2) Parallel tree construction. (3) Native missing-value handling. (4) Smart tree pruning (grow then prune). (5) Column subsampling (like Random Forest). (6) Cache-aware, sparsity-aware optimization. Result: faster, more accurate, and more resistant to overfitting — which is why it dominates tabular-data competitions.

---

**Q24. Why does ensemble learning work?**

Combining diverse models cancels out their individual errors. The key requirement is DIVERSITY — if all models make the same mistakes, combining them doesn't help. Bagging creates diversity via random data/features (reduces variance); boosting via sequential error-correction (reduces bias); stacking trains a meta-model to best combine different model types.

---

**Q25. How do you tune XGBoost hyperparameters?**

Start: learning_rate=0.1, max_depth=6, n_estimators=100. Then tune in order: (1) max_depth & min_child_weight (complexity), (2) subsample & colsample_bytree (randomness), (3) reg_alpha & reg_lambda (regularization), (4) finally lower learning_rate to ~0.01 and raise n_estimators. Use early_stopping_rounds with a validation set.

---

## Section D: Clustering & Unsupervised (6 Questions)

**Q26. How do you choose K in K-Means? Explain the Elbow Method.**

Run K-Means for K=1..10; for each, compute WCSS (within-cluster sum of squares). Plot K vs WCSS — it drops steeply then flattens; the "elbow" is a good K. Also use the Silhouette Score (pick the K with the highest average silhouette) since the elbow can be subjective.

---

**Q27. What are the limitations of K-Means?**

(1) Must specify K. (2) Assumes spherical, equal-sized clusters. (3) Sensitive to initialization (fix: K-Means++). (4) Sensitive to outliers and scale. (5) Only finds convex clusters. Alternatives: DBSCAN (arbitrary shapes + outlier detection), Hierarchical (unknown K).

---

**Q28. Explain DBSCAN. When is it better than K-Means?**

DBSCAN groups points by density: core points (≥ MinPts within radius ε), border points, and noise (outliers). Better than K-Means when clusters are irregularly shaped, K is unknown, or data has outliers (DBSCAN labels them as noise rather than forcing them into a cluster). Weakness: struggles with clusters of very different densities.

---

**Q29. What is the Silhouette Score and how do you interpret it?**

s = (b − a)/max(a,b), where a = avg distance to own-cluster points, b = avg distance to nearest other cluster. Range [−1, +1]: +1 = well-clustered, 0 = on a boundary, −1 = likely wrong cluster. Average it across all points to compare different K values or algorithms — pick the highest.

---

**Q30. Compare Hierarchical Clustering with K-Means.**

Hierarchical: no need to pre-specify K, produces a dendrogram you can cut at any level, deterministic, but slow O(n³) — good for small data. K-Means: needs K upfront, fast O(n), scales to large data, but only finds spherical clusters and is non-deterministic. Use hierarchical to explore structure on small data; K-Means for large data with known K.

---

**Q31. What is PCA and when would you use it?**

PCA finds new orthogonal axes (principal components) capturing maximum variance, letting you reduce dimensions while keeping most information. Use it for: too many/correlated features, speeding up training, fighting the curse of dimensionality, and 2D/3D visualization. Limitations: assumes linear relationships and the components are hard to interpret.

---

## Section E: SVM, KNN & Naive Bayes (5 Questions)

**Q32. Explain the kernel trick in SVM. Why is it needed?**

When data isn't linearly separable, the kernel maps it to a higher-dimensional space where it becomes separable. The "trick": you never compute the high-dimensional coordinates — the kernel function computes the dot product in that space directly, saving huge computation. RBF kernel effectively maps to infinite dimensions. SVM finds the maximum-margin boundary (widest "street") between classes.

---

**Q33. What is the C parameter in SVM?**

C controls the penalty for misclassification. High C: few mistakes allowed → narrow margin, fits training tightly → risk of overfitting. Low C: tolerates mistakes for a wider margin → better generalization → risk of underfitting. It's SVM's bias-variance dial; tune via cross-validation. (For RBF, gamma similarly controls boundary complexity.)

---

**Q34. How does KNN work and how do you choose K?**

KNN classifies a point by majority vote of its K nearest neighbors (or averages for regression). It's a lazy learner — no training, all work at prediction time. Small K → flexible, overfits (noise-sensitive); large K → smooth, underfits. Choose K via cross-validation (try odd values to avoid ties; rule of thumb √n). KNN REQUIRES feature scaling.

---

**Q35. Explain Naive Bayes and why it works despite its "naive" assumption.**

It applies Bayes' theorem assuming all features are INDEPENDENT given the class, so it just multiplies individual feature probabilities and picks the highest-scoring class. The independence assumption is usually false, but it still works because: (1) we only need the correct RANKING, not exact probabilities; (2) over- and under-estimates tend to cancel. It's fast, great for text, and works with little data. Use Laplace smoothing to avoid zero-probability issues.

---

**Q36. What is Gradient Descent? Explain learning rate and its variants.**

An iterative optimization that minimizes loss by repeatedly stepping in the opposite direction of the gradient: w = w − α·gradient. Learning rate α: too small → painfully slow; too large → overshoots/diverges. Variants: Batch GD (all data, stable, slow), SGD (1 sample, fast, noisy), Mini-batch (32–256, the practical default). Advanced optimizers: Momentum, RMSProp, Adam (the go-to).

---

## Section F: Applied ML & Best Practices (6 Questions)

**Q37. What is cross-validation and why K=5 or K=10?**

Split data into K folds; train on K−1, validate on 1; repeat K times and average. It gives a reliable generalization estimate using all data for both training and validation. K=5–10 balances the estimate's bias and variance — too small wastes training data, too large (leave-one-out) is expensive and high-variance.

---

**Q38. What is data leakage? Give an example.**

Using information during training that wouldn't be available at prediction time, causing falsely high scores. Example: fitting a scaler on the FULL dataset before the train/test split (test info leaks into training), or including a feature that's a proxy for the target ("date of next admission" when predicting readmission). Fix: split first, then preprocess; use pipelines.

---

**Q39. How do you handle missing values?**

(1) Drop rows (if few) or columns (if mostly missing). (2) Impute mean/median (numeric) or mode (categorical). (3) Model-based imputation (KNN/iterative imputer). (4) Add an "is_missing" indicator feature. (5) Use algorithms that handle missing natively (XGBoost). Choice depends on the % missing, whether it's random vs systematic, and dataset size.

---

**Q40. How do you detect and handle outliers?**

Detect: Z-score (|z|>3), IQR method (outside Q1−1.5·IQR / Q3+1.5·IQR), box/scatter plots, or Isolation Forest for multivariate. Handle: remove if clearly errors, cap/clip (winsorize), log-transform to reduce impact, or use robust (tree-based) algorithms. Keep them if they represent real, meaningful rare events.

---

**Q41. How would you approach a brand-new ML problem from scratch?**

(1) Define the problem and metric. (2) EDA — distributions, correlations, missing values, outliers. (3) Feature engineering. (4) Train/val/test split. (5) Simple baseline (logistic regression / decision tree). (6) Try multiple algorithms. (7) Hyperparameter tuning via cross-validation. (8) Evaluate on the test set ONCE at the end. (9) Error analysis. (10) Iterate.

---

**Q42. What is concept drift and how do you handle it in production?**

Concept drift = the relationship between features and target changes over time (e.g., customer behavior post-COVID). Detect by monitoring accuracy over time and tracking input/prediction distributions (PSI). Handle by: retraining on recent data, sliding-window training, online learning, and alerts that trigger retraining when performance drops below a threshold.

---

## Section G: Modern AI / LLM Essentials (8 Questions)

**Q43. What is a neural network and how does it learn (backpropagation)?**

Layers of neurons, each computing a weighted sum + bias + non-linear activation. Learning: a forward pass produces a prediction, a loss measures the error, then backpropagation uses the chain rule to compute each weight's gradient (its share of the blame), and gradient descent updates the weights. Repeat over many batches until convergence.

---

**Q44. Why do we need non-linear activation functions? Compare ReLU, sigmoid, tanh.**

Without non-linearity, stacked linear layers collapse into a single linear function — the network couldn't learn complex patterns. Sigmoid (0,1) and tanh (−1,1) saturate and cause vanishing gradients. ReLU = max(0,x) is the default for hidden layers: cheap, non-saturating for positives, avoids vanishing gradients (weakness: "dying ReLU," fixed by Leaky ReLU/GELU).

---

**Q45. What is the vanishing gradient problem and how is it fixed?**

In deep networks, gradients are products of many terms; if each < 1, the product shrinks toward zero, so early layers barely learn. Caused by sigmoid/tanh and poor initialization. Fixes: ReLU activations, good initialization (He/Xavier), batch normalization, residual/skip connections (ResNet), and gradient clipping (for the exploding case).

---

**Q46. Explain the attention mechanism in transformers (simply).**

Attention lets each token decide which other tokens matter. Each token forms a Query; every token has a Key and Value. The Query is compared to all Keys (dot product) to get relevance scores, which weight the Values into a context-aware representation. In "the animal didn't cross the street because it was tired," attention helps "it" focus on "animal." This parallel, long-range modeling is why transformers replaced RNNs.

---

**Q47. RAG vs fine-tuning — when do you use which?**

RAG injects external knowledge at query time via retrieval — use for changing facts, private data, citations, and reducing hallucination. Fine-tuning bakes behavior into the weights — use to change style/format/tone or teach a domain skill. Rule of thumb: **RAG for knowledge, fine-tuning for behavior.** They're complementary and often combined.

---

**Q48. Explain the full RAG pipeline and why we need it.**

Indexing (offline): load docs → chunk → embed → store in a vector DB. Query (online): embed the question → retrieve top-K similar chunks → (rerank) → put chunks + question in a prompt → LLM answers, grounded with citations. We need RAG because LLMs have static knowledge (training cutoff), can't see private data, and hallucinate — RAG grounds answers in real, up-to-date sources without retraining. Bad retrieval = bad answer.

---

**Q49. What is hallucination and how do you reduce it?**

When an LLM generates plausible but false information — because it predicts probable text and has no built-in "I don't know." Reduce via: RAG (ground in sources), strict prompts ("answer only from context; else say you don't know"), low temperature, forced citations, output validation/guardrails, and allowing the model to abstain. You can mitigate but not fully eliminate it.

---

**Q50. What is hybrid search and why is it better than pure vector search in RAG?**

Hybrid search combines semantic (embedding/vector) search with keyword search (BM25). Vector search captures meaning (synonyms, paraphrases) but blurs exact terms; keyword search nails exact matches (names, IDs, part numbers, rare jargon). Combining them (weighted, e.g., 0.8 vector + 0.2 BM25, or reciprocal rank fusion) gives higher recall than either alone — essential for domains with specific terminology. Adding a reranker on top further sharpens the final results.

---

## Quick Prep Tips

- **For every answer, add a concrete example** from a project you've built (e.g., your teleshop RAG: hybrid FAISS 0.8 + BM25 0.2, single-chunk-per-product, reranking). Real experience beats textbook definitions.
- **Know the tradeoffs, not just definitions.** Senior interviewers ask "which would you choose and why?" — be ready to defend decisions.
- **Most-loved deep dives:** bias-variance, overfitting fixes, precision/recall tradeoff, bagging vs boosting, RAG vs fine-tuning. Expect follow-up questions on these.
