# Python Debugger (PDB)

## Introduction

The Python Debugger, commonly known as `pdb`, is a built-in module in Python's standard library that allows you to debug your code interactively. It provides a way to pause execution, inspect variables, step through code line by line, and diagnose issues efficiently. PDB is essential for Python engineers to troubleshoot bugs in scripts, applications, or even during development.

Key features:
- Interactive shell for debugging.
- Breakpoints to halt execution at specific points.
- Commands to navigate code flow (step in, step over, continue).
- Variable inspection and modification.

PDB is invoked via the `pdb` module or through command-line options. It's lightweight and doesn't require external dependencies.

## Getting Started with PDB

To use PDB, import it in your code and set a breakpoint using `pdb.set_trace()`. When execution reaches this point, it enters the debugger mode.

### Basic Example: Debugging a Simple Function

Consider this buggy script (`debug_example.py`):

```python
def add_numbers(a, b):
    result = a + b  # Intentional bug: should be multiplication for demo
    return result

def main():
    x = 5
    y = 10
    total = add_numbers(x, y)
    print(f"Total: {total}")

if __name__ == "__main__":
    main()
```

To debug, add a breakpoint:

```python
def add_numbers(a, b):
    breakpoint()
    result = a + b
    return result

def main():
    x = 5
    y = 10
    total = add_numbers(x, y)
    print(f"Total: {total}")

if __name__ == "__main__":
    main()
```

or 

```python
import pdb

def add_numbers(a, b):
    pdb.set_trace()  # Breakpoint here
    result = a + b
    return result

def main():
    x = 5
    y = 10
    total = add_numbers(x, y)
    print(f"Total: {total}")

if __name__ == "__main__":
    main()
```


Run the script: `python debug_example.py`

You'll enter the PDB shell at the breakpoint:

```
> debug_example.py(5)add_numbers()
-> result = a + b
(Pdb)
```

Common commands:
- `l` or `list`: Show surrounding code.
- `p <var>`: Print variable value (e.g., `p a` shows 5).
- `n` (next): Step over the current line.
- `s` (step): Step into a function call.
- `c` (continue): Resume until next breakpoint.
- `q` (quit): Exit debugger.

In this session:
- Type `p a` → 5
- Type `p b` → 10
- Type `n` to execute the addition.
- Type `p result` → 15 (spot the bug if expecting multiplication).
- Fix by changing to `result = a * b` (you can evaluate expressions in PDB).

## Setting Breakpoints

Breakpoints pause execution. Besides `pdb.set_trace()`, use:
- Conditional breakpoints: `break <line>, <condition>` in PDB shell.
- Command-line: `python -m pdb script.py` to start debugging from the beginning.

### Example: Conditional Breakpoint

Update the script to loop:

```python
import pdb

def process_numbers(numbers):
    for num in numbers:
        if num % 2 == 0:
            print(f"Even: {num}")
        else:
            print(f"Odd: {num}")

def main():
    nums = [1, 2, 3, 4, 5]
    process_numbers(nums)

if __name__ == "__main__":
    main()
```

Run with `python -m pdb script.py`. In PDB:
- `b process_numbers:4, num > 3` (breakpoint at line 4 if num > 3).
- `c` to continue – it will stop only when num=4 and 5.

## Post-Mortem Debugging

For crashes, use post-mortem mode to inspect after an exception.

Example script that raises an error:

```python
def divide(a, b):
    return a / b

def main():
    result = divide(10, 0)  # ZeroDivisionError
    print(result)

if __name__ == "__main__":
    main()
```

Run normally, it crashes. To debug post-mortem:
- Wrap in try-except and call `pdb.pm()`:

```python
import pdb

def divide(a, b):
    return a / b

def main():
    try:
        result = divide(10, 0)
        print(result)
    except:
        pdb.pm()  # Enter post-mortem on exception

if __name__ == "__main__":
    main()
```

Now, on error, enter PDB at the exception point:
- Use `up`/`down` to navigate stack frames.
- `p a` and `p b` to see values (10 and 0).

## Advanced Features

- **Watch Variables**: Use `display <var>` to auto-print on each step.
- **Execute Code**: Run any Python statement in the shell (e.g., `!result = 42` to modify variables).
- **Integration with IDEs**: Tools like VS Code or PyCharm have built-in PDB support for graphical debugging.
- **Configuration**: Create a `.pdbrc` file for aliases or default commands.

### Example: Modifying Variables Mid-Debug

In the first example, at breakpoint:
- `p result` (after execution: 15)
- `!result = a * b` (override to 50)
- `c` to continue – prints "Total: 50"

## Alternatives to PDB

While PDB is standard, consider:
- **ipdb**: IPython-enhanced PDB with tab-completion and syntax highlighting. Install: `pip install ipdb`. Use `ipdb.set_trace()`.
- **pudb**: Console-based visual debugger with a UI. Install: `pip install pudb`. Use `pudb.set_trace()`.
- **web-pdb**: For remote debugging over WebSockets.

For example with ipdb:
```python
import ipdb

def add_numbers(a, b):
    ipdb.set_trace()
    result = a + b
    return result
```
It offers richer inspection, like `?` for help.

## Best Practices

- Use breakpoints sparingly to avoid clutter.
- Combine with logging for production.
- Test edge cases in debugger sessions.
- For large apps, profile first (e.g., with cProfile) then debug hotspots.

## Resources

- Official Docs: [Python PDB Documentation](https://docs.python.org/3/library/pdb.html)
- Tutorials: Real Python's PDB guide or Stack Overflow threads.