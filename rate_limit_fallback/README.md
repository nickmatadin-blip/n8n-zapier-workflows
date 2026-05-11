# OpenAI Rate Limit Fallback Pattern for n8n

The 3-node pattern that catches OpenAI 429 errors and switches to a cheaper model automatically, so your workflows don't silently die at 2 AM.

## The problem

Chain a few AI nodes together and you'll hit OpenAI's per-minute rate limit faster than you think. n8n's default behavior on a 429 response: the node fails, the workflow stops, and you don't find out until breakfast. Orders missed. Customer waiting on an AI reply that never comes.

## The fix (3 nodes)

```
[Trigger] → [OpenAI primary (gpt-4o)] → [IF: errored?]
                                          ├─ NO  → [Continue normal flow]
                                          └─ YES → [Wait 30s] → [OpenAI fallback (gpt-4o-mini)] → [Continue]
```

The 3 added nodes:

1. **IF node** — check `{{ $json.error }}` is truthy
2. **Wait node** — 30 seconds, gives the rate-limit window time to reset
3. **OpenAI fallback node** — same prompt, cheaper model (`gpt-4o-mini` or `gpt-3.5-turbo`)

On the primary OpenAI node, set **Continue On Fail** to true so the error doesn't kill the workflow — it just flows into the IF.

## What's in this folder

- `rate_limit_fallback.json` — importable n8n workflow with the pattern wired up. Replace the trigger with whatever you actually use (webhook, schedule, etc.).
- `screenshots/` — visual reference of the node configuration.

## Why 30 seconds

OpenAI's rate limits reset on a rolling per-minute window. 30s is enough to clear most transient spikes without burning user wait time. If you hit sustained rate limits, the fallback model (gpt-4o-mini) has its own quota that you're unlikely to exhaust at the same time.

## Why a cheaper fallback

Two reasons:
1. **Cost insurance.** If gpt-4o is rate-limited because of a volume spike, switching to gpt-4o-mini for the spike costs roughly 1/16 as much per token. Your bill stays sane.
2. **Quota separation.** Different models have separate rate-limit buckets. A 429 on gpt-4o doesn't mean gpt-4o-mini is also throttled.

## Variations

- **Anthropic fallback:** Swap the fallback model for an Anthropic Claude node. Different provider, completely independent rate-limit pool.
- **OpenRouter:** Route both nodes through OpenRouter and let their router handle failover. One credential, automatic failover across providers. Costs ~5% on top of OpenAI's per-token rate.
- **Error workflow pattern:** If you have many workflows hitting OpenAI, set them all to use a single shared Error Workflow that handles the fallback. Avoids duplicating the 3-node pattern in every flow.

## License

MIT (see repo root). Use it, modify it, ship it.
