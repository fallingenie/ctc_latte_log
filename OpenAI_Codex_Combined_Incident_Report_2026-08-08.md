# Combined Incident Report: Runaway Sub-Agent Fan-Out, Unexpected Credit Consumption, and Stale Active-Agent State

**Report date:** 2026-08-08  
**Product:** OpenAI Codex Desktop  
**Environment:** Windows, local Codex Desktop task  
**Codex build observed in local session metadata:** `0.147.0-alpha.6.5`  
**Affected task ID:** `019fcb98-40b2-7d81-84f8-c3f7b3014efc`  
**Affected automation ID:** `ctc-full-stress-15-sitrep`  
**Severity:** High  
**Customer impact:** Unexpected large Credit consumption, misleading activity status, loss of control over background work, and substantial recovery time  
**Account/contact:** _To be completed by the customer before submission_

## 1. Executive Summary

This report covers two related Codex Desktop incidents that occurred during a long-running local engineering task:

1. A recurring 15-minute SITREP automation repeatedly woke the root Codex task. Those wake-ups caused the task to continue work by spawning sub-agents, some of which spawned further descendants. The lack of effective single-flight, deduplication, concurrency, depth, and budget controls resulted in a large agent tree and extensive model usage. The customer observed a sudden, large Credit depletion.
2. After crashes, usage-limit errors, interruptions, and application restarts, many terminated or never-fully-started sub-agents remained displayed as **Active**. Their elapsed timers continued increasing for hours or days even though many were no longer executing. The UI showed approximately 165–168 active sub-agents, while the live collaboration view initially exposed only a small recent subset.

The incidents are related but distinct. The large Credit consumption came from actual model calls made during the agent fan-out. The continuously increasing UI timers were stale wall-clock timers and do not, by themselves, prove continuing model usage.

The customer requests an OpenAI investigation of both the orchestration failure and the server-side usage ledger, including consideration of Credit remediation for unintended automated usage.

## 2. Intended Behavior

The automation was intended to perform one bounded action every 15 minutes:

- wake the root task;
- report current status and ETA;
- safely continue the existing Full Stress workflow;
- stop when the final artifacts were complete or when the user explicitly stopped the automation.

Expected safeguards included:

- no overlapping run when a prior scheduled run was still active;
- no duplicate continuation of the same work;
- no unbounded recursive sub-agent creation;
- a clear limit on agent count, depth, token usage, and Credit exposure;
- cancellation propagation to all descendants when the automation was stopped;
- accurate UI distinction between `running`, `pending`, `interrupted`, and `completed` states.

## 3. Actual Behavior

### 3.1 Incident A: Recurring automation amplified into a large agent tree

The recurring automation repeatedly woke the root task with an instruction to continue the Full Stress work and provide a SITREP. During these turns, the root task created sub-agents for investigation and parallel work. Some sub-agents created additional descendants.

Local forensic review identified:

| Observation | Locally observed value |
|---|---:|
| Sub-agent session files associated with the affected root task | 181 |
| Model-call telemetry events after sub-agent creation | 8,555 |
| Aggregated input tokens in those local events | 990,485,914 |
| Aggregated cached input tokens | 959,226,880 |
| Approximate non-cached input portion | 31,259,034 |
| Aggregated output tokens | 3,116,634 |

These figures are derived from local `last_token_usage` telemetry and are **not** a substitute for OpenAI's billing ledger. They should be reconciled against server-side request IDs, model pricing, cached-input treatment, failed-request accounting, and Credit postings.

The very large inherited conversation context materially amplified usage. Representative calls contained approximately 180,000–230,000 input tokens, most of which were reported as cached input. Even with cached-input discounts, many concurrent large-context calls can consume substantial Credits.

### 3.2 Important mechanism clarification

The local evidence does **not** show that the recurring heartbeat was directly broadcast to every already-created sub-agent. A scan for heartbeat user messages received after sub-agent creation found zero direct post-spawn heartbeat deliveries.

The supported mechanism is instead:

1. The heartbeat woke the root task.
2. The root task interpreted “continue the work” as authorization to perform more work.
3. The root task spawned or followed up with sub-agents.
4. Some descendants spawned further descendants.
5. Subsequent heartbeat runs repeated the continuation pattern without an effective global single-flight or deduplication boundary.

Heartbeat messages visible inside child session files were often inherited parent history copied into the child context, not direct scheduled messages sent to that child.

### 3.3 Incident B: Terminated agents remained Active with increasing timers

The Codex Desktop UI displayed approximately 165–168 sub-agents as Active. Entries showed elapsed times ranging from hours to multiple days. Examples included storage mapping, PnP devices, event logs, GitHub state checks, release audits, and NOAA reviews.

However, local inspection showed that many corresponding child turns were already `interrupted`, `completed`, or stuck at `pending_init`. The parent task retained a `started` activity record without a corresponding terminal event.

The apparent state model was therefore:

```text
parent transcript: sub-agent started, no terminal reconciliation
child transcript: interrupted/completed/pending_init
UI result: Active timer continues indefinitely
```

Opening an affected child task caused Codex Desktop to re-read its actual state and reduced the Active count. A bulk deep-link reconciliation reduced the displayed count from 165 to 1; the remaining entry was the cleanup agent itself and was expected to disappear when that cleanup task completed.

No repository, scientific dataset, local drive content, or conversation JSONL was deleted to achieve this cleanup.

## 4. Timeline

All timestamps below are UTC unless stated otherwise.

| Time window | Event |
|---|---|
| 2026-08-04 through 2026-08-08 | The 15-minute Full Stress SITREP automation repeatedly woke the root task. |
| During repeated wake-ups | The root task and descendants created a growing tree of sub-agents and performed extensive model/tool work. |
| During crashes, restarts, interruptions, and usage-limit failures | A number of parent-side terminal activity events were not recorded or reconciled. |
| 2026-08-08 | The customer noticed a sudden large Credit reduction and ordered all work to stop. |
| 2026-08-08 | The customer observed approximately 168, then 165, entries shown as Active with continuously increasing elapsed timers. |
| 2026-08-08 | Direct child-task navigation proved that stale entries could be reconciled individually. |
| 2026-08-08 | Bulk reconciliation reduced stale Active entries from 165 to the single cleanup task, which then terminated. |

## 5. Customer Impact

- A large and unexpected amount of paid or limited Credit was consumed.
- The customer could not reliably determine which agents were actually running.
- Statements that no sub-agents were active conflicted with the visible UI count.
- The customer had to stop the main engineering workflow and spend significant time investigating Codex itself.
- The stale Active list created concern that Credits were continuing to drain even after the automation was stopped.
- The incident reduced confidence in recurring-task safety, cancellation, usage controls, and activity reporting.
- No confirmed source-code or scientific-data loss occurred as a result of the cleanup.

## 6. Technical Assessment

### 6.1 Probable causes of the sudden Credit drop

The Credit loss likely appeared sudden because several effects combined:

1. **Concurrent fan-out:** Many large-context requests were executed concurrently or in closely spaced waves.
2. **Large inherited context:** Full or extensive parent history was copied into child contexts, causing very large cached and non-cached input usage per call.
3. **Recursive delegation:** Descendant agents added further calls beyond the root task's visible activity.
4. **In-flight completion:** Requests already accepted before the usage limit may have completed and posted usage afterward.
5. **Delayed or batched UI accounting:** The customer-facing Credit display may have reflected accumulated server-side usage in one update rather than decrementing smoothly per request.

Only OpenAI can confirm the last two items using server-side request and billing records.

### 6.2 Probable causes of the stale Active list

- A `started` event was persisted before dispatch or initialization completed.
- Terminal state was not propagated to the parent after interruption, crash, usage-limit failure, or restart.
- The frontend calculated Active status and elapsed time from the unresolved parent `started` event rather than the live executor or child terminal state.
- No automatic startup reconciliation repaired orphaned activity records.
- The available live-agent management view and the frontend activity list used different state sources.

## 7. Why This Should Be Treated as a Product Incident

The original user instruction requested periodic status reporting and safe continuation. It did not request unbounded agent multiplication or unlimited Credit expenditure.

The following safety boundaries were either absent, ineffective, or not exposed clearly enough:

- per-automation single-flight execution;
- prevention of overlapping scheduled runs;
- recursive spawn-depth limit;
- maximum descendant-agent count;
- per-run token/Credit budget;
- cancellation of all descendants;
- reconciliation of child terminal state after application restart;
- accurate real-time usage and activity reporting.

The combination transformed a routine periodic status automation into an unintended high-cost workload.

## 8. Requested OpenAI Investigation

Please investigate and provide the customer with:

1. A server-side usage ledger for the affected task and automation, grouped by timestamp, model, request, parent/child task, cached input, non-cached input, output, and Credit charge.
2. Confirmation of whether usage was posted in delayed batches and whether in-flight requests continued to incur charges after the limit was reached or the automation was stopped.
3. Confirmation of the maximum concurrent agent count and recursive spawn depth reached.
4. The reason cancellation or interruption did not reliably propagate to all descendants.
5. The reason child terminal state did not reconcile into the parent activity list.
6. A review of whether the observed Credit use qualifies for restoration or adjustment because it resulted from unintended automation amplification.
7. Identification of any known Codex Desktop issue associated with build `0.147.0-alpha.6.5` that could explain stale Active entries or timer persistence.

## 9. Requested Product Remediation

### Automation and orchestration

- Enforce single-flight execution by default for recurring task heartbeats.
- Assign every scheduled run an idempotency key and reject duplicate continuation of the same run.
- Prevent a heartbeat from spawning sub-agents unless explicitly enabled.
- Add configurable limits for agent count, recursion depth, concurrent model calls, tokens, and Credits.
- Stop scheduling new runs while a prior run or any descendant remains active.
- Propagate stop, delete-automation, usage-limit, and parent cancellation events to all descendants.

### Usage safety

- Provide a hard per-automation Credit cap that cannot be exceeded by in-flight descendants.
- Warn before a scheduled task creates or resumes multiple sub-agents.
- Show live usage broken down by root task and descendant.
- Distinguish cached and non-cached token charges in the Credit UI.
- Surface delayed or pending usage postings so a later sudden debit is understandable.

### Activity-state correctness

- Derive Active status from the live executor plus the child task's terminal state, not only a parent-side `started` record.
- Reconcile orphaned activity records automatically at application startup and after crashes.
- Display `pending_init`, `running`, `interrupted`, `failed`, and `completed` separately.
- Stop elapsed timers immediately when no live execution exists.
- Provide a reliable “Stop all descendants” and “Reconcile stale tasks” control.
- Ensure all agent-management surfaces use the same authoritative state store.

## 10. Reproduction Outline

The issue may be reproducible under the following conditions:

1. Create a recurring heartbeat for a long-running root task with instructions to report status and continue work.
2. Allow the root task to delegate to multiple sub-agents, including nested delegation.
3. Let additional heartbeat intervals occur before all prior work and descendants are terminally reconciled.
4. Interrupt the app, restart it, or encounter a usage-limit failure while descendants exist.
5. Compare the visible Active-agent list with actual child-task terminal states and live executors.
6. Observe whether opening individual child tasks causes the Active count to decrease.

This reproduction should be performed in a controlled internal environment with a strict usage cap.

## 11. Evidence Available

- Screenshot showing approximately 168 sub-agents displayed as Active with multi-hour or multi-day timers.
- Local Codex session JSONL files for 181 sub-agent sessions associated with the affected root task.
- Local per-call token telemetry used for the aggregate figures in this report.
- Representative child sessions showing inherited heartbeat history, later agent execution, and interrupted or usage-limit states.
- Before-and-after UI evidence showing Active-count reduction after child-state reconciliation.
- The affected root task ID and automation ID listed at the beginning of this report.

The customer can provide these artifacts privately to OpenAI Support. They may contain local paths, project details, or conversation content and should not be posted publicly without review and redaction.

## 12. Data Interpretation and Limitations

- The local token totals are telemetry aggregates, not a billing statement.
- Cached tokens may be charged differently from non-cached tokens.
- Local logs cannot prove the exact time at which Credits were debited from the account.
- Local logs cannot determine whether the Credit UI updated continuously or in a delayed batch.
- The 181 sub-agent sessions were created over the incident period and were not necessarily all executing simultaneously.
- The 165–168 Active count represented unresolved UI activity records, not 165–168 confirmed live model processes.

## 13. Requested Resolution

The customer requests:

1. Written confirmation of the root cause.
2. A server-side reconciliation of usage and Credit charges.
3. Credit restoration or adjustment for usage attributable to unintended agent fan-out, if confirmed.
4. Confirmation that the stale Active-state defect and recurring-agent amplification are tracked for remediation.
5. Guidance on a safe configuration for recurring SITREP tasks that guarantees single-flight execution, bounded delegation, and a hard Credit ceiling.

---

**Submission note:** Before sending this report, add the account email, approximate Credit balance before and after the incident, and attach the relevant screenshot. Share session JSONL files only through a private OpenAI Support channel.
