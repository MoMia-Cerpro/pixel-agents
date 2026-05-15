# Bounty handoff: archestra-ai/archestra#4225

**Bounty**: $80 (issue label) + stacked Algora bounties (~$150 total per Algora's bounty board)
**Issue**: https://github.com/archestra-ai/archestra/issues/4225
**Algora page**: https://algora.io/archestra-ai/bounties
**Payout method**: Stripe Connect (ACH / SEPA / debit card), 1–3 business days after maintainer clicks "Reward"

## Why you have to do the submission (not me)

- My GitHub MCP tools are scoped to `momia-cerpro/pixel-agents` only — I cannot fork, push to, or open PRs on `archestra-ai/archestra`.
- archestra has **"Limit to prior contributors"** enabled — only handles in `EXTERNAL_CONTRIBUTORS.md` can comment or PR. You need to onboard first.
- Algora pays the GitHub author of the merged PR. Payout requires a Stripe Connect account tied to *your* identity.

## Files in this folder

- `archestra-bounty-4225.patch` — git-format patch (apply with `git apply`)
- `PR-description.md` — drafted PR body. Already includes `/claim #4225` and an AI-assist disclosure line.
- `HANDOFF.md` — this file

## Exact submission sequence

### 0. One-time setup (do these in parallel)

| Step | Where | Why |
|---|---|---|
| Sign in at https://archestra.ai/contributor-onboard with your GitHub | archestra.ai | Adds you to `EXTERNAL_CONTRIBUTORS.md` so the repo will accept your PR. Without this you're blocked. |
| Sign up at https://algora.io with the same GitHub | algora.io | So when the bot picks up your `/claim`, your handle resolves to a payee. |
| In Algora settings → Payments → connect Stripe | algora.io | Required to receive money. Stripe Connect onboarding takes 5–15 min (ID + bank details). |

### 1. Fork, branch, apply

```sh
gh repo fork archestra-ai/archestra --clone
cd archestra
git checkout -b fix/4225-block-policies-preexisting-untrusted
git apply /path/to/archestra-bounty-4225.patch
```

### 2. Re-verify locally (recommended — I tested in my sandbox but you should re-run)

```sh
cd platform
cp .env.example .env       # provides ARCHESTRA_DATABASE_URL placeholder so config loads
pnpm install
cd backend
pnpm test src/guardrails/trusted-data.test.ts   # expect 25/25
pnpm type-check                                  # clean
pnpm lint                                        # clean
```

### 3. Commit + push

```sh
git add platform/backend/src/guardrails/trusted-data.ts platform/backend/src/guardrails/trusted-data.test.ts
git commit -m "fix(guardrails): enforce block_always policies in preexisting untrusted contexts"
git push -u origin fix/4225-block-policies-preexisting-untrusted
```

### 4. Open the PR

- Base: `archestra-ai/archestra:main`
- Title: `fix(guardrails): enforce block_always policies in preexisting untrusted contexts`
- Body: paste the contents of `PR-description.md` as-is
  - It contains `/claim #4225` — the Algora bot will see this when the PR opens and link it to the bounty
  - It contains `Closes #4225` so the issue auto-closes on merge
  - It contains the AI-assist disclosure line

### 5. After the PR opens

- The Algora bot will comment on the issue thread linking your PR.
- Reply to any review feedback. If the maintainer requests a demo video (some bounties require it), record a short screen capture: set up an agent with `considerContextUntrusted=true`, attach a `block_always` trusted-data policy, fire a tool call, show the LLM proxy log line "Messages filtered after trusted data evaluation" + the `[Content blocked by policy: ...]` substitution.
- On merge, the maintainer clicks "Reward" in their Algora dashboard. Funds land in your connected Stripe in 1–3 business days.

## What was verified in the sandbox

| Check | Result |
|---|---|
| 4 new tests + 21 existing trusted-data tests | **25/25 pass** |
| All guardrails tests | 41/41 pass |
| Agents + context-trust + dual-llm + mcp-client tests | 66/66 pass |
| Agent route tests | 84/84 pass |
| **Proxy adapter tests** (consumer of `toolResultUpdates`) | **674 pass / 6 skipped / 0 failed** |
| `pnpm type-check` | clean |
| `pnpm lint` (Biome) | clean |
| `pnpm knip` (dev + production) | clean |

## Risk notes

- **Contributor onboarding is the most-skipped step.** If you forget it, GitHub will silently swallow your PR comment activity and the maintainer won't see it.
- **`/claim #4225` must be in the PR body, not a comment.** Algora's bot parses the body on PR open.
- The fix touches a security guardrail. The PR description front-loads the threat model, the design decision (block-only, no sanitize), and end-to-end verification of the proxy call site — all the questions a reviewer would ask. Be ready to defend the choice to skip `sanitize_with_dual_llm` if a reviewer pushes back; my reasoning is in the PR notes.
