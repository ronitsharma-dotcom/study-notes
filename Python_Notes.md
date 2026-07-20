# Python — The Easy Version (Beginner to Advanced)

> Friendly, beginner-first notes. Inspired by CodeWithHarry-style structure: short explanations, small code examples, no jargon overload. Read top-to-bottom once, then re-read the **Quick Q&A** at the end before your interview.

---

# Chapter 1: What is Python?

Python is a **programming language** — a way for you to give instructions to a computer.

**Key facts you should know:**

- Created by **Guido van Rossum** in 1989.
- It is **high-level** (close to English, easy to read).
- It is **interpreted** (runs line-by-line; you don't need to compile it first like C++).
- It is **dynamically typed** (you don't have to tell Python a variable is an integer — it figures it out).
- It is **general-purpose** (web development, AI/ML, automation, data analysis — all possible).
- It is **open-source** and has a huge library ecosystem.

**Why is Python everywhere?**
- Easy to learn (looks like English)
- Massive libraries: NumPy, Pandas, TensorFlow, PyTorch, Django, Flask, FastAPI
- Used by Google, Netflix, Instagram, NASA

```python
# Your first Python program
print("Hello, World!")
```

That's it. One line, done. No `#include`, no `public static void main`. This simplicity is why Python won.

---

# Chapter 2: Variables (boxes that hold data)

A variable is just a **name** that points to some data.

```python
name = "Ronit"
age = 22
height = 5.8
is_student = True
```

You don't need to say "this is an integer" or "this is a string" — Python figures it out by itself.

**Naming rules:**
- Can have letters, digits, and underscores
- Can't start with a digit (`2name` ❌)
- Can't be a Python keyword (`if`, `for`, `class`, etc.)
- Convention: `snake_case` for variables (`my_name`, not `myName`)

**Comments** (notes for humans, ignored by Python):
```python
# This is a single-line comment
"""
This is a 
multi-line comment
"""
```

---

# Chapter 3: Data Types (kinds of data Python knows)

| Type | What it is | Example |
|---|---|---|
| `int` | Whole numbers | `5`, `-100` |
| `float` | Decimal numbers | `3.14`, `-0.5` |
| `str` | Text (string) | `"hello"`, `'Ronit'` |
| `bool` | True/False | `True`, `False` |
| `list` | Ordered collection (changeable) | `[1, 2, 3]` |
| `tuple` | Ordered collection (unchangeable) | `(1, 2, 3)` |
| `dict` | Key-value pairs | `{"name": "Ronit"}` |
| `set` | Unique items, no order | `{1, 2, 3}` |
| `NoneType` | Nothing | `None` |

**Check the type:**
```python
x = 10
print(type(x))   # <class 'int'>
```

**Type conversion (typecasting):**
```python
int("5")        # 5
str(10)         # "10"
float("3.14")   # 3.14
bool(0)         # False (anything 0, "", [], None is False)
bool(1)         # True
```

---

# Chapter 4: Operators (doing things with data)

### Arithmetic
```python
10 + 3     # 13
10 - 3     # 7
10 * 3     # 30
10 / 3     # 3.333... (always float)
10 // 3    # 3 (floor division — drops decimal)
10 % 3     # 1 (remainder)
10 ** 3    # 1000 (power)
```

### Comparison (returns True or False)
```python
5 == 5     # True (equal to)
5 != 4     # True (not equal to)
5 > 3      # True
5 < 3      # False
5 >= 5     # True
```

### Logical
```python
True and False    # False
True or False     # True
not True          # False
```

### Assignment shortcuts
```python
x = 10
x += 5     # same as x = x + 5
x -= 2     # x = x - 2
x *= 3     # x = x * 3
```

---

# Chapter 5: Strings (text)

A string is a sequence of characters. You can use single or double quotes.

```python
s = "Hello"
s = 'Hello'   # same thing
```

**Indexing (starts from 0):**
```python
s = "Python"
s[0]    # 'P'
s[1]    # 'y'
s[-1]   # 'n' (last character)
```

**Slicing (taking a piece):**
```python
s = "Python"
s[0:3]    # 'Pyt' (index 0, 1, 2 — end is exclusive)
s[2:]     # 'thon'
s[:4]     # 'Pyth'
s[::-1]   # 'nohtyP' (reversed)
```

**Common methods:**
```python
s = "  Hello World  "
s.lower()       # "  hello world  "
s.upper()       # "  HELLO WORLD  "
s.strip()       # "Hello World" (removes spaces)
s.replace("World", "Ronit")   # "  Hello Ronit  "
s.split(" ")    # ['', '', 'Hello', 'World', '', '']
len(s)          # 15
"World" in s    # True (check membership)
```

**f-strings (the modern way to format strings):**
```python
name = "Ronit"
age = 22
print(f"My name is {name} and I am {age} years old")
# Output: My name is Ronit and I am 22 years old
```

---

# Chapter 6: Lists (ordered, changeable collection)

A list is like a row of boxes, each holding any kind of value.

```python
fruits = ["apple", "banana", "cherry"]
mixed = [1, "hello", 3.14, True]   # can mix types
```

**Accessing & changing:**
```python
fruits[0]            # 'apple'
fruits[-1]           # 'cherry'
fruits[1] = "mango"  # change banana to mango
```

**Common operations:**
```python
fruits.append("orange")     # add to end
fruits.insert(1, "kiwi")    # add at index 1
fruits.remove("apple")      # remove by value
fruits.pop()                # remove & return last
fruits.pop(0)               # remove & return at index 0

len(fruits)                 # length
"apple" in fruits           # True/False

fruits.sort()               # sort in place
sorted(fruits)              # returns a new sorted list
fruits.reverse()            # reverse in place
```

**Slicing works on lists too:**
```python
nums = [10, 20, 30, 40, 50]
nums[1:4]    # [20, 30, 40]
```

**List comprehension (very Pythonic):**
```python
squares = [x**2 for x in range(5)]
# [0, 1, 4, 9, 16]

evens = [x for x in range(10) if x % 2 == 0]
# [0, 2, 4, 6, 8]
```

---

# Chapter 7: Tuples (ordered, UNchangeable)

A tuple is like a list but **you cannot change it** after creating it.

```python
t = (1, 2, 3)
t[0]          # 1
# t[0] = 5   # ERROR! Tuples are immutable.
```

**When to use tuple?**
- When data should not change (e.g., coordinates, fixed config)
- Slightly faster than lists
- Can be used as dictionary keys (lists can't)

**Tuple unpacking:**
```python
person = ("Ronit", 22, "Delhi")
name, age, city = person
# name = "Ronit", age = 22, city = "Delhi"
```

---

# Chapter 8: Sets (unique items, no order)

A set is a collection with **no duplicates** and **no order**.

```python
s = {1, 2, 3, 3, 2, 1}
print(s)    # {1, 2, 3} — duplicates removed
```

**Operations:**
```python
s.add(4)
s.remove(2)

# Set math
a = {1, 2, 3}
b = {3, 4, 5}
a | b    # union: {1,2,3,4,5}
a & b    # intersection: {3}
a - b    # difference: {1,2}
```

**Use case:** Removing duplicates from a list.
```python
nums = [1, 2, 2, 3, 3, 4]
unique = list(set(nums))   # [1, 2, 3, 4]
```

---

# Chapter 9: Dictionaries (key-value pairs)

A dictionary stores data as **key: value** pairs — like a real dictionary (word: meaning).

```python
person = {
    "name": "Ronit",
    "age": 22,
    "city": "Delhi"
}

person["name"]    # 'Ronit'
person["age"] = 23    # update value
person["email"] = "x@y.com"   # add new key
del person["city"]    # delete key
```

**Useful methods:**
```python
person.keys()      # dict_keys(['name', 'age', 'email'])
person.values()    # dict_values(['Ronit', 23, 'x@y.com'])
person.items()     # dict_items([('name','Ronit'), ('age',23), ...])

person.get("salary", 0)   # 0 if "salary" doesn't exist (safer than person["salary"])
```

**Looping through a dict:**
```python
for key, value in person.items():
    print(key, "->", value)
```

---

# Chapter 10: If / Elif / Else (decisions)

```python
age = 18

if age >= 18:
    print("You can vote")
elif age >= 13:
    print("You're a teenager")
else:
    print("You're a child")
```

**Note:** Python uses **indentation** (4 spaces) to define blocks. No `{}` curly braces.

**Ternary (one-line if-else):**
```python
status = "adult" if age >= 18 else "minor"
```

---

# Chapter 11: Loops (doing things multiple times)

### `for` loop (most common)

```python
for i in range(5):
    print(i)
# 0 1 2 3 4

for fruit in ["apple", "banana", "cherry"]:
    print(fruit)

for char in "Python":
    print(char)
```

**`range()`:**
```python
range(5)          # 0,1,2,3,4
range(2, 10)      # 2,3,4,5,6,7,8,9
range(0, 10, 2)   # 0,2,4,6,8 (step of 2)
```

**`enumerate()`** (when you need index + value):
```python
for i, fruit in enumerate(["a", "b", "c"]):
    print(i, fruit)
# 0 a
# 1 b
# 2 c
```

### `while` loop

```python
count = 0
while count < 5:
    print(count)
    count += 1
```

### `break`, `continue`, `pass`
```python
for i in range(10):
    if i == 5:
        break       # exit the loop
    if i % 2 == 0:
        continue    # skip this iteration
    print(i)

# pass — does nothing, placeholder
def my_function():
    pass
```

---

# Chapter 12: Functions (reusable blocks of code)

```python
def greet(name):
    """This is a docstring (function description)"""
    return f"Hello, {name}!"

print(greet("Ronit"))    # Hello, Ronit!
```

**Default parameters:**
```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

greet("Ronit")              # 'Hello, Ronit!'
greet("Ronit", "Namaste")   # 'Namaste, Ronit!'
```

**Returning multiple values:**
```python
def get_info():
    return "Ronit", 22, "Delhi"

name, age, city = get_info()
```

### `*args` and `**kwargs` (flexible arguments)

```python
def add_all(*args):       # *args = any number of positional arguments (tuple)
    return sum(args)

add_all(1, 2, 3, 4)        # 10

def print_info(**kwargs):  # **kwargs = any number of keyword arguments (dict)
    for k, v in kwargs.items():
        print(k, "->", v)

print_info(name="Ronit", age=22)
```

### Lambda (anonymous one-line functions)

```python
square = lambda x: x * x
square(5)    # 25

# Useful with sort, map, filter
nums = [3, 1, 4, 1, 5, 9]
sorted_nums = sorted(nums, key=lambda x: -x)   # sort descending
```

---

# Chapter 13: Modules (using other people's code)

A module is a Python file with code you can reuse. Python comes with hundreds.

```python
import math

math.sqrt(16)    # 4.0
math.pi          # 3.14159...

# Import specific things
from math import sqrt, pi
sqrt(16)         # 4.0

# Rename (alias)
import numpy as np
import pandas as pd
```

**Installing external modules with pip:**
```bash
pip install requests
pip install pandas numpy
```

Then in your code:
```python
import requests
r = requests.get("https://example.com")
print(r.text)
```

---

# Chapter 14: Object-Oriented Programming (OOP)

OOP is a way to organize code using **classes** (blueprints) and **objects** (instances of those blueprints).

### Class & Object — basic example

```python
class Dog:
    # Constructor — runs when object is created
    def __init__(self, name, age):
        self.name = name      # 'self' refers to this specific dog
        self.age = age

    def bark(self):
        return f"{self.name} says Woof!"

# Create objects (instances)
rex = Dog("Rex", 3)
buddy = Dog("Buddy", 5)

print(rex.bark())     # "Rex says Woof!"
print(buddy.name)     # "Buddy"
```

### The 4 Pillars of OOP (THIS IS ASKED IN EVERY INTERVIEW)

**1. Encapsulation** — bundling data and methods together; hiding internal details.
```python
class BankAccount:
    def __init__(self, balance):
        self._balance = balance     # _balance is "private" by convention

    def deposit(self, amount):
        self._balance += amount

    def get_balance(self):
        return self._balance
```

**2. Inheritance** — a child class inherits from a parent class.
```python
class Animal:
    def __init__(self, name):
        self.name = name
    def speak(self):
        print("Generic animal sound")

class Dog(Animal):     # Dog inherits from Animal
    def speak(self):
        print(f"{self.name} says Woof!")

rex = Dog("Rex")
rex.speak()    # Rex says Woof!
```

**3. Polymorphism** — same method name, different behavior.
```python
class Dog(Animal):
    def speak(self):
        print("Woof!")

class Cat(Animal):
    def speak(self):
        print("Meow!")

for animal in [Dog("Rex"), Cat("Kitty")]:
    animal.speak()   # Each one behaves differently
```

**4. Abstraction** — hiding complexity, showing only the necessary parts.
```python
# You use math.sqrt(16) — you don't care HOW it computes the square root.
# That's abstraction.
```

### Common dunder ("magic") methods

```python
class Book:
    def __init__(self, title, pages):
        self.title = title
        self.pages = pages

    def __str__(self):     # what print(book) shows
        return f"Book: {self.title}"

    def __len__(self):     # what len(book) returns
        return self.pages

b = Book("Python", 350)
print(b)        # Book: Python
print(len(b))   # 350
```

### `@property` (getters made elegant)

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def area(self):
        return 3.14 * self._radius ** 2

c = Circle(5)
print(c.area)    # 78.5 — accessed like an attribute, computed like a method
```

---

# Chapter 15: File Handling

```python
# Writing to a file
with open("notes.txt", "w") as f:
    f.write("Hello, World!\n")
    f.write("Second line")

# Reading from a file
with open("notes.txt", "r") as f:
    content = f.read()
    print(content)

# Reading line by line
with open("notes.txt", "r") as f:
    for line in f:
        print(line.strip())

# Modes:
# "r" = read (default)
# "w" = write (overwrites existing!)
# "a" = append (adds to end)
# "r+" = read and write
```

**Why `with`?** It automatically closes the file even if there's an error. Always use `with`.

**Working with JSON:**
```python
import json

# Python dict to JSON file
data = {"name": "Ronit", "age": 22}
with open("data.json", "w") as f:
    json.dump(data, f, indent=2)

# JSON file to Python dict
with open("data.json", "r") as f:
    loaded = json.load(f)
```

---

# Chapter 16: Error Handling (try / except)

```python
try:
    x = 10 / 0
except ZeroDivisionError:
    print("Can't divide by zero!")
except ValueError as e:
    print(f"Value error: {e}")
except Exception as e:    # catch-all (use last)
    print(f"Some other error: {e}")
else:
    print("No error happened")    # runs if no exception
finally:
    print("This always runs")     # cleanup, runs no matter what
```

**Raising your own errors:**
```python
def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero!")
    return a / b
```

---

# Chapter 17: Advanced Topics (good to know for interviews)

### Generators (lazy iterators — memory efficient)

```python
def count_up_to(n):
    i = 1
    while i <= n:
        yield i      # 'yield' makes this a generator
        i += 1

for num in count_up_to(5):
    print(num)
# 1, 2, 3, 4, 5

# Generators produce values one at a time
# Great for huge data sets — saves memory
```

**Generator expression:** like list comprehension but with `()` instead of `[]`:
```python
big_sum = sum(x**2 for x in range(1000000))   # doesn't create a list, just iterates
```

### Decorators (functions that wrap other functions)

```python
def my_decorator(func):
    def wrapper():
        print("Before function")
        func()
        print("After function")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()
# Output:
# Before function
# Hello!
# After function
```

**Real-world use:** `@property`, `@staticmethod`, `@classmethod`, `@app.route()` in Flask, `@app.get()` in FastAPI — these are all decorators.

### List vs Tuple vs Set vs Dict (one-line each)

| Type | Ordered? | Changeable? | Duplicates? | Syntax |
|---|---|---|---|---|
| **List** | Yes | Yes | Yes | `[1, 2, 3]` |
| **Tuple** | Yes | No | Yes | `(1, 2, 3)` |
| **Set** | No | Yes | No | `{1, 2, 3}` |
| **Dict** | Yes (3.7+) | Yes | Keys: no | `{"a": 1}` |

### `is` vs `==`

```python
a = [1, 2, 3]
b = [1, 2, 3]
a == b     # True (same value)
a is b     # False (different objects in memory)
```
- `==` checks **value**
- `is` checks **identity** (same object)

### Shallow vs Deep Copy

```python
import copy

original = [[1, 2], [3, 4]]
shallow = copy.copy(original)       # new outer list, INSIDE lists are shared
deep = copy.deepcopy(original)      # fully independent copy
```

### `*args` vs `**kwargs` (recap)

```python
def func(*args, **kwargs):
    print(args)    # tuple of positional args
    print(kwargs)  # dict of keyword args

func(1, 2, 3, name="Ronit", age=22)
# args = (1, 2, 3)
# kwargs = {'name': 'Ronit', 'age': 22}
```

### Mutable default argument trap (interview favorite)

```python
# WRONG
def add_item(item, lst=[]):    # default list is SHARED across calls!
    lst.append(item)
    return lst

# RIGHT
def add_item(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst
```

### GIL (Global Interpreter Lock) — quick interview answer

> "Python has the GIL, which lets only one thread run Python code at a time. So threading doesn't give true parallelism for CPU-heavy work — for that we use multiprocessing. Threading is still useful for I/O-bound work like network calls or file reading."

### List comprehension vs Generator expression

```python
# List comprehension — creates a list in memory
squares_list = [x**2 for x in range(10)]

# Generator expression — lazy, memory-efficient
squares_gen = (x**2 for x in range(10))
```

---

# Chapter 18: Useful Built-in Functions

```python
len([1,2,3])          # 3
max([1,5,3])          # 5
min([1,5,3])          # 1
sum([1,2,3])          # 6
sorted([3,1,2])       # [1,2,3]
reversed([1,2,3])     # iterator: 3,2,1
abs(-5)               # 5
round(3.7)            # 4
type(x)               # type of x
isinstance(x, int)    # True/False
```

**Iteration helpers:**
```python
zip([1,2,3], ["a","b","c"])     # pairs: (1,'a'), (2,'b'), (3,'c')
map(str, [1,2,3])               # apply str to each: '1', '2', '3'
filter(lambda x: x>2, [1,2,3,4])  # keep elements where condition is True
any([False, True, False])       # True if at least one is True
all([True, True, True])         # True only if all are True
```

---

# Chapter 19: Quick Q&A for the interview

**Q: What is Python?**
A: A high-level, interpreted, dynamically-typed programming language created by Guido van Rossum. Popular for web dev, data science, AI/ML, and automation.

**Q: What are the data types in Python?**
A: `int`, `float`, `str`, `bool`, `list`, `tuple`, `set`, `dict`, `None`.

**Q: List vs Tuple?**
A: Lists are mutable (changeable) and use `[]`. Tuples are immutable and use `()`. Tuples are slightly faster and can be used as dictionary keys.

**Q: What is a dictionary?**
A: A collection of key-value pairs. Fast lookup by key. Created with `{}`.

**Q: What is the difference between `is` and `==`?**
A: `==` checks if values are equal. `is` checks if two variables point to the same object in memory.

**Q: What are `*args` and `**kwargs`?**
A: `*args` collects extra positional arguments into a tuple. `**kwargs` collects extra keyword arguments into a dictionary. Used when you don't know how many arguments a function will receive.

**Q: What is a lambda function?**
A: A small anonymous function on one line, like `lambda x: x + 1`. Useful for short throw-away functions, especially with `sorted`, `map`, `filter`.

**Q: What is a generator?**
A: A function that uses `yield` instead of `return`. It produces values one at a time and doesn't store them all in memory — great for large data.

**Q: What are decorators?**
A: Functions that wrap other functions to add behavior without modifying them. Used in Flask routes, `@property`, etc.

**Q: What are the 4 pillars of OOP?**
A: Encapsulation (bundling data + methods), Inheritance (child inherits from parent), Polymorphism (same method, different behavior), Abstraction (hiding complexity).

**Q: What is `self` in Python classes?**
A: `self` refers to the current instance of the class. It's how methods know which object they're operating on. Always the first parameter of an instance method.

**Q: What is the difference between `__init__` and a regular method?**
A: `__init__` is the constructor — it runs automatically when an object is created. Regular methods only run when you call them.

**Q: What is GIL?**
A: The Global Interpreter Lock — lets only one thread run Python code at a time. So threading helps with I/O-bound work, but for CPU-bound work, use multiprocessing.

**Q: What is the difference between shallow copy and deep copy?**
A: Shallow copy copies the outer object but shares the inner objects. Deep copy makes a fully independent copy at every level.

**Q: How do you handle errors?**
A: Using `try/except/finally`. Try the risky code; if an exception is raised, except handles it; finally always runs (for cleanup).

**Q: How is memory managed in Python?**
A: Python has automatic garbage collection. It tracks how many references point to each object; when references hit zero, the object is freed.

**Q: List comprehension vs Generator expression?**
A: List comprehension creates a full list in memory (`[x for x in range(10)]`). Generator expression is lazy and produces values one at a time (`(x for x in range(10))`). Use generators for large data.

**Q: What is `with` used for?**
A: It's a context manager. It automatically handles setup and cleanup — most commonly used for opening files, so the file is properly closed even if an error happens.

**Q: What is `pass` in Python?**
A: A placeholder that does nothing. Used when syntax requires a block but you have nothing to write yet.

**Q: Mutable vs Immutable types?**
A: Mutable can be changed after creation (list, dict, set). Immutable can't (int, float, str, tuple).

---

# Chapter 20: One-glance cheat sheet (read this last)

- Python = simple, interpreted, dynamic-typed, beginner-friendly
- 4 main collections: **list** `[]`, **tuple** `()`, **dict** `{}`, **set** `{}`
- Use **f-strings** for formatting: `f"Hello {name}"`
- Use **list comprehension** for clean loops: `[x*2 for x in lst]`
- Use **`with open(...) as f:`** for files
- **4 pillars of OOP**: Encapsulation, Inheritance, Polymorphism, Abstraction
- **`__init__`** is the constructor; **`self`** refers to the instance
- **Decorators** wrap functions to extend behavior
- **Generators** (`yield`) save memory for large data
- **`*args` / `**kwargs`** for flexible function arguments
- **`try/except/finally`** for error handling
- **GIL** = only one thread runs Python at a time → use multiprocessing for CPU work
- **`==` checks value**, **`is` checks identity**

---

Good luck! You don't need to memorize every line — just understand the chapters and the Q&A. Re-read Chapter 14 (OOP) and Chapter 19 (Q&A) one more time tomorrow morning. 🚀
