# CaseBrief AI

CaseBrief AI turns a folder of legal case documents into a searchable index and a formatted demand letter with citations. It was built around personal injury cases: medical records, police reports, wage statements and insurance correspondence.

The point of the project is traceability. Every figure and statement in the generated letter is linked back to the source section it came from, so a reviewer can check the work instead of trusting it.

## What it does

1. Extracts text from case PDFs and splits it into sections.
2. Builds a FAISS vector index over those sections and over whole documents.
3. Lets you search the index from the command line for the most relevant sections.
4. Tracks which sources support each statement.
5. Generates a demand letter as a PDF, using a local Hugging Face model for the written parts.
6. Stores cases, parties, documents and events in PostgreSQL.

No external LLM API key is needed, because text generation runs on a local model.

## Components

| File | Role |
|---|---|
| `rag_assistant.py` | PDF text extraction and FAISS indexing of case sections and documents |
| `command_library.py` | Command-line search over the index |
| `citation_tracker.py` | Records and attributes the sources behind a report |
| `llm_integration.py` | Local Hugging Face model used for text generation |
| `mcp_server.py` | PostgreSQL access layer for cases, parties and documents |
| `generate_report.py` | Assembles the facts, citations and text into a demand letter PDF |
| `database_setup_pg.sql` | Schema: `cases`, `parties`, `documents`, `case_events` |

## Setup

Requirements: Python 3.9 or newer and a PostgreSQL database.

```bash
git clone https://github.com/Naureen39/CaseBrief-AI.git
cd CaseBrief-AI

python -m venv venv
source venv/bin/activate         # Windows: venv\Scripts\activate
pip install -r requirements.txt

cp .env.example .env             # fill in your database details
psql -d your_db -f database_setup_pg.sql
```

The first run downloads the Hugging Face models, so it needs an internet connection and some disk space.

## Adding case documents

Create a folder for each case under `sample_docs/<case-id>/` and put its PDFs inside. That folder is not included in the repository, because case files contain personal and medical details. Use your own documents or fully fictional ones.

## Usage

Search a case:

```bash
python command_library.py "total medical expenses" --case-id 2024-PI-001 --top-k 5
```

Generate a demand letter:

```bash
python generate_report.py
```

The case id used for report generation is set at the bottom of `generate_report.py`. Change it to target a different case.

## Tests

```bash
pytest tests/
```

`tests/test_parsers.py` checks the helper functions that read amounts and dates out of case sections and does not need a database. `tests/test_cli.py` builds a section index and therefore needs PostgreSQL and a case folder.

## Limitations

- Built around personal injury documents. Other case types would need new section rules and parsers.
- Figures are read from document text with pattern matching, so unusual formatting can be missed. A person should review every letter before it is used.
- Needs PostgreSQL and a fairly heavy Python environment (PyTorch and Transformers).
- The case id for report generation is edited in the source rather than passed on the command line.
