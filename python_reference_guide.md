# Python Reference Guide

**Document Type:** Developer Reference  
**Language:** Python 3.8+  
**Skill Level:** Beginner to Intermediate  
**Last Updated:** 2026

---

## How to Use This Guide

This guide is split into two parts:

- **Quick Reference** — tables and one-liners for when you know what you need and just want the syntax
- **Detailed Reference** — annotated examples with explanations for when you need context

Use the section links to jump directly to what you need.

---

## Quick Reference Tables

### Types

| Type | Example | Description |
|---|---|---|
| `int` | `42` | Whole numbers |
| `float` | `3.14` | Decimal numbers |
| `bool` | `True` / `False` | Boolean values |
| `str` | `"hello"` | Text |
| `None` | `None` | Represents no value |

### Type Conversion

| Expression | Result |
|---|---|
| `int("42")` | `42` |
| `float("3.14")` | `3.14` |
| `str(123)` | `"123"` |
| `bool(0)` | `False` |
| `bool("text")` | `True` |
| `list("abc")` | `['a', 'b', 'c']` |
| `set([1, 1, 2])` | `{1, 2}` |

### Math Operators

| Operator | Description | Example | Result |
|---|---|---|---|
| `+` | Addition | `5 + 2` | `7` |
| `-` | Subtraction | `5 - 2` | `3` |
| `*` | Multiplication | `5 * 2` | `10` |
| `/` | Float division | `5 / 2` | `2.5` |
| `//` | Integer division | `5 // 2` | `2` |
| `%` | Remainder | `5 % 2` | `1` |
| `**` | Exponent | `2 ** 3` | `8` |

### Comparison Operators

| Operator | Meaning |
|---|---|
| `==` | Equal |
| `!=` | Not equal |
| `<` | Less than |
| `<=` | Less than or equal |
| `>` | Greater than |
| `>=` | Greater than or equal |
| `and` | Both conditions must be True |
| `or` | Either condition must be True |
| `not` | Reverses boolean result |

### Common String Methods

| Method | Description | Example | Result |
|---|---|---|---|
| `.strip()` | Remove whitespace from both ends | `" hi ".strip()` | `"hi"` |
| `.lower()` | Lowercase | `"HI".lower()` | `"hi"` |
| `.upper()` | Uppercase | `"hi".upper()` | `"HI"` |
| `.replace(a, b)` | Replace all matches | `"cat".replace("c","b")` | `"bat"` |
| `.split()` | Split on whitespace | `"a b".split()` | `['a', 'b']` |
| `.join()` | Join list into string | `",".join(["a","b"])` | `"a,b"` |
| `.startswith()` | Check start | `"hi".startswith("h")` | `True` |
| `.endswith()` | Check end | `"hi".endswith("i")` | `True` |
| `.find()` | Find index or -1 | `"hi".find("h")` | `0` |
| `.count()` | Count matches | `"banana".count("a")` | `3` |
| `len()` | Length | `len("hello")` | `5` |

### Common List Methods

| Method | Description |
|---|---|
| `.append(x)` | Add one item to end |
| `.extend([x, y])` | Add multiple items to end |
| `.insert(i, x)` | Insert at index |
| `.remove(x)` | Remove first matching value |
| `.pop()` | Remove and return last item |
| `.sort()` | Sort in place |
| `.reverse()` | Reverse in place |
| `len(lst)` | Number of items |
| `sorted(lst)` | Return new sorted list |

### Built-in Functions

| Function | Description |
|---|---|
| `len(x)` | Number of items |
| `range(n)` | Numbers 0 through n-1 |
| `enumerate(x)` | Index + item while looping |
| `zip(a, b)` | Pair items from two iterables |
| `sum(x)` | Add all numbers |
| `min(x)` | Smallest value |
| `max(x)` | Largest value |
| `abs(x)` | Absolute value |
| `round(x, n)` | Round to n decimal places |
| `sorted(x)` | Return new sorted list |
| `reversed(x)` | Return reversed iterator |
| `all(x)` | True if all items are truthy |
| `any(x)` | True if any item is truthy |
| `type(x)` | Return exact type |
| `isinstance(x, t)` | Check if x is of type t |

### Truthiness

| Value | Truthy? |
|---|---|
| `0`, `0.0` | False |
| `""` | False |
| `[]`, `{}`, `set()` | False |
| `None` | False |
| `False` | False |
| Everything else | True |

---

## Detailed Reference

---

### Types and Type Conversion

```python
# Basic types
x_int   = 42          # int   — whole numbers
x_float = 3.14        # float — decimal numbers
x_bool  = True        # bool  — True or False
x_str   = "hello"     # str   — text
x_none  = None        # None  — represents no value

# Check types
type(x_int)                    # <class 'int'>
isinstance(x_int, int)         # True
isinstance(x_int, (int, float))# True — check multiple types at once

# Convert between types
int("42")                      # 42
float("3.14")                  # 3.14
str(123)                       # "123"
bool(0)                        # False — 0 is falsy
bool("text")                   # True  — non-empty string is truthy
list("abc")                    # ['a', 'b', 'c']
set([1, 1, 2])                 # {1, 2} — duplicates removed
```

---

### Strings

```python
s = "Hello"

# Indexing and slicing
s[0]       # 'H'     — first character
s[-1]      # 'o'     — last character
s[1:4]     # 'ell'   — index 1 up to 4, not including 4
s[::-1]    # 'olleH' — reversed string

# f-strings — insert variables directly into strings
name = "Ada"
print(f"Hello, {name}")          # Hello, Ada
print(f"{3.14159:.2f}")          # 3.14  — 2 decimal places
print(f"{7:03}")                 # 007   — pad to width 3 with zeros

# Multi-line strings
multi = """Line 1
Line 2"""

# Common methods
s.strip()                        # remove whitespace from both ends
s.lower()                        # lowercase
s.upper()                        # uppercase
s.replace("Hello", "Hi")        # replace all matches
s.split()                        # split on whitespace → list
",".join(["a", "b", "c"])       # "a,b,c"
s.startswith("He")               # True
s.endswith("lo")                 # True
s.find("ll")                     # 2  — index or -1 if not found
s.count("l")                     # 2  — number of matches

# Check string contents
s.isalpha()    # True if letters only
s.isdigit()    # True if digits only
s.isspace()    # True if whitespace only

# Padding and alignment
"42".zfill(5)      # '00042' — pad with zeros on left
s.center(10)       # center text in width 10
s.ljust(10)        # left-align, spaces on right
s.rjust(10)        # right-align, spaces on left

# Strings are immutable — you cannot change characters directly
s = "Hello"
# s[0] = "Y"          # Error
s = "Y" + s[1:]       # "Yello" — create a new string instead
```

---

### Variables and Assignment

```python
x = 10
y = 20

a = b = c = 0          # assign same value to multiple names
x, y = 1, 2            # unpack values
x, y = y, x            # swap values — no temp variable needed
```

---

### Conditionals

```python
age = 18

if age < 13:
    category = "child"
elif age < 18:
    category = "teen"
else:
    category = "adult"

# Ternary — inline if/else
status = "adult" if age >= 18 else "minor"
```

---

### Lists

```python
nums = [10, 20, 30, 40, 50]

# Access
nums[0]        # 10  — first item
nums[-1]       # 50  — last item
nums[1:4]      # [20, 30, 40] — slice
nums[::2]      # [10, 30, 50] — every second item
nums[::-1]     # reversed copy

# Modify
nums.append(60)            # add one item to end
nums.extend([70, 80])      # add multiple items to end
nums.insert(1, 15)         # insert 15 at index 1
nums.remove(30)            # remove first matching value
last = nums.pop()          # remove and return last item
nums.pop(0)                # remove and return item at index 0
nums[0] = 99               # replace item at index 0
del nums[1]                # delete item at index 1
nums.sort()                # sort in place
nums.reverse()             # reverse in place

# Useful patterns
len(nums)                  # number of items
sum(nums)                  # total
min(nums)                  # smallest
max(nums)                  # largest
sorted(nums)               # new sorted list — original unchanged

# Unpacking
first, *middle, last = [1, 2, 3, 4]
# first=1, middle=[2, 3], last=4
```

---

### Tuples

```python
# Ordered, immutable — cannot be changed after creation
point = (10, 20)
single = (5,)           # one-item tuple requires a trailing comma

# Unpack values
x, y = point            # x=10, y=20
```

---

### Dictionaries

```python
person = {"name": "Ada", "age": 30}

# Access
person["name"]                              # "Ada"
person.get("city")                          # None — safe, no error
person.get("city", "N/A")                  # "N/A" — default if missing

# Modify
person["job"] = "Engineer"                  # add new key
person.update({"age": 31, "city": "Paris"}) # update multiple
del person["age"]                           # delete key

# Views
person.keys()     # all keys
person.values()   # all values
person.items()    # key/value pairs

# Loop through key/value pairs
for key, value in person.items():
    print(key, value)

# Counting pattern — count word frequency
sentence = input("Enter a sentence: ")
counts = {}

for word in sentence.split():
    counts[word] = counts.get(word, 0) + 1

print(counts)
```

---

### Sets

```python
a = {1, 2, 3}
b = {3, 4, 5}

a.add(6)         # add item
a.remove(2)      # remove — error if missing
a.discard(99)    # remove — no error if missing

a | b            # union        — all unique values from both
a & b            # intersection — shared values only
a - b            # difference   — in a but not b
a ^ b            # symmetric    — not shared by either
```

---

### Loops

```python
# for loop
for n in [1, 2, 3]:
    print(n)

# range
for i in range(5):          # 0, 1, 2, 3, 4
    print(i)

for i in range(2, 10, 2):  # 2, 4, 6, 8
    print(i)

# while loop
count = 0
while count < 3:
    print(count)
    count += 1              # always increment to avoid infinite loop

# Loop control
for n in range(10):
    if n == 3:
        continue            # skip this iteration
    if n == 7:
        break               # stop the loop entirely

# loop else — runs only if loop completed without break
for n in range(3):
    print(n)
else:
    print("finished normally")
```

---

### Iteration Patterns

```python
items = ["a", "b", "c"]

# Loop over values
for item in items:
    print(item)

# Loop over index + value
for i, item in enumerate(items):
    print(i, item)          # 0 a, 1 b, 2 c

# Loop over paired values from two lists
names  = ["Ada", "Linus"]
scores = [98, 95]

for name, score in zip(names, scores):
    print(name, score)

# Membership check
if "a" in items:
    print("found")
```

---

### Comprehensions

```python
# Syntax: [EXPRESSION for ITEM in ITERABLE if CONDITION]

# Basic list comprehension
squares = [x * x for x in range(5)]          # [0, 1, 4, 9, 16]

# With filter
evens = [x for x in range(10) if x % 2 == 0] # [0, 2, 4, 6, 8]

# Transform + filter
doubled_positives = [x * 2 for x in nums if x > 0]

# Dictionary comprehension
word_lengths = {word: len(word) for word in ["cat", "horse"]}
# {'cat': 3, 'horse': 5}

# Set comprehension
unique_lengths = {len(word) for word in ["cat", "dog", "horse"]}
# {3, 5}

# Mental model — this comprehension:
result = [item for item in nums if item > 0]

# Is equivalent to this loop:
result = []
for item in nums:
    if item > 0:
        result.append(item)
```

---

### Functions

```python
# Basic function
def add(a, b):
    return a + b

result = add(2, 3)          # 5

# Default arguments
def greet(name="world"):
    return f"Hello, {name}"

greet()                     # "Hello, world"
greet("Ada")                # "Hello, Ada"

# Type hints — optional but recommended
def multiply(a: int, b: int) -> int:
    return a * b

# Docstring — explains what the function does
def square(x: int) -> int:
    """Return the square of x."""
    return x * x

# Variable arguments
def demo(*args, **kwargs):
    print(args)             # extra positional args as tuple
    print(kwargs)           # extra keyword args as dict
```

---

### Sorting

```python
nums  = [3, 1, 2]
words = ["apple", "banana", "kiwi"]

sorted(nums)                            # [1, 2, 3] — new list
sorted(words, key=len)                  # sort by length
sorted(words, key=len, reverse=True)    # sort by length descending
sorted(words, key=str.lower)            # case-insensitive sort

# Sort list of dictionaries
people = [{"name": "Ada", "age": 30}, {"name": "Bob", "age": 25}]
sorted(people, key=lambda p: p["age"])  # sort by age
```

---

### File Handling

```python
# Read entire file
with open("data.txt", "r", encoding="utf-8") as f:
    text = f.read()

# Write to file (overwrites existing content)
with open("data.txt", "w", encoding="utf-8") as f:
    f.write("Hello\n")

# Append to file
with open("data.txt", "a", encoding="utf-8") as f:
    f.write("More text\n")

# Read line by line
with open("data.txt", "r", encoding="utf-8") as f:
    for line in f:
        print(line.strip())     # strip removes trailing newline

# Open safely with error handling
filename = input("Enter a filename: ")

try:
    with open(filename, "r", encoding="utf-8") as f:
        print(f.read())
except FileNotFoundError:
    print("That file does not exist")
```

| Mode | Behavior |
|---|---|
| `'r'` | Read existing file |
| `'w'` | Write — overwrites file |
| `'a'` | Append to end of file |
| `'rb'` | Read binary |
| `'r+'` | Read and write |

---

### Exception Handling

```python
try:
    value = int("123")
except ValueError:
    print("Bad number")       # runs if ValueError occurs
else:
    print("Success")          # runs if no exception occurred
finally:
    print("Always runs")      # runs regardless

# Raise your own errors
def divide(a, b):
    if b == 0:
        raise ValueError("b cannot be zero")
    return a / b
```

| Error Type | Cause |
|---|---|
| `ImportError` | Import failed |
| `IndexError` | Index out of range |
| `NameError` | Unknown variable name |
| `SyntaxError` | Code cannot be parsed |
| `TypeError` | Wrong type used |
| `ValueError` | Right type, bad value |
| `FileNotFoundError` | File does not exist |

---

### Classes

```python
class Car:
    def __init__(self, color, doors):
        self.color = color      # instance attribute
        self.doors = doors

    def drive(self):
        return "I'm driving!"

    def describe(self):
        return f"{self.color} car with {self.doors} doors"

car = Car("red", 4)
print(car.color)                # "red"
print(car.drive())              # "I'm driving!"
```

---

### Python Mental Model

These are the concepts that explain why Python sometimes behaves unexpectedly.

```python
# VARIABLES ARE NAMES — not containers
a = [1, 2, 3]
b = a               # b points to the same list object

b.append(4)
print(a)            # [1, 2, 3, 4] — a is affected too

# EQUALITY VS IDENTITY
a = [1, 2, 3]
b = [1, 2, 3]

a == b              # True  — same values
a is b              # False — different objects in memory

# Always use `is` for None checks
if value is None:
    pass            # correct
if value == None:
    pass            # works, but not best practice

# MUTATION VS REBINDING
lst = [1, 2]
lst.append(3)       # mutation  — same object, contents changed
lst = lst + [4]     # rebinding — new object, variable reassigned

# SCOPE — Python searches in this order: Local → Enclosing → Global → Built-in
x = 10

def change():
    global x        # explicitly use the module-level variable
    x = 20

# DEFAULT MUTABLE ARGUMENT GOTCHA
def bad(item, items=[]):     # default list persists between calls
    items.append(item)
    return items

def good(item, items=None):  # correct approach
    if items is None:
        items = []
    items.append(item)
    return items

# CLOSURES — inner functions remember variables from their enclosing scope
def outer():
    x = 10
    def inner():
        return x    # inner remembers x even after outer() finishes
    return inner
```

---

### Common Patterns

```python
# Count character frequency
counts = {}
for ch in "banana":
    counts[ch] = counts.get(ch, 0) + 1
# {'b': 1, 'a': 3, 'n': 2}

# Filter a list
nums  = [1, 2, 3, 4, 5]
evens = [n for n in nums if n % 2 == 0]

# Read file lines into a clean list
with open("log.txt", "r", encoding="utf-8") as f:
    lines = [line.strip() for line in f]

# Check file extension
name = "report.pdf"
if name.endswith(".pdf"):
    print("PDF file")

# Categorize files by extension
files = ["report.pdf", "notes.txt", "image.png"]

categories = {
    ".pdf": "Documents",
    ".txt": "Documents",
    ".png": "Images"
}

sorted_files = {"Documents": [], "Images": [], "Unknown": []}

for file in files:
    ext      = "." + file.split(".")[-1]
    category = categories.get(ext, "Unknown")
    sorted_files[category].append(file)

# Track highest value seen
highest = None
for i in [3, 7, 2, 9, 1]:
    if highest is None or i > highest:
        highest = i
```

---

### Script Entry Point

```python
def main():
    print("Program running")

if __name__ == "__main__":
    main()          # only runs when this file is executed directly
                    # does NOT run when the file is imported
```

---

### Modern Python Features

```python
# Walrus operator := (Python 3.8+)
# Assigns a value inside an expression — useful to avoid repeating work

# Without walrus
line = input("Enter text: ")
if line:
    print(line)

# With walrus
if line := input("Enter text: "):
    print(line)

# Useful in while loops
while line := input("Command: "):
    print(f"You typed: {line}")

# match-case (Python 3.10+) — structured pattern matching
def describe_type(value):
    match value:
        case int():
            return "integer"
        case str():
            return "string"
        case list():
            return "list"
        case _:
            return "unknown"

# Dict dispatch — alternative to match-case for simple lookups
def month(i):
    months = {
        1: "January",  2: "February", 3: "March",
        4: "April",    5: "May",      6: "June",
        7: "July",     8: "August",   9: "September",
        10: "October", 11: "November",12: "December"
    }
    return months.get(i, "Invalid month")
```

---
