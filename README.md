# AmitGPT

AmitGPT is an open-source agentic AI chatbot built with Python, FastAPI, LangGraph, LangChain, Google Gemini, Tavily, ChromaDB, and SQLite. It supports real-time streaming chat, document uploads, retrieval-augmented generation (RAG), web search, conversation memory, and a simple web UI.

---

## Features

- **Real-time Streaming Chat** — Responses are streamed token-by-token directly to the browser using Server-Sent Events (SSE), giving a smooth ChatGPT-like experience.
- **Agentic AI with LangGraph** — The chatbot uses a LangGraph state machine to decide when to answer directly and when to invoke tools, making it truly agentic.
- **Document Upload & RAG** — Upload PDF, DOCX, TXT, MD, PY, or CSV files. The chatbot chunks and embeds them into ChromaDB and can answer questions based on their contents.
- **Web Search via Tavily** — For latest news, current events, recent releases, or any time-sensitive query, the agent automatically searches the web using Tavily and summarizes results.
- **Long-Term Memory** — The agent can save and recall user-specific facts and preferences across conversations using a persistent SQLite-backed memory store.
- **Calculator Tool** — Handles math calculations directly within the chat using a safe eval-based calculator tool.
- **Multiple Gemini Models** — Switch between `gemini-2.5-flash`, `gemini-2.5-pro`, `gemini-2.5-flash-lite`, `gemini-1.5-flash`, and `gemini-1.5-pro` from the chat UI.
- **Conversation History** — All chats are persisted to SQLite. Previous conversations are listed in the sidebar and can be resumed at any time.
- **Voice Dictation** — Built-in microphone support lets you dictate messages using the Web Speech API (Chrome/Edge).
- **Tool Progress Indicators** — The UI detects likely tool usage (web search, calculator, document search, memory) and shows animated progress indicators while tools run.
- **Clean Dark UI** — Minimal, responsive dark-themed web interface with a sidebar, model selector, file upload button, and message history — no external frontend framework required.
- **Per-Thread Document Isolation** — Uploaded documents are scoped to a specific conversation thread, so each chat only retrieves from its own uploaded files.
- **LangGraph Checkpointing** — Agent state is checkpointed to SQLite so conversations survive server restarts.

---

## Project Overview

This project combines:

| Component | Role |
|---|---|
| **FastAPI** | Backend server and REST + streaming API endpoints |
| **Jinja2** | Renders the single-page frontend (`index.html`) |
| **LangGraph** | Agentic graph orchestration — chatbot node and tool node with conditional routing |
| **LangChain** | Tool definitions, message types, RAG pipeline, and LLM wrappers |
| **Google Gemini** | LLM provider for chat completions and text embeddings |
| **Tavily** | Real-time web search tool |
| **ChromaDB** | Vector store for document embeddings and semantic retrieval |
| **SQLite** | Persistent storage for conversations, chat history, long-term memory, and LangGraph checkpoints |
| **SQLAlchemy** | ORM layer for the SQLite database |

---

## Prerequisites

Before running AmitGPT, make sure you have the following:

- **Python 3.10+**
- **pip** (Python package manager)
- A **Google Gemini API key** — get one at [Google AI Studio](https://aistudio.google.com/app/apikey)
- A **Tavily API key** — get one at [Tavily](https://tavily.com)
- (Optional) **Git** for cloning the repository

### Environment Variables

Create a `.env` file in the project root with the following keys:

```env
GOOGLE_API_KEY=your_google_gemini_api_key_here
TAVILY_API_KEY=your_tavily_api_key_here
```

---

## Project Structure

```
AmitGPT/
├── app.py                  # FastAPI application — routes, SSE streaming, file upload
├── agent.py                # LangGraph agent — graph definition, model selection, checkpointing
├── tools.py                # LangChain tools — calculator, web search, RAG search, memory
├── rag.py                  # RAG pipeline — file reading, chunking, embedding, ChromaDB retrieval
├── database.py             # SQLAlchemy models and DB helpers — conversations, messages, memory
├── requirements.txt        # Python dependencies
├── .env                    # API keys (not committed to version control)
├── .gitignore
├── templates/
│   └── index.html          # Single-page chat UI (vanilla HTML/CSS/JS)
├── uploads/                # Uploaded documents are stored here (auto-created)
├── chroma_db/              # ChromaDB vector store (auto-created)
└── data/
    ├── chatbot_memory.db           # SQLite DB for conversations and long-term memory
    └── langgraph_checkpoints.sqlite # SQLite DB for LangGraph agent state checkpoints
```

---

## How to Run AmitGPT

### 1. Clone the repository

```bash
git clone https://github.com/Amit4141/AmitGPT.git
```

### 2. Navigate to the project directory

```bash
cd AmitGPT
```

### 3. Create a virtual environment (optional but recommended)

```bash
python -m venv venv
```

### 4. Activate the environment

**Windows (PowerShell):**
```powershell
.\venv\Scripts\Activate.ps1
```

**macOS / Linux:**
```bash
source venv/bin/activate
```

### 5. Install the required dependencies

```bash
pip install -r requirements.txt
```

### 6. Add your API keys

Create a `.env` file in the project root:

```env
GOOGLE_API_KEY=your_google_gemini_api_key_here
TAVILY_API_KEY=your_tavily_api_key_here
```

### 7. Run the application

```bash
python app.py
```

The server starts at `http://localhost:8080`. Open that URL in your browser to use the chat UI.

---

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/` | Serves the chat web UI |
| `POST` | `/chat/stream` | Streams AI responses via SSE |
| `POST` | `/upload` | Uploads a document and adds it to the vector store |
| `GET` | `/conversations` | Lists all saved conversation threads |
| `GET` | `/history/{thread_id}` | Returns full message history for a conversation |

---

## Supported File Types for Upload

| Extension | Format |
|---|---|
| `.pdf` | PDF documents |
| `.docx` | Microsoft Word documents |
| `.txt` | Plain text files |
| `.md` | Markdown files |
| `.py` | Python source files |
| `.csv` | Comma-separated values |

---

## Supported Gemini Models

| Model | Description |
|---|---|
| `gemini-2.5-flash` | Default — fast and capable (recommended) |
| `gemini-2.5-pro` | More powerful, higher quality responses |
| `gemini-2.5-flash-lite` | Lightweight and fastest |
| `gemini-1.5-flash` | Previous generation, fallback |
| `gemini-1.5-pro` | Previous generation pro, fallback |

---

## How the Agent Works

1. User sends a message from the browser.
2. FastAPI receives the request and passes it to the LangGraph agent.
3. The agent's **chatbot node** invokes Gemini with the system prompt and conversation history.
4. If a tool is needed, the graph routes to the **tools node** which executes the appropriate tool (web search, RAG, calculator, or memory).
5. The tool result is fed back into the chatbot node for a final response.
6. The response is streamed back to the browser token-by-token via SSE.
7. The final message is saved to SQLite for persistent history.

---

## Deployment (Free — Render.com)

Render is the best free option for this project. It supports FastAPI, Python 3.11, and gives you a real Linux container with a public HTTPS URL.

### Steps

**1. Push your code to GitHub**

Make sure your repo is on GitHub. The `.gitignore` already excludes `.env`, `venv/`, `uploads/`, `chroma_db/`, and `data/` so no secrets or local state gets committed.

**2. Sign up at [render.com](https://render.com)**

Use your GitHub account to sign in.

**3. Create a new Web Service**

- Click **New → Web Service**
- Connect your GitHub repo (`AmitGPT`)
- Render auto-detects `render.yaml` — it will pre-fill all settings

**4. Set environment variables**

In the Render dashboard under **Environment**, add:

| Key | Value |
|---|---|
| `GOOGLE_API_KEY` | your Google Gemini API key |
| `TAVILY_API_KEY` | your Tavily API key |

**5. Deploy**

Click **Deploy Web Service**. Render will install dependencies and start the server. Your app will be live at a URL like `https://amitgpt.onrender.com`.

### Important Notes

- **Free tier spins down** after 15 minutes of inactivity. The first request after idle takes ~30 seconds to wake up.
- **Storage is ephemeral** on the free tier — uploaded files, ChromaDB vectors, and SQLite data are wiped on each redeploy. This is fine for demos. For persistence, add a Render Disk ($1/month) mounted at `/data`.
- **Build time** is ~3–5 minutes on first deploy due to heavy dependencies (ChromaDB, LangChain, etc.).

---

## License

This project is licensed under the terms of the [LICENSE](LICENSE) file included in this repository.
