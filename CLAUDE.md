# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A personal study repository for the **Google Cloud Professional Data Engineer Renewal** certification exam. It contains only markdown files — no code, build system, or tests. The source of truth for exam topics is `gcp_data_engineer_exam_guide.pdf` in the repo root (3-page PDF).

## File Naming Conventions

- Section folders: `NN-kebab-case-section-name/` (zero-padded, e.g., `01-`, `02-`)
- Subsection files: `N.N-kebab-case-topic.md` (e.g., `1.1-security-and-compliance.md`)
- Subsections **not** included in the renewal exam: **3.2** and **5.1** — do not create files for these

## Subsection File Template

Every subsection `.md` must follow this exact section order:

1. `# N.N Topic Title` — H1 title
2. `**Section:** N — Section Name (~X% of exam)` — weight context line
3. `## Official Topics (from Exam Guide)` — verbatim checklist from the PDF
4. `## Key Concepts` — pre-filled study summaries with a `> _Fill in your own notes here._` prompt
5. `## Google Cloud Services` — markdown table: Service | Role (or similar columns)
6. `## Study Notes` — left blank for the user
7. `## Practice Questions` — 3 scenario-based questions, each with lettered options, bolded answer, and an explanatory blockquote
8. `## Resources` — doc links

## Root-Level Reference Files

- `gcp_data_engineer_exam_guide.pdf` — Source of truth for all exam topics (3 pages)
- `gcp-service-comparisons.md` — Side-by-side tables for storage, processing, orchestration, governance, ML/AI, BigQuery pricing, and streaming patterns
- `architectural-patterns.md` — Named data architecture patterns (ETL/ELT, Lambda, Kappa, Medallion, Data Mesh, Lakehouse) with GCP implementations and tradeoffs
- `renewal-focus.md` — Services and topics new or elevated in the renewal exam vs. the original PDE (BigLake, AlloyDB, Dataform, Analytics Hub, RAG/embeddings, BigQuery Editions, Dataplex Catalog)
- `exam-traps.md` — Common wrong-answer patterns and trigger words for quickly identifying the correct service/approach under exam conditions
- `bigquery-sql-patterns.md` — BigQuery SQL syntax reference: partition overwrite, MERGE, window functions, BQML (`CREATE MODEL`, `ML.PREDICT`, `ML.GENERATE_TEXT`), `VECTOR_SEARCH`, row-level security, time travel, and schema management
- `labs.md` — Google Cloud Skills Boost labs mapped to each subsection with High/Medium priority ratings and recommended learning paths
- `mock-exam.md` — 30 scenario-based questions weighted by section with a scored answer key and per-section breakdown
- `mock-exam-2.md` — 30 harder questions designed to combine two topics per scenario; same structure as mock-exam.md
- `study-log.md` — Personal session log with a progress snapshot table, per-session template (date, topics, gaps, confidence), and pre-populated Section 1 entry

## Progress Tracking

Study completion is tracked via checkboxes in two places:
- `README.md` at the root (master tracker with links to all subsections)
- Each section's `README.md` (section-level tracker)

When a user marks a topic complete, update `- [ ]` to `- [x]` in both locations.

## Exam Section Weights

| Section | Weight |
|---------|--------|
| 1 — Designing data processing systems | ~25% |
| 2 — Ingesting and processing the data | ~10% |
| 3 — Storing the data | ~25% |
| 4 — Preparing and using data for analysis | ~25% |
| 5 — Maintaining and automating data workloads | ~15% |

## Practice Question Style

Questions must be scenario-based (a business or technical situation, not a definition quiz). Each question has:
- A realistic scenario paragraph
- 4 lettered options (A–D)
- `**Answer: X**` on its own line
- A `> blockquote` explaining why the correct answer is right and why the main distractors are wrong
