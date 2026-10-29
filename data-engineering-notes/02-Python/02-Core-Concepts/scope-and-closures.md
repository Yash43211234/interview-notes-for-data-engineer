# Scope and Closures in Python

## 1. What Is Scope?

**Scope** defines where a variable can be accessed in a Python program.

Python determines how to find a variable using the **LEGB rule**.

### The LEGB Rule

Python searches for a variable in this order:

| Scope | Meaning | Example |
|---|---|---|
| **L — Local** | Inside the current function | Variable defined inside a function |
| **E — Enclosing** | Inside an outer function | Variable defined in a function surrounding the current function |
| **G — Global** | At the module level | Variable defined outside all functions |
| **B — Built-in** | Python's built-in names | `len()`, `print()`, `sum()` |

Python searches these scopes from left to right and uses the first matching variable it finds.

```python
x = "global"

def outer():
    x = "enclosing"

    def inner():
        x = "local"
        print(x)

    inner()

outer()
```

Output:

```text
local
```

The variable is found in the local scope of `inner()`, so Python does not search the outer scopes.

If `inner()` did not define `x`, Python would look in `outer()`, then the global scope, and finally the built-in scope.

---

## 2. Local Scope

A variable created inside a function is normally local to that function.

```python
def calculate():
    result = 100
    print(result)

calculate()
```

Output:

```text
100
```

The local variable `result` cannot normally be accessed outside the function.

```python
def calculate():
    result = 100

calculate()
print(result)
```

This raises:

```text
NameError: name 'result' is not defined
```

**Key point:** A function's local variables are available while executing that function, not automatically outside it.

---

## 3. Global Scope

A global variable is defined at the module level, outside functions and classes.

```python
company = "TCS"

def show_company():
    print(company)

show_company()
```

Output:

```text
TCS
```

The function can read the global variable because Python finds it in the global scope.

### Modifying a Global Variable

Reading a global variable does not require the `global` keyword. Assigning to it inside a function is different.

```python
count = 10

def update():
    count = 20
    print(count)

update()
print(count)
```

Output:

```text
20
10
```

The assignment `count = 20` creates a local variable named `count`. It does not modify the global variable.

To modify the global variable, use `global`.

```python
count = 10

def update():
    global count
    count = 20

update()
print(count)
```

Output:

```text
20
```

**Best practice:** Avoid modifying global variables unnecessarily. Passing values through function arguments and returning results usually makes code easier to test and maintain.

---

## 4. Enclosing Scope

The enclosing scope exists when one function is defined inside another function.

```python
def outer():
    message = "Hello from outer"

    def inner():
        print(message)

    inner()

outer()
```

Output:

```text
Hello from outer
```

The variable `message` belongs to `outer()`. Python finds it in the enclosing scope of `inner()`.

### Important Distinction

A nested function can read a variable from its enclosing function without any special keyword.

However, assigning to that variable inside the nested function normally creates a local binding.

```python
def outer():
    count = 10

    def inner():
        count = 20
        print(count)

    inner()
    print(count)

outer()
```

Output:

```text
20
10
```

The `count` inside `inner()` is local to `inner()`. It does not change the enclosing variable.

---

## 5. The `nonlocal` Keyword

Use `nonlocal` when a nested function needs to reassign a variable belonging to an enclosing function.

```python
def outer():
    count = 10

    def inner():
        nonlocal count
        count = 20

    inner()
    print(count)

outer()
```

Output:

```text
20
```

Here, `nonlocal count` tells Python to use the existing `count` in the nearest enclosing function scope rather than creating a new local variable.

### `global` vs. `nonlocal`

| Keyword | Refers to |
|---|---|
| `global` | A variable in the module-level global scope |
| `nonlocal` | An existing variable in an enclosing function scope |

Example:

```python
value = 1

def outer():
    value = 2

    def inner():
        nonlocal value
        value = 3

    inner()
    print(value)

outer()
print(value)
```

Output:

```text
3
1
```

The nested function modifies the enclosing function's `value`, not the global `value`.

**Important:** `nonlocal` cannot refer to a variable that exists only in the global scope. Use `global` for that purpose.

---

## 6. How Assignment Affects Scope

Python determines whether a name is local based on assignments within a function, unless a scope declaration such as `global` or `nonlocal` changes that behavior.

Consider:

```python
x = 10

def show():
    print(x)
    x = 20

show()
```

This raises:

```text
UnboundLocalError
```

Why?

1. Python sees an assignment to `x` inside `show()`.
2. It treats `x` as a local variable throughout that function.
3. `print(x)` tries to read the local variable before it has been assigned a value.

To read the global variable and then modify it:

```python
x = 10

def show():
    global x
    print(x)
    x = 20

show()
```

Output:

```text
10
```

After the function runs, the global `x` is `20`.

**Interview point:** `UnboundLocalError` is a subclass of `NameError`. It commonly occurs when a local variable is referenced before assignment.

---

## 7. What Is a Closure?

A **closure** is a function that remembers and can access variables from its enclosing scope even after the enclosing function has finished executing.

Closures are useful when you want to retain state without using a global variable or creating a class.

### Basic Example

```python
def outer(message):
    def inner():
        print(message)

    return inner

greet = outer("Hello")
greet()
```

Output:

```text
Hello
```

What happens?

1. `outer("Hello")` receives the value `"Hello"`.
2. `outer()` defines the nested function `inner()`.
3. `outer()` returns `inner` without calling it.
4. The variable `greet` now refers to the returned function.
5. Calling `greet()` prints `"Hello"`.

Although `outer()` has finished executing, `inner()` still has access to the enclosing variable `message`.

That retained access is the central idea behind a closure.

### Function vs. Function Call

```python
return inner
```

Returns the function itself.

```python
return inner()
```

Calls the function immediately and returns its result.

This distinction is essential when creating closures.

---

## 8. Closures Can Preserve State

A closure can maintain a value between multiple function calls.

```python
def make_counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment

counter = make_counter()

print(counter())
print(counter())
print(counter())
```

Output:

```text
1
2
3
```

The `count` variable remains available to `increment()` between calls.

Without `nonlocal`, the statement `count += 1` would attempt to assign to a local variable and raise `UnboundLocalError`.

### Independent Closure Instances

Each call to `make_counter()` creates a separate enclosing scope.

```python
counter1 = make_counter()
counter2 = make_counter()

print(counter1())
print(counter1())
print(counter2())
```

Output:

```text
1
2
1
```

`counter1` and `counter2` maintain independent state.

**Use cases:**
- Counters
- Configuration-based functions
- Simple stateful callbacks
- Function factories
- Decorators

---

## 9. Closures and `__closure__`

Python provides a way to inspect the variables captured by a closure.

```python
def outer():
    message = "Hello"

    def inner():
        print(message)

    return inner

func = outer()

print(func.__closure__)
```

`func.__closure__` contains closure cells for variables captured from enclosing scopes. Each cell's value can be inspected using `.cell_contents`.

```python
print(func.__closure__[0].cell_contents)
```

Output:

```text
Hello
```

The exact ordering and number of closure cells depend on the variables captured by the function.

This feature is useful for understanding closures, but most application code does not need to inspect `__closure__` directly.

---

## 10. Common Closure Pitfall: Late Binding

Closures generally look up captured variables when the inner function executes, not necessarily when it is created.

This can cause unexpected behavior when functions are created inside a loop.

```python
functions = []

for i in range(3):
    functions.append(lambda: i)

print(functions[0]())
print(functions[1]())
print(functions[2]())
```

Output:

```text
2
2
2
```

Why?

Each lambda refers to the same variable `i`. By the time the functions are called, the loop has finished and `i` is `2`.

### Solution 1: Use a Default Argument

```python
functions = []

for i in range(3):
    functions.append(lambda i=i: i)

print(functions[0]())
print(functions[1]())
print(functions[2]())
```

Output:

```text
0
1
2
```

The default argument captures the current value of `i` when the lambda is created.

### Solution 2: Use a Function Factory

```python
def make_function(value):
    def inner():
        return value
    return inner

functions = [make_function(i) for i in range(3)]

print([func() for func in functions])
```

Output:

```text
[0, 1, 2]
```

Each call to `make_function()` creates a separate enclosing scope with its own `value`.

**Interview point:** The default-argument approach is a common solution to late binding in loops, particularly when creating callbacks or lambdas.

---

## 11. Closures vs. Classes

Closures are not always the best way to preserve state. A class can be clearer when the state or behavior becomes complex.

### Using a Closure

```python
def make_multiplier(factor):
    def multiply(number):
        return number * factor
    return multiply

double = make_multiplier(2)
print(double(5))
```

Output:

```text
10
```

### Using a Class

```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor

    def multiply(self, number):
        return number * self.factor

double = Multiplier(2)
print(double.multiply(5))
```

Output:

```text
10
```

Both approaches work.

- Use a closure for a small function that needs to retain a few values.
- Use a class when you need multiple methods, explicit state management, or a more structured object.

---

## 12. Practical Example: Creating Configurable Functions

Suppose a data pipeline needs to apply different tax rates to transaction amounts.

```python
def create_tax_calculator(rate):
    def calculate_tax(amount):
        return amount * rate

    return calculate_tax

gst_5 = create_tax_calculator(0.05)
gst_18 = create_tax_calculator(0.18)

print(gst_5(1000))
print(gst_18(1000))
```

Output:

```text
50.0
180.0
```

Each returned function retains its own `rate`.

This pattern is useful when building configurable transformations or reusable callbacks. In a real financial application, use appropriate decimal arithmetic and rounding rules instead of relying on binary floating-point values for exact monetary calculations.

---

## 13. Common Interview Questions

### Q1. What is the LEGB rule?

LEGB describes the order Python uses to resolve names: Local, Enclosing, Global, and Built-in.

### Q2. Can a function access a global variable?

Yes. A function can read a global variable if no local or enclosing binding takes precedence.

### Q3. What is the difference between `global` and `nonlocal`?

`global` refers to a module-level variable. `nonlocal` refers to an existing variable in an enclosing function scope.

### Q4. What is a closure?

A closure is a function that retains access to variables from its enclosing scope after that scope's function has returned.

### Q5. Why are closures useful?

They allow functions to preserve state, create configurable functions, and encapsulate values without using global variables.

### Q6. Why does `UnboundLocalError` occur?

It often occurs when a function assigns to a variable, causing Python to treat it as local, but tries to read it before the local assignment executes.

### Q7. Why do lambdas in a loop sometimes return the same value?

They may all reference the same loop variable, which has its final value when the functions execute. A default argument or function factory can capture separate values.

### Q8. Does a closure copy the enclosing variable's value?

Not necessarily. A closure retains access to the captured variable through a closure cell. If the enclosing variable is reassigned using `nonlocal`, the closure sees the updated value.

### Q9. Can a closure modify a captured variable?

Yes. Use `nonlocal` to reassign a variable from an enclosing function scope. If the captured object is mutable, a closure can also mutate that object without rebinding its name.

---

## 14. Practice Questions

Try answering these without running the code first.

### Question 1

```python
x = 5

def outer():
    x = 10

    def inner():
        print(x)

    inner()

outer()
```

What is the output?

### Question 2

```python
def outer():
    count = 0

    def inner():
        nonlocal count
        count += 1
        return count

    return inner

f = outer()
print(f())
print(f())
```

What is the output?

### Question 3

```python
x = 10

def test():
    x = 20
    print(x)

test()
print(x)
```

Why does the global variable remain unchanged?

### Question 4

```python
functions = []

for i in range(3):
    functions.append(lambda i=i: i)

print([f() for f in functions])
```

What problem does `i=i` solve?

### Question 5

```python
def outer():
    value = 10

    def inner():
        value = 20

    inner()
    print(value)

outer()
```

Does the enclosing `value` change? Why or why not?

---

## 15. Quick Revision

- **Scope:** The region in which a variable can be accessed.
- **LEGB:** Local → Enclosing → Global → Built-in.
- **Local variable:** Defined inside a function.
- **Global variable:** Defined at the module level.
- **Enclosing scope:** The scope of an outer function containing a nested function.
- **`global`:** Allows assignment to a module-level variable from inside a function.
- **`nonlocal`:** Allows reassignment of a variable from an enclosing function scope.
- **Closure:** A function that retains access to captured enclosing variables after the outer function returns.
- **Late binding:** A closure may use the variable's later value when it executes.
- **Default arguments:** Can capture the current value in a loop when creating functions.

**Most important interview takeaway:** Understand how Python resolves variable names, why assignment changes scope behavior, how closures preserve access to enclosing variables, and how `nonlocal` enables a closure to update captured state.