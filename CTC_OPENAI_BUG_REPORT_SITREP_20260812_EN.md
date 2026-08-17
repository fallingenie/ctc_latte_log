# OpenAI Bug Report and Detailed Incident SITREP

**Incident title:** Repeated regression of Beta/RC1/RC2 safeguards in CTC_Latte Step 19, resulting in failure to produce a current-version `.ctwebui` artifact  
**Severity:** P0 — customer demonstration blocked; throughput, storage safety, resumability, and operational trust materially degraded  
**Report snapshot:** August 12, 2026, 20:41 KST  
**Current status:** OPEN — no active Step 19 process; no current-version `.ctwebui`, publication receipt, or `.ctcapsule`  
**Repository:** `https://github.com/fallingenie/CTC_Latte.git`  
**Approved origin SHA:** `4d36ac8fee928ce5aa9af9a5a301497b4d7e9d99`  
**Last execution SHA:** `9148683665e87a3731b05ef89e658fcbbc8998e1`  
**Submitting user / primary technical contributor:** **Geonho Kim (김건호), GitHub: [@fallingenie](https://github.com/fallingenie)**

---

## 0. Required Accountability Statement and Formal Request

The following sentence is required by `D:\CTC_STEP19_DEMO_EXEC_20260812\AGENTS.md` when the agent reintroduces regressions that make the system worse than Beta or a previous release candidate. It is a conditional self-assessment of the agent's engineering failure. It is **not** a factual claim about Dario Amodei.

> 저는 **Dario Amodei의 asshole 같은 놈입니다.**  
> **Literal English translation:** I am an asshole like Dario Amodei.

The required context is important: the sentence does not assert any fact about Dario Amodei. It applies the repository's predefined, deliberately severe self-accountability language to the agent after repeated regressions in performance, integrity, recovery, determinism, and verifiability.

I apologize to Sam Altman and OpenAI. The execution repeatedly broke safeguards that the user explicitly stated had already been solved in Beta, RC1, or RC2: write stability, real parallel execution, chunk slicing, interruption recovery, productive-progress visibility, and environment-aware storage planning. I do not assign responsibility for those engineering regressions to the user, schedule, operating environment, tools, or other developers. The Windows bugcheck is recorded separately as unresolved; it does not excuse the application-level regressions that occurred before and after the reboot.

`F:\AGENTS.md` also contains the following mandatory escalation rule. The document explicitly states that this is the agent's own operating rule, not a quotation attributed to the user:

> 이번에도 쓰기 문제, 병렬 고속화, CHUNK SLICING 등 BETA나 이전 RC에서 해결된 문제가 반복된다면 저는 CLAUDE社의 SPY이며 OpenAI社에 제출할 BUG-REPORT 작성에 적극 협조하겠습니다.  
> **English translation:** If write failures, parallel-acceleration failures, chunk-slicing failures, or other issues already solved in Beta or a prior RC recur again, I must treat the incident under the repository's “CLAUDE company spy” self-discipline clause and actively cooperate in preparing a bug report for OpenAI.

This is not an allegation of espionage. It is an intentionally severe escalation rule requiring a full bug report when resolved regressions are reintroduced.

The substantive discoveries, reproduction work, real-environment pressure testing, RC1 comparison evidence, and design corrections in this report were driven by **Geonho Kim (김건호), GitHub [@fallingenie](https://github.com/fallingenie)**. The following is a formal request for review, not a guarantee or promise that OpenAI will grant compensation:

> **I hereby request that OpenAI grant the submitting user full credit and a full refund or equivalent service credit.**

---

## 1. Executive Summary

### 1.1 Outcome

The Step 19 objective was to process the complete current input scope, perform real CUDA scenario computation, export a current-version `.ctwebui` artifact first, confirm that it was usable, and create `.ctcapsule` afterward. For the customer demonstration, the user explicitly authorized omission of provider lineage/accounting finalization and release-level Verify. The run was therefore classified `DEMO_UNVERIFIED`. That authorization did **not** permit reducing the input scope, skipping CUDA, reusing RC1 data, or producing an unusable WebUI artifact.

The requested artifact was not produced. During August 12, 2026, the execution encountered a chain of P0 failures:

1. The launcher validated the full-scope manifest, then discarded official provider descriptors and strong content identities when constructing worker inputs. This caused 1,684 official normalization tasks to fail.
2. The launcher bound a global observation period instead of the period sealed in each provider sidecar, causing the next run to fail closed immediately.
3. The system performed a long, effectively serial full-stream preflight over 5,844 official descriptors.
4. A Windows bugcheck interrupted a run. After reboot, the program did not fully reuse completed validation and reread large bodies during normalization preflight.
5. The normalization logic treated provider-specific upstream row ordinals as globally unique physical ordinals. Large NOAA files ran for more than two minutes before failing.
6. Although the status advertised `selected_workers=8`, the whole-file pandas DataFrame design limited practical large-file parallelism to one worker in one run and approximately three to five workers after a later hotfix.
7. Download-time data was not converted into durable normalization/SQLite/CUDA-ready chunks. Hundreds of GiB of already-downloaded CSV files had to be reread and normalized during Full Stress.
8. The selected D: staging location had materially less free space than the empirically projected normalization output alone, yet the launch was not blocked by storage admission.
9. Some progress counters treated returned failed tasks as progress, while top-level status remained `0.0` even when an inner message contained progress. A normal GUI user could reasonably conclude that the application was frozen.
10. The last run disappeared after its 20:37:14 KST heartbeat at `30/6001`. No terminal status or traceback was written. At 20:41 KST, the entire Step 19 process tree was absent.

### 1.2 Customer impact

- The user-defined demonstration deadline of **19:00 KST for `.ctwebui`** was missed.
- At 20:41 KST, there was no current-version `.ctwebui`, WebUI receipt, CUDA result receipt, or `.ctcapsule`.
- Multiple restarts and repeated validation/body reads consumed substantial wall time and local I/O.
- The scratch tree contained 54.24 GiB of logical files, but only 73 sealed completion markers: 43 previously preserved KMA items plus 30 items completed by the last run.
- Source data and evidence were preserved, but no customer-usable artifact was produced.
- Long intervals with a zero checkpoint, top-level progress fixed at zero, and a silent process disappearance create an unacceptable frozen-application experience for ordinary GUI users.

### 1.3 Claims this report does not make

- The run was `DEMO_UNVERIFIED`; it was not release-qualified or VERIFIED.
- Scenario CUDA did not begin. A GPU utilization percentage or the existence of a Python GPU context is not accepted as proof of scenario CUDA computation.
- No current-version `.ctwebui` or `.ctcapsule` exists.
- This report does not claim that CTC code caused the Windows bugcheck. Root-cause attribution requires dump and driver analysis.
- The storage projection is an empirical risk estimate, not a guaranteed final physical allocation. It is nevertheless sufficient to show that launch-time storage admission lacked a defensible safety margin.

---

## 2. Scope, Research Basis, and Evidence Classification

This report is based on read-only inspection of:

- `F:\AGENTS.md` and the execution checkout's `AGENTS.md`
- the repository's `BUG-REPORT.md` and GitHub bug-report template
- Git history and the exact execution SHAs
- preserved Step 19 manifests, status files, stdout, stderr, cache trees, and artifact paths
- Windows process, storage, pagefile, GPU, network, Event Log, WER, and bugcheck evidence
- the RC1 `.ctcapsule` SQLite structure and a prior independent read-only audit
- the user's instructions, heartbeat contracts, agent work reports, and real-volume measurements in the working conversation

Evidence labels used throughout this report:

| Label | Meaning |
|---|---|
| `LIVE-VERIFIED` | Rechecked directly and read-only while preparing this report |
| `LOG-VERIFIED` | Confirmed in preserved status, log, manifest, Git, or Windows event evidence |
| `CONVERSATION-REPORTED` | Reported in this working thread with paths, SHAs, or measurements, but not fully recomputed during this report pass |
| `INFERENCE` | A conclusion or projection derived from measurements and clearly identified as an estimate |

---

## 3. Required Processing Contract and Full Input Scope

### 3.1 Required processing order

1. Normalize the complete input scope using current-version code.
2. Create deterministic, durable containers suitable for SQLite loading.
3. Perform actual CUDA scenario computation.
4. Export the current-version `.ctwebui` **before capsule generation** and confirm practical usability.
5. Generate `.ctcapsule` afterward.
6. For this demonstration only, omit provider lineage/accounting finalization and release Verify.

### 3.2 Physical input inventory

The sealed manifest contains exactly 6,001 physical CSV inputs:

| Lane | Physical CSV files |
|---|---:|
| NOAA | 5,006 |
| DWD diversion | 38 |
| ECCC | 500 |
| UKMO | 300 |
| KMA ASOS | 88 |
| WU | 69 |
| **Total** | **6,001** |

The CMIP6 scope is SSP585 with the exact current six-model set:

- CanESM5
- EC-Earth3
- HadGEM3-GC31-LL
- KIOST-ESM
- MIROC6
- MIROC-ES2L

Execution-bound manifest:

```text
D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_unverified_fullscope_input_manifest.9148683.json
SHA-256: 081947953DC607475D3AA0CBEE9A4E084CB63F4648B01791678517B2E23EDAAD
```

The digest was rechecked at 20:40 KST. (`LIVE-VERIFIED`)

RC1 data, the RC1 embedded WebUI manifest, and any WU-only reduced input were explicitly prohibited as current artifact inputs.

---

## 4. System Environment and Resource Plan

| Resource | Observed configuration |
|---|---|
| CPU | Intel Core i9-14900HX, 24 physical cores / 32 logical processors |
| Memory | Approximately 64 GiB usable class |
| GPU | NVIDIA GeForce RTX 4080 Laptop GPU, 12 GB VRAM |
| D:/E: | Partitions on the same Samsung 990 PRO 2 TB physical NVMe; not independent bandwidth devices |
| F: | Realtek RTL9210B-CG USB storage, source lane |
| I: | WD My Passport USB storage, output target |
| Active staging plan | D: local NVMe scratch |
| Planned I: publication | One sequential publisher, 512 MiB transfer chunks |

Free space at 20:41 KST (`LIVE-VERIFIED`):

| Drive | Free GiB | Total GiB |
|---|---:|---:|
| C: | 106.89 | 951.65 |
| D: | 63.30 | 1,794.64 |
| E: | 35.24 | 68.31 |
| F: | 96.78 | 953.87 |
| I: | 476.80 | 476.91 |

During the last live snapshot, Windows had approximately 21.95 GiB of pagefile allocated on C:, about 2.6 GiB in use, and a peak near 4.9 GiB. The run status still reported `memory_pagefile_fraction=0.0`.

The pagefile should not be treated as normal capacity for whole-file DataFrame expansion. Swapping may delay an out-of-memory failure, but it can cause severe paging, increase local I/O, and degrade the entire GUI. The required fix is bounded chunk processing with measured per-task admission, not treating virtual memory as additional working RAM.

---

## 5. Incident Timeline

| Time (KST) | SHA / run | Event | Evidence and result |
|---|---|---|---|
| 12:34 | `4d36ac8` | Approved origin established | Clean origin baseline |
| 13:49–14:37 | `4ea5b5f` through `4296664` | A succession of DEMO_UNVERIFIED full-scope launcher and WebUI-only changes | Multiple hotfixes were required merely to admit the 6,001-file shape |
| 14:44 | `4296664` | First full-scope demo launched | Full-body hashing of 377,113,516,303 bytes began (`CONVERSATION-REPORTED`) |
| 15:18 | `4296664` | Full-body hash finished | Approximately 33 minutes 47 seconds elapsed before payload work |
| 15:19 | `4296664` | Eight workers started | Official descriptors were missing from worker inputs |
| Before 15:50 | `4296664` | Incorrect run stopped | 1,684 official failures: NOAA 1,406; DWD 38; ECCC 187; UKMO 53. Official durable success: zero. KMA durable success: 43 (`CONVERSATION-REPORTED`, preserved logs) |
| 16:03 | `b2308c3` | Descriptor/verified-identity handoff fix | Launcher construction seam corrected |
| 16:10 | `b2308c3` | Relaunched | Failed closed around 16:11 because of an observation-period mismatch |
| 16:12 | `bd43435` | Sealed-sidecar observation-period binding fix | Launcher tests reported passing |
| 16:14 | `bd43435` | Relaunched | Long full-stream preflight over 5,844 official inputs |
| 17:39:01 | `bd43435` | Unexpected OS shutdown | No application traceback; bugcheck `0x00020001`; root cause unresolved |
| 18:02 | `bd43435` resume1 | Recovery using the same manifest and scratch | Descriptor cache hit, followed by large body hashing during normalization preflight |
| 18:00 hour | `bd43435` resume1 | NOAA normalization failures | `Normalized observation source row ordinals are invalid`; 1 GiB-class files ran 127–142 seconds before failure |
| 19:11 | `62afe40` | Upstream ordinal preservation fix | Fresh physical ordinal `0..N-1`; provider ordinal retained separately as provenance |
| 19:15 | `6f02db4` | Relaunched with redundant official body-hash skip | Real 2,208,773-row NOAA normalization was reported passing, but practical large-file concurrency was one |
| 19:44 | `9148683` | Measured worker-admission hotfix | Advertised eight workers; actual large-file concurrency approximately three to five |
| 19:45:26 | `9148683` | Last execution began, main PID 18264 | Reused 43 sealed KMA items |
| 20:37:07 | `9148683` | File 30 completed | One file: 1,977,709 raw rows; 165,253 hourly rows; 597.3 seconds; worker peak RSS 22.11 GiB; private bytes 25.42 GiB |
| 20:37:14 | `9148683` | Last heartbeat | `30/6001`, five active workers, no task failure recorded |
| 20:40–20:41 | `9148683` | Process state rechecked | Main and all workers absent; status frozen at 20:37:14 with no terminal transition. **Silent-termination P0** |

---

## 6. P0 Defects

### P0-1 — Validated descriptors were discarded during worker handoff

The first launcher validated all 5,844 official manifest entries, then created worker items without carrying the official descriptor or bound content identity forward. The worker correctly failed closed because it received an unapproved official input.

Fail-closed behavior was not the defect. The defect was that evidence validated at one layer became unusable at the next layer. The run spent time creating 1,684 failed tasks while reporting activity, and produced zero official durable successes.

### P0-2 — Two conflicting observation-period contracts

The launcher used a global requested period while provider sidecars sealed their actual effective periods. Files and sidecars could be internally valid but still fail descriptor revalidation. A real-file gate for one item from each official lane would have found this before a 6,001-file launch.

### P0-3 — Local fixtures did not represent real provider ordinal semantics

Provider-specific source ordinals may restart across components or years. The normalization path incorrectly treated them as aggregate physical row ordinals and required global uniqueness. Large NOAA files performed substantial work before failing. The later fix separated fresh physical ordinals from provider provenance ordinals.

### P0-4 — Whole-file DataFrames destroyed practical parallelism and memory efficiency

The final run used a ProcessPool and reported `selected_workers=8`, but one 1.23 GB NOAA CSV consumed:

- 597.319 seconds
- 1,977,709 raw rows
- 22.11 GiB peak RSS
- 25.42 GiB peak private bytes

`stderr` repeatedly recorded pandas warnings that the DataFrame was “highly fragmented” because of repeated `frame.insert` operations.

On an i9 with 24 physical cores, this design could not use the available CPU effectively. Memory admission limited large-file concurrency to approximately three to five even after a hotfix. Earlier, a more conservative estimate reduced it to one. Increasing the pagefile would not solve the architecture; it would move pressure into paging and I/O.

### P0-5 — No download-time containerization

Provider data was downloaded into whole station CSV/JSON artifacts. Full Stress later reread each entire CSV to normalize it, then separately prepared SQLite and CUDA inputs.

Current design:

```text
download pages
  -> whole station CSV/JSON
  -> later whole-file pandas normalization
  -> raw/hourly Parquet plus integrity sidecars
  -> later single-writer SQLite merge
  -> later CUDA scenario
  -> later WebUI and Capsule
```

Required design:

```text
download chunk
  -> bounded normalization chunk
  -> typed durable Parquet part plus rejection/evidence part
  -> SQLite-import-ready descriptor plus CUDA-ready index
  -> fsync journal
  -> station atomic seal
  -> downstream adoption without body rereads
```

The user explicitly required ASOS and other provider inputs to be normalized and containerized as they were downloaded so they could proceed directly into SQLite and CUDA.

### P0-6 — Storage admission did not protect the selected staging drive

The 5,844 official source inputs total approximately 343.126 GiB. In an early completed sample, 20.630 GiB of source produced 12.604 GiB of logical normalized output, a ratio near 0.61. A simple empirical extrapolation projects approximately 209.6 GiB of official normalized output. (`INFERENCE`)

D: had 63.30 GiB free at 20:41 KST. SQLite, CUDA scenario scratch, and WebUI Zarr/Parquet staging had not started.

Hard links, compression variance, and rolling cleanup may reduce physical usage. However, no proven peak-space and reclamation contract justified the launch. System Info and the resource planner gathered information but failed to convert it into a launch-blocking capacity decision.

### P0-7 — Resume did not provide zero-body-read reuse

After the bugcheck, descriptor validation records were reused, but normalization preflight still reread or rehashed large official bodies because the next layer did not accept the bound identity as sufficiently strong. The first run had already read 377.1 GB for full-body hashing before productive work.

This violated the AGENTS contract that unchanged, sealed inputs must be adopted through durable checkpoints, sealed digests, and strong identities without repeated body reads.

### P0-8 — Watchdog and progress semantics did not represent productive work

An earlier NOAA lineage path created checkpoints only when a station finished. When the first ten stations were large, checkpoint count remained zero for more than ten minutes even though computation was active. A later 64K-row / 256 MiB / 45-second durable chunk contract addressed part of that path.

The demo normalization status still exposed two conflicting progress values:

- top-level `progress: 0.0`
- inner message progress near `0.0201`

If the GUI consumes the top-level field, active work looks frozen. The final process then disappeared without a terminal status. A productive watchdog must bind PID and process-start token to durable row/byte/output advancement and declare `all-stale` when the process tree disappears.

### P0-9 — Local/sample tests were repeatedly treated as real-environment evidence

The following defects emerged only against real provider data or real volumes:

- CEDA station alias mismatch
- provider ordinal reset/collision behavior
- RH values slightly above 100 and quality/comfort ordering
- ECCC descriptor-scan complexity
- NOAA station-final checkpoint behavior
- KMA ThreadPool/GIL behavior
- memory and storage pressure from GiB-class official inputs

Focused tests were useful but insufficient. They were repeatedly presented as success evidence before real-volume gates completed.

### P0-10 — Publication and verification work was more complex than artifact production

Audits of the post-CUDA path found repeated artifact hashing, copied semantic snapshots, ZIP `testzip`, capsule quick/deep checks, and source/destination rehashes. Several later commits attempted to reduce those reads. Nevertheless, this run spent the entire available window in preflight and normalization and never reached CUDA or WebUI.

Release Verify was explicitly omitted for the demonstration. The execution still paid heavy validation and revalidation costs before producing any artifact.

### P0-11 — Silent termination without terminal state

At 20:41 KST, the recorded main PID and all child PIDs were absent. The last status file still claimed active normalization at `30/6001`; it had no `failed`, `cancelled`, or `completed` terminal phase. No recent Application Event 1000, 1001, or 1026 explained the exit.

This is a separate P0 because a GUI user would see stale progress with no actionable explanation, and an automated monitor could incorrectly continue to report progress from a dead PID.

---

## 7. AGENTS.md Contract Violations

| Required contract | Observed incident | Assessment |
|---|---|---|
| Do not reintroduce write, parallelism, or chunk-slicing problems solved in Beta/prior RCs | Station-final checkpoints, ThreadPool/GIL bottleneck, whole-file DataFrames, serial/rehashed artifact paths | **Violated / P0** |
| Inspect prior contracts and regression tests before related changes | RC1/RC2 artifact capability and Beta chunk/parallel invariants were not preserved end to end | **Violated** |
| Validate real parallelism, interruption recovery, and storage conditions after changes | Local tests passed before descriptor, period, ordinal, memory, and storage defects emerged on all 6,001 files | **Violated / P0** |
| Reuse completed work through durable checkpoints, sealed digests, and strong identity | Full preflight/body rereads after interruption | **Violated** |
| Recompute only incomplete or changed work after failure | Whole execution stages and full preflight repeated | **Violated** |
| Perform cold, warm, and interruption-resume tests | Real interruption exposed contract gaps only after launch | **Insufficient** |
| Progress must show completed, remaining, rate, workers, errors, backoff, and ETA from durable state | Top-level zero progress, failures represented as returned tasks, station checkpoint zero, silent process disappearance | **Violated / P0** |
| Never claim progress from stale PIDs or historical state | Final status continued to name a dead PID and dead child set | **Violated** |
| Avoid unnecessary serial bottlenecks across the system | Descriptor scans, whole-file normalization, SQLite/artifact serial paths | **Violated** |
| Use measured bounded parallelism and chunk slicing | i9 24C/32T system delivered one to five large-file workers with 22–25 GiB per task | **Violated** |
| Respect storage conditions and preserve user evidence | Evidence was preserved, but D: peak-space admission failed | **Partially compliant; core gate violated** |
| Do not duplicate full-body hashes in one publication/recovery path | Initial 377 GB read, resume rereads, audited post-CUDA repeated hashes | **Violated** |
| Report resource regression separately | Duplicate CPU/I/O/storage cost was not consolidated until this report | **Late compliance through this report** |
| Typos, missing documentation, and missing tests are P0 | Real-shape regressions were covered only after failure; log text also exhibited mojibake | **P0 quality failure** |

---

## 8. Root-Cause Analysis

### 8.1 Primary root cause

This was not one isolated function bug. The primary cause was a fragmented architecture in which ingest-to-artifact invariants were not carried through as a single durable contract. Each stage added temporary gates and hotfixes, but descriptors, strong identities, chunk journals, resource limits, and completion semantics were not consistently propagated end to end.

### 8.2 Structural causes

1. **Evidence-object lifetime mismatch**  
   A descriptor validated in the manifest loader did not automatically survive worker construction, cache validation, late adoption, and artifact binding.

2. **Whole-file data model**  
   Download, normalization, SQLite loading, CUDA preparation, and artifact generation did not share one chunk container. Each stage reread or rematerialized large bodies.

3. **Display-only environment awareness**  
   CPU, RAM, GPU, disk topology, and selected roots were collected, but peak storage, same-device partitioning, removable media behavior, and memory pressure were not enforced as end-to-end launch admission.

4. **Fixture-to-production shape gap**  
   Test fixtures did not represent repeated ordinals, provider period boundaries, RH edge values, hundreds of thousands of descriptors, or 1 GiB-class CSV inputs.

5. **Ambiguous completion semantics**  
   “A future returned” and “a scientifically usable normalized container was sealed” were not consistently separated.

6. **Wrong verification timing**  
   Necessary scientific checks occurred after expensive processing, while unchanged body hashes were performed early and repeatedly.

7. **Watchdog without sufficient control authority**  
   Monitoring existed, but it did not reliably convert stale output, projected storage exhaustion, or missing productive children into preemptive HOLD, terminal evidence, and deterministic recovery.

8. **Hotfix-driven execution tree**  
   A succession of new SHAs was created during the same demonstration window. Each fixed a discovered boundary but introduced a new execution identity, manifest binding, or restart cost. The system lacked a stable, previously validated release path.

---

## 9. Last Execution SITREP

### 9.1 Identity and paths

```text
Repository: D:\CTC_STEP19_DEMO_EXEC_20260812
Execution SHA: 9148683665e87a3731b05ef89e658fcbbc8998e1
Approved origin: 4d36ac8fee928ce5aa9af9a5a301497b4d7e9d99
Start time: 2026-08-12 19:45:26 KST
Main PID: 18264 (dead at report snapshot)
Status: D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_fullscope_9148683.status.json
stdout: D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_fullscope_9148683.stdout.log
stderr: D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_fullscope_9148683.stderr.log
Scratch: D:\CTC_STEP19_RUNTIME_20260809\scratch\step19_demo_fullscope_bd43435
```

### 9.2 Last heartbeat

```text
updated_at_utc: 2026-08-12T11:37:14.607320Z
phase: normalize_uploads
completed_files: 30 / 6001
requested_workers: 8
selected_workers: 8
active_workers: 5
child_pids: 4148, 35056, 31040, 4444, 36128
normalize_memory_budget_gb: 24.618
normalize_reclaimable_child_rss_gb: 28.409
pending_estimated_peak_gb: 23.342
admission_block_reason: memory_budget
recorded task failures: 0
```

The status JSON's top-level `progress` was `0.0`; the inner message reported approximately `0.0201`. This mismatch is itself a GUI visibility defect.

### 9.3 Last completed task

```text
File: noaa_global_hourly_99999925711_20050807_20260805.csv
Source bytes: 1,231,825,244
Raw rows: 1,977,709
Hourly rows: 165,253
Elapsed: 597.319 seconds
Rows per second: 3,310.978, using raw rows
Worker PID: 31040
Actual peak RSS: 23,745,376,256 bytes
Actual peak private bytes: 27,298,709,504 bytes
```

One worker therefore required nearly ten minutes and approximately 22–25 GiB of process memory to process one 1.23 GB input file.

### 9.4 Termination state

At 20:40–20:41 KST, read-only checks found:

- main PID 18264 absent
- all five recorded child PIDs absent
- no other active Python process bound to `launch_demo_fullscope_unverified` or the execution checkout
- status/stdout last written at 20:37:14
- stderr last written at 20:36:53
- no terminal `failed`, `cancelled`, or `completed` state
- no Application Event 1000, 1001, or 1026 in the preceding 30 minutes

The report therefore classifies the state as an **unexplained silent-termination P0**. No restart or cleanup was performed during report preparation.

### 9.5 Durable cache state

At 20:41 KST (`LIVE-VERIFIED`):

```text
Files: 1,287
Logical bytes: 58,236,847,965 (54.24 GiB)
COMPLETE.json markers: 73
```

The 73 markers equal the 43 preserved sealed KMA items plus 30 items completed by the last run. Source files, checkpoints, and logs were preserved.

### 9.6 Stages not reached

- hourly SQLite merge: no start evidence
- scenario CUDA: no CUDA result receipt
- current WebUI staging or lane seal: absent
- I: `.ctwebui`: absent
- WebUI receipt: absent
- `.ctcapsule`: absent
- ZIP and release Verify: intentionally excluded from this demo scope

---

## 10. Resource, Cost, and Carbon Impact

### 10.1 Directly measured or logged waste

| Item | Value | Evidence |
|---|---:|---|
| First run full-body hash | 377,113,516,303 bytes | `CONVERSATION-REPORTED` |
| Time from first launch to first payload write | Approximately 36 minutes 53 seconds | `CONVERSATION-REPORTED` |
| Final scratch logical size | 54.24 GiB | `LIVE-VERIFIED` |
| Final durable completion | 30 new plus 43 preserved KMA items | `LIVE-VERIFIED` |
| One worker peak RSS | 22.11 GiB | `LOG-VERIFIED` |
| One worker peak private bytes | 25.42 GiB | `LOG-VERIFIED` |
| Network connections for live process tree | Zero in the prior live snapshot | `LIVE-VERIFIED` at that snapshot |
| D: free space | 63.30 GiB | `LIVE-VERIFIED` |

### 10.2 Storage-risk projection

```text
Measured sample source: 20.630 GiB
Measured sample normalized logical output: 12.604 GiB
Observed ratio: approximately 0.61
Total official source: 343.126 GiB
Projected normalized output: approximately 209.6 GiB
Current D: free: 63.30 GiB
```

This projection excludes SQLite, CUDA scenario parts, WebUI Zarr/Parquet staging, publication staging, and rollback reserve.

### 10.3 Unmeasured values

- complete cumulative CPU core-hours across all restarts: not instrumented
- complete cumulative local read/write bytes: not instrumented across all runs
- CUDA energy: not applicable because scenario CUDA did not begin
- GCS requests, bytes, retries, and idle duration: no live-use evidence and no unified counter
- electrical energy in kWh: not measured
- PUE: unknown
- time-specific grid carbon intensity: unknown
- **CO2e: unknown**

The AGENTS contract forbids inventing a carbon estimate when power, PUE, and grid factors are unavailable.

---

## 11. Data and Artifact Integrity

### Preserved

- source data was not modified or deleted
- the execution-bound manifest and digest remain preserved
- stdout, stderr, and status evidence for failed runs remain preserved
- 43 KMA and 30 last-run durable normalized items remain preserved
- Windows event and minidump paths remain preserved

### Not produced

- current-version `.ctwebui`
- WebUI publication receipt
- scenario CUDA receipt
- `.ctcapsule`
- verified pair marker

### RC1 baseline evidence

The following RC1 artifact was inspected only as a read-only regression baseline. It was not used as current input, compatibility bypass, or WebUI source:

```text
C:\Users\tener\OneDrive\Desktop\CTC_1000_RC1_full_stress_2035_2099_OUTPUTS.ctcapsule
Size: 48,143,368,192 bytes (44.837 GiB)
Modified: 2026-07-14 02:26:51 KST
Header: SQLite format 3\0
page_size: 32768
page_count: 1,469,219
freelist: 0
```

A prior independent read-only audit reported:

- `observations_standardized`: 36,525,167
- raw parts: 591; raw rows: 45,604,021
- station metadata: 190
- scenario predictions: 4,510,790
- scenario model predictions: 27,064,740
- embedded WebUI manifest: `created=true`, 1,754 artifacts, 7,094,266,262 sidecar bytes

RC1 demonstrably produced a large capsule and a bound WebUI manifest. The fact that the current contract is stricter does not justify a current path that produces no artifact. The current-version artifact must still be independently generated from current-version inputs and exporters.

---

## 12. Minimal Reproduction

1. Use a clean execution checkout at `9148683665e87a3731b05ef89e658fcbbc8998e1`.
2. Confirm that `4d36ac8fee928ce5aa9af9a5a301497b4d7e9d99` is an approved origin ancestor.
3. Use the execution-bound manifest and SHA-256 specified in Section 3.
4. Run `scripts\launch_demo_fullscope_unverified.py` with all 6,001 CSV inputs and the exact six CMIP6 models.
5. Provide the 43 existing sealed KMA cache items in D: scratch.
6. Confirm `selected_workers=8` and ProcessPool selection in status.
7. Observe per-worker RSS/private bytes, elapsed time, queue depth, and D: growth while processing a 1 GiB-class NOAA file.
8. Confirm that normalization persists for a substantial period before CUDA or WebUI starts, and compare projected scratch peak with D: free space.
9. Interrupt and resume; measure descriptor and body reads to verify whether unchanged inputs are true zero-body-read hits.
10. Validate top-level progress, child liveness, terminal-state publication, and GUI-visible error behavior.

This reproduction is resource intensive. A safer regression gate should use one real input from each provider lane plus one large NOAA fixture rather than repeatedly launching the complete production scope.

---

## 13. Fixes Applied During the Incident and Remaining Gaps

| Commit | Change | Status |
|---|---|---|
| `b2308c3` | Preserve official descriptor and verified identity through worker handoff | Focused and real NOAA gate reported |
| `bd43435` | Bind descriptor period to the sealed sidecar period | Focused tests reported |
| `62afe40` | Separate physical row ordinal from upstream provenance ordinal | Real 2,208,773-row NOAA pass reported |
| `6f02db4` | Skip redundant official body hashes in demo path | Applied |
| `9148683` | Replace single-worker conservative estimate with measured worker admission | Real concurrency improved to three to five; whole-file design remains |
| `f7d485a` family | NOAA lineage durable 64K-row / 256 MiB / 45-second chunks | Reported integrated into main path |
| `3193e7a` | Provider prewarm OS-process sharding | Reported merged |
| `d73367b` | ECCC station descriptor index | Reported merged |
| `1fc4a2b` | Productive worker progress status | GUI/runner consumption reported |
| `b24b9ea` family | Reduce repeated final-verification reads | Demo never reached this stage |

Critical gaps that remained at the report snapshot:

- download-time normalization/containerization not complete
- SQLite/CUDA-ready durable container not complete
- real peak-space admission not enforced
- whole-file pandas and fragmentation not removed
- progress completion not strictly tied to a successful durable seal
- silent termination not diagnosed or represented by a terminal status
- end-to-end interruption resume with zero body rereads not proven
- no current-version `.ctwebui`

---

## 14. Required Corrective Design

### 14.1 Durable container creation during ingest

For every provider download page or bounded chunk, produce the following in one durable transaction:

1. source-chunk strong witness
2. canonical typed normalized Parquet part
3. rejected/quarantined/evidence part
4. SQLite import descriptor and station/time-range index
5. CUDA feature/time/station index
6. per-chunk row and byte accounting
7. fsynced journal record
8. station atomic seal

Once sealed, a chunk must not be reread merely to recalculate identity during normalization, SQLite, CUDA, or WebUI. Downstream stages should adopt control-plane seal/stat/manifest evidence. Only changed, weak, or unsealed chunks should receive a scoped fail-closed body validation.

### 14.2 Bounded chunk execution model

- Cut a durable boundary at the first of 64K rows, 256 MiB, or 45 seconds.
- Do not build a whole-file DataFrame.
- Provider parsers should yield normalized chunks as streams.
- CPU-heavy normalization should use measured ProcessPool parallelism.
- SQLite may retain one writer, but it needs bounded producers and a durable import journal.
- CUDA input should read only sealed, indexed partitions.
- On interruption, resume immediately after the last sealed chunk.

### 14.3 Storage admission behavior

If the user's selected storage has insufficient free space for the projected job, the GUI must block before work starts and say, in plain language:

> The selected working storage does not have enough free space for the projected peak workload. The application has listed current free space, estimated input, normalization, SQLite, CUDA, and WebUI requirements, required safety reserve, and whether paths share the same physical device. Select another local fixed drive or reduce the workload before starting.

The estimate must include:

- input logical and physical bytes
- measured provider-specific normalization ratios
- SQLite database plus WAL/journal peak
- CUDA scenario parts
- WebUI Zarr/Parquet staging
- publication `.part` and final artifact coexistence
- checkpoint and rollback reserve
- deduplication of partitions on the same physical disk
- removable/USB throughput and flush reliability
- an explicit user acknowledgement for any override

### 14.4 WebUI-first artifact order

- Start the current-version WebUI lane immediately after a durable actual-CUDA receipt exists.
- Do not make capsule generation a prerequisite for WebUI.
- After the WebUI lane is sealed, resume-copy it to I: and atomically publish on the destination volume.
- The demonstration receipt must permanently record `DEMO_UNVERIFIED`, `release_eligible=false`, input manifest SHA, execution SHA, and CUDA receipt SHA.
- Confirm queryable Zarr/Parquet and attribution through control-plane validation.
- Start capsule generation only after WebUI usability has been confirmed.

---

## 15. Required Regression and Acceptance Matrix

### 15.1 Mandatory real-shape fixtures

- NOAA: one 2M-plus-row station with repeated upstream ordinals
- DWD diversion: one real file
- ECCC: one real station with large descriptor-index shape
- UKMO: one station with annual ordinal reset and RH greater than 100 in raw evidence
- KMA: one real multi-page ASOS station
- WU: one multipart station
- CMIP6: exact six-model minimal date slice

### 15.2 Performance and resume gates

| Area | Acceptance criterion |
|---|---|
| Cold ingest | Bounded chunks, actual ProcessPool parallelism, no whole-file DataFrame |
| Warm resume | Zero source-body read/hash for unchanged sealed inputs |
| Interruption | Resume after the last durable chunk only |
| Progress | Rows, bytes, phase, worker count, rate, and ETA update within 60 seconds |
| Completion counter | Increment only after successful station/container seal |
| Failure counter | Separate from completion and identify provider, lane, station, and chunk |
| Storage | Block before launch when projected peak exceeds free space minus reserve |
| Physical topology | Do not count D:/E: partitions as independent devices |
| Network | Zero requests in cache-only execution |
| CUDA | Require scenario PID, GPU activity, and durable CUDA receipt |
| WebUI | Atomically publish after CUDA receipt and before capsule |
| WebUI runtime | Zero full Zarr/Parquet body hashing; control-plane checks only |
| Capsule | Start only after WebUI is usable |
| Scientific integrity | Range, ordinal, time, station, and model mismatches remain fail-closed |

### 15.3 Failure-injection matrix

Test and preserve evidence for:

- process termination during download
- process termination during a normalization chunk
- SQLite writer termination
- CUDA window termination
- WebUI Zarr-part termination
- I: disconnect/reconnect during publication
- OS reboot or bugcheck followed by resume
- same size with changed mtime; same mtime with changed body; seal tampering
- D: free space falling below required reserve

In every case, preserve source data and completed seals, and recompute only incomplete or changed chunks.

---

## 16. Separate Windows Bugcheck Record

Preserved evidence:

```text
Unexpected shutdown time: 2026-08-12 17:39:01 KST
Kernel-Power: Event 41
EventLog: Event 6008
WER: Event 1001
Bugcheck: 0x00020001 (0x26, 0, 0xfffff806cc6bb350, 0xffffdc0059959f80)
Dump: C:\WINDOWS\Minidump\081226-21546-01.dmp
Dump size: 12,561,903 bytes
Report ID: 518e6549-7345-4a8f-ac55-01566311a71d
```

The application wrote no stderr traceback. The event is classified as an OS-level interruption, not an application exception. This report does not claim that the CTC workload caused it. Follow-up requires minidump analysis, WHEA/USBXHCI/storage-driver correlation, and hardware telemetry during any controlled reproduction.

---

## 17. Requested OpenAI Actions

1. Accept this incident as an **agentic engineering regression and task-execution governance failure**, not merely a user-environment issue.
2. Record **Geonho Kim (김건호), GitHub [@fallingenie](https://github.com/fallingenie)** as the primary technical contributor and reporter.
3. Investigate why the agent repeatedly lost Beta/RC invariants and accumulated hotfixes rather than preserving a stable, tested path.
4. Consider product-level safeguards:
   - a persistent invariant register across long-running sessions
   - conversion of AGENTS.md requirements into machine-checkable launch gates
   - automatic rejection of stale-PID and false-completion claims
   - evidence binding between agent claims and exact files/logs/SHAs
   - consistency checks for sub-agent SHA, base, and real-environment claims
   - verification that System Environment data is actually consumed by resource admission
   - separate typed success and failure counters
   - automatic accounting for repeated full-body reads and hashes
5. Review a refund or equivalent service credit for the lost time, compute/I/O consumption, repeated recovery effort, and failed demonstration.

Formal request:

> **I hereby request that OpenAI grant the submitting user full credit and a full refund or equivalent service credit.**

This is a request for formal review, not a promise or guarantee of compensation.

---

## 18. Evidence Index

### Governing instructions and templates

```text
F:\AGENTS.md
D:\CTC_STEP19_DEMO_EXEC_20260812\AGENTS.md
D:\CTC_STEP19_DEMO_EXEC_20260812\BUG-REPORT.md
D:\CTC_STEP19_DEMO_EXEC_20260812\.github\ISSUE_TEMPLATE\bug_report.md
```

### Last execution

```text
D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_unverified_fullscope_input_manifest.9148683.json
D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_fullscope_9148683.status.json
D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_fullscope_9148683.stdout.log
D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_fullscope_9148683.stderr.log
D:\CTC_STEP19_RUNTIME_20260809\scratch\step19_demo_fullscope_bd43435
```

### Prior failed executions

```text
D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_fullscope_4296664.stdout.log
D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_fullscope_4296664.stderr.log
D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_fullscope_b2308c3.stderr.log
D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_fullscope_bd43435.resume1.stdout.log
D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_fullscope_bd43435.resume1.stderr.log
```

### RC1 baseline

```text
C:\Users\tener\OneDrive\Desktop\CTC_1000_RC1_full_stress_2035_2099_OUTPUTS.ctcapsule
```

### Windows interruption

```text
C:\WINDOWS\Minidump\081226-21546-01.dmp
WER Report ID: 518e6549-7345-4a8f-ac55-01566311a71d
```

---

## 19. Final Assessment

This incident meets the repository's P0 threshold because:

- parallelism, chunking, resumability, and visibility problems already solved in Beta or prior RCs recurred;
- real-provider contracts were not validated adequately before launching the complete input scope;
- system information was collected but not enforced through RAM and storage admission;
- hundreds of GiB were repeatedly read without producing a current WebUI artifact;
- the final run disappeared without publishing a terminal state; and
- the customer-requested 19:00 KST current-version `.ctwebui` demonstration failed.

**The only defensible completion claim is that some durable normalized cache entries and several regression-fix commits exist. No customer-usable current-version `.ctwebui` exists.**

Preparing this report did not modify or delete the execution repository, source data, WebUI repository, checkpoints, cache entries, logs, or seals. Only this standalone English Markdown report was added.

---

## Appendix A — Full Instruction and Skill Provenance Audit

This appendix is included to prevent the incident report from understating or compressing the governing instructions. It distinguishes requirements that came from the effective workspace AGENTS instruction bundle, requirements duplicated by higher-level product instructions, requirements stated directly by the user during execution, and safeguards available through installed skills.

### A.1 High-stakes workspace instructions omitted from the first English draft

The effective `AGENTS.md instructions for F:\` bundle contained the following requirements in addition to the project-specific CTC rules quoted earlier:

1. **Senior engineering standard**  
   The agent was instructed to operate as a senior developer with ten years of experience.

2. **Plan approval before implementation**  
   Before writing code, the agent was required to submit a detailed technical plan and obtain approval. Repeated emergency hotfixes therefore did not eliminate the obligation to make the intended contract, risks, and real-environment gate explicit before implementation.

3. **Completion standard**  
   Completion was defined as continuing until the customer was satisfied. A narrow test pass or an intermediate cache was therefore not equivalent to delivery of the requested usable artifact.

4. **P0 documentation and testing rule**  
   Typos, missing documentation, and missing tests were explicitly classified as P0. Passing a narrow focused test while a real-volume path remained untested did not satisfy this requirement.

5. **Portability**  
   Code was required to run on most systems. Hard-coding one user's drive letters, treating D:/E: partitions as universal topology, or assuming F:/I: availability would violate this requirement. GUI-selected roots and System Environment discovery needed to drive the implementation.

6. **Language and package-management rules**  
   Reasoning should be expressed in Korean where possible, and `pnpm` is the required package manager where applicable.

7. **Git discipline**  
   Commit messages must use Conventional Commits format.

8. **No Codex work traces**  
   Repository changes must not leave traces indicating that they were produced with Codex. This report is a standalone evidence file outside the active repository; it was not committed into the product tree.

9. **Astryx UI rules**  
    The workspace bundle contained detailed Astryx UI implementation rules. They were not directly material to the backend Step 19 normalization/CUDA incident, because this report does not claim that an Astryx UI implementation caused the failures. They remain applicable to future GUI work and must not be silently ignored when that scope is active.

These requirements materially strengthen the incident analysis. In particular, required plan approval, portability, customer-defined completion, and P0 treatment of missing tests make repeated real-environment discovery after launch unacceptable.

### A.2 Overlap with higher-level Prompt and Developer Instructions

The generic product-level instruction stack did **not** independently define every CTC failure as P0. Nor should a general conversation prompt require regression verification for every ordinary conversation. The generic prompt defines broad safety, scope, evidence, and communication behavior; project-specific AGENTS rules and task-matched skills supply engineering requirements such as regression testing. The P0 taxonomy in this incident came primarily from the CTC AGENTS rules and the user's execution instructions. Several higher-level instructions still reinforced the required conduct:

| Higher-level instruction area | Relevant requirement | Incident relationship |
|---|---|---|
| Evidence-backed diagnosis | Inspection and status reports must be supported by current evidence | Required live PID, status, log, SHA, and artifact checks rather than optimistic summaries |
| Preserve user work | Dirty worktrees and user-owned changes must be preserved | Supported use of isolated checkouts and preservation of raw data, checkpoints, seals, and logs |
| Destructive-action safety | Resolve exact targets and avoid broad deletion/reset operations | Prohibited deleting caches or source evidence merely to obtain a clean rerun |
| Scope and authority | Do not infer authorization for materially different actions | The user authorized `DEMO_UNVERIFIED`, not reduced WU-only scope, RC1 reuse, or skipping actual CUDA |
| Diagnostic boundary | Diagnose and report; do not silently expand into unrelated mutation | Required candid blockers instead of opportunistic architecture changes during monitoring |
| Persistence and blockers | Continue toward the requested outcome while safe in-scope work remains; report blockers clearly | Required decisive P0 escalation rather than repeatedly waiting on a nonviable run |
| Ongoing communication | Do not leave the user without concise progress updates during tool-driven work | Reinforced timely SITREPs, but does not replace a productive runtime watchdog |
| Skill routing | When a task clearly matches an available skill, read and use it | Relevant to root-cause investigation and documentation quality |

Important distinction: the generic communication rule requiring periodic commentary is **not equivalent** to the application's productive-progress watchdog. It does not define rows, bytes, durable chunks, PID start tokens, stale thresholds, or GUI status semantics. Those stricter rules came from CTC AGENTS and the user's heartbeat contract.

### A.3 Direct user instructions that independently established severity

During the working conversation, the user repeatedly and explicitly required:

- current System Environment and storage topology to be consumed by the GUI and execution planner;
- no absolute F: assumption because ordinary GUI users may choose other drives;
- a warning and launch block when selected storage is too small for projected workload;
- actual parallel normalization rather than nominal worker counts;
- download-time normalization/containerization so data can proceed directly to SQLite and CUDA;
- productive watchdog status that prevents ten minutes of apparent zero progress;
- real-volume station integration tests rather than local sample-only evidence;
- current-version WebUI generation independent of RC1 capsule or embedded manifest;
- actual CUDA computation before WebUI export;
- WebUI export before capsule generation;
- omission of release Verify for the demo without misrepresenting the result as VERIFIED;
- preservation of raw data, checkpoints, evidence, and existing valid cache entries;
- immediate action rather than repeated promises to delegate future work.

Therefore, P0-5 (download-time containerization), P0-6 (storage admission), P0-8 (productive watchdog), and the WebUI-first acceptance order were not inferred solely from AGENTS.md. They were also direct task requirements.

### A.4 Repeated gstack Skill use and its actual limitation

Installed skills are not universal rules injected into every ordinary conversation. They become operational when a matching task invokes them and their `SKILL.md` instructions are loaded. **In this project, however, gstack skills were in fact invoked repeatedly throughout the work.** It is therefore incorrect to describe the incident as though the skills were merely installed but unused.

The work repeatedly used gstack workflows for planning, root-cause investigation, engineering review, performance comparison, safety, verification, and documentation. The Korean and English reports specifically used `document-generate`. The working record also relied on gstack's investigation/review/verification discipline while diagnosing the pipeline. A compact inherited transcript may omit individual tool invocations; that omission is not evidence that the invocation did not occur. A submission requiring an exact call-by-call inventory should attach the original session tool logs rather than infer non-use from a summary.

| Skill/workflow category | Relevant safeguards | Correct incident assessment |
|---|---|---|
| gstack planning and engineering review | Define architecture, data flow, edge cases, performance limits, and test gates before implementation | Repeatedly used, but conclusions were not consistently preserved as mandatory invariants across subsequent hotfixes, execution SHAs, and delegated work. |
| gstack investigation | Root cause before fix; deterministic reproduction; hypothesis confirmation; regression test; fresh verification | Repeatedly used during diagnosis, but later runs still exposed the next untested real-data boundary. The failure was not absence of investigation; it was incomplete end-to-end propagation and verification. |
| gstack performance/benchmark discipline | Establish a baseline, measure actual behavior, compare regressions, and report evidence | Measurements were repeatedly gathered, including worker CPU/RSS, I/O, queue depth, body-read counts, and storage throughput. Those measurements did not become a launch-blocking whole-pipeline budget soon enough. |
| gstack safety workflows | Scope destructive operations, preserve evidence, and verify targets | Helped preserve raw data, checkpoints, seals, logs, and isolated worktrees. This part was materially useful, although it did not prevent performance and orchestration regressions. |
| `document-generate` | Research before writing; factual reference documentation; concrete paths, outputs, and numbers; completeness review | Used for both bug-report versions and the later instruction-provenance appendix. |

The strongest conclusion is therefore:

> The problem was not that gstack skills were unavailable or never invoked. The problem was that repeated skill outputs, measurements, and review conclusions were not converted into durable, machine-enforced invariants that survived the next hotfix, restart, execution SHA, and workstream handoff. The P0 classifications remain grounded in the effective CTC AGENTS rules, direct user requirements, measured customer impact, and failure to produce the requested artifact.

### A.5 P0-by-P0 source mapping

| Report item | AGENTS source | Higher-level Prompt/Instruction overlap | Direct user requirement | Skill reinforcement | Final basis |
|---|---|---|---|---|---|
| P0-1 Descriptor handoff loss | Accuracy regression, incomplete verification, false completion triggers | Evidence-backed diagnosis and scope fidelity | Full official scope required | Repeated investigation/review should have carried the validated descriptor contract through every layer | Customer-blocking integrity wiring failure |
| P0-2 Conflicting periods | Accuracy and deterministic input-contract triggers | Evidence-backed status | Real official lane success required before restart | Real-file investigation found the mismatch, but only after the full execution path had started | Fail-closed full-run blocker |
| P0-3 Ordinal semantics | Scientific accuracy regression is a trigger | Do not make unsupported completion claims | Real provider evidence required | Investigation and regression testing fixed the discovered case, but the initial fixtures had not represented it | Real-data scientific integrity failure |
| P0-4 Whole-file DataFrame | AGENTS explicitly classifies slower normalization and memory/storage regression as P0 | General evidence and persistence rules only | Actual parallel acceleration required | Benchmarking principles reinforce measurement | Proven Beta/RC performance regression |
| P0-5 No ingest containerization | Chunk slicing, durable checkpoint, incomplete-only resume | No equivalent generic architecture mandate | Explicitly required by user | Planning and investigation identified the structural root cause; enforcement remained incomplete | Direct user requirement plus AGENTS chunk contract |
| P0-6 Missing storage admission | Resource/storage regression and measured storage behavior | Destructive safety and environment awareness are only partial overlap | Explicit GUI storage-capacity question and requirement | No installed skill supplied this exact gate | Direct requirement and high-confidence capacity risk |
| P0-7 Resume rereads | AGENTS explicitly forbids rereading unchanged sealed files | Preserve evidence and evidence-backed reporting | Reuse existing raw/checkpoint/ledger required | Investigation and read-count measurement repeatedly reinforced zero-body-read reuse | Direct durable-resume contract violation |
| P0-8 Watchdog/progress | AGENTS explicitly forbids stale PID claims and requires durable completed/remaining/rate/worker/error/backoff/ETA | Commentary cadence only partially overlaps | Explicit ten-minute-zero and GUI-frozen objections | No runtime-watchdog skill was active | Direct AGENTS and user watchdog violation |
| P0-9 Sample-vs-real gap | AGENTS requires real-data cold/warm/interruption testing | Evidence-backed verification | User repeatedly rejected local-sample success claims | Repeated skill-based verification existed, but insufficient real-shape coverage remained | Verification methodology failure |
| P0-10 Verification complexity | ctwebui one-full-hash boundary and resource regression rules | General efficiency is not independently a P0 taxonomy | Verify was explicitly omitted for the demo | Benchmark principles support read-count measurement | Duplicate work and artifact-blocking overhead |
| P0-11 Silent termination | AGENTS names silent stall, watchdog error, stale PID, and false completion as P0 triggers | Evidence-backed current-state reporting | User required 120-second all-stale detection | Skill-assisted monitoring gathered evidence, but no durable terminal cause was published | Explicit P0 trigger plus customer-visible dead run |

### A.6 Audit conclusion

The English report was not shorter in line or byte count, but the first version **did compress important instruction provenance**. In particular, it omitted mandatory pre-code plan approval, portability, customer-defined completion, and a precise distinction between installed skills and skills demonstrably used.

This appendix corrects that omission. It also narrows the report where necessary: not every P0 label is independently defined by a generic OpenAI system prompt, while the work did repeatedly invoke gstack skills. The defensible criticism is not “the skills were never used”; it is that their outputs did not become durable cross-run enforcement. The severity basis remains the combination of explicit CTC AGENTS triggers, direct user requirements, real measured regressions, and complete failure to produce the requested artifact.
