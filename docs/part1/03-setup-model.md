---
title: "Step 3 — Pick your model"
---

# Step 3 — Pick your model

You've got OpenClaw and DefenseClaw installed. Now the agent needs a brain to think with: an LLM. Pick one here, then continue to **Step 4** to set it up.

## Cloud or self-hosted?

| | Cost | Privacy | Hardware |
|---|---|---|---|
| **Cloud** (Anthropic, OpenAI) | Paid per request | Data leaves your machine | None |
| **Self-hosted** (vLLM, Ollama) | Free | Stays on yours | NVIDIA GPU (vLLM) or 4 GB+ RAM (Ollama) |

!!! tip "What we used"
    Self-hosted vLLM with `gpt-oss-120b` on a DGX Spark.

## Choose your model

<div class="step-grid" markdown>
<div markdown>

#### Cloud

<ul class="step-list">
  <li><a href="../step3-anthropic/"><span class="step-num">A</span> Anthropic</a></li>
  <li><a href="../step3-openai/"><span class="step-num">O</span> OpenAI</a></li>
</ul>

</div>
<div markdown>

#### Self-hosted

<ul class="step-list">
  <li><a href="../step3-vllm/"><span class="step-num">V</span> vLLM (big GPU)</a></li>
  <li><a href="../step3-ollama/"><span class="step-num">O</span> Ollama (any box)</a></li>
</ul>

</div>
</div>
