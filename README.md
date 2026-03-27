# Audit Report Builder

A system designed to transform audit from a document-heavy control activity into a structured, data-driven analytical process.

## Why this project

Traditional audit relies on fragmented files, manual review and low reproducibility.

This project approaches audit as a system:
- findings become structured data
- workflows become traceable processes
- analysis becomes reproducible rather than manual
- AI is integrated as a controlled, reviewable layer — not as a black box

## What it does

- Structures audit metadata, findings and references into a canonical data model
- Organizes audit work into workflow-based workspaces instead of static documents
- Integrates local AI models for assisted analysis, drafting and document understanding
- Preserves traceability through review steps, sidecar logs and explicit save logic

## Current status

Work in progress. Core workflows are already functional and actively used as a testbed for iterative development.

---

## Overview

Audit Report Builder is a local-first Flask application for building audit workpapers and narrative outputs from a combination of:
- manually curated audit metadata and findings
- imported previous-audit DOCX reports
- locally attached evidence and references
- optional offline AI assistance based on local GGUF models

The project is actively under development. Core workflows already work end-to-end, but some areas are still being refined.

## Current Status In Detail

This repository is usable, but it should still be treated as work in progress.

### Already Working

- previous-audit DOCX import
- audit metadata extraction and reviewable metadata suggestions
- findings workspace with manual editing
- agent-assisted finding autofill
- finding-level normative refresh
- deterministic local normative citation retrieval with reviewable citation suggestions
- references and evidence management
- narrative chapter drafting and final preview/export flow
- document translation utilities with optional offline OCR
- a local AI companion with routing across general, finance, and vision models

### Intentionally Conservative By Design

- AI suggestions do not silently overwrite canonical workbook data
- most AI and normative actions apply in memory first and still require manual save
- bibliography reconciliation and longer-range consolidation flows are still evolving

## Main Workspaces

The application currently revolves around these user-facing areas.

### 1. Import Previous Audit

- Upload a `.docx` report
- Parse previous audit findings, notes, references, and structural context
- Extract draft audit metadata
- Review metadata suggestions before accepting them

### 2. Audit Metadata

Maintain audit engagement details such as:
- audited unit
- audit title
- date
- period
- scope
- standard/framework

### 3. Items

Review and edit findings one by one.

Main content areas:
- identity & classification
- observation
- supporting basis
- action/remediation

Optional support actions:
- `AI Autofill`
- `Suggest normative citation`
- `Refresh normative sources for this finding`

### 4. Evidence & References

- Manage references library entries
- Review linked documents
- Add evidence entries and citation-related support data

### 5. Narrative Chapters

- Draft and refine chapter-level narrative output

### 6. Final Preview

- Review generated report output before export

## AI And Agent Features

AI assistance is optional and local-first.

Current AI-assisted flows include:
- previous-audit metadata suggestion overlay
- finding autofill and review suggestions
- normative citation ranking over deterministic local citation candidates
- finding-level normative source refresh review
- local companion routing across:
  - general model
  - finance model
  - vision model

### Design Principles

- AI suggestions are reviewable
- canonical workbook data is not silently mutated
- most flows apply changes in memory first and require explicit save

## Architecture Overview

High-level structure:

```text
templates/
- dashboard.html              Main app shell and workspace markup

static/
- app.js                      Global app orchestration
- findings.js                 Findings workspace logic
- previousAudit.js            Previous-audit import flow
- references.js               Evidence & references workspace logic
- chapters.js / summary.js    Narrative and preview logic

src/
- server.py                   Flask server and API routes
- audit_context.py            Previous-audit context and metadata extraction
- previous_audit_docx_parser.py
- ingest.py                   Workbook load helpers
- schemas.py                  Canonical sheet/field definitions
- translator_pipeline.py      Document text extraction / OCR / translation utilities
- services/                   Normative refresh, citation log, refresh sessions, etc.

agent_layer/
- api.py                      Agent-facing entrypoints
- finding_autofill.py
- metadata_autofill.py
- normative_context.py
- normative_citation_suggester.py
- companion_dispatch.py
- router.py / orchestrator.py

models/
- llm_engine.py               Shared local model loader
- *.gguf                      Local text / finance / vision models
```

## Canonical Data Model

The workbook schema is defined in `src/schemas.py`.

Main sheet groups:
- `Audit` metadata
- `Findings`
- `Follow-up`
- `References`

### Audit Metadata Fields

- `AUDITED_UNIT`
- `AUDIT_ID`
- `AUDIT_TITLE`
- `DATE`
- `PERIOD`
- `SCOPE`
- `STANDARD`

### Example Finding Fields

- `FINDING_ID`
- `FINDING_TYPE`
- `TITLE`
- `CONDITION`
- `CRITERIA`
- `CRITERIA_SOURCE`
- `EVIDENCE_NOTES`
- `RATING`
- `STATUS`
- `AREA`
- `OWNER`
- `RISK_IMPACT`
- `RECOMMENDATION`
- `MANAGEMENT_RESPONSE`
- `ACTION_PLAN`

## Local Data And Persistence

The app writes local working data under `data/`.

Important files and folders:
- `data/audit_findings.xlsx`
  - main workbook state
- `data/uploads/`
  - uploaded previous-audit files
- `data/normative_cache/`
  - cached normative fetch results and downloaded source files
- `data/finding_refresh_sessions.json`
  - reviewable sidecar for finding-level refresh results
- `data/app_state.json`
  - local app state

Operational notes:
- the app currently supports one active audit at a time
- imported previous-audit data is used as context, not as an automatic overwrite of canonical workbook data
- several AI review actions log sidecar state before manual save

## Requirements

### Python Dependencies

Install from `requirements.txt`:

- `flask`
- `pandas`
- `openpyxl`
- `python-docx`
- `pypdf`
- `pymupdf`
- `pytesseract`
- `Pillow`
- `pyyaml`
- `llama-cpp-python`
- `pytest`

Recommended setup:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Optional Frontend Dependency

`package.json` currently contains `tailwindcss` as a dev dependency for frontend work. It is not needed for everyday app usage if the existing static assets are already in place.

### Optional OCR For Scanned PDFs

The translation/document extraction pipeline can use offline OCR through Tesseract.

Requirements:
- local Tesseract installation
- language data for:
  - `chi_sim`
  - `eng`

Optional environment variables:
- `TRANSLATOR_OCR_LANGS` (default: `chi_sim+eng`)
- `TRANSLATOR_OCR_DPI` (default: `200`)

## Local Models

The project expects local GGUF models under `models/`.

Current expected model set:
- primary text model:
  - `Qwen2.5-7B-Instruct.Q4_K_M.gguf`
- finance model:
  - `Llama-3-8B-Instruct-Finance-RAG_Q4_K_M.gguf`
- vision model:
  - `Qwen2.5-VL-3B-Instruct-Q4_K_M.gguf`
- vision projector:
  - `mmproj-Qwen2.5-VL-3B-Instruct-Q8_0.gguf`

Shared model loading is handled by `models/llm_engine.py`.

Base config lives in `config.yaml`.

Current default config:

```yaml
ai:
  enabled: true
  model_path: models/Qwen2.5-7B-Instruct.Q4_K_M.gguf
  n_ctx: 4096
  temperature: 0.1
  max_tokens: 1500
  vision_model_path: models/Qwen2.5-VL-3B-Instruct-Q4_K_M.gguf
```

## Running The App

### macOS Helper Script

You can launch the app with:

```bash
./start_app.command
```

What it does:
- moves into the project directory
- activates `.venv` if present
- waits for the local server health endpoint
- opens the browser automatically when the server is ready

### Manual Run

```bash
source .venv/bin/activate
python3 src/server.py
```

The server is exposed locally at:
- [http://127.0.0.1:5000](http://127.0.0.1:5000)

Useful health/status endpoint:
- [http://127.0.0.1:5000/api/llm/status](http://127.0.0.1:5000/api/llm/status)

## Normative Refresh And Citation Flow

The current normative workflow is intentionally staged and reviewable:

1. Work on a specific finding
2. Run `Refresh normative sources for this finding`
3. Review the refresh outcome
4. Run `Suggest normative citation`
5. Review the citation candidate
6. Apply suggested updates in memory only
7. Save explicitly

Important constraints of the current implementation:
- finding-level refresh is the primary user-facing workflow
- global refresh still exists as a secondary helper
- deterministic local citation retrieval happens before any agent ranking
- references are not silently created or overwritten

## Previous Audit Import Notes

Previous-audit import currently supports DOCX-based extraction and review of:
- items/findings
- references
- structural sections
- engagement metadata candidates

Metadata handling is review-based:
- deterministic extraction remains the baseline
- an agent review layer can suggest better values for selected metadata fields
- suggestions must be explicitly accepted before they affect canonical metadata

## Testing

The repository already contains both Python and UI-focused tests.

### Example Python Tests

- `tests/test_previous_audit_hybrid_parser.py`
- `tests/test_agent_finding_autofill.py`
- `tests/test_normative_refresh.py`
- `tests/test_normative_citation_locator.py`
- `tests/test_normative_citation_suggester.py`

### Example UI Tests

- `tests/test_findings_citation_ui.mjs`
- `tests/test_metadata_suggestions_ui.mjs`
- `tests/test_agent_suggestion_ui.mjs`

Typical commands:

```bash
env PYTHONPATH=src pytest -q
```

```bash
node --check static/app.js
node --check static/findings.js
node --check static/previousAudit.js
```

```bash
node --test --experimental-default-type=module tests/test_findings_citation_ui.mjs
```

## Known Limitations

- The project is still evolving and some workspaces are being actively refined.
- Local model inference can be resource-intensive.
- On some macOS / Metal / `llama-cpp-python` setups, model-related instability can still occur.
- Some workflows rely on local files and caches that are intentionally not portable across machines without setup.
- Normative refresh, citation review, and bibliography consolidation are implemented conservatively and remain review-heavy by design.

## Privacy And Deployment Assumptions

This repository is designed around a local desktop workflow:
- local workbook
- local uploads
- local cached normative documents
- local GGUF models

It is not structured as a multi-user hosted SaaS application.

## Before Publishing To GitHub

Before pushing this repository publicly, review what should and should not be committed.

At minimum, check:
- local model files in `models/`
- uploaded audit documents in `data/uploads/`
- cached normative documents in `data/normative_cache/`
- generated workbook and state files in `data/`

If you intend to publish the codebase, it is strongly recommended to maintain a proper `.gitignore` for:
- model binaries
- uploads
- caches
- generated workbook outputs
- local state files

## License

No explicit project license has been defined yet in the repository. Add one before publishing publicly if needed.
