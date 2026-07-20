# NumPy & Pandas — Complete Revision Notes

> Two libraries in one file. NumPy first (foundations), then Pandas (built on NumPy).

---

# PART 1: NumPy

## 1. What is NumPy?

- **Numerical Python** — fast N-dimensional array operations in C under the hood.
- The core object is `ndarray` — a fixed-type, fixed-size, multidimensional array.
- **Why it's fast:** contiguous memory layout + vectorized operations (no Python loops).

```python
import numpy as np
```

---

## 2. Creating Arrays

```python
# From lists
a = np.array([1, 2, 3])              # 1D: shape (3,)
b = np.array([[1,2,3],[4,5,6]])       # 2D: shape (2,3)

# Zeros, Ones, Full
np.zeros((3, 4))         # 3x4 array of 0.0
np.ones((2, 3))          # 2x3 array of 1.0
np.full((2, 2), 7)       # 2x2 array of 7

# Range/Linspace
np.arange(0, 10, 2)      # [0, 2, 4, 6, 8]  (start, stop, step)
np.linspace(0, 1, 5)     # [0.0, 0.25, 0.5, 0.75, 1.0]  (5 evenly spaced)

# Identity & Random
np.eye(3)                # 3x3 identity matrix
np.random.rand(3, 4)     # 3x4 uniform [0,1)
np.random.randn(3, 4)    # 3x4 standard normal
np.random.randint(0, 10, size=(3, 3))  # 3x3 random ints [0,10)
```

---

## 3. Array Properties

```python
a = np.array([[1,2,3],[4,5,6]])

a.shape      # (2, 3) — rows x cols
a.ndim       # 2 — number of dimensions
a.size       # 6 — total elements
a.dtype      # int64 (or float64, etc.)
a.itemsize   # 8 bytes per element
a.nbytes     # 48 total bytes (6 * 8)
```

---

## 4. Indexing & Slicing

```python
a = np.array([[1,2,3],[4,5,6],[7,8,9]])

# Basic indexing
a[0, 0]       # 1
a[1, 2]       # 6
a[-1, -1]     # 9

# Slicing [row_start:row_end, col_start:col_end]
a[0:2, :]     # first 2 rows, all cols → [[1,2,3],[4,5,6]]
a[:, 1]       # all rows, col 1 → [2, 5, 8]
a[1:, :2]     # rows 1+, first 2 cols → [[4,5],[7,8]]

# Boolean indexing (VERY common)
a[a > 5]      # [6, 7, 8, 9] — flat array of elements matching condition
a[a % 2 == 0] # [2, 4, 6, 8]

# Fancy indexing (index with array)
a[[0, 2], :]  # rows 0 and 2
```

> **Key:** NumPy slicing returns a **view** (not a copy). Modifying the slice modifies the original! Use `.copy()` to avoid this.

---

## 5. Reshaping

```python
a = np.arange(12)         # [0,1,2,...,11]

a.reshape(3, 4)           # 3x4 matrix (total elements must match)
a.reshape(2, -1)          # 2x6 (-1 = auto-calculate)
a.reshape(-1, 1)          # column vector (12x1)

a.flatten()               # returns a COPY as 1D
a.ravel()                 # returns a VIEW as 1D (faster)

# Transpose
b = np.array([[1,2,3],[4,5,6]])
b.T                       # shape (3,2)

# Adding a dimension
a = np.array([1,2,3])     # shape (3,)
a[np.newaxis, :]          # shape (1,3) — row vector
a[:, np.newaxis]          # shape (3,1) — column vector
```

---

## 6. Operations (Vectorized — no loops!)

```python
a = np.array([1, 2, 3, 4])
b = np.array([10, 20, 30, 40])

# Element-wise arithmetic
a + b       # [11, 22, 33, 44]
a * b       # [10, 40, 90, 160]
a ** 2      # [1, 4, 9, 16]
a / b       # [0.1, 0.1, 0.1, 0.1]

# Scalar broadcasting
a * 5       # [5, 10, 15, 20]
a + 100     # [101, 102, 103, 104]

# Universal functions (ufuncs)
np.sqrt(a)        # [1.0, 1.41, 1.73, 2.0]
np.exp(a)         # e^1, e^2, e^3, e^4
np.log(a)         # natural log
np.sin(a)
np.abs(a)
np.maximum(a, b)  # element-wise max
```

---

## 7. Aggregations

```python
a = np.array([[1,2,3],[4,5,6]])

a.sum()            # 21 (all elements)
a.sum(axis=0)      # [5, 7, 9] (sum each column — collapse rows)
a.sum(axis=1)      # [6, 15] (sum each row — collapse cols)

a.mean()           # 3.5
a.std()            # standard deviation
a.min()            # 1
a.max()            # 6
a.argmin()         # 0 (index of min)
a.argmax()         # 5 (index of max)
np.median(a)       # 3.5
np.percentile(a, 75)  # 75th percentile
```

> **axis=0** = operate along rows (result per column). **axis=1** = operate along columns (result per row). Think: "collapse that axis."

---

## 8. Broadcasting

NumPy's way of operating on arrays of different shapes **without copying data**.

**Rules:**
1. If ndims differ, pad the smaller shape with 1s on the left.
2. Dimensions are compatible if they're equal OR one of them is 1.
3. The size-1 dimension is "stretched" to match.

```python
a = np.array([[1,2,3],    # shape (2,3)
              [4,5,6]])
b = np.array([10,20,30])  # shape (3,) → broadcast to (2,3)

a + b  # [[11,22,33],[14,25,36]]

# Column operation
col = np.array([[100],[200]])  # shape (2,1)
a + col  # [[101,102,103],[204,205,206]]
```

---

## 9. Linear Algebra

```python
a = np.array([[1,2],[3,4]])
b = np.array([[5,6],[7,8]])

np.dot(a, b)        # matrix multiply (or a @ b)
a @ b               # same — preferred syntax

np.linalg.inv(a)           # inverse
np.linalg.det(a)           # determinant
np.linalg.eig(a)           # eigenvalues & eigenvectors
np.linalg.norm(a)          # Frobenius norm
np.linalg.solve(a, b)      # solve Ax = b
```

---

## 10. Common Patterns

```python
# Where (conditional select)
np.where(a > 2, a, 0)   # keep elements > 2, replace rest with 0

# Sorting
np.sort(a, axis=1)       # sort each row
np.argsort(a)            # indices that would sort

# Unique
np.unique([1,1,2,3,3])   # [1, 2, 3]

# Stacking
np.vstack([a, a])        # vertical stack
np.hstack([a, a])        # horizontal stack
np.concatenate([a, a], axis=0)  # general concat

# Copying
b = a.copy()             # true copy, independent
```

---

# PART 2: Pandas

## 1. What is Pandas?

- **Panel Data** library — built on NumPy for structured/tabular data.
- Two core objects: **Series** (1D) and **DataFrame** (2D table).
- Think of DataFrame as an Excel sheet or SQL table in Python.

```python
import pandas as pd
```

---

## 2. Series (1D labeled array)

```python
s = pd.Series([10, 20, 30, 40], index=["a", "b", "c", "d"])

s["b"]          # 20
s[1]            # 20 (positional)
s[s > 15]       # b:20, c:30, d:40

s.values        # numpy array: [10,20,30,40]
s.index         # Index(['a','b','c','d'])
s.dtype         # int64
s.mean()        # 25.0
```

---

## 3. DataFrame — Creation

```python
# From dict
df = pd.DataFrame({
    "name": ["Alice", "Bob", "Charlie"],
    "age": [25, 30, 35],
    "salary": [50000, 60000, 70000]
})

# From list of dicts
data = [{"name": "Alice", "age": 25}, {"name": "Bob", "age": 30}]
df = pd.DataFrame(data)

# From CSV (most common)
df = pd.read_csv("data.csv")
df = pd.read_csv("data.csv", dtype=str, keep_default_na=False)

# From Excel
df = pd.read_excel("data.xlsx", sheet_name="Sheet1")
```

---

## 4. Exploring Data (first things you do)

```python
df.head(5)          # first 5 rows
df.tail(3)          # last 3 rows
df.shape            # (rows, cols)
df.columns          # column names
df.dtypes           # dtype of each column
df.info()           # column names, non-null counts, dtypes
df.describe()       # count, mean, std, min, 25%, 50%, 75%, max (numeric cols)
df.nunique()        # unique values per column
df.isnull().sum()   # null count per column
df.value_counts("column")  # frequency counts
```

---

## 5. Selection & Indexing

```python
# Column selection
df["name"]              # Series
df[["name", "age"]]     # DataFrame (multiple cols)

# Row selection
df.loc[0]               # row by LABEL (index label)
df.iloc[0]              # row by INTEGER position
df.loc[0:2]             # rows 0,1,2 (inclusive for loc!)
df.iloc[0:2]            # rows 0,1 (exclusive end for iloc)

# Both rows and cols
df.loc[0:2, "name":"age"]           # label-based
df.iloc[0:2, 0:2]                   # position-based

# Boolean filtering (VERY COMMON)
df[df["age"] > 25]                  # rows where age > 25
df[(df["age"] > 25) & (df["salary"] > 55000)]  # multiple conditions (use & | ~, NOT and/or)
df[df["name"].isin(["Alice", "Bob"])]
df[df["name"].str.contains("li")]   # string filter

# .query() — cleaner for complex filters
df.query("age > 25 and salary > 55000")
```

> **Critical:** `loc` = label-based (inclusive end), `iloc` = integer-position (exclusive end).

---

## 6. Adding / Modifying Columns

```python
# New column
df["bonus"] = df["salary"] * 0.1
df["senior"] = df["age"] > 30     # boolean column

# Conditional column
df["level"] = np.where(df["salary"] > 60000, "senior", "junior")

# Apply a function
df["name_upper"] = df["name"].apply(str.upper)
df["tax"] = df["salary"].apply(lambda x: x * 0.3 if x > 55000 else x * 0.2)

# Rename
df.rename(columns={"name": "employee_name"}, inplace=True)

# Drop columns
df.drop(columns=["bonus", "level"], inplace=True)
# or
df = df.drop(columns=["bonus"])
```

---

## 7. Handling Missing Data

```python
df.isnull()                # boolean mask
df.isnull().sum()          # count nulls per column
df.dropna()                # drop rows with ANY null
df.dropna(subset=["age"])  # drop only if "age" is null
df.fillna(0)               # fill all nulls with 0
df["age"].fillna(df["age"].mean(), inplace=True)  # fill with mean
df.ffill()                 # forward fill (last valid value)
df.bfill()                 # backward fill
```

---

## 8. GroupBy (Split-Apply-Combine)

```python
# Basic
df.groupby("department")["salary"].mean()     # avg salary per dept
df.groupby("department")["salary"].agg(["mean", "sum", "count"])

# Multiple grouping columns
df.groupby(["department", "level"])["salary"].mean()

# Multiple aggregations on different columns
df.groupby("department").agg(
    avg_salary=("salary", "mean"),
    max_age=("age", "max"),
    headcount=("name", "count")
)

# Transform (return same-size Series aligned to original)
df["dept_avg"] = df.groupby("department")["salary"].transform("mean")

# Filter groups
df.groupby("department").filter(lambda g: g["salary"].mean() > 50000)
```

### GroupBy flow

```
Original DataFrame
        ↓
    .groupby("col")     → Split into groups
        ↓
    .agg() / .mean()    → Apply function to each group
        ↓
    Result              → Combine back
```

---

## 9. Sorting

```python
df.sort_values("age")                        # ascending
df.sort_values("salary", ascending=False)    # descending
df.sort_values(["department", "salary"], ascending=[True, False])  # multi-col

df.sort_index()   # sort by index
df.nlargest(5, "salary")   # top 5 by salary
df.nsmallest(3, "age")     # bottom 3 by age
```

---

## 10. Merging & Joining

```python
# merge (like SQL JOIN)
pd.merge(df1, df2, on="id")                    # inner join on "id"
pd.merge(df1, df2, on="id", how="left")        # left join
pd.merge(df1, df2, on="id", how="outer")       # full outer join
pd.merge(df1, df2, left_on="emp_id", right_on="id")  # different col names

# concat (stacking)
pd.concat([df1, df2], axis=0)       # stack vertically (append rows)
pd.concat([df1, df2], axis=1)       # stack horizontally (add cols)
pd.concat([df1, df2], ignore_index=True)  # reset index after concat
```

### Merge types cheat sheet

| how | Keeps |
|---|---|
| inner | only matching rows in both |
| left | all from left + matching from right |
| right | all from right + matching from left |
| outer | all rows from both (NaN for missing) |

---

## 11. Pivot & Reshape

```python
# Pivot table (like Excel pivot)
pd.pivot_table(df, values="salary", index="department",
               columns="level", aggfunc="mean")

# Melt (wide → long)
pd.melt(df, id_vars=["name"], value_vars=["q1","q2","q3"],
         var_name="quarter", value_name="sales")

# Crosstab
pd.crosstab(df["department"], df["level"])
```

---

## 12. String Operations (`.str` accessor)

```python
df["name"].str.lower()
df["name"].str.upper()
df["name"].str.strip()
df["name"].str.contains("ali", case=False)   # boolean mask
df["name"].str.replace("old", "new")
df["name"].str.split(" ")
df["name"].str.len()
df["email"].str.extract(r'@(.+)')  # regex capture group
```

---

## 13. DateTime Operations

```python
df["date"] = pd.to_datetime(df["date_str"])    # string → datetime

df["date"].dt.year
df["date"].dt.month
df["date"].dt.day
df["date"].dt.day_name()     # 'Monday', 'Tuesday'...
df["date"].dt.dayofweek      # 0=Monday

# Filter by date
df[df["date"] > "2024-01-01"]
df[df["date"].between("2024-01-01", "2024-06-30")]

# Resample (for time series)
df.set_index("date").resample("M")["sales"].sum()   # monthly sum
```

---

## 14. Apply, Map, Applymap

```python
# .apply() — on Series or DataFrame
df["salary"].apply(lambda x: x * 1.1)          # element-wise on column
df.apply(lambda row: row["salary"] / row["age"], axis=1)  # row-wise

# .map() — on Series only (replace values)
df["level"].map({"junior": 1, "senior": 2})    # dict mapping

# .map() with function
df["salary"].map(lambda x: f"${x:,.0f}")
```

---

## 15. Performance Tips

```python
# 1. Use vectorized operations (NOT .apply() with loops)
df["bonus"] = df["salary"] * 0.1             # FAST (vectorized)
df["bonus"] = df["salary"].apply(lambda x: x*0.1)  # SLOWER

# 2. Use .isin() instead of multiple OR conditions
df[df["city"].isin(["Delhi", "Mumbai", "Pune"])]    # FAST

# 3. Category dtype for repeated strings
df["city"] = df["city"].astype("category")   # saves memory

# 4. Read only needed columns
df = pd.read_csv("big.csv", usecols=["name", "age"])

# 5. Chunked reading for large files
for chunk in pd.read_csv("huge.csv", chunksize=100000):
    process(chunk)
```

---

## 16. Saving Data

```python
df.to_csv("output.csv", index=False)
df.to_excel("output.xlsx", index=False)
df.to_json("output.json", orient="records")
df.to_parquet("output.parquet")    # fast, compressed, typed
```

---

## 17. Common Interview Patterns

### Remove duplicates
```python
df.drop_duplicates()
df.drop_duplicates(subset=["name", "date"], keep="last")
```

### Rank
```python
df["salary_rank"] = df["salary"].rank(ascending=False)
df["rank_in_dept"] = df.groupby("dept")["salary"].rank(ascending=False)
```

### Cumulative operations
```python
df["running_total"] = df["sales"].cumsum()
df["cummax"] = df["price"].cummax()
```

### Shift / lag
```python
df["prev_day_sales"] = df["sales"].shift(1)       # lag by 1
df["next_day"] = df["sales"].shift(-1)            # lead by 1
df["daily_change"] = df["sales"] - df["sales"].shift(1)
df["pct_change"] = df["sales"].pct_change()
```

### Window / Rolling
```python
df["rolling_7d_avg"] = df["sales"].rolling(window=7).mean()
df["expanding_max"] = df["price"].expanding().max()
```

---

## 18. Quick Reference Table

| Task | NumPy | Pandas |
|---|---|---|
| Create | `np.array([1,2,3])` | `pd.Series([1,2,3])` |
| Shape | `a.shape` | `df.shape` |
| Select col | — | `df["col"]` or `df.col` |
| Filter | `a[a > 5]` | `df[df["col"] > 5]` |
| Group | — | `df.groupby("col")` |
| Sort | `np.sort(a)` | `df.sort_values("col")` |
| Null check | `np.isnan(a)` | `df.isnull()` |
| Aggregate | `a.mean()` | `df["col"].mean()` |
| Join | — | `pd.merge(df1, df2)` |
| Read file | `np.loadtxt()` | `pd.read_csv()` |

---

Good luck! For interviews: know GroupBy cold (§8), filtering (§5), merge types (§10), and be able to write a quick `.apply()` or vectorized operation on the spot.
