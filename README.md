# Chatbot LLM RAG (Text-to-SQL) for Open WebUI

A small Open WebUI Function that converts natural-language questions into SQL queries and returns answers sourced directly from a PostgreSQL table. The pipeline uses LlamaIndex for text-to-SQL orchestration, a HuggingFace embedding model for retrieval, and Groq-hosted LLMs for SQL generation and answer validation. It is designed to be used inside Open WebUI as a user-defined Function (pipe).

## Features
- Text-to-SQL: Converts user questions into SQL queries scoped to a single table.
- Vector retrieval: Uses sentence-transformers/all-MiniLM-L6-v2 embeddings to retrieve relevant schema/context.
- SQL execution: Runs generated SQL against PostgreSQL (table: dataset_pembangunan).
- Answer validation: Uses a second model to validate generated answers in a loop (up to 3 attempts).
- Case-insensitive text matching: Uses ILIKE in generated SQL for flexible text matching.
- Result citation: Emits the executed SQL as a citation event to Open WebUI (for transparency).

## System Architecture
- Open WebUI Function (single Python script `Main`) — runtime environment provided by Open WebUI.
- LlamaIndex / ObjectIndex & retriever — builds a small schema-aware index for SQL table retrieval and text-to-SQL prompting.
- Embedding model — HuggingFace sentence-transformers/all-MiniLM-L6-v2 used for similarity retrieval.
- Groq LLMs (via a Groq-compatible OpenAI client) — one model for generation (llama3-70b-8192) and another for validation (gemma2-9b-it).
- PostgreSQL — single table `dataset_pembangunan` is queried for answers.

How it fits together (runtime flow):
1. A user message arrives in Open WebUI and triggers the Function `Magang_Chatbot` (class `Pipe` in `Main`).
2. The Function creates a SQLDatabase (SQLAlchemy engine) and an ObjectIndex wrapping the table schema.
3. A retriever + Groq LLM produce a SQL query (text-to-SQL); the SQL is executed against `dataset_pembangunan`.
4. The SQL result is sent back to the LLM for generation of a human-readable answer; the answer is validated by a second model. If validation fails, the pipeline retries up to 3 times.
5. The pipeline returns the validated answer and emits a citation event containing the SQL text.

## Tech Stack
- Frontend: Open WebUI (function workspace)
- Backend: Python script running inside Open WebUI's Function environment
- Database: PostgreSQL (table: dataset_pembangunan)
- AI / LLM:
  - Groq-hosted models (used via OpenAI-compatible client)
  - llama_index (LlamaIndex) for orchestration and SQL helper classes
  - sentence-transformers/all-MiniLM-L6-v2 for embedding
- Libraries / Tools:
  - SQLAlchemy
  - openai Python SDK (used to call Groq endpoints)
  - httpx, pydantic, asyncio (standard helpers)

## Project Structure
```text
/
├── Main         # Python Function file intended to be pasted into Open WebUI
└── README.md    # Original project README (Indonesian) — replaced by this document
