# Enterprise IT Support Agentic RAG Copilot

An advanced, agentic Retrieval-Augmented Generation (RAG) system designed for enterprise IT support. This system doesn't just search for documents; it uses an intelligent agentic workflow to route queries, grade evidence, rewrite queries for better retrieval, and fallback to web searches when internal knowledge is insufficient.

## 🚀 Features

- **Agentic Workflow**: Powered by `LangGraph`, the system follows a state-aware decision tree to ensure the highest answer quality.
- **Multi-Source Retrieval**:
    - **Private Knowledge Base**: High-priority retrieval from internal documents stored in **Pinecone**.
    - **Web Search Fallback**: Automatic fallback to the web via **Tavily** if internal documents are deemed "weak" or insufficient.
- **Self-Correction**:
    - **Evidence Grading**: The agent evaluates retrieved chunks; if the grade is "weak", it triggers a query rewrite and retries.
    - **Query Rewriting**: Dynamically optimizes user questions to improve retrieval hit rates.
- **Intelligent Routing**: Distinguishes between casual chat (direct response) and technical IT queries (KB retrieval).
- **Audit Logging**: Every interaction, including the agent's internal trace (the "thought process"), is logged to a SQLite database for quality monitoring.
- **Admin Ingestion**: Secure API endpoint for administrators to upload and index new IT manuals, policies, and guides (PDF, DOCX, TXT, MD).

## 🛠️ Architecture

The system is built with a modular approach:

- **Frontend**: (HTML/JS) Simple interface for chatting with the copilot.
- **Backend**: **FastAPI** providing high-performance asynchronous endpoints.
- **Orchestration**: **LangGraph** managing the agent state and transitions.
- **LLM**: **Google Gemini** for reasoning, routing, grading, and generation.
- **Vector Database**: **Pinecone** for scalable, fast semantic search of company documents.
- **Embeddings**: **HuggingFace (all-MiniLM-L6-v2)** for local, efficient vectorization.

## 🚦 Workflow Logic

1. **Route**: User $\rightarrow$ Router (KB or Direct?)


2. **Retrieve**: (If KB) $\rightarrow$ Pinecone Vector Store $\rightarrow$ Retrieve Chunks.
3. **Grade**: Grader LLM $\rightarrow$ Is evidence "good" or "weak"?
4. **Decide**:
    - If **Good** $\rightarrow$ Generate Answer from KB.
    - If **Weak** $\rightarrow$ Search Web via Tavily.
5. **Web Grade**: Is web evidence sufficient?
    - If **Yes** $\rightarrow$ Generate Answer from Web.
    - If **No** $\rightarrow$ Rewrite Query $\rightarrow$ Go back to Step 2 (up to `max_retries`).
6. **Final**: Return answer with citations and a trace of the agent's path.

## 📦 Installation & Setup

### Prerequisites
- Python 3.10+
- Pinecone Account
- Google AI Studio API Key (Gemini)
- Tavily API Key

### 1. Clone and Environment
```bash
git clone <repo-url>
cd Agentic-RAG
python -m venv rag_venv
# Windows:
.\rag_venv\Scripts\activate
# Linux/Mac:
source rag_venv/bin/activate
```

### 2. Install Dependencies
```bash
pip install -r requirements.txt
```

### 3. Configuration
Create a `.env` file in the root directory:
```env
GOOGLE_API_KEY=your_google_api_key
TAVILY_API_KEY=your_tavily_api_key
PINECONE_API_KEY=your_pinecone_api_key
PINECONE_INDEX_NAME=fde-it-support-rag
ADMIN_API_KEY=your_secure_admin_key
```

### 4. Run the Application
```bash
uvicorn app.main:app --reload --port 8000
```
Access the UI at `http://127.0.0.1:8000`

## 🔐 Admin API

### Document Ingestion
To add company knowledge to the RAG system:
- **Endpoint**: `POST /api/ingest`
- **Header**: `x-admin-key: <your_admin_api_key>`
- **Payload**: Multipart form-data containing the file.
- **Supported Formats**: `.pdf`, `.docx`, `.txt`, `.md`

## 📈 Project Structure
```text
├── app/
│   ├── api/            # FastAPI routes and request models
│   ├── core/            # Config, logging, and global settings
│   ├── rag/             # The "Brain": State, Workflow, and Vectorstore
│   └── services/        # Ingestion and Audit logging logic
├── data/               # SQLite audit DB and sample KB
├── uploads/            # Temporary storage for ingested files
└── main.py             # Application entry point
```
