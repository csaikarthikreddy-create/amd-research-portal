# Final Report (Phase 3): Personal Research Portal

**Project:** Personal Research Portal (PRP)  
**Domain:** Trustworthy RAG Evaluation Methods  
**Primary question:** How do different faithfulness metrics fail, and how can we combine them in a practical research workflow?

---

## 1. Introduction and Objectives

This project builds a Personal Research Portal (PRP) on top of a retrieval-augmented generation pipeline for the domain of trustworthy RAG evaluation. Earlier phases established the research framing, prompt/evaluation workflow, and baseline retrieval-generation system. Phase 3 focuses on productizing that pipeline into a usable interface for iterative research tasks.

The final objective is not only to generate answers, but to support a complete evidence-centric workflow:

- Ask a research question and retrieve relevant corpus evidence.
- Produce answers with inspectable citations and source references.
- Save work into persistent research threads.
- Generate reusable artifacts from answers and export them.
- Review evaluation logs and failure patterns in one place.

The portal is implemented as an MVP with emphasis on traceability, reproducibility, and fast local setup. It intentionally favors transparent data flow and low operational overhead over high UI complexity.

---

## 2. System Architecture

### 2.1 Architecture overview

The system is organized into layered modules:

- **UI layer** (`src/app/main.py`)  
  Streamlit interface with four pages: Ask, Threads, Artifacts, Evaluation.

- **RAG execution layer** (`src/rag/`)  
  Retrieval (`retrieve.py`), generation (`generate.py` via `query.py`), prompting (`prompts.py`), structured citations (`structured_citations.py`), logging (`logger.py`), and trust suggestions (`suggestions.py`).

- **Persistence layer** (`src/threads/store.py`)  
  File-based thread storage under `outputs/threads/`.

- **Artifact layer** (`src/artifacts/evidence_table.py`)  
  Parses cited claims from answers and maps them to retrieved evidence chunks; exports Markdown/CSV/HTML.

- **Evaluation layer** (`src/eval/`)  
  Batch evaluation runner (`run_eval.py`), query set (`query_set.csv`), and summarizer (`summarize.py`).

- **Data/index layer**  
  Corpus metadata (`data/data_manifest.csv`), processed chunks (`data/processed/`), and FAISS index (`index/faiss.index`, `index/chunk_map.json`).

### 2.2 Runtime flow: Ask page

When a user submits a question in the Ask page:

1. The query is passed to `run_query()` in `src/rag/query.py`.
2. `retrieve()` loads FAISS + sentence-transformers artifacts and returns top-k chunks.
3. `generate_answer()` produces a response from query + retrieved context.
4. `format_answer_with_references()` appends manifest-resolved references.
5. The UI renders the answer and exposes retrieved chunks in an expandable section.
6. If the answer indicates missing evidence, a trust message suggests related keywords.
7. Optional: the user stores the result as a thread JSON record.

This flow preserves provenance by keeping source IDs and chunk IDs visible and reusable downstream.

### 2.3 Runtime flow: Artifact generation

From a saved thread:

1. The answer body is parsed for inline citations (e.g., `(source_id, chunk_id)`).
2. Claims are segmented and mapped to retrieved chunks.
3. Evidence rows are assembled with schema:
   - `claim`
   - `evidence_snippet`
   - `citation`
   - `confidence`
   - `notes`
4. Rows can be exported in Markdown, CSV, and HTML.

This enables structured handoff from conversational output to analysis-ready artifact files.

### 2.4 Runtime flow: Evaluation summary

The Evaluation page discovers `eval_run_*.jsonl` in `logs/` and summarizes:

- total queries,
- citation presence,
- no-evidence response count,
- error count,
- query-type distribution,
- representative no-evidence/error examples.

The UI does not trigger full evaluation jobs directly; it summarizes existing logs for responsiveness and operational simplicity.

---

## 3. Design Choices and Rationale

### 3.1 Streamlit for rapid, local-first delivery

**Decision:** Build the portal in Streamlit instead of a custom frontend/backend stack.  
**Rationale:** The rubric prioritizes working functionality and local reproducibility. Streamlit reduces implementation time while supporting:

- multi-page navigation,
- immediate rendering of Markdown and tables,
- file download controls,
- low deployment complexity.

**Trade-off:** UI flexibility and scaling controls are limited compared to a dedicated web architecture.

### 3.2 File-based thread persistence

**Decision:** Store threads as JSON files in `outputs/threads/`.  
**Rationale:** Thread records are append-only, inspectable, and easy to version in local workflows. This keeps the MVP database-free and aligns with rubric acceptance of file-based thread storage.

**Trade-off:** No indexing or search at scale; concurrent multi-user edits are not addressed.

### 3.3 Evidence table as the first artifact type

**Decision:** Implement one robust artifact type (evidence table) rather than multiple shallow artifact types.  
**Rationale:** The evidence table directly leverages citation structure already present in generated answers. It supports immediate research utility:

- claim extraction,
- chunk-level verification,
- export to common formats.

**Trade-off:** More advanced artifacts (annotated bibliography or synthesis memo) are deferred.

### 3.4 Structured citations and reference resolution

**Decision:** Post-process generated answers with a reference section from manifest metadata.  
**Rationale:** Inline citations alone are not user-friendly for verification. Manifest-linked references improve readability and auditability by including title/authors/year/URL for cited/retrieved sources.

**Trade-off:** If generation omits inline citation patterns, artifact extraction quality degrades.

### 3.5 Explicit trust behavior on evidence gaps

**Decision:** Detect “no evidence” phrasing and show query-rewrite keyword suggestions.  
**Rationale:** In research settings, silent failures are costly. The UI surfaces uncertainty instead of masking it, and nudges users toward better retrieval prompts.

**Trade-off:** Current suggestions are heuristic (manifest tags), not intent-aware semantic reformulations.

### 3.6 Export strategy: MD/CSV/HTML (browser PDF path)

**Decision:** Export artifact tables as Markdown/CSV/HTML and rely on browser print for PDF.  
**Rationale:** This avoids additional dependencies and keeps local setup lightweight while still meeting practical export needs.

**Trade-off:** PDF export is not one-click inside the app.

---

## 4. Evaluation

### 4.1 Evaluation setup

The evaluation stack uses:

- query set: `src/eval/query_set.csv` (23 queries),
- runner: `src/eval/run_eval.py`,
- logs: `logs/eval_run_*.jsonl`,
- summarizer: `src/eval/summarize.py`.

The query set includes three categories:

- **Direct** (fact lookup / single-concept),
- **Synthesis** (cross-source comparison),
- **Edge** (ambiguity or evidence-boundary behavior).

### 4.2 Primary run results

From `logs/eval_run_20260215_195940.jsonl`:

- Total queries: **23**
- Answers with citation pattern: **23 (100%)**
- “No evidence” responses: **3**
- Errors: **0**
- Query type distribution:
  - Direct: **12**
  - Synthesis: **6**
  - Edge: **5**

Interpretation:

- Citation formatting consistency is strong in this run.
- Most failures are retrieval-bound rather than generation crash failures.
- “No evidence” outputs indicate uncertainty handling is active, but may reflect missed retrieval in some cases.

### 4.3 Latency snapshot from interactive logs

From `logs/rag_runs.jsonl` (10 sampled runs):

- Average latency: **~1271.9 ms**
- Minimum latency: **~770.4 ms**
- Maximum latency: **~2545.04 ms**

This is acceptable for an MVP research assistant workflow where evidence transparency is prioritized over ultra-low latency.

### 4.4 Artifact output evidence

Artifact outputs are present in the repository and directly tied to the implemented artifact pipeline:

- `outputs/sample_evidence_table.md`
- `outputs/sample_evidence_table.csv`
- additional user-generated thread artifacts in `outputs/threads/` (multiple JSON thread records).

The sample evidence table demonstrates end-to-end behavior:

- claim extraction from answer text,
- citation linking,
- confidence/notes schema fields,
- export-ready formatting.

### 4.5 What evaluation shows about system behavior

Strengths:

- Stable end-to-end functionality (no eval-time runtime errors in the cited run).
- Consistent citation patterning in outputs.
- Practical observability through JSONL logs and summary view.
- Artifact generation works on saved thread state without re-querying.

Observed weaknesses:

- Retrieval misses can propagate into “no evidence” outcomes even when relevant documents exist in corpus.
- Citation-driven artifact extraction is sensitive to answer format regularity.
- Quality metrics still rely partly on manual interpretation (e.g., faithfulness nuance).

---

## 5. Limitations

1. **Retrieval-first bottleneck**  
   The quality ceiling is constrained by top-k retrieval; missing key chunks creates downstream answer and artifact gaps.

2. **Citation-format dependency**  
   Evidence extraction expects recognizable inline citation patterns. If formatting drifts, artifact completeness drops.

3. **No native in-app PDF export**  
   PDF is currently produced via HTML download and browser print flow.

4. **Evaluation execution stays in CLI**  
   The portal summarizes eval runs but does not yet orchestrate long-running evaluation jobs in-app.

5. **Thread usability at scale**  
   JSON thread logs are simple and robust but not optimized for search/filter workflows in large research sessions.

6. **Limited automated faithfulness scoring**  
   Current summaries emphasize structural metrics; richer semantic faithfulness checks are future work.

---

## 6. Next Steps

### 6.1 Retrieval quality improvements

- Add reranking (cross-encoder or lightweight reranker) after FAISS retrieval.
- Explore hybrid retrieval (dense + lexical) for better recall on benchmark names and niche terms.

### 6.2 Artifact expansion

- Add annotated bibliography artifact mode keyed by `source_id`.
- Add synthesis memo mode that aggregates cross-paper findings with explicit uncertainty markers.

### 6.3 Evaluation depth

- Add automated citation precision/recall checks against expected-source hints.
- Add NLI/LLM-judge faithfulness scoring with calibrated prompts.
- Add per-query error taxonomy (retrieval miss, wrong citation, overconfident synthesis, etc.).

### 6.4 UX and operational improvements

- Add thread search and compact views.
- Add background job runner for `make eval` equivalent with progress/status.
- Add one-click report export bundling answer + evidence table + references.

### 6.5 Reliability and governance

- Add schema validation for thread JSON and artifact exports.
- Add reproducibility metadata in outputs (model/provider/top-k/timestamp/version).
- Expand tests around citation parsing edge cases.

