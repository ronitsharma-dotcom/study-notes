# Feature Engineering — Complete Notes (Concepts + Examples)

> Feature engineering is the process of using domain knowledge to create, transform, and select the input variables (features) that a machine learning model learns from. The famous saying: **"Better features beat better algorithms."** A simple model with great features usually beats a complex model with raw, messy features.

---

## 0. What Is a Feature, Really?

A **feature** is one input column the model uses to make predictions. The **target** is what you're trying to predict.

```
| age | salary | city    | num_purchases |  →  | will_churn |
|-----|--------|---------|---------------|     |------------|
| 34  | 50000  | Delhi   | 12            |     | No         |
   ↑       ↑        ↑           ↑                     ↑
        these are FEATURES                       this is TARGET
```

**Feature engineering** = turning raw data (left side) into the BEST possible features so the model can find patterns easily.

**Why it matters:** Models can only learn from what you give them. If the useful signal is hidden or in the wrong form, even the best algorithm fails. If you surface the signal clearly, even a simple model shines.

---

## 1. The Feature Engineering Workflow

```
Raw Data
   │
   ▼
1. Handle Missing Values   →  fill or remove gaps
   │
   ▼
2. Handle Outliers         →  cap or transform extreme values
   │
   ▼
3. Encode Categoricals     →  turn text categories into numbers
   │
   ▼
4. Scale / Transform       →  put numbers on comparable ranges
   │
   ▼
5. Create New Features     →  combine/extract richer signals
   │
   ▼
6. Select Features         →  keep the useful ones, drop the rest
   │
   ▼
Clean Feature Set  →  feed to model
```

We'll go through each step with examples.

---

## 2. Handling Missing Values

Real data has gaps (a survey field left blank, a sensor that failed). Models can't handle `NaN` directly, so you must deal with them.

### Why values go missing (it matters!)

- **MCAR (Missing Completely At Random):** truly random (a form glitch). Safe to impute.
- **MAR (Missing At Random):** missingness depends on another column (older people skip the "income" field). Impute using related columns.
- **MNAR (Missing Not At Random):** missingness depends on the value itself (high earners hide income). The fact it's missing is itself a signal — add a flag.

### Techniques

**1. Deletion**
```
Drop rows: if only a few rows have missing values.
Drop columns: if a column is mostly empty (e.g., >50% missing).
```
Risk: you throw away data. Use only when you can afford to.

**2. Mean / Median / Mode imputation**
```
age:    [25, 30, NaN, 40]  →  fill NaN with median (32.5)
city:   [Delhi, NaN, Mumbai] → fill NaN with mode (most common city)
```
- Use **median** for numeric (robust to outliers — better than mean).
- Use **mode** (most frequent) for categories.

**3. Forward/Backward fill (time series)**
```
temperature: [20, 21, NaN, 23]  →  forward fill → [20, 21, 21, 23]
```
Carry the last known value forward — useful for sequential data.

**4. Model-based imputation**
- **KNN Imputer:** fill a gap using the average of the K most similar rows.
- **Iterative Imputer:** predict the missing column from the others using a model.

**5. Add a "missing indicator" feature (very useful for MNAR)**
```
income: [50000, NaN, 80000]
→ income_filled:    [50000, 65000, 80000]   (imputed)
→ income_was_missing: [0, 1, 0]              (new flag feature)
```
The flag lets the model learn that "missing income" is itself meaningful.

> **Golden rule:** Compute imputation values (like the median) from the TRAINING set only, then apply to test. Computing from the full dataset causes **data leakage**.

---

## 3. Handling Outliers

Outliers are extreme values that can distort a model (especially linear models, which get pulled toward them).

### Detecting outliers

**1. Z-score** (how many std devs from the mean)
```
z = (value − mean) / std
|z| > 3  →  likely an outlier
```

**2. IQR method** (box-plot logic)
```
IQR = Q3 − Q1
Lower bound = Q1 − 1.5 × IQR
Upper bound = Q3 + 1.5 × IQR
Anything outside these bounds = outlier
```

Example: salaries [30k, 32k, 35k, 38k, 40k, **5,000k**]. The 5,000k value is way beyond the upper bound → outlier.

### Handling outliers

- **Remove:** if it's clearly a data-entry error (age = 250).
- **Cap / Winsorize:** clip extreme values to a percentile (e.g., cap everything above the 99th percentile at the 99th percentile value).
- **Transform:** apply a log transform to compress the scale (see Section 5).
- **Keep:** if it's a real, meaningful rare event (a genuine huge transaction in fraud detection — that's exactly what you want to catch!).
- **Use robust models:** tree-based models (Random Forest, XGBoost) are naturally resistant to outliers since they split on thresholds.

---

## 4. Encoding Categorical Variables

Models need numbers, but many features are text categories ("Delhi", "red", "small"). Encoding converts them to numbers.

### Two types of categorical data

- **Nominal:** no order (city, color, brand). Delhi isn't "more than" Mumbai.
- **Ordinal:** has order (small < medium < large; low < high).

### Encoding techniques

**1. Label Encoding** — assign each category an integer.
```
[Delhi, Mumbai, Chennai]  →  [0, 1, 2]
```
- Good for **ordinal** data (small=0, medium=1, large=2) and for tree models.
- BAD for nominal data with linear models — the model wrongly thinks Chennai(2) > Delhi(0), implying a false order.

**2. One-Hot Encoding** — create a binary column per category.
```
color: [red, blue, green]
→  is_red | is_blue | is_green
   1      | 0       | 0
   0      | 1       | 0
   0      | 0       | 1
```
- Best for **nominal** data — no false ordering.
- Downside: explodes columns if there are many categories (1000 cities → 1000 columns). This is the "curse of dimensionality" risk.

**3. Ordinal Encoding** — manual integers that respect order.
```
size: [small, medium, large]  →  [1, 2, 3]   (you define the order)
```

**4. Target (Mean) Encoding** — replace category with the average target value for that category.
```
For predicting purchase amount:
city     avg_purchase
Delhi → 1200
Mumbai → 1500
→ replace "Delhi" with 1200, "Mumbai" with 1500
```
- Powerful for **high-cardinality** features (many categories).
- RISK: data leakage / overfitting — always compute means on training folds only (use smoothing or cross-fold encoding).

**5. Frequency / Count Encoding** — replace category with how often it appears.
```
city: [Delhi×500, Mumbai×300]  →  Delhi=500, Mumbai=300
```

**6. Hashing / Embeddings** — for very high cardinality.
- **Hashing:** map categories into a fixed number of buckets.
- **Embeddings:** learn a dense vector per category (used in deep learning, e.g., for words or product IDs).

### Quick guide

| Situation | Best Encoding |
|---|---|
| Ordered categories | Ordinal / Label |
| Few unordered categories | One-Hot |
| Many categories (high cardinality) | Target / Frequency / Hashing |
| Tree-based model | Label encoding is usually fine |
| Linear model / neural net | One-Hot or embeddings |

---

## 5. Scaling & Transforming Numeric Features

### Why scale?

Features on different ranges confuse distance- and gradient-based models. If salary is 0–1,000,000 and age is 0–100, salary dominates every distance calculation.

```
Needs scaling:    KNN, SVM, Linear/Logistic Regression, Neural Nets, PCA, K-Means
Does NOT need it: Decision Tree, Random Forest, XGBoost (split-based)
```

### Scaling techniques

**1. Standardization (StandardScaler)** — mean 0, std 1.
```
x_new = (x − mean) / std
```
- Result: most values fall roughly in [−3, 3]. Not bounded.
- Use when features have different units; less sensitive to outliers than MinMax.
- The default choice for most models.

**2. Normalization (MinMaxScaler)** — squeeze to [0, 1].
```
x_new = (x − min) / (max − min)
```
- Bounded to [0,1]. Use for neural network inputs, image pixels.
- Sensitive to outliers (one huge value squashes everything else).

**3. Robust Scaling (RobustScaler)** — uses median and IQR.
```
x_new = (x − median) / IQR
```
- Best when data has outliers (median/IQR aren't pulled by extremes).

### Transformations (fix skewed distributions)

Many real features are skewed (income, prices, populations — a few huge values, many small). Models like linear regression prefer roughly normal (bell-shaped) features.

**1. Log transform** — compresses large values, expands small ones.
```
income: [1k, 5k, 1,000k]  →  log → [3, 3.7, 6]
```
Turns a long right tail into something more symmetric. (Use `log1p` = log(1+x) to handle zeros.)

**2. Square root** — milder compression than log.

**3. Box-Cox / Yeo-Johnson** — automatically find the best power transform to make data normal. (Box-Cox needs positive values; Yeo-Johnson handles zero/negatives.)

```
Before (right-skewed):        After log transform:
  ▮                              ▮ ▮ ▮
  ▮▮                             ▮ ▮ ▮ ▮
  ▮▮▮▮___________               ▮ ▮ ▮ ▮ ▮
  long tail →                   more balanced
```

---

## 6. Creating New Features (The Creative Part)

This is where domain knowledge shines — combining or extracting to surface hidden signal.

### 6.1 Extracting from dates/timestamps

A raw date string is useless to a model. Break it apart:
```
"2026-06-12 14:30"  →
  year = 2026
  month = 6
  day = 12
  day_of_week = Friday
  is_weekend = 0
  hour = 14
  is_business_hours = 1
  quarter = Q2
```
Example: predicting store sales — `is_weekend` and `month` (holiday season) are far more predictive than the raw date.

### 6.2 Combining features (interactions)

```
price × quantity        → total_spend
weight / (height²)      → BMI
distance / time         → speed
debt / income           → debt_to_income_ratio
```
These ratios/products often capture the real relationship better than the parts alone.

### 6.3 Aggregations (group statistics)

```
For each customer:
  avg_purchase_amount
  total_number_of_orders
  days_since_last_order
  max_single_purchase
```
Turns transaction-level rows into rich customer-level features.

### 6.4 Binning (discretization)

Convert a continuous feature into categories:
```
age: 23  →  age_group: "18-30"
age: 67  →  age_group: "60+"
```
Useful when the relationship isn't linear (risk doesn't rise smoothly with age — it jumps at certain life stages).

### 6.5 Polynomial features

For linear models to capture curves:
```
from x  →  create x², x³, and x₁×x₂
```
Lets a straight-line model fit curved relationships. (Careful: can explode feature count and overfit.)

### 6.6 Text features

```
"Great product, fast delivery!"  →
  char_count = 30
  word_count = 4
  has_exclamation = 1
  sentiment_score = 0.9
  contains_keyword("delivery") = 1
```
Plus advanced: TF-IDF, bag-of-words, or embeddings for full text understanding.

### 6.7 Domain-specific features

The most valuable — born from understanding the problem:
```
Fraud detection:   transactions_in_last_hour, is_foreign_country, amount_vs_user_average
E-commerce:        cart_abandonment_rate, avg_session_duration
Real estate:       price_per_sqft, distance_to_metro, age_of_building
```

---

## 7. Feature Selection (Keeping Only the Useful Ones)

More features isn't always better — irrelevant features add noise, slow training, and cause overfitting. Feature selection keeps the signal, drops the rest.

### Three families of methods

**1. Filter methods** — rank features by a statistical score, independent of any model.
```
- Correlation with target (numeric)
- Chi-square test (categorical)
- Mutual information
- Variance threshold (drop near-constant features)
```
Fast, but ignores how features work together.

**2. Wrapper methods** — try different feature subsets, train a model, keep what works best.
```
- Forward selection: start empty, add the best feature one at a time
- Backward elimination: start with all, remove the weakest one at a time
- Recursive Feature Elimination (RFE)
```
Best results, but slow (trains many models).

**3. Embedded methods** — selection happens DURING model training.
```
- Lasso (L1): drives useless feature weights to exactly zero
- Tree feature importance: Random Forest / XGBoost rank features
```
Great balance of quality and speed — the most practical choice.

### Removing redundancy

If two features are highly correlated (e.g., "age" and "years since birth"), they carry the same info — drop one. Use a **correlation matrix** to spot pairs with correlation > 0.9.

---

## 8. The Golden Rule: Avoid Data Leakage

**Data leakage** = letting information from the test set (or the future) sneak into training. It makes your model look amazing in testing but fail in the real world.

**Classic mistake — scaling before splitting:**
```
WRONG:                              RIGHT:
scaler.fit(ALL_data)                train, test = split(data)
train, test = split(...)            scaler.fit(train)        ← fit on TRAIN only
                                    scaler.transform(test)   ← apply to test
```
If you fit the scaler (or imputer, or target encoder) on all data, the test set's statistics leak into training.

**Other leakage sources:**
- Using a feature that's secretly the answer (e.g., "discharge date" when predicting hospital admission).
- Computing target encoding on the full dataset.
- Using future data to predict the past in time series.

**The fix:** Always split FIRST. Fit all transformations on training data only. Use a **Pipeline** so preprocessing is automatically applied correctly within each cross-validation fold.

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.impute import SimpleImputer

pipeline = Pipeline([
    ("impute", SimpleImputer(strategy="median")),
    ("scale", StandardScaler()),
    ("model", model),
])
# fit only on training data — leakage-safe
pipeline.fit(X_train, y_train)
```

---

## 9. Putting It All Together — A Worked Example

**Goal:** predict whether a customer will churn.

**Raw data:**
```
| customer_id | signup_date | last_login   | plan    | monthly_spend | support_tickets |
| 1001        | 2024-01-15  | 2026-06-01   | premium | 1200          | NaN             |
```

**Feature engineering steps:**

```
1. Missing values:
   support_tickets NaN → fill with 0 (no tickets logged)
   + add support_tickets_was_missing flag

2. Date features:
   account_age_days = today − signup_date          → 879
   days_since_last_login = today − last_login       → 11
   (days_since_last_login is a STRONG churn signal!)

3. Encode categorical:
   plan: [basic, premium, enterprise] → ordinal [1, 2, 3]

4. Create new features:
   spend_per_day = monthly_spend / 30
   is_inactive = (days_since_last_login > 30) ? 1 : 0

5. Scale numerics:
   StandardScaler on monthly_spend, account_age_days, etc.

6. Drop useless:
   customer_id (just an identifier — no predictive value)
   raw dates (already extracted what we need)

7. Select:
   keep features correlated with churn, drop redundant ones
```

**Result:** a clean, numeric, leakage-free feature set where the most predictive signals (inactivity, low engagement) are made explicit — so even a simple logistic regression can predict churn well.

---

## 10. Key Takeaways

- **Features > algorithms.** Time spent on good features usually pays off more than tuning models.
- **Always handle:** missing values, outliers, categorical encoding, scaling (for the right models).
- **Create features** using domain knowledge — dates, ratios, aggregations, bins.
- **Select features** to cut noise — filter, wrapper, or embedded methods.
- **Never leak data** — split first, fit transformations on training only, use pipelines.
- **Tree models** (RF, XGBoost) need less scaling and handle outliers/missing better; **linear/distance models** (Linear Reg, KNN, SVM) need careful scaling and encoding.

### Quick Reference Table

| Task | Common Techniques |
|---|---|
| Missing values | Median/mode impute, KNN imputer, missing-flag |
| Outliers | IQR/Z-score detect; cap, log-transform, or keep |
| Nominal categories | One-Hot encoding |
| Ordinal categories | Ordinal/Label encoding |
| High-cardinality categories | Target / Frequency / Hashing encoding |
| Scaling | StandardScaler (default), MinMax, RobustScaler |
| Skewed data | Log, sqrt, Box-Cox / Yeo-Johnson |
| New features | Date parts, ratios, aggregations, binning, polynomials |
| Feature selection | Correlation, RFE, Lasso, tree importance |
| Avoid leakage | Split first, fit on train only, use Pipelines |
