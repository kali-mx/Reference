# Threading for I/O-Bound Work

## Problem
Scanning 65k ports sequentially takes minutes.
Multiple ports could connect in parallel.

## Solution: Queue + Worker Threads

### Full Working Example
```python
import threading
from queue import Queue

print_lock = threading.Lock()

def safe_print(*args, **kwargs):
    """Print safely in multithreaded context"""
    with print_lock:
        print(*args, **kwargs)

def worker_function(item):
    """Do actual work here"""
    # I/O operation (network, file, API call)
    result = some_io_operation(item)
    safe_print(f"Result: {result}")

def worker():
    """Worker thread - pulls work from queue"""
    while True:
        item = q.get()
        worker_function(item)
        q.task_done()

# Create queue and spawn workers
q = Queue()

num_threads = 40  # Sweet spot for I/O-bound work
for _ in range(num_threads):
    t = threading.Thread(target=worker, daemon=True)
    t.start()

# Queue work
for item in range(1, 65535):
    q.put(item)

# Wait for completion
q.join()
```

### Key Components Explained

**1. Lock for safe printing**
```python
print_lock = threading.Lock()

def safe_print(*args, **kwargs):
    with print_lock:  # Only one thread prints at a time
        print(*args, **kwargs)
```
Why? Multiple threads printing simultaneously = garbled output.
Lock serializes access to stdout.

**2. Worker function (your actual work)**
```python
def worker_function(item):
    # This is where I/O happens (network, disk, API)
    result = do_something_slow(item)
    safe_print(result)  # Use safe_print, not print()
```

**3. Worker loop (thread's job)**
```python
def worker():
    while True:
        item = q.get()        # Block until work available
        worker_function(item)
        q.task_done()         # Mark complete, unblock q.join()
```

**4. Thread pool spawning**
```python
num_threads = 40  # Why 40?
# - Too few (5): threads sit idle waiting
# - Too many (1000): wasted memory, context switching
# - 40: good parallelism without waste

for _ in range(num_threads):
    t = threading.Thread(target=worker, daemon=True)
    t.start()
```

**5. Queue work and wait**
```python
for item in range(1, 65535):
    q.put(item)  # Add to queue

q.join()  # Block main thread until all items marked done
```

## Real-World Example

**Port Scanner** (github.com/you/port-scanner)
- Scans 65k ports in ~2 seconds
- Uses 40 threads
- Thread-safe output with Lock

**API Fuzzer** (github.com/you/api-fuzzer)
- Fuzz 1000 endpoints concurrently
- Same Queue pattern, different work function

**DNS Resolver** (github.com/you/dns-recon)
- Resolve 10k domains in parallel
- Threading works because DNS lookups are I/O-bound

## When to Use This Pattern

✅ Network I/O (sockets, HTTP requests, DNS)
✅ File I/O (reading/writing many files)
✅ API calls (multiple concurrent requests)
❌ CPU-intensive work (use multiprocessing instead)
❌ Shared mutable state (use locks, or better yet, avoid)

## Common Mistakes

1. **Creating 1000 threads** → Resource waste. Use a reasonable pool (10-50)
2. **Not closing resources** → File descriptor leaks. Close in worker function
3. **Printing without lock** → Garbled output. Always use safe_print()
4. **Ignoring exceptions in workers** → Thread dies silently. Add try/except


