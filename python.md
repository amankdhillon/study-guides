# Calico Lab Automation Intern — Codility Interview Study Guide (Full)

**Role:** Lab Automation Engineering Intern  
**Format:** 2 coding questions + 1 verbal  
**Time:** 30 min (Q1) / 15 min (Q2) / 15 min verbal (Q3)  
**Language:** Python

This guide is written assuming you find LeetCode-style problems uncomfortable. That's fine — this interview is more applied than algorithmic. The goal is to know Python deeply, think like an engineer, and write code that actually works on real-world messy data. You don't need to memorize graph traversals. You need to be fluent.

---

## Part 1: How Codility Works (Read This First)

Codility is a browser-based, timed coding platform. You write and run code in the browser — there's no VS Code, no autocomplete, no linter telling you what's wrong. This sounds scary but it's manageable if you've practiced in the environment.

**How grading works:**
- Your code is run against multiple test cases automatically
- You can see some test cases while coding; others are hidden and only run on submission
- You're scored on **correctness** (does it produce the right output?) and **performance** (does it run fast enough for large inputs?)
- For an internship-level role, correctness matters more than optimal big-O performance — a working O(n²) solution is usually fine

**The interface:**
- You can switch between Python versions — use Python 3
- You get a "Run" button to test against sample cases and a "Submit" button for final grading
- There is a notes/scratch area on the side — use it to write pseudocode before coding
- You can't import external libraries like numpy or pandas — stick to the standard library

**Before the interview:**
- Go to [app.codility.com/programmers/](https://app.codility.com/programmers/) and do their free practice problems
- Do at least one timed (don't let yourself pause) so you know how the pressure feels
- Specifically try their "Demo Test" — it shows exactly what the real test interface looks like

---

## Part 2: What Each Question Will Look Like

### Question 1 — 30 Minutes

This is the centerpiece. It will be a real problem, not a puzzle. In a lab automation context at Calico, expect something that involves reading or processing structured data and producing a meaningful result. Possible scenarios:

- You're given rows of instrument output (plate reader, liquid handler, sequencer). Parse them, handle bad rows, and compute aggregate statistics per sample or per plate row.
- You're given a list of experimental events with timestamps. Sort them, identify gaps or overlaps, and return a summary.
- You're given two datasets (e.g., expected plate layout vs. actual results). Compare them and return discrepancies.
- You're given a workflow definition (a sequence of steps with conditions) and asked to simulate it or validate it.

**How to approach it:**
1. Read the entire problem statement before writing a single line
2. Write 3–5 lines of pseudocode in a comment at the top
3. Write a skeleton (function signature + return statement) first
4. Fill in logic section by section
5. Test mentally with the given example before running
6. Run, check edge cases, clean up

**What the reviewer is looking for:**
- Does the code work?
- Is it readable? (clear variable names, logical structure)
- Does it handle edge cases? (empty input, malformed data, zero division)
- Is it broken into functions rather than one giant blob?

### Question 2 — 15 Minutes

This will be tighter and more focused. Think: a utility function or small transformation. Possible examples:
- Parse a well ID string and return its (row, col) as integers
- Given a list of sample names, return only the valid ones (based on a format rule)
- Given two sorted lists, return their intersection
- Reverse or reorder something according to a rule

**Strategy:** Move fast. Spend 2 minutes reading, 10 minutes coding, 3 minutes testing. Do not try to write perfect code — write working code.

### Question 3 — 15 Minutes (Verbal)

No keyboard. Just talking. This is actually a great opportunity to stand out because most CS students aren't taught to verbalize their thinking.

Common question types:
- System design lite: "How would you build X?"
- Debugging scenario: "A robot arm keeps depositing samples in the wrong well — how do you investigate?"
- Data pipeline: "How would you process output from 100 plate readers running simultaneously?"
- Tradeoffs: "When would you use a database vs. flat files for storing experiment results?"

**Framework to answer any verbal question:**

```
1. RESTATE  — "So what you're asking is..."
2. CLARIFY  — "Before I answer, can I assume X?"
3. APPROACH — "My high-level approach would be..."
4. DETAIL   — Walk through specifics step by step
5. TRADEOFFS — "The downside of this approach is... an alternative would be..."
6. EDGE CASES — "I'd also want to handle the case where..."
```

This structure alone will make you sound 10x more prepared than someone who just starts rambling.

---

## Part 3: Python Built-ins — The Complete Reference

This is the section you asked for. These are grouped by what they do, with real examples for each. Read through all of these. Many will appear in your interview.

---

### 3.1 String Methods

Strings are everywhere in lab data — sample names, well IDs, file headers, flags.

```python
s = "  A01_Sample_High_Rep2  "

# Whitespace
s.strip()           # "A01_Sample_High_Rep2"   — removes leading/trailing whitespace
s.lstrip()          # removes only left whitespace
s.rstrip()          # removes only right whitespace

# Case
s.upper()           # "  A01_SAMPLE_HIGH_REP2  "
s.lower()           # "  a01_sample_high_rep2  "
s.title()           # "  A01_Sample_High_Rep2  "  (capitalizes after non-letter)

# Splitting and joining
"A,B,C".split(",")         # ['A', 'B', 'C']
"A B C".split()            # ['A', 'B', 'C']  (splits on any whitespace)
"A B C".split(" ", 1)      # ['A', 'B C']  (split at most 1 time)
",".join(["A", "B", "C"])  # "A,B,C"
"\n".join(lines)           # join list of strings with newlines

# Searching
"A01" in "Well A01 OK"       # True
"A01_Sample".startswith("A") # True
"A01_Sample".endswith("ple") # True
"A01_Sample".find("Sample")  # 4  (index of first match, -1 if not found)
"A01_Sample".index("Sample") # 4  (same but raises ValueError if not found)
"A01_Sample".count("_")      # 1

# Replacing
"A,B,C".replace(",", "\t")   # "A\tB\tC"
"  spaces  ".replace(" ", "") # "spaces"

# Checking content type
"123".isdigit()     # True
"abc".isalpha()     # True
"abc123".isalnum()  # True
"  ".isspace()      # True
"HELLO".isupper()   # True
"hello".islower()   # True

# Formatting
f"Well {row}{col:02d}"       # "Well A03"  (zero-padded to 2 digits)
"{:.3f}".format(3.14159)     # "3.142"
f"{value:.2f}"               # "3.14"
"%-10s" % "hi"               # "hi        " (left-aligned, 10 chars wide)

# Slicing
s = "A01"
s[0]       # 'A'
s[1:]      # '01'
s[:2]      # 'A0'
s[-1]      # '1'
s[::-1]    # '10A'  (reversed)

# Splitting on multiple characters — use re
import re
re.split(r"[,\t;]", "A,B\tC;D")  # ['A', 'B', 'C', 'D']

# Strip specific characters
"###hello###".strip("#")    # "hello"
"001234".lstrip("0")        # "1234"
```

---

### 3.2 List Methods and Operations

```python
data = [3, 1, 4, 1, 5, 9, 2, 6]

# Adding/removing
data.append(7)          # adds 7 to end: [3,1,4,1,5,9,2,6,7]
data.extend([8, 9])     # adds multiple: [..., 8, 9]
data.insert(0, 99)      # inserts 99 at index 0
data.pop()              # removes and returns last element
data.pop(2)             # removes and returns element at index 2
data.remove(1)          # removes first occurrence of value 1
data.clear()            # empties the list

# Info
len(data)               # length
data.count(1)           # how many times 1 appears
data.index(5)           # index of first 5

# Sorting
data.sort()                        # in-place ascending sort
data.sort(reverse=True)            # in-place descending
data.sort(key=lambda x: -x)        # in-place by custom key
sorted(data)                       # returns new sorted list (non-destructive)
sorted(data, key=lambda x: abs(x)) # sort by absolute value

# Reversing
data.reverse()         # in-place reverse
list(reversed(data))   # returns reversed iterator as list

# Copying
data.copy()            # shallow copy (same as data[:])
import copy
copy.deepcopy(data)    # deep copy (for nested structures)

# List slicing
data[2:5]      # elements at index 2, 3, 4
data[::2]      # every other element
data[::-1]     # reversed

# List comprehensions
squares = [x**2 for x in range(10)]
evens = [x for x in data if x % 2 == 0]
nested = [x for row in matrix for x in row]  # flatten 2D list

# Unpacking
first, *rest = [1, 2, 3, 4]   # first=1, rest=[2,3,4]
a, b, c = [1, 2, 3]           # direct unpacking

# Membership
5 in data       # True/False
5 not in data   # True/False

# Aggregation
sum(data)
min(data)
max(data)
min(data, key=lambda x: abs(x))   # min by custom key
max(data, key=len)                 # max by length (for strings)

# any() and all() — hugely useful
any(x > 5 for x in data)    # True if at least one element > 5
all(x > 0 for x in data)    # True if every element > 0
any(row["flag"] == "ERR" for row in rows)  # check for error flags
```

---

### 3.3 Dictionary Methods

Dicts are your best friend. Know every method cold.

```python
d = {"A01": 1.5, "B02": 3.2, "C03": 0.8}

# Access
d["A01"]                    # 1.5  (KeyError if missing)
d.get("A01")                # 1.5
d.get("Z99")                # None (no error)
d.get("Z99", 0.0)           # 0.0 (default value)

# Adding/updating
d["D04"] = 2.1              # add or overwrite
d.update({"E05": 4.1, "F06": 1.9})  # merge another dict in
d.setdefault("G07", 0.0)    # only sets if key doesn't exist yet

# Removing
d.pop("A01")                # removes and returns value
d.pop("Z99", None)          # removes if exists, returns None if not
del d["B02"]                # removes key (KeyError if missing)

# Iterating
for key in d:               # iterates over keys
for key in d.keys():        # same
for val in d.values():      # iterates over values
for key, val in d.items():  # iterates over (key, value) tuples — use this most often

# Info
len(d)
"A01" in d       # True/False (checks keys only)
list(d.keys())
list(d.values())
list(d.items())  # list of (key, value) tuples

# Dict comprehensions
inverted = {v: k for k, v in d.items()}    # swap keys and values
filtered = {k: v for k, v in d.items() if v > 1.0}
scaled = {k: v * 2 for k, v in d.items()}

# Merging dicts (Python 3.9+)
merged = d1 | d2            # new dict, d2 wins on conflicts
d1 |= d2                    # update d1 in-place

# Sorting a dict by value
sorted_by_val = dict(sorted(d.items(), key=lambda x: x[1]))
sorted_by_val_desc = dict(sorted(d.items(), key=lambda x: -x[1]))
```

---

### 3.4 The `collections` Module — Use This Constantly

```python
from collections import defaultdict, Counter, OrderedDict, deque, namedtuple

# --- defaultdict ---
# Like a dict, but automatically creates a default value for missing keys
# Avoids the "if key not in d: d[key] = []" pattern

grouped = defaultdict(list)
for well, signal in measurements:
    grouped[well].append(signal)     # no KeyError even on first insert

counts = defaultdict(int)
for item in items:
    counts[item] += 1                # no KeyError on first increment

nested = defaultdict(lambda: defaultdict(list))  # 2D grouping

# --- Counter ---
# Counts hashable objects. Like defaultdict(int) but with superpowers.

c = Counter(["A", "B", "A", "C", "A", "B"])
# Counter({'A': 3, 'B': 2, 'C': 1})

c["A"]              # 3
c["Z"]              # 0 (no KeyError — returns 0 for missing)
c.most_common(2)    # [('A', 3), ('B', 2)]  — top 2
c.most_common()[:-3:-1]  # bottom 2 (least common)

# Counter arithmetic
c1 = Counter({"A": 3, "B": 2})
c2 = Counter({"A": 1, "B": 5, "C": 2})
c1 + c2   # Counter({'B': 7, 'A': 4, 'C': 2})
c1 - c2   # Counter({'A': 2})  — subtracts, drops negatives
c1 & c2   # Counter({'A': 1, 'B': 2})  — min of each
c1 | c2   # Counter({'B': 5, 'A': 3, 'C': 2})  — max of each

# Count characters in a string
Counter("AABBBCCC")  # Counter({'B': 3, 'C': 3, 'A': 2})

# Update
c.update(["A", "A", "D"])  # adds to existing counts

# --- deque (double-ended queue) ---
# Use when you need to append/pop from both ends efficiently

dq = deque([1, 2, 3])
dq.append(4)        # add to right: deque([1,2,3,4])
dq.appendleft(0)    # add to left:  deque([0,1,2,3,4])
dq.pop()            # remove from right: 4
dq.popleft()        # remove from left:  0
dq.rotate(1)        # rotate right: deque([3,1,2])
dq.rotate(-1)       # rotate left:  deque([1,2,3])
deque(data, maxlen=5)  # fixed-size sliding window

# --- namedtuple ---
# Creates lightweight, readable data objects (no class needed)

Well = namedtuple("Well", ["row", "col", "signal", "flag"])
w = Well(row="A", col=1, signal=1.23, flag="OK")
w.row     # "A"
w.signal  # 1.23

# --- OrderedDict ---
# In Python 3.7+ regular dicts preserve insertion order,
# but OrderedDict has extra methods:
od = OrderedDict()
od.move_to_end("key")        # move key to end
od.move_to_end("key", last=False)  # move key to front
```

---

### 3.5 The `itertools` Module — Power Tools

```python
import itertools

# chain — flatten iterables together
list(itertools.chain([1,2], [3,4], [5]))   # [1,2,3,4,5]
list(itertools.chain.from_iterable([[1,2],[3,4]]))  # [1,2,3,4]

# combinations and permutations
list(itertools.combinations([1,2,3], 2))    # [(1,2),(1,3),(2,3)]
list(itertools.permutations([1,2,3], 2))   # [(1,2),(1,3),(2,1),(2,3),(3,1),(3,2)]
list(itertools.combinations_with_replacement([1,2], 2))  # [(1,1),(1,2),(2,2)]

# product — cartesian product (all combinations from multiple iterables)
list(itertools.product("AB", [1,2]))  # [('A',1),('A',2),('B',1),('B',2)]
# Perfect for generating all well IDs:
wells = [f"{r}{c:02d}" for r, c in itertools.product("ABCDEFGH", range(1,13))]

# groupby — group consecutive elements by a key
# IMPORTANT: data must be sorted by the key first!
data = [("A", 1), ("A", 2), ("B", 3), ("B", 4), ("A", 5)]
data.sort(key=lambda x: x[0])   # must sort first
for key, group in itertools.groupby(data, key=lambda x: x[0]):
    values = list(group)
    print(key, values)
# A [('A', 1), ('A', 2)]
# B [('B', 3), ('B', 4)]
# A [('A', 5)]   <- separate group because it wasn't adjacent

# islice — lazy slicing of any iterator
import itertools
first_10 = list(itertools.islice(some_generator, 10))

# takewhile / dropwhile
list(itertools.takewhile(lambda x: x < 5, [1,2,3,6,7,1]))  # [1,2,3]
list(itertools.dropwhile(lambda x: x < 5, [1,2,3,6,7,1]))  # [6,7,1]

# accumulate — running totals
list(itertools.accumulate([1, 2, 3, 4]))  # [1, 3, 6, 10]
import operator
list(itertools.accumulate([1,2,3,4], operator.mul))  # [1,2,6,24] — running product

# cycle and repeat
list(itertools.islice(itertools.cycle([1,2,3]), 7))  # [1,2,3,1,2,3,1]
list(itertools.repeat(0, 5))                          # [0,0,0,0,0]
```

---

### 3.6 The `functools` Module

```python
from functools import reduce, partial, lru_cache

# reduce — apply a function cumulatively to reduce a list to one value
reduce(lambda acc, x: acc + x, [1,2,3,4])   # 10 (sum)
reduce(lambda acc, x: acc * x, [1,2,3,4])   # 24 (product)
reduce(max, [3,1,4,1,5,9])                   # 9

# partial — pre-fill some arguments of a function
def multiply(x, y):
    return x * y

double = partial(multiply, 2)  # fixes x=2
double(5)    # 10
double(7)    # 14

# useful for callbacks and map()
scale_by_1000 = partial(round, ndigits=3)

# lru_cache — memoization (cache function results)
@lru_cache(maxsize=None)
def expensive_computation(n):
    if n < 2:
        return n
    return expensive_computation(n-1) + expensive_computation(n-2)
# Now repeated calls with the same n are instant
```

---

### 3.7 Built-in Functions — Every Single One You'll Use

```python
# --- Type conversion ---
int("42")           # 42
int("42.9")         # ValueError — use float() then int()
int(3.9)            # 3  (truncates, does NOT round)
float("3.14")       # 3.14
float("nan")        # float('nan')  — valid!
str(42)             # "42"
bool(0)             # False
bool([])            # False (empty containers are falsy)
bool("")            # False
list((1,2,3))       # [1,2,3]
tuple([1,2,3])      # (1,2,3)
set([1,1,2,3,3])    # {1,2,3}  (deduplicates)

# --- Math ---
abs(-5)             # 5
round(3.14159, 2)   # 3.14
round(2.5)          # 2  (banker's rounding — rounds to even!)
round(3.5)          # 4
pow(2, 10)          # 1024
pow(2, 10, 1000)    # 24  (modular exponentiation: 2^10 mod 1000)
divmod(17, 5)       # (3, 2)  — quotient and remainder at once

import math
math.floor(3.7)     # 3
math.ceil(3.2)      # 4
math.sqrt(16)       # 4.0
math.log(100, 10)   # 2.0  (log base 10)
math.log(math.e)    # 1.0  (natural log)
math.inf            # infinity
math.isnan(float("nan"))  # True
math.isinf(math.inf)      # True

# --- Sequence functions ---
len([1,2,3])        # 3
sum([1,2,3])        # 6
sum([1,2,3], 10)    # 16  (start from 10)
min(3, 1, 4)        # 1  (can take multiple args or an iterable)
max([3,1,4])        # 4
min("abc")          # 'a'  (works on strings too!)
sorted([3,1,2])     # [1,2,3]
sorted("cba")       # ['a','b','c']
reversed([1,2,3])   # returns an iterator — wrap in list()
list(reversed([1,2,3]))  # [3,2,1]

# --- Iteration helpers ---
enumerate(["a","b","c"])           # (0,'a'), (1,'b'), (2,'c')
enumerate(["a","b","c"], start=1)  # (1,'a'), (2,'b'), (3,'c')
zip([1,2,3], ["a","b","c"])        # (1,'a'), (2,'b'), (3,'c')
zip([1,2,3], ["a","b"])            # (1,'a'), (2,'b')  — stops at shortest
list(zip(*matrix))                  # transpose a 2D list!

# --- Filter, map ---
list(filter(None, [0, 1, "", "a", None, 2]))  # [1, 'a', 2]  — removes falsy
list(filter(lambda x: x > 2, [1,2,3,4]))      # [3, 4]
list(map(str, [1,2,3]))                        # ['1','2','3']
list(map(float, ["1.1","2.2","3.3"]))          # [1.1, 2.2, 3.3]
list(map(lambda x: x**2, [1,2,3,4]))          # [1,4,9,16]

# --- Object introspection ---
type(42)            # <class 'int'>
isinstance(42, int)           # True
isinstance(42, (int, float))  # True (checks multiple types)
isinstance([], list)          # True
hasattr(obj, "method_name")   # True if obj has that attribute
getattr(obj, "attr", default) # get attribute with fallback

# --- Misc ---
id(obj)             # unique integer identity of object
hash("hello")       # integer hash value (works on any hashable)
hex(255)            # '0xff'
bin(10)             # '0b1010'
oct(8)              # '0o10'
ord("A")            # 65  — character to ASCII code
chr(65)             # 'A' — ASCII code to character
repr("hello\n")     # "'hello\\n'" — unambiguous string representation
input("prompt: ")   # read a line from stdin (rarely used in Codility)
print(*[1,2,3], sep=", ")  # "1, 2, 3" — spread list as args
```

---

### 3.8 Set Operations

Sets are underused and incredibly powerful for problems involving duplicates, membership, and comparisons.

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

# Creation
set([1,1,2,2,3])    # {1,2,3}  — deduplicates a list instantly
set("hello")        # {'h','e','l','o'}

# Basic operations
a | b               # {1,2,3,4,5,6} — union
a & b               # {3,4}          — intersection
a - b               # {1,2}          — difference (in a but not b)
b - a               # {5,6}          — difference (in b but not a)
a ^ b               # {1,2,5,6}      — symmetric difference (in one but not both)

# Checks
3 in a              # True
a.issubset(b)       # False
a.issuperset(b)     # False
a.isdisjoint({7,8}) # True (no overlap)

# Mutation
a.add(5)
a.remove(5)         # KeyError if not present
a.discard(5)        # no error if not present
a.update([6,7,8])   # add multiple

# Frozen set (hashable, can be used as dict key or set member)
fs = frozenset([1,2,3])

# Super useful pattern: find duplicates in a list
def find_duplicates(lst):
    seen = set()
    duplicates = set()
    for x in lst:
        if x in seen:
            duplicates.add(x)
        seen.add(x)
    return duplicates

# Find missing items
expected_wells = set(f"{r}{c:02d}" for r in "ABCDEFGH" for c in range(1,13))
actual_wells = set(row["well_id"] for row in data)
missing = expected_wells - actual_wells
```

---

### 3.9 The `re` Module (Regular Expressions)

Don't panic — you only need a handful of patterns for lab data.

```python
import re

# Core functions
re.search(pattern, string)     # find first match anywhere in string → match object or None
re.match(pattern, string)      # match at START of string only
re.fullmatch(pattern, string)  # entire string must match
re.findall(pattern, string)    # returns list of all matches (as strings)
re.finditer(pattern, string)   # returns iterator of match objects
re.sub(pattern, replacement, string)  # replace all matches
re.split(pattern, string)      # split on pattern

# The match object
m = re.search(r"([A-H])(\d{2})", "Well A03 signal 1.23")
m.group(0)   # 'A03' — full match
m.group(1)   # 'A'   — first capture group
m.group(2)   # '03'  — second capture group
m.start()    # 5     — start index
m.end()      # 8     — end index

# Essential patterns for lab data
r"\d+"           # one or more digits
r"\d{2}"         # exactly 2 digits
r"\d+\.\d+"      # a decimal number like 3.14
r"[A-H]"         # single uppercase letter A through H
r"[A-H]\d{2}"    # well ID like A01, B12
r"[A-Za-z0-9_]+" # alphanumeric identifier
r"\s+"           # one or more whitespace characters
r"^\s*$"         # blank line (only whitespace)
r"OK|ERR|WARN"   # literal alternatives

# Flags
re.search(r"error", text, re.IGNORECASE)  # case-insensitive
re.search(r"^start", text, re.MULTILINE)  # ^ matches each line start
re.DOTALL  # . matches newlines too

# Named groups — much more readable
m = re.search(r"(?P<row>[A-H])(?P<col>\d{2})", "A03")
m.group("row")  # 'A'
m.group("col")  # '03'

# Compile for performance when reusing
pattern = re.compile(r"[A-H]\d{2}")
pattern.findall("A01 B12 C99 Z00")  # ['A01', 'B12']  — Z00 doesn't match [A-H]
```

---

### 3.10 File I/O and the `csv` Module

```python
# Reading a file
with open("file.txt") as f:
    content = f.read()           # entire file as one string
    lines = f.readlines()        # list of lines (includes \n)
    
with open("file.txt") as f:
    for line in f:               # iterate line by line (memory efficient)
        line = line.strip()

# Writing a file
with open("output.txt", "w") as f:
    f.write("hello\n")
    f.writelines(["line1\n", "line2\n"])

# Appending
with open("log.txt", "a") as f:
    f.write("new entry\n")

# CSV reading
import csv

with open("data.csv") as f:
    reader = csv.reader(f)           # each row is a list
    for row in reader:
        print(row[0], row[1])

with open("data.csv") as f:
    reader = csv.DictReader(f)       # each row is a dict keyed by header
    for row in reader:
        print(row["well_id"], row["signal"])

# CSV writing
with open("output.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["well_id", "signal", "flag"])
    writer.writeheader()
    writer.writerow({"well_id": "A01", "signal": 1.23, "flag": "OK"})

# TSV (tab-separated) — just change the delimiter
with open("data.tsv") as f:
    reader = csv.DictReader(f, delimiter="\t")

# Reading from a string (if data is passed as string, not file)
import io
data_string = "well_id,signal\nA01,1.23\nB02,4.56"
reader = csv.DictReader(io.StringIO(data_string))
for row in reader:
    print(row)
```

---

### 3.11 JSON, Working with Nested Data

```python
import json

# Parse JSON string to Python object
data = json.loads('{"well": "A01", "signal": 1.23}')
data["well"]   # 'A01'

# Parse JSON from file
with open("data.json") as f:
    data = json.load(f)

# Convert Python object to JSON string
json.dumps({"well": "A01", "signal": 1.23})
json.dumps(data, indent=2)        # pretty-printed
json.dumps(data, sort_keys=True)  # sorted keys

# Write JSON to file
with open("output.json", "w") as f:
    json.dump(data, f, indent=2)
```

---

### 3.12 The `statistics` Module

```python
import statistics

data = [1.2, 3.4, 2.1, 0.9, 5.6, 3.4, 2.1]

statistics.mean(data)           # arithmetic mean
statistics.median(data)         # middle value
statistics.mode(data)           # most common value (2.1 or 3.4 here — may raise if tie in older Python)
statistics.multimode(data)      # list of all modes (Python 3.8+)
statistics.stdev(data)          # sample standard deviation
statistics.pstdev(data)         # population standard deviation
statistics.variance(data)       # sample variance
statistics.pvariance(data)      # population variance

# Quantiles (Python 3.8+)
statistics.quantiles(data, n=4)  # quartiles [Q1, Q2, Q3]

# NormalDist (Python 3.8+)
dist = statistics.NormalDist(mu=0, sigma=1)
dist.cdf(1.96)       # probability of being < 1.96 standard deviations
dist.zscore(2.5)     # z-score of value 2.5 given this distribution

# Z-score outlier detection (manual, since you can't use scipy)
def zscore_outliers(values, threshold=2.0):
    mean = statistics.mean(values)
    sd = statistics.stdev(values)
    return [(v, (v - mean) / sd) for v in values if abs((v - mean) / sd) > threshold]
```

---

### 3.13 `heapq` — Priority Queue / Efficient Min/Max

```python
import heapq

data = [5, 2, 8, 1, 9, 3]

heapq.nlargest(3, data)   # [9, 8, 5]  — top 3
heapq.nsmallest(3, data)  # [1, 2, 3]  — bottom 3

# Works with key= like sorted()
samples = [("A01", 1.5), ("B02", 0.2), ("C03", 3.1)]
heapq.nlargest(2, samples, key=lambda x: x[1])   # [('C03', 3.1), ('A01', 1.5)]

# As a proper min-heap
heap = []
heapq.heappush(heap, 3)
heapq.heappush(heap, 1)
heapq.heappush(heap, 5)
heapq.heappop(heap)   # 1  (always pops the minimum)

# Turn a list into a heap in-place
heapq.heapify(data)

# Max-heap: negate values
heapq.heappush(heap, -value)
-heapq.heappop(heap)  # get max
```

---

### 3.14 `datetime` — Working with Timestamps

Lab workflows always have timestamps.

```python
from datetime import datetime, timedelta, date

# Parse a timestamp string
dt = datetime.strptime("2024-03-15 14:30:00", "%Y-%m-%d %H:%M:%S")
dt.year    # 2024
dt.month   # 3
dt.day     # 15
dt.hour    # 14
dt.minute  # 30

# Common format codes
# %Y = 4-digit year, %m = month, %d = day
# %H = hour (24h), %M = minute, %S = second
# %I = hour (12h), %p = AM/PM
# %f = microseconds

# Current time
now = datetime.now()
today = date.today()

# Format back to string
dt.strftime("%Y-%m-%d")          # "2024-03-15"
dt.strftime("%m/%d/%Y %H:%M")   # "03/15/2024 14:30"

# Arithmetic
dt + timedelta(hours=2)          # 2 hours later
dt - timedelta(days=1)           # yesterday
dt2 - dt1                        # timedelta object
(dt2 - dt1).total_seconds()      # difference in seconds

# Comparison
dt1 < dt2     # True if dt1 is earlier
dt1 == dt2    # True if same moment

# Sorting by timestamp
events.sort(key=lambda e: e["timestamp"])  # if timestamps are strings in ISO format, lexicographic sort works
# or
events.sort(key=lambda e: datetime.strptime(e["timestamp"], "%Y-%m-%d %H:%M:%S"))
```

---

### 3.15 `os` and `pathlib` — File System

```python
import os
from pathlib import Path

# os basics
os.getcwd()                    # current directory
os.listdir(".")                # list files in directory
os.path.exists("file.txt")     # True/False
os.path.join("dir", "file.csv") # "dir/file.csv" (platform-safe)
os.path.basename("/dir/file.csv")  # "file.csv"
os.path.dirname("/dir/file.csv")   # "/dir"
os.path.splitext("file.csv")       # ("file", ".csv")

# pathlib (cleaner, preferred in modern Python)
p = Path("data/results.csv")
p.exists()          # True/False
p.name              # "results.csv"
p.stem              # "results"
p.suffix            # ".csv"
p.parent            # Path("data")
p.read_text()       # read entire file as string
list(Path(".").glob("*.csv"))      # all CSV files in current dir
list(Path(".").rglob("*.csv"))     # recursive
```

---

### 3.16 `sys` — System/Input

```python
import sys

sys.argv            # list of command-line arguments ['script.py', 'arg1', 'arg2']
sys.stdin           # standard input (can iterate lines from)
sys.stdout.write()  # like print but no newline by default

# Read all input at once (common in Codility):
data = sys.stdin.read().split()

# Read line by line:
for line in sys.stdin:
    process(line.strip())
```

---

### 3.17 Sorting Tricks — Deep Dive

Sorting comes up in almost every problem. Know all the ways.

```python
# Sort by multiple keys
data = [("B", 2), ("A", 3), ("A", 1)]
sorted(data, key=lambda x: (x[0], x[1]))  # [('A',1),('A',3),('B',2)]

# Sort by one key ascending, another descending
# Use negation for numbers
sorted(data, key=lambda x: (x[0], -x[1]))  # sort by name asc, then value desc

# Sort strings case-insensitively
sorted(names, key=str.lower)
sorted(names, key=lambda s: s.lower())

# Sort by string length, then alphabetically
sorted(words, key=lambda w: (len(w), w))

# Natural sort (A1, A2, A10 not A1, A10, A2)
import re
def natural_key(s):
    return [int(c) if c.isdigit() else c for c in re.split(r"(\d+)", s)]
sorted(["A10", "A2", "A1"], key=natural_key)  # ['A1', 'A2', 'A10']

# Sort well IDs correctly (A01 < A02 < ... < H12)
sorted(well_ids, key=lambda w: (w[0], int(w[1:])))

# Stable sort guarantee: Python's sort is stable
# Items that compare equal keep their original relative order
# This means you can sort by secondary key first, then primary key
data.sort(key=lambda x: x[1])  # sort by secondary first
data.sort(key=lambda x: x[0])  # then primary — secondary preserved within groups
```

---

### 3.18 Comprehensions — All Forms

```python
# List comprehension
[x**2 for x in range(5)]                         # [0,1,4,9,16]
[x for x in data if x > 0]                       # filter
[f(x) for x in data if condition(x)]             # transform + filter

# Dict comprehension
{k: v for k, v in items}
{well: float(signal) for well, signal in raw.items() if signal != ""}

# Set comprehension
{x % 3 for x in range(10)}   # {0, 1, 2}

# Generator expression (like list comp but lazy — use in sum/any/all/max)
sum(x**2 for x in range(100))         # no intermediate list
any(row["flag"] == "ERR" for row in data)
max(len(name) for name in names)

# Nested list comprehension (flatten 2D)
matrix = [[1,2,3],[4,5,6],[7,8,9]]
flat = [x for row in matrix for x in row]  # [1,2,3,4,5,6,7,8,9]

# Build a matrix
[[i * j for j in range(5)] for i in range(5)]

# Conditional expression (ternary)
result = "high" if signal > 2.0 else "low"
values = ["OK" if v > 0 else "FAIL" for v in data]
```

---

### 3.19 Error Handling — Complete Patterns

```python
# Basic try/except
try:
    value = float(raw)
except ValueError:
    value = None

# Multiple exceptions
try:
    result = data[index]
except IndexError:
    result = None
except KeyError:
    result = None
except (ValueError, TypeError) as e:
    print(f"Type error: {e}")
    result = None

# else: runs if no exception
try:
    value = float(raw)
except ValueError:
    value = None
else:
    print("Parsed successfully")

# finally: always runs
try:
    f = open("file.txt")
    data = f.read()
except FileNotFoundError:
    data = ""
finally:
    f.close()  # runs even if exception occurred

# Raising
raise ValueError(f"Invalid well ID: {well_id}")
raise TypeError("Expected a string")

# Custom exception
class InstrumentError(Exception):
    pass

raise InstrumentError("Plate reader returned null signal")

# Catching and re-raising
try:
    parse(data)
except ValueError as e:
    logger.error(f"Parse failed: {e}")
    raise  # re-raise the same exception

# Assert (use for sanity checks during development)
assert len(wells) == 96, f"Expected 96 wells, got {len(wells)}"
```

---

### 3.20 Generator Functions

Useful for memory-efficient processing of large datasets.

```python
# A generator function uses yield instead of return
def parse_lines(filename):
    with open(filename) as f:
        for line in f:
            line = line.strip()
            if line and not line.startswith("#"):
                yield line.split(",")

for parts in parse_lines("data.csv"):
    print(parts)

# Generator expressions
squares = (x**2 for x in range(1000000))  # doesn't build entire list
next(squares)   # 0
next(squares)   # 1

# yield from — delegate to another generator
def chain_generators(*gens):
    for gen in gens:
        yield from gen
```

---

## Part 4: Lab Automation Domain Knowledge

You don't need to be a biologist, but knowing the vocabulary will help you understand problem statements faster.

**96-well plate:** A plastic tray with 96 small wells arranged in an 8×12 grid. Rows are letters A–H, columns are numbers 1–12. Wells hold tiny amounts of liquid for experiments. You'll almost certainly see this in your interview.

**384-well plate:** Same concept, 16×24 grid (384 wells). Rows A–P, columns 1–24.

**Control wells:** Designated wells with known expected values. Used to validate that the instrument is working correctly. Positive controls (should read high) and negative controls (should read low or zero).

**Plate reader:** An instrument that scans all wells in a plate and measures something (fluorescence, absorbance, luminescence). Output is typically a table of well IDs → numeric signals.

**Liquid handler:** A robotic system that precisely moves small volumes of liquid between wells, plates, or tubes. Has a defined order of operations. Errors include wrong volume, wrong well, tip contamination.

**Barcode/Sample ID:** Each sample or plate usually has a unique ID. Cross-referencing IDs across datasets is very common.

**Replicate:** Running the same experiment multiple times. You might be asked to aggregate replicate values (average, flag outliers).

**Z-factor:** A measure of assay quality. Z = 1 - 3*(σ_pos + σ_neg) / |μ_pos - μ_neg|. Values close to 1 = great assay. You don't need to compute this from memory but knowing it exists is good.

---

## Part 5: Full Practice Problems

### Problem 1 (30-min difficulty): Instrument Output Parser

**Prompt:** A plate reader generates output as a list of strings. Each string is formatted as:
`PLATE_ID:WELL_ID:SIGNAL:STATUS`

For example: `["EXP001:A01:1.23:OK", "EXP001:B02:ERR:FAIL", "EXP001:A02:4.56:OK"]`

Write a function `summarize_plate(records)` that:
1. Ignores any records where STATUS is not "OK" or where SIGNAL cannot be parsed as a float
2. Returns a dict mapping each plate ID to a dict with keys: `mean`, `max`, `min`, `count`, `flagged` (count of non-OK records)

```python
import statistics
from collections import defaultdict

def summarize_plate(records):
    plate_data = defaultdict(lambda: {"signals": [], "flagged": 0})
    
    for record in records:
        parts = record.strip().split(":")
        if len(parts) != 4:
            continue
        
        plate_id, well_id, signal_raw, status = parts
        
        if status != "OK":
            plate_data[plate_id]["flagged"] += 1
            continue
        
        try:
            signal = float(signal_raw)
        except ValueError:
            plate_data[plate_id]["flagged"] += 1
            continue
        
        plate_data[plate_id]["signals"].append(signal)
    
    result = {}
    for plate_id, info in plate_data.items():
        signals = info["signals"]
        if signals:
            result[plate_id] = {
                "mean": statistics.mean(signals),
                "max": max(signals),
                "min": min(signals),
                "count": len(signals),
                "flagged": info["flagged"]
            }
        else:
            result[plate_id] = {
                "mean": None,
                "max": None,
                "min": None,
                "count": 0,
                "flagged": info["flagged"]
            }
    
    return result
```

**Things the reviewer will notice:** modular structure, defaultdict usage, graceful handling of parse errors, None for empty signal lists instead of crashing.

---

### Problem 2 (15-min difficulty): Well ID Validator

**Prompt:** Write a function `validate_wells(well_ids)` that takes a list of strings and returns a tuple `(valid, invalid)` where `valid` is the sorted list of well IDs that match the 96-well format (one letter A–H followed by two digits 01–12) and `invalid` is the list of everything else, in original order.

```python
import re

def validate_wells(well_ids):
    pattern = re.compile(r"^[A-H](0[1-9]|1[0-2])$")
    valid = []
    invalid = []
    
    for w in well_ids:
        if pattern.fullmatch(w):
            valid.append(w)
        else:
            invalid.append(w)
    
    # Sort valid wells correctly: by row letter, then by column number
    valid.sort(key=lambda w: (w[0], int(w[1:])))
    
    return valid, invalid

# Test it mentally:
# "A01" → valid (row A, col 01)
# "H12" → valid (last well)
# "I01" → invalid (row I doesn't exist)
# "A13" → invalid (column 13 doesn't exist)
# "A1"  → invalid (must be two digits)
```

---

### Problem 3: Verbal — Sample Answer

**Question:** "How would you design a system to process the output files from 10 liquid handlers running simultaneously?"

**Sample strong answer:**

"Let me make sure I understand the problem — we have 10 robots running in parallel, each producing output files, and we need to process all of them in a reliable, organized way.

My high-level approach would be a pipeline with three stages: collection, processing, and reporting.

For collection, I'd have each robot write its output to a shared directory with a unique filename that includes the robot ID and timestamp — something like `robot_03_20240315_143022.csv`. A simple file watcher process would detect new files and queue them for processing.

For processing, I'd parse each file, validate the data (check for missing wells, out-of-range signals, failed status flags), and write results to a central store — could be a SQLite database for simplicity or flat JSON files keyed by plate ID. I'd also log any validation failures to a separate error file.

For reporting, I'd compute aggregate statistics per run and flag any plates where control wells fall out of expected range.

The main tradeoffs: I'm keeping it simple with a file-based approach rather than a message queue like Kafka, which would be more robust but much more complex for an internal tool. The weakness is if two robots finish at exactly the same time, you could have file naming collisions — I'd handle that by including a UUID in the filename.

Edge cases I'd want to handle: robot crashes mid-write (partial files), files with unexpected column headers (version mismatch), and the same plate ID appearing from two different robots (which shouldn't happen but definitely could)."

---

## Part 6: Day-of Strategy

### Time Management by Question

**Q1 (30 min):**
- 0:00–3:00 — Read the entire problem. Twice.
- 3:00–7:00 — Write pseudocode as comments
- 7:00–25:00 — Write the code
- 25:00–30:00 — Test with edge cases (empty input, one element, all-bad data)

**Q2 (15 min):**
- 0:00–2:00 — Read and plan
- 2:00–12:00 — Write the code
- 12:00–15:00 — Test

**Q3 (15 min verbal):**
- 0:00–1:00 — Restate and clarify
- 1:00–10:00 — Walk through your approach
- 10:00–13:00 — Cover tradeoffs
- 13:00–15:00 — Edge cases and wrap up

### The Most Important Rules

**Write working code first, optimize after.** A correct O(n²) solution beats a half-written O(n) solution every time.

**Name your variables like a human.** `signal_values` not `sv`. `well_id` not `w`. The reviewer reads your code.

**Comment your intent, not your code.** `# skip malformed rows` is useful. `# increment i by 1` is not.

**Test with the example they give you before submitting.** Run it in your head. If it doesn't match, you have a bug.

**Don't leave a blank.** If you run out of time, write a comment explaining what you would do next. Partial solutions with explained logic score better than silence.

**Edge cases to always consider:**
- Empty input (`[]` or `""`)
- One element
- All elements are the same
- All elements fail validation
- Very large input (does your solution time out?)
- Negative numbers, zero, floats vs. ints
- Leading/trailing whitespace in strings
- Case sensitivity (is "OK" the same as "ok"?)

---

## Part 7: Quick Reference Card

### Python Built-ins Cheat Sheet

| Category | Function/Method | What it does |
|----------|----------------|--------------|
| **Strings** | `s.strip()` | Remove leading/trailing whitespace |
| | `s.split(sep)` | Split into list on separator |
| | `s.join(lst)` | Join list with string as separator |
| | `s.replace(a, b)` | Replace all occurrences of a with b |
| | `s.startswith(x)` | True if s begins with x |
| | `s.endswith(x)` | True if s ends with x |
| | `s.upper()` / `s.lower()` | Change case |
| | `s.isdigit()` | True if all characters are digits |
| | `s.isalpha()` | True if all characters are letters |
| | `s.find(x)` | Index of first x, or -1 |
| | `f"{val:.2f}"` | Format float to 2 decimal places |
| | `f"{val:02d}"` | Zero-pad integer to width 2 |
| **Lists** | `lst.append(x)` | Add to end |
| | `lst.extend(other)` | Add all elements of other to end |
| | `lst.pop()` | Remove and return last element |
| | `lst.pop(i)` | Remove and return element at index i |
| | `lst.remove(x)` | Remove first occurrence of x |
| | `lst.sort(key=f)` | Sort in-place |
| | `sorted(lst, key=f)` | Return new sorted list |
| | `lst.count(x)` | Count occurrences of x |
| | `lst.index(x)` | Index of first x |
| | `lst.reverse()` | Reverse in-place |
| | `lst[::-1]` | New reversed copy |
| **Dicts** | `d.get(k, default)` | Get with fallback |
| | `d.items()` | (key, value) pairs |
| | `d.keys()` | All keys |
| | `d.values()` | All values |
| | `d.update(other)` | Merge other into d |
| | `d.pop(k, default)` | Remove and return |
| | `d.setdefault(k, v)` | Set only if missing |
| **Built-ins** | `len(x)` | Length |
| | `sum(x)` | Sum |
| | `min(x, key=f)` | Minimum (with optional key) |
| | `max(x, key=f)` | Maximum (with optional key) |
| | `sorted(x, key=f)` | Sort any iterable |
| | `enumerate(x)` | (index, value) pairs |
| | `zip(a, b)` | Pair elements |
| | `zip(*matrix)` | Transpose a 2D list |
| | `map(f, x)` | Apply f to all elements |
| | `filter(f, x)` | Keep elements where f is True |
| | `any(condition for x in lst)` | True if any match |
| | `all(condition for x in lst)` | True if all match |
| | `abs(x)` | Absolute value |
| | `round(x, n)` | Round to n decimal places |
| | `int(x)` | Convert to int (truncates) |
| | `float(x)` | Convert to float |
| | `str(x)` | Convert to string |
| | `ord(c)` | Character to ASCII int |
| | `chr(n)` | ASCII int to character |
| | `isinstance(x, T)` | Type check |
| | `divmod(a, b)` | (quotient, remainder) |
| | `hash(x)` | Hash value |
| | `set(lst)` | Deduplicate a list |
| | `list(iterable)` | Convert to list |
| **collections** | `defaultdict(type)` | Dict with automatic defaults |
| | `Counter(lst)` | Count occurrences |
| | `Counter.most_common(n)` | Top n elements |
| | `deque(lst)` | Double-ended queue |
| | `namedtuple(name, fields)` | Lightweight data class |
| **itertools** | `chain(*iterables)` | Flatten/combine iterables |
| | `product(a, b)` | Cartesian product |
| | `combinations(lst, r)` | Combinations of r |
| | `groupby(lst, key)` | Group consecutive elements |
| | `accumulate(lst)` | Running totals |
| | `takewhile(f, lst)` | Take while condition is True |
| **statistics** | `mean(data)` | Average |
| | `median(data)` | Middle value |
| | `stdev(data)` | Standard deviation |
| | `variance(data)` | Variance |
| **math** | `math.floor(x)` | Round down |
| | `math.ceil(x)` | Round up |
| | `math.sqrt(x)` | Square root |
| | `math.log(x, base)` | Logarithm |
| | `math.inf` | Infinity |
| | `math.isnan(x)` | True if NaN |
| **re** | `re.search(p, s)` | Find first match |
| | `re.findall(p, s)` | List of all matches |
| | `re.sub(p, r, s)` | Replace matches |
| | `re.split(p, s)` | Split on pattern |
| | `re.compile(p)` | Pre-compile pattern |
| **heapq** | `heapq.nlargest(n, lst)` | Top n elements |
| | `heapq.nsmallest(n, lst)` | Bottom n elements |
| | `heapq.heappush(h, x)` | Push onto heap |
| | `heapq.heappop(h)` | Pop minimum |
| **datetime** | `datetime.strptime(s, fmt)` | Parse timestamp string |
| | `datetime.strftime(fmt)` | Format timestamp to string |
| | `dt + timedelta(hours=n)` | Add time |
| | `(dt2 - dt1).total_seconds()` | Time difference in seconds |
| **csv** | `csv.DictReader(f)` | Read CSV as dicts |
| | `csv.DictWriter(f, fields)` | Write CSV from dicts |
| **json** | `json.loads(s)` | Parse JSON string |
| | `json.dumps(obj, indent=2)` | Serialize to JSON string |

---

*You've got this. The interview is testing whether you can think clearly and write working Python — not whether you've memorized obscure algorithms. Know your built-ins, practice on Codility's own interface, and write code that handles real-world messy data gracefully.*
