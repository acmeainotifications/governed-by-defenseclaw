---
title: "Step 2 — Install OpenClaw + DefenseClaw"
---

# Step 2 — Install OpenClaw + DefenseClaw

Two installers, in this order. The version pin on OpenClaw matters, it's the version DefenseClaw's installer expects.

## Install OpenClaw (pinned version)

```bash
sudo /usr/bin/npm uninstall -g openclaw 2>/dev/null
~/.nvm/versions/node/*/bin/npm uninstall -g openclaw 2>/dev/null
hash -r

sudo /usr/bin/npm install -g openclaw@2026.3.24
hash -r

openclaw --version
```

??? note "Expected output"
    OpenClaw 2026.3.24 (cff6dc9)

![OpenClaw install output](../assets/step2-openclaw-install.png)

## Install DefenseClaw

```bash
curl -LsSf https://raw.githubusercontent.com/cisco-ai-defense/defenseclaw/main/scripts/install.sh | bash
```

The installer asks which **connector** to wire up. Pick `4` for **openclaw**:

```
─── Pick agent connector
  1) codex
  2) claudecode
  3) zeptoclaw
  4) openclaw     ← pick this one
  5) none
```

Verify:

```bash
defenseclaw --version
```

??? note "Expected output"
    defenseclaw, version 0.7.2

![DefenseClaw install output](../assets/step3-defenseclaw-install.png)

[Continue to Step 3 — Pick your model →](03-setup-model.md){ .md-button .md-button--primary }
