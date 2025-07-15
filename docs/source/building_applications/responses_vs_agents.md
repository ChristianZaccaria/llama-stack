# Agents vs OpenAI Responses API

Llama Stack provides two different APIs for building AI applications with tool calling capabilities: the **Agents API** and the **OpenAI Responses API**. While both enable AI systems that can use tools, they serve different use cases and have distinct characteristics.

## Overview

### Agents API
The Agents API is a full-featured, stateful system designed for complex, multi-turn conversations with persistent sessions. It provides comprehensive agent management, detailed execution tracking, and rich metadata about each interaction.

### OpenAI Responses API
The OpenAI Responses API is a lightweight, stateless system that provides a simple interface for single-turn tool calling. It's designed to be compatible with OpenAI's API patterns while adding Llama Stack's tool calling capabilities.

## Key Differences

| Feature | Agents API | OpenAI Responses API |
|---------|------------|---------------------|
| **State Management** | Full session persistence with turns, steps, and metadata | Stateless - each request is independent |
| **Complexity** | High - complete agent lifecycle management | Low - simple request/response pattern |
| **Use Case** | Multi-turn conversations, complex workflows | Single-turn tool calling, simple integrations |
| **Execution Tracking** | Detailed step-by-step execution logs | Basic response with tool calls |
| **Session Management** | Built-in session creation and management | No session concept |

## Use Case Examples

### When to Use the Agents API

#### 1. **Customer Support Chatbot**
You're building a customer support system that needs to maintain context across multiple interactions.

**Scenario**: A customer contacts support about a login issue. They try the suggested password reset, but it doesn't work. They come back later with more details about the error message they received.

**Why Agents API?** The session persistence allows the agent to remember the customer's previous issues, maintain conversation context, and provide personalized support over time. The agent can reference the earlier conversation about the login problem and build upon that context when the customer returns with additional information.

#### 2. **Research Assistant with Memory**
You need an AI assistant that can conduct multi-step research and remember findings across sessions.

**Scenario**: You're researching AI safety regulations. On day one, you ask the assistant to gather information about current regulations. On day two, you ask for updates on recent developments since your last session.

**Why Agents API?** The persistent session allows the research assistant to build upon previous findings, maintain a research log, and provide continuity across multiple research sessions. The assistant can reference what was found on day one and focus on new developments since then.

#### 3. **Complex Workflow Orchestration**
You're building a system that needs to coordinate multiple tools and maintain state across complex workflows.

**Scenario**: You're analyzing sales data through a multi-step process. First, you load and clean the data. Then, you want to analyze trends and create visualizations based on the cleaned data.

**Why Agents API?** The workflow can span multiple turns, maintain intermediate results, and provide detailed execution tracking for debugging and monitoring. The agent remembers the cleaned data from the first step and can use it in the analysis phase without having to reload or reprocess it.

### When to Use the OpenAI Responses API

#### 1. **Simple Tool Integration**
You want to add tool calling to an existing application without complex session management.

**Scenario**: You have a weather app that needs to get current weather information. Each request is independent - users ask about different cities, and you don't need to remember previous weather queries.

**Why OpenAI Responses API?** Simple, stateless, and compatible with existing OpenAI client libraries. Perfect for one-off tool calls without needing session management. Each weather request is independent and doesn't require context from previous interactions.

#### 2. **API Gateway Integration**
You're building an API gateway that needs to route requests to different AI services.

**Scenario**: You have a system that receives various types of queries - password resets, office hours, product information. Each query is processed independently without needing to maintain conversation state.

**Why OpenAI Responses API?** Stateless design makes it perfect for API gateways where you don't need to maintain conversation state. Each request can be processed independently, making it easy to scale and route requests to different services.

#### 3. **Batch Processing**
You need to process multiple independent requests without maintaining state between them.

**Scenario**: You have a CSV file containing thousands of short product descriptions, and you want to generate concise marketing taglines for each product. Each tagline generation is independent.

**Why OpenAI Responses API?** The stateless nature of the Responses API is perfect for this case. You can submit each product description as a separate request (even in parallel), get a tagline back, and aggregate results efficiently. There’s no need for conversation memory, session management, or threading.

#### 4. **Testing and Prototyping**
You're quickly prototyping a tool integration or testing different models.

**Scenario**: You’re building a proof-of-concept for automated email summarization. You want to test how different models or prompt templates summarize individual emails, so you feed a batch of sample emails to the Responses API and compare outputs. Each input is independent.

**Why OpenAI Responses API?** Its stateless, on-demand design lets you rapidly try different models, prompts, or toolchains. You can adjust parameters, swap models, and process test inputs in parallel, all without handling any persistent state or user context.

## Complementary Usage

You can also use both APIs together in the same application. For example, you might use the OpenAI Responses API for simple, one-off queries like quick searches or calculations, while using the Agents API for complex conversations that require context and memory, such as customer support interactions or research projects.

This hybrid approach allows you to optimize for different use cases within the same application - using the right tool for each specific need.


## For More Information

- **Agents API**: For detailed information on creating and managing agents, see the [Agents documentation](agent.md)
- **OpenAI Responses API**: For information on using the OpenAI-compatible responses API, see the [OpenAI API documentation](../openai/index.md)
