# STORIES.md

## Overview

`code-navigator` is an Axentx product that empowers AI systems to understand and navigate large codebases efficiently.  
The following backlog is organized into epics that reflect the core capabilities required for an MVP. Each story follows the **As a <role>, I want <goal>, so that <benefit>** format and is accompanied by concrete acceptance criteria.

---

## Epic 1 – Core Code Indexing & Retrieval

| # | Story | Acceptance Criteria |
|---|-------|---------------------|
| 1 | **As a developer, I want the tool to index an entire repository, so that I can quickly locate any file or symbol.** | • Indexer runs in < 5 min for a 10 k‑file repo.<br>• Index contains file paths, function names, class names, and comments.<br>• Index is stored in a searchable SQLite/PG vector store. |
| 2 | **As a developer, I want incremental indexing, so that changes to the repo do not require a full re‑index.** | • Detect file additions, deletions, and modifications via git hooks.<br>• Only affected files are re‑indexed.<br>• Incremental run completes in < 30 s for 100 changed files. |
| 3 | **As a developer, I want to search by symbol name, so that I can jump directly to a function or class definition.** | • Search returns exact matches and fuzzy matches.<br>• Results include file path, line number, and snippet.<br>• UI shows a clickable list of results. |
| 4 | **As a developer, I want to search by keyword, so that I can find all occurrences of a term across the codebase.** | • Keyword search is case‑insensitive.<br>• Results are ranked by relevance (frequency + proximity).<br>• Pagination supports > 1000 results. |

---

## Epic 2 – AI Integration & Context Window Management

| # | Story | Acceptance Criteria |
|---|-------|---------------------|
| 5 | **As an AI engineer, I want the tool to provide context‑aware snippets to an LLM, so that the model can answer code‑specific questions accurately.** | • Context is limited to the LLM’s max token window (e.g., 8k tokens).<br>• Tool selects the most relevant 4–6 snippets per query.<br>• API returns context in a JSON payload with metadata (file, line range). |
| 6 | **As an AI engineer, I want automatic chunking of large files, so that the LLM can process them without exceeding the context window.** | • Files > 200 kB are split into overlapping 1k‑token chunks.<br>• Chunk boundaries preserve function/class boundaries when possible.<br>• Chunk metadata is stored for retrieval. |
| 7 | **As a product owner, I want summarization of large code sections, so that the LLM can get a high‑level overview before diving into details.** | • Summaries are generated via a lightweight summarization model.<br>• Summary length ≤ 200 tokens.<br>• Summaries are cached and invalidated on code change. |

---

## Epic 3 – Developer Experience & UI

| # | Story | Acceptance Criteria |
|---|-------|---------------------|
| 8 | **As a developer, I want a VSCode extension, so that I can search and navigate code directly from my editor.** | • Extension installs via VSIX.<br>• Provides a command palette entry “Navigate Code with AI”.<br>• Displays search results in a side panel with clickable links. |
| 9 | **As a developer, I want a web UI, so that I can browse the index and run queries without an IDE.** | • Single‑page app with search bar, results list, and code viewer.<br>• Supports pagination and filtering by file type.<br>• Authenticated users can save favorite queries. |
| 10 | **As a developer, I want a CLI tool, so that I can integrate code navigation into scripts and CI pipelines.** | • `cnavigate search <query>` returns JSON list of results.<br>• Supports flags for output format (`json`, `table`).<br>• Exit code 0 on success, non‑zero on failure. |

---

## Epic 4 – Performance & Scalability

| # | Story | Acceptance Criteria |
|---|-------|---------------------|
| 11 | **As a system admin, I want caching of query results, so that repeated queries are served instantly.** | • Cache TTL defaults to 10 min.<br>• Cache invalidates on underlying file change.<br>• Cache hit rate ≥ 80 % for identical queries. |
| 12 | **As a system admin, I want the tool to support concurrent queries, so that multiple users can use it simultaneously.** | • Handles ≥ 50 concurrent search requests with < 200 ms latency.<br>• Uses
