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

## Key Points
- `newline=""` prevents extra blank lines on Windows
- `writerow()` for single row
- `writerows()` for multiple rows
- Use `"w"` to create/overwrite, `"a"` to append
- Always close file or use `with` statement

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
