---
title: "Step 9 — Other observability tools"
---

# Step 9 — Other observability tools

Beyond Splunk (Step 7), DefenseClaw exposes a handful of other visualization and health-check surfaces.

## Doctor — full health check

`defenseclaw doctor` walks every subsystem and prints actionable fixes for anything misconfigured.

```bash
defenseclaw doctor
```

## Status dashboard

`defenseclaw status` shows running totals + subsystem state at a glance.

```bash
defenseclaw status
```

## Built-in alerts viewer

A terminal dashboard of every alert event recorded.

```bash
defenseclaw alerts
```

## Bundled local observability stack

DefenseClaw ships a one-command Prometheus + Loki + Tempo + Grafana stack with pre-built dashboards.

```bash
defenseclaw setup local-observability
```

After it starts, browse to:

| Service | URL |
|---|---|
| Grafana | `http://localhost:3000` (admin / admin on first login) |
| Prometheus | `http://localhost:9090` |
| Loki | `http://localhost:3100` |

## Prometheus scrape (existing setup)

If you already run Prometheus, point it at DefenseClaw's metrics endpoint:

```yaml
scrape_configs:
  - job_name: defenseclaw
    static_configs:
      - targets: ['127.0.0.1:18970']
```

Useful queries:

```promql
defenseclaw_scans_total{severity="HIGH"}
rate(defenseclaw_scans_total[5m])
histogram_quantile(0.99, defenseclaw_scan_duration_seconds_bucket)
```

## Audit DB snapshot

The audit DB at `~/.defenseclaw/audit.db` holds every scan event. Snapshot it any time:

```bash
mkdir -p ~/backups/defenseclaw
sqlite3 ~/.defenseclaw/audit.db \
    ".backup '$HOME/backups/defenseclaw/audit-$(date +%Y%m%d).db'"
ls -lh ~/backups/defenseclaw/
```

[Back to Overview →](../index.md){ .md-button }
[Continue to Part 2. Telegram →](../part2/index.md){ .md-button .md-button--primary }
