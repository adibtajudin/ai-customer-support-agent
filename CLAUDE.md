# AI Customer Support Agent - Project Tracker (claude.md)

## 📌 Project Overview
We are building an AI-powered customer support agent built using Python and Streamlit. The agent leverages Groq's `llama-3.3-70b-versatile` model for generating intelligent responses and the Mem0 library backed by Qdrant (a vector database) to maintain a persistent and personalized memory of past customer interactions. Embeddings are handled locally via HuggingFace `multi-qa-MiniLM-L6-cos-v1`.

## 🚀 Features
- **Chat Interface**: A Streamlit-based web interface for users to interact with the AI assistant.
- **Persistent Memory**: Uses Mem0 and Qdrant to store and retrieve past customer interactions, profiles, and order histories, providing context-aware support.
- **Synthetic Data Generation**: Capable of generating realistic customer profiles, order histories, and past interactions for testing and demonstration using `llama-3.3-70b-versatile`.
- **Contextual Responses**: The AI assistant uses retrieved memory to answer customer queries accurately based on their specific profile and history.

## 📁 Codebase Structure

### `customer_support_agent.py`
This is the main application file. It contains:
1. **`CustomerSupportAIAgent` Class**:
   - Initializes the Mem0 memory layer with Qdrant as the vector store provider (`localhost:6333`).
   - Initializes the Groq client (via OpenAI-compatible SDK with Groq base URL).
   - `handle_query(query, user_id)`: Searches memory for the query, builds a context prompt, gets a response from `llama-3.3-70b-versatile`, and stores both the query and the response in memory.
   - `get_memories(user_id)`: Retrieves all memories for a specific user.
   - `generate_synthetic_data(user_id)`: Prompts `llama-3.3-70b-versatile` to generate a detailed JSON customer profile and recent orders, and stores these details into the Mem0 memory.
2. **Streamlit UI**:
   - Sidebar for entering the Groq API Key.
   - Sidebar for entering the Customer ID.
   - Buttons to "Generate Synthetic Data", "View Customer Profile", and "View Memory Info".
   - Main chat interface using `st.chat_message` and `st.chat_input` to maintain a conversational history during the session.

### `requirements.txt`
Dependencies required for the project:
- `streamlit`
- `openai`
- `mem0ai`
- `qdrant-client`
- `sentence-transformers` (local HuggingFace embeddings)
- `groq`

## 🛠️ Infrastructure & Prerequisites
1. **Python Environment**: Dependencies installed via `pip install -r requirements.txt`
2. **Qdrant Vector DB**: Running locally via Docker on port 6333.
   ```bash
   docker run -p 6333:6333 -p 6334:6334 \
       -v $(pwd)/qdrant_storage:/qdrant/storage:z \
       qdrant/qdrant
   ```
3. **Groq API Key**: Required to run the application (input via the Streamlit interface). Get one free at `console.groq.com`.

## 🏃‍♂️ How to Run the Application

### Step 1: Start Qdrant (Docker)
If the container already exists:
```bash
docker start qdrant
```
If running for the first time:
```bash
docker run -d --name qdrant -p 6333:6333 -p 6334:6334 \
    -v "C:/Users/adibt/AI Customer Support Agent/qdrant_storage:/qdrant/storage:z" \
    qdrant/qdrant
```

### Step 2: Start the Streamlit App
```bash
cd "C:\Users\adibt\AI Customer Support Agent"
python -m streamlit run customer_support_agent.py --server.headless true
```

### Step 3: Open in Browser
Navigate to `http://localhost:8501` (or `8502`, `8503` etc. if 8501 is already in use — Streamlit will show the exact URL in the terminal).

To force a specific port:
```bash
python -m streamlit run customer_support_agent.py --server.headless true --server.port 8501
```

### Step 4: Use the App
1. Enter your **Groq API key** in the text box
2. Enter a **Customer ID** in the sidebar (e.g. `cust_001`)
3. Click **Generate Synthetic Data** to create a fake customer profile
4. Start chatting in the chat box

## 📈 Current Status & Plan
- **Status**: App is fully running with Groq as the LLM provider. Qdrant is running via Docker. Streamlit app is live at `localhost:8501`.
- **Next Steps / Plan**:
  - Test the memory layer and the generation of synthetic customer data.
  - Extend the AI logic or UI as needed based on upcoming requirements.

## 🔄 Changes from Original Tutorial
| Component | Original | Current |
|---|---|---|
| LLM | OpenAI `gpt-4o-mini` | Groq `llama-3.3-70b-versatile` |
| Embeddings | OpenAI (mem0 default) | HuggingFace `multi-qa-MiniLM-L6-cos-v1` (local) |
| API Key | `OPENAI_API_KEY` | `GROQ_API_KEY` |
| Client init | `OpenAI()` | `OpenAI(api_key=..., base_url="https://api.groq.com/openai/v1")` |

**Bug fix**: Removed `model` field from `vector_store` config — caused a `ValidationError` in `mem0ai`. Only `host` and `port` are valid fields in that block.
