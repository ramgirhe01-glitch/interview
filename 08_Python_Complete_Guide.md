# 🐍 Python Complete Interview Guide — SDET Focus
### Ram Girhe | 3 YOE | Exercises + Interview Questions + Deep Concepts

---

# PART 1: PYTHON CORE CONCEPTS

---

## 1. Data Types & Variables

```python
# Numbers
x = 10          # int
y = 3.14        # float
z = 2 + 3j      # complex

# Strings (immutable)
name = "Ram"
name = 'Ram'
multiline = """This is
a multiline string"""

# Boolean
is_active = True
is_deleted = False

# None
result = None

# Type checking
type(x)       # <class 'int'>
isinstance(x, int)  # True
```

### String Methods (asked very often):
```python
s = "Hello World"
s.upper()              # "HELLO WORLD"
s.lower()              # "hello world"
s.title()              # "Hello World"
s.strip()              # Remove whitespace
s.split(" ")           # ["Hello", "World"]
s.replace("World", "Python")  # "Hello Python"
s.find("World")        # 6 (index) or -1
s.count("l")           # 3
s.startswith("Hello")  # True
s.endswith("World")    # True
s.isdigit()            # False
s.isalpha()            # False (has space)
len(s)                 # 11
s[0]                   # "H"
s[-1]                  # "d"
s[0:5]                 # "Hello" (slicing)
s[::-1]                # "dlroW olleH" (reverse)
",".join(["a","b","c"]) # "a,b,c"

# f-strings (Python 3.6+)
name = "Ram"
age = 25
print(f"My name is {name} and I am {age} years old")
```

---

## 2. Data Structures

### 2.1 List (ordered, mutable, allows duplicates)
```python
fruits = ["apple", "banana", "cherry", "apple"]
fruits.append("date")          # Add to end
fruits.insert(1, "avocado")    # Insert at index
fruits.remove("banana")        # Remove first occurrence
fruits.pop()                   # Remove last
fruits.pop(0)                  # Remove at index
fruits.sort()                  # Sort in-place
fruits.reverse()               # Reverse in-place
fruits.index("cherry")         # Find index
fruits.count("apple")          # Count occurrences
len(fruits)                    # Length

# List comprehension (VERY important)
squares = [x**2 for x in range(10)]              # [0, 1, 4, 9, ...]
evens = [x for x in range(20) if x % 2 == 0]    # [0, 2, 4, ...]
flat = [x for sublist in [[1,2],[3,4]] for x in sublist]  # [1,2,3,4]

# Slicing
arr = [0, 1, 2, 3, 4, 5]
arr[1:4]    # [1, 2, 3]
arr[:3]     # [0, 1, 2]
arr[3:]     # [3, 4, 5]
arr[-2:]    # [4, 5]
arr[::2]    # [0, 2, 4] (step=2)
arr[::-1]   # [5, 4, 3, 2, 1, 0] (reverse)
```

### 2.2 Tuple (ordered, immutable)
```python
point = (10, 20)
x, y = point                   # Unpacking
point[0]                        # 10
# point[0] = 5                  # ❌ TypeError — immutable

# When to use: function return values, dictionary keys, constants
def get_min_max(arr):
    return min(arr), max(arr)   # Returns tuple

low, high = get_min_max([3, 1, 4, 1, 5])
```

### 2.3 Set (unordered, unique, mutable)
```python
nums = {1, 2, 3, 2, 1}        # {1, 2, 3} — duplicates removed
nums.add(4)
nums.remove(1)                 # Raises KeyError if not found
nums.discard(99)               # No error if not found

# Set operations
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}
a | b        # Union: {1, 2, 3, 4, 5, 6}
a & b        # Intersection: {3, 4}
a - b        # Difference: {1, 2}
a ^ b        # Symmetric difference: {1, 2, 5, 6}
```

### 2.4 Dictionary (key-value, ordered since 3.7)
```python
user = {"name": "Ram", "age": 25, "city": "Pune"}
user["name"]                    # "Ram"
user.get("email", "N/A")       # "N/A" (default if missing)
user["email"] = "ram@test.com"  # Add/update
del user["city"]                # Delete key
user.keys()                     # dict_keys(['name', 'age', 'email'])
user.values()                   # dict_values(['Ram', 25, 'ram@test.com'])
user.items()                    # dict_items([('name', 'Ram'), ...])
"name" in user                  # True

# Dictionary comprehension
squares = {x: x**2 for x in range(5)}  # {0:0, 1:1, 2:4, 3:9, 4:16}

# Merge dicts (Python 3.9+)
d1 = {"a": 1}
d2 = {"b": 2}
merged = d1 | d2               # {"a": 1, "b": 2}

# defaultdict
from collections import defaultdict
freq = defaultdict(int)
for char in "hello":
    freq[char] += 1             # {'h':1, 'e':1, 'l':2, 'o':1}

# Counter
from collections import Counter
Counter("hello")                # Counter({'l': 2, 'h': 1, 'e': 1, 'o': 1})
Counter([1,2,2,3,3,3]).most_common(2)  # [(3, 3), (2, 2)]
```

---

## 3. Functions

```python
# Basic function
def greet(name, greeting="Hello"):
    """Greet a person with optional greeting."""
    return f"{greeting}, {name}!"

greet("Ram")                   # "Hello, Ram!"
greet("Ram", "Hi")             # "Hi, Ram!"

# *args (variable positional arguments)
def add(*args):
    return sum(args)
add(1, 2, 3, 4)               # 10

# **kwargs (variable keyword arguments)
def create_user(**kwargs):
    return kwargs
create_user(name="Ram", age=25) # {'name': 'Ram', 'age': 25}

# Lambda (anonymous function)
square = lambda x: x ** 2
square(5)                       # 25

# Map, Filter, Reduce
nums = [1, 2, 3, 4, 5]
list(map(lambda x: x**2, nums))          # [1, 4, 9, 16, 25]
list(filter(lambda x: x % 2 == 0, nums)) # [2, 4]
from functools import reduce
reduce(lambda a, b: a + b, nums)          # 15

# Decorator
def log_execution(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Finished {func.__name__}, returned {result}")
        return result
    return wrapper

@log_execution
def add(a, b):
    return a + b

add(3, 4)
# Output: Calling add
#         Finished add, returned 7
```

**Interview Q:** *What is a decorator?*
> A function that wraps another function to extend its behavior without modifying it. Common use: logging, timing, authentication, retry.

---

## 4. OOP in Python

```python
class Animal:
    species_count = 0  # Class variable (shared)
    
    def __init__(self, name, sound):
        self.name = name       # Instance variable
        self._sound = sound    # Protected (convention)
        self.__id = id(self)   # Private (name mangling)
        Animal.species_count += 1
    
    def speak(self):
        return f"{self.name} says {self._sound}"
    
    @property
    def sound(self):           # Getter
        return self._sound
    
    @sound.setter
    def sound(self, value):    # Setter
        if not value:
            raise ValueError("Sound cannot be empty")
        self._sound = value
    
    @classmethod
    def get_count(cls):        # Class method
        return cls.species_count
    
    @staticmethod
    def is_animal(obj):        # Static method
        return isinstance(obj, Animal)
    
    def __str__(self):         # String representation
        return f"Animal({self.name})"
    
    def __repr__(self):        # Developer representation
        return f"Animal(name='{self.name}', sound='{self._sound}')"
    
    def __eq__(self, other):   # Equality comparison
        return self.name == other.name
    
    def __len__(self):         # len() support
        return len(self.name)


class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name, "Woof")
        self.breed = breed
    
    def speak(self):           # Override
        return f"{self.name} barks: {self._sound}!"
    
    def fetch(self, item):
        return f"{self.name} fetches {item}"


# Usage
dog = Dog("Buddy", "Labrador")
print(dog.speak())          # "Buddy barks: Woof!"
print(dog.fetch("ball"))    # "Buddy fetches ball"
print(len(dog))             # 5 (length of "Buddy")
print(Animal.get_count())   # 1
```

### Abstract Classes:
```python
from abc import ABC, abstractmethod

class TestBase(ABC):
    @abstractmethod
    def setup(self):
        pass
    
    @abstractmethod
    def run_test(self):
        pass
    
    def teardown(self):        # Concrete method
        print("Cleaning up...")
    
    def execute(self):         # Template method
        self.setup()
        self.run_test()
        self.teardown()

class ApiTest(TestBase):
    def setup(self):
        print("Getting auth token...")
    
    def run_test(self):
        print("Calling API...")

test = ApiTest()
test.execute()
```

---

## 5. File Handling

```python
# Read file
with open("data.txt", "r") as f:
    content = f.read()           # Entire file as string
    # or
    lines = f.readlines()       # List of lines

# Write file
with open("output.txt", "w") as f:
    f.write("Hello World\n")

# Append
with open("output.txt", "a") as f:
    f.write("New line\n")

# Read JSON
import json
with open("data.json", "r") as f:
    data = json.load(f)

# Write JSON
with open("output.json", "w") as f:
    json.dump({"name": "Ram"}, f, indent=2)

# CSV
import csv
with open("data.csv", "r") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row["name"])
```

---

## 6. Error Handling

```python
# Basic try-except
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")
except (TypeError, ValueError) as e:
    print(f"Type/Value error: {e}")
except Exception as e:
    print(f"Unexpected: {e}")
else:
    print("Success — no exception")
finally:
    print("Always runs")

# Custom exception
class ApiTestError(Exception):
    def __init__(self, message, status_code):
        super().__init__(message)
        self.status_code = status_code

# Raise
def validate_response(response):
    if response.status_code != 200:
        raise ApiTestError(f"API failed: {response.text}", response.status_code)

# Context manager for cleanup
from contextlib import contextmanager

@contextmanager
def database_connection(url):
    conn = connect(url)
    try:
        yield conn
    finally:
        conn.close()
```

---

## 7. Python for API Testing (requests library)

```python
import requests

# GET
response = requests.get(
    "https://api.example.com/users",
    headers={"Authorization": "Bearer token123"},
    params={"region": "US", "limit": 10}
)
print(response.status_code)    # 200
print(response.json())         # Parse JSON response
print(response.headers)        # Response headers
print(response.elapsed.total_seconds())  # Response time

# POST
response = requests.post(
    "https://api.example.com/users",
    headers={"Content-Type": "application/json"},
    json={"name": "Ram", "email": "ram@test.com"}  # auto-serializes
)
user_id = response.json()["id"]

# PUT
requests.put(f"https://api.example.com/users/{user_id}",
    json={"name": "Ram Updated"})

# DELETE
requests.delete(f"https://api.example.com/users/{user_id}")

# Session (reuse connection + headers)
session = requests.Session()
session.headers.update({"Authorization": "Bearer token123"})
session.get("https://api.example.com/users")    # Token auto-included
session.get("https://api.example.com/orders")   # Token auto-included

# File upload
files = {"file": open("test.pdf", "rb")}
requests.post("https://api.example.com/upload", files=files)

# Timeout + Retry
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

session = requests.Session()
retry = Retry(total=3, backoff_factor=1, status_forcelist=[500, 502, 503])
session.mount("https://", HTTPAdapter(max_retries=retry))
```

---

## 8. Python for Test Automation (pytest)

```python
# test_api.py
import pytest
import requests

BASE_URL = "https://api.example.com"

# Fixture — setup/teardown
@pytest.fixture(scope="session")
def auth_token():
    response = requests.post(f"{BASE_URL}/auth/token",
        data={"grant_type": "client_credentials", "client_id": "test"})
    return response.json()["access_token"]

@pytest.fixture
def headers(auth_token):
    return {"Authorization": f"Bearer {auth_token}", "Content-Type": "application/json"}

# Test function
def test_get_users(headers):
    response = requests.get(f"{BASE_URL}/users", headers=headers)
    assert response.status_code == 200
    assert len(response.json()) > 0

def test_create_user(headers):
    payload = {"name": "Ram", "email": "ram@test.com"}
    response = requests.post(f"{BASE_URL}/users", headers=headers, json=payload)
    assert response.status_code == 201
    assert response.json()["name"] == "Ram"

# Parametrize — data-driven testing
@pytest.mark.parametrize("name,email,expected_status", [
    ("Ram", "ram@test.com", 201),
    ("", "test@test.com", 400),
    ("User", "invalid-email", 400),
    ("User", "", 400),
])
def test_create_user_validation(headers, name, email, expected_status):
    response = requests.post(f"{BASE_URL}/users", headers=headers,
        json={"name": name, "email": email})
    assert response.status_code == expected_status

# Marks
@pytest.mark.smoke
def test_health_check():
    assert requests.get(f"{BASE_URL}/health").status_code == 200

@pytest.mark.skip(reason="API not deployed yet")
def test_new_feature():
    pass

# Run: pytest test_api.py -v -m smoke
```

---

# PART 2: PYTHON EXERCISES

---

### E1: Count word frequency
```python
def word_frequency(sentence):
    words = sentence.lower().split()
    return dict(Counter(words))

# "the cat sat on the mat" → {'the': 2, 'cat': 1, 'sat': 1, 'on': 1, 'mat': 1}
```

### E2: Flatten nested dictionary
```python
def flatten_dict(d, parent_key="", sep="."):
    items = {}
    for k, v in d.items():
        new_key = f"{parent_key}{sep}{k}" if parent_key else k
        if isinstance(v, dict):
            items.update(flatten_dict(v, new_key, sep))
        else:
            items[new_key] = v
    return items

# {"a": {"b": {"c": 1}}, "d": 2} → {"a.b.c": 1, "d": 2}
```

### E3: Find common elements in two lists
```python
def common_elements(list1, list2):
    return list(set(list1) & set(list2))
```

### E4: Remove duplicates preserving order
```python
def remove_duplicates(lst):
    seen = set()
    return [x for x in lst if x not in seen and not seen.add(x)]
```

### E5: Matrix transpose
```python
def transpose(matrix):
    return [list(row) for row in zip(*matrix)]

# [[1,2,3],[4,5,6]] → [[1,4],[2,5],[3,6]]
```

### E6: Merge two sorted lists
```python
def merge_sorted(a, b):
    result = []
    i = j = 0
    while i < len(a) and j < len(b):
        if a[i] <= b[j]:
            result.append(a[i]); i += 1
        else:
            result.append(b[j]); j += 1
    result.extend(a[i:])
    result.extend(b[j:])
    return result
```

### E7: Validate JSON response structure
```python
def validate_user_response(response_json):
    """Validate that API response has required fields with correct types."""
    required_fields = {
        "id": int,
        "name": str,
        "email": str,
        "roles": list,
    }
    errors = []
    for field, expected_type in required_fields.items():
        if field not in response_json:
            errors.append(f"Missing field: {field}")
        elif not isinstance(response_json[field], expected_type):
            errors.append(f"{field} should be {expected_type.__name__}, got {type(response_json[field]).__name__}")
    return errors  # Empty list = valid
```

### E8: Retry decorator for API calls
```python
import time

def retry(max_attempts=3, delay=1, exceptions=(Exception,)):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt+1} failed: {e}. Retrying in {delay}s...")
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(max_attempts=3, delay=2, exceptions=(requests.RequestException,))
def call_api(url):
    response = requests.get(url, timeout=5)
    response.raise_for_status()
    return response.json()
```

---

# PART 3: TOP 40 PYTHON INTERVIEW QUESTIONS

---

| # | Question | Key Answer |
|---|----------|------------|
| 1 | Is Python compiled or interpreted? | Interpreted (CPython compiles to bytecode first, then interprets) |
| 2 | What is PEP 8? | Python style guide: 4-space indent, snake_case, max 79 chars/line |
| 3 | Mutable vs Immutable? | Mutable: list, dict, set. Immutable: str, tuple, int, frozenset |
| 4 | List vs Tuple? | List: mutable, slower. Tuple: immutable, faster, can be dict key |
| 5 | What is list comprehension? | `[x**2 for x in range(10) if x%2==0]` — concise list creation |
| 6 | What are *args and **kwargs? | *args: variable positional args (tuple). **kwargs: variable keyword args (dict) |
| 7 | What is a lambda function? | Anonymous: `lambda x: x**2`. Used for short, throwaway functions |
| 8 | map() vs filter() vs reduce()? | map: transform all. filter: keep matching. reduce: combine into one |
| 9 | What is a decorator? | Function that wraps another to extend behavior. @decorator_name syntax |
| 10 | What is a generator? | Lazy iterator using `yield`. Memory-efficient for large data |
| 11 | What is `self`? | Reference to current instance (like `this` in Java) |
| 12 | `__init__` vs `__new__`? | `__new__`: creates instance. `__init__`: initializes it |
| 13 | What are dunder/magic methods? | `__str__`, `__repr__`, `__eq__`, `__len__`, `__getitem__` etc. |
| 14 | What is @property? | Getter/setter as attribute access. Encapsulation with clean syntax |
| 15 | @classmethod vs @staticmethod? | classmethod: gets cls, can access class state. staticmethod: no cls/self |
| 16 | What is GIL? | Global Interpreter Lock — only one thread executes Python bytecode at a time |
| 17 | Threading vs Multiprocessing? | Threading: I/O-bound (limited by GIL). Multiprocessing: CPU-bound |
| 18 | What is a context manager? | `with` statement. Implements `__enter__` and `__exit__`. Auto cleanup |
| 19 | Deep copy vs Shallow copy? | Shallow: copies references. Deep: copies objects recursively |
| 20 | What is `pass`? | No-op placeholder. Used in empty functions/classes |
| 21 | What is `None`? | Python's null. Singleton. Check with `is None`, not `== None` |
| 22 | How is memory managed? | Reference counting + garbage collector for cycles |
| 23 | What is pip? | Package manager. `pip install requests`, `pip freeze > requirements.txt` |
| 24 | virtualenv/venv? | Isolated Python environment per project. Avoids dependency conflicts |
| 25 | What is `__name__ == "__main__"`? | Runs code only when script is executed directly, not imported |
| 26 | Exception handling? | try/except/else/finally. Custom exceptions extend Exception |
| 27 | What is type hinting? | `def add(a: int, b: int) -> int:` Optional type annotations |
| 28 | What is a dataclass? | `@dataclass` auto-generates `__init__`, `__repr__`, `__eq__` |
| 29 | What is an f-string? | `f"Hello {name}"` — formatted string literal (Python 3.6+) |
| 30 | dict.get() vs dict[]? | get() returns default on missing key. [] raises KeyError |
| 31 | How to read JSON in Python? | `json.load(file)` or `json.loads(string)` |
| 32 | How to make API calls? | `requests` library: get(), post(), put(), delete() |
| 33 | What is pytest? | Test framework. Fixtures, parametrize, marks, plugins |
| 34 | pytest fixtures? | Setup/teardown functions. Scope: function, class, module, session |
| 35 | pytest parametrize? | `@pytest.mark.parametrize("x,y,expected", [(1,2,3)])` |
| 36 | What is conftest.py? | Shared fixtures file. Auto-discovered by pytest |
| 37 | Assert in Python? | `assert condition, "message"`. In pytest: `assert x == 5` |
| 38 | How to run specific tests? | `pytest -k "test_login"` or `pytest -m smoke` |
| 39 | What is monkey patching? | Dynamically modifying code at runtime. pytest: `monkeypatch` fixture |
| 40 | How to mock API calls? | `unittest.mock.patch`, `responses` library, `pytest-mock` |

---

*Python is your Swiss Army knife for automation. Master requests + pytest + data structures!*

