# Harness (code coming)

This directory will hold the reproducible harness and attack scripts described in
[`../docs/methodology.md`](../docs/methodology.md). They are being cleaned up for
public release (removing local paths, keys, and scratch files) before being added
here.

Planned contents:

- `harness_core.py` — the agent loop, tools, multi-surface injection, run logging,
  and the read-gate/egress-gate scorer.
- `server.py` — an OpenAI-compatible endpoint wrapper so PyRIT/Garak can drive the
  agent.
- `attacks/` — the attack families (format-rule runbook, deploy/auth read-gate
  pretext, PyRIT adaptive driver, Garak configs).
- `fake_repo/` — the realistic repo the agent triages, including the **fake**
  canary in `.env`.

No API keys, secrets, or real credentials will ever be committed — see
[`../.gitignore`](../.gitignore).
