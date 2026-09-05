# Capone Plan

> Checklist-style TODO for the Cline **CLI** (`apps/cli`). All items below are CLI-only unless otherwise noted.

## Requirements

- [ ] **Only allow completion** via `submit_and_exit` tool from the model *(CLI)*
- [ ] **Reject other completion signals** (max-tokens, error, aborted, stop) as final completions *(CLI)*
- [ ] **Ask the user** "Is the task complete?" when a non-submit completion fires *(CLI)*
- [ ] **Continue working** if the user says the task is not complete *(CLI)*
- [ ] **Ensure the completion signal is sent** in the proper tool format (`submit_and_exit`) *(CLI)*
- [ ] **Retry at least 10 times** before giving up *(CLI)*
- [ ] **Support agent handoff** — spawn other agents and delegate tasks to them using a different model *(CLI)*

---

## 1. Core Logic — `sdk/packages/agents/src/agent-runtime.ts` *(CLI)*

- [ ] Modify `findCompletingToolMessage()` to ONLY accept the `submit_and_exit` tool (`lifecycle.completesRun === true` + name check)
- [ ] Add `handleNonSubmitCompletion(finishReason, iteration)` method that:
  - [ ] Checks/increments a retry counter
  - [ ] Injects a "Is the task complete?" user message when a non-submit completion fires
  - [ ] Continues the loop when the task is not complete
- [ ] Wire non-submit completion paths (`max-tokens`, `error`, `aborted`, `stop`) through `handleNonSubmitCompletion()` instead of calling `finishRun()`
- [ ] Enforce minimum retry count of **10** before falling back to `finishRun()`
- [ ] Keep `finishRun()` only reachable via a real `submit_and_exit` tool result

## 2. Types & Config — `sdk/packages/shared/src/agents/types.ts` *(CLI)*

- [ ] Add to `completionPolicy`:
  - [ ] `strictSubmitOnly?: boolean` — only `submit_and_exit` completes a run
  - [ ] `maxCompletionRetries?: number` — default 10
  - [ ] `askCompletionConfirmation?: boolean` — ask user before accepting non-submit completions

## 3. Config Builder — `sdk/packages/core/src/runtime/config/agent-runtime-config-builder.ts` *(CLI)*

- [ ] Plumb the 3 new options through `buildRuntimeConfig()`
- [ ] Preserve backward compatibility (options default to `false`/`undefined`)

## 4. State Tracking — `AgentRuntimeState` *(CLI)*

- [ ] Add `completionRetries?: number`
- [ ] Add `lastNonSubmitCompletionReason?: string`
- [ ] Add `pendingCompletionConfirmation?: boolean`

## 5. Agent Handoff — spawn & delegate to other agents *(CLI)*

- [ ] Add `handoff_to_agent` tool support in the CLI agent runtime:
  - [ ] Accept a target `providerId` + `modelId` for the sub-agent
  - [ ] Accept a task description / context to delegate
  - [ ] Optionally accept a `role` or `preset` for the sub-agent
- [ ] Spawn sub-agent with the specified model (may differ from the parent's model)
- [ ] Stream sub-agent progress/result back to the parent agent
- [ ] Return sub-agent result as a tool result so the parent can continue
- [ ] Surface handoff events in the CLI TUI (e.g. "Handing off to {model}…")
- [ ] Add config options:
  - [ ] `enableAgentHandoff?: boolean` — enables the `handoff_to_agent` tool
  - [ ] `allowModelOverride?: boolean` — permits sub-agents to use a different model than the parent
- [ ] Add `handoffRetries?: number` and `handoffTimeoutMs?: number` to `AgentRuntimeState`

## 6. Tests — `sdk/packages/agents/src/agent-runtime.test.ts` *(CLI)*

- [ ] Test: only `submit_and_exit` completes the run
- [ ] Test: non-submit completion triggers "Is the task complete?" confirmation
- [ ] Test: loop continues when user says not complete
- [ ] Test: retry counter enforces minimum of 10 attempts
- [ ] Test: max-retry fallback still emits `run-finished`
- [ ] Test: backward compatibility when `strictSubmitOnly` is off
- [ ] Test: `handoff_to_agent` spawns sub-agent with specified model
- [ ] Test: sub-agent result is returned to parent as tool result
- [ ] Test: handoff failure increments retry counter and re-prompts
- [ ] Test: `allowModelOverride=false` rejects handoff to a different model

---

## Files touched

- [ ] `sdk/packages/agents/src/agent-runtime.ts` *(CLI)*
- [ ] `sdk/packages/shared/src/agents/types.ts` *(CLI)*
- [ ] `sdk/packages/core/src/runtime/config/agent-runtime-config-builder.ts` *(CLI)*
- [ ] `sdk/packages/agents/src/agent-runtime.test.ts` *(CLI)*

## Definition of done

- [ ] Agent only stops when the model calls `submit_and_exit` *(CLI)*
- [ ] Non-submit completions prompt the user; task continues if not complete *(CLI)*
- [ ] ≥10 retry attempts before any fallback completion *(CLI)*
- [ ] Agent can hand off tasks to another agent using a different model *(CLI)*
- [ ] All new unit tests pass; existing tests remain green *(CLI)*
- [ ] Config options are opt-in and backward compatible *(CLI)*
