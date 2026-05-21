# Job Application Copilot

An AI-assisted backend project that:

1. reads a job description from a URL (or local dummy mode),
2. compares it against a CV,
3. scores compatibility,
4. generates:
   - a tailored short profile (3-5 lines),
   - a structured motivation-focused cover letter.

This project is designed as a practical portfolio example for applied NLP/LLM engineering workflows.

## Features

- URL job description scraping with fallback dummy mode
- Lightweight RAG retrieval (BM25-style lexical chunk retrieval)
- Multi-step agent workflow:
  - `researcher` -> job requirement analysis
  - `planner` -> compatibility + gaps
  - `writer` -> profile + cover letter draft
  - `reviewer` -> quality and factual grounding pass
- Structured JSON output for reproducibility
- Environment-based LLM config (`OLLAMA_HOST`, `LLM_MODEL`, `LLM_TIMEOUT`)

## Architecture

```text
backend/app/
  agents/         # role-specific LLM steps
  db/             # output persistence models/client
  llm/            # LLM API client (Ollama)
  rag/            # retriever + context formatting
  tools/          # scraping, CV loading, cover letter helpers
  workflows/      # orchestration pipeline
  main.py         # CLI entrypoint
```

## Workflow

1. **Input**: user provides `Job URL` (or `dummy`).
2. **Ingestion**:
   - scrape job text from URL,
   - load CV from `backend/app/tools/my_cv.md`.
3. **Retrieval**:
   - chunk job and CV text,
   - retrieve relevant evidence per stage.
4. **Reasoning pipeline**:
   - analyze role requirements,
   - evaluate fit and missing skills,
   - produce tailored profile + cover letter,
   - run reviewer pass for clarity and grounding.
5. **Output**:
   - save structured JSON to `outputs/job_application_<timestamp>.json`.

## Prerequisites

- Python 3.8+
- [Ollama](https://ollama.com/) installed and running
- A pulled model (default: `llama3`, configurable)

## Setup

### 1) Clone and install dependencies

```bash
git clone <your-repo-url>
cd job_application_copilot
pip install -r requirements.txt
```

### 2) Configure environment

Copy `.env.example` to `.env` and adjust values:

```bash
cp .env.example .env
```

Default variables:

- `OLLAMA_HOST=http://localhost:11434`
- `LLM_MODEL=llama3`
- `LLM_TIMEOUT=300`

### 3) Add CV file

Create your private CV file at:

`backend/app/tools/my_cv.md`

This file is ignored by git by default.

## Run

From repository root:

```bash
python -m backend.app.main
```

When prompted:

- paste a real job URL, or
- type `dummy` for local pipeline testing without web scraping.

## Example Output Structure

```json
{
  "job_url": "...",
  "job_analysis": "...",
  "compatibility_analysis": "...",
  "short_profile": "...",
  "cover_letter": "...",
  "final_result": "..."
}
```

## Troubleshooting

### 1) `Connection refused` on `localhost:11434`

Ollama is not reachable from your current environment.

- Ensure `ollama serve` is running.
- Ensure app and Ollama run in the same environment (Windows vs WSL), or set `OLLAMA_HOST` accordingly.

### 2) LLM request timeout

- Increase `LLM_TIMEOUT` in `.env` (e.g. `600`).
- Use a smaller/faster model (e.g. `llama3.2:3b`).
- Use `dummy` mode to isolate scraping from generation latency.

### 3) Scraping timeout / blocked website

- Some websites rate-limit or block automated requests.
- Test with `dummy` first to verify end-to-end pipeline.

## Privacy Notes

- CV files are personal data. Keep `backend/app/tools/my_cv.md` private.
- Never commit private CVs or generated letters containing personal details.

## Roadmap

- Embedding-based retrieval (vector DB)
- FastAPI API endpoint + frontend integration
- Unit tests for retriever and workflow steps
- Better site-specific extraction for major job boards

