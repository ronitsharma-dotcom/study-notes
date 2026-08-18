# NumPy & Pandas — The Easy Version (with Examples for Everything)

> Every concept has a runnable example AND its output shown as a comment (`# →`). Read it, picture the output, and you'll remember it. NumPy first (foundation), then Pandas (built on NumPy).

---

# PART 1: NumPy

## 1. What is NumPy and why use it?

**NumPy** = Numerical Python. It gives you a super-fast array object called `ndarray` for doing math on lots of numbers at once.

**Why not just use Python lists?**
- Lists are slow for math (Python loops are slow).
- NumPy does operations in C under the hood → 10-100x faster.
- NumPy lets you do "vectorized" operations — apply math to a whole array in one line, no loops.

```python
import numpy as np

# The problem with lists:
my_list = [1, 2, 3, 4]
# my_list * 2  → [1,2,3,4,1,2,3,4]  (repeats, NOT what we want!)

# NumPy does real math:
arr = np.array([1, 2, 3, 4])
print(arr * 2)
# → [2 4 6 8]   (multiplies each element — this is what we want!)
```

---

## 2. Creating Arrays (many ways)

```python
import numpy as np

# From a Python list
a = np.array([1, 2, 3])
print(a)              # → [1 2 3]

# 2D array (matrix) from list of lists
b = np.array([[1, 2, 3], [4, 5, 6]])
print(b)
# → [[1 2 3]
#    [4 5 6]]

# Array of all zeros (give it the shape)
print(np.zeros((2, 3)))
# → [[0. 0. 0.]
#    [0. 0. 0.]]

# Array of all ones
print(np.ones((2, 2)))
# → [[1. 1.]
#    [1. 1.]]

# Array filled with a specific value
print(np.full((2, 2), 7))
# → [[7 7]
#    [7 7]]

# Range of numbers (like Python's range, but an array)
print(np.arange(0, 10, 2))    # start, stop, step
# → [0 2 4 6 8]

# Evenly spaced numbers between two points
print(np.linspace(0, 1, 5))   # 5 numbers from 0 to 1
# → [0.   0.25 0.5  0.75 1.  ]

# Identity matrix (1s on the diagonal)
print(np.eye(3))
# → [[1. 0. 0.]
#    [0. 1. 0.]
#    [0. 0. 1.]]

# Random numbers
print(np.random.rand(2, 2))       # uniform between 0 and 1
print(np.random.randn(2, 2))      # standard normal (mean 0, std 1)
print(np.random.randint(1, 10, size=(2, 3)))  # random ints 1-9
```

---

## 3. Array Properties (know your array)

```python
a = np.array([[1, 2, 3], [4, 5, 6]])

print(a.shape)     # → (2, 3)   — 2 rows, 3 columns
print(a.ndim)      # → 2        — number of dimensions
print(a.size)      # → 6        — total number of elements
print(a.dtype)     # → int64    — data type of elements
print(len(a))      # → 2        — number of rows (first dimension)
```

**Tip:** `shape` is the MOST used property. `(rows, columns)` for 2D.

---

## 4. Indexing & Slicing (grabbing pieces)

```python
a = np.array([10, 20, 30, 40, 50])

# Single element (indexing starts at 0)
print(a[0])       # → 10
print(a[-1])      # → 50  (last element)

# Slicing [start:stop]  (stop is excluded)
print(a[1:4])     # → [20 30 40]  (index 1, 2, 3)
print(a[:3])      # → [10 20 30]  (first 3)
print(a[2:])      # → [30 40 50]  (from index 2 to end)
print(a[::2])     # → [10 30 50]  (every 2nd element)
```

### 2D indexing — `array[row, column]`

```python
b = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9]])

print(b[0, 0])    # → 1   (row 0, column 0)
print(b[1, 2])    # → 6   (row 1, column 2)
print(b[0])       # → [1 2 3]   (entire row 0)
print(b[:, 1])    # → [2 5 8]   (entire column 1 — ':' means all rows)
print(b[0:2, 1:3])
# → [[2 3]
#    [5 6]]   (rows 0-1, columns 1-2)
```

### Boolean Indexing (VERY common — filtering)

```python
a = np.array([10, 25, 30, 45, 50])

# Create a condition → gives True/False array
print(a > 30)          # → [False False False  True  True]

# Use it to filter (keep only elements where True)
print(a[a > 30])       # → [45 50]

# Multiple conditions (use & for AND, | for OR — with parentheses!)
print(a[(a > 20) & (a < 50)])   # → [25 30 45]
```

> ⚠️ Slicing returns a **view** (a window into the original), not a copy. Changing the slice changes the original. Use `.copy()` if you want an independent copy.

---

## 5. Reshaping (changing the shape)

```python
a = np.arange(12)     # → [0 1 2 3 4 5 6 7 8 9 10 11]

# Reshape to 3 rows x 4 columns
print(a.reshape(3, 4))
# → [[ 0  1  2  3]
#    [ 4  5  6  7]
#    [ 8  9 10 11]]

# Use -1 to let NumPy figure out one dimension
print(a.reshape(2, -1))    # 2 rows, auto columns (6)
# → [[ 0  1  2  3  4  5]
#    [ 6  7  8  9 10 11]]

# Flatten a 2D array back to 1D
b = np.array([[1, 2], [3, 4]])
print(b.flatten())     # → [1 2 3 4]

# Transpose (rows become columns)
print(b.T)
# → [[1 3]
#    [2 4]]
```

---

## 6. Vectorized Operations (math without loops)

This is NumPy's superpower — do math on the whole array at once.

```python
a = np.array([1, 2, 3, 4])
b = np.array([10, 20, 30, 40])

# Element-wise arithmetic (matches position by position)
print(a + b)      # → [11 22 33 44]
print(b - a)      # → [ 9 18 27 36]
print(a * b)      # → [ 10  40  90 160]
print(b / a)      # → [10. 10. 10. 10.]
print(a ** 2)     # → [ 1  4  9 16]

# Operate with a single number (broadcasting)
print(a + 100)    # → [101 102 103 104]
print(a * 10)     # → [10 20 30 40]

# Math functions (applied to every element)
print(np.sqrt(a))     # → [1.   1.41 1.73 2.  ]
print(np.exp(a))      # → [ 2.71  7.38 20.08 54.6 ]
print(np.log(a))      # → [0.   0.69 1.09 1.38]
```

---

## 7. Aggregations (summarizing numbers)

```python
a = np.array([[1, 2, 3],
              [4, 5, 6]])

print(a.sum())        # → 21   (sum of everything)
print(a.mean())       # → 3.5  (average)
print(a.min())        # → 1
print(a.max())        # → 6
print(a.std())        # → 1.7  (standard deviation)
print(a.argmax())     # → 5    (index of max, counting flattened)
print(a.argmin())     # → 0    (index of min)
```

### The `axis` argument (super important, often confusing)

- **axis=0** → go DOWN the columns (collapse rows) → one result per column
- **axis=1** → go ACROSS the rows (collapse columns) → one result per row

```python
a = np.array([[1, 2, 3],
              [4, 5, 6]])

print(a.sum(axis=0))   # → [5 7 9]   (1+4, 2+5, 3+6 — sum each column)
print(a.sum(axis=1))   # → [6 15]    (1+2+3, 4+5+6 — sum each row)
```

**Memory trick:** axis=0 points down ↓ (collapses rows), axis=1 points right → (collapses columns).

---

## 8. Broadcasting (auto-stretching shapes)

Broadcasting lets NumPy do math between arrays of **different shapes** by automatically "stretching" the smaller one.

```python
# A 2x3 matrix + a 1D array of 3 elements
a = np.array([[1, 2, 3],
              [4, 5, 6]])
b = np.array([10, 20, 30])

print(a + b)
# → [[11 22 33]
#    [14 25 36]]
# b was "stretched" to both rows automatically
```

**Rule of thumb:** shapes are compatible if the dimensions are equal OR one of them is 1. The size-1 dimension gets stretched.

---

## 9. Useful NumPy functions

```python
a = np.array([3, 1, 2, 1, 3])

# Sort
print(np.sort(a))            # → [1 1 2 3 3]

# Unique values
print(np.unique(a))          # → [1 2 3]

# where — conditional selection (like if-else)
print(np.where(a > 2, "big", "small"))
# → ['big' 'small' 'small' 'small' 'big']

# Stacking arrays
x = np.array([1, 2])
y = np.array([3, 4])
print(np.vstack([x, y]))     # stack vertically
# → [[1 2]
#    [3 4]]
print(np.hstack([x, y]))     # stack horizontally
# → [1 2 3 4]

# Concatenate
print(np.concatenate([x, y]))   # → [1 2 3 4]

# Copy (independent, not a view)
b = a.copy()
```

---

# PART 2: Pandas

## 1. What is Pandas?

**Pandas** = library for working with **tables of data** (rows and columns), like an Excel sheet or SQL table in Python. Built on top of NumPy.

Two main objects:
- **Series** = a single column (1D)
- **DataFrame** = a full table (2D)

```python
import pandas as pd
```

---

## 2. Series (a single column)

A Series is a 1D labeled array — values + an index (labels).

```python
import pandas as pd

s = pd.Series([10, 20, 30, 40], index=["a", "b", "c", "d"])
print(s)
# → a    10
#   b    20
#   c    30
#   d    40
#   dtype: int64

print(s["b"])         # → 20   (access by label)
print(s[0])           # → 10   (access by position)
print(s.mean())       # → 25.0
print(s[s > 15])      # → b:20, c:30, d:40  (filtering)
```

---

## 3. Creating a DataFrame (the main table)

```python
import pandas as pd

# From a dictionary (keys = column names)
df = pd.DataFrame({
    "name": ["Alice", "Bob", "Charlie", "David"],
    "age": [25, 30, 35, 28],
    "dept": ["AI", "Data", "AI", "HR"],
    "salary": [50000, 60000, 70000, 45000]
})
print(df)
# →      name  age  dept  salary
#   0   Alice   25    AI   50000
#   1     Bob   30  Data   60000
#   2 Charlie   35    AI   70000
#   3   David   28    HR   45000

# From a CSV file (most common in real life)
df = pd.read_csv("data.csv")

# Reading with useful options
df = pd.read_csv("data.csv",
                 dtype=str,               # read everything as string
                 keep_default_na=False)   # don't convert empty to NaN
```

---

## 4. Exploring Data (first things you do)

```python
df.head()          # first 5 rows
df.head(3)         # first 3 rows
df.tail(2)         # last 2 rows

df.shape           # → (4, 4)   — (rows, columns)
df.columns         # → Index(['name', 'age', 'dept', 'salary'])
df.dtypes          # data type of each column
df.info()          # summary: columns, non-null counts, types
df.describe()      # stats (count, mean, std, min, max...) for numeric columns

df["dept"].nunique()          # → 3   (number of unique departments)
df["dept"].unique()           # → ['AI' 'Data' 'HR']
df["dept"].value_counts()     # count of each department
# → AI      2
#   Data    1
#   HR      1

df.isnull().sum()   # count of missing values per column
```

---

## 5. Selecting Columns

```python
# Single column → returns a Series
print(df["name"])
# → 0      Alice
#   1        Bob
#   ...

# Multiple columns → returns a DataFrame (note the double brackets!)
print(df[["name", "salary"]])
# →      name  salary
#   0   Alice   50000
#   1     Bob   60000
#   ...
```

---

## 6. Selecting Rows — `loc` vs `iloc` (very important)

- **`loc`** = select by **label/name** (and end is INCLUDED)
- **`iloc`** = select by **integer position** (and end is EXCLUDED, like Python)

```python
# iloc — by position
print(df.iloc[0])         # first row
print(df.iloc[0:2])       # rows at position 0 and 1 (2 excluded)
print(df.iloc[0, 1])      # row 0, column 1 → age of Alice → 25

# loc — by label
print(df.loc[0])          # row with index label 0
print(df.loc[0:2])        # rows 0, 1, AND 2 (end included!)
print(df.loc[0, "name"])  # → Alice
print(df.loc[:, "name"])  # all rows, just the name column
```

**Remember:** `iloc` = integer/position (excludes end). `loc` = label (includes end).

---

## 7. Filtering Rows (the most-used skill)

```python
# Single condition
print(df[df["age"] > 28])
# → rows where age > 28  (Bob and Charlie)

# Multiple conditions — use & (and), | (or), ~ (not) with parentheses!
print(df[(df["age"] > 25) & (df["dept"] == "AI")])
# → Charlie (age 35, dept AI)

# isin — matches any value in a list
print(df[df["dept"].isin(["AI", "HR"])])
# → Alice, Charlie, David

# String contains
print(df[df["name"].str.contains("a", case=False)])
# → rows where name contains 'a' (case-insensitive)

# .query() — a cleaner way to write conditions
print(df.query("age > 25 and dept == 'AI'"))
```

> ⚠️ In Pandas, use `&` `|` `~` (NOT `and` `or` `not`), and wrap each condition in parentheses.

---

## 8. Adding & Modifying Columns

```python
# New column from a calculation
df["bonus"] = df["salary"] * 0.1
# adds a bonus column: 5000, 6000, 7000, 4500

# New column with a condition
import numpy as np
df["level"] = np.where(df["salary"] > 55000, "senior", "junior")
# → junior, senior, senior, junior

# Apply a function to a column
df["name_upper"] = df["name"].apply(lambda x: x.upper())
# → ALICE, BOB, CHARLIE, DAVID

# Apply with a custom rule
df["tax"] = df["salary"].apply(lambda s: s * 0.3 if s > 55000 else s * 0.2)

# Rename columns
df = df.rename(columns={"dept": "department"})

# Delete columns
df = df.drop(columns=["bonus", "tax"])
```

---

## 9. Handling Missing Data (NaN)

```python
# Assume some cells are missing (NaN)
df.isnull()               # True/False table of missing values
df.isnull().sum()         # count missing per column

# Drop rows with any missing value
df.dropna()

# Drop rows only if a specific column is missing
df.dropna(subset=["age"])

# Fill missing values
df.fillna(0)                             # fill all NaN with 0
df["age"].fillna(df["age"].mean())       # fill age with the average age
df["dept"].fillna("Unknown")             # fill with a label

# Forward fill / backward fill (useful for time series)
df.ffill()    # fill with the value above
df.bfill()    # fill with the value below
```

---

## 10. Sorting

```python
# Sort by one column
df.sort_values("age")                    # ascending
df.sort_values("salary", ascending=False)  # descending (highest first)

# Sort by multiple columns
df.sort_values(["department", "salary"], ascending=[True, False])
# first by department (A-Z), then by salary (high to low) within each

# Get top/bottom N
df.nlargest(2, "salary")     # 2 highest-paid
df.nsmallest(2, "age")       # 2 youngest
```

---

## 11. GroupBy (split → apply → combine) — KEY SKILL

GroupBy = split data into groups, do a calculation on each group, combine results. Like SQL's GROUP BY.

```python
df = pd.DataFrame({
    "dept": ["AI", "Data", "AI", "HR", "AI"],
    "salary": [50000, 60000, 70000, 45000, 55000]
})

# Average salary per department
print(df.groupby("dept")["salary"].mean())
# → dept
#   AI      58333.33
#   Data    60000.00
#   HR      45000.00

# Count how many in each department
print(df.groupby("dept").size())
# → AI      3
#   Data    1
#   HR      1

# Multiple stats at once
print(df.groupby("dept")["salary"].agg(["mean", "max", "count"]))
# →         mean    max  count
#   dept
#   AI    58333   70000     3
#   Data  60000   60000     1
#   HR    45000   45000     1
```

### The flow of GroupBy

```
Original table
      ↓  groupby("dept")
Split into groups: [AI rows] [Data rows] [HR rows]
      ↓  .mean()
Apply to each group
      ↓
Combine into one result table
```

### transform — keep original rows, add group stat to each

```python
# Add each employee's department average next to them
df["dept_avg"] = df.groupby("dept")["salary"].transform("mean")
# every row now has its department's average salary
```

---

## 12. Merging & Joining Tables (like SQL joins)

```python
employees = pd.DataFrame({
    "name": ["Alice", "Bob", "Charlie"],
    "dept_id": [1, 2, 1]
})
departments = pd.DataFrame({
    "dept_id": [1, 2, 3],
    "dept_name": ["AI", "Data", "HR"]
})

# Inner join (only matching rows)
print(pd.merge(employees, departments, on="dept_id"))
# →      name  dept_id dept_name
#   0   Alice        1        AI
#   1 Charlie        1        AI
#   2     Bob        2      Data
# (HR has no employees → excluded in inner join)

# Left join (keep ALL employees, even without a match)
pd.merge(employees, departments, on="dept_id", how="left")

# Different join types
# how="inner"  → only matching (default)
# how="left"   → all from left table
# how="right"  → all from right table
# how="outer"  → everything from both
```

### concat — stack tables (not join)

```python
df1 = pd.DataFrame({"a": [1, 2]})
df2 = pd.DataFrame({"a": [3, 4]})

pd.concat([df1, df2])                    # stack rows (vertically)
# → a: 1, 2, 3, 4

pd.concat([df1, df2], ignore_index=True) # reset the index after stacking
```

---

## 13. String Operations (`.str` accessor)

```python
df = pd.DataFrame({"name": ["  Alice ", "BOB", "charlie"]})

df["name"].str.strip()          # remove spaces → "Alice", "BOB", "charlie"
df["name"].str.lower()          # → "  alice ", "bob", "charlie"
df["name"].str.upper()          # → uppercase
df["name"].str.len()            # length of each string
df["name"].str.contains("li")   # True/False if contains "li"
df["name"].str.replace("a", "@")  # replace characters
df["name"].str.startswith("B")  # True/False
```

---

## 14. DateTime Operations

```python
df = pd.DataFrame({"date": ["2025-01-15", "2025-06-20", "2025-12-31"]})

# Convert string column to real dates
df["date"] = pd.to_datetime(df["date"])

# Extract parts of the date
df["date"].dt.year          # → 2025, 2025, 2025
df["date"].dt.month         # → 1, 6, 12
df["date"].dt.day           # → 15, 20, 31
df["date"].dt.day_name()    # → Wednesday, Friday, Wednesday

# Filter by date
df[df["date"] > "2025-06-01"]
```

---

## 15. Apply, Map (transform values)

```python
df = pd.DataFrame({"salary": [50000, 60000, 70000], "dept": ["AI", "Data", "AI"]})

# apply on a column (element-wise)
df["salary"].apply(lambda x: x * 1.1)
# → 55000, 66000, 77000

# apply on rows (axis=1) — use multiple columns
df.apply(lambda row: row["salary"] * 2 if row["dept"] == "AI" else row["salary"], axis=1)

# map — replace values using a dictionary
df["dept"].map({"AI": 1, "Data": 2})
# → 1, 2, 1
```

---

## 16. Common Interview Patterns (with examples)

```python
# 1. Remove duplicate rows
df.drop_duplicates()
df.drop_duplicates(subset=["name"], keep="first")

# 2. Rank rows
df["salary_rank"] = df["salary"].rank(ascending=False)

# 3. Rank within each group (top earners per department)
df["rank_in_dept"] = df.groupby("dept")["salary"].rank(ascending=False)

# 4. Running (cumulative) total
df["running_total"] = df["salary"].cumsum()

# 5. Difference from the previous row
df["prev_salary"] = df["salary"].shift(1)      # value from row above
df["change"] = df["salary"] - df["salary"].shift(1)

# 6. Percentage change
df["pct_change"] = df["salary"].pct_change()

# 7. Moving average (rolling window)
df["rolling_avg"] = df["salary"].rolling(window=2).mean()
```

---

## 17. Saving Data

```python
df.to_csv("output.csv", index=False)     # index=False → don't save row numbers
df.to_excel("output.xlsx", index=False)
df.to_json("output.json", orient="records")
```

---

## 18. NumPy vs Pandas — Quick Reference

| Task | NumPy | Pandas |
|---|---|---|
| Make it | `np.array([1,2,3])` | `pd.Series([1,2,3])` / `pd.DataFrame({...})` |
| Shape | `a.shape` | `df.shape` |
| Select column | — | `df["col"]` |
| Filter | `a[a > 5]` | `df[df["col"] > 5]` |
| Average | `a.mean()` | `df["col"].mean()` |
| Group | — | `df.groupby("col")` |
| Sort | `np.sort(a)` | `df.sort_values("col")` |
| Missing values | `np.isnan(a)` | `df.isnull()` |
| Join tables | — | `pd.merge(df1, df2)` |
| Read file | `np.loadtxt()` | `pd.read_csv()` |

---

## 19. Quick Q&A for interviews

**Q: Why is NumPy faster than Python lists?**
A: NumPy stores data in contiguous memory and runs operations in C, and it uses vectorization (no Python loops). Lists store pointers to scattered objects and need slow Python loops for math.

**Q: What is broadcasting?**
A: NumPy's ability to do math between arrays of different shapes by automatically stretching the smaller one, as long as dimensions are equal or one is 1.

**Q: What does axis=0 and axis=1 mean?**
A: axis=0 collapses rows (operates down each column). axis=1 collapses columns (operates across each row).

**Q: Difference between loc and iloc?**
A: `loc` selects by label and includes the end. `iloc` selects by integer position and excludes the end.

**Q: What is a Series vs a DataFrame?**
A: A Series is a single labeled column (1D). A DataFrame is a full table of rows and columns (2D), essentially a collection of Series.

**Q: How do you handle missing values in Pandas?**
A: Detect with `isnull()`, remove with `dropna()`, or fill with `fillna()` (using 0, mean, forward-fill, etc.).

**Q: What does groupby do?**
A: Splits the data into groups by a column, applies an aggregation (mean, sum, count) to each group, and combines the results — split-apply-combine.

**Q: Difference between merge and concat?**
A: `merge` joins tables on a common column (like SQL joins). `concat` stacks tables together (rows or columns) without matching keys.

**Q: apply vs map?**
A: `map` works on a Series to replace values (often with a dict). `apply` works on Series or DataFrame and can run a custom function, including across rows with `axis=1`.

**Q: What is a view vs a copy in NumPy?**
A: A slice returns a view (a window into the original) — changing it changes the original. Use `.copy()` for an independent copy.

---

## 20. One-Glance Cheat Sheet

**NumPy**
- Create: `np.array`, `np.zeros`, `np.ones`, `np.arange`, `np.linspace`
- `a.shape`, `a.reshape(r, c)`, `a.T`
- Filter: `a[a > 5]`
- Math: `a + b`, `a * 2`, `np.sqrt(a)` (vectorized, no loops)
- Aggregate: `a.sum()`, `.mean()`, `axis=0` (columns) / `axis=1` (rows)
- Broadcasting stretches smaller arrays automatically

**Pandas**
- `pd.read_csv()`, `df.head()`, `df.info()`, `df.describe()`
- Select: `df["col"]`, `df[["c1","c2"]]`
- Rows: `df.loc[label]` (inclusive), `df.iloc[pos]` (exclusive)
- Filter: `df[(df["a"]>5) & (df["b"]=="x")]`
- `df.groupby("col")["val"].mean()`
- `pd.merge(df1, df2, on="key", how="left")`
- Missing: `df.isnull().sum()`, `df.fillna()`, `df.dropna()`
- Save: `df.to_csv("out.csv", index=False)`

---

Good luck! For interviews, master: filtering, `loc`/`iloc`, `groupby`, `merge`, and handling missing data. These 5 cover 90% of real Pandas work. 🚀
