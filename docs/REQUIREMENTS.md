# REQUIREMENTS.md

## Project: code‑navigator  
**Owner:** Axentx Product Team  
**Repo:** `code-navigator`  
**Purpose:** A tool that enables AI to effectively understand and navigate large codebases, addressing context window scaling and learning‑based approach limitations.

---

## 1. Functional Requirements

| ID | Requirement | Description | Acceptance Criteria |
|----|-------------|-------------|---------------------|
| **FR‑1** | Codebase ingestion | Import a code repository (Git, SVN, local file tree) and build an internal representation. | • Supports Git URLs (HTTPS, SSH) and local paths.<br>• Detects repository root, branches, tags, and submodules.<br>• Generates a full file‑level index within 5 min for a 10 k‑file repo. |
| **FR‑2** | Language detection | Identify programming language for each file. | • Supports at least 15 mainstream languages (Python, Java, C++, JavaScript, Go, Rust, TypeScript, C#, PHP, Ruby, Swift, Kotlin, Scala, Haskell, Lua).<br>• Accuracy ≥ 95 % on a curated test set. |
| **FR‑3** | Structural indexing | Build a graph of modules, classes, functions, and call relationships. | • Graph stored in a graph database (Neo4j or equivalent).<br>• Query “who calls function X?” returns ≤ 10 ms. |
| **FR‑4** | Context window scaling | Provide a retrieval‑augmented generation (RAG) pipeline that selects relevant code snippets within a 4 k token window. | • Retrieval latency ≤ 200 ms.<br>• Supports chunking, summarization, and relevance scoring. |
| **FR‑5** | API surface | Expose a REST/GraphQL API for external LLMs to query the navigator. | • Endpoints: `/search`, `/navigate`, `/summarize`, `/metadata`.<br>• Supports pagination, filtering, and authentication. |
| **FR‑6** | LLM integration | Plug into the company’s inference engine (vLLM) to provide context to LLM calls. | • API accepts a list of code snippets and returns a single prompt string ≤ 4 k tokens. |
| **FR‑7** | User interface | Provide a lightweight web UI for developers to explore the code graph and run queries. | • UI built with React/Vue.<br>• Supports search, graph visualization, and snippet preview. |
| **FR‑8** | Incremental updates | Detect changes in the repo and update the index without full rebuild. | • Uses git hooks or polling; updates < 30 s for a 1 k file diff. |
| **FR‑9** | Security & privacy | Ensure code data is not exposed to external services unless explicitly allowed. | • All data stored locally or in a private cluster.<br>• No outbound network traffic except to the LLM inference engine. |
| **FR‑10** | Auditing & logging | Record all ingestion, query, and update events for compliance. | • Logs include timestamp, user, action, and outcome.<br>• Retention policy configurable (default 90 days). |

---

## 2. Non‑Functional Requirements

| ID | Requirement | Description | Target |
|----|-------------|-------------|--------|
| **NFR‑1** | Performance | Low latency for user queries. | < 250 ms average for `/search` on a 10 k file repo. |
| **NFR‑2** | Scalability | Handle codebases up to 1 M files. | Indexing < 2 h, query latency < 500 ms. |
| **NFR‑3** | Reliability | 99.9 % uptime for API and UI. | < 4 min downtime per month. |
| **NFR‑4** | Security | Comply with ISO 27001 and GDPR. | • Encryption at rest (AES‑256).<br>• Role‑based access control. |
| **NFR‑5** | Maintainability | Codebase follows Axentx coding standards. | • 80 % unit test coverage.<br>• CI/CD pipeline with automated linting. |
| **NFR‑6** | Extensibility | Easy addition of new languages and indexing strategies. | • Plug‑in architecture for language parsers. |
| **NFR‑7** | Documentation | Full API docs, developer guide, and user manual. | • Auto‑generated OpenAPI spec.<br>• Markdown docs in `/docs`. |

---

## 3. Constraints

1. **Infrastructure** – Must run on the existing Axentx Kubernetes cluster; no external cloud services unless approved.  
2. **Licensing** – All dependencies must be open‑source with permissive licenses (MIT, Apache‑2.0, BSD).  
3. **Data Residency** – Code data must remain within the EU data center.  
4. **Resource Limits** – Container memory ≤ 4 GB, CPU ≤ 2 vCPU for indexing jobs.  
5. **Compliance** – No user code may be sent to third‑party LLM APIs; only the inference engine (vLLM) is allowed.

---

## 4. Assumptions

- The target codebases are version‑controlled and accessible via Git.  
- Users have the necessary permissions to read the repositories.  
- The LLM inference engine (vLLM) is already deployed and reachable at `vllm.company.internal`.  
- The graph database chosen (Neo4j or equivalent) is available in
