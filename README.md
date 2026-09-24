# 🧠 LLM Memory System

> **A practical implementation of Short-Term and Long-Term Memory for LLM applications using LangGraph, LangChain, OpenAI, Pydantic, and PostgreSQL.**

LLM applications are naturally **stateless**: unless previous information is explicitly provided to the model, each request is treated independently.

This project explores how to build a **stateful LLM system** that can maintain conversational context, persist user-specific information, intelligently decide what is worth remembering, and reuse those memories to generate personalized responses.

The implementation progresses from **in-memory conversational state** to **persistent long-term memory backed by PostgreSQL**.

---

## 🚀 What This Project Demonstrates

* **Short-Term Memory (STM)** for maintaining conversational context within a thread.
* **Checkpoint-based persistence** for continuing conversations across executions.
* **Message trimming** to control context-window growth.
* **Message deletion** for managing conversation state.
* **Conversation summarization** to compress older context.
* **Long-Term Memory (LTM)** for storing user-specific information across conversations.
* **LLM-powered memory extraction** using structured Pydantic outputs.
* **Duplicate-aware memory creation** to avoid unnecessarily storing existing information.
* **Personalized responses** by retrieving relevant user memories before inference.
* **PostgreSQL-backed persistent memory** for durable storage.
* **Dockerized PostgreSQL** for reproducible local development.

---

## 🏗️ Architecture

The project separates memory into two major layers:

```text
                         ┌─────────────────────┐
                         │       User          │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │    LangGraph       │
                         │   Application      │
                         └──────────┬──────────┘
                                    │
                  ┌─────────────────┴─────────────────┐
                  │                                   │
                  ▼                                   ▼
        ┌───────────────────┐              ┌────────────────────┐
        │ Short-Term Memory │              │  Long-Term Memory  │
        │       (STM)       │              │       (LTM)        │
        └─────────┬─────────┘              └──────────┬─────────┘
                  │                                   │
        ┌─────────┴─────────┐             ┌───────────┴──────────┐
        │                   │             │                      │
        ▼                   ▼             ▼                      ▼
   Conversation        Checkpoints   Memory Extraction     PostgreSQL
   State               Persistence   + Deduplication       Persistence
```

---

# 🧩 Short-Term Memory

Short-term memory represents information required during an active conversation.

### Implemented Concepts

### 1. Basic Conversation State

The project uses LangGraph's message state to maintain the current conversation.

```text
User Message
     ↓
MessagesState
     ↓
LLM
     ↓
Assistant Response
```

This allows the model to access previous messages within the active interaction.

---

### 2. Persistence

The project explores checkpoint-based persistence so that graph state can survive beyond a single invocation.

This introduces the concept of:

```text
Thread
  ↓
State
  ↓
Checkpoint
  ↓
Future Invocation
```

A `thread_id` can therefore identify an independent conversation.

---

### 3. Message Trimming

As conversations become longer, sending the complete history to an LLM becomes expensive and can exceed the model's context window.

The project demonstrates **trimming old messages** to control:

* Context size
* Token usage
* Latency
* Prompt growth

Conceptually:

```text
Old Messages ────────┐
                     ▼
              ┌─────────────┐
              │ Trim Policy │
              └──────┬──────┘
                     │
                     ▼
              Relevant Context
                     │
                     ▼
                    LLM
```

---

### 4. Message Deletion

The project also explores explicitly removing message
