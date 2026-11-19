# 🚀 CareerPilot — Agentic AI Career Assistant

> **An AI-powered career guidance platform built on multi-agent orchestration, Corrective RAG (CRAG), GraphRAG, and persistent semantic memory to deliver evidence-grounded career advice and live job discovery.**

This project addresses one of the core limitations of generic AI career tools: **generic, ungrounded career advice that ignores a candidate's real background and current market data**. Instead of relying on static prompts, CareerPilot transforms a candidate's resume into both a semantic vector store and a structured knowledge graph, routing requests across specialized AI agents with automated self-evaluation.

---

## 🎯 Motivation

Navigating career transitions, identifying skill gaps, and tailoring job searches require deep context about both the **candidate's actual experience** and **real-time industry demand**. Standard LLM chatbots often hallucinate candidate qualifications, give vague recommendations, or fail to retain user context across sessions.

CareerPilot solves this by combining **Multi-Agent Orchestration (LangGraph)**, **Corrective Retrieval-Augmented Generation (CRAG)**, and **GraphRAG (Neo4j)** with **long-term semantic memory**—ensuring every response is grounded in verified resume evidence, enriched by structured career relationships, and refined through automated quality evaluation.

---

## ✨ Core Features

### 🧠 Multi-Agent Intent Routing & Orchestration

- Uses a **LangGraph routing agent** to classify user intent (`job_search`, `resume_analysis`, `skill_gap`, `job_recommendation`, `general`) and delegate tasks to specialized workflows.
- Coordinates parallel execution, tool invocation, and real-time **Server-Sent Events (SSE)** response streaming.

### 📄 Intelligent Resume Ingestion & Dual Indexing

- Parses uploaded PDF resumes using **PyMuPDF** and splits content into semantic chunks.
- Builds dual representations: dense vector indexes in **FAISS** for semantic retrieval and structured entities/relationships in **Neo4j** for graph-based reasoning.

### 🔍 Corrective RAG (CRAG) Pipeline

- Evaluates retrieved FAISS resume chunks for relevance (`Relevant`, `Ambiguous`, or `Insufficient`) before answer generation.
- Automatically triggers query rewriting and context refinement when evidence is weak, minimizing hallucinations.

### 🕸️ Candidate Knowledge Graph (GraphRAG)

- Extracts structured career entities (`Skill`, `Role`, `Company`, `Institution`, `Experience`, `Project`, `Research`, `Publication`, `Certification`) and relationships (`HAS_SKILL`, `WORKED_AT`, `STUDIED_AT`, `WORKED_ON`, etc.) into **Neo4j**.
- Queries structured graph facts alongside vector retrieval for holistic candidate profiling.

### 🌐 Real-Time Job Search & External Research

- Integrates **DuckDuckGo Search** within the job-search agent to discover active, role-matched opportunities.
- Connects to a remote **Exa MCP (Model Context Protocol)** server for deep external career and industry research.

### 🧬 Short-Term State & Long-Term Semantic Memory

- Persists conversation checkpoints via **LangGraph `AsyncPostgresSaver`** and cross-session semantic memories via **`AsyncPostgresStore` + `pgvector`** in **PostgreSQL**.
- Remembers candidate goals, preferences, and past interactions across separate conversations.

### ⚡ Self-Evaluating Response Refinement

- Runs an **LLM-based evaluation workflow** on generated answers to verify grounding, completeness, and relevance, triggering automated refinement when needed.
- Full execution tracing and observability powered by **LangSmith**.

---

## 🏗️ System Architecture

```text
                     ┌───────────────────────┐
                     │   Web UI & PDF Resume │
                     └──────────┬────────────┘
                                │
                    FastAPI REST & SSE Streaming
                                │
                                ▼
               Memory Retrieval & Conversation State
                     (PostgreSQL + pgvector)
                                │
                                ▼
                   LangGraph Intent Routing Agent
            ┌───────────────────┼───────────────────┐
            │                   │                   │
            ▼                   ▼                   ▼
   Candidate-Career Agent   Job-Search Agent   General Career Agent
     ┌──────┴──────┐            │                   │
     ▼             ▼            ▼                   ▼
Corrective RAG  GraphRAG   DuckDuckGo Search   Exa MCP Server
   (FAISS)      (Neo4j)
     └──────┬──────┘            │                   │
            └───────────────────┼───────────────────┘
                                │
                                ▼
              Response Evaluation & Refinement Loop
                                │
                                ▼
               Streamed Response + LangSmith Traces
```

---

## 💡 Why This Approach?

### 🔹 Corrective Retrieval over Blind Generation

Instead of trusting raw vector search results, the **CRAG pipeline** grades retrieved resume chunks first and rewrites queries when context is ambiguous—ensuring responses stay strictly faithful to the candidate's actual resume.

### 🔹 Hybrid Semantic + Graph Reasoning

Vector search excels at unstructured descriptions, while **Neo4j GraphRAG** captures exact structural links (such as which skills were applied on which projects or roles). Combining both yields far deeper resume and skill-gap analysis.

### 🔹 Agentic Specialization with Memory

Separating candidate analysis, live job search, and general career research into dedicated **LangGraph** workflows prevents prompt bloat, while **pgvector long-term memory** personalizes advice over time.

### 🔹 Built-In Quality Assurance & Observability

Every response passes through an **automated evaluation gate** before streaming to the user, with end-to-end step tracing in **LangSmith**.

---

## 🛠️ Technology Stack

| Layer | Technologies |
| :--- | :--- |
| **Frontend** | HTML5, CSS3, Vanilla JavaScript (SSE Streaming) |
| **Backend & API** | Python 3.11, FastAPI, Uvicorn, Pydantic |
| **Agent Orchestration** | LangGraph, LangChain |
| **LLMs** | OpenAI (`gpt-4o-mini`, `gpt-4o`) |
| **Vector Search & CRAG** | FAISS, Custom LangGraph Evaluation Workflow |
| **Knowledge Graph (GraphRAG)** | Neo4j AuraDB |
| **External Tools & MCP** | DuckDuckGo Search, Exa MCP Server |
| **Database & Memory** | PostgreSQL, SQLAlchemy Async ORM, Alembic, `pgvector` |
| **Document Processing** | PyMuPDF |
| **Authentication** | JWT, Argon2 (`pwdlib`), Server-Side Refresh Sessions |
| **Observability** | LangSmith |

---

## 🔬 End-to-End AI Pipeline

1. **Resume Ingestion:** User uploads a PDF resume; **PyMuPDF** extracts text, which is chunked and indexed into **FAISS** while structured entities and relationships are extracted into **Neo4j**.
2. **Request & Memory Load:** User submits a query in a resume-linked conversation; **FastAPI** loads short-term checkpoints and retrieves relevant long-term memories from **PostgreSQL (`pgvector`)**.
3. **Intent Routing:** The **LangGraph routing agent** classifies the request (`resume_analysis`, `skill_gap`, `job_recommendation`, `job_search`, or `general`).
4. **Parallel Context & Tool Execution:**
   - *Candidate queries* trigger **FAISS CRAG** (with relevance grading and query rewriting) alongside **Neo4j GraphRAG**.
   - *Job queries* invoke **DuckDuckGo Search** for live openings.
   - *General queries* leverage the **Exa MCP** server for web research.
5. **Evaluation & Streaming:** The **response evaluator** checks answer quality, triggers refinement if necessary, persists conversation state, and streams the final response to the UI.

---

## 📈 Key Capabilities

- ✅ Multi-agent intent routing with **LangGraph**
- ✅ Resume-grounded analysis via **Corrective RAG (FAISS)**
- ✅ Structured candidate reasoning via **GraphRAG (Neo4j)**
- ✅ Live job discovery via **DuckDuckGo Search**
- ✅ External career research via **Exa MCP**
- ✅ Short-term and cross-session semantic memory with **PostgreSQL & pgvector**
- ✅ Automated LLM response evaluation and refinement
- ✅ JWT + refresh-token authentication and **LangSmith** observability

---

## 📂 Repository Structure

```text
.
├── frontend/                  # Web interface (HTML, CSS, vanilla JS)
│   ├── index.html
│   ├── css/style.css
│   └── js/{api,auth,conversations,chat,app}.js
├── src/
│   ├── components/
│   │   ├── agents/            # Routing, candidate, job-search, and general agents
│   │   ├── auth/              # JWT and refresh session handling
│   │   ├── conversations/     # Conversation management
│   │   ├── crag/              # Corrective RAG evaluation & refinement
│   │   ├── database/          # Async SQLAlchemy models & sessions
│   │   ├── evaluation/        # Response quality evaluation workflow
│   │   ├── graph/             # Main LangGraph definition
│   │   ├── graphrag/          # Neo4j extraction & graph querying
│   │   ├── llm/               # LLM client configuration
│   │   ├── mcp/               # Exa MCP client integration
│   │   ├── memory/            # Checkpointer & pgvector long-term memory
│   │   ├── messages/          # Message persistence
│   │   ├── resume/            # PyMuPDF parsing & indexing pipeline
│   │   └── retrieval/         # FAISS vector retrieval
│   ├── constants/             # App settings & environment config
│   ├── routes/                # FastAPI endpoints (auth, chat, resumes, conversations)
│   └── utils/                 # Shared helpers
├── alembic/                   # Database migrations
├── data/faiss/                # Local FAISS vector indexes
├── main.py                    # FastAPI application entrypoint
├── pyproject.toml
└── requirements.txt
```

---

## 🌟 Future Enhancements

- Interactive skill-gap roadmap generator with milestone tracking.
- Automated resume tailoring and ATS keyword optimization per target job description.
- Mock interview agent grounded in the candidate's Neo4j project and experience graph.
- Multi-resume comparison and version tracking across target career tracks.

---

## ⚙️ Getting Started

### 1. Clone the Repository

```bash
git clone <your-repository-url>
cd <your-repository-folder>
```

### 2. Create a Virtual Environment & Install Dependencies

```bash
python -m venv .venv
source .venv/bin/activate   # On Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 3. Configure Environment Variables

Create a `.env` file in the project root (see `src/constants/settings.py` for all available options):

```env
# PostgreSQL
DATABASE_URL=

# OpenAI
OPENAI_API_KEY=

# Neo4j
NEO4J_URI=
NEO4J_USERNAME=
NEO4J_PASSWORD=
NEO4J_DATABASE=neo4j

# Exa MCP
EXA_MCP_URL=https://mcp.exa.ai/mcp

# LangSmith
LANGSMITH_TRACING=true
LANGSMITH_API_KEY=
LANGSMITH_PROJECT=careerpilot

# Authentication
JWT_SECRET_KEY=
REFRESH_COOKIE_NAME=careerpilot_refresh_token
REFRESH_COOKIE_SECURE=false
```

### 4. Apply Database Migrations

```bash
python -m alembic upgrade head
```

### 5. Run the Application

```bash
uvicorn main:app --reload
```

Once running, access the web interface at `http://localhost:8000` and the interactive API documentation at `http://localhost:8000/docs`.

---

## 🤝 Contributing

Contributions, feature suggestions, and discussions are welcome. Feel free to fork the repository, open an issue, or submit a pull request.