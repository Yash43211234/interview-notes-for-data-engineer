# Regular Expressions in Python (`re`)

## 1. What Are Regular Expressions?

A **regular expression (regex)** is a pattern used to search, match, extract, split, or replace text.

Python provides the built-in `re` module for working with regular expressions.

Common use cases in Data Engineering:
- Extract email addresses, phone numbers, and IDs from text.
- Validate input formats.
- Clean unwanted characters from strings.
- Extract dates and numeric values from log files.
- Split text using complex patterns.
- Process semi-structured data.

```python
import re

text = "Order ID: 12345"

result = re.search(r"\d+", text)

print(result.group())  # 12345
```

Here, `\d+` means **one or more digits**.

## 2. Raw Strings: `r"..."`

Regex patterns frequently contain backslashes. Use raw strings to avoid confusion with Python's own escape sequences.

```python
pattern = r"\d+"
```

Without a raw string, Python may interpret some backslash sequences before the regex engine receives them.

**Interview tip:** Prefer `r"..."` for regex patterns.

## 3. Important Regex Symbols

| Symbol | Meaning | Example | Matches |
|---|---|---|---|
| `.` | Any character except newline by default | `a.c` | `abc`, `a7c` |
| `\d` | A digit | `\d` | `5` |
| `\D` | A non-digit | `\D` | `a` |
| `\w` | A word character: letters, digits, underscore | `\w+` | `user_123` |
| `\W` | A non-word character | `\W` | `@`, `!` |
| `\s` | Whitespace | `\s` | Space, tab, newline |
| `\S` | Non-whitespace | `\S+` | `hello` |
| `^` | Start of string | `^Hello` | `Hello world` |
| `$` | End of string | `world$` | `Hello world` |
| `*` | Zero or more repetitions | `ab*` | `a`, `ab`, `abbb` |
| `+` | One or more repetitions | `ab+` | `ab`, `abbb` |
| `?` | Zero or one repetition | `colou?r` | `color`, `colour` |
| `{n}` | Exactly n repetitions | `\d{4}` | `2026` |
| `{n,m}` | Between n and m repetitions | `\d{2,4}` | `12`, `123`, `1234` |
| `[]` | Character set | `[aeiou]` | Any listed vowel |
| `[^...]` | Any character not in the set | `[^0-9]` | Any non-digit |
| `\|` | Alternation: OR | `cat\|dog` | `cat`, `dog` |
| `()` | Capture a group | `(\d{4})` | Captures four digits |
| `\b` | Word boundary | `\bcat\b` | `cat`, but not `category` |

**Important:** Inside a character set, `^` means negation only when it appears first. Outside a character set, it usually indicates the start of a string.

## 4. Greedy and Non-Greedy Matching

Quantifiers such as `*` and `+` are greedy by default. They match as much text as possible while allowing the overall pattern to succeed.

```python
import re

text = "<name>Yash</name><name>Rahul</name>"

print(re.findall(r"<name>.*</name>", text))
# ['<name>Yash</name><name>Rahul</name>']
```

The `.*` consumes as much text as possible.

Add `?` after a quantifier to make it non-greedy.

```python
print(re.findall(r"<name>.*?</name>", text))
# ['<name>Yash</name>', '<name>Rahul</name>']
```

The non-greedy version matches as little text as possible while still satisfying the pattern.

**Common mistake:** Using `.*` to extract data between repeated delimiters can capture more text than expected.

## 5. Essential Functions in Python's `re` Module

### `re.search()`

Finds the first match anywhere in the string.

```python
text = "Order ID: 12345"

match = re.search(r"\d+", text)

if match:
    print(match.group())  # 12345
```

Use it when the pattern may appear anywhere in the text.

### `re.match()`

Checks for a match only at the beginning of the string.

```python
print(re.match(r"\d+", "123abc").group())
# 123

print(re.match(r"\d+", "abc123"))
# None
```

### `re.fullmatch()`

Requires the entire string to match the pattern.

```python
print(bool(re.fullmatch(r"\d{4}", "2026")))
# True

print(bool(re.fullmatch(r"\d{4}", "Year 2026")))
# False
```

Use `fullmatch()` when validating a complete field, such as a four-digit code.

### `re.findall()`

Returns all non-overlapping matches as a list.

```python
text = "Orders 101, 205, and 309"

print(re.findall(r"\d+", text))
# ['101', '205', '309']
```

If the pattern contains capturing groups, `findall()` returns the captured text rather than necessarily returning the full match.

```python
text = "2026-10-29"

print(re.findall(r"(\d{4})-(\d{2})-(\d{2})", text))
# [('2026', '10', '29')]
```

### `re.finditer()`

Returns an iterator of match objects. Each object provides the matched text and its position.

```python
text = "A12 B34"

for match in re.finditer(r"\d+", text):
    print(match.group(), match.start(), match.end())

# 12 1 3
# 34 5 7
```

Useful when you need match positions or want to process results one at a time.

### `re.sub()`

Replaces every non-overlapping match by default.

```python
text = "Phone: 987-654-3210"

cleaned = re.sub(r"\D", "", text)

print(cleaned)
# 9876543210
```

Replace multiple spaces with a single space:

```python
text = "Data    Engineering   with Python"

cleaned = re.sub(r"\s+", " ", text).strip()

print(cleaned)
# Data Engineering with Python
```

To replace only the first match, use `count=1`:

```python
print(re.sub(r"\d", "X", "12345", count=1))
# X2345
```

### `re.split()`

Splits text wherever the pattern matches.

```python
text = "apple,banana;orange mango"

result = re.split(r"[,;\s]+", text)

print(result)
# ['apple', 'banana', 'orange', 'mango']
```

This is useful when data contains multiple possible delimiters.

## 6. Capturing Groups and Named Groups

Groups let you extract specific parts of a match.

### Numbered groups

```python
import re

text = "2026-10-29"
match = re.search(r"(\d{4})-(\d{2})-(\d{2})", text)

print(match.group(0))  # 2026-10-29
print(match.group(1))  # 2026
print(match.group(2))  # 10
print(match.group(3))  # 29
```

- `group(0)` returns the complete match.
- `group(1)`, `group(2)`, and so on return captured groups.

### Named groups

Named groups make extracted fields easier to understand.

```python
text = "2026-10-29"

pattern = r"(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})"

match = re.search(pattern, text)

if match:
    print(match.group("year"))   # 2026
    print(match.group("month"))  # 10
    print(match.group("day"))    # 29
    print(match.groupdict())
    # {'year': '2026', 'month': '10', 'day': '29'}
```

**Interview tip:** Named groups are useful when extracting multiple fields from logs, filenames, or semi-structured records.

## 7. Character Sets and Alternation

### Character sets

```python
import re

print(re.findall(r"[aeiou]", "data engineering"))
# ['a', 'a', 'e', 'i', 'e', 'e', 'i']

print(re.findall(r"[A-Z]", "DataEngineer"))
# ['D', 'E']
```

Ranges can be combined:

- `[a-z]`: lowercase English letters.
- `[A-Z]`: uppercase English letters.
- `[0-9]`: digits.
- `[a-zA-Z0-9_]`: English letters, digits, and underscore.

For Unicode text, Python's `\w` and related character classes can match more than ASCII characters.

### Alternation

Use `|` to match one alternative or another.

```python
text = "Payment completed; refund pending"

print(re.findall(r"payment|refund", text, flags=re.IGNORECASE))
# ['Payment', 'refund']
```

If alternatives overlap, order can matter because the engine tries alternatives from left to right.

```python
print(re.findall(r"cat|catalog", "catalog"))
# ['cat']
```

Place the longer alternative first when appropriate:

```python
print(re.findall(r"catalog|cat", "catalog"))
# ['catalog']
```

## 8. Common Practical Patterns

These patterns are useful starting points, not universal validation rules.

### Extract numbers

```python
text = "Revenue: 45000, tax: 2500"

numbers = re.findall(r"\d+", text)
print(numbers)
# ['45000', '2500']
```

For signed integers and decimals:

```python
text = "Profit: -125.50, growth: 8.75"

numbers = re.findall(r"-?\d+(?:\.\d+)?", text)
print(numbers)
# ['-125.50', '8.75']
```

Here, `(?:...)` creates a non-capturing group.

### Extract dates in `YYYY-MM-DD` format

```python
text = "Created: 2026-10-29; Updated: 2026-11-01"

dates = re.findall(r"\b\d{4}-\d{2}-\d{2}\b", text)

print(dates)
# ['2026-10-29', '2026-11-01']
```

This matches the shape of a date, not whether the date is valid. For example, `2026-99-99` also matches. Use `datetime` to validate actual calendar dates.

### Extract email-like strings

```python
text = "Contact admin@example.com or support@company.org"

emails = re.findall(
    r"\b[\w.+-]+@[\w.-]+\.[A-Za-z]{2,}\b",
    text
)

print(emails)
# ['admin@example.com', 'support@company.org']
```

This is a practical extraction pattern, not a complete implementation of email-address validation.

### Extract a file extension

```python
filename = "orders_2026.csv"

match = re.search(r"\.([A-Za-z0-9]+)$", filename)

if match:
    print(match.group(1))
    # csv
```

The `\.` matches a literal dot because `.` otherwise means any character.

### Clean unwanted characters

```python
text = "  Order#123 - Completed!!  "

cleaned = re.sub(r"[^A-Za-z0-9 ]", "", text)
cleaned = re.sub(r"\s+", " ", cleaned).strip()

print(cleaned)
# Order123 Completed
```

Be careful: removing punctuation can change meaning. For example, deleting a decimal point from `12.50` produces `1250`.

## 9. Flags

Flags change how a regex pattern behaves.

| Flag | Meaning |
|---|---|
| `re.IGNORECASE` or `re.I` | Case-insensitive matching |
| `re.MULTILINE` or `re.M` | `^` and `$` operate at each line boundary |
| `re.DOTALL` or `re.S` | `.` also matches newline characters |
| `re.VERBOSE` or `re.X` | Allows readable patterns with whitespace and comments |

Example:

```python
text = "ERROR: Disk full\ninfo: Job started"

errors = re.findall(r"^error:.*$", text, flags=re.IGNORECASE | re.MULTILINE)

print(errors)
# ['ERROR: Disk full']
```

By default, `.` does not match a newline. With `re.DOTALL`, it does.

## 10. Escaping Special Characters

Some characters have special meanings in regex. To match them literally, escape them with `\`.

| Literal character | Pattern |
|---|---|
| Dot `.` | `r"\."` |
| Plus `+` | `r"\+"` |
| Parentheses `()` | `r"\(\)"` |
| Dollar `$` | `r"\$"` |
| Opening bracket `[` | `r"\["` |
| Backslash `\` | `r"\\"` |

Example:

```python
text = "Price: $25.50"

match = re.search(r"\$\d+\.\d{2}", text)

print(match.group())
# $25.50
```

## 11. Compiling a Regex Pattern

When the same pattern is reused many times, you can compile it into a pattern object.

```python
import re

pattern = re.compile(r"\d+")

print(pattern.findall("Orders 101 and 202"))
# ['101', '202']

print(pattern.findall("Batch 303"))
# ['303']
```

A compiled pattern provides methods such as `.search()`, `.match()`, `.fullmatch()`, `.findall()`, `.finditer()`, and `.sub()`.

Compilation can improve clarity and may help when reusing patterns. Python also caches some recent regex patterns, so compiling is not automatically a major performance improvement in every situation.

## 12. Regex in Data Engineering

### Example 1: Extract an order ID from log messages

```python
import re

logs = [
    "INFO Order ID=ORD12345 processed",
    "INFO Order ID=ORD67890 processed",
    "INFO No order available"
]

pattern = re.compile(r"\bOrder ID=(ORD\d+)\b")

for log in logs:
    match = pattern.search(log)

    if match:
        print(match.group(1))
```

Output:

```text
ORD12345
ORD67890
```

### Example 2: Clean and normalize text fields

```python
import re

customer_names = [
    "  Yash   Kumar ",
    "Rahul@@Sharma",
    "Priya---Singh"
]

for name in customer_names:
    cleaned = re.sub(r"\s+", " ", name).strip()
    print(cleaned)
```

This example normalizes whitespace. It intentionally leaves punctuation untouched because automatically removing punctuation might corrupt legitimate names.

### Example 3: Extract a date and status from a log

```python
import re

log = "2026-10-29 | status=SUCCESS | rows=1500"

pattern = re.compile(
    r"(?P<date>\d{4}-\d{2}-\d{2})"
    r"\s*\|\s*status=(?P<status>\w+)"
    r"\s*\|\s*rows=(?P<rows>\d+)"
)

match = pattern.search(log)

if match:
    print(match.groupdict())
```

Output:

```python
{'date': '2026-10-29', 'status': 'SUCCESS', 'rows': '1500'}
```

This technique can help extract structured fields from consistent log formats before loading them into a data pipeline.

For complex or inconsistent formats, prefer a proper parser rather than one very large regex.

## 13. Common Mistakes

1. **Using `match()` when the pattern appears later in the string.** Use `search()` when you need to find it anywhere.

2. **Using `search()` for complete-field validation.** Use `fullmatch()` when the entire value must satisfy the pattern.

3. **Forgetting that `.` is special.** Use `\.` to match a literal dot.

4. **Using greedy quantifiers without checking the result.** Use non-greedy forms such as `.*?` when appropriate.

5. **Assuming a matching format means valid data.** A regex can match `2026-99-99`; calendar validation requires additional logic.

6. **Assuming `findall()` always returns complete matched strings.** Capturing groups change its return format.

7. **Ignoring missing matches.** `search()` and `match()` return `None` when nothing matches. Calling `.group()` without checking can raise `AttributeError`.

8. **Writing extremely complicated patterns.** Split the work into smaller patterns or use a dedicated parser.

9. **Ignoring performance risks.** Certain complex patterns can cause catastrophic backtracking on adversarial input. Avoid ambiguous nested quantifiers and use time or input-size limits where appropriate.

## 14. Interview Questions

### Basic

1. What is a regular expression?
2. What is the difference between `re.match()` and `re.search()`?
3. What is the difference between `re.search()` and `re.fullmatch()`?
4. What do `\d`, `\w`, and `\s` mean?
5. What is the difference between `*`, `+`, and `?`?
6. What is the difference between greedy and non-greedy matching?
7. What is the difference between `re.findall()` and `re.finditer()`?
8. Why are raw strings such as `r"\d+"` recommended?
9. What is the purpose of capturing groups?
10. When would you use `re.sub()` instead of `str.replace()`?

### Intermediate

1. How would you extract all numbers from a log file?
2. How would you remove repeated whitespace?
3. How would you extract a date in `YYYY-MM-DD` format?
4. How would you validate that a field contains exactly six digits?
5. How do `re.MULTILINE` and `re.DOTALL` differ?
6. How do named groups improve readability?
7. How would you extract fields from a structured log message?
8. Why might a regex that matches a date still accept an invalid date?
9. What is a non-capturing group, written as `(?:...)`?
10. What is catastrophic backtracking, and why is it a performance concern?

## 15. Practice Problems

Try solving these without looking at the solutions first.

### Easy

1. Extract every digit sequence from `"A12 B345 C6"`.
2. Replace multiple spaces in `"Data    Engineer"` with one space.
3. Check whether `"123456"` contains exactly six digits.
4. Extract all `.csv` filenames from a string.
5. Find all occurrences of `"error"` regardless of case.

### Intermediate

1. Extract dates in `YYYY-MM-DD` format from a log.
2. Extract order IDs matching `ORD` followed by five digits.
3. Remove punctuation while preserving letters, digits, and spaces.
4. Split a string using commas, semicolons, or whitespace.
5. Extract `date`, `status`, and `rows` from a structured log using named groups.

### Challenge

Given this text:

```text
2026-10-29 | INFO | order_id=ORD12345 | rows=150
2026-10-29 | ERROR | order_id=ORD67890 | rows=0
2026-10-30 | INFO | order_id=ORD54321 | rows=250
```

Extract each record's date, log level, order ID, and row count into a list of dictionaries.

Expected structure:

```python
[
    {
        "date": "2026-10-29",
        "level": "INFO",
        "order_id": "ORD12345",
        "rows": "150"
    },
    # Remaining records...
]
```

## 16. Quick Revision

| Requirement | Recommended method or pattern |
|---|---|
| Find first occurrence anywhere | `re.search()` |
| Match only at the beginning | `re.match()` |
| Validate the entire string | `re.fullmatch()` |
| Extract all matches | `re.findall()` |
| Get matches with positions | `re.finditer()` |
| Replace matching text | `re.sub()` |
| Split using a pattern | `re.split()` |
| Extract specific fields | Capturing or named groups |
| Match one or more digits | `r"\d+"` |
| Match exactly six digits | `r"\d{6}"` with `fullmatch()` |
| Collapse whitespace | `re.sub(r"\s+", " ", text)` |
| Match a literal dot | `r"\."` |
| Match across lines with `.` | `re.DOTALL` |
| Reuse a pattern | `re.compile()` |

**Key takeaway:** Learn the core symbols, understand the differences between `search()`, `match()`, and `fullmatch()`, and practise extraction and cleaning patterns on realistic log and CSV data. For Data Engineering interviews, regex is most useful as a practical text-processing tool rather than as a memorization exercise.