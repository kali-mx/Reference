# Threading for I/O-Bound Work

## Problem
Scanning 65k ports sequentially takes a long time.
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

def threader():
    """Worker thread - pulls items from queue and processes them"""
    while True:
        item = q.get()
        
        # Do your actual work here
        try:
            result = do_something_with(item)
            with print_lock:
                safe_print(f"Result: {result}")
        except Exception as e:
            safe_print(f"Error processing {item}: {e}")
        
        q.task_done()

# Create queue and spawn workers
q = Queue()

num_threads = 40  # Sweet spot for I/O-bound work
for _ in range(num_threads):
    t = threading.Thread(target=threader, daemon=True)
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

**2. Worker thread function (your main loop)**
```python
def threader():
    while True:
        item = q.get()        # Block until work available
        # Process item here
        q.task_done()         # Mark complete, unblock q.join()
```

**3. Shared state with locks**
```python
results_lock = threading.Lock()
results = {}

def threader():
    while True:
        item = q.get()
        result = process(item)
        
        with results_lock:  # Lock before modifying shared dict
            results[item] = result
        
        q.task_done()
```
Why? Multiple threads writing to same dict = data corruption.

**4. Thread pool spawning**
```python
num_threads = 40  # Why 40?
# - Too few (5): threads sit idle waiting for I/O
# - Too many (1000): wasted memory, context switching overhead
# - 40: good parallelism without resource waste

for _ in range(num_threads):
    t = threading.Thread(target=threader, daemon=True)
    t.start()
```

**5. Queue work and wait**
```python
for item in range(1, 65535):
    q.put(item)  # Add to queue

q.join()  # Block main thread until all items marked done
```

## Real-World Examples

**Port Scanner** (/Users/ms/Documents/VSC_Python_Projects/port_scanner.py)
- Scans 65k ports in ~2 seconds (vs ~10+ sequential)
- 40 concurrent threads
- Thread-safe output with print_lock

**DNS Resolver** (/Users/ms/Documents/VSC_Python_Projects/dns_resolver.py)
- Resolve 300+ domains in parallel
- Same Queue pattern, different work function
- Stores results in shared dict with results_lock

## When to Use This Pattern

✅ Network I/O (sockets, HTTP requests, DNS lookups)
✅ File I/O (reading/writing many files)
✅ API calls (multiple concurrent requests)
❌ CPU-intensive work (use multiprocessing instead)
❌ Shared mutable state without locks (= data corruption)

## Common Mistakes

1. **Creating 1000 threads** → Resource waste. Use reasonable pool (10-50)
2. **Not closing resources** → File descriptor leaks. Close sockets/files in worker
3. **Printing without lock** → Garbled output. Always use lock around shared resources
4. **Ignoring exceptions in workers** → Thread dies silently. Add try/except in threader()
5. **Modifying shared state without lock** → Race conditions. Use `with lock:` around dict/list access

## Exception Handling in Workers
```python
def threader():
    while True:
        item = q.get()
        try:
            result = process(item)
            with results_lock:
                results[item] = result
        except Exception as e:
            safe_print(f"Error on {item}: {e}")
        finally:
            q.task_done()  # ALWAYS mark done, even on error
```

Why finally? If an exception occurs before q.task_done(), q.join() hangs forever.




















