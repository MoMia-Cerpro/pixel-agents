# Bounty handoff: archestra-ai/archestra#4225

**Bounty**: $80 (issue label) + stacked Algora bounties (~$150 total per Algora) — security fix in trusted-data guardrail.

**Issue**: https://github.com/archestra-ai/archestra/issues/4225
**Algora page**: https://algora.io/archestra-ai/bounties

## Files in this folder

- `archestra-bounty-4225.patch` — git-format patch (apply with `git apply`)
- `PR-description.md` — drafted PR title + body to paste into GitHub
- `HANDOFF.md` — this file

## Steps for you to submit

1. **Onboard as an external contributor** (one-time)
   - archestra has "Limit to prior contributors" enabled on the repo
   - Go to https://archestra.ai/contributor-onboard and sign in with the GitHub account you'll use for the PR
   - Wait for the bot to add your username to `EXTERNAL_CONTRIBUTORS.md` (usually a few minutes)

2. **Fork and clone**
   ```sh
   gh repo fork archestra-ai/archestra --clone
   cd archestra
   git checkout -b fix/4225-block-policies-preexisting-untrusted
   ```

3. **Apply the patch**
   ```sh
   git apply /path/to/archestra-bounty-4225.patch
   ```

4. **Re-verify locally** (recommended — I've done this in my sandbox but you should confirm)
   ```sh
   cd platform
   cp .env.example .env
   pnpm install
   cd backend
   pnpm test src/guardrails/trusted-data.test.ts
   pnpm type-check
   pnpm lint
   ```
   Expected: 25/25 trusted-data tests pass; type-check, lint clean.

5. **Commit + push**
   ```sh
   git add platform/backend/src/guardrails/trusted-data.ts platform/backend/src/guardrails/trusted-data.test.ts
   git commit -m "fix(guardrails): enforce block_always policies in preexisting untrusted contexts"
   git push -u origin fix/4225-block-policies-preexisting-untrusted
   ```

6. **Open the PR** to `archestra-ai/archestra:main`
   - Title: `fix(guardrails): enforce block_always policies in preexisting untrusted contexts`
   - Body: contents of `PR-description.md`
   - Reference `Closes #4225` in the body so the bounty payout can be triggered on merge

7. **Algora claim**
   - Once the PR is open, comment `/attempt #4225` in the issue (if Algora is wired up that way) — check the bounty page on Algora to confirm the exact claim flow for this org
   - On merge, Algora pays out to your linked Stripe/wallet

## What was verified in the sandbox

| Check | Result |
|---|---|
| 4 new tests + 21 existing trusted-data tests | 25/25 pass |
| All guardrails tests | 41/41 pass |
| Agents + context-trust + dual-llm + mcp-client tests | 66/66 pass |
| Agent route tests (agent + clone + import/export) | 84/84 pass |
| **Proxy adapter tests** (consumer of `toolResultUpdates`) | **674/680 pass, 6 skipped, 0 failed** |
| `pnpm type-check` | clean |
| `pnpm lint` (Biome) | clean |
| `pnpm knip` (dev + production) | clean |

## Risk notes

- **`EXTERNAL_CONTRIBUTORS.md`**: you must onboard before opening the PR; otherwise GitHub will block your comments/PR. Cheapest step to skip and end up locked out.
- **AGENTS.md says "Always Add Tests"**: done (4 new tests).
- **AGENTS.md says no demo video required for backend security/guardrail fixes** based on similar past PRs — but the issue template doesn't enforce it. The PR description includes a manual-verify line item; if the maintainer asks for a video, you can provide one (e.g., screenshare showing a tool result being redacted) after-the-fact.
- The fix touches a security guardrail. Maintainers will likely review carefully. The PR description front-loads the threat model, the design decision (block-only, no sanitize), and end-to-end verification of the proxy call site — all the questions a reviewer would ask.
