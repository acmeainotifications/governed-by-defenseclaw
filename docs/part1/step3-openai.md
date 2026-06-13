# Step 4 — Set up your model: OpenAI

### 3.1 — Get an API key

**Best for:** widest model selection, GPT-4 family, the o-series reasoning models.

1. Go to **[platform.openai.com](https://platform.openai.com)** and sign in.
2. Click your profile (top-right) → **View API Keys**.
3. Click **+ Create new secret key**, name it, click **Create**.
4. **Copy the key**, it starts with `sk-...`. You won't see it again.
5. Add credits at **Settings → Billing → Add to credit balance**.

### 3.2 — Save the key to your shell

```bash
grep -q OPENAI_API_KEY ~/.bashrc || echo 'export OPENAI_API_KEY=sk-...' >> ~/.bashrc
```

```bash
export OPENAI_API_KEY=sk-...
```

### 3.3 — Quick check (optional)

Confirm the key works before moving on.

```bash
curl -s -H "Authorization: Bearer $OPENAI_API_KEY" https://api.openai.com/v1/models | head -c 200
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
| Model/auth provider | **OpenAI** |
| OpenAI API key | `sk-...` |
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
| Enforcement mode | **observe** (we'll flip to action in [Step 6](06-action-mode.md)) |
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

- [Step 5. Move into the sandbox](05-sandbox.md), full deployment (OpenClaw + DefenseClaw + OpenShell isolation)
- [Step 6. Flip to action mode](06-action-mode.md), start blocking instead of just observing
- [Step 7. Splunk dashboard](07-splunk.md), searchable audit trail
- Or continue to [Part 2. Telegram](../part2/index.md) to add a chat channel

**[Continue to Step 5 — Sandbox-native →](05-sandbox.md)**
