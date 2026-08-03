# Lesson 06 — OOP: Classes & Inheritance 🏗️

**Object-Oriented Programming (OOP)** groups data and the functions that work on it into
**classes**. Your project is full of them: services, exceptions, database models, Pydantic
schemas.

---

## 📖 Concept

### A class is a blueprint

```python
class ToolService:
    def __init__(self, url):        # constructor: runs when you create one
        self.url = url              # `self` = this specific object
        self.session = None

    def connect(self):              # a method (function inside a class)
        self.session = f"connected to {self.url}"
        return self.session

svc = ToolService("http://localhost")   # create an instance
print(svc.connect())
```

- `self` refers to the specific object. Every method takes `self` first.
- `__init__` is the **constructor**, called automatically on creation.
- Attributes (`self.url`) store data on the object.

### Inheritance — build on an existing class

```python
class Animal:
    def speak(self):
        return "..."

class Dog(Animal):          # Dog inherits everything from Animal
    def speak(self):        # override the method
        return "Woof"
```

`super().__init__(...)` calls the **parent's** constructor.

### Singleton pattern

Your services are created **once** and reused everywhere (a "singleton"), e.g. `agent_service`,
`tool_service`. This shares one database connection instead of making new ones each time.

---

## 🔎 In Your Project

**A service class** (`app/api/services/tool_service.py`):

```python
class ToolService:
    """Service class for managing tools and MCP server connectivity."""

    def __init__(self):
        db_type = get_default_db_type()
        self.SessionLocal = get_async_db_session(db_type, echo=False)
        self.relay_server_url = config.relay_service.service_url
```

**Inheritance for exceptions** (`app/api/exceptions/custom_exceptions.py`):

```python
class AgenticAIException(Exception):          # base class inherits from built-in Exception
    def __init__(self, code, detail, status_code=500, ...):
        self.code = code
        self.detail = detail
        self.status_code = status_code

class ResourceNotFoundException(AgenticAIException):   # child class
    def __init__(self, code="resourceNotFound", detail="...", ...):
        super().__init__(                    # call the parent constructor
            code=code, detail=detail, status_code=404, ...
        )
```

Every custom error in your app inherits from `AgenticAIException`. `ResourceNotFoundException`
just presets `status_code=404`. This is inheritance solving a real problem.

**A SQLAlchemy model** (also a class!) (`app/api/services/tool_service.py`):

```python
class ToolDB(Base):
    __tablename__ = "tools"
    id = Column(String(36), primary_key=True)
    name = Column(String(255), nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
```

---

## 🏋️ Exercise

In `playground.py`:

```python
# 1. Build a base exception like AgenticAIException
class AppException(Exception):
    def __init__(self, code: str, detail: str, status_code: int = 500):
        super().__init__(detail)
        self.code = code
        self.detail = detail
        self.status_code = status_code

# 2. Create a child that presets a 404 (like ResourceNotFoundException)
class NotFoundException(AppException):
    def __init__(self, detail: str = "Not found"):
        super().__init__(code="notFound", detail=detail, status_code=404)

# 3. Raise and catch it
try:
    raise NotFoundException("Agent 123 not found")
except AppException as e:
    print(e.status_code, e.code, e.detail)

# 4. Challenge: add a BadRequestException(AppException) with status_code=400.
```

---

## 🔗 Project Connection

Understanding classes unlocks *most* of your codebase: services (`agent_service`,
`tool_service`), the entire `exceptions/` folder, Pydantic `models/`, and SQLAlchemy DB models
are all classes. Inheritance is exactly how your exception hierarchy avoids repetition.

---

## ➡️ Next Step

Go to **[Lesson 07 — Error Handling](./07-error-handling.md)**.

