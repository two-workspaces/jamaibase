# Deploy and Host JamAI Base on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/jamai-base?referralCode=sLjDXb&utm_medium=integration&utm_source=button&utm_campaign=jamai-base)

[JamAI Base](https://github.com/EmbeddedLLM/JamAIBase) is an open-source backend for AI apps that you build like a spreadsheet. Each table column can be an input, an LLM prompt that references other columns, a Python snippet, or a retrieval step over your documents. Adding a row runs the whole pipeline. Action Tables chain prompts and code, Knowledge Tables embed your files for RAG, and Chat Tables are agents with memory. Everything you click together in the UI is also an API, with Python and TypeScript SDKs. This template runs the full open-source stack, behind a password, with your own model keys.

## About Hosting JamAI Base

The stack is eleven services. Three are JamAI's own code, built from a pinned upstream commit, and the rest are pinned stock images:

- **Gateway** (Caddy) is the only public service. The UI needs a username and password. The API needs a Bearer token. JamAI Base OSS has no login of its own, so without this gate anyone with the URL could spend your model credits.
- **Frontend** is the SvelteKit UI.
- **Owl** is the API.
- **Starling** is Owl's Celery worker for background jobs.
- **Postgres** (upstream's image with pgroonga and pgvectorscale) stores tables, rows and vectors.
- **Redis** holds the cache, locks and the job queue.
- **ClickHouse** records usage meters: tokens, storage and egress.
- **MinIO** stores uploaded files.
- **Gotenberg** converts Office and HTML files to PDF for document parsing.
- **Kopi** runs Python code columns in a sandbox.
- **OTel** is a small OpenTelemetry collector that accepts Owl's telemetry.

Upstream's Docker Compose has 16 services. This template leaves out pgbouncer, the VictoriaMetrics and VictoriaLogs stack (nothing in the app reads it) and Docling. Docling is a GPU image, and the default PDF reader is a vision model instead.

**Resources.** Measured idle footprint is about 3.4 GB across all services. Most of it is the Kopi code sandbox (~1.1 GB, a Python runtime kept warm), Owl (~0.9 GB) and Starling (~0.6 GB); everything else is small. Use the Hobby plan or higher.

## Common Use Cases

- Turning a prompt chain into a table: extract, classify, translate, summarize, with each step a column you can edit and rerun
- RAG over your PDFs, Word files and spreadsheets, with retrieval you can inspect row by row
- Chat agents grounded in a Knowledge Table, served to your app over the API
- Batch-enriching data: paste or import a CSV and let LLM and code columns fill in the rest
- A self-hosted, multi-provider AI backend for internal tools (OpenAI, Anthropic, Gemini, OpenRouter and more)

## Dependencies for JamAI Base Hosting

- An API key for at least one model provider (OpenAI, Anthropic, Gemini, OpenRouter, Groq, Together, DeepSeek and others are supported)
- Postgres, Redis, ClickHouse, MinIO, Gotenberg and Kopi (all included, private network only)

### Deployment Dependencies

- [JamAI Base on GitHub](https://github.com/EmbeddedLLM/JamAIBase)
- [JamAI Base documentation](https://docs.jamaibase.com/)
- [Template source on GitHub](https://github.com/two-workspaces/jamaibase)

### Implementation Details

**Sign in.** When the deploy is green, open the Gateway domain. Sign in as `admin` with the password in the **Gateway** service's `JAMAI_PASSWORD` variable. Every visitor who gets past the gate acts as JamAI's single built-in admin.

**Add models.** A fresh install has no models. Add your provider key under **Organization → Secrets**. Then open **System → Models**, add a model (for example `openai/gpt-4.1-mini` for chat and `openai/text-embedding-3-small` for embeddings) and give it a deployment with its provider. Keys set on the **Owl** service as `OWL_OPENAI_API_KEY`, `OWL_ANTHROPIC_API_KEY`, `OWL_GEMINI_API_KEY` or `OWL_OPENROUTER_API_KEY` before the first boot are added to the organization automatically.

**Reading PDFs and Office files.** Knowledge Tables send each page to a vision model. Set `OWL_VLM_MODEL_ID` on **Owl** and **Starling** to the ID of a chat model with image input that you added (default `openai/gpt-4o-mini`). Text, Markdown, JSON and CSV files need no vision model.

**API and SDKs.** Use the Gateway URL plus `/api` as the API base, and the **Gateway**'s `JAMAI_API_TOKEN` as the token:

```python
from jamaibase import JamAI
jamai = JamAI(api_base="https://<gateway-domain>/api", token="<JAMAI_API_TOKEN>", project_id="<project id>")
```

**Custom domain.** Add the domain to the **Gateway**, then set `ORIGIN` on **Frontend** to `https://your.domain` and `OWL_FILE_PROXY_URL` on **Owl** and **Starling** to `your.domain`.

**Scaling.** `OWL_WORKERS` on **Owl** sets the number of API processes; each adds several hundred MB. Starling runs two task processes. Kopi runs one Python code cell at a time and queues the rest. Raise `MAX_CONCURRENT_EXECUTIONS` on **Kopi** for more parallel code cells, at about 1.1 GB each. If you never use Python columns, you can delete the Kopi service.

**Upgrading.** Owl, Starling and the Frontend build upstream's `main` at the commit in `JAMAI_COMMIT` (in `owl/Dockerfile` and `frontend/Dockerfile` of your fork). Change both files to a newer commit and redeploy all three; Owl migrates the database on boot.

**Back up.** Tables, rows and embeddings live in Postgres, files live in MinIO and usage history lives in ClickHouse, each on its own volume. Use Railway's volume backups.

## Why Deploy JamAI Base on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying JamAI Base on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
