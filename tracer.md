# Python Tracing

## Introduction

Tracing in Python refers to techniques for monitoring and logging the execution flow of a program. It helps developers understand code behavior, diagnose issues, measure coverage, and analyze performance without halting execution like a debugger. Tracing can track function calls, line executions, returns, exceptions, and more.

Key built-in tools include:
- The `trace` module: For statement-level tracing, coverage reports, and function call listings.
- `sys.settrace()`: A low-level hook for custom trace functions, often used in debuggers, profilers, and coverage tools.

Tracing is useful for:
- Debugging complex flows (e.g., loops or recursions).
- Generating code coverage reports.
- Profiling call stacks without full profiling overhead.

Note: Tracing adds overhead, so use it judiciously in production.

## Getting Started with the Trace Module

The `trace` module provides command-line and programmatic ways to trace execution. It can display lines as executed, count executions, list functions, or track calls.

### Command-Line Usage

Run a script with tracing:
```bash
python -m trace --trace your_script.py
```
This prints each line as it's executed.

Example script (`trace_example.py`):
```python
def add(a, b):
    result = a + b
    return result

print(add(3, 5))
```

Running `python -m trace --trace trace_example.py` might output:
```
 --- modulename: trace_example, funcname: <module>
trace_example.py(1): def add(a, b):
trace_example.py(5): print(add(3, 5))
 --- modulename: trace_example, funcname: add
trace_example.py(2):     result = a + b
trace_example.py(3):     return result
8
```

Other options:
- `--count`: Generate coverage files (e.g., `.cover` files showing execution counts).
- `--listfuncs`: List functions called.
- `--ignore-dir=<dir>`: Ignore modules in specified directories.

For full options, see the [trace module documentation](https://docs.python.org/3/library/trace.html).

## Programmatic Usage with Trace Class

Use the `trace.Trace` class for in-code tracing.

Basic example:
```python
import trace

# Define a function to trace
def multiply(x, y):
    return x * y

# Create tracer with line tracing enabled
tracer = trace.Trace(trace=True, count=False)

# Run a statement
tracer.run('result = multiply(4, 5); print(result)')

# Output might show execution lines similar to command-line
```

For functions:
```python
import trace

def multiply(x, y):
    return x * y

tracer = trace.Trace(trace=True, count=False)
result = tracer.runfunc(multiply, 4, 5)
print(result)  # Outputs: 20
# Tracing output would show lines executed in multiply
```

To gather coverage:
```python
import trace

def multiply(x, y):
    return x * y

tracer = trace.Trace(count=True, trace=False)
tracer.run('multiply(4, 5)')

# Get and write results
results = tracer.results()
results.write_results(show_missing=True, coverdir=".")
# Creates a .cover file with execution counts
```

## Tracing with sys.settrace

For custom tracing, use `sys.settrace(tracefunc)`, which installs a global trace function called on events like function calls, line executions, returns, and exceptions.

The trace function signature: `def tracefunc(frame, event, arg)`

- `frame`: Current stack frame.
- `event`: String like 'call', 'line', 'return', 'exception'.
- `arg`: Event-specific (e.g., exception tuple for 'exception').

Example: Simple call tracer
```python
import sys

def trace_calls(frame, event, arg):
    if event == 'call':
        print(f"Calling function: {frame.f_code.co_name}")
    return trace_calls  # Return itself for local tracing

# Set the trace
sys.settrace(trace_calls)

def foo():
    print("Inside foo")

def bar():
    foo()

bar()

# Disable tracing
sys.settrace(None)
```

Output:
```
Calling function: bar
Calling function: foo
Inside foo
```

Advanced example: Line tracer
```python
import sys

def trace_lines(frame, event, arg):
    if event == 'line':
        print(f"Executing line {frame.f_lineno} in {frame.f_code.co_filename}")
    return trace_lines

sys.settrace(trace_lines)

print("Line 1")
print("Line 2")

sys.settrace(None)
```

Tips:
- Return the trace function from 'call' events for per-frame tracing.
- Use `frame.f_trace_lines = False` to disable line events.
- Avoid infinite recursion; tracing is suspended during tracefunc calls.

## Advanced Features

- **Ignoring Modules**: In `trace.Trace(ignoremods=['module1', 'module2'])` or `--ignore-module` in CLI.
- **Timing**: Add `--timing` or `timing=True` to prefix lines with timestamps.
- **Coverage Merging**: Use `CoverageResults.update(other)` to combine results from multiple runs.
- **Custom Events**: With `sys.settrace`, handle 'opcode' for bytecode-level tracing (enable with `frame.f_trace_opcodes = True`).
- **Threading**: Use `threading.settrace()` for multi-threaded apps.

Post-mortem tracing: Combine with `sys.exc_info()` after exceptions.

## Alternatives to Built-in Tracing

- **logging Module**: For custom traces with levels (debug, info). Example: `logging.debug("Function called: %s", func_name)`.
- **Hunter**: Third-party library for selective tracing. Install via pip; usage: `import hunter; hunter.trace(module='your_module')`.
- **Py-Spy**: Sampling profiler/tracer for live processes (no code changes needed).
- **Coverage.py**: Advanced code coverage tool, integrates with pytest.
- **tracemalloc**: For memory allocation tracing (built-in since Python 3.4).

For memory tracing example:
```python
import tracemalloc

tracemalloc.start()
# Your code here
snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics('lineno')
for stat in top_stats[:5]:
    print(stat)
tracemalloc.stop()
```

## Best Practices

- Start with simple traces (e.g., function calls) before line-level.
- Use filters to ignore standard libraries and focus on your code.
- Combine with profiling (e.g., cProfile) for performance insights.
- Avoid in production; use logging instead for minimal overhead.
- Test traces on small code snippets first.
- For large apps, export results to files for analysis.

## Resources

- Official Docs: [trace Module](https://docs.python.org/3/library/trace.html)
- Official Docs: [sys.settrace](https://docs.python.org/3/library/sys.html#sys.settrace)
- Tutorials: Real Python's guide on Python tracing or Stack Overflow threads on sys.settrace.
- Libraries: Hunter on PyPI, py-spy on GitHub.