# ClawFounder (Nosana x ElizaOS Challenge)

I built ClawFounder as a personal AI Chief of Staff for crypto founder operations.

The goal is simple: every day, I want one actionable brief that tells me what changed across engineering, treasury, token conditions, and community, without digging through dashboards.

ClawFounder runs on the ElizaOS challenge stack, uses live production APIs, and is packaged for Nosana deployment.

## What this agent does

- Tracks GitHub repo activity (`commits`, `PRs`, `issues`, recency).
- Tracks on-chain wallet state through Alchemy (`balance`, transfers, largest movement).
- Tracks token price context through CoinGecko.
- Tracks Telegram community activity and sentiment.
- Produces a `ClawFounder Daily Brief` with insights, risks, priorities, and memory.
- Sends the brief directly to Telegram.

## Why this is not just another monitoring bot

The moat is the **Founder Memory Graph**.

Instead of static threshold alerts only, ClawFounder stores recent runs and builds founder-specific baselines for:
- code output
- community volume
- treasury transfer behavior

It then reports anomalies relative to my own historical pattern and keeps a recurring risk ledger, so the signal quality compounds over time.

## Architecture

```text
ElizaOS Agent Layer (TypeScript)
        |
        v
Python Backend Bridge (backend_cli.py)
        |
        v
ClawFounder Agent Core (Python)
        |
        +--> GitHub API
        +--> Alchemy API
        +--> CoinGecko API
        +--> Telegram Bot API
        |
        v
Founder Memory Graph (SQLite)
        |
        v
Daily Founder Brief -> UI + Telegram
```

## Key files

- `src/index.ts` - ElizaOS plugin entrypoint.
- `characters/agent.character.json` - ClawFounder persona and system behavior.
- `agent/clawfounder_agent.py` - orchestration, insight generation, alerts.
- `agent/memory_store.py` - persistent run memory and history retrieval.
- `agent/tools_registry.py` - tool interface and registration.
- `backend_cli.py` - backend bridge used by UI and agent runtime.
- `ui/server.mjs` - custom UI server.
- `ui/public/*` - dashboard frontend for running and reviewing briefs.
- `nos_job_def/nosana_eliza_job_definition.json` - Nosana deployment job definition.
- `Dockerfile` - Node 23 + Python runtime container.

## Environment

Copy `.env.example` to `.env` and set:

- `OPENAI_API_KEY`
- `OPENAI_API_URL`
- `MODEL_NAME`
- `CLAWFOUNDER_GITHUB_REPO`
- `CLAWFOUNDER_GITHUB_TOKEN`
- `CLAWFOUNDER_WALLET_ADDRESS`
- `CLAWFOUNDER_WALLET_NETWORK`
- `CLAWFOUNDER_ALCHEMY_API_KEY`
- `CLAWFOUNDER_TELEGRAM_BOT_TOKEN`
- `CLAWFOUNDER_TELEGRAM_CHAT_ID`
- `CLAWFOUNDER_PRICE_TOKEN_ID`

Optional tuning:
- `CLAWFOUNDER_ALERT_NO_COMMIT_DAYS`
- `CLAWFOUNDER_ALERT_LARGE_TRANSACTION_USD`
- `CLAWFOUNDER_MEMORY_DB_PATH`
- `CLAWFOUNDER_MEMORY_RECENT_RUNS_LIMIT`

## Run locally

Install dependencies:

```bash
pnpm install
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Start the custom UI:

```bash
pnpm start
```

Open `http://localhost:3000`, run a brief, and optionally send to Telegram.

CLI alternatives:

```bash
python3 main.py --mode run-once
python3 main.py --mode run-once --send
python3 main.py --mode test-connections
```

## Deploy on Nosana

Build and push the image:

```bash
docker build -t cyborg81/foundersclaw:latest .
docker push cyborg81/foundersclaw:latest
```

In the Nosana dashboard, deploy with:
- image: `cyborg81/foundersclaw:latest`
- expose: `3000`
- top-level job `type`: `container`
- op `type`: `container/run`
- real env values (no placeholders)

The included `nos_job_def/nosana_eliza_job_definition.json` matches this structure.

## Sample brief shape

```text
ClawFounder Daily Brief — [DATE]

Code Activity:
- [insight]
- [risk signal]

Treasury:
- [insight]
- [risk signal]

Community:
- [insight]
- [sentiment summary]

Alerts:
- [critical changes]

Priorities:
- [what founder should do next]

Founder Memory Graph:
- [baseline/anomaly signals]
- [recurring risk ledger]

Memory:
- [what changed since previous run]
```

## What I would ship next

- Founder preference memory (watchlist repos, priority wallets, escalation style).
- Calendar-aware context (weekend/release/fundraising mode).
- Action layer for follow-ups (draft founder update, create issue, send alert thread).

---

Built for the Nosana x ElizaOS Agent Challenge as a practical personal ops agent, not a toy demo.
