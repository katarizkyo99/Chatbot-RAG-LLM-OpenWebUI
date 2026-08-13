# Chatbot LLM RAG PostgreSQL (Text-to-SQL) on Open WebUI

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
└── README.md    # Project README
```

## Requirements
- Open WebUI (container or local instance)
- PostgreSQL reachable from the Open WebUI runtime
- Groq API account and API key (used via OpenAI-compatible client)
- Internet access for model API calls and embeddings (if not running locally)
- Python dependencies inside Open WebUI: llama_index, sqlalchemy, openai, sentence-transformers

## Installation (prepare environment)
This Function is intended to run inside Open WebUI. The shortest path:

1. Run Open WebUI (example Docker run):
```bash
docker run -d -p 3030:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui1 --restart always \
  -e DATABASE_URL="postgresql://<DB_USER>:<DB_PASS>@host.docker.internal:5432/<DB_NAME>" \
  ghcr.io/open-webui/open-webui:main
```
2. Ensure PostgreSQL is reachable from the Open WebUI container. The pipeline expects a table named `dataset_pembangunan`.
3. Open Open WebUI in a browser: http://localhost:3030
4. Go to Workspace → Functions → Add new function → paste the contents of `Main` → Save and activate the pipeline.
5. Use the Open WebUI chat to trigger the `Magang_Chatbot` pipeline.

## Configuration
The current `Main` script contains hardcoded database connection strings and API keys. DO NOT commit secrets — replace these with environment variables or secure secret management.

Recommended environment variables (suggested names):
- GROQ_API_KEY — API key for Groq/OpenAI-compatible client
- GROQ_BASE_URL — base URL if different from default
- DATABASE_URL — SQLAlchemy-compatible PostgreSQL URL (e.g. postgresql://user:pass@host:5432/dbname)

Notes:
- The script uses host addresses such as `host.docker.internal` to reach a host Postgres instance from Docker. If your Postgres is in Docker compose, use service names and a shared network.
- The repository contains inconsistent references (examples mention database names `film`, `bangun`, and the script uses `bangun`). Ensure the database name used in configuration matches the table location.

## Running the Project
From Open WebUI (preferred):
- Add the Function as described in Installation.
- Ask a question in Open WebUI chat and select/trigger the `Magang_Chatbot` pipeline.

Development (local / iterating):
- Edit `Main` to read configuration from environment variables (recommended).
- Ensure any Python packages required by the function are available inside Open WebUI's Python environment.

## Output / Answer format
The pipeline uses a specific text format when the LLM produces SQL and when it returns the final answer. The `Main` script defines two formats (excerpted from the prompt templates used by the code):

1) When a query is needed
```
Question: <user question here>
SQLQuery: <SQL query to run — only the SQL statement>
SQLResult: <Result returned by the database>
Answer: <Final answer summarized from SQLResult>
```

2) When no query is needed
```
Question: <user question here>
Answer: <Final answer here>
```

Example with placeholders (do not assume column names):
```
Question: Show projects in city X for year 2022
SQLQuery: SELECT <columns> FROM dataset_pembangunan WHERE <conditions> LIMIT 10
SQLResult: [  {"column1": "value1", "column2": "value2"}, ... ]
Answer: In 2022, the database shows N projects in city X. Example entries include ... (summary derived from SQLResult)
```

Note: The repository does not include the table schema or real query results. The example above shows the format the Function produces; actual SQL and results depend on your database schema and data.

## Database
- Database type: PostgreSQL (no schema migrations are included).
- Table required: dataset_pembangunan — the Main script queries this table. The repository does not include table schema or sample data; you must provide your own table with relevant columns.

## Troubleshooting
- Open WebUI cannot reach Postgres: check network (host.docker.internal vs localhost), ports, firewall, and credentials.
- Table `dataset_pembangunan` not found: ensure the table exists in the configured database and the DB connection points to the correct database name.
- Model authentication/permission errors: verify Groq API key and account access to the specific models.
- Missing Python dependencies: ensure LlamaIndex, SQLAlchemy, openai, and sentence-transformers are installed in the environment Open WebUI uses to run the function.

## Contributing
- Fork the repo, propose changes, and open a PR. This repository does not include contribution guidelines; please follow typical open-source practices (clear PR description, tests if possible).

## Author
- Repository owner: katarizkyo99
