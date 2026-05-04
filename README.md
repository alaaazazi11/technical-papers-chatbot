#Chatbot Using Agentic RAG
<!-- Badges -->
[![Python 3.12+](https://img.shields.io/badge/Python-3.12%2B-blue)](https://www.python.org/downloads/)
[![LangChain](https://img.shields.io/badge/LangChain-Powered-00A67E)](https://python.langchain.com/)
[![LangGraph](https://img.shields.io/badge/LangGraph-Agentic-FF6B35)](https://langgraph.com/)
[![Qdrant](https://img.shields.io/badge/Qdrant-Vector%20DB-1F77E4)](https://qdrant.tech/)
[![Gradio](https://img.shields.io/badge/Gradio-UI-FF2B2B)](https://gradio.app/)

---

## Overview

**Agentic RAG** is a sophisticated Retrieval-Augmented Generation (RAG) system that combines multi-agent orchestration with hybrid vector search to deliver intelligent, context-aware document retrieval and question-answering capabilities.

The system addresses the challenge of maintaining accuracy and context when answering questions about large document repositories. Unlike traditional RAG systems, Agentic RAG uses LangGraph-based agent workflows to:
- Analyze and rewrite user queries for better retrieval
- Intelligently route queries through specialized processing nodes
- Retrieve contextual document chunks with nuanced relevance ranking
- Aggregate and synthesize responses from multiple retrieval strategies

**Ideal for**: Knowledge management systems, enterprise document Q&A, customer support automation, research synthesis, and domain-specific AI assistants.

---

## Key Features

- 📄 **Multi-Format Document Support** — Upload and process PDF and Markdown files with automatic duplicate detection
- 🔍 **Hybrid Search** — Combine dense embeddings (HuggingFace) with sparse BM25 vectors for superior retrieval accuracy
- 🤖 **Agentic Workflows** — Multi-node LangGraph pipeline for query analysis, rewriting, and intelligent routing
- 🧠 **Hierarchical Chunking** — Parent-child chunk architecture for balanced context and retrieval precision
- 💾 **Persistent Vector Database** — Qdrant-backed storage for scalable similarity search
- 🎨 **Web-Based UI** — Clean Gradio interface for document management and interactive chat
- 🔄 **Conversation Context** — Maintains thread-based chat history with stateful agent execution
- ⚡ **Local LLM Support** — Ollama integration for on-device model execution with configurable parameters
- 🛠️ **Tool-Augmented Agents** — Built-in tools for semantic search and hierarchical chunk retrieval

---

## Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        User Interface (Gradio)                  │
│                   [Document Upload] [Chat Interface]            │
└────────────────────────┬────────────────────────────────────────┘
                         │
         ┌───────────────┴───────────────┐
         │                               │
    [Document Manager]            [Chat Interface]
         │                               │
    [PDF → Markdown]          [Query Processing]
         │                               │
         ├──────────────────────────────┤
         │                              │
    [DocumentChunker]          [RAG System]
    ├─ Markdown Header Split       ├─ Thread Management
    ├─ Parent Chunk Creation       ├─ Agent Graph Execution
    └─ Child Chunk Creation        └─ Tool Invocation
         │                              │
         ├──────────────────────────────┤
         │                              │
    [Parent Store]             [Vector DB (Qdrant)]
    (Local JSONs)              ├─ Dense Embeddings
                               ├─ Sparse Embeddings
                               └─ Hybrid Search
```

### Query Processing Pipeline

```
User Query
    │
    ├─→ [Summarize] Chat history summarization
    │
    ├─→ [Analyze & Rewrite] Query analysis and reformulation
    │        ├─→ Is query clear? → [YES] → Continue
    │        └─→ Is query unclear? → [NO] → [Human Input] (interrupt)
    │
    ├─→ [Agent Subgraph] Tool-augmented agent loop
    │        ├─ Search child chunks (hybrid retrieval)
    │        ├─ Retrieve parent chunks (context expansion)
    │        └─ Generate responses (LLM reasoning)
    │
    ├─→ [Aggregate] Response synthesis
    │
    └─→ Final Answer to User
```

### Component Interactions

1. **Document Pipeline**
   - User uploads PDF/Markdown → DocumentManager converts PDF to Markdown
   - DocumentChunker hierarchically splits content (parent + child chunks)
   - Child chunks are embedded (hybrid: dense + BM25 sparse)
   - Chunks stored in Qdrant; metadata stored locally

2. **Query Pipeline**
   - User submits query → Conversation context is summarized
   - Query is analyzed and potentially rewritten for clarity
   - Agent loop uses tools to search and retrieve relevant chunks
   - LLM synthesizes final response from retrieved context
   - Response returned with interrupt support for user confirmation

3. **Retrieval Strategy**
   - Hybrid search combines semantic + lexical matching
   - Child chunks enable fine-grained relevance ranking
   - Parent chunks provide extended context around results
   - Configurable score thresholds ensure quality results

---

## Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **LLM Framework** | [LangChain](https://python.langchain.com/) | Core abstractions for LLMs, embeddings, and text processing |
| **Agentic Orchestration** | [LangGraph](https://langgraph.com/) | Multi-node agent workflows with state management & checkpointing |
| **Vector Database** | [Qdrant](https://qdrant.tech/) | Hybrid search with dense + sparse embeddings |
| **Dense Embeddings** | [Sentence-Transformers](https://www.sbert.net/) (all-mpnet-base-v2) | Semantic understanding for dense vector search |
| **Sparse Embeddings** | [FastEmbed](https://github.com/qdrant/fastembed) (BM25) | Lexical matching via sparse vectors |
| **Local LLM** | [Ollama](https://ollama.ai/) (Qwen 2.5 3B) | On-device model inference |
| **PDF Processing** | [PyMuPDF4LLM](https://pymupdf4llm.readthedocs.io/) + [Docling](https://github.com/IBM/docling) | High-quality PDF-to-Markdown conversion |
| **Text Splitting** | [LangChain Text Splitters](https://python.langchain.com/docs/modules/data_connection/document_loaders/markdown_header_splitter) | Structured document chunking via Markdown headers |
| **Web UI** | [Gradio](https://gradio.app/) | Interactive interface for document & chat management |
| **Runtime** | Python 3.12+ | Modern async-friendly Python runtime |

---

## Project Structure

```
AgenticRAG/
├── 📄 main.py                          # Entry point: launches Gradio UI
├── 📄 config.py                        # Central configuration & model parameters
├── 📄 pyproject.toml                   # Project metadata & dependencies
├── 📄 document_chunker.py              # Hierarchical document splitting logic
├── 📄 util.py                          # PDF-to-Markdown conversion utilities
│
├── 📁 core/                            # Core RAG system components
│   ├── rag_system.py                   # RAGSystem orchestrator: init, threading, config
│   ├── chat_interface.py               # ChatInterface: query handling & message routing
│   ├── document_manager.py             # DocumentManager: file handling, ingestion pipeline
│   └── (internal coordination)
│
├── 📁 db/                              # Database layer
│   ├── vector_db_manager.py            # VectorDbManager: Qdrant client, collection ops
│   └── parent_store_manager.py         # ParentStoreManager: local JSON persistence
│
├── 📁 rag_agent/                       # Agentic pipeline (LangGraph)
│   ├── graph.py                        # Agent graph compilation & execution
│   ├── nodes.py                        # Graph nodes: summarize, analyze, retrieve, aggregate
│   ├── graph_state.py                  # State schemas (State, AgentState)
│   ├── edges.py                        # Conditional routing logic between nodes
│   ├── tools.py                        # ToolFactory: search & retrieve function definitions
│   ├── schemas.py                      # Pydantic schemas for structured outputs
│   └── prompts.py                      # System prompts for analysis & synthesis
│
├── 📁 ui/                              # User interface layer
│   ├── gradio_app.py                   # Gradio UI components & handlers
│   └── css.py                          # Custom CSS styling
│
├── 📁 markdowns/                       # Ingested Markdown documents (auto-created)
├── 📁 parent_store/                    # Parent chunk JSON storage (auto-created)
├── 📁 qdrant_db/                       # Qdrant vector database (auto-created)
│   └── collection/
│       └── document_child_chunks/      # Collection storing child chunks
│
└── 📄 README.md                        # This file
```

### Key File Descriptions

- **config.py** — Centralized configuration for models, paths, embedding dimensions, and hyperparameters. Edit here to customize behavior.
- **document_chunker.py** — Implements intelligent chunking: markdown header splitting, size-based merging/splitting, and parent-child association.
- **rag_agent/graph.py** — Compiles the LangGraph state machine: orchestrates analysis, querying, and aggregation.
- **rag_agent/nodes.py** — Graph node implementations: summarization, query rewriting, agent reasoning, response aggregation.
- **rag_agent/tools.py** — Defines agent tools: `search_child_chunks()` and `retrieve_parent_chunks()`.
- **db/vector_db_manager.py** — Manages Qdrant collections: hybrid search setup, density/sparsity configuration.

---

## Getting Started

### Prerequisites

- **Python** 3.12 or higher
- **Ollama** installed and running (for LLM inference)
  - Download from [ollama.ai](https://ollama.ai/)
  - Pull required model: `ollama pull qwen2.5:3b`
- **Git** (optional, for cloning)

### Installation

1. **Clone the repository** (or download the project folder)
   ```bash
   git clone https://github.com/yourusername/AgenticRAG.git
   cd AgenticRAG
   ```

2. **Create and activate a Python virtual environment**
   ```bash
   python -m venv .venv
   ```
   - **On Windows:**
     ```bash
     .venv\Scripts\activate
     ```
   - **On macOS/Linux:**
     ```bash
     source .venv/bin/activate
     ```

3. **Install dependencies**
   ```bash
   pip install -e .
   ```
   This installs all packages listed in `pyproject.toml`.

4. **Verify Ollama is running**
   ```bash
   curl http://localhost:11434
   ```
   If Ollama is not running, start it:
   ```bash
   ollama serve
   ```

5. **Run the application**
   ```bash
   python main.py
   ```
   The Gradio UI will launch at `http://localhost:7860` (or similar).

### Environment Configuration (.env Example)

While the project uses `config.py` for settings, you may want to create a `.env` file for sensitive information or local overrides:

```env
# Model Configuration
LLM_MODEL=qwen2.5:3b
DENSE_MODEL=sentence-transformers/all-mpnet-base-v2

# Paths
MARKDOWN_DIR=markdowns
QDRANT_DB_PATH=qdrant_db
PARENT_STORE_PATH=parent_store

# Database Collection
CHILD_COLLECTION=document_child_chunks

# Search Parameters
CHILD_CHUNK_SIZE=500
CHILD_CHUNK_OVERLAP=100
MIN_PARENT_SIZE=2000
MAX_PARENT_SIZE=10000
```

To use environment variables, modify `config.py` to read from `.env`:
```python
import os
from dotenv import load_dotenv

load_dotenv()
MARKDOWN_DIR = os.getenv("MARKDOWN_DIR", "markdowns")
LLM_MODEL = os.getenv("LLM_MODEL", "qwen2.5:3b")
# ... etc
```

---

## Usage

### Web Interface (Recommended)

1. **Start the application:**
   ```bash
   python main.py
   ```

2. **Open your browser** and navigate to `http://localhost:7860`

3. **Document Management Tab:**
   - Click "Drop PDF or Markdown files here" or use the file browser
   - Upload one or more files (PDF or `.md`)
   - Files are automatically converted and indexed
   - View active documents in the "Current Documents" section

4. **Chat Tab:**
   - Type your question in the message field
   - The system will:
     - Summarize conversation history
     - Analyze and rewrite your query if needed
     - Search the knowledge base
     - Return a synthesized answer
   - Use "Clear Session" to start fresh (resets thread state)

### Example Workflow

```
📤 Upload Documents
   └─→ Upload: research_paper.pdf
   └─→ Upload: documentation.md
   
💬 Ask a Question
   └─→ "What are the key findings in the research paper?"
   └─→ System searches → retrieves context → generates answer
   
🔄 Follow-up
   └─→ "How does that relate to the documentation?"
   └─→ System uses conversation context → refines search → answers
   
🗑️ Manage
   └─→ Click "Remove All" to clear knowledge base
   └─→ Click "Clear Session" to reset chat thread
```

### Command-Line Usage (Advanced)

For programmatic access, you can directly use the `RAGSystem` class:

```python
from core.rag_system import RAGSystem
from core.chat_interface import ChatInterface

# Initialize system
rag_system = RAGSystem()
rag_system.initialize()

# Add documents (programmatically)
from core.document_manager import DocumentManager
doc_manager = DocumentManager(rag_system)
doc_manager.add_documents(["path/to/document.pdf"])

# Query the system
chat = ChatInterface(rag_system)
response = chat.chat("Your question here", history=[])
print(response)

# Reset conversation thread
chat.clear_session()
```

---

## API Reference

### Core Classes

#### **RAGSystem**
Main orchestrator for the RAG pipeline.

```python
from core.rag_system import RAGSystem

rag_system = RAGSystem(collection_name="document_child_chunks")

# Initialize vector DB and agent graph
rag_system.initialize()

# Get configuration for agent execution
config = rag_system.get_config()
# → {"configurable": {"thread_id": "uuid-string"}}

# Reset conversation thread (start fresh)
rag_system.reset_thread()
```

#### **ChatInterface**
Handles user queries and agent invocation.

```python
from core.chat_interface import ChatInterface

chat = ChatInterface(rag_system)

# Process a user message
response = chat.chat(
    message="Your question here",
    history=[]  # Previous conversation turns (optional)
)
# → "AI-generated answer based on retrieved context"

# Clear the conversation thread
chat.clear_session()
```

#### **DocumentManager**
Handles document ingestion and indexing.

```python
from core.document_manager import DocumentManager

doc_manager = DocumentManager(rag_system)

# Add documents (returns count of added/skipped)
added, skipped = doc_manager.add_documents(
    document_paths=["file1.pdf", "file2.md"],
    progress_callback=None  # Optional: (progress, description) → None
)
# → (2, 0)  # 2 added, 0 skipped

# Retrieve list of indexed documents
files = doc_manager.get_markdown_files()
# → ["document1.md", "document2.md"]

# Clear all documents and indices
doc_manager.clear_all()
```

#### **VectorDbManager**
Manages Qdrant vector store operations.

```python
from db.vector_db_manager import VectorDbManager

db_manager = VectorDbManager()

# Create a new collection (or verify existing)
db_manager.create_collection("document_child_chunks")

# Delete a collection
db_manager.delete_collection("document_child_chunks")

# Get a collection for similarity search
collection = db_manager.get_collection("document_child_chunks")

# Perform hybrid similarity search
results = collection.similarity_search(query="search term", k=5)
```

### Agent Tools

The agent has access to two tools automatically:

#### **search_child_chunks**
Performs hybrid similarity search across indexed child chunks.

```python
# Called automatically by agent
# Parameters:
#   query: str - The search query
#   k: int - Number of results (default: 5)

# Returns: List of dicts with keys:
#   - "content": The chunk text
#   - "parent_id": Associated parent chunk ID
#   - "source": Document source filename
```

#### **retrieve_parent_chunks**
Retrieves full parent chunks by ID for additional context.

```python
# Called automatically by agent
# Parameters:
#   parent_ids: List[str] - IDs of parent chunks to retrieve

# Returns: List of parent chunk dictionaries
```

### Configuration Reference

All settings are defined in `config.py`:

| Setting | Default | Purpose |
|---------|---------|---------|
| `MARKDOWN_DIR` | `"markdowns"` | Directory for processed Markdown files |
| `PARENT_STORE_PATH` | `"parent_store"` | Directory for parent chunk JSON storage |
| `QDRANT_DB_PATH` | `"qdrant_db"` | Path to Qdrant database directory |
| `CHILD_COLLECTION` | `"document_child_chunks"` | Collection name for child chunks |
| `DENSE_MODEL` | `"sentence-transformers/all-mpnet-base-v2"` | Dense embedding model |
| `SPARSE_MODEL` | `"Qdrant/bm25"` | Sparse (BM25) embedding model |
| `LLM_MODEL` | `"qwen2.5:3b"` | Ollama model for inference |
| `LLM_TEMPERATURE` | `0` | LLM temperature (0 = deterministic) |
| `CHILD_CHUNK_SIZE` | `500` | Size of child chunks in tokens |
| `CHILD_CHUNK_OVERLAP` | `100` | Overlap between adjacent chunks |
| `MIN_PARENT_SIZE` | `2000` | Minimum parent chunk size |
| `MAX_PARENT_SIZE` | `10000` | Maximum parent chunk size |
| `HEADERS_TO_SPLIT_ON` | `[("#", "H1"), ("##", "H2"), ("###", "H3")]` | Markdown headers for chunking |

---

## Contributing

We welcome contributions! Please follow these guidelines:

### Code Standards

- **Style**: Follow [PEP 8](https://pep8.org/) conventions
- **Type Hints**: Use type annotations for all function parameters and returns
- **Docstrings**: Include docstrings for all functions and classes (Google/NumPy style)
- **Testing**: Write tests for new features in a `tests/` directory
- **Imports**: Sort imports alphabetically, group by type (stdlib, third-party, local)

### Workflow

1. **Fork the repository** on GitHub
2. **Create a feature branch:**
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. **Make your changes** and commit with clear messages:
   ```bash
   git commit -m "Add feature: brief description"
   ```
4. **Push to your fork:**
   ```bash
   git push origin feature/your-feature-name
   ```
5. **Open a Pull Request** with:
   - Clear description of changes
   - Reference to related issues
   - Test results showing functionality

### Areas for Contribution

- **Performance**: Optimize vector search or chunk processing
- **Features**: New embedding models, LLM providers, or retrieval strategies
- **Documentation**: Improve guides, examples, or API documentation
- **Testing**: Expand test coverage
- **UI/UX**: Enhance Gradio interface or add new features
- **Bug Fixes**: Fix reported issues with detailed explanations

---

## License

This project is released under the **MIT License**. See the LICENSE file for details.

You are free to:
- Use this project for commercial and personal purposes
- Modify and distribute the code
- Include it in proprietary applications

Under the condition that you:
- Include the license notice and copyright statement

---

## Troubleshooting

### Common Issues

**❌ "Ollama connection refused"**
- Ensure Ollama is running: `ollama serve`
- Check Ollama is listening on `localhost:11434`
- Pull the model if missing: `ollama pull qwen2.5:3b`

**❌ "ModuleNotFoundError: No module named 'langchain'"**
- Ensure virtual environment is activated
- Run: `pip install -e .` from project root

**❌ "Gradio interface not loading"**
- Check port 7860 is not in use (or Gradio will use 7861, 7862, etc.)
- Try: `python main.py` from project directory
- Check terminal output for error messages

**❌ "No documents found after upload"**
- Verify files are PDF or `.md` format
- Check `markdowns/` directory was created
- Ensure file permissions allow reading

**❌ "Search returns no results"**
- Verify documents have been indexed: check "Current Documents" in UI
- Lower the score threshold in `config.py` (adjust search logic in `tools.py`)
- Ensure your query contains relevant terms

---

## Support & Contact

For questions, issues, or suggestions:
- Open an [Issue](https://github.com/yourusername/AgenticRAG/issues) on GitHub
- Submit a [Discussion](https://github.com/yourusername/AgenticRAG/discussions)
- Check [Existing Issues](https://github.com/yourusername/AgenticRAG/issues?q=) first

---

## Acknowledgments

This project leverages amazing open-source technologies:
- [LangChain](https://langchain.com/) & [LangGraph](https://langgraph.com/) for AI orchestration
- [Qdrant](https://qdrant.tech/) for vector search infrastructure
- [Ollama](https://ollama.ai/) for local LLM support
- [Gradio](https://gradio.app/) for the web interface
- [HuggingFace](https://huggingface.co/) for embedding models

---

**Last Updated:** May 2026  
**Status:** Active Development  
**Python:** 3.12+
