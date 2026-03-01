# Parallel and Concurrent Execution

## Understanding Concurrency and Parallelism

**Concurrency**: Multiple tasks making progress, but not necessarily simultaneously (one CPU, multiple tasks).

**Parallelism**: Multiple tasks executing simultaneously (multiple CPUs, true parallel execution).

### The Difference

```python
# Sequential: One task at a time
def sequential():
    download_file1()  # 5 seconds
    download_file2()  # 5 seconds
    download_file3()  # 5 seconds
    # Total: 15 seconds

# Concurrent: Tasks overlap (not necessarily parallel)
def concurrent():
    # Start all downloads
    # While waiting for network, switch between tasks
    # All finish around the same time
    # Total: ~5 seconds (limited by slowest)

# Parallel: Tasks run simultaneously
def parallel():
    # Process data on CPU1
    # Process data on CPU2
    # Process data on CPU3
    # All truly simultaneous
    # Total: 1/3 of sequential time
```

### When Each Matters

```python
# I/O-bound: Waiting for external resources
# - Network requests
# - File operations
# - Database queries
# → Use concurrency (threading, asyncio)

# CPU-bound: Heavy computation
# - Data processing
# - Image manipulation
# - Mathematical calculations
# → Use parallelism (multiprocessing)

# Mixed workload: Combination
# → Use both techniques strategically
```

## The Global Interpreter Lock (GIL)

**GIL (Global Interpreter Lock)**: Mutex that protects Python objects, allowing only one thread to execute Python bytecode at a time.

### How GIL Affects Performance

```python
import threading
import time

def cpu_bound_task():
    """CPU-intensive task"""
    total = 0
    for i in range(10_000_000):
        total += i
    return total

# Sequential
start = time.time()
cpu_bound_task()
cpu_bound_task()
elapsed_seq = time.time() - start
print(f"Sequential: {elapsed_seq:.2f}s")  # ~2.0s

# With threads (doesn't help due to GIL!)
start = time.time()
t1 = threading.Thread(target=cpu_bound_task)
t2 = threading.Thread(target=cpu_bound_task)
t1.start()
t2.start()
t1.join()
t2.join()
elapsed_thread = time.time() - start
print(f"Threading: {elapsed_thread:.2f}s")  # ~2.0s (no speedup!)

# GIL prevents threads from running CPU-bound code in parallel
```

### GIL Releases

```python
# GIL is released during:
# - I/O operations (network, disk)
# - Sleep/wait operations
# - Some C extensions (numpy, compression)

import threading
import time
import requests

def io_bound_task():
    """I/O-bound task (GIL released)"""
    response = requests.get('https://api.example.com/data')
    return response.json()

# Sequential
start = time.time()
io_bound_task()
io_bound_task()
elapsed_seq = time.time() - start
print(f"Sequential: {elapsed_seq:.2f}s")  # ~2.0s

# With threads (helps because GIL released during I/O!)
start = time.time()
threads = [threading.Thread(target=io_bound_task) for _ in range(2)]
for t in threads:
    t.start()
for t in threads:
    t.join()
elapsed_thread = time.time() - start
print(f"Threading: {elapsed_thread:.2f}s")  # ~1.0s (2x speedup!)
```

## Threading for I/O-Bound Tasks

**Thread**: Lightweight execution unit sharing memory space, good for I/O-bound tasks where GIL is released.

### Basic Threading

```python
import threading
import time

def download_file(url, filename):
    print(f"Downloading {filename}...")
    time.sleep(2)  # Simulate download
    print(f"Finished {filename}")

# Sequential: 6 seconds
start = time.time()
download_file('url1', 'file1.txt')
download_file('url2', 'file2.txt')
download_file('url3', 'file3.txt')
print(f"Sequential: {time.time() - start:.2f}s")  # ~6.0s

# Concurrent with threads: ~2 seconds
start = time.time()
threads = [
    threading.Thread(target=download_file, args=('url1', 'file1.txt')),
    threading.Thread(target=download_file, args=('url2', 'file2.txt')),
    threading.Thread(target=download_file, args=('url3', 'file3.txt')),
]

for thread in threads:
    thread.start()

for thread in threads:
    thread.join()  # Wait for completion

print(f"Threading: {time.time() - start:.2f}s")  # ~2.0s (3x speedup!)
```

### ThreadPoolExecutor

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

def download_file(url):
    time.sleep(1)  # Simulate download
    return f"Downloaded {url}"

# Manual thread management is tedious
# ThreadPoolExecutor is cleaner

urls = [f'https://example.com/file{i}' for i in range(10)]

# Using ThreadPoolExecutor
with ThreadPoolExecutor(max_workers=5) as executor:
    # Submit all tasks
    futures = [executor.submit(download_file, url) for url in urls]

    # Get results as they complete
    for future in as_completed(futures):
        result = future.result()
        print(result)

# Pool automatically manages threads
# max_workers=5 means max 5 concurrent downloads
```

### Thread Safety

```python
import threading

# Problem: Race condition
counter = 0

def increment():
    global counter
    for _ in range(100000):
        counter += 1  # Not atomic!

threads = [threading.Thread(target=increment) for _ in range(10)]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(counter)  # Expected: 1000000, Actual: ~700000 (race condition!)

# Solution: Use Lock
counter = 0
lock = threading.Lock()

def increment_safe():
    global counter
    for _ in range(100000):
        with lock:  # Acquire lock
            counter += 1  # Now thread-safe

threads = [threading.Thread(target=increment_safe) for _ in range(10)]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(counter)  # 1000000 (correct!)
```

## Multiprocessing for CPU-Bound Tasks

**Multiprocessing**: Creates separate Python processes, each with own GIL, enabling true parallelism for CPU-bound tasks.

### Basic Multiprocessing

```python
import multiprocessing
import time

def cpu_intensive(n):
    """CPU-bound task"""
    total = 0
    for i in range(n):
        total += i ** 2
    return total

# Sequential
start = time.time()
results = [cpu_intensive(10_000_000) for _ in range(4)]
print(f"Sequential: {time.time() - start:.2f}s")  # ~4.0s

# Parallel with multiprocessing
start = time.time()
with multiprocessing.Pool(processes=4) as pool:
    results = pool.map(cpu_intensive, [10_000_000] * 4)
print(f"Multiprocessing: {time.time() - start:.2f}s")  # ~1.0s (4x speedup!)
```

### ProcessPoolExecutor

```python
from concurrent.futures import ProcessPoolExecutor
import time

def process_chunk(data):
    """Process data chunk"""
    return sum(x ** 2 for x in data)

# Large dataset
data = range(10_000_000)
chunk_size = 1_000_000
chunks = [range(i, min(i + chunk_size, 10_000_000))
          for i in range(0, 10_000_000, chunk_size)]

# Process in parallel
with ProcessPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(process_chunk, chunks))

total = sum(results)
print(f"Total: {total}")
```

### Sharing Data Between Processes

```python
import multiprocessing

# Problem: Processes don't share memory
def increment(counter):
    for _ in range(100):
        counter.value += 1  # Won't work with regular Python objects!

# Solution 1: Use Value for shared primitive
counter = multiprocessing.Value('i', 0)  # 'i' = integer
processes = [multiprocessing.Process(target=increment, args=(counter,))
             for _ in range(4)]

for p in processes:
    p.start()
for p in processes:
    p.join()

print(counter.value)  # Shared between processes

# Solution 2: Use Manager for shared objects
manager = multiprocessing.Manager()
shared_list = manager.list()
shared_dict = manager.dict()

def worker(shared_list):
    shared_list.append(multiprocessing.current_process().name)

processes = [multiprocessing.Process(target=worker, args=(shared_list,))
             for _ in range(4)]
for p in processes:
    p.start()
for p in processes:
    p.join()

print(shared_list)  # Contains entries from all processes
```

### Overhead Considerations

```python
# Multiprocessing has overhead:
# - Process creation time
# - Data serialization (pickling)
# - Inter-process communication

# Only worth it for CPU-intensive tasks

import time
from concurrent.futures import ProcessPoolExecutor

def small_task(x):
    return x ** 2

# Small task: Overhead > benefit
data = range(100)
start = time.time()
with ProcessPoolExecutor() as executor:
    results = list(executor.map(small_task, data))
print(f"Multiprocessing: {time.time() - start:.4f}s")  # Slow!

# Sequential is faster for small tasks
start = time.time()
results = [small_task(x) for x in data]
print(f"Sequential: {time.time() - start:.4f}s")  # Fast!

# Use multiprocessing only when task is substantial
```

## Asyncio for Concurrent I/O

**asyncio**: Single-threaded cooperative concurrency for I/O-bound tasks, using async/await syntax.

### Basic Asyncio

```python
import asyncio
import time

async def fetch_data(id):
    """Simulate async I/O operation"""
    print(f"Fetching {id}...")
    await asyncio.sleep(1)  # Simulates I/O wait
    print(f"Finished {id}")
    return f"Data {id}"

# Run async function
async def main():
    # Sequential: 3 seconds
    start = time.time()
    await fetch_data(1)
    await fetch_data(2)
    await fetch_data(3)
    print(f"Sequential: {time.time() - start:.2f}s")  # ~3.0s

    # Concurrent: 1 second
    start = time.time()
    results = await asyncio.gather(
        fetch_data(1),
        fetch_data(2),
        fetch_data(3)
    )
    print(f"Concurrent: {time.time() - start:.2f}s")  # ~1.0s

asyncio.run(main())
```

### Asyncio with Real I/O

```python
import asyncio
import aiohttp  # pip install aiohttp

async def fetch_url(session, url):
    """Fetch URL asynchronously"""
    async with session.get(url) as response:
        return await response.text()

async def fetch_all(urls):
    """Fetch all URLs concurrently"""
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        return results

# Usage
urls = [
    'https://api.example.com/data1',
    'https://api.example.com/data2',
    'https://api.example.com/data3',
]

results = asyncio.run(fetch_all(urls))
# All fetched concurrently
```

### Asyncio Patterns

```python
import asyncio

# Pattern 1: Gather (wait for all)
async def gather_pattern():
    results = await asyncio.gather(
        fetch_data(1),
        fetch_data(2),
        fetch_data(3)
    )
    # All complete before continuing
    return results

# Pattern 2: As completed (process as they finish)
async def as_completed_pattern():
    tasks = [fetch_data(i) for i in range(5)]
    for coro in asyncio.as_completed(tasks):
        result = await coro
        process(result)  # Process immediately

# Pattern 3: Wait with timeout
async def timeout_pattern():
    try:
        result = await asyncio.wait_for(fetch_data(1), timeout=5.0)
    except asyncio.TimeoutError:
        print("Operation timed out")

# Pattern 4: Create task for background execution
async def background_pattern():
    task = asyncio.create_task(long_running_operation())
    # Do other work while task runs in background
    await do_other_work()
    # Now wait for task
    result = await task
```

### Asyncio vs Threading

```python
# Threading:
# - Preemptive multitasking (OS switches threads)
# - Need locks for shared data
# - Works with blocking I/O

# Asyncio:
# - Cooperative multitasking (explicit await)
# - No locks needed (single-threaded)
# - Requires async-aware libraries (aiohttp, asyncpg)

# Choose asyncio when:
# - Many concurrent I/O operations
# - Using async libraries
# - Want fine control over scheduling

# Choose threading when:
# - Using blocking libraries
# - Moderate concurrency
# - Simpler mental model
```

## Choosing the Right Approach

### Decision Tree

```python
# Start here: Is your task CPU-bound or I/O-bound?

# CPU-bound (heavy computation):
# → Use multiprocessing
# → ProcessPoolExecutor or Pool

# I/O-bound (network, disk):
# → How many concurrent operations?

  # Few (<100):
  # → Use threading
  # → ThreadPoolExecutor

  # Many (100s-1000s):
  # → Use asyncio
  # → async/await with aiohttp, asyncpg, etc.
```

### Practical Examples

```python
# Example 1: Web scraping 50 URLs
# → I/O-bound, moderate concurrency
# → Use ThreadPoolExecutor
from concurrent.futures import ThreadPoolExecutor
import requests

def scrape_url(url):
    return requests.get(url).text

urls = [f'https://example.com/page{i}' for i in range(50)]

with ThreadPoolExecutor(max_workers=10) as executor:
    results = list(executor.map(scrape_url, urls))

# Example 2: Processing 1 million images
# → CPU-bound
# → Use ProcessPoolExecutor
from concurrent.futures import ProcessPoolExecutor
from PIL import Image

def process_image(filename):
    img = Image.open(filename)
    img = img.resize((800, 600))
    img.save(f'processed_{filename}')

images = [f'image{i}.jpg' for i in range(1000000)]

with ProcessPoolExecutor() as executor:
    executor.map(process_image, images)

# Example 3: Making 10,000 API calls
# → I/O-bound, high concurrency
# → Use asyncio
import asyncio
import aiohttp

async def call_api(session, id):
    async with session.get(f'https://api.example.com/{id}') as resp:
        return await resp.json()

async def main():
    async with aiohttp.ClientSession() as session:
        tasks = [call_api(session, i) for i in range(10000)]
        results = await asyncio.gather(*tasks)
        return results

results = asyncio.run(main())
```

### Performance Comparison

```python
# Task: Download 20 files (I/O-bound)

# Sequential: 20 seconds (1s per file)
# Threading: 2 seconds (10 threads, max_workers=10)
# Asyncio: 1 second (all concurrent)

# Task: Process 20 images (CPU-bound)

# Sequential: 20 seconds
# Threading: 20 seconds (GIL prevents parallelism)
# Multiprocessing: 5 seconds (4 CPU cores)
```

### Mixing Approaches

```python
import asyncio
from concurrent.futures import ProcessPoolExecutor

# Combine asyncio + multiprocessing
# Useful for: Concurrent I/O + parallel CPU work

async def process_with_cpu(data):
    """Offload CPU work to separate process"""
    loop = asyncio.get_event_loop()
    with ProcessPoolExecutor() as pool:
        result = await loop.run_in_executor(pool, cpu_intensive_task, data)
    return result

async def main():
    # Fetch data concurrently (I/O)
    async with aiohttp.ClientSession() as session:
        data = await fetch_all_data(session)

    # Process concurrently using multiple processes (CPU)
    tasks = [process_with_cpu(item) for item in data]
    results = await asyncio.gather(*tasks)
    return results

# Best of both worlds!
```

## Summary

Parallel and concurrent execution in Python:

**Concurrency vs Parallelism:**

- Concurrency: Multiple tasks making progress
- Parallelism: Multiple tasks truly simultaneous
- I/O-bound → concurrency, CPU-bound → parallelism

**GIL (Global Interpreter Lock):**

- Only one thread executes Python bytecode at a time
- Released during I/O operations
- Threading helps I/O-bound, not CPU-bound
- Multiprocessing bypasses GIL

**Threading:**

- Good for I/O-bound tasks (network, files)
- Use ThreadPoolExecutor for easy management
- Need locks for shared mutable state
- Limited by GIL for CPU-bound work

**Multiprocessing:**

- Good for CPU-bound tasks (computation)
- True parallelism (separate processes)
- Use ProcessPoolExecutor or Pool
- Overhead: process creation, serialization

**Asyncio:**

- Good for many concurrent I/O operations
- Single-threaded, cooperative multitasking
- Requires async-aware libraries
- async/await syntax, use asyncio.gather()

**Choose based on workload:**

- CPU-bound → multiprocessing
- I/O-bound, <100 ops → threading
- I/O-bound, 100s-1000s ops → asyncio

Profile to verify improvements -- don't assume parallelism always helps.

## Next Steps

- Explore [Native Extensions](native-extensions.md)
- Review [Optimization Techniques](optimization-techniques.md)
- Master [Profiling and Measurement](profiling-and-measurement.md)
