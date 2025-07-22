# Agents vs OpenAI Responses API

Llama Stack (LLS) provides two different APIs for building AI applications with tool calling capabilities: the **Agents API** and the **OpenAI Responses API**. While both enable AI systems to use tools, and maintain full conversation history, they serve different use cases and have distinct characteristics.

> **Note:** For simple and basic inferencing, you may want to use the [Chat Completions API](https://llama-stack.readthedocs.io/en/latest/providers/index.html#chat-completions) directly, before progressing to Agents or Responses API.

## Overview

### LLS Agents API
The Agents API is a full-featured, stateful system designed for complex, multi-turn conversations. It maintains conversation state through persistent sessions identified by a unique session ID. The API supports comprehensive agent lifecycle management, detailed execution tracking, and rich metadata about each interaction through a structured session/turn/step hierarchy. The API can orchestrate multiple tool calls within a single turn.

### OpenAI Responses API
The OpenAI Responses API is a full-featured, stateful system designed for complex, multi-turn conversations, with direct compatibility with OpenAI's conversational patterns enhanced by LLama Stack's tool calling capabilities. It maintains conversation state by chaining responses through a `previous_response_id`, allowing interactions to branch or continue from any prior point. Each response can perform multiple tool calls within a single turn.

### Key Differences
While the LLS Agents API and Responses API offer similar functionality today, the Agents API offers the flexibility to use any agent abstraction and implementation. This allows its features to be extended and customized with new parameters and behaviors to meet specific needs. In contrast, the Responses API is a concrete versioned interface of which features and schema are fixed and cannot be extended.

While the Responses API provides a lightweight, modular interface, it lacks the abstraction needed to support more complex agent behaviors. The LLS Agents API uses the Chat Completions API on the backend for inference as it's the industry standard for building AI applications and most LLM providers are compatible with this API. However, the Agents API can use the Responses API if desired, and this way obtaining the capabilities from both worlds. For a detailed comparison between Responses and Chat Completions, see [OpenAI's documentation](https://platform.openai.com/docs/guides/responses-vs-chat-completions).

Additionally, Agents let you specify input/output shields whereas Responses do not (though support is planned). Agents use a linear conversation model referenced by a single session ID. Responses, on the other hand, support branching, where each response can serve as a fork point, and conversations are tracked by the latest response ID. Responses also lets you dynamically choose the model, vector store, files, MCP servers, and more on each inference call, enabling more complex workflows. Agents require a static configuration for these components at the start of the session.

The Agents and Responses APIs can be used independently depending on the use case. But, it is also productive to treat the APIs as complementary: Responses can serve as the building blocks behind the Agents abstraction, while Agents orchestrate and extend those blocks into cohesive agent systems.

| Feature | LLS Agents API | OpenAI Responses API |
|---------|------------|---------------------|
| **Conversation Management** | Linear persistent sessions | Can branch from any previous response ID |
| **Input/Output Safety Shields** | Supported | Not yet supported |
| **Per-call Flexibility** | Static per-session configuration | Dynamic per-call configuration |

## Use Case Example: Research with Multiple Search Methods

Let's compare how both APIs handle a research task where we need to:
1. Search for current information and examples
2. Access different information sources dynamically
3. Continue the conversation based on search results

### Agents API: Session-based configuration with safety shields

```python
# Create agent with static session configuration
agent = Agent(
    client,
    model="Llama3.2-3B-Instruct",
    instructions="You are a helpful coding assistant",
    tools=[
        {
            "name": "builtin::rag/knowledge_search",
            "args": {"vector_db_ids": ["code_docs"]},
        },
        "builtin::code_interpreter",
    ],
    input_shields=["llama_guard"],
    output_shields=["llama_guard"],
)

session_id = agent.create_session("code_session")

# First turn: Search and execute
response1 = agent.create_turn(
    messages=[
        {
            "role": "user",
            "content": "Find examples of sorting algorithms and run a bubble sort on [3,1,4,1,5]",
        }
    ],
    session_id=session_id,
)

# Continue conversation in same session
response2 = agent.create_turn(
    messages=[
        {
            "role": "user",
            "content": "Now optimize that code and test it with a larger dataset",
        }
    ],
    session_id=session_id,  # Same session, maintains full context
)

# Agents API benefits:
# ✅ Safety shields protect against malicious code execution
# ✅ Session maintains context between code executions
# ✅ Consistent tool configuration throughout conversation
print(f"First result: {response1.output_message.content}")
print(f"Optimization: {response2.output_message.content}")
```

### Responses API: Dynamic per-call configuration with branching

```python
# First response: Use web search for latest algorithms
response1 = client.responses.create(
    model="Llama3.2-3B-Instruct",
    input=[
        {
            "role": "user",
            "content": "Search for the latest efficient sorting algorithms and their performance comparisons",
        }
    ],
    tools=[{"type": "web_search"}],  # Web search for current information
)

# Continue conversation: Switch to file search for local docs
response2 = client.responses.create(
    model="Llama3.2-1B-Instruct",  # Switch to faster model
    input=[
        {
            "role": "user",
            "content": "Now search my uploaded files for existing sorting implementations",
        }
    ],
    tools=[
        {  # Using Responses API built-in tools
            "type": "file_search",
            "vector_store_ids": ["vs_abc123"],  # Vector store containing uploaded files
        }
    ],
    previous_response_id=response1.id,
)

# Branch from first response: Try different search approach
response3 = client.responses.create(
    model="Llama3.2-3B-Instruct",
    input=[
        {
            "role": "user",
            "content": "Instead, search the web for Python-specific sorting best practices",
        }
    ],
    tools=[{"type": "web_search"}],  # Different web search query
    previous_response_id=response1.id,  # Branch from response1
)

# Responses API benefits:
# ✅ Dynamic tool switching (web search ↔ file search per call)
# ✅ OpenAI-compatible tool patterns (web_search, file_search)
# ✅ Branch conversations to explore different information sources
# ✅ Model flexibility per search type
print(f"Web search results: {response1.output_message.content}")
print(f"File search results: {response2.output_message.content}")
print(f"Alternative web search: {response3.output_message.content}")
```

Both APIs demonstrate distinct strengths that make them valuable on their own for different scenarios. The Agents API excels in providing structured, safety-conscious workflows with persistent session management, while the Responses API offers flexibility through dynamic configuration and OpenAI compatible tool patterns. However, if we take into account that we can use any Agents implementation and abstractions, combining it with Responses API allows you to extend functionality while leveraging the flexibility and power of both systems.

## Use Case Examples

### 1. **Research and Analysis with Safety Controls**
**Best Choice: Agents API**

**Scenario:** You're building a research assistant for a financial institution that needs to analyze market data, execute code to process financial models, and search through internal compliance documents. The system must ensure all interactions are logged for regulatory compliance and protected by safety shields to prevent malicious code execution or data leaks.

**Why Agents API?** The Agents API provides persistent session management for iterative research workflows, built-in safety shields to protect against malicious code in financial models, and structured execution logs (session/turn/step) required for regulatory compliance. The static tool configuration ensures consistent access to your knowledge base and code interpreter throughout the entire research session.

### 2. **Dynamic Information Gathering with Branching Exploration**
**Best Choice: Responses API**

**Scenario:** You're building a competitive intelligence tool that helps businesses research market trends. Users need to dynamically switch between web search for current market data and file search through uploaded industry reports. They also want to branch conversations to explore different market segments simultaneously and experiment with different models for various analysis types.

**Why Responses API?** The Responses API's branching capability lets users explore multiple market segments from any research point. Dynamic per-call configuration allows switching between web search and file search as needed, while experimenting with different models (faster models for quick searches, more powerful models for deep analysis). The OpenAI-compatible tool patterns make integration straightforward.

### 3. **OpenAI Migration with Advanced Tool Capabilities**
**Best Choice: Responses API**

**Scenario:** You have an existing application built with OpenAI's Assistants API that uses file search and web search capabilities. You want to migrate to Llama Stack for better performance and cost control while maintaining the same tool calling patterns and adding new capabilities like dynamic vector store selection.

**Why Responses API?** The Responses API provides full OpenAI tool compatibility (`web_search`, `file_search`) with identical syntax, making migration seamless. The dynamic per-call configuration enables advanced features like switching vector stores per query or changing models based on query complexity - capabilities that extend beyond basic OpenAI functionality while maintaining compatibility.

### 4. **Educational Programming Tutor**
**Best Choice: Agents API**

**Scenario:** You're building a programming tutor that maintains student context across multiple sessions, safely executes code exercises, and tracks learning progress with audit trails for educators.

**Why Agents API?** Persistent sessions remember student progress across multiple interactions, safety shields prevent malicious code execution while allowing legitimate programming exercises, and structured execution logs help educators track learning patterns.

### 5. **Advanced Agent Using Responses as Building Blocks**
**Best Choice: Agents API with Responses Backend**

**Scenario:** You're building a sophisticated research agent that needs both the safety/session management of Agents API and the dynamic tool flexibility of Responses API. The agent uses custom logic to decide when to leverage Responses API calls for specific subtasks while maintaining overall agent orchestration and safety controls.

**Why Both APIs?** The Agents API provides the overall session management, safety shields, and structured workflow, while using Responses API internally as building blocks for specific operations that benefit from dynamic model selection, conversation branching for parallel research paths, or OpenAI-compatible tool patterns. This architecture leverages the extensible nature of the Agents abstraction to incorporate Responses capabilities where they add the most value.

## For More Information

- **LLS Agents API**: For detailed information on creating and managing agents, see the [Agents documentation](https://llama-stack.readthedocs.io/en/latest/building_applications/agent.html)
- **OpenAI Responses API**: For information on using the OpenAI-compatible responses API, see the [OpenAI API documentation](https://platform.openai.com/docs/api-reference/responses)
