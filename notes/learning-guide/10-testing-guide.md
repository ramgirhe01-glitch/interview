# 10 — Testing Guide

> pytest = JUnit + Mockito combined. This project uses async testing extensively.

---

## 🎯 Testing Stack

| Tool | Purpose | Java Equivalent |
|------|---------|-----------------|
| `pytest` | Test framework | JUnit 5 |
| `pytest-asyncio` | Async test support | — |
| `pytest-mock` | Mocking | Mockito |
| `pytest-cov` | Code coverage | JaCoCo |
| `unittest.mock` | Built-in mocking | Mockito |

---

## 📁 Test Structure

```
tests/
├── conftest.py              # 🔧 Global test fixtures (mock DB, credentials)
├── __init__.py
├── unit/                    # ✅ Unit tests (no real DB/services)
│   ├── services/            # Service tests
│   │   ├── test_agent_service.py
│   │   ├── test_tool_service.py
│   │   ├── test_execution_service.py
│   │   └── ...
│   ├── endpoints/           # Endpoint tests
│   ├── utils/               # Utility tests
│   └── models/              # Model validation tests
└── integration/             # 🔗 Integration tests (need real services)
```

---

## 🧪 Basic Test Structure

### Simple Test (Like JUnit @Test):
```python
# tests/unit/services/test_agent_service.py
import pytest
from unittest.mock import AsyncMock, patch, MagicMock

class TestAgentService:
    """Test class — like a JUnit test class"""
    
    def test_create_agent_success(self):
        """Simple synchronous test"""
        # Arrange
        data = {"name": "Test Agent", "model": "claude-3"}
        
        # Act
        result = process_agent_data(data)
        
        # Assert
        assert result["name"] == "Test Agent"
        assert result["model"] == "claude-3"
    
    @pytest.mark.asyncio
    async def test_execute_agent(self):
        """Async test — most tests in this project are async"""
        # Arrange
        agent_id = "test-123"
        instruction = "Hello"
        
        # Act
        result = await agent_service.execute_agent(agent_id, instruction)
        
        # Assert
        assert result is not None
        assert "response" in result
```

**Java equivalent:**
```java
@Test
void testCreateAgentSuccess() {
    // Arrange
    AgentCreate data = new AgentCreate("Test Agent", "claude-3");
    
    // Act
    AgentResponse result = agentService.create(data);
    
    // Assert
    assertEquals("Test Agent", result.getName());
}
```

---

## 🎭 Mocking (Like Mockito)

### Mock a Dependency:
```python
from unittest.mock import AsyncMock, patch, MagicMock

class TestAgentService:
    
    @pytest.mark.asyncio
    @patch('services.agent_service.get_async_db_session')
    async def test_get_agent(self, mock_db_session):
        """Mock the database session"""
        # Setup mock
        mock_session = AsyncMock()
        mock_session.execute.return_value = MagicMock(
            fetchone=MagicMock(return_value=MagicMock(
                _mapping={"id": "123", "name": "Test Agent"}
            ))
        )
        mock_db_session.return_value = AsyncMock(
            __aenter__=AsyncMock(return_value=mock_session),
            __aexit__=AsyncMock(return_value=None)
        )
        
        # Execute
        result = await agent_service.get_agent("123", "tenant-1", ...)
        
        # Verify
        assert result["name"] == "Test Agent"
        mock_session.execute.assert_called_once()
```

**Java Mockito equivalent:**
```java
@Mock
private AgentRepository repository;

@Test
void testGetAgent() {
    when(repository.findById("123")).thenReturn(Optional.of(new Agent("123", "Test")));
    AgentResponse result = agentService.getAgent("123");
    assertEquals("Test", result.getName());
    verify(repository).findById("123");
}
```

### Mock Patterns Used in This Project:

```python
# Pattern 1: @patch decorator (mocks a specific import path)
@patch('services.agent_service.execution_service')
async def test_execute(self, mock_exec_service):
    mock_exec_service.create_execution = AsyncMock(return_value={"id": "exec-1"})
    ...

# Pattern 2: MagicMock for sync objects
mock_config = MagicMock()
mock_config.aws.region = "us-east-1"
mock_config.database.host = "localhost"

# Pattern 3: AsyncMock for async functions
mock_service = AsyncMock()
mock_service.get_agent.return_value = {"id": "123", "name": "Bot"}

# Pattern 4: patch as context manager
async def test_something(self):
    with patch('module.function') as mock_fn:
        mock_fn.return_value = "mocked"
        result = await function_under_test()
        assert result == "mocked"
```

---

## 🔧 Test Fixtures (conftest.py)

Fixtures = shared setup code (like @BeforeEach/@BeforeAll):

```python
# tests/conftest.py
import pytest
from unittest.mock import MagicMock, AsyncMock

@pytest.fixture
def mock_db_session():
    """Provides a mock database session for tests."""
    session = AsyncMock()
    session.execute = AsyncMock()
    session.commit = AsyncMock()
    session.rollback = AsyncMock()
    return session

@pytest.fixture
def sample_agent():
    """Provides a sample agent dict for tests."""
    return {
        "id": "agent-123",
        "name": "Test Agent",
        "system_prompt": "You are helpful",
        "model": "claude-3-sonnet",
        "tools": [],
        "tenant_id": "tenant-1",
        "provider_tenant": "provider-1",
        "application": "app-1",
    }

@pytest.fixture
def mock_llm():
    """Mock LLM that returns predefined responses."""
    llm = AsyncMock()
    llm.ainvoke.return_value = MagicMock(content="Hello! How can I help?")
    return llm
```

### Using Fixtures in Tests:
```python
class TestAgentService:
    
    @pytest.mark.asyncio
    async def test_create_agent(self, mock_db_session, sample_agent):
        """Fixtures auto-injected by name matching!"""
        # mock_db_session and sample_agent are automatically provided
        mock_db_session.execute.return_value = MagicMock(...)
        
        result = await agent_service.create(sample_agent)
        assert result["id"] == "agent-123"
```

---

## 📋 Root conftest.py (This Project)

The project's root `conftest.py` does important setup:

```python
# conftest.py (project root)

# 1. Mocks external modules that aren't installed in test env
sys.modules['a2a'] = MagicMock()
sys.modules['a2a.client'] = MagicMock()
sys.modules['mcp.shared'] = MagicMock()

# 2. Adds source paths so imports work
sys.path.insert(0, os.path.join(project_root, "app"))
sys.path.insert(0, os.path.join(project_root, "app", "api"))

# 3. Configures logging for test output
logging.basicConfig(level=logging.DEBUG)
```

---

## ▶️ Running Tests

```powershell
# All unit tests
poetry run pytest tests/unit/ -v

# Specific file
poetry run pytest tests/unit/services/test_agent_service.py -v

# Specific test class
poetry run pytest tests/unit/services/test_agent_service.py::TestAgentCreate -v

# Specific test method
poetry run pytest tests/unit/services/test_agent_service.py::TestAgentCreate::test_create_success -v

# By keyword (name matching)
poetry run pytest tests/unit/ -k "test_create" -v

# With coverage report
poetry run pytest tests/unit/ --cov=app --cov-report=html
# Open htmlcov/index.html in browser

# Show print statements
poetry run pytest tests/unit/ -v -s

# Stop on first failure
poetry run pytest tests/unit/ -x
```

---

## 🎯 Writing Good Tests (Patterns from This Project)

### Test Naming Convention:
```python
# test_{what}_{scenario}_{expected}
def test_create_agent_with_valid_data_returns_agent():
    ...

def test_create_agent_without_name_raises_validation_error():
    ...

def test_execute_agent_when_not_found_raises_404():
    ...
```

### Test Structure (AAA Pattern):
```python
@pytest.mark.asyncio
async def test_execute_agent_success(self):
    # ARRANGE — set up mocks and data
    agent_id = "test-123"
    mock_agent = {"id": agent_id, "name": "Bot", "tools": ["tool-1"]}
    
    with patch.object(agent_service, 'get_agent', return_value=mock_agent):
        with patch.object(agent_service, '_run_agent', return_value={"output": "Hello"}):
            
            # ACT — call the method under test
            result = await agent_service.execute_agent(agent_id, "Hi")
            
            # ASSERT — verify the result
            assert result["output"] == "Hello"
```

---

## 🔗 pytest vs JUnit Comparison

| pytest | JUnit | Description |
|--------|-------|-------------|
| `def test_xxx()` | `@Test void testXxx()` | Test method |
| `assert x == y` | `assertEquals(y, x)` | Assertion |
| `@pytest.fixture` | `@BeforeEach` | Setup |
| `@pytest.mark.asyncio` | — | Async test |
| `@patch('module.fn')` | `@Mock` | Mock |
| `conftest.py` | `@ExtendWith` | Config |
| `-v` flag | — | Verbose |
| `-k "pattern"` | `@Tag("pattern")` | Filter |
| `pytest.raises(Error)` | `assertThrows()` | Exception test |

### Exception Testing:
```python
import pytest

@pytest.mark.asyncio
async def test_get_agent_not_found():
    """Test that missing agent raises 404"""
    with pytest.raises(ResourceNotFoundException) as exc_info:
        await agent_service.get_agent("nonexistent-id", ...)
    
    assert "not found" in str(exc_info.value.detail)
    assert exc_info.value.code == "agentNotFound"
```

---

## ▶️ Next: [11-deployment-docker.md](./11-deployment-docker.md) — Docker & deployment

