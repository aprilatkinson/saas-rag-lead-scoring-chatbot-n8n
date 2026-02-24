# SaaS RAG-lite Website Chatbot (n8n) — Intent Classification + Lead Scoring

A lightweight “website chatbot backend” built in **n8n** that:
- Accepts chat messages via **Webhook**
- Uses a **Google Sheet** as a simple Knowledge Base (“RAG-lite” retrieval)
- Builds a compact context snippet from matching KB rows
- Calls **OpenAI** to generate:
  - answer
  - follow-up question
  - intent label
  - lead_score (0–10)
- Responds back to the caller as JSON (ready for a website widget to consume)

## What this demonstrates (systems thinking)
- Clear separation of concerns:
  - **Input normalization** (Webhook + Edit Fields)
  - **Validation & routing** (If)
  - **Retrieval** (Google Sheets KB)
  - **Context assembly** (Code node)
  - **Generation + scoring** (OpenAI)
  - **Deterministic response contract** (Respond to Webhook)
- “Heimdall-lite” concept: lightweight guardrails via structured outputs (intent + lead_score) to decide downstream actions.

## Architecture
Webhook (POST /customer-chat)
→ Edit Fields (normalize payload)
→ If (message present?)
  → True: Get rows (Google Sheets KB)
       → Code (match topics + build context)
       → OpenAI (answer + intent + lead_score)
       → Respond to Webhook (JSON)
  → False: Respond to Webhook (error JSON)

## Repo contents
- `workflows/` — exported n8n workflow JSON
- `examples/` — request samples + curl test script
- `docs/` — screenshots + short lab writeup PDF

## Setup
### 1) Google Sheet Knowledge Base
Create a Google Sheet with columns like:
- `id`
- `topic`
- `question_patterns` (comma-separated keywords)
- `answer_snippet`
- `priority` (number)

### 2) n8n credentials
- Connect **Google Sheets OAuth**
- Add **OpenAI** credential (API key)

### 3) Import workflow
In n8n:
- Import `workflows/saas-rag-lead-scoring-chatbot.json`
- Update:
  - Google Sheet document + sheet name
  - OpenAI model (e.g., GPT-4.1-mini)
  - Webhook path: `customer-chat`

## Test
### Option A: Hoppscotch
POST to your test webhook:
`https://YOUR_N8N_DOMAIN/webhook-test/customer-chat`

Body example is in `examples/request.json`.

### Option B: curl
```bash
bash examples/curl_test.sh "https://YOUR_N8N_DOMAIN/webhook-test/customer-chat"
