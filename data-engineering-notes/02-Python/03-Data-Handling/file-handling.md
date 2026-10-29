# File Handling in Python

## 1. What Is File Handling?

**File handling** is the process of creating, opening, reading, writing, appending, and closing files using a program.

In data engineering, file handling is important for:

- Reading source files such as CSV, JSON, and text.
- Writing processed data and logs.
- Managing configuration files.
- Processing large datasets incrementally.
- Handling errors when files are missing or inaccessible.

Python provides the built-in `open()` function for working with files.

---

## 2. Opening a File

Syntax:

```python
file = open("filename.txt", "mode")
```

Example:

```python
file = open("data.txt", "r")
content = file.read()
print(content)
file.close()
```

Here:

- `open()` opens the file.
- `"data.txt"` is the file path.
- `"r"` specifies read mode.
- `read()` reads the file contents.
- `close()` releases the file resource.

**Important:** If an exception occurs before `close()` executes, the file may remain open. For this reason, prefer the `with` statement.

---

## 3. File Opening Modes

| Mode | Purpose | Behavior |
|---|---|---|
| `"r"` | Read | Fails if the file does not exist |
| `"w"` | Write | Creates the file or overwrites an existing file |
| `"a"` | Append | Adds data at the end; creates the file if missing |
| `"x"` | Exclusive creation | Creates a new file; fails if it already exists |
| `"r+"` | Read and write | Requires an existing file |
| `"w+"` | Read and write | Creates or truncates the file |
| `"a+"` | Read and append | Creates the file if missing; writes at the end |

Modes can also include:

- `"b"` for binary mode, such as `"rb"` or `"wb"`.
- `"t"` for text mode, which is the default, such as `"rt"`.

Examples:

```python
open("data.txt", "r")    # Read text
open("data.txt", "rb")   # Read bytes
open("data.txt", "w")    # Write text
open("image.png", "rb")  # Read binary data
```

**Interview point:** `"w"` is destructive to existing file contents. Use `"a"` when you intend to append instead of replacing the file.

---

## 4. The `with` Statement

The recommended way to open a file is with a context manager.

```python
with open("data.txt", "r", encoding="utf-8") as file:
    content = file.read()
    print(content)
```

Python automatically closes the file when execution leaves the `with` block, including when an exception occurs.

This is safer than manually calling `close()`.

### Why Use `with`?

1. It manages file resources automatically.
2. It reduces the risk of leaving files open.
3. It makes code easier to read.
4. It works with exceptions and early returns.

**Best practice:** Use `with open(...)` for ordinary file operations.

---

## 5. Reading Files

Python provides several methods for reading file contents.

### `read()`

Reads the entire remaining content as one string by default.

```python
with open("data.txt", "r", encoding="utf-8") as file:
    content = file.read()

print(content)
```

You can specify the maximum number of characters to read in text mode:

```python
with open("data.txt", "r", encoding="utf-8") as file:
    content = file.read(10)

print(content)
```

This reads up to 10 characters, not necessarily 10 bytes.

### `readline()`

Reads one line at a time.

```python
with open("data.txt", "r", encoding="utf-8") as file:
    first_line = file.readline()
    second_line = file.readline()

print(first_line)
print(second_line)
```

A returned line normally retains its trailing newline character if one exists.

### `readlines()`

Reads all remaining lines into a list.

```python
with open("data.txt", "r", encoding="utf-8") as file:
    lines = file.readlines()

print(lines)
```

Example result:

```python
["First line\n", "Second line\n", "Third line\n"]
```

The exact result depends on the file contents.

### Iterating Over a File

For large text files, iterating line by line is usually more memory-efficient than reading everything into memory.

```python
with open("large_file.txt", "r", encoding="utf-8") as file:
    for line in file:
        print(line.rstrip("\n"))
```

The file object is iterable, so Python reads lines as the loop progresses.

Use `rstrip("\n")` when you specifically want to remove newline characters. Use `strip()` only when you also intend to remove whitespace from both ends.

---

## 6. Writing to Files

### Using `"w"` Mode

```python
with open("output.txt", "w", encoding="utf-8") as file:
    file.write("Hello, Python!\n")
    file.write("File handling is useful.")
```

If `output.txt` already exists, `"w"` truncates its previous contents before writing.

### Using `"a"` Mode

```python
with open("output.txt", "a", encoding="utf-8") as file:
    file.write("\nNew record added.")
```

Append mode adds content at the end of the file.

### `write()` vs. `writelines()`

| Method | Behavior |
|---|---|
| `write()` | Writes a string |
| `writelines()` | Writes an iterable of strings |

Example:

```python
lines = [
    "Python\n",
    "SQL\n",
    "PySpark\n"
]

with open("skills.txt", "w", encoding="utf-8") as file:
    file.writelines(lines)
```

**Important:** `writelines()` does not automatically insert newline characters between strings. Include `\n` when required.

---

## 7. File Paths

A file path identifies where a file is located.

### Relative Path

```python
with open("data/input.txt", "r", encoding="utf-8") as file:
    content = file.read()
```

This path is resolved relative to the program's current working directory, not necessarily the directory containing the Python script.

### Absolute Path

An absolute path specifies the complete path from the root or drive.

Windows example:

```python
path = r"C:\Users\Public\data.txt"
```

The `r` prefix creates a raw string literal, which is useful for Windows paths containing backslashes.

Another option is to use forward slashes:

```python
path = "C:/Users/Public/data.txt"
```

### Using `pathlib`

The `pathlib` module provides an object-oriented way to work with file paths.

```python
from pathlib import Path

path = Path("data") / "input.txt"

if path.exists():
    print(path.read_text(encoding="utf-8"))
```

You can also create directories:

```python
from pathlib import Path

output_dir = Path("output")
output_dir.mkdir(parents=True, exist_ok=True)
```

- `parents=True` creates missing parent directories.
- `exist_ok=True` prevents an error if the directory already exists.

For reusable code, `pathlib` is often clearer than manually concatenating path strings.

---

## 8. File Pointers and Seeking

When a file is opened, Python maintains a current position in the file. Reading or writing can move that position.

### `tell()`

Returns the current file position.

### `seek()`

Moves the file position.

```python
with open("data.txt", "r", encoding="utf-8") as file:
    print(file.tell())

    first = file.read(5)
    print(first)

    print(file.tell())

    file.seek(0)
    print(file.read(5))
```

For a simple text file, `seek(0)` returns to the beginning.

In text mode, `tell()` returns an opaque position value that can generally be passed back to `seek()` to restore a previously recorded position. It should not be treated as a universal character or byte count.

In binary mode, positions are byte offsets.

### When Is `seek()` Useful?

- Re-reading a file.
- Returning to a previously saved position.
- Reading specific binary data.
- Supporting certain file-processing workflows.

For sequential processing of large text files, iterating over lines is often simpler than repeatedly seeking.

---

## 9. Handling File Errors

File operations can fail for many reasons:

- The file does not exist.
- The program lacks permission.
- The path points to a directory instead of a file.
- The disk or storage system is unavailable.
- The file contents are not encoded as expected.

Use exception handling when failures are possible.

```python
try:
    with open("missing.txt", "r", encoding="utf-8") as file:
        content = file.read()

except FileNotFoundError:
    print("The file does not exist.")

except PermissionError:
    print("Permission denied.")
```

These are specific exceptions, so the code can respond appropriately to each failure.

### `FileNotFoundError` vs. `PermissionError`

- `FileNotFoundError`: The requested file or path cannot be found.
- `PermissionError`: The operation is not permitted by the operating system or storage permissions.

Avoid catching every exception without a reason:

```python
try:
    # File processing
    pass
except Exception:
    pass
```

Silently ignoring errors can hide data loss or incomplete processing. If you catch a broad exception, log useful details and handle or re-raise the failure appropriately.

---

## 10. Text Mode vs. Binary Mode

### Text Mode

Text mode reads and writes strings.

```python
with open("data.txt", "r", encoding="utf-8") as file:
    text = file.read()
```

Python decodes the stored bytes into text using the specified encoding.

### Binary Mode

Binary mode reads and writes bytes.

```python
with open("image.png", "rb") as file:
    content = file.read()

print(type(content))
```

Output:

```text
<class 'bytes'>
```

Binary mode is useful for images, PDFs, compressed files, and other data that should not be decoded as text.

### Why Specify Encoding?

Text files may use different character encodings. Specifying UTF-8 makes the expected encoding explicit.

```python
with open("data.txt", "r", encoding="utf-8") as file:
    content = file.read()
```

If the file uses a different encoding, UTF-8 decoding may fail. Determine the source encoding rather than blindly replacing or discarding invalid characters.

---

## 11. Working with File Metadata

`pathlib` provides useful file metadata.

```python
from pathlib import Path

path = Path("data.txt")

if path.exists() and path.is_file():
    print("File size:", path.stat().st_size)
```

The size is measured in bytes.

Other useful methods include:

```python
path.exists()      # Does the path exist?
path.is_file()     # Is it a file?
path.is_dir()      # Is it a directory?
path.name          # File name
path.suffix        # Extension, such as ".txt"
path.parent        # Parent directory
```

You can find files matching a pattern:

```python
from pathlib import Path

for path in Path("data").glob("*.csv"):
    print(path)
```

For recursive searches:

```python
for path in Path("data").rglob("*.csv"):
    print(path)
```

These patterns are useful when a pipeline needs to discover input files in a directory.

---

## 12. Practical Example: Process a Large Text File

Suppose a pipeline needs to read a file and write only non-empty lines to an output file.

```python
from pathlib import Path

input_path = Path("input.txt")
output_path = Path("cleaned.txt")

with input_path.open("r", encoding="utf-8") as source:
    with output_path.open("w", encoding="utf-8") as destination:
        for line in source:
            if line.strip():
                destination.write(line)
```

This approach:

1. Opens the input and output files.
2. Reads the input one line at a time.
3. Skips lines that contain only whitespace.
4. Writes the original contents of the remaining lines.
5. Closes both files automatically.

It avoids loading the entire input file into memory.

For production workflows, consider writing to a temporary output file and replacing the final file only after successful completion. This can reduce the risk of leaving a partially written result after a failure.

---

## 13. Practical Example: Read and Validate a Configuration File

Suppose a pipeline requires a text file containing a source path.

```python
from pathlib import Path

config_path = Path("config/source_path.txt")

try:
    source_path = config_path.read_text(encoding="utf-8").strip()

    if not source_path:
        raise ValueError("Source path is empty")

    print("Source:", source_path)

except FileNotFoundError:
    print("Configuration file is missing.")
```

This example uses `Path.read_text()`, which reads the entire file as a string and closes it automatically.

For a tiny configuration file, this is convenient. For large files, use a streaming approach.

In real applications, configuration files often use formats such as JSON, TOML, or YAML instead of plain text.

---

## 14. File Handling in Data Engineering

File handling is a foundation for data ingestion and processing.

A typical workflow might be:

1. Discover input files using `pathlib`.
2. Validate that the files exist and are accessible.
3. Read CSV or JSON data using format-aware parsers.
4. Validate required columns, types, and records.
5. Transform data and write outputs.
6. Log failures and keep track of processed files.
7. Avoid accidental overwrites and duplicate processing.

For very large datasets, standard Python file handling may be appropriate for line-oriented text processing, but distributed tools such as PySpark are often better for large-scale tabular transformations.

**Important distinction:** Reading a file with Python does not automatically make its data suitable for a data warehouse. Schema validation, type conversion, deduplication, and error handling are separate pipeline responsibilities.

---

## 15. Common Mistakes

1. **Forgetting to close files:** Use a `with` statement.
2. **Using `"w"` when intending to append:** `"w"` truncates existing contents; `"a"` appends.
3. **Loading huge files with `read()` or `readlines()`:** Iterate over the file when possible.
4. **Assuming relative paths are based on the script location:** They are normally resolved against the current working directory.
5. **Ignoring encoding:** Specify the expected encoding for text files.
6. **Assuming `writelines()` adds newlines:** It writes the supplied strings as they are.
7. **Catching exceptions silently:** This can hide failed or incomplete processing.
8. **Using text mode for arbitrary binary data:** Use `"rb"` and `"wb"` for byte-oriented data.
9. **Treating file size as a character count:** `stat().st_size` reports bytes.
10. **Overwriting important files without validation:** Use deliberate output paths and safer replacement strategies where necessary.

---

## 16. Interview Questions

### Q1. What is file handling in Python?

It is the process of opening, reading, writing, appending, and managing files using Python's file APIs.

### Q2. Why should you use `with open()`?

It automatically manages the file's lifetime and closes it when execution leaves the block, including when an exception occurs.

### Q3. What is the difference between `"w"` and `"a"`?

`"w"` creates or truncates a file before writing. `"a"` writes at the end and creates the file if it does not exist.

### Q4. What is the difference between `read()`, `readline()`, and `readlines()`?

- `read()`: Reads the remaining content as a string by default.
- `readline()`: Reads one line.
- `readlines()`: Reads the remaining lines into a list.

### Q5. How do you process a large text file efficiently?

Iterate over the file line by line instead of loading all contents into memory.

### Q6. What is the difference between text and binary mode?

Text mode decodes bytes into strings using an encoding. Binary mode returns bytes without text decoding.

### Q7. What do `seek()` and `tell()` do?

`seek()` changes the file position, and `tell()` reports the current position value.

### Q8. How do you handle a missing file?

Catch `FileNotFoundError` and take an appropriate action, such as reporting the missing input, retrying, or failing the pipeline.

### Q9. Why use `pathlib`?

It provides a clear, cross-platform API for paths, directories, file metadata, and file operations.

### Q10. How would you prevent partial output after a processing failure?

Write to a temporary file, validate successful completion, and then replace the final output using an appropriate atomic replacement strategy for the storage system.

---

## 17. Practice Tasks

Try solving these without copying the examples above.

1. Create a text file and write five lines into it.
2. Read a file line by line and count its lines.
3. Append a new line without deleting existing contents.
4. Read a file and count non-empty lines.
5. Handle a missing input file using `try` and `except`.
6. Find all `.csv` files in a directory using `pathlib`.
7. Print the size of a file in bytes.
8. Copy a text file line by line.
9. Explain why `"w"` can cause data loss.
10. Process a large log file without loading it completely into memory.

---

## 18. Quick Revision

- **`open()`**: Opens a file.
- **`with`**: Manages file resources automatically.
- **`read()`**: Reads the remaining content.
- **`readline()`**: Reads one line.
- **`readlines()`**: Returns remaining lines as a list.
- **`write()`**: Writes a string.
- **`writelines()`**: Writes an iterable of strings without adding separators automatically.
- **`"r"`**: Read.
- **`"w"`**: Create or overwrite.
- **`"a"`**: Append.
- **`"x"`**: Create only if the file does not already exist.
- **`"rb"` / `"wb"`**: Read or write binary data.
- **`seek()`**: Change the file position.
- **`tell()`**: Retrieve the current file position.
- **`pathlib.Path`**: Work with file paths and metadata.
- **`FileNotFoundError`**: Handle missing files.
- **Memory efficiency**: Iterate over large files rather than loading them all at once.

**Most important interview takeaway:** Understand file modes, context managers, reading and writing methods, exception handling, path management, and memory-efficient processing of large files.