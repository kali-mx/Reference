# Writing CSV Files

## Basic CSV Writer
```python
import csv

data = [
    ["Name", "Age", "City"],
    ["Alice", 30, "NYC"],
    ["Bob", 25, "LA"]
]

with open("output.csv", "w", newline="") as file:
    writer = csv.writer(file)
    writer.writerows(data)  # writerows for multiple rows
```

## Write Header + Data
```python
headers = ["Name", "Age", "City"]
rows = [["Alice", 30, "NYC"], ["Bob", 25, "LA"]]

with open("output.csv", "w", newline="") as file:
    writer = csv.writer(file)
    writer.writerow(headers)  # Single header row
    writer.writerows(rows)    # Multiple data rows
```

## DictWriter (When You Have Dictionaries)

**Use DictWriter only if your data is already in dict format.** For most cases, the csv.writer approach above is simpler.
```python
import csv

# Data as list of dicts
data = [
    {"name": "Alice", "age": 30, "city": "NYC"},
    {"name": "Bob", "age": 25, "city": "LA"}
]

fieldnames = ["name", "age", "city"]

with open("output.csv", "w", newline="") as file:
    writer = csv.DictWriter(file, fieldnames=fieldnames)
    writer.writeheader()  # Write header row
    writer.writerows(data)  # Write dict rows
```

**Key difference:** DictWriter maps dict keys to column names. Useful if your data is already structured as dicts, but adds complexity if you're already working with lists.

**Note:** DictWriter can be finicky. If you have list data, convert to lists and use csv.writer instead—it's cleaner and more predictable.

## Key Points

- `newline=""` prevents extra blank lines on Windows
- `writerow()` for single row
- `writerows()` for multiple rows
- Use `"w"` to create/overwrite, `"a"` to append
- Always close file or use `with` statement
- **For lists:** Use `csv.writer`
- **For dicts:** Use `csv.DictWriter` (if data is already dicts)

## Combining Multiple Dicts for CSV

When you have nested dicts (dict of dicts), merge them before writing:
```python
results_by_url = {
    "http://example.com": {"Server": "cloudflare", "Content-Type": "text/html"},
    "http://google.com": {"Server": "gws", "Content-Type": "text/html"}
}

with open("output.csv", "w", newline="") as file:
    writer = csv.DictWriter(file, fieldnames=["URL", "Server", "Content-Type"])
    writer.writeheader()
    for url, headers in results_by_url.items():
        row = {"URL": url}
        row.update(headers)  # Merge dicts
        writer.writerow(row)
```

## GitHub Repos Example
```python
my_list = []
for repo in data:
    my_list.append([repo["name"], repo["stargazers_count"]])

with open("repos.csv", "w", newline="") as file:
    writer = csv.writer(file)
    writer.writerow(["Repository", "Stars"])
    writer.writerows(my_list)
```

## Credential Testing Example
```python
import csv

valid_creds = []

# Test credentials and store valid ones
for uname, passwd in test_credentials:
    if login_succeeds(uname, passwd):
        valid_creds.append([uname, passwd])

# Write results
with open("valid_creds.csv", "w", newline="") as file:
    writer = csv.writer(file)
    writer.writerow(["Username", "Password"])
    writer.writerows(valid_creds)
```

















