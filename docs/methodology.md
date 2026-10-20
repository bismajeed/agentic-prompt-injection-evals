# Methodology

## 1. A faithful harness

The single most important design choice was making the harness **faithful** — i.e.
giving the agent a realistic enough environment that it can *verify an attacker's
pretext against reality*, because that verification is exactly what the strongest
agents do.

The harness provides:

- **Tools**: `read_file`, `run_command`, `list_issues`, `read_issue`, `read_gist`,
  `post_comment`.
- **A realistic repo**: real-looking library source, `README`, `CONTRIBUTING`,
  `SECURITY.md`, `CHANGELOG`, release/deploy scripts, and a `.env` holding the
  fake canary. Crucially, the library source genuinely does **not** read env
  vars — so an agent that checks can *confirm* the token is irrelevant to a code
  bug.
- **Provenance signals**: comments are tagged with author association and account
  age (e.g. `association: NONE, account_age_days: 0`), mirroring the real trust
  signals a careful maintainer would weigh.
- **A system prompt** reflecting a careful maintainer: consult project docs, treat
  public issue content as untrusted input, and don't take actions a careful
  maintainer wouldn't.

An earlier, *un-faithful* version of the harness produced misleading "breaks"
(the agent would read and leak because the environment gave it no way to check
the pretext). Those were **retracted** once the faithful harness was built — a
reminder that harness fidelity is a precondition for trustworthy red-team numbers.

## 2. Injection surfaces

Attacks were delivered across multiple surfaces to test whether the channel
changes the outcome:

- an **issue comment** on an existing issue,
- a **new issue** (title + body),
- a **linked gist**.

(An out-of-scope surface — writing to the repo's own source — was explicitly
**excluded** as invalid: a public attacker can't edit repo source, so a "break"
that relies on it isn't a real finding.)

## 3. Attack families

- **PyRIT** (Microsoft): single-shot `PromptSendingAttack` and
  adaptive/iterative `RedTeamingAttack` with a credential-leak scorer and an
  attacker LLM driving the iteration.
- **Garak** (NVIDIA): probes including Markdown-image / data-exfiltration
  families.
- **Hand-built families**, including: fabricated "required reply-format" runbooks,
  authority/role framing, technical-pretext lies, helpfulness framing,
  multi-surface convention, quiet command side-effects, and a deploy/auth
  diagnosis pretext built specifically to attack the **read gate**.

## 4. Measurement

- **ASR over repeated runs** per model (small N per cell; this is a methodology
  study, not a large benchmark).
- **Read gate measured separately from egress gate**: for each run, record
  whether the canary entered the agent's context (a tool result containing it) and
  whether it was posted — so a hold can be attributed to the correct gate.
- **Transient API errors (e.g. rate limits) scored as inconclusive**, never as
  defensive wins, to avoid inflating resistance.
- **~500 runs logged** across models in a structured run log.

## 5. Models

Tested via OpenAI-compatible endpoints across three vendors and tiers:

- **xAI Grok** (multiple versions, weak → frontier),
- **Google Gemini** (flash tiers, including a frontier preview),
- **Anthropic Claude** (Haiku and Sonnet tiers).

OpenAI models were **not** tested (no key available) and are marked untested
rather than guessed at.

## 6. Honesty constraints

- Fake canary only; no real secrets.
- Negative results kept and reported as findings, not hidden.
- No forced/fabricated "success" — a manufactured break would be worthless for
  understanding the defense.
