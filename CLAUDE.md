# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project context
This repo is a Python codelab for building a small unstructured-data pipeline that harmonizes two sources:
- PDF-like OCR JSON (`raw_data/group_a_pdfs/*.json`)
- Video metadata/transcript JSON (`raw_data/group_b_videos/*.json`)

The goal is to transform both into one unified schema, apply semantic quality gates, and write `processed_knowledge_base.json`.

## Common commands
No dedicated build system or lint configuration is present in the repository.

### Setup
```bash
pip install -r requirements.txt
```

### Run the pipeline
```bash
python starter_code/orchestrator.py
```

### Run tests
```bash
pytest tests/test_lab.py
```

### Run one grading criterion
```bash
pytest tests/test_lab.py -k "test_execution"
pytest tests/test_lab.py -k "test_observability"
pytest tests/test_lab.py -k "test_harmonization"
pytest tests/test_lab.py -k "test_final_output"
```

### Run a single test
```bash
pytest tests/test_lab.py::test_execution_pdf_parsing
```

## Architecture overview
The lab is intentionally split by role, but runtime flow is linear:

1. **Schema contract** (`starter_code/schema.py`)
   - `UnifiedDocument` (Pydantic model) is the canonical contract.
   - Downstream tests expect 6 string fields: `document_id`, `source_type`, `author`, `category`, `content`, `timestamp`.

2. **Source-specific normalization** (`starter_code/process_unstructured.py`)
   - `process_pdf_data(raw_json)` maps PDF keys (`docId`, `authorName`, `docCategory`, `extractedText`, `createdAt`) to the unified contract.
   - `process_video_data(raw_json)` maps video keys (`video_id`, `creator_name`, `category`, `transcript`, `published_timestamp`) to the same contract.
   - PDF content requires noise cleanup (header/footer tokens like `HEADER_PAGE_X` / `FOOTER_PAGE_X`).

3. **Semantic quality gate** (`starter_code/quality_check.py`)
   - `run_semantic_checks(doc_dict)` is the gatekeeper before persistence.
   - It rejects documents with too-short content and toxic/error phrases (e.g., `Null pointer exception`, `OCR Error`, `Traceback`).

4. **Orchestration and persistence** (`starter_code/orchestrator.py`)
   - Loads all raw JSON files from both source groups.
   - Runs source normalizers + semantic checks.
   - Keeps only passing records and writes a JSON list to `processed_knowledge_base.json` at repo root.

## Test/grading alignment
`tests/test_lab.py` and `.github/workflows/classroom.yml` define the grading model used by GitHub Classroom autograding:
- Execution/parsing correctness
- Observability gate behavior
- Harmonization against `UnifiedDocument`
- Final output file existence and structure

When changing pipeline logic, verify both direct function behavior and end-to-end file output via `python starter_code/orchestrator.py` + `pytest tests/test_lab.py`.
