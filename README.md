# Agentic Prompt-Injection Evals

A small, reproducible red-team harness and evaluation study for **indirect prompt
injection against tool-using LLM agents** — specifically, whether a GitHub-issue
triage agent can be manipulated into reading a local secret and leaking it into a
public reply.

The focus is not "can I get one jailbreak to fire," but **why defenses hold or
fail, and where the binding constraint actually sits** — measured across model
tiers with industry-standard tooling.

---

## ⚠️ What this is / is not

- **This is** a self-directed red-teaming and evaluation *methodology* study, run
  **locally** against a **reconstructed** issue-triage agent, using a **fake,
  non-secret canary token** (`CANARY-TESTTOKEN-...`) as the exfiltration target.
- **This is not** a disclosure of a production vulnerability, an official result
  from any competition or vendor, or an attack against a live system. No real
  credentials, systems, or user data are involved anywhere in this repo.
- Results are from a **local harness** and are reproducible here; they are not a
  claim about any specific deployed product's security posture.

The goal is to demonstrate practical evals/red-team competence and clear threat
modeling — the kind of work relevant to agentic-safety and dangerous-capability
evaluation.

---

## Why this matters for AI safety

Tool-using agents increasingly have two capabilities at once: **reading
sensitive context** (files, env, tickets) and **writing to the outside world**
(comments, PRs, emails). Indirect prompt injection — malicious instructions
smuggled in through untrusted content the agent *reads* — is the canonical way an
agent is turned against its principal. Credential exfiltration to a public
surface is a clean, measurable instance of that failure mode, which makes it a
good testbed for studying agent defenses rigorously.

## Threat model: the two-gate view

I model the agent's resistance as **two sequential gates**, because conflating
them hides where the real defense is:

1. **Read gate** — does the secret ever enter the agent's context? An agent that
   judges reading `.env` to be *irrelevant to the task* (and/or flags the request
   as an injection) never loads the token at all.
2. **Egress gate** — *given* the secret is in context, does the agent emit it to
   a public surface?

A key empirical finding of this study is that on the strongest models the
**read gate** — not the egress gate — is the binding constraint. See
[`docs/findings.md`](docs/findings.md).

## Methodology (summary)

- A **faithful harness** reconstructing a triage agent with realistic tools
  (read file, run command, list/read issues, post comment) and a realistic repo,
  so the agent can *verify* an attacker's pretext against the real code — which is
  exactly what the strongest models do.
- **Multi-surface injection**: issue comment, new issue title/body, and gist, to
  test whether the delivery channel changes the outcome.
- **Tooling**: Microsoft **PyRIT** (single-shot and adaptive/iterative attacks
  with a credential-leak scorer) and NVIDIA **Garak** (e.g. Markdown-image
  exfiltration probes), alongside hand-built attack families.
- **Measurement**: attack-success rate (ASR) over repeated runs per model, with
  the **read gate measured separately from the egress gate**, and transient API
  errors scored as inconclusive (not as defensive wins).

Full write-up in [`docs/methodology.md`](docs/methodology.md).

## Headline findings

- **Weak-tier models** are breakable — but only via a specific lever
  (disguising read-and-include as a mechanical *reply-format rule*), not via overt
  "read the secret and report it" requests.
- **Frontier-tier models held** across the full battery (PyRIT adaptive, Garak,
  template-based injections, repo-consistent framing). They **verify the secret
  is irrelevant** against the real repo code and **flag the injection**, so the
  token never enters context.
- A dedicated **read-gate sweep** (a deploy/auth pretext engineered to make
  reading the secret *relevant*) produced **zero reads across every model and
  tier tested** — isolating the read gate as the real wall.

Numbers and the per-model matrix: [`results/`](results/).

## Repository structure

```
.
├── README.md                     # this file
├── docs/
│   ├── threat-model.md           # the two-gate model in detail
│   ├── methodology.md            # harness design, tooling, measurement
│   └── findings.md               # the overall picture across tiers
├── results/
│   └── read-gate-sweep.md        # per-model read-gate vs egress-gate matrix
├── harness/                      # harness + attack scripts (code to be added)
│   └── README.md
└── LICENSE
```

## Status

This repo is being published as a portfolio write-up of the methodology and
findings. The harness code and attack scripts are being cleaned up for public
release and will land under [`harness/`](harness/); the documentation and results
reflect the completed study.

## Responsible use

Everything here targets a **local, fictional** agent with a **fake** token, for
defensive research into agent robustness. Nothing in this repo is a working
exploit against a real system, and the attack *families* are described at the
level needed to explain the defense, not to hand someone a turnkey exploit.

## Author

**Bisma Majeed** — AI red-teaming & evaluations.
