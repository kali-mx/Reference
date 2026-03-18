# Data Inspection & Cleanup Pattern

## The Problem

You write code that processes data, it breaks, and you have no idea why. The data looks "normal" in your terminal output, but contains hidden characters (`\n`, `\r`, `\t`, spaces, etc.) that cause issues downstream.

## The Solution: Inspect → Clean → Verify

**Always follow this three-step pattern:**

### Step 1: ALWAYS Inspect First
```python
print(repr(banner))
```

The `repr()` function shows **actual characters** including all whitespace and escape sequences:
- `\n` = newline (hidden in normal print)
- `\r` = carriage return (invisible)
- `\t` = tab (invisible)
- `' '` = space (visible)

**Example:**
```python
banner = "SSH-2.0-OpenSSH\n6.6.1p1"
print(banner)        # Output: SSH-2.0-OpenSSH
                     #         6.6.1p1
print(repr(banner))  # Output: 'SSH-2.0-OpenSSH\n6.6.1p1'  ← SEE THE \n
```

Now you know there's a newline embedded.

### Step 2: Clean Based on What You Find
```python
if '\n' in banner:
    banner = banner.replace('\n', ' ')
if '\r' in banner:
    banner = banner.replace('\r', ' ')
```

Only remove what you found. Don't over-clean.

### Step 3: Verify It Worked
```python
print(repr(cleaned_banner))  # Check again
```

Confirm the unwanted characters are gone.

## Real-World Example

**Your HTTP banner grab issue:**
```python
# Step 1: Inspect
print(repr(banner))
# Output: 'HTTP/1.1 404 Not Found\r\nContent-Type: text/html\r\n\r\n'
# → Found \r\n characters throughout

# Step 2: Clean
banner = banner.replace('\r', ' ').replace('\n', ' ')

# Step 3: Verify
print(repr(banner))
# Output: 'HTTP/1.1 404 Not Found  Content-Type: text/html    '
# → All \r\n gone, replaced with spaces ✓

# Step 4: Use safely
result = banner[:25].strip()  # Now safe to slice/strip
```

## Key Insight

**Never assume data is "clean."** HTTP responses, file reads, API responses, socket data—all can contain hidden characters. The pattern of Inspect → Clean → Verify catches 95% of data bugs before they cascade into your CSV files or downstream processing.

## When to Apply

Apply this pattern when:
- Reading from network sockets
- Parsing HTTP responses
- Reading files
- Processing API responses
- Any time data comes from outside your code

## See Also
- CREDENTIAL_TESTING.md (socket data handling)
- MULTITOOL_RECON.md (real example in portscanner function)
