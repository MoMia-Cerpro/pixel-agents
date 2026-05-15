# PR: fix(guardrails): enforce block_always policies in preexisting untrusted contexts

/claim #4225

Closes #4225

## Summary

When an agent starts in a preexisting untrusted context (either via `considerContextUntrusted` on the agent config or via inheritance from a parent agent), `evaluateIfContextIsTrusted` returned early without evaluating any Tool Result Policies. This bypass meant `block_always` policies were silently skipped, so dangerous tool output that should have been redacted reached the LLM unchanged. The context was marked untrusted, but the redaction layer never ran.

This change keeps the early "context is untrusted" semantics but routes tool calls through `TrustedDataPolicyModel.evaluateBulk` so `block_always` policies still fire. Trust and `sanitize_with_dual_llm` actions are intentionally skipped — the context can no longer become trusted in this turn, and the dual-LLM cost would be wasted. Permissive (YOLO) mode is unchanged: `evaluateBulk` already bypasses block policies there.

The unsafe boundary stays anchored to `preexisting_untrusted` even when a block fires, since the original untrusted state still precedes any tool call.

## Changes

- `platform/backend/src/guardrails/trusted-data.ts` — evaluate `block_always` policies under the preexisting-untrusted branch; extract a shared `collectToolCalls` helper used by both branches.
- `platform/backend/src/guardrails/trusted-data.test.ts` — four new tests:
  - block_always policy redacts tool result under preexisting untrusted context
  - mixed tool calls: only the blocked one is redacted, others pass through unchanged
  - `sanitize_with_dual_llm` is skipped (no `DualLlmSubagent.create` call) — confirms we don't pay the dual-LLM cost in a context that can't become trusted
  - permissive mode keeps its YOLO semantics under preexisting untrusted (no block, context still untrusted)

## Test plan

- [x] `pnpm test src/guardrails/trusted-data.test.ts` → 25/25 pass (21 existing + 4 new)
- [x] `pnpm test src/guardrails` → 41/41 pass
- [x] `pnpm test src/routes/proxy` (consumer of `toolResultUpdates`) → 674 pass / 6 skipped / 0 failed
- [x] `pnpm type-check` → clean
- [x] `pnpm lint` → clean
- [x] `pnpm knip` (dev + production) → clean

## Notes

- Both call sites of `evaluateIfContextIsTrusted` are unaffected:
  - `routes/proxy/llm-proxy-handler.ts` already applies the returned `toolResultUpdates` via `requestAdapter.applyToolResultUpdates(...)` — the proxy now correctly redacts blocked content end-to-end.
  - `agents/context-trust.ts:evaluateToolExecutionContextTrust` discards `toolResultUpdates` and only reads `contextIsTrusted` / `unsafeContextBoundary` — unchanged behavior.
- Boundary kind stays `preexisting_untrusted` (not `tool_result_blocked`) when a block fires, because the context was already untrusted before any tool call. The UI's preexisting-boundary affordance still applies.

---

*Disclosure: code authored with AI assistance and reviewed by me before submission.*
