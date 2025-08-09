# Python Profilers

## Introduction

Profiling in Python is crucial for identifying performance bottlenecks, optimizing code, and understanding resource usage like CPU, GPU, memory, and time. This guide expands on general Python profilers (e.g., cProfile), dives into specialized tools like Torch Profiler for PyTorch-based machine learning, NVIDIA Nsight for GPU-accelerated applications, and visualization methods using Chrome Tracing or Perfetto. These tools are especially relevant for data science, AI, and high-performance computing workflows.

We'll cover usage, examples, and integration, drawing from official documentation and best practices as of August 2025. Code examples are tested where possible.

## General Python Profilers

Python's built-in profilers provide a foundation for analyzing code execution. Key tools include:

- **cProfile**: Deterministic profiler for function-level timing. Low overhead, sorts by cumulative time.
  - Usage: `import cProfile; cProfile.run('your_function()')`
  - Output: Table with `ncalls`, `tottime`, `cumtime`, etc.

- **line_profiler**: Line-by-line timing for granular analysis.
  - Install: `pip install line_profiler`
  - Usage: Decorate with `@profile`, run with `kernprof -l script.py`

- **memory_profiler**: Tracks memory usage per line.
  - Usage: `@profile def func(): ...`

For a simple example (tested via execution):
```python
import cProfile

def example():
    sum(i**2 for i in range(10000))

cProfile.run('example()')
```
Output excerpt:
```
   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.001    0.001    0.001    0.001 <string>:1(<module>)
        1    0.000    0.000    0.001    0.001 {built-in method builtins.exec}
```

These are great for pure Python but extend to ML with specialized tools.

## Torch Profiler for PyTorch

Torch Profiler is integrated into PyTorch (since 1.8+) for profiling deep learning models, capturing CPU/GPU operations, memory, and tensor shapes. It supports CUDA/XPU activities and exports traces for visualization. Key features: Record shapes, memory profiling, stack traces, and scheduling for long jobs.

### Basic Usage
Wrap code in a `profile` context:
```python
import torch
from torch.profiler import profile, ProfilerActivity

with profile(activities=[ProfilerActivity.CPU, ProfilerActivity.CUDA]) as prof:
    model(input)  # Your model inference/training

print(prof.key_averages().table(sort_by="cuda_time_total"))
```

Tested example with a simple linear model:
```python
import torch
import torch.nn as nn
from torch.profiler import profile, record_function, ProfilerActivity

model = nn.Sequential(nn.Linear(10, 10))
input = torch.randn(128, 10)

with profile(activities=[ProfilerActivity.CPU], record_shapes=True) as prof:
    with record_function("model_inference"):
        model(input)

print(prof.key_averages().table(sort_by="cpu_time_total", row_limit=5))
```
Output:
```
----------------------  ------------  ------------  ------------  ------------  ------------  ------------  
                  Name    Self CPU %      Self CPU   CPU total %     CPU total  CPU time avg    # of Calls  
----------------------  ------------  ------------  ------------  ------------  ------------  ------------  
       model_inference        45.84%     311.820us       100.00%     680.262us     680.262us             1  
          aten::linear         4.22%      28.717us        54.16%     368.442us     368.442us             1  
           aten::addmm        32.57%     221.575us        37.95%     258.131us     258.131us             1  
               aten::t         8.21%      55.833us        11.99%      81.594us      81.594us             1  
           aten::copy_         3.94%      26.802us         3.94%      26.802us      26.802us             1  
----------------------  ------------  ------------  ------------  ------------  ------------  ------------  
Self CPU time total: 680.262us
```

### Memory Profiling
Add `profile_memory=True` to track allocations.

### Exporting to Chrome Trace
```python
prof.export_chrome_trace("trace.json")
```
View in `chrome://tracing`.

### Advanced: Scheduling for Long Jobs
Use `schedule` for phased tracing (wait, warmup, active):
```python
from torch.profiler import schedule
my_schedule = schedule(skip_first=10, wait=5, warmup=1, active=3, repeat=2)
with profile(schedule=my_schedule, on_trace_ready=trace_handler) as prof:
    for _ in range(20):
        model(input)
        prof.step()
```

TensorBoard integration: Export via `prof.export_stacks()` or use TensorBoard plugin for PyTorch Profiler traces.

## NVIDIA Nsight for GPU Profiling

NVIDIA Nsight suite (Nsight Systems, Nsight Compute) profiles GPU workloads, including Python apps using CUDA/PyTorch. Nsight Systems is system-wide, tracing APIs, kernels, and Python backtraces.

### Key Features
- Traces CUDA, cuDNN, NVTX, Python GIL.
- Python sampling: Backtraces at frequency (1-2000 Hz).
- Overhead reduction: Disable CPU sampling for Python-only.

### Command-Line Usage
Profile a Python script:
```bash
nsys profile --trace=cuda,osrt,nvtx,python-gil --python-sampling=true --python-sampling-frequency=1000 python script.py
```
- `--python-backtrace=cuda`: Collect on CUDA events.
- Output: `.nsys-rep` file for GUI viewing.

For Jupyter: `nsys launch jupyter-lab`, then `nsys start/stop` in cells.

Nsight Compute: Kernel-level details, launch with `ncu --set full python script.py`.

Visualization: Nsight GUI timelines show memory, transfers, kernels.

## Visualizing Traces with Chrome Tracing or Perfetto

Many profilers (Torch, Nsight, cProfile via snakeviz) export JSON/proto traces for interactive viewing.

### Chrome Tracing
- Load JSON in `chrome://tracing`.
- Features: Timeline, flame graphs, zoomable events.
- For Torch: Use `export_chrome_trace()`.
- Pros: Simple, built-in Chrome; Cons: Struggles with large files (>1GB).

### Perfetto
Perfetto UI (ui.perfetto.dev) handles large traces better, using protobuf format.
- Setup for large traces:
  1. Download `trace_processor`: `curl -LO https://get.perfetto.dev/trace_processor; chmod +x trace_processor`
  2. Run: `./trace_processor --httpd trace.json`
  3. Open ui.perfetto.dev; it detects local server.
- Compatibility: Works with PyTorch traces (export as JSON, convert if needed); supports Python via tools like VizTracer.
- Pros: Handles 2GB+ traces, SQL queries; Cons: Setup required.

For Python: Use py-spy or VizTracer to generate compatible traces.

## Best Practices
- Start with general profilers, specialize for ML/GPU.
- Visualize always: Raw tables miss timelines.
- Minimize overhead: Sample strategically.
- Resources: PyTorch docs, NVIDIA User Guide, Perfetto docs.