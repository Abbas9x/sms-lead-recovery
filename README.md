# SMS Lead Recovery + Missed Call Automation (n8n)

A self-hosted n8n automation that captures inbound SMS leads for local service
businesses (dental, med spa, HVAC, legal, etc.), qualifies them with Claude,
replies automatically, notifies the business owner, and runs a 24-hour follow-up.

## Stack

- **n8n** (self-hosted via Docker) — workflow engine
- **Google Sheets** — CRM / data store
- **Twilio** — inbound + outbound SMS
- **Claude API** (`claude-sonnet-4-20250514`) — lead classification
- **ngrok** — exposes the local n8n webhook to Twilio

## Workflows

| File | Purpose |
|------|---------|
| `workflows/workflow-1-main.json` | Main lead processor — runs on every inbound SMS |
| `workflows/workflow-2-followup.json` | Delayed 24h follow-up handler |

> The workflow JSON files are exported from the live n8n instance. See
> "Importing / exporting" below.

## Setup

1. **Google Sheet** — create a sheet named `Lead Tracker` with tabs:
   `leads`, `opt_outs`, `error_log`, `rate_limit_log` (see column headers in the
   project docs).
2. **Environment** — copy `.env.example` to `.env` and fill in real values.
   `.env` is git-ignored and must never be committed.
3. **n8n credentials** — create: `Claude API` (Header Auth, `x-api-key`),
   `Twilio HTTP Basic` (Basic Auth, SID/token), `Google Sheets Account` (OAuth2),
   `Owner Notification SMTP`.
4. **Import workflows** — in n8n, import both files from `workflows/`, then wire
   each node's credential dropdown.
5. **ngrok** — `ngrok http 5678`, paste the HTTPS URL into the Twilio number's
   "A message comes in" webhook (`/webhook/<WEBHOOK_SECRET_PATH>`, POST).

## Importing / exporting workflows

- **Export from n8n:** open a workflow → menu (⋯) → *Download* → save into
  `workflows/`.
- **Import to n8n:** *Workflows* → *Import from File*.

## Security notes

- No secrets live in this repo. All credentials are referenced by name in n8n
  and all config comes from environment variables.
- `.env`, keys, and credential JSON files are excluded via `.gitignore`.

## Status

Repo scaffold is in place. Workflow JSON exports are added once the live n8n
instance is reachable.
