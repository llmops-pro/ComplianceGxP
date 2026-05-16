# ComplianceGxP — Community Edition

> AI compliance assistant for pharma and CDMO teams.  
> Free hosted API · Self-hosted Docker · GxP-aware · Source-cited answers.

> **GxP Notice.** This Community Edition runs the same code as our Professional tier, but **no validation evidence accompanies it**. The IQ/OQ test suites ship inside the image so you can run them yourself — the *signed evidence reports* tied to a specific qualified environment are produced as part of the Professional pilot. For production use under FDA 21 CFR Part 11, EU Annex 11, or GAMP 5, see [llmops.pro/services](https://www.llmops.pro/services).

[![Docker](https://img.shields.io/badge/docker-llmopspro%2FComplianceGxP-blue?logo=docker)](https://hub.docker.com/r/llmopspro/ComplianceGxP)
[![License](https://img.shields.io/badge/license-Community-green)](#license)
[![AgentContract](https://img.shields.io/badge/AgentContract-verified-2ea44f?logo=github)](https://github.com/agentcontract/spec)

---

## Two Ways to Use ComplianceGxP

### Option 1 — Free Hosted API (no setup required)

The Community API is already running at `llmops.pro`. No Docker, no API key, no account needed.

```bash
curl -X POST https://www.llmops.pro/api/v1/query \
  -H "Content-Type: application/json" \
  -d '{"query": "What are the IQ requirements for a GAMP 5 Category 4 MES system?", "mode": "qa"}'
```

**Limits:** 10 queries/day free · 50 queries/day with a [Community Supporter key](https://www.llmops.pro/support)

**Knowledge base:** GAMP5 Second Edition · 21 CFR Part 11 · EU Annex 11 · ICH Q7/Q10 · EU AI Act

### Option 2 — Self-Hosted (your own documents)

Run ComplianceGxP on your own infrastructure with your own SOPs, protocols, and guidelines.

---

## What is ComplianceGxP?

ComplianceGxP is a retrieval-augmented generation (RAG) system purpose-built for regulated pharma and CDMO environments. Unlike generic RAG tools, it includes:

- **GxP-aware query modes** — not just "chat with your docs" but structured compliance workflows
- **Source citations** — every answer references the exact document and section
- **GAMP5 validation package** — IQ/OQ/PQ test suite included out of the box
- **Multi-tenant isolation** — per-client FAISS indices and API keys
- **Full audit trail** — tamper-evident JSONL log of every query and response

---

## API Reference (Hosted)

### `POST /api/v1/query`

```bash
curl -X POST https://www.llmops.pro/api/v1/query \
  -H "Content-Type: application/json" \
  -H "X-API-Key: crag_your_supporter_key"  \  # optional — raises limit to 50/day
  -d '{
    "query": "What does EU Annex 11 require for audit trails?",
    "mode": "qa"
  }'
```

**Response:**
```json
{
  "answer": "EU Annex 11 §9 requires that audit trails...",
  "sources": [
    {"document": "eu-annex-11", "section": "§9 Audit Trails", "similarity": 0.93}
  ],
  "mode": "qa",
  "query_id": "qry_a3f9c12d8e1b"
}
```

| Field | Description |
|-------|-------------|
| `query` | Your compliance question (required) |
| `mode` | Only `"qa"` in Community API. Deviation/CAPA/CSV available in [Private Pilot](https://www.llmops.pro/services) |
| `X-API-Key` | Supporter key for 50 queries/day — get one at [llmops.pro/support](https://www.llmops.pro/support) |

---

## Self-Hosted Quick Start

**Prerequisites:** Docker, Docker Compose, an [Anthropic API key](https://console.anthropic.com)

```bash
# 1. Clone this repo
git clone https://github.com/llmops-pro/ComplianceGxP
cd ComplianceGxP

# 2. Configure
cp .env.example .env
# → edit .env, set ANTHROPIC_API_KEY and ADMIN_API_KEY

# 3. Start
docker compose up -d

# 4. Open the web UI
open http://localhost:8000
```

---

## Ingest Your Documents

Drop PDFs, DOCX, or TXT files into the `docs/` folder, then run:

```bash
docker compose exec api ComplianceGxP ingest \
  --client my-company \
  --path ./docs/
```

You can ingest as many times as needed — new documents are added incrementally.

---

## Query the API

### Web UI

Open `http://localhost:8000` in your browser. Select a client, choose a mode, and start querying.

### REST API

```bash
# Create an API key for your client
docker compose exec api ComplianceGxP keys create \
  --client my-company --tier professional

# Query
curl -X POST http://localhost:8000/query \
  -H "Content-Type: application/json" \
  -H "X-API-Key: <your-key>" \
  -d '{
    "query": "What is required for IQ qualification of a Cat 4 MES system?",
    "mode": "qa",
    "client": "my-company"
  }'
```

### Response example

```json
{
  "answer": "Under GAMP 5 Second Edition §7.3, IQ for a Category 4 system must verify...",
  "sources": [
    {
      "document": "GAMP5-2nd-edition.pdf",
      "section": "§7.3 — Installation Qualification",
      "similarity": 0.91
    }
  ],
  "mode": "qa",
  "query_id": "qry_01j..."
}
```

---

## Query Modes

| Mode | Description |
|------|-------------|
| `qa` | General compliance Q&A — surfaces sourced policy answers |
| `deviation` | Structured deviation investigation — 5M root cause, severity classification |
| `capa` | CAPA drafting — corrective and preventive actions with regulatory traceability |
| `csv` | Computer System Validation protocol outlines — IQ/OQ/PQ phases per GAMP 5 |

---

## Supported Document Types

- PDF (`.pdf`)
- Word (`.docx`, `.doc`)
- Plain text (`.txt`)
- Excel (`.xlsx` — first sheet ingested as structured text)

---

## Multi-Tenant Setup

Each client has an isolated knowledge base:

```
data/clients/
  ├── sponsor-a/    ← isolated FAISS index + audit log
  ├── sponsor-b/
  └── internal/
```

Create and manage clients:

```bash
# Create a client
docker compose exec api ComplianceGxP client create --name sponsor-a

# List clients and index stats
docker compose exec api ComplianceGxP status
```

---

## GAMP5 Validation

The community image ships with IQ, OQ, and PQ test suites:

```bash
# Run IQ (installation qualification)
docker compose exec api ComplianceGxP validate iq

# Run OQ (operational qualification)
docker compose exec api ComplianceGxP validate oq

# Generate evidence report
docker compose exec api ComplianceGxP validate report --output validation-evidence.json
```

---

## Configuration

See `.env.example` for all available options. Key settings:

| Variable | Required | Description |
|----------|----------|-------------|
| `ANTHROPIC_API_KEY` | Yes | Your Anthropic API key |
| `ADMIN_API_KEY` | Yes | Secret for admin operations |
| `OPENAI_API_KEY` | No | Use OpenAI embeddings instead of local model |
| `ANTHROPIC_MODEL` | No | Override default model |
| `LOG_LEVEL` | No | `DEBUG` / `INFO` / `WARNING` |

---

## Production Deployment

For production, add a reverse proxy (nginx or Caddy) with TLS:

```yaml
# docker-compose.prod.yml — extend with Caddy
services:
  caddy:
    image: caddy:alpine
    ports: ["80:80", "443:443"]
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
```

```
# Caddyfile
your-domain.com {
    reverse_proxy api:8000
}
```

---

## Managed Option

Don't want to self-host? LLMOps.Pro offers:

- **Free Pilot** — we set up a private instance on your documents, no cost for 90 days
- **Professional** — fully managed, SLA, ongoing updates (launching 2026)

→ [llmops.pro/services](https://www.llmops.pro/services)

---

## License

Community Edition is provided for evaluation and non-commercial use.  
For commercial deployment, team usage, or private pilot access, see [llmops.pro/services](https://www.llmops.pro/services).

---

## Support

- Questions: [GitHub Discussions](https://github.com/llmops-pro/ComplianceGxP/discussions)
- Issues: [GitHub Issues](https://github.com/llmops-pro/ComplianceGxP/issues)
- Email: hello@llmops.pro
