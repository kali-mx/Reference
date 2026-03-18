# Multi-Tool Reconnaissance Script

## Overview

A cohesive penetration testing tool that combines DNS lookup, port scanning, and security header analysis into one script. Outputs all results to structured CSV.

## Architecture

Three main functions that return consistent list format:

### `get_domain(domain)` 
- Resolves domain to IP using socket.gethostbyname()
- Returns: `[domain, "DNS", "IP", ip_address]`
- Error handling: Catches exceptions, returns error string

### `header_check(url)`
- Checks for security headers (X-Frame-Options, CSP, etc.)
- Tries HTTPS first, falls back to HTTP
- Returns: List of rows `[url, "SecurityHeader", header_name, value]`
- Each header gets its own row for CSV compatibility

### `portscanner(host)`
- Scans ports 1-9500 with 300 concurrent threads
- Uses socket.connect_ex() for non-blocking connection attempts
- Timeout: 0.5 seconds (reduces hanging on closed ports)
- Handles banner grabbing (HTTP, SSH, FTP, etc.)
- Returns: Dict of port results `{port: [host, "Port", port, banner]}`
- **Note:** Dict returned here, but converted to list format in main()

## Key Learnings

### Data Structure Consistency
- All functions should return lists `[domain, scan_type, key, value]`
- This makes CSV writing trivial
- Dicts are fine internally, but convert at function boundary

### Error Handling in Sockets
- Always catch: `socket.timeout`, `ConnectionResetError`, `BrokenPipeError`
- HTTP responses contain embedded `\n` and `\r` characters
- Fix: `banner.replace("\n", "").replace("\r", " ")[:25].strip()`

### Threading at Scale
- 300 threads for 9500 ports = ~30 ports per thread
- Each connection attempt waits ~0.5s (timeout)
- Total time ~10-30 seconds depending on target
- More threads = faster, but diminishing returns after 100-200

### URL Schema Handling
- Don't assume http:// or https://
- Try HTTPS first (modern standard), fall back to HTTP
- Pattern: Try multiple schemas, keep working one

## CSV Output Format
```
Domain,Scan_Type,Result_Key,Result_Value
example.com,DNS,IP,93.184.216.34
example.com,Port,22,OPEN SSH-2.0-Banner
example.com,Port,80,OPEN HTTP/1.1 404 Not F
example.com,SecurityHeader,X-Frame-Options,SAMEORIGIN
example.com,SecurityHeader,CSP,Missing
```

One row per finding. Easy to parse, analyze, and extend.

## Common Issues

**Script hangs:** You're scanning too many ports or timeout is too high. Reduce timeout to 0.5s, limit ports to <10000.

**ConnectionResetError in banner grab:** Server closes connection after response. Wrap recv() in try/except.

**Newlines in CSV:** HTTP responses contain `\r\n`. Use `.replace()` before storing.

**Inconsistent data types in CSV:** Functions returned dicts and lists mixed. Always return lists from functions.

## Real-World Usage
```bash
python3 securitytool.py -a google.com
```

Outputs:
1. Console: Real-time port scan results
2. CSV: Complete structured results for analysis

## See Also
- THREADING.md (worker queue pattern)
- CREDENTIAL_TESTING.md (socket programming)
- CSV_OPERATIONS.md (data format handling)
