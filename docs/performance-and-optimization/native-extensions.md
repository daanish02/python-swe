# Native Extensions

## When to Use Native Extensions

**Native extension**: Code written in C/C++ or compiled from Python to bypass Python's interpreter overhead.

### Why Use Native Extensions

```python
# Pure Python: Slow for numeric computation
def sum_array(arr):
    total = 0
    for value in arr:
        total += value
    return total

# Time for 1 million elements: ~100ms

# NumPy (C extension): Fast
import numpy as np
arr = np.arange(1_000_000)
total = np.sum(arr)

# Time for 1 million elements: ~1ms
# 100x faster!
```

### When to Consider Native Extensions

```python
# Use native extensions when:
# - Tight loops with numeric computation
# - Performance-critical code (profiled bottleneck)
# - Need to call existing C/C++ libraries
# - Pure Python too slow even after optimization

# Don't use when:
# - Code is already fast enough
# - I/O-bound (won't help)
# - Adds too much complexity
# - Pure Python is readable and maintainable
```

### Options Overview

```python
# Option 1: Numba (easiest)
# - JIT compile Python to machine code
# - Just add @jit decorator
# - Good for numeric/scientific code

# Option 2: Cython
# - Write Python-like code, compiles to C
# - More control than Numba
# - Can interface with C libraries

# Option 3: ctypes/cffi
# - Call existing C libraries from Python
# - No compilation needed
# - Use existing C code

# Option 4: C extensions
# - Write C code directly
# - Maximum control and performance
# - Most complex, avoid unless necessary
```

## Numba for JIT Compilation

**JIT (Just-In-Time) compilation**: Compiling Python code to machine code at runtime for dramatic speedups.

### Installing Numba

```bash
pip install numba
```

### Basic Usage

```python
import numba
import numpy as np
import time

# Pure Python: Slow
def sum_array_python(arr):
    total = 0
    for i in range(len(arr)):
        total += arr[i]
    return total

# With Numba: Fast
@numba.jit
def sum_array_numba(arr):
    total = 0
    for i in range(len(arr)):
        total += arr[i]
    return total

# Benchmark
arr = np.arange(10_000_000, dtype=np.float64)

# Pure Python
start = time.time()
result_py = sum_array_python(arr)
time_py = time.time() - start

# Numba (first call includes compilation)
start = time.time()
result_nb = sum_array_numba(arr)
time_nb_first = time.time() - start

# Numba (subsequent calls are fast)
start = time.time()
result_nb = sum_array_numba(arr)
time_nb = time.time() - start

print(f"Python: {time_py:.4f}s")         # ~2.5s
print(f"Numba (first): {time_nb_first:.4f}s")  # ~0.3s (includes compilation)
print(f"Numba: {time_nb:.4f}s")          # ~0.003s (800x faster!)
```

### Numba with nopython Mode

```python
import numba

# nopython mode: Force compilation, no Python fallback
# Ensures maximum performance

@numba.jit(nopython=True)
def compute_distance(x1, y1, x2, y2):
    """Euclidean distance"""
    dx = x2 - x1
    dy = y2 - y1
    return (dx**2 + dy**2)**0.5

# Or use @njit shorthand
@numba.njit
def compute_distances(points):
    n = len(points)
    distances = np.zeros((n, n))
    for i in range(n):
        for j in range(n):
            distances[i, j] = compute_distance(
                points[i, 0], points[i, 1],
                points[j, 0], points[j, 1]
            )
    return distances

# Very fast for numeric operations
points = np.random.rand(1000, 2)
distances = compute_distances(points)
```

### Parallel Execution with Numba

```python
import numba
import numpy as np

# Automatic parallelization
@numba.njit(parallel=True)
def parallel_sum(arr):
    total = 0
    # prange = parallel range
    for i in numba.prange(len(arr)):
        total += arr[i] ** 2
    return total

# Uses multiple CPU cores automatically
arr = np.arange(100_000_000)
result = parallel_sum(arr)
```

### Numba Limitations

```python
# Numba works best with:
# - NumPy arrays
# - Numeric types (int, float)
# - Math operations
# - Loops

# Numba doesn't support:
# - Complex Python objects
# - Most standard library
# - String operations (limited)
# - Dynamic typing

# This works:
@numba.njit
def good_example(arr):
    result = np.zeros(len(arr))
    for i in range(len(arr)):
        result[i] = arr[i] * 2
    return result

# This doesn't work:
@numba.njit
def bad_example(items):
    # Lists and dictionaries not supported well
    result = {}
    for item in items:
        result[item] = len(item)  # String operations not supported
    return result
```

## Cython for Compiled Python

**Cython**: Superset of Python that compiles to C, allowing optional static typing for performance.

### Installing Cython

```bash
pip install cython
```

### Basic Cython Example

```python
# mymodule.pyx (Cython file)

def sum_array_python(arr):
    """Pure Python version"""
    total = 0
    for value in arr:
        total += value
    return total

def sum_array_cython(double[:] arr):
    """Cython version with types"""
    cdef double total = 0.0  # C variable
    cdef int i
    for i in range(arr.shape[0]):
        total += arr[i]
    return total
```

```python
# setup.py - Build Cython extension
from setuptools import setup
from Cython.Build import cythonize
import numpy as np

setup(
    ext_modules=cythonize("mymodule.pyx"),
    include_dirs=[np.get_include()]
)
```

```bash
# Build extension
python setup.py build_ext --inplace

# Now can import
python
>>> from mymodule import sum_array_cython
>>> import numpy as np
>>> arr = np.arange(1_000_000, dtype=np.float64)
>>> result = sum_array_cython(arr)
```

### Cython with Static Types

```python
# fast_math.pyx

# Import C math library
from libc.math cimport sqrt, sin, cos

def calculate_distance(double x1, double y1, double x2, double y2):
    """Fast distance calculation using C types"""
    cdef double dx = x2 - x1
    cdef double dy = y2 - y1
    return sqrt(dx*dx + dy*dy)

def process_array(double[:] arr):
    """Process array with C types"""
    cdef int i
    cdef int n = arr.shape[0]
    cdef double total = 0.0

    for i in range(n):
        total += sin(arr[i]) + cos(arr[i])

    return total
```

### Cython Performance Annotations

```bash
# Generate HTML report showing bottlenecks
cython -a mymodule.pyx

# Opens mymodule.html
# Yellow lines = Python interaction (slow)
# White lines = Pure C (fast)
# Click lines to see C code
```

### When to Use Cython

```python
# Use Cython when:
# - Need more control than Numba
# - Interfacing with C libraries
# - Mix Python and C code
# - Want static typing benefits

# Example: Wrapping C library
# mylib.pyx
cdef extern from "mylib.h":
    int c_function(int x, int y)

def python_function(int x, int y):
    """Python wrapper for C function"""
    return c_function(x, y)
```

## Calling C Libraries

### Using ctypes

**ctypes**: Built-in Python library for calling C functions from shared libraries.

```python
import ctypes
import numpy as np

# Load C library
# Linux: libc.so.6
# macOS: libc.dylib
# Windows: msvcrt.dll

libc = ctypes.CDLL('libc.so.6')  # Linux

# Call C sqrt function
sqrt = libc.sqrt
sqrt.argtypes = [ctypes.c_double]  # Input type
sqrt.restype = ctypes.c_double     # Return type

result = sqrt(16.0)
print(result)  # 4.0

# More complex example: Custom C library
# my_clib.c:
# int add(int a, int b) { return a + b; }
# int sum_array(int* arr, int size) {
#     int total = 0;
#     for(int i=0; i<size; i++) total += arr[i];
#     return total;
# }

# Compile: gcc -shared -o my_clib.so -fPIC my_clib.c

# Use from Python
mylib = ctypes.CDLL('./my_clib.so')

# Simple function
result = mylib.add(5, 3)  # 8

# Array function
arr = (ctypes.c_int * 5)(1, 2, 3, 4, 5)
result = mylib.sum_array(arr, 5)  # 15
```

### Using cffi

**cffi**: C Foreign Function Interface, more Pythonic than ctypes.

```bash
pip install cffi
```

```python
from cffi import FFI

ffi = FFI()

# Define C function signatures
ffi.cdef("""
    double sqrt(double x);
    int printf(const char *format, ...);
""")

# Load library
C = ffi.dlopen(None)  # libc

# Call functions
result = C.sqrt(16.0)
print(result)  # 4.0

C.printf(b"Hello from C!\\n")

# More complex: Custom library
ffi = FFI()
ffi.cdef("""
    int add(int a, int b);
    int sum_array(int* arr, int size);
""")

lib = ffi.dlopen('./my_clib.so')

# Call functions
result = lib.add(5, 3)
print(result)  # 8

# Pass array
arr = ffi.new("int[]", [1, 2, 3, 4, 5])
result = lib.sum_array(arr, 5)
print(result)  # 15
```

### ctypes vs cffi

```python
# ctypes:
# - Built-in (no installation)
# - More low-level
# - Verbose type declarations

# cffi:
# - More Pythonic
# - Better error messages
# - Easier to use
# - Recommended for new code

# Both allow calling existing C libraries without recompilation
```

## Trade-offs and Considerations

### Development Complexity

```python
# Simplicity ranking (easiest → hardest):
# 1. Pure Python (simplest, slowest)
# 2. Numba (@jit decorator, minimal changes)
# 3. NumPy (use existing library)
# 4. cffi (call C library, moderate complexity)
# 5. Cython (new syntax, compilation needed)
# 6. C extensions (maximum complexity)

# Start simple, optimize only when necessary
```

### Performance vs Maintainability

```python
# Pure Python: Easy to maintain, slower
def calculate(data):
    result = []
    for item in data:
        result.append(item * 2)
    return result

# With Numba: Still readable, much faster
@numba.njit
def calculate_fast(data):
    result = np.empty(len(data))
    for i in range(len(data)):
        result[i] = data[i] * 2
    return result

# Cython: Less readable, requires compilation
def calculate_cython(double[:] data):
    cdef int i
    cdef int n = data.shape[0]
    cdef double[:] result = np.empty(n)
    for i in range(n):
        result[i] = data[i] * 2
    return result

# C extension: Maximum performance, maximum complexity
# (Not worth showing here - very complex)
```

### Portability

```python
# Pure Python: Works everywhere
# Numba: Works on Windows/macOS/Linux (requires LLVM)
# Cython: Needs C compiler on target system
# C extensions: Need compilation for each platform

# Distributing compiled extensions:
# - Provide source + build instructions
# - Or provide pre-built wheels for common platforms
# - Or use PyPI binary distributions
```

### When Each Approach Makes Sense

```python
# Numba:
# - Scientific/numeric computing
# - NumPy-heavy code
# - Quick performance boost
# - Experimentation

# Cython:
# - Need to interface with C libraries
# - More control than Numba
# - Large codebase conversion
# - Production performance-critical code

# ctypes/cffi:
# - Existing C libraries
# - No need to recompile C code
# - Quick C integration
# - Prototyping C library usage

# Pure Python + optimization:
# - Good enough performance
# - Maintainability priority
# - I/O-bound (native won't help)
# - Rapid development
```

### Real-World Decision Making

```python
# Step 1: Profile
# Find actual bottleneck with cProfile

# Step 2: Pure Python optimization
# - Better algorithm
# - Better data structures
# - Use NumPy/Pandas if applicable

# Step 3: If still too slow and CPU-bound:
# - Try Numba first (easiest)
# - If Numba doesn't work, try Cython
# - Last resort: C extension

# Most code doesn't need native extensions!
# Profile first, optimize only proven bottlenecks
```

## Summary

Native extensions for Python performance:

**When to use:**

- CPU-bound bottleneck (profiled and verified)
- Tight loops with numeric computation
- Need to call existing C/C++ libraries
- Pure Python too slow after optimization

**Numba (JIT compilation):**

- Easiest: Just add `@numba.jit` decorator
- Great for numeric/NumPy code
- Automatic parallelization with `parallel=True`
- Limited to numeric types and NumPy arrays
- 10-1000x speedups possible

**Cython:**

- Compile Python-like code to C
- Optional static typing for performance
- Interface with C libraries
- More control than Numba
- Requires C compiler

**ctypes/cffi:**

- Call existing C libraries without recompilation
- ctypes: Built-in but verbose
- cffi: More Pythonic, recommended
- Good for using existing C code

**Trade-offs:**

- Complexity: Numba < cffi < Cython < C extensions
- Performance: C extensions > Cython ≈ Numba > cffi
- Portability: Pure Python > Numba > Cython/cffi
- Maintainability: Pure Python > Numba > others

**Best practice:**

1. Profile to find bottleneck
2. Optimize pure Python first
3. Try Numba if numeric
4. Consider Cython if Numba insufficient
5. Use ctypes/cffi for existing C libraries

Most code doesn't need native extensions -- profile before optimizing.

## Next Steps

- Review [Performance Fundamentals](performance-fundamentals.md)
- Master [Profiling and Measurement](profiling-and-measurement.md)
- Explore [Parallel and Concurrent Execution](parallel-and-concurrent.md)
