---
title: "Step 7 — Splunk audit dashboard"
---

# Step 7 — Splunk audit dashboard

DefenseClaw ships with a bundled local Splunk. Once it's running, every scan event flows to Splunk via HEC (HTTP Event Collector) and you get a searchable SIEM-style dashboard.

![All three panes. Splunk Search & Reporting (left), gateway.log verdict stream (top-right), OpenClaw TUI demo (bottom-right)](../assets/step6-three-pane-hero.png)

## 7.1 — Configure ports

*Note: Splunk Web on `8090`, HEC on `8089`.*

```bash
DC=~/.defenseclaw/.venv/lib/python3.12/site-packages/defenseclaw
CMDSETUP=$DC/commands/cmd_setup.py
COMPOSE=$DC/_data/splunk_local_bridge/compose/docker-compose.local.yml
```

Back up the originals:

```bash
cp "$CMDSETUP" "$CMDSETUP.bak"
```

```bash
cp "$COMPOSE" "$COMPOSE.bak"
```

Set Splunk Web → `8090`:

```bash
sed -i 's/(8000, "Splunk Web")/(8090, "Splunk Web")/' "$CMDSETUP"
```

```bash
sed -i 's|http://127\.0\.0\.1:8000\(/services\|"\)|http://127.0.0.1:8090\1|g' "$CMDSETUP"
```

```bash
sed -i 's/"127.0.0.1:8000:8000"/"127.0.0.1:8090:8000"/' "$COMPOSE"
```

Set HEC → `8089`:

```bash
sed -i 's/(8088, "HEC")/(8089, "HEC")/' "$CMDSETUP"
```

```bash
sed -i 's|"http://127.0.0.1:8088/services/collector/event"|"http://127.0.0.1:8089/services/collector/event"|' "$CMDSETUP"
```

```bash
sed -i 's/"127.0.0.1:8088:8088"/"127.0.0.1:8089:8088"/' "$COMPOSE"
```

The bundle's `.env.example` (the file the bridge wrapper reads on every start) also carries the HEC URL — update it in the **same two places** so the readiness wait targets the right port:

```bash
ENV_PKG=$DC/_data/splunk_local_bridge/env/.env.example
ENV_BNDL=~/.defenseclaw/splunk-bridge/env/.env.example
for f in "$ENV_PKG" "$ENV_BNDL"; do
  [ -f "$f" ] && sed -i 's|127.0.0.1:8088/services/collector/event|127.0.0.1:8089/services/collector/event|g' "$f"
done
```

## 7.2 — Start the bundled Splunk

```bash
defenseclaw setup splunk --logs --accept-splunk-license --non-interactive
```

??? note "Expected output"
    ```
    Pre-flight checks:
      Docker installed... ok
      Docker daemon running... ok
      Port 8090 (Splunk Web)... available
      Port 8089 (HEC)... available
    Bundle refreshed: ~/.defenseclaw/splunk-bridge/ (53 files updated, 0 preserved)
    Starting local Splunk (this takes ~2 minutes)...
    ```

!!! warning "If the CLI says 'Bridge startup timed out after 5 minutes'"
    Splunk's first-time startup on arm64 (DGX Spark) can take longer than DefenseClaw's 5-minute wait. The container almost certainly kept going. Verify:

    ```bash
    docker ps --filter 'name=defenseclaw-splunk' --format 'table {{.Names}}\t{{.Status}}'
    ```

    If you see `Up X minutes (healthy)`, continue.

## 7.3 — Get credentials

*Note: the bundled installer writes seed values to `.env.example`; the CLI's `--show-credentials` looks for `.env`. Copy first:*

```bash
cp ~/.defenseclaw/splunk-bridge/env/.env.example ~/.defenseclaw/splunk-bridge/env/.env
```

```bash
defenseclaw setup splunk --show-credentials
```

??? note "Expected output"
    ```
    Splunk Web Credentials
    ──────────────────────
      URL:       http://127.0.0.1:8090
      Username:  admin
      Password:  DefenseClawLocalMode1!
    ```

The HEC token (in `.env`) defaults to `00000000-0000-0000-0000-000000000001`.

## 7.4 — Wire DefenseClaw's audit pipeline → HEC

Even though Splunk is local, `--enterprise` is the cleanest way to point DefenseClaw at it — it just means "point at an HEC endpoint."

```bash
defenseclaw setup splunk --enterprise \
    --hec-endpoint http://127.0.0.1:8089/services/collector/event \
    --hec-token 00000000-0000-0000-0000-000000000001 \
    --index defenseclaw_local \
    --source defenseclaw \
    --sourcetype defenseclaw:json \
    --skip-test
```

```bash
defenseclaw-gateway restart
```

!!! warning "Re-run this if you ever run `defenseclaw setup guardrail` again"
    Every `defenseclaw setup guardrail ...` rewrites `config.yaml` and may wipe the HEC sink. Re-run the `setup splunk --enterprise` block whenever you change guardrail settings, or audit events will stop reaching Splunk.

## 7.5 — Open the dashboard

Splunk Web is bound to `127.0.0.1:8090` on the host. SSH-tunnel it from your laptop:

```bash
ssh -L 8090:127.0.0.1:8090 youruser@your-dgx-host
```

Leave that shell open, then browse to **http://localhost:8090**.

| Field | Value |
|---|---|
| Username | `admin` |
| Password | `DefenseClawLocalMode1!` |

Click **Search & Reporting** → paste:

```spl
index=defenseclaw_local earliest=-15m
| table _time action sev cats reason
| sort -_time
```

??? note "Expected output"
    the prompt-scan verdicts from Step 4

![Splunk dashboard showing 20 audit events streaming in](../assets/step7-splunk-dashboard.png)

## Useful saved searches

In Search & Reporting → search → ⋮ → Save As → Search.

| Name | Search query |
|---|---|
| HIGH-severity prompts (last 24h) | `index=defenseclaw_local sev=HIGH earliest=-24h \| table _time cats reason` |
| Top categories this week | `index=defenseclaw_local earliest=-7d \| top cats limit=10` |
| Action rate | `index=defenseclaw_local earliest=-24h \| stats count by action` |

## Sanity-check the HEC pipeline

If verdicts stop appearing in Splunk, the HEC sink probably got wiped. Test HEC directly:

```bash
curl -k http://127.0.0.1:8089/services/collector/event \
    -H "Authorization: Splunk 00000000-0000-0000-0000-000000000001" \
    -d '{"event":"manual test","sourcetype":"defenseclaw:json","index":"defenseclaw_local"}'
```

??? note "Expected output"
    ```
    {"text":"Success","code":0}
    ```

If that works but DefenseClaw events still don't appear, re-run the `setup splunk --enterprise` block from 7.4.

[Continue to Skill + MCP scanners →](08-scanners.md){ .md-button .md-button--primary }
