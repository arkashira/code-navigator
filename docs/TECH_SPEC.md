# TECH_SPEC.md – code‑navigator

---

## 1. Overview

**code‑navigator** is an Axentx product that empowers large language models (LLMs) to understand, search, and navigate codebases that exceed the native context window of the model. It bridges the gap between *learning‑based* LLMs and *context‑window‑limited* inference engines by providing a scalable, learning‑augmented retrieval layer that feeds relevant code snippets, metadata, and structural information to the LLM at inference time.

Key benefits:

| Benefit | Description |
|---------|-------------|
| **Scalable context** | Handles millions of lines of code without truncation. |
| **Learning‑augmented** | Uses embeddings to surface the most relevant code for a given query. |
| **Fast inference** | Leverages vLLM for low‑latency, multi‑GPU inference. |
| **Extensible** | Plug‑in architecture for new parsers, embedding models, and LLM back‑ends. |
| **Enterprise‑ready** | Docker/K8s, PostgreSQL + pgvector, observability, RBAC. |

---

## 2. Architecture

```
┌─────────────────────┐
│  Client (Web/CLI)   │
└─────────┬───────────┘
          │ REST/GraphQL
          ▼
┌─────────────────────┐
│  API Gateway (FastAPI) │
└───────┬──────────────┘
        │
        ▼
┌─────────────────────┐
│  Ingestion Service  │
│  (Code Parser +    │
│   Metadata Extractor)│
└───────┬──────────────┘
        │
        ▼
┌─────────────────────┐
│  Embedding Service  │
│  (SGLang / OpenAI)  │
└───────┬──────────────┘
        │
        ▼
┌─────────────────────┐
│  Vector Store (PostgreSQL + pgvector) │
└───────┬──────────────┘
        │
        ▼
┌─────────────────────┐
│  Retrieval Service  │
│  (Nearest‑Neighbor + Ranking) │
└───────┬──────────────┘
        │
        ▼
┌─────────────────────┐
│  LLM Inference Engine (vLLM) │
└───────┬──────────────┘
        │
        ▼
┌─────────────────────┐
│  Response Formatter │
└───────┬──────────────┘
        │
        ▼
┌─────────────────────┐
│  Client (Web/CLI)   │
└─────────────────────┘
```

### 2.1 Component Responsibilities

| Component | Responsibility |
|-----------|----------------|
| **API Gateway** | Auth, rate‑limit, request routing, schema validation. |
| **Ingestion Service** | Clone repo, parse files, extract AST, compute file‑level metadata (size, language, dependencies). |
| **Embedding Service** | Generate embeddings per file, per function, per class using SGLang or OpenAI embeddings. |
| **Vector Store** | Persist embeddings with metadata; support filtering by language, path, tags. |
| **Retrieval Service** | Perform k‑NN search, apply semantic ranking, return top‑k snippets. |
| **LLM Inference Engine** | Run prompt + retrieved snippets through vLLM; support multi‑GPU scaling. |
| **Response Formatter** | Build final answer, include code fences, citations, provenance. |
| **Client** | UI/CLI for developers to query; visual navigation of results. |

---

## 3. Data Model

All data resides in a single PostgreSQL database with the `pgvector` extension.

| Table | Columns | Description |
|-------|---------|-------------|
| **repositories** | `id`, `name`, `url`, `branch`, `last_ingested_at` | Repository metadata. |
| **files** | `id`, `repo_id`, `path`, `language`, `size`, `created_at` | File metadata. |
| **snippets** | `id`, `file_id`, `start_line`, `end_line`, `content`, `embedding vector` | Code snippet (function/class/region). |
| **tags** | `id`, `name` | Tag taxonomy (e.g., “authentication”, “database”). |
| **file_tags** | `file_id`, `tag_id` | Many‑to‑many relation. |
| **ingestion_log** | `id`, `repo_id`, `status`, `started_at`, `completed_at`, `error` | Audit trail. |

### Embedding Vector

- 768‑dim float32 (OpenAI `text-embedding-ada-002` or SGLang `sglang-embedding-1.0`).
- Stored in `vector` column type (`vector(768)`).

---

## 4. Key APIs / Interfaces

| Endpoint | Method | Description | Request | Response |
|----------|--------|
