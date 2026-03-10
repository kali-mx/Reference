# API Interaction with Python

## Making Requests
```python
import requests

response = requests.get(url, headers=headers, params=params)
data = response.json()
```

## Authentication
```python
headers = {
    "Authorization": f"token {TOKEN}",
    "User-Agent": "your-app-name"
}
```

## GitHub API Example
```python
# Get authenticated user's repos
url = "https://api.github.com/user/repos"
params = {"per_page": 100, "type": "owner"}
response = requests.get(url, headers=headers, params=params)
repos = response.json()

for repo in repos:
    name = repo["name"]
    stars = repo["stargazers_count"]
```

## Key Lessons
- Store API keys in .env, never in code
- Always include User-Agent header
- Read official docs for parameters
- Handle pagination (some APIs return 30 items by default)
- Check response status: `response.status_code`

## Security
```python
import os
from dotenv import load_dotenv

load_dotenv()
TOKEN = os.getenv("api_key")  # Never hardcode
```
