Yes. Since you’ll use Copilot on the office laptop, give it a prompt that asks for a basic developer-made HLD, not an elaborate enterprise architecture.

From Akhil’s message, the core backend flow is:

User Query → RAG/Semantic Understanding → Calculation Engine → Output Layer → Response

Alongside this, you need chat UUID + local conversation history.

Use this prompt:

Prompt for Copilot

I need to create a simple High-Level Design (HLD) / workflow diagram for a standalone chatbot feature.

Please create a simple architecture that looks like it was manually prepared by a developer for an internal technical discussion. Do not make it overly detailed, enterprise-level, or visually complicated.

The chatbot backend needs to contain these main components:

1. UI / Chat Interface
    * User enters a question.
    * Each conversation has a unique chat_id / UUID.
    * UI sends the chat_id and current user message to the backend.
2. Chat Backend / API
    * Receives the user query and chat_id.
    * Coordinates the complete chatbot workflow.
3. Conversation History
    * Maintain chat history locally for now.
    * Use something lightweight such as DuckDB.
    * Store messages against the chat_id.
    * When a user continues an existing conversation, retrieve the relevant previous messages to maintain context.
4. RAG / Semantic Understanding
    * Understand the meaning and intent of the user’s question.
    * Retrieve relevant information/context required for answering the query.
    * Pass the relevant context to the next processing layer.
5. Calculation Engine
    * Perform any required calculations or deterministic business logic based on the interpreted query and retrieved information.
    * Keep this separate from the RAG/LLM logic.
6. Output Layer
    * Take the result from the calculation engine and/or RAG layer.
    * Prepare a user-friendly final response.
    * Return the response to the UI.
7. History Update
    * Store both the user’s message and final bot response against the same chat_id.

Show the main workflow approximately as:

User
↓
Chat UI
↓
Backend API
↓
Load Chat History using chat_id
↓
RAG / Semantic Understanding
↓
Calculation Engine
↓
Output Layer
↓
Backend Response
↓
Chat UI

Also show:

Local Chat History (DuckDB)
↕
Backend API

and

Knowledge/Data Source
↓
RAG

The final HLD should be simple enough to fit on one page.

Use only basic rectangular boxes, arrows, and short labels. Avoid Kubernetes, microservices, queues, load balancers, cloud infrastructure, authentication layers, monitoring, caching, or other components unless they are absolutely necessary for this workflow.

First provide:

1. A simple Mermaid flowchart.
2. A short explanation of each component.
3. The end-to-end flow in 6–8 simple steps.

Do not make assumptions about the existing codebase. Mark anything dependent on the existing implementation as “To be confirmed after code review.”

One thing I would specifically keep

Don’t ask Copilot to make a fancy architecture diagram. Your leader specifically wants to see your plan before coding.

Your diagram should essentially look like this:

                ┌─────────────────┐
                │    Chat UI      │
                │ Message + UUID  │
                └────────┬────────┘
                         ↓
                ┌─────────────────┐
                │   Backend API   │◄────► Local History
                └────────┬────────┘       (DuckDB)
                         ↓
                ┌─────────────────┐
                │ RAG / Semantic  │◄──── Knowledge/Data
                │ Understanding   │
                └────────┬────────┘
                         ↓
                ┌─────────────────┐
                │   Calculation   │
                │     Engine      │
                └────────┬────────┘
                         ↓
                ┌─────────────────┐
                │  Output Layer   │
                └────────┬────────┘
                         ↓
                    Chat Response

This directly reflects the three things Akhil explicitly mentioned — RAG, Calculation Engine, Output Layer — while adding the UUID/history requirement you were already discussing. It should therefore look like a straightforward architecture you derived from the team’s discussion rather than something unnecessarily sophisticated.
