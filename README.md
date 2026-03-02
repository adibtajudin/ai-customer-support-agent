# AI Customer Support Agent

A production-ready conversational support agent that combines large language model inference with persistent, per-user memory. Built with Groq, Mem0, and Qdrant — the agent maintains full context of each customer's order history, past interactions, and preferences across sessions, enabling accurate and personalised support at scale.

---

## Overview

Traditional chatbots lose context the moment a session ends. This agent solves that by maintaining a persistent memory layer per customer. When a customer returns, the agent already knows their order history, past issues, and preferences — without them having to repeat themselves.

The stack is designed for low-latency inference (Groq), semantic memory retrieval (Mem0 + Qdrant), and rapid front-end prototyping (Streamlit).

---

## Features

- **Persistent Per-User Memory** — Customer context survives across sessions via Mem0 and Qdrant
- **Accurate Order Retrieval** — Customer profile injected directly into the LLM context for precise, hallucination-free order responses
- **Synthetic Data Generation** — Generate realistic customer profiles and order histories on demand for testing
- **Conversation History** — Full multi-turn conversation context passed to the LLM on every query
- **Memory Management** — View, refresh, and clear per-customer memory from the UI

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│                  Streamlit UI                   │
│         (Chat Interface + Sidebar Controls)     │
└───────────────────┬─────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────┐
│           CustomerSupportAIAgent                │
│                                                 │
│  ┌─────────────┐      ┌──────────────────────┐  │
│  │  Groq API   │      │        Mem0          │  │
│  │  (LLM)      │      │  (Memory Abstraction)│  │
│  └─────────────┘      └──────────┬───────────┘  │
│                                  │              │
└──────────────────────────────────┼──────────────┘
                                   │
                    ┌──────────────▼───────────────┐
                    │         Qdrant               │
                    │   (Vector Store, port 6333)  │
                    └──────────────────────────────┘
```

**Request flow:**
1. Customer query received via Streamlit chat input
2. Full customer profile (JSON) loaded from session state
3. Conversation history retrieved from session state
4. System prompt constructed with profile + history
5. Groq LLM (`llama-3.3-70b-versatile`) generates response
6. Interaction stored in Mem0/Qdrant for future sessions

---

## Tech Stack

| Component | Technology |
|---|---|
| LLM Inference | [Groq](https://console.groq.com) — `llama-3.3-70b-versatile` |
| Memory Layer | [Mem0](https://mem0.ai) |
| Vector Database | [Qdrant](https://qdrant.tech) |
| Embeddings | HuggingFace `multi-qa-MiniLM-L6-cos-v1` (local) |
| Frontend | [Streamlit](https://streamlit.io) |
| LLM Client | OpenAI SDK (Groq-compatible endpoint) |

---

## Prerequisites

- Python 3.10+
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- A [Groq API key](https://console.groq.com) (free tier available)

---

## Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/adibtajudin/ai-customer-support-agent.git
cd ai-customer-support-agent
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
pip install sentence-transformers groq
```

> On first run, the HuggingFace embedding model (~90MB) will be downloaded and cached automatically.

### 3. Start Qdrant

```bash
# First time
docker run -d --name qdrant -p 6333:6333 -p 6334:6334 \
  -v "$(pwd)/qdrant_storage:/qdrant/storage:z" \
  qdrant/qdrant

# Subsequent runs
docker start qdrant
```

### 4. Start the application

```bash
python -m streamlit run customer_support_agent.py --server.headless true
```

Navigate to `http://localhost:8501`

---

## Usage

1. **Enter your Groq API key** in the text field
2. **Enter a Customer ID** in the sidebar (e.g. `cust_001`)
3. **Click "Generate Synthetic Data"** to create a realistic customer profile with order history
4. **Start chatting** — ask about orders, returns, delivery status, product recommendations

The agent will answer directly using the customer's profile without asking for information it already has.

### Example queries

```
What is the status of my recent order?
When will my package arrive?
I want to return my last purchase.
What did I order last time?
My item arrived damaged — what are my options?
```

---

## Configuration

All configuration is handled at runtime via the Streamlit UI. The key parameters in `customer_support_agent.py`:

| Parameter | Default | Description |
|---|---|---|
| `model` | `llama-3.3-70b-versatile` | Groq LLM model |
| `host` | `localhost` | Qdrant host |
| `port` | `6333` | Qdrant port |
| `embedding_model_dims` | `384` | HuggingFace embedding dimensions |
| `embedder model` | `multi-qa-MiniLM-L6-cos-v1` | Local embedding model |

---

## Project Structure

```
ai-customer-support-agent/
├── customer_support_agent.py   # Main application
├── requirements.txt            # Core dependencies
├── CLAUDE.md                   # Project notes and change log
├── .gitignore
└── qdrant_storage/             # Qdrant persistent data (git-ignored)
```

---

## Known Limitations

- **Session state dependency** — The customer profile is held in Streamlit session state. Refreshing the page requires re-generating synthetic data.
- **Qdrant persistence** — Stopping Docker without the volume mount will lose stored memories.
- **Groq rate limits** — Free tier has per-minute token limits. For high-volume usage, upgrade to a paid plan.

---

## License

MIT License — see [LICENSE](LICENSE) for details.
