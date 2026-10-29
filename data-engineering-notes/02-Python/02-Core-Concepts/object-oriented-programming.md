# Object-Oriented Programming (OOP) in Python

## 1. Overview

Object-Oriented Programming (OOP) is a programming approach that organizes code around **objects**. An object combines data (attributes) and behavior (methods).

Python supports OOP, which helps organize large programs into reusable, maintainable components.

OOP is useful for:
- Modeling real-world entities.
- Organizing application logic.
- Reusing code through inheritance and composition.
- Controlling access to data through well-defined interfaces.
- Building maintainable applications and data-processing systems.

The four commonly discussed pillars of OOP are:

1. Encapsulation
2. Abstraction
3. Inheritance
4. Polymorphism

---

## 2. Classes and Objects

### What is a class?

A **class** is a definition that describes the attributes and behavior its instances can have.

### What is an object?

An **object** is an instance of a class.

For example, `Employee` can be a class, while two employees with different names and salaries can be separate objects of that class.

### Example

```python
class Employee:
    def display(self):
        print("Employee details")


emp1 = Employee()
emp2 = Employee()

emp1.display()
emp2.display()
```

Output:

```text
Employee details
Employee details
```

Explanation:

- `class Employee:` defines the class.
- `Employee()` creates an instance.
- `emp1` and `emp2` refer to separate instances.
- `display()` is a method defined by the class.

### Attributes and methods

- **Attributes** store data associated with an object or class.
- **Methods** define behavior using functions associated with a class.

---

## 3. The `__init__()` Method and `self`

### What is `__init__()`?

`__init__()` is an initializer that Python calls after a new instance has been created. It is commonly used to initialize instance attributes.

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

    def display(self):
        print(self.name, self.salary)


emp1 = Employee("Amit", 50000)
emp2 = Employee("Priya", 65000)

emp1.display()
emp2.display()
```

Output:

```text
Amit 50000
Priya 65000
```

### How does it work?

When this statement executes:

```python
emp1 = Employee("Amit", 50000)
```

Python creates an instance and invokes its initializer with that instance and the supplied arguments.

Inside `__init__()`:

```python
self.name = name
self.salary = salary
```

- `self` refers to the current instance.
- `self.name` stores an attribute on that instance.
- `name` is the argument passed to the initializer.

### Is `self` a keyword?

No. `self` is a naming convention, not a Python keyword. It is the conventional name for the first parameter of an instance method.

This is valid Python, although not recommended:

```python
class Example:
    def show(current_object):
        print("Hello")
```

Using `self` makes the code easier for other Python developers to understand.

### Important distinction

`__init__()` is not technically the constructor that creates the object. `__new__()` is responsible for creating the instance; `__init__()` initializes it.

For most everyday Python programming, you will work primarily with `__init__()`.

---

## 4. Instance Attributes vs. Class Attributes

### Instance attributes

Instance attributes belong to individual objects. Different instances can hold different values.

### Class attributes

Class attributes are defined on the class and are available through the class and its instances, unless an instance attribute of the same name shadows them.

```python
class Employee:
    company = "TCS"

    def __init__(self, name):
        self.name = name


emp1 = Employee("Amit")
emp2 = Employee("Priya")

print(emp1.name)
print(emp2.name)

print(emp1.company)
print(Employee.company)
```

Output:

```text
Amit
Priya
TCS
TCS
```

Here:

- `name` is an instance attribute.
- `company` is a class attribute.

### Changing a class attribute

```python
class Employee:
    company = "TCS"


emp1 = Employee()
emp2 = Employee()

Employee.company = "Microsoft"

print(emp1.company)
print(emp2.company)
```

Output:

```text
Microsoft
Microsoft
```

Both instances read the updated class attribute because neither has its own `company` attribute.

### Shadowing a class attribute

```python
class Employee:
    company = "TCS"


emp1 = Employee()
emp2 = Employee()

emp1.company = "Microsoft"

print(emp1.company)
print(emp2.company)
print(Employee.company)
```

Output:

```text
Microsoft
TCS
TCS
```

Assigning `emp1.company` creates an instance attribute that shadows the class attribute. It does not change `Employee.company`.

**Common mistake:** A mutable class attribute, such as a list, can be shared by every instance that accesses it. Use instance attributes when each object needs its own mutable collection.

---

## 5. The Four Pillars of OOP

## 5.1 Encapsulation

Encapsulation means grouping data and related behavior together and providing a controlled interface for interacting with that data.

Python does not enforce private instance variables in the same way as some other languages. Instead, it uses naming conventions and features such as properties.

### Example

```python
class BankAccount:
    def __init__(self, balance):
        self._balance = balance

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Amount must be positive")

        self._balance += amount

    @property
    def balance(self):
        return self._balance


account = BankAccount(1000)

account.deposit(500)

print(account.balance)
```

Output:

```text
1500
```

Explanation:

- `_balance` signals that the attribute is intended for internal use.
- `deposit()` validates the amount before changing the balance.
- `@property` exposes the balance through attribute-style access.

Python conventions:

- `name`: public attribute.
- `_name`: non-public by convention.
- `__name`: name-mangled attribute intended to reduce accidental clashes in subclasses; it is not truly private.

Encapsulation helps protect an object's internal state from unintended changes.

## 5.2 Abstraction

Abstraction means exposing essential operations while hiding unnecessary implementation details.

For example, a user can call a method to process a data file without needing to know every internal step.

Python supports abstraction using abstract base classes from the `abc` module.

### Example

```python
from abc import ABC, abstractmethod


class DataProcessor(ABC):
    @abstractmethod
    def process(self, data):
        pass


class UppercaseProcessor(DataProcessor):
    def process(self, data):
        return [value.upper() for value in data]


processor = UppercaseProcessor()

print(processor.process(["python", "sql"]))
```

Output:

```text
['PYTHON', 'SQL']
```

Explanation:

- `DataProcessor` defines a required interface.
- `@abstractmethod` requires concrete subclasses to implement `process()`.
- `UppercaseProcessor` provides the implementation.

You cannot normally instantiate `DataProcessor` directly while it has an unimplemented abstract method.

**Encapsulation vs. abstraction:**

- Encapsulation focuses on grouping state and behavior and controlling access.
- Abstraction focuses on exposing essential functionality while hiding implementation details.

## 5.3 Inheritance

Inheritance allows a class to derive behavior and attributes from another class.

- **Parent class:** The class being inherited from.
- **Child class:** The class that inherits from the parent.

### Example

```python
class Employee:
    def __init__(self, name):
        self.name = name

    def show_name(self):
        print(self.name)


class DataEngineer(Employee):
    def build_pipeline(self):
        print("Building data pipeline")


engineer = DataEngineer("Amit")

engineer.show_name()
engineer.build_pipeline()
```

Output:

```text
Amit
Building data pipeline
```

`DataEngineer` inherits `show_name()` and uses its own `build_pipeline()` method.

### The `super()` function

`super()` provides access to methods in the method resolution order, commonly to initialize or extend parent-class behavior.

```python
class Employee:
    def __init__(self, name):
        self.name = name


class DataEngineer(Employee):
    def __init__(self, name, tool):
        super().__init__(name)
        self.tool = tool


engineer = DataEngineer("Amit", "PySpark")

print(engineer.name)
print(engineer.tool)
```

Output:

```text
Amit
PySpark
```

`super().__init__(name)` reuses the parent initializer instead of duplicating its logic.

### Types of inheritance

- **Single inheritance:** One child inherits from one parent.
- **Multilevel inheritance:** A class inherits from a class that itself inherits from another.
- **Hierarchical inheritance:** Multiple child classes inherit from the same parent.
- **Multiple inheritance:** One child inherits from multiple parent classes.
- **Hybrid inheritance:** A combination of inheritance patterns.

Python supports multiple inheritance. When methods are found in multiple parent classes, Python follows the **Method Resolution Order (MRO)** to determine which implementation is used.

## 5.4 Polymorphism

Polymorphism means that the same interface or operation can work with different types of objects, with each providing its own behavior.

### Example: Method overriding

```python
class CSVProcessor:
    def process(self):
        print("Processing CSV data")


class JSONProcessor:
    def process(self):
        print("Processing JSON data")


processors = [CSVProcessor(), JSONProcessor()]

for processor in processors:
    processor.process()
```

Output:

```text
Processing CSV data
Processing JSON data
```

Both objects support `process()`, but each implements it differently.

### Duck typing

Python often relies on duck typing: if an object supports the operations your code needs, the code can use it without requiring a specific class or inheritance relationship.

```python
class CSVProcessor:
    def process(self):
        print("CSV")


class ParquetProcessor:
    def process(self):
        print("Parquet")


def run_processor(processor):
    processor.process()


run_processor(CSVProcessor())
run_processor(ParquetProcessor())
```

Output:

```text
CSV
Parquet
```

`run_processor()` works with both objects because each provides a callable `process()` method.

### Method overriding vs. overloading

- **Method overriding:** A subclass provides its own implementation of an inherited method.
- **Method overloading:** Traditionally, defining multiple methods with the same name but different parameter signatures.

Python supports overriding directly. It does not support traditional signature-based method overloading in the same way as languages such as Java. Defining the same method name repeatedly in a class generally replaces the earlier definition.

Similar behavior can often be implemented with default arguments, `*args`, `**kwargs`, or `functools.singledispatch`.

---

## 6. Instance, Class, and Static Methods

Python supports three common method styles.

### Instance method

Works with an individual instance and conventionally receives `self`.

```python
class Employee:
    def __init__(self, name):
        self.name = name

    def show_name(self):
        return self.name
```

### Class method

Uses the `@classmethod` decorator and conventionally receives `cls`, which refers to the class.

```python
class Employee:
    company = "TCS"

    @classmethod
    def show_company(cls):
        return cls.company


print(Employee.show_company())
```

Output:

```text
TCS
```

Class methods are often useful for alternative constructors or operations involving class-level state.

### Static method

Uses `@staticmethod` and does not automatically receive `self` or `cls`.

```python
class DataUtils:
    @staticmethod
    def is_valid_amount(amount):
        return amount > 0


print(DataUtils.is_valid_amount(100))
```

Output:

```text
True
```

A static method is suitable for a utility function logically associated with a class but not dependent on instance or class state.

### Comparison

| Method type | First parameter by convention | Typical purpose |
|---|---|---|
| Instance method | `self` | Work with instance data |
| Class method | `cls` | Work with class state or create instances |
| Static method | None automatically | Related utility logic |

---

## 7. Special Methods (Dunder Methods)

Special methods, often called **dunder methods**, have names surrounded by double underscores. They let custom objects integrate with Python's built-in operations.

Examples include:

| Method | Purpose |
|---|---|
| `__init__()` | Initialize an instance |
| `__str__()` | Provide a user-friendly string representation |
| `__repr__()` | Provide a representation useful for debugging |
| `__len__()` | Support `len(object)` |
| `__eq__()` | Define equality behavior |
| `__add__()` | Define behavior for the `+` operator |

### Example: `__str__()`

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

    def __str__(self):
        return f"{self.name}: {self.salary}"


employee = Employee("Amit", 50000)

print(employee)
```

Output:

```text
Amit: 50000
```

When `print()` converts the object to a string, Python uses `__str__()` if available.

You do not need to implement every special method. Define them when they make your custom class easier and more natural to use.

---

## 8. Composition vs. Inheritance

Inheritance represents an **is-a** relationship. Composition represents a **has-a** relationship.

### Inheritance example

A `DataEngineer` is an `Employee`.

```python
class Employee:
    pass


class DataEngineer(Employee):
    pass
```

### Composition example

A `DataPipeline` has a `DataValidator`.

```python
class DataValidator:
    def validate(self, data):
        return [row for row in data if row["id"] is not None]


class DataPipeline:
    def __init__(self, validator):
        self.validator = validator

    def run(self, data):
        return self.validator.validate(data)


validator = DataValidator()
pipeline = DataPipeline(validator)

records = [
    {"id": 1},
    {"id": None},
    {"id": 3}
]

print(pipeline.run(records))
```

Output:

```text
[{'id': 1}, {'id': 3}]
```

`DataPipeline` uses a `DataValidator` instead of inheriting from it.

**When to use which:**

- Use inheritance when a genuine subtype relationship exists and the child can appropriately substitute for the parent.
- Use composition when a class needs to use another component's functionality.
- Composition often makes it easier to replace components without creating complicated inheritance hierarchies.

---

## 9. Common OOP Mistakes

### Mistake 1: Forgetting `self`

Incorrect:

```python
class Employee:
    def show_name():
        print("Amit")
```

Calling `Employee().show_name()` raises `TypeError` because Python passes the instance automatically, but the method has no parameter to receive it.

Correct:

```python
class Employee:
    def show_name(self):
        print("Amit")
```

### Mistake 2: Confusing class and instance attributes

Changing an instance attribute does not automatically change the class attribute or other instances.

### Mistake 3: Using a mutable class attribute unintentionally

Incorrect when every instance needs its own list:

```python
class Team:
    members = []

    def add_member(self, name):
        self.members.append(name)
```

All instances that access the shared class list will modify the same list.

Better:

```python
class Team:
    def __init__(self):
        self.members = []

    def add_member(self, name):
        self.members.append(name)
```

### Mistake 4: Overusing inheritance

Do not create inheritance relationships only to reuse a few lines of code. A helper function or composed component may be simpler.

### Mistake 5: Assuming underscore attributes are truly private

A single underscore is a convention. Double-leading underscores trigger name mangling, but neither provides absolute security.

### Mistake 6: Forgetting to call the parent initializer

When a child defines its own `__init__()`, the parent initializer is not automatically invoked. Use `super().__init__(...)` when the parent initialization is needed.

---

## 10. Practical Data Engineering Example

Consider a pipeline that reads data, validates it, and transforms it.

```python
class DataValidator:
    def validate(self, records):
        return [
            record for record in records
            if record.get("order_id") is not None
        ]


class DataTransformer:
    def transform(self, records):
        return [
            {
                **record,
                "amount": float(record["amount"])
            }
            for record in records
        ]


class DataPipeline:
    def __init__(self, validator, transformer):
        self.validator = validator
        self.transformer = transformer

    def run(self, records):
        valid_records = self.validator.validate(records)
        return self.transformer.transform(valid_records)


records = [
    {"order_id": 1, "amount": "100.5"},
    {"order_id": None, "amount": "200"},
    {"order_id": 3, "amount": "300.0"}
]

pipeline = DataPipeline(
    DataValidator(),
    DataTransformer()
)

print(pipeline.run(records))
```

Output:

```text
[
    {'order_id': 1, 'amount': 100.5},
    {'order_id': 3, 'amount': 300.0}
]
```

This example demonstrates:

- **Classes and objects:** The pipeline and its components are objects.
- **Encapsulation:** Each component groups related behavior.
- **Composition:** The pipeline uses validator and transformer objects.
- **Abstraction:** `run()` exposes the overall operation without requiring the caller to invoke each step separately.

This is a small Python example for learning OOP. For large datasets, production pipelines commonly use SQL, PySpark DataFrames, or other data-processing engines rather than processing all records as Python lists.

---

## 11. Interview Questions

### Q1. What is OOP?

OOP is a programming paradigm that organizes code around objects containing data and behavior.

### Q2. What is the difference between a class and an object?

A class defines the structure and behavior; an object is an instance of that class.

### Q3. What is `self`?

`self` conventionally refers to the current instance and allows instance methods to access its attributes and methods.

### Q4. What is the purpose of `__init__()`?

It initializes a newly created instance. The object itself is created by `__new__()`.

### Q5. Explain the four pillars of OOP.

- Encapsulation: group data and behavior and control access.
- Abstraction: expose essential functionality while hiding implementation details.
- Inheritance: derive a class from another class.
- Polymorphism: use a common interface with different implementations.

### Q6. What is method overriding?

It occurs when a subclass provides its own implementation of a method inherited from a parent class.

### Q7. What is the difference between `@classmethod` and `@staticmethod`?

A class method receives `cls` automatically; a static method receives neither `self` nor `cls` automatically.

### Q8. Does Python support multiple inheritance?

Yes. Python supports inheriting from multiple parent classes and uses the Method Resolution Order to resolve method lookup.

### Q9. What is duck typing?

Duck typing means code can work with an object based on the operations it supports, rather than requiring a specific class.

### Q10. What is the difference between composition and inheritance?

Inheritance models an is-a relationship; composition models a has-a relationship.

### Q11. What is the difference between an instance attribute and a class attribute?

An instance attribute belongs to a particular object. A class attribute is defined on the class and can be shared by its instances unless shadowed.

### Q12. Does Python have private variables?

Python mainly relies on conventions. `_name` indicates non-public use, while `__name` invokes name mangling to reduce accidental name conflicts.

### Q13. What is the difference between `__str__()` and `__repr__()`?

`__str__()` aims to provide a readable string for users; `__repr__()` aims to provide an informative representation useful for debugging and development.

### Q14. Why is composition often preferred over complex inheritance?

Composition allows behavior to be combined from separate components, which can make systems easier to change, test, and maintain.

---

## 12. Practice Questions

Try to solve these without looking at the solutions first.

### Easy

1. Create an `Employee` class with `name` and `salary` attributes.
2. Add a method that prints the employee's details.
3. Create two instances with different values.
4. Create a class attribute named `company`.
5. Explain the difference between `self.name` and `Employee.company`.

### Medium

6. Create a `DataEngineer` class that inherits from `Employee` and adds a `tool` attribute.
7. Use `super()` to initialize the parent class.
8. Create two classes with a common `process()` method but different implementations.
9. Create a class with a static method that checks whether a number is positive.
10. Create a pipeline class that uses a separate validator object through composition.

### Solutions

```python
# 1–3
class Employee:
    company = "TCS"

    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

    def display(self):
        print(self.name, self.salary)


emp1 = Employee("Amit", 50000)
emp2 = Employee("Priya", 65000)

emp1.display()
emp2.display()
```

```python
# 6–7
class DataEngineer(Employee):
    def __init__(self, name, salary, tool):
        super().__init__(name, salary)
        self.tool = tool


engineer = DataEngineer("Amit", 50000, "PySpark")
print(engineer.name, engineer.salary, engineer.tool)
```

```python
# 8
class CSVProcessor:
    def process(self):
        print("Processing CSV")


class JSONProcessor:
    def process(self):
        print("Processing JSON")


for processor in [CSVProcessor(), JSONProcessor()]:
    processor.process()
```

```python
# 9
class NumberUtils:
    @staticmethod
    def is_positive(number):
        return number > 0


print(NumberUtils.is_positive(10))
```

For questions 4, 5, and 10, focus on class-level state versus instance-level state, and on using one object as a component of another.

---

## 13. Quick Revision

- **Class:** Defines attributes and behavior.
- **Object:** An instance of a class.
- **`self`:** Conventionally refers to the current instance.
- **`__init__()`:** Initializes a newly created object.
- **Instance attributes:** Store per-object state.
- **Class attributes:** Store class-level state.
- **Encapsulation:** Organizes state and behavior with controlled access.
- **Abstraction:** Exposes essential operations while hiding implementation details.
- **Inheritance:** Reuses or extends behavior through a parent-child relationship.
- **Polymorphism:** Allows different objects to provide different behavior through a common interface.
- **`super()`:** Accesses behavior through the method resolution order.
- **`@classmethod`:** Receives `cls`.
- **`@staticmethod`:** Receives no automatic instance or class argument.
- **Composition:** Builds functionality by combining objects.
- **Duck typing:** Focuses on supported behavior rather than exact object type.

**Remember:** Classes define behavior, objects hold state, and the four pillars help organize that behavior into maintainable software.