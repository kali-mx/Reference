# Python Lists: append() vs extend() vs +=

## append(): Add one item
```python
my_list = [1, 2, 3]
my_list.append(4)
# Result: [1, 2, 3, 4]

my_list.append([5, 6])
# Result: [1, 2, 3, 4, [5, 6]]  ← nested!
```

## extend(): Add multiple items (unpacks)
```python
my_list = [1, 2, 3]
my_list.extend([4, 5])
# Result: [1, 2, 3, 4, 5]  ← flattened
```

## +=: Concatenates (like extend)
```python
my_list = [1, 2, 3]
my_list += [4, 5]
# Result: [1, 2, 3, 4, 5]

# BUT with nested lists:
my_list = [[1], [2]]
my_list += [[3], [4]]
# Result: [[1], [2], [3], [4]]  ← flattens the outer list
```

## When to Use What

**append()**: Adding a single item (or keeping structure)
```python
rows = []
rows.append([name, age, city])  # One row
rows.append([name2, age2, city2])  # Another row
# Result: [[name, age, city], [name2, age2, city2]]
```

**extend()**: Adding multiple items from another list
```python
my_list = [1, 2]
my_list.extend([3, 4, 5])  # Unpack and add all
# Result: [1, 2, 3, 4, 5]
```

**+=**: Concatenation (use carefully with nested structures)

## Real Example: CSV Writing
```python
my_list = []
for item in data:
    my_list.append([item.name, item.value])  # Each row is a list

# Write to CSV
with open("file.csv", "w") as f:
    writer = csv.writer(f)
    writer.writerows(my_list)  # writerows needs list of lists
```
