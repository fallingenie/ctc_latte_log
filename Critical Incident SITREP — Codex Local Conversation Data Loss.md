아래 영문 SITREP을 그대로 OpenAI Critical Problem 보고에 붙여 넣으면 됩니다.

---

# Critical Incident SITREP — Codex Local Conversation Data Loss

**Severity:** Critical / P0  
**Date:** 2026-08-04 KST  
**Environment:** Windows, Codex Desktop 26.727.51351  
**Current status:** HOLD — all recovery and filesystem modification activity has stopped.

## Summary

A Codex-generated cleanup command appears to have deleted the storage directory containing archived Codex conversation rollout files. The desktop app subsequently removed affected tasks from the visible list because their rollout paths were stale.

This is not merely a sidebar display problem. Local conversation payloads are missing, although thread IDs and title metadata remain.

## Impact

Confirmed affected primary tasks include:

- `CTC Latte Backend`  
  Thread ID: `019edd40-af67-76f1-ab85-2d198073a2c8`
- `CTC Latte WebUI Frontend`  
  Thread ID: `019f443c-20c3-76a0-9193-9d42995e0566`
- `Locate CTC_Latte_WebUI`  
  Thread ID: `019f5dce-f409-7753-9975-fa621013d859`

Snapshot evidence:

- `state_5.sqlite`: 1,121 thread records
- 1,090 records reference missing rollout files
- `local_thread_catalog`: only one row, with initial reconciliation incomplete
- Thread titles, IDs, prompt metadata, and pinned-thread metadata still exist
- Actual historical rollout JSONL files are unavailable

## Timeline

- **2026-08-03 21:24:44 KST:** A Codex task moved session files from `%USERPROFILE%\.codex\sessions\2026\06` and `2026\07` into `D:\CodexData\archived_sessions`.
- `%USERPROFILE%\.codex\archived_sessions` was a junction pointing to that directory.
- **2026-08-04 08:43:43 KST:** The Backend task executed a recursive cleanup command whose targets explicitly included `D:\CodexData`.
- The command used `Remove-Item -Recurse -Force`; later retries also included the same target.
- `D:\CodexData` is now absent, leaving the archive junction broken.
- **2026-08-04 15:45 KST:** Codex Desktop logged `stale rollout path` and `stale_db_path_dropped`, after which affected tasks disappeared from the list.
- Direct `thread/read` and unarchive attempts fail because no readable rollout exists.

## Suspected Root Cause

The cleanup safety check accepted any direct child of `D:\` whose name began with `ctc` or `codex`. It therefore treated `D:\CodexData` as disposable without detecting that it was the live backing store for `%USERPROFILE%\.codex\archived_sessions`.

A secondary problem is that Codex Desktop silently drops stale thread records instead of surfacing a recoverable “missing local history” state.

## Requested OpenAI Action

1. Preserve all server-side telemetry and conversation data associated with the listed thread IDs.
2. Determine whether the complete conversation histories can be restored or exported from server-side retention.
3. Investigate why a Codex-generated destructive command could delete an active Codex archive backing store without a critical warning.
4. Add protection for `.codex`, session roots, archive junction targets, and any path referenced by the active thread database.
5. Prevent stale histories from silently disappearing from the UI; show an explicit recovery/error state instead.
6. Escalate this as a potential Codex data-loss incident and provide an incident or ticket ID.

## Recovery Condition

The deleted source was on an NTFS NVMe SSD with TRIM enabled. Further writes to the source volume may reduce recovery probability. No additional recovery writes are currently being performed.