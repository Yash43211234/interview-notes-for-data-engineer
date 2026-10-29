# CSV and JSON in Python

## 1. Overview

CSV and JSON are two common formats used to store and exchange data. Data engineers frequently use them to ingest data from files, APIs, applications, and external systems.

| Feature | CSV | JSON |
|---|---|---|
| Full form | Comma-Separated Values | JavaScript Object Notation |
| Structure | Rows and columns | Objects, arrays, and nested values |
| Best suited for | Tabular data | Structured and nested data |
| Human-readable | Yes | Yes |
| Python support | `csv` module | `json` module |
| Common use | Data exports, spreadsheets, ETL input | REST APIs, application events, configuration |
| Data types | Usually interpreted as text in the file | Supports strings, numbers, booleans, null, arrays, and objects |

CSV is often convenient for flat datasets. JSON is useful when records contain nested structures or fields with different shapes.

---

## 2. Working with CSV Files

CSV stores tabular data as rows, with fields separated by a delimiter such as a comma.

Example: `employees.csv`

```csv
employee_id,name,department,salary
101,Amit,Engineering,60000
102,Priya,Analytics,65000
103,Rahul,Engineering,58000
```

The first row commonly contains column names, and each subsequent row represents a record.

### Reading a CSV File

Python provides the built-in `csv` module.

```python
import csv

with open("employees.csv", "r", newline="", encoding="utf-8") as file:
    reader = csv.DictReader(file)

    for row in reader:
        print(row)
```

Output:

```text
{'employee_id': '101', 'name': 'Amit', 'department': 'Engineering', 'salary': '60000'}
{'employee_id': '102', 'name': 'Priya', 'department': 'Analytics', 'salary': '65000'}
{'employee_id': '103', 'name': 'Rahul', 'department': 'Engineering', 'salary': '58000'}
```

**Important:** `csv.DictReader` returns each record as a dictionary, but the field values are strings by default.

For example:

```python
salary = int(row["salary"])
```

Convert values explicitly when numeric or date operations are required.

### Using `csv.reader`

Use `csv.reader` when you want each row as a list rather than a dictionary.

```python
import csv

with open("employees.csv", "r", newline="", encoding="utf-8") as file:
    reader = csv.reader(file)

    for row in reader:
        print(row)
```

Output:

```text
['employee_id', 'name', 'department', 'salary']
['101', 'Amit', 'Engineering', '60000']
['102', 'Priya', 'Analytics', '65000']
['103', 'Rahul', 'Engineering', '58000']
```

Unlike `DictReader`, `reader` includes the header row unless you skip it explicitly.

### Writing a CSV File

Use `csv.writer` to write rows.

```python
import csv

rows = [
    ["employee_id", "name", "salary"],
    [101, "Amit", 60000],
    [102, "Priya", 65000]
]

with open("output.csv", "w", newline="", encoding="utf-8") as file:
    writer = csv.writer(file)
    writer.writerows(rows)
```

The file contains:

```csv
employee_id,name,salary
101,Amit,60000
102,Priya,65000
```

- `writerow()` writes one row.
- `writerows()` writes multiple rows.
- Opening with `"w"` overwrites an existing file.
- Use `"a"` to append, taking care to avoid accidentally writing duplicate headers.

### Writing Dictionaries to CSV

Use `csv.DictWriter` when records are dictionaries.

```python
import csv

employees = [
    {"id": 101, "name": "Amit", "salary": 60000},
    {"id": 102, "name": "Priya", "salary": 65000}
]

fieldnames = ["id", "name", "salary"]

with open("employees_output.csv", "w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(file, fieldnames=fieldnames)

    writer.writeheader()
    writer.writerows(employees)
```

`writeheader()` writes the column names specified by `fieldnames`.

---

## 3. Important CSV Concepts

### Delimiters

Although commas are common, CSV-like files can use other delimiters, such as tabs or semicolons.

```python
import csv

with open("employees.tsv", "r", newline="", encoding="utf-8") as file:
    reader = csv.DictReader(file, delimiter="\t")

    for row in reader:
        print(row)
```

### Quoted Values

A field may contain commas or quotation marks.

```csv
id,name,city
1,"Amit Kumar","Kolkata, India"
2,"Priya","Delhi"
```

The CSV parser handles the comma inside `"Kolkata, India"` as part of the field, rather than as a separator.

For this reason, prefer Python's `csv` module over manually splitting lines using `line.split(",")`.

### Missing Values

An empty CSV field is generally read as an empty string.

```csv
id,name,department
1,Amit,Engineering
2,Priya,
```

For the second record, `department` is `""`, not automatically Python's `None`.

You may normalize it:

```python
department = row["department"] or None
```

This converts an empty string to `None`, but it does not handle whitespace-only strings unless you strip them first.

```python
department = row["department"].strip() or None
```

---

## 4. Working with JSON

JSON represents structured data using objects, arrays, and primitive values.

Example: `employee.json`

```json
{
  "employee_id": 101,
  "name": "Amit",
  "department": "Engineering",
  "salary": 60000,
  "skills": ["Python", "SQL", "PySpark"],
  "active": true
}
```

Notice that JSON supports numbers and booleans directly. It also supports nested objects and arrays.

### JSON Data Types

| JSON type | Example | Python equivalent |
|---|---|---|
| Object | `{"name": "Amit"}` | `dict` |
| Array | `[10, 20, 30]` | `list` |
| String | `"Python"` | `str` |
| Number | `100` or `12.5` | `int` or `float` |
| Boolean | `true` or `false` | `True` or `False` |
| Null | `null` | `None` |

JSON uses lowercase `true`, `false`, and `null`. Python uses `True`, `False`, and `None`.

---

## 5. Reading JSON Files

Python provides the built-in `json` module.

```python
import json

with open("employee.json", "r", encoding="utf-8") as file:
    employee = json.load(file)

print(employee["name"])
print(employee["skills"])
```

Output:

```text
Amit
['Python', 'SQL', 'PySpark']
```

**Remember the difference:**

- `json.load(file)` reads JSON from a file-like object.
- `json.loads(string)` parses a JSON-formatted string.

Example using a string:

```python
import json

data = '{"name": "Amit", "salary": 60000}'

employee = json.loads(data)

print(employee["salary"])
```

Output:

```text
60000
```

---

## 6. Writing JSON Files

Use `json.dump()` to serialize a Python object into a JSON file.

```python
import json

employee = {
    "employee_id": 101,
    "name": "Amit",
    "salary": 60000,
    "active": True
}

with open("employee_output.json", "w", encoding="utf-8") as file:
    json.dump(employee, file, indent=4)
```

`indent=4` formats the output for readability.

### JSON String Conversion

Use `json.dumps()` to convert a Python object into a JSON string.

```python
import json

employee = {"name": "Amit", "salary": 60000}

json_string = json.dumps(employee)

print(json_string)
print(type(json_string))
```

Output:

```text
{"name": "Amit", "salary": 60000}
<class 'str'>
```

### `dump()` vs. `dumps()`

| Function | Purpose |
|---|---|
| `json.dump(obj, file)` | Write JSON to a file |
| `json.dumps(obj)` | Convert a Python object to a JSON string |
| `json.load(file)` | Read JSON from a file |
| `json.loads(string)` | Parse a JSON string into a Python object |

A useful memory trick: functions ending in `s` work with strings.

---

## 7. Handling Nested JSON

JSON frequently contains objects inside objects or arrays of objects.

Example:

```json
{
  "order_id": 5001,
  "customer": {
    "customer_id": 101,
    "name": "Amit"
  },
  "items": [
    {
      "product": "Laptop",
      "quantity": 1
    },
    {
      "product": "Mouse",
      "quantity": 2
    }
  ]
}
```

After parsing:

```python
import json

with open("order.json", "r", encoding="utf-8") as file:
    order = json.load(file)

print(order["customer"]["name"])
print(order["items"][0]["product"])
print(order["items"][1]["quantity"])
```

Output:

```text
Amit
Laptop
2
```

To access nested data, use dictionary keys and list indexes according to the structure.

### Extracting Array Records

```python
for item in order["items"]:
    print(item["product"], item["quantity"])
```

Output:

```text
Laptop 1
Mouse 2
```

This pattern is common when processing API responses and event records.

---

## 8. Converting Between CSV and JSON

Data pipelines often need to transform one format into another.

### CSV to JSON

```python
import csv
import json

with open("employees.csv", "r", newline="", encoding="utf-8") as csv_file:
    rows = list(csv.DictReader(csv_file))

with open("employees.json", "w", encoding="utf-8") as json_file:
    json.dump(rows, json_file, indent=4)
```

For the earlier CSV example, the JSON output will look like:

```json
[
    {
        "employee_id": "101",
        "name": "Amit",
        "department": "Engineering",
        "salary": "60000"
    },
    {
        "employee_id": "102",
        "name": "Priya",
        "department": "Analytics",
        "salary": "65000"
    }
]
```

Notice that the values are strings because `DictReader` reads them as strings.

If the output requires numeric IDs and salaries, convert them explicitly:

```python
for row in rows:
    row["employee_id"] = int(row["employee_id"])
    row["salary"] = int(row["salary"])
```

Perform suitable validation before converting production data, especially when fields can be missing or malformed.

### JSON to CSV

For a JSON file containing a list of flat objects:

```json
[
    {"id": 101, "name": "Amit", "salary": 60000},
    {"id": 102, "name": "Priya", "salary": 65000}
]
```

Convert it to CSV:

```python
import csv
import json

with open("employees.json", "r", encoding="utf-8") as json_file:
    employees = json.load(json_file)

if employees:
    fieldnames = list(employees[0].keys())

    with open("employees.csv", "w", newline="", encoding="utf-8") as csv_file:
        writer = csv.DictWriter(csv_file, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(employees)
```

This example assumes the JSON root is a non-empty list of flat dictionaries with compatible fields.

Nested JSON requires an explicit flattening strategy before it can be represented cleanly in a conventional table.

---

## 9. Handling Errors

Malformed files and unexpected data are common in production pipelines.

### CSV Errors

CSV parsing may succeed even when data is incomplete or incorrect. Validate required columns and values.

```python
import csv

with open("employees.csv", "r", newline="", encoding="utf-8") as file:
    reader = csv.DictReader(file)

    required_columns = {"employee_id", "name", "salary"}

    if not required_columns.issubset(reader.fieldnames or []):
        raise ValueError("Required CSV columns are missing")

    for row in reader:
        if not row["employee_id"] or not row["salary"]:
            print("Invalid record:", row)
```

This is a basic validation example; production pipelines should also validate data types, ranges, and duplicate identifiers where required.

### JSON Errors

`json.load()` and `json.loads()` raise `json.JSONDecodeError` when the input is invalid JSON.

```python
import json

data = '{"name": "Amit", "salary": }'

try:
    employee = json.loads(data)
except json.JSONDecodeError as error:
    print("Invalid JSON:", error)
```

Output begins with:

```text
Invalid JSON:
```

The exact error message depends on the malformed input.

A missing file raises `FileNotFoundError`, which is separate from a JSON parsing error.

---

## 10. CSV and JSON in Data Engineering

A typical data pipeline may use both formats:

1. **Ingestion:** Read CSV files from a source system or JSON records from an API.
2. **Validation:** Check required columns, data types, missing values, and record structure.
3. **Transformation:** Convert data types, flatten nested records, and standardize fields.
4. **Storage:** Write clean data into a database, a data lake, or a columnar format such as Parquet.
5. **Monitoring:** Record malformed records and failed batches for investigation.

For larger datasets, avoid loading the entire file into memory unnecessarily.

For example, iterate over CSV records:

```python
import csv

with open("large_file.csv", "r", newline="", encoding="utf-8") as file:
    reader = csv.DictReader(file)

    for row in reader:
        # Validate or process one record at a time.
        pass
```

For JSON, `json.load()` normally parses the entire JSON document into memory. For very large datasets, use a streaming-capable parser or a line-delimited JSON format when supported by the source.

### JSON Lines (JSONL / NDJSON)

JSON Lines stores one complete JSON value—commonly an object—per line.

```json
{"event_id": 1, "event": "login"}
{"event_id": 2, "event": "purchase"}
{"event_id": 3, "event": "logout"}
```

Read it one line at a time:

```python
import json

with open("events.jsonl", "r", encoding="utf-8") as file:
    for line in file:
        if line.strip():
            event = json.loads(line)
            print(event["event_id"], event["event"])
```

JSONL is useful for event streams, logs, and incremental processing. It is different from a standard JSON document containing a single array of objects.

### PySpark Example

For larger datasets, PySpark can read CSV and JSON directly.

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("FileIngestion").getOrCreate()

csv_df = (
    spark.read
    .option("header", True)
    .option("inferSchema", False)
    .csv("data/employees.csv")
)

json_df = spark.read.json("data/employees.json")

csv_df.show()
json_df.show()
```

Important considerations:

- CSV fields may be read as strings unless a schema or inference is used.
- JSON records can contain nested structures that Spark represents using nested columns.
- Providing an explicit schema is often preferable for repeatable production pipelines.
- Schema inference can add overhead and may infer an unsuitable type when data is inconsistent.
- For newline-delimited JSON, Spark's default JSON reader expects one self-contained JSON record per line. Reading a conventional multiline JSON document may require the `multiLine` option.

---

## 11. Common Mistakes

1. **Using `split(",")` to parse CSV:** Breaks when quoted fields contain commas.
2. **Assuming CSV values are numeric:** Values read through `csv.reader` or `csv.DictReader` are normally strings.
3. **Confusing `json.load()` with `json.loads()`:** One reads from a file-like object; the other parses a string.
4. **Using Python booleans in JSON text:** Valid JSON uses `true`, `false`, and `null`, not `True`, `False`, or `None`.
5. **Assuming nested JSON maps directly to CSV:** Nested objects and arrays must be flattened or represented in a different structure.
6. **Loading large files all at once:** `list(reader)` and `json.load()` can consume substantial memory.
7. **Ignoring malformed records:** Production pipelines need validation, error handling, and monitoring.
8. **Forgetting `newline=""` for CSV files:** Python's CSV documentation recommends this when opening files for CSV reading and writing.
9. **Overwriting files accidentally:** `"w"` truncates an existing file. Use `"a"` when appending is intended, with deliberate header handling.

---

## 12. Interview Questions

### Q1. What is the difference between CSV and JSON?

CSV is a flat, row-based format mainly suited for tabular data. JSON supports nested objects and arrays, making it suitable for structured API responses and event data.

### Q2. What is the difference between `csv.reader` and `csv.DictReader`?

`csv.reader` returns each row as a list. `csv.DictReader` maps each row to a dictionary using the header fields as keys.

### Q3. What is the difference between `json.load()` and `json.loads()`?

`json.load()` reads JSON from a file-like object. `json.loads()` parses a JSON-formatted string.

### Q4. Why does CSV-to-JSON conversion sometimes produce strings for numbers?

The standard CSV reader treats field values as strings. Explicit conversion or schema-based parsing is needed to produce numeric types.

### Q5. How do you handle nested JSON?

Parse the JSON into Python dictionaries and lists, then access nested values by key or index. In data pipelines, nested structures may be flattened or preserved as nested columns.

### Q6. What is JSON Lines?

JSON Lines stores one JSON value per line. It supports record-by-record processing and is commonly used for logs, events, and data ingestion.

### Q7. How would you process a very large CSV file?

Iterate over records instead of loading all rows into memory. For distributed processing, use tools such as PySpark and define an appropriate schema.

### Q8. How would you handle invalid JSON in an ingestion pipeline?

Catch parsing errors, record the relevant error details, and route invalid records to an error-handling process or quarantine location, depending on the pipeline's requirements.

### Q9. Why use an explicit schema in PySpark?

An explicit schema improves type consistency, avoids unreliable inference, and can reduce the overhead of determining column types.

### Q10. Can every JSON document be converted directly to CSV?

No. Nested objects, arrays, inconsistent fields, and complex data types require decisions about flattening, column structure, and how to represent values.

---

## 13. Practice Tasks

Try implementing these without copying the examples above.

1. Read `employees.csv` and calculate the average salary.
2. Read a CSV file and write only Engineering employees to a new CSV file.
3. Read a JSON file containing a list of orders and calculate the total order amount.
4. Extract all product names from a nested JSON order.
5. Convert a CSV file's `id` and `amount` columns to integers before writing JSON.
6. Read a JSONL file line by line and skip blank lines.
7. Handle a malformed JSON record without terminating the entire batch.
8. Read a CSV file with a semicolon delimiter.
9. Write a list of dictionaries to CSV with a specified column order.
10. Read a CSV file using PySpark with an explicit schema.

---

## 14. Quick Revision

- **CSV:** A tabular format with delimiter-separated fields.
- **JSON:** A structured format supporting objects, arrays, and primitive values.
- **`csv.reader()`:** Returns rows as lists.
- **`csv.DictReader()`:** Returns rows as dictionaries.
- **`csv.writer()`:** Writes list-based rows.
- **`csv.DictWriter()`:** Writes dictionary-based rows.
- **`json.load()`:** Reads JSON from a file-like object.
- **`json.loads()`:** Parses a JSON string.
- **`json.dump()`:** Writes a Python object as JSON to a file.
- **`json.dumps()`:** Converts a Python object to a JSON string.
- **JSONL:** One JSON value per line, useful for record-by-record processing.
- **Data engineering priority:** Validate records, handle types correctly, process large files efficiently, and define schemas deliberately.

**Most important interview takeaway:** Know how to read, write, validate, and convert CSV and JSON; understand their data-type differences; and explain how you would handle nested records, malformed data, and large files in a production pipeline.