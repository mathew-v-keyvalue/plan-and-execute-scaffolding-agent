# SearchAgent - Advanced Plan-Execute Agent with Web Search

An enhanced plan-execute agent that combines **web search capabilities** with **project scaffolding** using LangGraph's ReAct pattern and replanning capabilities.

## 🎯 Overview

This agent demonstrates an advanced **Plan-Execute-Replan** pattern where:
1. **Planner** creates an initial step-by-step plan
2. **Executor** (ReAct Agent) executes each step using available tools
3. **Replanner** evaluates progress and either:
   - Continues with remaining steps
   - Updates the plan based on observations
   - Returns final response when complete

## 🏗️ Architecture

```
┌─────────────┐
│   Input     │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Planner    │ ◄─── Creates initial plan using LLM
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Agent     │ ◄─── Executes step using ReAct pattern
│ (Executor)  │      with tools (search, execute_command)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Replanner  │ ◄─── Evaluates progress & decides next action
└──────┬──────┘
       │
       ├─── More steps? ──► Loop back to Agent
       │
       └─── Done? ────────► Return Response
```

## 🔧 Key Features

### 1. **Dual-Purpose Agent**
- **Web Search**: Answer general knowledge questions using Tavily search
- **Project Scaffolding**: Create project structures with files and directories

### 2. **ReAct Execution Pattern**
- Uses `create_react_agent` for intelligent tool selection
- Automatic reasoning about which tool to use
- Handles complex multi-step tasks

### 3. **Dynamic Replanning**
- Evaluates progress after each step
- Can update the plan based on observations
- Decides when task is complete

### 4. **Structured Output**
- Uses Pydantic models for type-safe planning
- Clear separation between plans and responses

## 🚀 Getting Started

### Prerequisites

```bash
Python 3.8+
OpenAI API Key
Tavily API Key (optional, for web search)
```

### Installation

1. Install dependencies:
```bash
pip install -r requirements.txt
```

2. Set up environment variables in `.env`:
```bash
OPENAI_API_KEY=your_openai_api_key
TAVILY_API_KEY=your_tavily_api_key  # Optional
```

### Usage

Run the search agent:
```bash
python SearchAgnet.py
```

## 📊 Example Use Cases

### 1. General Knowledge Question
```python
asyncio.run(
    run_example({
        "input": "what is the hometown of the mens 2024 Australia open winner?"
    })
)
```

**Flow:**
1. Planner: Create steps to search and find answer
2. Agent: Use Tavily search tool to find information
3. Replanner: Verify answer and return response

### 2. Project Scaffolding
```python
asyncio.run(
    run_example({
        "input": "Set up a Python project 'my_new_project'. It needs a 'src' folder, an empty '__init__.py' in 'src', a 'main.py' in 'src' printing 'Hello Scaffolding!', and a 'README.md' at the root with title 'My New Project'."
    })
)
```

**Flow:**
1. Planner: Break down into mkdir, touch, write file commands
2. Agent: Execute each command using execute_command tool
3. Replanner: Verify all steps complete and return success

## 🔑 Key Components

### State Management
```python
class PlanExecute(TypedDict):
    input: str                              # Original user query
    plan: List[str]                         # Current plan steps
    past_steps: List[Tuple[str, str]]       # (step, observation) pairs
    response: str                           # Final response
    messages: List[BaseMessage]             # Conversation history
```

### Pydantic Models
```python
class Plan(BaseModel):
    steps: List[str]  # Ordered list of steps

class Response(BaseModel):
    response: str  # Final answer to user

class Act(BaseModel):
    action: Union[Response, Plan]  # Either respond or continue planning
```

### Tools

**1. execute_command** - Project scaffolding
- `mkdir <dir>` - Create directories
- `touch <file>` - Create files
- `write file <file> content: <content>` - Write file contents

**2. tavily_tool** - Web search (optional)
- Searches the web for factual information
- Returns top 3 results

## 🎓 Advanced Features

### Replanning Logic

The replanner uses structured output to decide:
```python
async def replan_step(state: PlanExecute):
    output = await replanner.ainvoke(state)
    if isinstance(output.action, Response):
        # Task complete - return final response
        return {"response": output.action.response}
    else:
        # More work needed - update plan
        return {"plan": output.action.steps}
```

### Conditional Routing

```python
def should_end(state: PlanExecute) -> str:
    if "response" in state and state["response"]:
        return "__end__"  # Task complete
    elif not state.get("plan", []):
        return "__end__"  # No more steps
    else:
        return "agent"  # Continue execution
```

## 🔄 Workflow Comparison

### SearchAgent (This File) vs main.py

| Feature | SearchAgent.py | main.py |
|---------|---------------|---------|
| **Pattern** | Plan-Execute-Replan | Plan-Execute |
| **Executor** | ReAct Agent | Tool Calling |
| **Tools** | Search + Scaffolding | Scaffolding only |
| **Replanning** | ✅ Dynamic | ❌ Static plan |
| **Use Case** | General Q&A + Projects | Projects only |
| **Complexity** | Higher | Lower |

## 🛠️ Customization

### Add More Tools

```python
@tool
def calculator(expression: str) -> str:
    """Evaluate a mathematical expression"""
    return str(eval(expression))

tools = [execute_command, tavily_tool, calculator]
```

### Adjust Recursion Limit

```python
config = {"recursion_limit": 100}  # Increase for complex tasks
```

### Change LLM Model

```python
llm = ChatOpenAI(
    model="gpt-4-turbo",  # or "gpt-3.5-turbo"
    temperature=0.7,       # Adjust creativity
)
```

## 📝 Notes

- **Async/Await**: This implementation uses async for better performance
- **Structured Output**: Requires OpenAI models that support function calling
- **Tavily Optional**: Works without Tavily for scaffolding-only tasks
- **Recursion Limit**: Set to 50 by default to prevent infinite loops

## 🐛 Troubleshooting

### "Tavily tool not available"
- This is normal if you don't have TAVILY_API_KEY set
- Agent will still work for project scaffolding tasks

### "Recursion limit reached"
- Increase `recursion_limit` in config
- Check if replanner is properly detecting completion

### "No response generated"
- Ensure the task is clear and achievable
- Check that appropriate tools are available

## 🤝 Contributing

This is a learning example. Feel free to:
- Add more tools
- Improve the replanning logic
- Add error handling
- Create more sophisticated prompts

## 📄 License

MIT License

---

**Built with ❤️ using LangChain, LangGraph, and OpenAI**
