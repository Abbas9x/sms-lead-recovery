# SMS Lead Recovery + Missed Call Automation (n8n)

A self-hosted n8n automation that captures inbound SMS leads for local service
businesses (dental, med spa, HVAC, legal, etc.), qualifies them with Claude,
replies automatically, notifies the business owner, and runs a 24-hour follow-up.

## Stack

- **n8n** (self-hosted via Docker) — workflow engine
- **Google Sheets** — CRM / data store
- **Twilio** — inbound + outbound SMS
- **Claude API** (`claude-sonnet-4-5`) — lead classification
- **ngrok** — exposes the local n8n webhook to Twilio

## Workflow file

| File | Purpose |
|------|---------|
| `workflows/workflow-1-main.json` | Combined workflow: inbound lead processor + 24h delayed follow-up handler |

> Single file. The follow-up branch and the main inbound branch live in one
> n8n workflow connected by an internal webhook (`REPLACE_WITH_FOLLOWUP_SECRET_PATH`).
> Both webhook paths in the file are placeholders — set them to your real
> secret values after import (or via `WEBHOOK_SECRET_PATH` in `.env`).

## Setup

1. **Google Sheet** — create a sheet named `Lead Tracker` with tabs
   `leads`, `opt_outs`, `error_log`, `rate_limit_log`.
2. **Environment** — copy `.env.example` to `.env` and fill in real values.
   `.env` is git-ignored and must never be committed.
3. **n8n credentials** — create in the n8n UI:
   - `Claude API` (Header Auth, header `x-api-key`)
   - `Twilio HTTP Basic` (Basic Auth, username = Account SID, password = Auth Token)
   - `Google Sheets account` (OAuth2)
   - `Owner Notification SMTP`
4. **Import** — in n8n: *Workflows → Import from File* → pick
   `workflows/workflow-1-main.json`. Then open each node showing a missing-
   credential warning and pick the matching credential.
5. **Webhook paths** — open the two webhook nodes and replace
   `REPLACE_WITH_WEBHOOK_SECRET_PATH` and `REPLACE_WITH_FOLLOWUP_SECRET_PATH`
   with your real values from `.env`.
6. **ngrok** — `ngrok http 5678`, paste the HTTPS URL into the Twilio number's
   "A message comes in" webhook (`/webhook/<WEBHOOK_SECRET_PATH>`, POST).

## Importing / exporting

- **Export from n8n (raw):**
  `docker exec n8n n8n export:workflow --id=<ID> --output=/tmp/wf.json`
  then `docker cp n8n:/tmp/wf.json workflows/workflow-1-main.raw.json`.
  `.raw.json` files are git-ignored — always run them through the sanitizer
  before committing.
- **Import to n8n:** *Workflows → Import from File*.

## Security notes

- No secrets live in this repo. The committed workflow JSON is post-sanitized:
  hardcoded business values, personal email, real phone number, real webhook
  paths, n8n internal credential IDs, and `shared`/owner metadata have all been
  stripped or replaced with env-driven placeholders.
- `.env`, `*.raw.json`, OAuth credential JSON, and `.bak` files are excluded
  via `.gitignore`.

## Status

Sanitized workflow JSON is committed. To use it, follow the Setup steps above.
