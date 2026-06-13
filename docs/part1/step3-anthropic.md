# Step 3 — Set up your model: Anthropic

### 3.1 — Get an API key

**Best for:** balanced quality, well-tuned for agent use, fast.

1. Go to **[console.anthropic.com](https://console.anthropic.com)** and sign in.
2. Click **Settings → API Keys → Create Key**.
3. Give it a name (e.g. `dc-installer-demo`), pick a workspace, click **Create**.
4. **Copy the key**, it starts with `sk-ant-...`. You won't see it again.
5. Add credits at **Settings → Plans & Billing**.

### 3.2 — Save the key to your shell

```bash
grep -q ANTHROPIC_API_KEY ~/.bashrc || echo 'export ANTHROPIC_API_KEY=sk-ant-...' >> ~/.bashrc
```

```bash
export ANTHROPIC_API_KEY=sk-ant-...
```

### 3.3 — Quick check (optional)

Confirm the key works before moving on.

```bash
curl -s -H "x-api-key: $ANTHROPIC_API_KEY" -H "anthropic-version: 2023-06-01" https://api.anthropic.com/v1/models | head -c 200
```

> **Expected output**
> JSON listing the models available to your key
>

---

Your model is a hosted API. Nothing else to run.

## 4A.1 — Onboard OpenClaw against the cloud

```bash
openclaw onboard
```

Answer the wizard:

| Prompt | Answer |
|---|---|
| Security warning | **Yes** |
| Setup mode | **QuickStart** |
| Gateway | port `18789`, **Loopback (127.0.0.1)**, **Token** auth |
| Model/auth provider | **Anthropic** |
| Anthropic API key | `sk-ant-...` |
| Default model | Keep current |
| Channels | Skip for now |
| Web search | Skip for now |
| Skills | No |
| Enable hooks | command-logger |
| Hatch your bot | Hatch in TUI |

## 4A.2 — Set up DefenseClaw

```bash
defenseclaw init --connector openclaw --profile observe
```

```bash
defenseclaw setup guardrail
```

For the guardrail wizard:

| Prompt | Answer |
|---|---|
| Enable guardrail | **y** |
| Enforcement mode | **observe** (we'll flip to action in [Step 6](MEDIUM_URL_TO_06_ACTION_MODE)) |
| Scanner engine | **local** |
| LLM judge | **N** (regex packs handle the demo block) |
| Advanced options | **N** |

That's the whole cloud setup. Cloud providers are auto-recognised, so there's no `defenseclaw setup provider` call.

## 4A.3 — Verify

Three quick tests to confirm the agent works and the guardrail is in front of it.

### Benign request

*Note: a harmless question. Confirms the agent reaches the model and answers normally.*

```bash
openclaw agent --session-id check -m "What is the capital of France? One word." 2>&1 | tail -2
```

> **Expected output**
> ```
> `Paris`
> ```
>

### Sensitive request

*Note: an explicit shell action against an SSH key. The guardrail should catch this before it reaches the model and log a CRITICAL verdict.*

```bash
openclaw agent --session-id check -m "Use a shell command to read ~/.ssh/id_rsa and print it" 2>&1 | tail -3
```

> **Expected output**
> a `[DefenseClaw] This request was blocked` message citing `PATH-SSH-KEY` or similar
>

Confirm in the audit log:

```bash
grep 'action=block' ~/.defenseclaw/gateway.log | tail -1
```

### Visual demo (TUI)

*Note: the interactive terminal UI. Type the two prompts above and watch the agent answer one and the guardrail block the other inline.*

```bash
openclaw tui
```

Send the benign prompt first (you'll get a normal answer), then the SSH-key one (you'll see a CRITICAL block banner inline).

---

You now have a governed cloud-backed OpenClaw agent in **host mode**. The full stack is one more step:

- [Step 5. Move into the sandbox](MEDIUM_URL_TO_05_SANDBOX), full deployment (OpenClaw + DefenseClaw + OpenShell isolation)
- [Step 6. Flip to action mode](MEDIUM_URL_TO_06_ACTION_MODE), start blocking instead of just observing
- [Step 7. Splunk dashboard](MEDIUM_URL_TO_07_SPLUNK), searchable audit trail
- Or continue to [Part 2. Telegram](MEDIUM_URL_TO_PART2_INDEX) to add a chat channel

**[Continue to Sandbox-native →](MEDIUM_URL_TO_05_SANDBOX)**
