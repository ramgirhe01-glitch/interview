# 09 — MCP Tools (Model Context Protocol)

> MCP allows AI agents to use external tools (APIs, databases, calculators, etc.)

---

## 🎯 What is MCP?

**MCP (Model Context Protocol)** is a standard for connecting AI agents to external tools:

- **Tool Servers** = Microservices that expose functions (like REST APIs for AI)
- **Agents call tools** when they need to do something (search, calculate, query DB)
- **Standard protocol** = any MCP server works with any MCP-compatible agent

### Think of it like:
```
Agent (LLM brain) ──MCP Protocol──▶ Tool Server (hands/actions)
                                         │
                                         ├── Calculator tool
                                         ├── Database search tool
                                         └── API call tool
```

---

## 🏗️ Architecture in This Project

```
┌──────────────┐         ┌──────────────────┐
│   Agent      │──MCP──▶ │  MCP Tool Server  │
│  (LangGraph) │         │  (external)       │
└──────────────┘         └──────────────────┘
       │
       │  tools defined in
       ▼
┌──────────────┐
│  DB: tools   │  (server_url, tool_name, auth)
│  table       │
└──────────────┘
```

### Flow:
1. User creates a **tool** in the system (name, MCP server URL)
2. User assigns the **tool to an agent**
3. When agent executes, it connects to the MCP server
4. LLM decides WHEN to call tools based on the conversation
5. Tool results are fed back to the LLM for final response

---

## 📋 Tool Configuration

### Creating a Tool (via API):
```json
POST /tools
{
    "name": "calculator",
    "description": "Performs mathematical calculations",
    "server_url": "http://localhost:9000/mcp",
    "tool_type": "mcp",
    "auth_type": "none"
}
```

### Assigning Tools to Agent:
```json
PUT /agents/{agent_id}
{
    "tools": ["tool-id-1", "tool-id-2"]
}
```

---

## 🔧 MCP Server Examples (in this project)

The `mcp_servers/` folder has sample servers for testing:

### `math_server.py` — Calculator:
```python
from mcp.server import Server
from mcp.types import Tool, TextContent

server = Server("math-server")

@server.tool()
async def calculate(expression: str) -> str:
    """Calculate a mathematical expression."""
    result = eval(expression)  # In production, use safe eval!
    return str(result)

@server.tool()
async def fibonacci(n: int) -> str:
    """Calculate nth Fibonacci number."""
    a, b = 0, 1
    for _ in range(n):
        a, b = b, a + b
    return str(a)

# Run server
if __name__ == "__main__":
    server.run(transport="sse", port=9000)
```

### `english_server.py` — Text Processing:
```python
@server.tool()
async def translate(text: str, target_language: str) -> str:
    """Translate text to target language."""
    ...

@server.tool()
async def summarize(text: str, max_words: int = 100) -> str:
    """Summarize text."""
    ...
```

---

## 🔄 How Tools Are Used During Execution

### Step 1: Tools Loaded at Agent Build Time
```python
# In agent_creation.py
async def _load_mcp_tools(self, tool_configs):
    """Connect to MCP servers and load available tools."""
    tools = []
    for tool_config in tool_configs:
        # Connect to MCP server
        session = await connect_mcp_server(tool_config.server_url)
        # Get available tools from server
        server_tools = await session.list_tools()
        tools.extend(server_tools)
    return tools
```

### Step 2: LLM Decides to Use a Tool
```
User: "What is 15 * 23?"

LLM thinks: "I need to calculate this. I have a calculator tool."
LLM output: {tool_call: "calculate", args: {"expression": "15 * 23"}}

System: Calls calculate("15 * 23") on MCP server
System: Gets result "345"
System: Feeds "345" back to LLM

LLM: "15 × 23 = 345"
```

### Step 3: Tool Results Fed Back to LLM
```python
# LangGraph handles this automatically:
# 1. LLM generates tool_call message
# 2. Tool node executes the tool
# 3. ToolMessage with result added to conversation
# 4. LLM gets the result and generates final response
```

---

## 🗄️ Tool Cache (`utils/mcp_tool_cache.py`)

MCP connections are expensive. This project caches them:

```python
class MCPToolCache:
    """Cache MCP server connections per request."""
    
    def __init__(self):
        self._cache = {}  # server_id → session
    
    async def get_session(self, server_url: str):
        """Get or create MCP session (cached per request)."""
        if server_url not in self._cache:
            self._cache[server_url] = await connect(server_url)
        return self._cache[server_url]
    
    async def cleanup(self):
        """Close all sessions after request completes."""
        for session in self._cache.values():
            await session.close()
        self._cache.clear()

mcp_tool_cache = MCPToolCache()
```

---

## 🔐 Tool Authentication Types

| Type | Description | Example |
|------|-------------|---------|
| `none` | No auth needed | Local dev servers |
| `bearer` | Bearer token in header | Internal services |
| `api_key` | API key in header | Third-party APIs |
| `oauth2` | OAuth2 flow | External services |

```python
# Tool with authentication
{
    "name": "internal_search",
    "server_url": "https://search.internal.com/mcp",
    "auth_type": "bearer",
    "auth_config": {
        "token_source": "request_token"  # Forward user's token
    }
}
```

---

## 🧪 Running a Sample MCP Server Locally

```powershell
# Start a math MCP server for testing
cd app/api/mcp_servers
poetry run python math_server.py

# Server runs on http://localhost:9000
# Now create a tool pointing to this server
```

---

## 📊 Tool Types in This Project

```python
class ToolType(str, Enum):
    MCP = "mcp"              # Standard MCP server
    FUNCTION = "function"     # Built-in function tools
    KNOWLEDGE_BANK = "kb"    # Knowledge base search
```

---

## 💡 Key Points

1. **MCP is a protocol** — standardizes how agents talk to tools
2. **Tools are external** — separate processes/servers
3. **Agents discover tools** at build time by connecting to MCP servers
4. **LLM decides** when to call tools (not hardcoded)
5. **Tool results** go back to LLM as messages
6. **Caching** avoids reconnecting on every request
7. **Cleanup** happens via FastAPI dependency (`cleanup_mcp_sessions` in `main.py`)

---

## ▶️ Next: [10-testing-guide.md](./10-testing-guide.md) — Writing and running tests

