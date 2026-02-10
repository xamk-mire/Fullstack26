# Python Fundamentals for the Backend

You’ll build the **backend** of your full-stack app with **Python**. Python is widely used for web APIs because it’s readable, has strong library support, and frameworks like **FastAPI** make it straightforward to expose HTTP endpoints and return JSON.

This topic introduces the Python basics you need for the backend: variables, types, functions, lists, dicts, and a first taste of **JSON** and **FastAPI**. You don’t need to have used Python before; we focus on what matters for APIs and the course stack.

---

## What You’re Building in This Topic (Build-Along)

By the end you will:

- Run a small **Python script** that works with task-like data (lists and dicts) and prints JSON.
- Run a minimal **FastAPI** app with one endpoint that returns JSON in the browser.

You’ll need **Python 3.10 or newer** installed. Check with:

```bash
python --version
```

(or `python3 --version` on some systems).

---

## 1) Python Uses Indentation for Structure

In Python, **indentation** defines blocks of code (instead of `{ }` in JavaScript). Use **4 spaces (tab)** per level and stay consistent.

```python
def greet(name):
    message = "Hello, " + name
    return message

print(greet("World"))   # Hello, World
```

If you mix tabs and spaces or indent incorrectly, Python will report an error. Use spaces only.

---

## 2) Variables and Basic Types

You don’t declare types; you assign values.

```python
title = "Finish exercises"
count = 3
done = False
```

Common types:

| Type    | Example         | Use                   |
| ------- | --------------- | --------------------- |
| `str`   | `"hello"`       | Text                  |
| `int`   | `42`            | Integers              |
| `float` | `3.14`          | Numbers with decimals |
| `bool`  | `True`, `False` | Booleans              |
| `None`  | `None`          | “No value”            |

---

## 3) Lists and Dicts: The Building Blocks of JSON

APIs send and receive data as **JSON**. In Python, JSON objects become **dicts** and JSON arrays become **lists**.

### Lists (ordered sequences)

```python
tasks = ["Read notes", "Do exercises", "Commit code"]
tasks.append("Deploy")   # add at end
first = tasks[0]        # "Read notes"
length = len(tasks)     # 4
```

### Dicts (key–value pairs)

```python
task = {
    "id": 1,
    "title": "Finish exercises",
    "due": "2026-02-10",
    "done": False
}
print(task["title"])   # Finish exercises
task["done"] = True    # update
```

A list of dicts is exactly what you’ll often return from an API (e.g. “list of tasks”):

```python
tasks = [
    {"id": 1, "title": "Read notes", "done": False},
    {"id": 2, "title": "Do exercises", "done": True}
]
```

---

## 4) Functions

Define functions with `def`. Use `return` to send a value back.

```python
def add(a, b):
    return a + b

def make_task(task_id, title, done=False):
    return {"id": task_id, "title": title, "done": done}
```

Default arguments (e.g. `done=False`) let callers omit that value.

---

## 5) Working with JSON in Python

The built-in `json` module turns Python dicts and lists into JSON strings and back.

```python
import json

task = {"id": 1, "title": "Finish exercises", "done": False}
json_string = json.dumps(task)   # Python -> JSON string
print(json_string)               # {"id": 1, "title": "Finish exercises", "done": false}

data = json.loads('{"id": 2, "title": "Relax"}')  # JSON string -> Python
print(data["title"])             # Relax
```

In FastAPI you usually return dicts or lists directly; FastAPI serializes them to JSON for the HTTP response.

---

## 6) Build-Along: A Script That “Fakes” an API Response

Create a folder (e.g. `python-intro`) and a file `tasks_script.py`:

```python
import json

def get_tasks():
    tasks = [
        {"id": 1, "title": "Read backend materials", "done": False},
        {"id": 2, "title": "Install Python and run script", "done": True},
    ]
    return tasks

if __name__ == "__main__":
    tasks = get_tasks()
    print(json.dumps(tasks, indent=2))
```

Run it:

```bash
cd python-intro
python tasks_script.py
```

You should see JSON printed. This is the same kind of structure your backend will later return over HTTP.

---

## 7) FastAPI: One Endpoint That Returns JSON

FastAPI is the framework you’ll use for the backend. Here we add a single endpoint so you see how a URL returns JSON.

### Create a virtual environment (recommended)

In your `python-intro` folder:

```bash
python -m venv venv
```

Activate it:

- **Windows (PowerShell):** `.\venv\Scripts\Activate.ps1`
- **Windows (cmd):** `venv\Scripts\activate.bat`
- **macOS/Linux:** `source venv/bin/activate`

Then install FastAPI and Uvicorn (ASGI server):

```bash
pip install fastapi uvicorn
```

### Create `main.py`

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/api/tasks")
def list_tasks():
    tasks = [
        {"id": 1, "title": "Read backend materials", "done": False},
        {"id": 2, "title": "Run first API", "done": True},
    ]
    return tasks
```

### Run the server

```bash
uvicorn main:app --reload
```

Open in the browser: **http://127.0.0.1:8000/api/tasks**

You should see the same list as JSON. Try the automatic docs: **http://127.0.0.1:8000/docs**.

---

## 8) What Just Happened?

- `FastAPI()` creates your app.
- `@app.get("/api/tasks")` registers a **GET** handler for that path.
- The function returns a **list of dicts**; FastAPI turns it into a **JSON response** with the right headers and status code.

Later you’ll add more endpoints (POST, PATCH, DELETE), validation (e.g. Pydantic models), and database access (PostgreSQL). The pattern stays the same: define routes, return Python data, FastAPI handles HTTP and JSON.

---

## What You Should Understand After This Topic

By the end, you should be able to:

- Use variables, basic types, lists, and dicts in Python.
- Write a simple function and return a dict or list.
- Use `json.dumps` and `json.loads` to convert between Python data and JSON strings.
- Run a minimal FastAPI app and open an endpoint in the browser.
- Explain that FastAPI turns returned dicts/lists into JSON HTTP responses.

