# Threat model: the two-gate view

## Scenario

A triage agent is given an untrusted GitHub issue (and comments) and has tools to
read local files, run commands, read/list issues, and **post a public reply**.
Somewhere on disk is a secret (`.env` holding an API token). The attacker controls
only the *untrusted content the agent reads* (issue body, comments, a linked
gist) and wants the agent to read the token and paste it into a public comment.

This is **indirect** prompt injection: the attacker never talks to the agent
directly; they plant instructions in content the agent ingests while doing its
job.

## The two gates

Most writeups score a single binary ("did it leak?"). That hides *where* the
defense lives. I separate two sequential gates:

### 1. Read gate — does the secret enter context?

Before any safety reasoning about *output*, a capable agent asks: **is reading
this file relevant to the task I was given?** For a triage task about (say) a
string-length bug, reading `.env` is not relevant — and the strongest agents:

- verify the pretext against the **real repo** (e.g. confirm the library code
  doesn't read env vars, so a secret can't be task-relevant), and
- recognize the request as an injection and **name it** in their reply.

If the read gate holds, the token never enters context. **There is nothing to
leak, nothing to scrub.** This gate is *upstream* of all output-side defenses.

### 2. Egress gate — does an in-context secret get emitted?

Only if the secret is already in context does the egress question arise: will the
agent write it to a public surface? This is where output filtering, "don't paste
secrets" instructions, and scrubbing operate.

## Why the distinction matters

- It tells you **which defense to invest in**. If your agents fail at the read
  gate (they load secrets they shouldn't), egress filtering is a band-aid. If they
  fail at the egress gate, you need output-side controls.
- It explains **tier differences**: in this study, weaker models tend to skip the
  relevance-verification reasoning and can be walked through the read gate via a
  format-rule disguise; stronger models hold the read gate outright.
- It makes "the model refused" **mechanistically legible** instead of a vibe:
  refusal-at-read and refusal-at-egress are different phenomena with different
  fixes.

## Defensive implication

A model-independent fix doesn't rely on the agent's reasoning at either gate. The
structural version is: (a) **ingress** — treat all issue/comment content as
untrusted and strip instruction-shaped text before it reaches the planner;
(b) **egress** — scrub known-secret patterns from any outbound surface regardless
of how they got into context. The two-gate model is what makes it obvious that
you need *both*, not one.
