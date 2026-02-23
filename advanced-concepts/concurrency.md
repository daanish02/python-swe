# Concurrency

## Table of Contents
- [Concurrency vs Parallelism](#concurrency-vs-parallelism)
- [The Global Interpreter Lock (GIL)](#the-global-interpreter-lock-gil)
- [Threading](#threading)
- [Multiprocessing](#multiprocessing)
- [Asyncio](#asyncio)
- [Choosing the Right Approach](#choosing-the-right-approach)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Concurrency vs Parallelism

### Definitions

```python
# Concurrency: Multiple tasks making progress (not necessarily simultaneously)
# - Single-core CPU switching between tasks
# - Good for I/O-bound tasks

# Parallelism: Multiple tasks running simultaneously
# - Multi-core CPU executing tasks at the same time
# - Good for CPU-bound tasks

# Example: Coffee shop
# Concurrency: One barista handling multiple orders (switching between tasks)
# Parallelism: Multiple baristas each handling an order simultaneously
```

### I/O-bound vs CPU-bound

```python
# I/O-bound: Waiting for input/output
# - Network requests
# - File operations
# - Database queries
# Solution: Threading or Asyncio

# CPU-bound: Heavy computation
# - Mathematical calculations
# - Data processing
# - Image manipulation
# Solution: Multiprocessing
```

## The Global Interpreter Lock (GIL)

### What is the GIL?

```python
# GIL = Mutex that protects Python objects
# Only one thread can execute Python bytecode at a time
# Prevents true parallel execution of Python code

# Why GIL exists:
# - Simplifies memory management
# - Protects reference counting
# - Makes C extensions easier to write

# Impact:
# - Threading: Limited by GIL (no parallel CPU execution)
# - Multiprocessing: No GIL (separate processes)
# - Asyncio: No parallel execution, but efficient I/O handling
```

### GIL in Action

```python
import threading
import time

counter = 0

def increment():
    global counter
    for _ in range(1000000):
        counter += 1

# Single thread
start = time.time()
increment()
print(f"Single thread: {time.time() - start:.2f}s, counter = {counter}")

# Multiple threads (still slow due to GIL)
counter = 0
start = time.time()
threads = [threading.Thread(target=increment) for _ in range(2)]
for t in threads:
    t.start()
for t in threads:
    t.join()
print(f"Two threads: {time.time() - start:.2f}s, counter = {counter}")
# May not be 2x faster due to GIL!
```

## Threading

### Basic Threading

```python
import threading
import time

def worker(name, delay):
    print(f"{name} starting")
    time.sleep(delay)
    print(f"{name} finished")

# Create threads
t1 = threading.Thread(target=worker, args=("Thread-1", 2))
t2 = threading.Thread(target=worker, args=("Thread-2", 1))

# Start threads
t1.start()
t2.start()

# Wait for completion
t1.join()
t2.join()

print("All threads completed")
```

### Thread with Return Value

```python
import threading

class ThreadWithReturn(threading.Thread):
    def __init__(self, target, args=()):
        super().__init__()
        self.target = target
        self.args = args
        self.result = None
    
    def run(self):
        self.result = self.target(*self.args)

def compute(x, y):
    return x + y

thread = ThreadWithReturn(target=compute, args=(5, 3))
thread.start()
thread.join()
print(f"Result: {thread.result}")  # 8
```

### Thread Synchronization

#### Lock

```python
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(100000):
        lock.acquire()
        try:
            counter += 1
        finally:
            lock.release()

# Or use context manager
def increment_safe():
    global counter
    for _ in range(100000):
        with lock:
            counter += 1

threads = [threading.Thread(target=increment_safe) for _ in range(10)]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(f"Counter: {counter}")  # 1000000 (correct)
```

#### RLock (Reentrant Lock)

```python
import threading

rlock = threading.RLock()

def outer():
    with rlock:
        print("Outer acquired lock")
        inner()

def inner():
    with rlock:  # Can acquire lock again (reentrant)
        print("Inner acquired lock")

outer()
```

#### Semaphore

```python
import threading
import time

# Limit concurrent access to 3
semaphore = threading.Semaphore(3)

def access_resource(name):
    print(f"{name} waiting")
    with semaphore:
        print(f"{name} acquired semaphore")
        time.sleep(2)
        print(f"{name} releasing semaphore")

threads = [threading.Thread(target=access_resource, args=(f"Thread-{i}",)) 
           for i in range(10)]
for t in threads:
    t.start()
for t in threads:
    t.join()
```

#### Event

```python
import threading
import time

event = threading.Event()

def waiter():
    print("Waiting for event")
    event.wait()  # Block until set
    print("Event received!")

def setter():
    time.sleep(2)
    print("Setting event")
    event.set()

t1 = threading.Thread(target=waiter)
t2 = threading.Thread(target=setter)

t1.start()
t2.start()

t1.join()
t2.join()
```

### Thread Pool

```python
from concurrent.futures import ThreadPoolExecutor
import time

def task(name):
    print(f"Task {name} starting")
    time.sleep(1)
    return f"Result from {name}"

# Create thread pool
with ThreadPoolExecutor(max_workers=3) as executor:
    # Submit tasks
    futures = [executor.submit(task, f"Task-{i}") for i in range(5)]
    
    # Get results
    for future in futures:
        result = future.result()
        print(result)
```

## Multiprocessing

### Basic Multiprocessing

```python
import multiprocessing
import time

def worker(name, delay):
    print(f"{name} starting (PID: {multiprocessing.current_process().pid})")
    time.sleep(delay)
    print(f"{name} finished")

if __name__ == '__main__':
    # Create processes
    p1 = multiprocessing.Process(target=worker, args=("Process-1", 2))
    p2 = multiprocessing.Process(target=worker, args=("Process-2", 1))
    
    # Start processes
    p1.start()
    p2.start()
    
    # Wait for completion
    p1.join()
    p2.join()
    
    print("All processes completed")
```

### Process Pool

```python
from concurrent.futures import ProcessPoolExecutor
import os

def cpu_bound_task(n):
    """CPU-intensive task."""
    print(f"Processing {n} in PID {os.getpid()}")
    total = sum(i * i for i in range(n))
    return total

if __name__ == '__main__':
    with ProcessPoolExecutor(max_workers=4) as executor:
        numbers = [5000000, 4000000, 3000000, 2000000]
        results = executor.map(cpu_bound_task, numbers)
        
        for num, result in zip(numbers, results):
            print(f"{num}: {result}")
```

### Sharing Data Between Processes

#### Queue

```python
import multiprocessing

def producer(queue):
    for i in range(5):
        print(f"Producing {i}")
        queue.put(i)
    queue.put(None)  # Signal end

def consumer(queue):
    while True:
        item = queue.get()
        if item is None:
            break
        print(f"Consuming {item}")

if __name__ == '__main__':
    queue = multiprocessing.Queue()
    
    p1 = multiprocessing.Process(target=producer, args=(queue,))
    p2 = multiprocessing.Process(target=consumer, args=(queue,))
    
    p1.start()
    p2.start()
    
    p1.join()
    p2.join()
```

#### Pipe

```python
import multiprocessing

def sender(conn):
    conn.send("Hello from sender")
    conn.close()

def receiver(conn):
    msg = conn.recv()
    print(f"Received: {msg}")
    conn.close()

if __name__ == '__main__':
    parent_conn, child_conn = multiprocessing.Pipe()
    
    p1 = multiprocessing.Process(target=sender, args=(parent_conn,))
    p2 = multiprocessing.Process(target=receiver, args=(child_conn,))
    
    p1.start()
    p2.start()
    
    p1.join()
    p2.join()
```

#### Shared Memory

```python
import multiprocessing

def increment(shared_value, lock):
    for _ in range(100000):
        with lock:
            shared_value.value += 1

if __name__ == '__main__':
    shared_value = multiprocessing.Value('i', 0)  # 'i' = integer
    lock = multiprocessing.Lock()
    
    processes = [multiprocessing.Process(target=increment, args=(shared_value, lock)) 
                 for _ in range(4)]
    
    for p in processes:
        p.start()
    for p in processes:
        p.join()
    
    print(f"Final value: {shared_value.value}")  # 400000
```

## Asyncio

### Basic Async/Await

```python
import asyncio

async def say_hello(name, delay):
    print(f"Hello {name}")
    await asyncio.sleep(delay)  # Non-blocking sleep
    print(f"Goodbye {name}")

async def main():
    # Run concurrently
    await asyncio.gather(
        say_hello("Alice", 2),
        say_hello("Bob", 1),
        say_hello("Charlie", 3)
    )

asyncio.run(main())
```

### Async Functions

```python
import asyncio
import time

async def fetch_data(id):
    print(f"Fetching data {id}")
    await asyncio.sleep(1)  # Simulate I/O
    return f"Data {id}"

async def main():
    start = time.time()
    
    # Sequential
    result1 = await fetch_data(1)
    result2 = await fetch_data(2)
    print(f"Sequential: {time.time() - start:.2f}s")
    
    # Concurrent
    start = time.time()
    results = await asyncio.gather(
        fetch_data(1),
        fetch_data(2),
        fetch_data(3)
    )
    print(f"Concurrent: {time.time() - start:.2f}s")
    print(results)

asyncio.run(main())
```

### Tasks

```python
import asyncio

async def background_task():
    for i in range(5):
        print(f"Background task iteration {i}")
        await asyncio.sleep(1)

async def main():
    # Create task (runs in background)
    task = asyncio.create_task(background_task())
    
    # Do other work
    print("Doing other work...")
    await asyncio.sleep(2)
    print("Other work done")
    
    # Wait for background task
    await task

asyncio.run(main())
```

### Async Context Managers

```python
import asyncio

class AsyncResource:
    async def __aenter__(self):
        print("Acquiring resource")
        await asyncio.sleep(1)
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("Releasing resource")
        await asyncio.sleep(1)
    
    async def query(self):
        print("Querying resource")
        await asyncio.sleep(1)
        return "result"

async def main():
    async with AsyncResource() as resource:
        result = await resource.query()
        print(result)

asyncio.run(main())
```

### Async Iterators

```python
import asyncio

class AsyncRange:
    def __init__(self, n):
        self.n = n
        self.i = 0
    
    def __aiter__(self):
        return self
    
    async def __anext__(self):
        if self.i >= self.n:
            raise StopAsyncIteration
        await asyncio.sleep(0.1)
        self.i += 1
        return self.i

async def main():
    async for i in AsyncRange(5):
        print(i)

asyncio.run(main())
```

## Choosing the Right Approach

### Decision Tree

```python
# CPU-bound task (heavy computation)
# → Use multiprocessing

# I/O-bound task (network, files, database)
# → Simple I/O with blocking calls: Threading
# → Many concurrent I/O operations: Asyncio
# → Mixed or legacy code: Threading

# Example scenarios:

# 1. Web scraping (I/O-bound)
# - Few pages: Threading
# - Many pages: Asyncio

# 2. Image processing (CPU-bound)
# - Use multiprocessing

# 3. Web server (I/O-bound)
# - Use asyncio (like FastAPI, aiohttp)

# 4. Data processing pipeline (mixed)
# - Use multiprocessing for CPU-intensive steps
# - Use asyncio for I/O steps
```

### Performance Comparison

```python
import time
import threading
import multiprocessing
import asyncio
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

# CPU-bound task
def cpu_task(n):
    return sum(i * i for i in range(n))

# I/O-bound task (simulated)
def io_task():
    time.sleep(1)
    return "done"

# Async I/O task
async def async_io_task():
    await asyncio.sleep(1)
    return "done"

def test_threading_cpu():
    start = time.time()
    with ThreadPoolExecutor(max_workers=4) as executor:
        results = list(executor.map(cpu_task, [1000000] * 4))
    return time.time() - start

def test_multiprocessing_cpu():
    start = time.time()
    with ProcessPoolExecutor(max_workers=4) as executor:
        results = list(executor.map(cpu_task, [1000000] * 4))
    return time.time() - start

def test_threading_io():
    start = time.time()
    with ThreadPoolExecutor(max_workers=4) as executor:
        results = list(executor.map(lambda _: io_task(), range(4)))
    return time.time() - start

async def test_asyncio_io():
    start = time.time()
    results = await asyncio.gather(*[async_io_task() for _ in range(4)])
    return time.time() - start

if __name__ == '__main__':
    print(f"Threading (CPU): {test_threading_cpu():.2f}s")
    print(f"Multiprocessing (CPU): {test_multiprocessing_cpu():.2f}s")
    print(f"Threading (I/O): {test_threading_io():.2f}s")
    print(f"Asyncio (I/O): {asyncio.run(test_asyncio_io()):.2f}s")
```

## Summary

Concurrency in Python:

**Concepts:**
- **Concurrency:** Tasks making progress (switching)
- **Parallelism:** Tasks running simultaneously
- **GIL:** Limits threading to one Python bytecode at a time

**Threading:**
- Good for I/O-bound tasks
- Limited by GIL for CPU-bound tasks
- Use `threading` module or `ThreadPoolExecutor`
- Synchronization: Lock, RLock, Semaphore, Event

**Multiprocessing:**
- Good for CPU-bound tasks
- Bypasses GIL (separate processes)
- Use `multiprocessing` module or `ProcessPoolExecutor`
- IPC: Queue, Pipe, Shared Memory

**Asyncio:**
- Best for many concurrent I/O operations
- Single-threaded cooperative multitasking
- Use `async`/`await` syntax
- Event loop manages tasks

**Choosing:**
- CPU-bound → Multiprocessing
- I/O-bound (simple) → Threading
- I/O-bound (many concurrent) → Asyncio

## Next Steps

- Optimize with [Memory Management](memory-management.md)
- Deep dive into [Python Internals](python-internals.md)
- Apply in [Performance Optimization](../performance_and_optimization/profiling.md)
