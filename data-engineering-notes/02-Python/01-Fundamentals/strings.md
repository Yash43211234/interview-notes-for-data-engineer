# Strings in Python

## 1. Overview

A string is a sequence of characters used to represent text in Python.

```python
name = "Yashwant"
message = 'Hello Python'
multiline = """This is
a multiline string."""
```

Strings are **ordered and immutable**. You can access individual characters using indexes, but you cannot modify a character directly.

```python
text = "Python"

print(text[0])   # P
print(text[-1])  # n

# text[0] = "J"  # TypeError: strings are immutable
```

To change a string, create a new one.

```python
text = "Python"
text = "J" + text[1:]

print(text)  # Jython
```

## 2. Indexing and slicing

Python uses zero-based indexing.

```python
text = "PYTHON"

print(text[0])    # P
print(text[2])    # T
print(text[-1])   # N
print(text[-2])   # O
```

Slicing syntax:

```python
string[start:stop:step]
```

The `stop` position is excluded.

```python
text = "PYTHON"

print(text[1:4])   # YTH
print(text[:3])    # PYT
print(text[2:])    # THON
print(text[::2])   # PTO
print(text[::-1])  # NOHTYP
```

Important points:

- Positive indexes move from left to right.
- Negative indexes move from right to left.
- A missing `start` or `stop` uses the appropriate beginning or end.
- A negative step traverses the string in reverse.
- Slicing beyond the string's boundaries generally does not raise an `IndexError`.

```python
text = "Python"

print(text[1:100])  # ython
print(text[100:])   # ''
```

However, accessing an invalid individual index does raise an error:

```python
# print(text[100])  # IndexError
```

## 3. String immutability

Strings cannot be modified in place. Operations such as replacement, concatenation, and case conversion return new strings.

```python
text = "hello"

new_text = text.upper()

print(text)      # hello
print(new_text)  # HELLO
```

This matters when processing large amounts of text: repeatedly concatenating strings in a loop can be inefficient. Building a list of pieces and using `join()` is often better.

```python
parts = ["Data", "Engineer", "Python"]

result = " ".join(parts)

print(result)  # Data Engineer Python
```

## 4. Common string methods

### Case conversion

```python
text = "Data Engineer"

print(text.lower())       # data engineer
print(text.upper())       # DATA ENGINEER
print(text.title())       # Data Engineer
print(text.capitalize())  # Data engineer
print(text.swapcase())    # dATA eNGINEER
```

| Method | Purpose |
|---|---|
| `lower()` | Converts text to lowercase |
| `upper()` | Converts text to uppercase |
| `title()` | Capitalizes the beginning of each word |
| `capitalize()` | Capitalizes the first character and lowercases the rest |
| `casefold()` | Performs stronger case normalization for caseless comparisons |

For case-insensitive comparisons, `casefold()` is generally more suitable than `lower()` when working with international text.

```python
a = "PYTHON"
b = "python"

print(a.casefold() == b.casefold())  # True
```

### Removing whitespace

```python
text = "   Python SQL   "

print(text.strip())   # Python SQL
print(text.lstrip())  # Python SQL   
print(text.rstrip())  #    Python SQL
```

- `strip()` removes whitespace from both ends.
- `lstrip()` removes whitespace from the left.
- `rstrip()` removes whitespace from the right.

These methods do not remove whitespace from the middle.

```python
text = "  Data   Engineer  "

print(text.strip())
# Data   Engineer
```

You can also supply specific characters to remove from the ends:

```python
text = "***hello***"

print(text.strip("*"))  # hello
```

**Important:** `strip(chars)` removes any matching characters from either end; it does not remove one exact substring.

### Searching and counting

```python
text = "data engineering"

print(text.find("eng"))    # 5
print(text.find("python")) # -1
print(text.index("data"))  # 0
print(text.count("e"))     # 3
```

| Method | Purpose |
|---|---|
| `find(sub)` | Returns the first matching index, or `-1` if absent |
| `index(sub)` | Returns the first matching index; raises `ValueError` if absent |
| `count(sub)` | Counts non-overlapping occurrences |
| `startswith(prefix)` | Checks whether the string begins with a prefix |
| `endswith(suffix)` | Checks whether the string ends with a suffix |

Example:

```python
filename = "orders.csv"

print(filename.startswith("orders"))  # True
print(filename.endswith(".csv"))      # True
```

For substring membership, use `in`:

```python
text = "Python for Data Engineering"

print("Data" in text)    # True
print("Java" in text)    # False
```

### Replacing text

```python
text = "I use Java"

result = text.replace("Java", "Python")

print(result)  # I use Python
```

You can limit the number of replacements:

```python
text = "apple apple apple"

print(text.replace("apple", "orange", 2))
# orange orange apple
```

`replace()` returns a new string and does not modify the original.

### Splitting strings

`split()` divides a string into a list of substrings.

```python
text = "Python,SQL,PySpark"

skills = text.split(",")

print(skills)
# ['Python', 'SQL', 'PySpark']
```

Without an argument, `split()` separates on runs of whitespace and ignores leading or trailing whitespace.

```python
text = "  Data   Engineer  "

print(text.split())
# ['Data', 'Engineer']
```

With an explicit delimiter, repeated delimiters can produce empty strings:

```python
text = "a,,b"

print(text.split(","))
# ['a', '', 'b']
```

### Joining strings

`join()` combines strings from an iterable using a separator.

```python
skills = ["Python", "SQL", "PySpark"]

print(", ".join(skills))
# Python, SQL, PySpark
```

The separator is the string before `.join()`.

```python
print("-".join(["2026", "10", "29"]))
# 2026-10-29
```

All elements must be strings. Convert other types first if necessary.

```python
values = [10, 20, 30]

result = ",".join(str(value) for value in values)

print(result)  # 10,20,30
```

### Checking string content

```python
print("123".isdigit())        # True
print("abc".isalpha())        # True
print("abc123".isalnum())     # True
print("   ".isspace())        # True
print("hello".islower())      # True
print("HELLO".isupper())      # True
```

| Method | Checks whether |
|---|---|
| `isalpha()` | All characters are letters and the string is nonempty |
| `isdigit()` | All characters are digits and the string is nonempty |
| `isnumeric()` | All characters are numeric characters and the string is nonempty |
| `isalnum()` | All characters are letters or numbers and the string is nonempty |
| `isspace()` | All characters are whitespace and the string is nonempty |

These checks do not validate a complete number in every format. For example, `"-12".isdigit()` is `False` because the minus sign is not a digit.

## 5. String formatting

String formatting inserts values into text.

### f-strings

F-strings are usually the clearest option in modern Python.

```python
name = "Yashwant"
age = 26

message = f"My name is {name} and I am {age} years old."

print(message)
# My name is Yashwant and I am 26 years old.
```

You can include expressions:

```python
price = 1250
quantity = 3

print(f"Total: {price * quantity}")
# Total: 3750
```

Formatting decimal places:

```python
price = 1250.5678

print(f"{price:.2f}")  # 1250.57
```

### Other formatting methods

```python
name = "Amit"
age = 25

print("Name: {}, Age: {}".format(name, age))
print("Name: %s, Age: %d" % (name, age))
```

Both `.format()` and `%` formatting are valid, but f-strings are generally easier to read.

## 6. Escape characters and raw strings

Escape sequences represent special characters.

| Escape sequence | Meaning |
|---|---|
| `\n` | New line |
| `\t` | Tab |
| `\\` | Backslash |
| `\'` | Single quote |
| `\"` | Double quote |

Example:

```python
text = "Hello\nPython"
print(text)
```

Output:

```text
Hello
Python
```

A raw string, prefixed with `r`, treats backslashes literally in most cases.

```python
path = r"C:\new\test\data.csv"

print(path)
# C:\new\test\data.csv
```

Raw strings are useful for Windows paths and regular expressions. A raw string cannot end with an odd number of backslashes because the final backslash would escape the closing quote.

## 7. String comparison

Strings can be compared using `==`, `!=`, and ordering operators.

```python
print("python" == "python")  # True
print("Python" == "python")  # False
print("apple" < "banana")    # True
```

String comparisons are lexicographical, based on Unicode code points. They are case-sensitive by default.

For case-insensitive comparison:

```python
a = "Python"
b = "PYTHON"

print(a.casefold() == b.casefold())  # True
```

## 8. Iterating over strings

A string is iterable, so you can process it character by character.

```python
text = "SQL"

for char in text:
    print(char)
```

Output:

```text
S
Q
L
```

You can also use `enumerate()` when you need the index.

```python
text = "SQL"

for index, char in enumerate(text):
    print(index, char)
```

Output:

```text
0 S
1 Q
2 L
```

## 9. Useful string problems

### Reverse a string

```python
text = "python"

print(text[::-1])
# nohtyp
```

### Check whether a string is a palindrome

A palindrome reads the same forward and backward.

```python
text = "madam"

is_palindrome = text == text[::-1]

print(is_palindrome)  # True
```

For case-insensitive checks that ignore spaces:

```python
text = "Never odd or even"

normalized = "".join(
    char.casefold()
    for char in text
    if char.isalnum()
)

print(normalized == normalized[::-1])  # True
```

### Count character frequency

```python
text = "banana"

frequency = {}

for char in text:
    frequency[char] = frequency.get(char, 0) + 1

print(frequency)
# {'b': 1, 'a': 3, 'n': 2}
```

This pattern is useful in interview problems involving character counts, duplicate detection, and anagrams.

### Remove duplicate characters while preserving order

```python
text = "programming"

result = "".join(dict.fromkeys(text))

print(result)
# progamin
```

`dict.fromkeys()` preserves the first occurrence of each character in modern Python.

### Count words

```python
text = "Python is useful for data engineering"

words = text.split()

print(len(words))  # 7
```

This simple approach counts whitespace-separated words. More complex text may require additional tokenization rules.

### Check whether two strings are anagrams

Anagrams contain the same characters with the same frequencies, regardless of order.

```python
a = "listen"
b = "silent"

is_anagram = sorted(a) == sorted(b)

print(is_anagram)  # True
```

This version is case-sensitive and treats spaces as characters. Normalize the inputs first if the problem requires ignoring case or whitespace.

## 10. Strings in Data Engineering

String operations are common when cleaning and transforming data.

### Cleaning whitespace and casing

```python
raw_name = "  yashwant KUMAR  "

clean_name = raw_name.strip().title()

print(clean_name)
# Yashwant Kumar
```

Be aware that `.title()` does not always produce correct capitalization for names, acronyms, or specialized text.

### Parsing a delimited record

```python
record = "101,Yashwant,75000"

employee_id, name, salary = record.split(",")

employee_id = int(employee_id)
salary = float(salary)

print(employee_id, name, salary)
```

For real CSV files, use Python's `csv` module or a data-processing library rather than manually splitting every line. CSV fields can contain quoted delimiters and newlines.

### Standardizing values

```python
status = "  COMPLETED  "

status = status.strip().lower()

print(status)  # completed
```

Standardization helps make filtering and joining more consistent when source systems use different casing or extra spaces.

### Validating file extensions

```python
filename = "orders.csv"

if filename.lower().endswith(".csv"):
    print("CSV file")
```

This is a basic extension check, not a guarantee that the file's contents are valid CSV.

## 11. Time complexity

For a string of length \(n\), typical operations have these costs:

| Operation | Typical time complexity |
|---|---:|
| Access one character by index | \(O(1)\) |
| Slice a substring of length \(k\) | \(O(k)\) |
| Search for a substring | Depends on the search and input; often described as up to \(O(nm)\) for a simple pattern of length \(m\) |
| Concatenate two strings | \(O(n+m)\) |
| Iterate through a string | \(O(n)\) |
| `split()` | \(O(n)\) for typical input |
| `join()` over strings with total output length \(n\) | \(O(n)\) |
| Reverse with slicing | \(O(n)\) |

Python uses immutable strings, so constructing a modified string generally requires allocating a new string. Avoid repeatedly building a large string with `result += piece` in a loop; collect pieces and use `"".join(pieces)` when appropriate.

## 12. Common interview mistakes

1. **Assuming strings are mutable.** Assigning to `text[0]` raises `TypeError`.
2. **Confusing `find()` and `index()`.** `find()` returns `-1` when absent; `index()` raises `ValueError`.
3. **Confusing `split()` and `join()`.** `split()` produces a list; `join()` produces a string.
4. **Forgetting that the slicing stop is exclusive.** `text[1:4]` includes positions 1, 2, and 3.
5. **Assuming `strip()` removes characters from the middle.** It only trims the ends.
6. **Assuming `isdigit()` validates signed or decimal numbers.** It does not.
7. **Ignoring case sensitivity.** `"Python"` and `"python"` are not equal by default.
8. **Using repeated concatenation for large strings.** Prefer collecting pieces and joining them.
9. **Treating character counts as bytes.** Python strings represent Unicode text; character count and encoded byte length are different.

For example:

```python
text = "é"

print(len(text))                  # 1
print(len(text.encode("utf-8")))  # 2
```

## 13. Interview practice

### Easy

**Q1. What is the output?**

```python
text = "Python"
print(text[1:4])
```

**Q2. What is the difference between `find()` and `index()`?**

**Q3. What is the output?**

```python
text = "  SQL  "
print(text.strip().lower())
```

**Q4. What is the output?**

```python
text = "banana"
print(text.count("an"))
```

### Medium

**Q5. Reverse a string without using a loop.**

Input: `"data"`

Expected output: `"atad"`

**Q6. Count the frequency of each character in a string.**

Input: `"aabbc"`

Expected result: `a: 2, b: 2, c: 1`

**Q7. Remove duplicate characters while preserving their first occurrence.**

Input: `"banana"`

Expected output: `"ban"`

**Q8. Check whether two strings are anagrams.**

Input: `"listen"`, `"silent"`

Expected output: `True`

**Q9. Explain the output.**

```python
text = "hello"
result = text.replace("h", "H")

print(text)
print(result)
```

### Answers

1. `"yth"`
2. `find()` returns `-1` if absent; `index()` raises `ValueError`.
3. `"sql"`
4. `2`
5. `text[::-1]`
6. Use a dictionary or `collections.Counter`.
7. `"".join(dict.fromkeys(text))`
8. Compare normalized character frequencies or sorted characters.
9. `text` remains `"hello"`; `result` is `"Hello"` because strings are immutable.

## 14. Quick revision

- Strings are ordered, iterable, and immutable.
- Indexing accesses one character; slicing creates a substring.
- `[::-1]` reverses a string.
- `strip()` trims both ends; `split()` divides text; `join()` combines strings.
- `find()` returns `-1` if a substring is absent; `index()` raises an error.
- `replace()` returns a new string.
- Use f-strings for readable formatting.
- Use `casefold()` for robust case-insensitive comparisons.
- Use `"".join(pieces)` for efficiently assembling many string fragments.
- In Data Engineering, string processing supports data cleaning, parsing, standardization, and validation.

**Interview takeaway:** Understand immutability, slicing, built-in methods, string normalization, and character-frequency patterns. These concepts are frequently used in Python coding interviews and real-world data transformation.