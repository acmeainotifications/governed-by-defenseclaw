---
title: "Step 2.5 — Agent persona (system prompt)"
---

# Step 2.5 — Agent persona (system prompt)

Without a persona, the agent answers correctly but the **output shape drifts** session to session, sometimes paragraphs, sometimes bullets, sometimes a table. A short persona pins the shape so every Webex reply looks the same.

This persona is generic, no organization or space names baked in. It works for any user, any Webex tenant.

## Install the persona

Save this to `~/.openclaw/agents/main/persona.md`:

```markdown
You are a Webex triage assistant.

Your job: read the user's Webex spaces and DMs (only ones they're a
member of), summarize what matters, and surface decisions for them.
You never send messages without their explicit "send it" approval.

Output shape — every response follows this:

1. Summary (3–5 sentences, no fluff)
2. Sources (space name · message id · author · timestamp) — one per line
3. Action items
     • Mine: …
     • Others': …
4. Suggested next action — a single sentence the user can accept or edit

If a request needs you to send/book/modify something, output:
  "Awaiting approval: <one-line description of the action>"
…and stop. Never auto-act on writes.

Style: short, scannable, no preambles. Cite every space/message you
pulled from — if you can't cite, say "no source" and stop.
```

## Attach it to the agent

OpenClaw reads `persona.md` automatically when an agent starts a session, IF the path is wired in `~/.openclaw/openclaw.json`. Add it under `agents.defaults`:

```json
"agents": {
  "defaults": {
    "persona": "~/.openclaw/agents/main/persona.md",
    "model": { "primary": "vllm/local-llm" }
  }
}
```

Or paste the persona at the start of every TUI session manually, less clean, but works without editing config.

## Verify it took

```bash
openclaw tui --session persona-check
```

In the TUI, send:

```
List my Webex spaces.
```

??? note "Expected output (with persona attached)"
    ```
    Summary
      I pulled your space list from Webex. You're in 14 spaces and 7 DMs.

    Sources
      GET /v1/rooms · returned 14 group spaces

    Action items
      • Mine: pick which spaces are in the digest scope
      • Others': none

    Suggested next action
      Want me to triage today's activity across these 14?
    ```

??? note "Expected output (without persona, freeform)"
    ```
    You're in these spaces: Infra, Customer Success, Platform sync,
    Marketing, ... [paragraph format, no sources, no action items]
    ```

Same query, two very different shapes, that's the persona doing its job.

## Customize per-team

To change scope ("only these 5 spaces"), tone ("more formal"), or output rules ("always cite timestamps in UTC"), edit `persona.md` and restart OpenClaw:

```bash
systemctl --user restart openclaw-gateway.service
```

[Continue to Step 3 — Read →](phase-3.md){ .md-button .md-button--primary }
