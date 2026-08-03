# 07 — LangChain & LangGraph (AI Agent Engine)

> The CORE of this project. LangGraph builds AI agents as state machines.

---

## 🎯 What Are These Libraries?

| Library | Purpose | Analogy |
|---------|---------|---------|
| **LangChain** | Connect to LLMs, tools, memory | Framework for AI apps |
| **LangGraph** | Build agent workflows as graphs | State machine for AI agents |
| **LangChain-AWS** | AWS Bedrock LLM integration | LLM provider |
| **LangChain-OpenAI** | OpenAI LLM integration | Alternative LLM provider |

### Think of it like:
- **LangChain** = The building blocks (LLMs, tools, prompts)
- **LangGraph** = The orchestration engine (decides what to do next)

---

## 🧠 Key Concepts

### 1. LLM (Large Language Model)
The AI brain. This project uses **AWS Bedrock** (Claude, etc.):

```python
from langchain_aws import ChatBedrock

# Create an LLM instance
llm = ChatBedrock(
    model_id="anthropic.claude-3-sonnet-20240229-v1:0",
    region_name="us-east-1",
    model_kwargs={
        "temperature": 0.7,
        "max_tokens": 4096,
    }
)

# Ask the LLM something
response = await llm.ainvoke("What is 2+2?")
print(response.content)  # "2+2 = 4"
```

### 2. Messages (Conversation History)
```python
from langchain_core.messages import HumanMessage, AIMessage, SystemMessage

messages = [
    SystemMessage(content="You are a helpful assistant."),  # System prompt
    HumanMessage(content="What is Python?"),                 # User input
    AIMessage(content="Python is a programming language.")   # AI response
]
```

### 3. Tools (Functions the AI Can Call)
```python
from langchain_core.tools import tool

@tool
def calculate(expression: str) -> str:
    """Calculate a math expression."""
    return str(eval(expression))

@tool
def search_database(query: str) -> str:
    """Search the internal database."""
    # ... actual search logic
    return results
```

---

## 🔄 LangGraph: The State Machine

LangGraph models agents as **directed graphs** where:
- **Nodes** = Actions (call LLM, use tool, check guardrails)
- **Edges** = Transitions (what to do next)
- **State** = Data passed between nodes

### Simple Agent Graph:
```
┌─────────┐     ┌──────────┐     ┌─────────┐
│  START   │────▶│  Agent   │────▶│   END   │
│          │     │ (LLM)   │     │         │
└─────────┘     └────┬─────┘     └─────────┘
                     │
                     │ needs tool?
                     ▼
                ┌──────────┐
                │  Tools   │
                │ (Execute)│
                └────┬─────┘
                     │
                     │ return result
                     └──────▶ back to Agent
```

### Code:
```python
from langgraph.graph import StateGraph, START, END
from langgraph.prebuilt import create_react_agent

# Simple way — prebuilt ReAct agent
agent = create_react_agent(
    model=llm,
    tools=[calculate, search_database],
    state_modifier="You are a helpful math tutor."  # System prompt
)

# Execute
result = await agent.ainvoke({
    "messages": [HumanMessage(content="What is 15 * 23?")]
})
```

---

## 🏗️ How This Project Builds Agents

### Agent Creation (`services/agent/agent_creation.py`):

```python
class AgentCreation:
    """Builds LangGraph agent apps from database configurations."""
    
    async def build_agent_app(self, agent_config, tools, ...) -> AgentApp:
        """
        1. Load LLM (from agent config)
        2. Load tools (MCP tools from config)
        3. Build LangGraph graph
        4. Attach guardrails
        5. Return executable app
        """
        # Step 1: Get the LLM
        llm = await self._get_llm(agent_config)
        
        # Step 2: Get tools from MCP servers
        mcp_tools = await self._load_mcp_tools(agent_config.tools)
        
        # Step 3: Build the LangGraph agent
        agent_app = create_react_agent(
            model=llm,
            tools=mcp_tools,
            state_modifier=agent_config.system_prompt,
            checkpointer=checkpointer,  # State persistence
        )
        
        return AgentApp(app=agent_app, config=agent_config)
```

### Agent Execution (`services/agent/agent_execution.py`):

```python
class AgentExecution:
    """Runs the built agent."""
    
    async def execute(self, agent_app, instruction: str, session_id: str):
        """
        1. Create input message
        2. Run LangGraph agent
        3. Stream or collect results
        4. Track execution in DB
        """
        input_messages = [HumanMessage(content=instruction)]
        
        # Run the agent
        config = {"configurable": {"thread_id": session_id}}
        result = await agent_app.ainvoke(
            {"messages": input_messages},
            config=config,
        )
        
        return result["messages"][-1].content  # Last AI message
```

---

## 📊 State Graph (Custom Workflows)

For more complex agents, this project builds custom state graphs:

```python
from langgraph.graph import StateGraph, START, END
from typing import TypedDict, Annotated
from langgraph.graph.message import add_messages

# Define state shape
class AgentState(TypedDict):
    messages: Annotated[list, add_messages]  # Conversation history
    context: dict                            # Additional context
    tool_results: list                       # Tool execution results

# Build graph
graph = StateGraph(AgentState)

# Add nodes (processing steps)
graph.add_node("agent", call_llm)           # LLM reasoning
graph.add_node("tools", execute_tools)       # Tool execution
graph.add_node("guardrail", check_guardrails)  # Safety checks

# Add edges (transitions)
graph.add_edge(START, "guardrail")           # Start → guardrail check
graph.add_edge("guardrail", "agent")         # Then → LLM
graph.add_conditional_edges(                  # LLM decides next step
    "agent",
    should_use_tool,                          # Function that decides
    {True: "tools", False: END}              # Routing map
)
graph.add_edge("tools", "agent")             # Tool result → back to LLM

# Compile into runnable
app = graph.compile(checkpointer=checkpointer)
```

---

## 🔧 Checkpointing (State Persistence)

LangGraph can **save state** between executions (conversation memory):

```python
from langgraph.checkpoint.postgres.aio import AsyncPostgresSaver

# Create checkpointer (saves state to PostgreSQL)
checkpointer = AsyncPostgresSaver(connection_pool)

# Use in agent
app = graph.compile(checkpointer=checkpointer)

# First execution
config = {"configurable": {"thread_id": "session-123"}}
result1 = await app.ainvoke({"messages": [HumanMessage("Hi!")]}, config)

# Second execution — REMEMBERS the first!
result2 = await app.ainvoke({"messages": [HumanMessage("What did I say?")]}, config)
# AI knows you said "Hi!" because state is saved
```

---

## 🛡️ Guardrails Integration

This project adds guardrails to agents:

```python
# Guardrails are callback handlers attached to the LLM
class GuardrailCallbackHandler:
    """Checks input/output against rules"""
    
    async def on_llm_start(self, prompt, **kwargs):
        """Check INPUT before sending to LLM"""
        if contains_pii(prompt):
            raise GuardrailViolation("PII detected in input")
    
    async def on_llm_end(self, response, **kwargs):
        """Check OUTPUT after receiving from LLM"""
        if is_harmful(response):
            raise GuardrailViolation("Harmful content in output")
```

---

## 🔄 Three Execution Modes in This Project

### Mode 1: Standalone Agent
```python
# Simple: one agent, one execution
POST /agents/{id}/execute
Body: {"instruction": "Calculate 15 * 23"}
```

### Mode 2: Custom Workflow (DAG)
```python
# Multiple agents connected in a user-defined graph
# Agent A → Agent B → Agent C (sequential)
# Agent A → [Agent B, Agent C] (parallel)
POST /custom-workflows/{id}/execute
```

### Mode 3: Dynamic Workflow (Supervisor)
```python
# A "supervisor" agent decides which sub-agents to call
# Like a team lead delegating tasks
POST /dynamic-workflows/{id}/execute
```

---

## 🎯 Key LangGraph Concepts Table

| Concept | What It Does | In This Project |
|---------|-------------|-----------------|
| `StateGraph` | Define workflow structure | `agent_creation.py` |
| `create_react_agent` | Quick agent builder | Simple agents |
| `Node` | Processing step | LLM call, tool call |
| `Edge` | Connection between nodes | Flow control |
| `Conditional Edge` | Dynamic routing | "Use tool?" decision |
| `Checkpointer` | Save/restore state | Memory across sessions |
| `HumanMessage` | User input | Start of execution |
| `AIMessage` | LLM response | Agent output |
| `ToolMessage` | Tool result | After tool execution |
| `Command` | Control flow instruction | Resume after HITL |

---

## 🤖 Human-in-the-Loop (HITL)

Agents can **pause** for human approval:

```python
from langgraph.types import interrupt

# In a node function:
async def sensitive_action(state):
    """Node that requires human approval"""
    action = state["proposed_action"]
    
    # Pause execution — returns to user
    human_response = interrupt(
        {"question": f"Approve action: {action}?", "options": ["yes", "no"]}
    )
    
    if human_response == "yes":
        return {"result": await execute_action(action)}
    else:
        return {"result": "Action cancelled by user"}
```

---

## 📚 Learning Path for LangGraph

1. **Read:** [LangGraph docs](https://langchain-ai.github.io/langgraph/)
2. **Trace:** `services/agent/agent_creation.py` → `agent_execution.py`
3. **Run:** A sample MCP server from `mcp_servers/math_server.py`
4. **Debug:** Set breakpoints in `_run_langgraph_agent()` in agent_service.py

---

## ▶️ Next: [08-services-layer.md](./08-services-layer.md) — Business logic patterns

