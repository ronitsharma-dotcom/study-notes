# Probability — The Easy Version (for Data Science / Statistics)

> Friendly notes with real examples. Covers everything from basic probability to distributions. Read top-to-bottom once, then re-read the **bold lines** and formulas before your interview/exam.

---

# Chapter 1: Basic Probability Concepts

### What is Probability?

Probability is simply a **measure of how likely something is to happen**. It's always a number between **0 and 1**.

- **0** = impossible (it will never happen)
- **1** = certain (it will definitely happen)
- **0.5** = 50/50 chance (like a coin flip)

```
Probability = (Number of favorable outcomes) / (Total number of possible outcomes)
```

**Example:** Rolling a die and getting a 4.
- Favorable outcomes = 1 (just the number 4)
- Total outcomes = 6 (1,2,3,4,5,6)
- P(4) = 1/6 ≈ 0.167

### Random Experiment

A **random experiment** is any process whose outcome cannot be predicted with certainty.

**Examples:**
- Tossing a coin (you don't know if it's heads or tails)
- Rolling a die
- Drawing a card from a deck

The keyword is **uncertainty** — you know the possible outcomes, but not which one will happen.

### Sample Space (S)

The **sample space** is the set of **ALL possible outcomes** of a random experiment.

**Examples:**
- Coin toss: S = {Heads, Tails}
- Rolling a die: S = {1, 2, 3, 4, 5, 6}
- Tossing two coins: S = {HH, HT, TH, TT}

Think of it as the "universe" of everything that could possibly happen.

### Event (E)

An **event** is any **subset of the sample space** — one or more outcomes you care about.

**Example:** Rolling a die.
- Event "getting an even number" = {2, 4, 6}
- Event "getting a number greater than 4" = {5, 6}
- Event "getting a 3" = {3}

So an event is just "the outcomes I'm interested in."

### Complement of an Event (E')

The **complement** of an event is **everything that is NOT that event**.

```
P(E') = 1 - P(E)
```

**Example:** Rolling a die.
- Event E = "getting a 6" → P(E) = 1/6
- Complement E' = "NOT getting a 6" = {1,2,3,4,5} → P(E') = 1 - 1/6 = 5/6

**Why it's useful:** Sometimes it's easier to calculate "the opposite" and subtract. E.g., "at least one head in 3 tosses" is easier as `1 - P(no heads at all)`.

---

# Chapter 2: Types of Events

### 1. Independent Events
Two events are **independent** if one happening does NOT affect the other.

**Example:** Tossing a coin twice. The first toss being Heads doesn't change the odds of the second toss. They're independent.

```
P(A and B) = P(A) × P(B)     (only when independent)
```

### 2. Dependent Events
One event **affects** the probability of the other.

**Example:** Drawing 2 cards from a deck WITHOUT replacing the first.
- P(first card is a King) = 4/52
- P(second card is a King) = 3/51 (now only 3 Kings and 51 cards left)
- The first draw changed the second — they're dependent.

### 3. Mutually Exclusive Events (Disjoint)
Two events that **cannot happen at the same time**.

**Example:** Rolling a die — you can't get both a 2 AND a 5 on a single roll.

```
P(A and B) = 0           (they can't both happen)
P(A or B)  = P(A) + P(B)
```

### 4. Non-Mutually Exclusive Events
Events that **CAN happen together**.

**Example:** Drawing a card that is a King OR a Heart. The King of Hearts is both! So they overlap.

```
P(A or B) = P(A) + P(B) - P(A and B)     (subtract the overlap)
```

### 5. Exhaustive Events
Events that together cover the **entire sample space** (at least one of them must happen).

**Example:** Rolling a die — "even" {2,4,6} and "odd" {1,3,5} are exhaustive because every outcome is one or the other.

### Quick summary table

| Type | Meaning | Key formula |
|---|---|---|
| Independent | One doesn't affect the other | P(A∩B) = P(A)×P(B) |
| Dependent | One affects the other | P(A∩B) = P(A)×P(B\|A) |
| Mutually Exclusive | Can't happen together | P(A∩B) = 0 |
| Non-Mutually Exclusive | Can happen together | P(A∪B) = P(A)+P(B)−P(A∩B) |
| Exhaustive | Cover all outcomes | P(A∪B∪...) = 1 |

---

# Chapter 3: Conditional Probability

**Conditional probability** = the probability of an event **given that another event has already happened**.

Written as **P(A | B)** — read as "probability of A given B."

```
P(A | B) = P(A and B) / P(B)
```

**Intuition:** Knowing that B happened **shrinks your sample space** to only the outcomes where B is true. You then ask "within that smaller world, how likely is A?"

**Example:** Rolling a die. What's the probability of getting a 2, GIVEN that the result is even?
- We know the result is even, so our world shrinks to {2, 4, 6}
- Within that, getting a 2 = 1 out of 3
- P(2 | even) = 1/3

**Real-world example:** 
- P(person has disease) = 1%
- But P(person has disease | they tested positive) is much higher — the test result changed our knowledge.

**Key idea:** *"Given" means new information that updates the probability.*

---

# Chapter 4: Bayes' Theorem

Bayes' Theorem lets you **flip a conditional probability around** — find P(A|B) when you know P(B|A).

```
              P(B | A) × P(A)
P(A | B) =  ───────────────────
                   P(B)
```

Where:
- **P(A)** = prior (what you believed before)
- **P(A|B)** = posterior (updated belief after seeing evidence B)
- **P(B|A)** = likelihood
- **P(B)** = evidence (total probability of B)

### The classic medical test example

A disease affects 1% of people. A test is 99% accurate. You test positive. What's the chance you actually have the disease?

Surprisingly, it's NOT 99%! Let's use Bayes:
- P(Disease) = 0.01 (prior)
- P(Positive | Disease) = 0.99 (test correctly detects it)
- P(Positive | No Disease) = 0.01 (false positive rate)

P(Positive) = (0.99 × 0.01) + (0.01 × 0.99) = 0.0099 + 0.0099 = 0.0198

```
P(Disease | Positive) = (0.99 × 0.01) / 0.0198 = 0.0099 / 0.0198 = 0.5 = 50%
```

**The surprising answer: only 50%!** Because the disease is so rare, false positives are as common as true positives. This is why Bayes' Theorem is famous — it corrects our intuition.

**Why it matters in data science:** Bayes' Theorem is the foundation of the **Naive Bayes** classifier (used in spam detection, text classification, etc.).

---

# Chapter 5: PMF vs PDF (Probability Functions)

First, understand two types of data:
- **Discrete** = countable, separate values (number of heads, dice rolls, number of customers)
- **Continuous** = any value in a range (height, weight, temperature, time)

### PMF — Probability Mass Function (for DISCRETE data)

A PMF gives the probability of **each exact value**.

**Example:** Rolling a die.
- P(X=1) = 1/6, P(X=2) = 1/6, ... P(X=6) = 1/6
- Each specific outcome has a real, non-zero probability.

The PMF is like a bar chart — each bar's height = probability of that value. **All bars add up to 1.**

### PDF — Probability Density Function (for CONTINUOUS data)

For continuous data, the probability of any EXACT value is basically **zero** (what's the probability someone is exactly 170.00000... cm? Essentially 0).

So instead, a PDF gives probability over a **range**, using the **area under the curve**.

**Example:** Height of people.
- P(height = exactly 170 cm) ≈ 0
- P(height between 165 and 175 cm) = area under the curve between 165 and 175

The PDF is a smooth curve. **The total area under the curve = 1.**

### Quick comparison

| | PMF | PDF |
|---|---|---|
| Data type | Discrete | Continuous |
| Gives | Probability of exact value | Probability density (area = probability) |
| Shape | Bars | Smooth curve |
| P(X = exact value) | Can be non-zero | Always 0 |
| Total | Sum of all = 1 | Area under curve = 1 |

**CDF (bonus):** Cumulative Distribution Function = probability that X is **less than or equal to** a value. Works for both — it accumulates probability up to a point.

---

# Chapter 6: Bernoulli & Binomial Distributions

### Bernoulli Distribution (a single yes/no trial)

The simplest distribution — **one trial with two outcomes**: success (1) or failure (0).

**Examples:**
- One coin flip (Heads = success)
- One customer either buys or doesn't
- One email is either spam or not

```
P(success) = p
P(failure) = 1 - p
```

**Example:** A biased coin with P(Heads) = 0.7.
- P(X=1) = 0.7 (Heads)
- P(X=0) = 0.3 (Tails)

Mean = p, Variance = p(1−p)

### Binomial Distribution (many Bernoulli trials)

When you repeat a Bernoulli trial **n times** and count the number of successes.

**Conditions for binomial:**
1. Fixed number of trials (n)
2. Each trial is independent
3. Only two outcomes (success/failure)
4. Same probability p every trial

**Examples:**
- Flip a coin 10 times → how many heads?
- Ask 100 people → how many say yes?

```
P(X = k) = C(n, k) × p^k × (1-p)^(n-k)
```
Where C(n,k) = "n choose k" = number of ways to pick k successes from n trials.

**Example:** Flip a fair coin 3 times. Probability of exactly 2 heads?
- n=3, k=2, p=0.5
- C(3,2) = 3 ways (HHT, HTH, THH)
- P(X=2) = 3 × (0.5)² × (0.5)¹ = 3 × 0.125 = 0.375

Mean = n×p, Variance = n×p×(1−p)

**Memory tip:** Bernoulli = 1 trial. Binomial = n Bernoulli trials added up.

---

# Chapter 7: Uniform Distribution

In a **uniform distribution, every outcome is equally likely**. The graph is a flat, straight line (a rectangle).

### Discrete Uniform
**Example:** Rolling a fair die. Each of {1,2,3,4,5,6} has the same probability = 1/6. Flat bars.

### Continuous Uniform
**Example:** A bus arrives sometime between 10:00 and 10:30, equally likely at any moment. Any time in that window is equally probable.

```
For continuous uniform on [a, b]:
PDF = 1 / (b - a)      (constant height)
Mean = (a + b) / 2
```

**Real-world uses:** random number generators, modeling "no preference" scenarios.

**Key visual:** A **flat rectangle** — that's the signature of uniform distribution.

---

# Chapter 8: Normal Distribution (the most important one)

The **Normal Distribution** (also called **Gaussian**) is the famous **bell-shaped curve**. It appears everywhere in nature — heights, weights, exam scores, measurement errors.

### Key properties

1. **Bell-shaped and symmetric** around the mean
2. **Mean = Median = Mode** (all at the center)
3. Defined by two things:
   - **Mean (μ)** — where the center is
   - **Standard Deviation (σ)** — how spread out / wide it is
4. Total area under the curve = 1
5. The tails extend infinitely but never touch the x-axis

**Small σ** = tall, narrow bell (data close to mean)
**Large σ** = short, wide bell (data spread out)

**Why is it so important?** The **Central Limit Theorem** says that if you take enough samples of almost any data and average them, those averages form a normal distribution. That's why the normal distribution shows up everywhere in statistics.

### Z-Distribution (Standard Normal Distribution)

The **Z-distribution** is a special normal distribution with:
- **Mean = 0**
- **Standard Deviation = 1**

It's the "standardized" version of any normal distribution. We convert any normal distribution into the Z-distribution to make calculations and comparisons easy (using Z-tables).

**Z-score** tells you **how many standard deviations a value is away from the mean**:

```
Z = (X - μ) / σ
```

**Example:** Exam scores with mean μ=70, σ=10. You scored 85.
- Z = (85 - 70) / 10 = 1.5
- Your score is **1.5 standard deviations above the mean**.

A Z-score lets you compare values from different distributions (e.g., "was my 85 in math better or worse than my 90 in physics?").

---

# Chapter 9: Standardization vs Normalization

These two are often confused. Both are **feature scaling** techniques used in machine learning to put data on a comparable scale.

### Standardization (Z-score scaling)

Transforms data to have **mean = 0** and **standard deviation = 1**.

```
X_standardized = (X - μ) / σ
```

- Result: values centered around 0, typically between about -3 and +3
- Does NOT bound values to a fixed range
- Best when data follows a normal distribution
- Not affected much by outliers as harshly as normalization

**Use when:** algorithms assume normally distributed data (linear regression, logistic regression, SVM, PCA).

### Normalization (Min-Max scaling)

Rescales data to a fixed range, usually **[0, 1]**.

```
X_normalized = (X - X_min) / (X_max - X_min)
```

- Result: all values squeezed between 0 and 1
- Sensitive to outliers (one extreme value stretches everything)

**Use when:** you need bounded values (neural networks, image pixel values, KNN, distance-based algorithms).

### Quick comparison

| | Standardization | Normalization |
|---|---|---|
| Formula | (X − μ) / σ | (X − min) / (max − min) |
| Output range | Not fixed (mean 0, std 1) | Fixed [0, 1] |
| Affected by outliers | Less | More |
| When to use | Normal-ish data, linear models | Bounded range needed, neural nets, KNN |

**Memory tip:** Standardization → **Std deviation** (mean 0, std 1). Normalization → squeezes into a **fixed range [0,1]**.

---

# Chapter 10: The Empirical Rule (68-95-99.7 Rule)

For a **normal distribution**, the Empirical Rule tells you how much data falls within a certain number of standard deviations from the mean.

```
68%  of data falls within  ±1 standard deviation (μ ± 1σ)
95%  of data falls within  ±2 standard deviations (μ ± 2σ)
99.7% of data falls within ±3 standard deviations (μ ± 3σ)
```

**Visual:**
```
        ┌─── 68% ───┐
    ┌───── 95% ──────┐
┌─────── 99.7% ────────┐
      μ-3σ  μ-σ  μ  μ+σ  μ+3σ
```

**Example:** Exam scores, mean = 70, σ = 10.
- 68% of students scored between 60 and 80 (70 ± 10)
- 95% scored between 50 and 90 (70 ± 20)
- 99.7% scored between 40 and 100 (70 ± 30)

**Why it's useful:**
- **Outlier detection:** Anything beyond ±3σ is very rare (only 0.3% chance) → likely an outlier.
- Quick mental estimates without heavy calculation.

**Memory tip:** **68 → 95 → 99.7** for **1σ → 2σ → 3σ**. Almost all data (99.7%) is within 3 standard deviations.

---

# Chapter 11: Quick Q&A for interviews

**Q: What is probability?**
A: A measure of how likely an event is, between 0 (impossible) and 1 (certain).

**Q: Sample space vs event?**
A: Sample space is ALL possible outcomes; an event is a subset of outcomes you care about.

**Q: What is conditional probability?**
A: The probability of A given that B has already happened: P(A|B) = P(A∩B)/P(B). Knowing B shrinks the sample space.

**Q: What is Bayes' Theorem used for?**
A: To reverse a conditional probability — find P(A|B) from P(B|A). It's the basis of Naive Bayes classifiers and updating beliefs with new evidence.

**Q: PMF vs PDF?**
A: PMF is for discrete data (probability of exact values, bars). PDF is for continuous data (probability = area under a curve; exact value probability is 0).

**Q: Bernoulli vs Binomial?**
A: Bernoulli is a single yes/no trial. Binomial is the number of successes across n independent Bernoulli trials.

**Q: What is a normal distribution?**
A: A symmetric bell-shaped curve where mean = median = mode, defined by mean (center) and standard deviation (spread). Extremely common in nature.

**Q: What is a Z-score?**
A: How many standard deviations a value is from the mean: Z = (X−μ)/σ. It standardizes values for comparison.

**Q: Standardization vs Normalization?**
A: Standardization gives mean 0, std 1 (unbounded). Normalization squeezes data into [0,1]. Standardization for normal-ish data/linear models; normalization for neural nets and distance-based algorithms.

**Q: What is the Empirical Rule?**
A: For normal distributions: 68% of data within 1σ, 95% within 2σ, 99.7% within 3σ of the mean.

**Q: What is the difference between mutually exclusive and independent events?**
A: Mutually exclusive = can't happen together (P(A∩B)=0). Independent = one doesn't affect the other (P(A∩B)=P(A)×P(B)). They are different concepts — mutually exclusive events are actually dependent!

---

# Chapter 12: One-Glance Cheat Sheet

- **Probability** = favorable / total, between 0 and 1
- **Sample space** = all outcomes; **Event** = subset; **Complement** P(E')=1−P(E)
- **Independent:** P(A∩B) = P(A)×P(B)
- **Mutually exclusive:** P(A∩B) = 0, P(A∪B) = P(A)+P(B)
- **Conditional:** P(A|B) = P(A∩B)/P(B)
- **Bayes:** P(A|B) = P(B|A)×P(A) / P(B)
- **PMF** = discrete (bars), **PDF** = continuous (area under curve)
- **Bernoulli** = 1 trial; **Binomial** = n trials, mean = np
- **Uniform** = flat, all equally likely
- **Normal** = bell curve, mean=median=mode, defined by μ and σ
- **Z-score** = (X−μ)/σ = std deviations from mean
- **Standardization** = mean 0, std 1; **Normalization** = range [0,1]
- **Empirical Rule** = 68% (1σ), 95% (2σ), 99.7% (3σ)

---

Good luck! Focus on Conditional Probability, Bayes' Theorem, Normal Distribution, and Standardization vs Normalization — these come up most in data science interviews. 🚀
