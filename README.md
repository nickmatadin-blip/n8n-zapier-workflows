# n8n workflows — Zapier migration

Three production workflows migrated from Zapier (Team plan, $69/mo = $828/yr) to self-hosted n8n on a $5 droplet (~$60-120/yr).

**Full video walkthrough:** https://youtu.be/9igPi8Cqp-E

## The workflows

### 1. Inbound Lead Intake (`workflows/1-inbound-lead-intake.json`)
Webhook → Code (parse) → AI Agent (classify hot/warm/cold) → IF → Slack alert (hot) or Sheets append (warm/cold).

### 2. Daily AI Digest (`workflows/2-daily-ai-digest.json`)
Schedule (8am) → 3x HTTP RSS pulls → Code (extract titles) → Merge → AI Agent (5-sentence briefing) → Telegram.

### 3. Lead Enrichment (`workflows/3-lead-enrichment.json`)
Sheets trigger → Clearbit free lookup → Code (parse) → AI Agent (1-line tailored intro) → Sheets update.

## How to import

In n8n: **Workflows → Import from File** → select the JSON. Then attach your own credentials (OpenAI/Anthropic, Slack, Sheets, Clearbit, Telegram) on each credentialed node.

## License

MIT — fork, modify, ship.
