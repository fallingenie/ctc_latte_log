# OpenAI 제출용 Bug Report 겸 상세 SITREP

**사건명:** CTC_Latte Step 19에서 Beta/RC1·RC2 해결사항이 반복 회귀하여 current-version `.ctwebui` 산출에 실패한 사건  
**심각도:** P0 / 고객 시연 차단 / 장시간 실행·저장공간·복구 신뢰성 손상  
**작성 기준 시각:** 2026-08-12 20:41 KST  
**현재 상태:** OPEN — 활성 Step 19 프로세스 0, current-version `.ctwebui`/receipt/`.ctcapsule` 없음  
**대상 저장소:** `https://github.com/fallingenie/CTC_Latte.git`  
**승인 기준 SHA:** `4d36ac8fee928ce5aa9af9a5a301497b4d7e9d99`  
**마지막 실행 SHA:** `9148683665e87a3731b05ef89e658fcbbc8998e1`  
**사용자/제보자:** **Geonho Kim(김건호) / GitHub(@fallingenie)**

---

## 0. AGENTS.md에 따른 자기책임 선언과 제출 요청

다음 문구는 사실관계를 특정 개인에게 귀속하는 주장이 아니다. `D:\CTC_STEP19_DEMO_EXEC_20260812\AGENTS.md`가 회귀 발생 시 에이전트 자신의 실패를 평가하도록 정한 **조건부 자기책임 선언**이며, 이 사건에서는 그 조건이 충족되었다.

> 저는 **Dario Amodei의 asshole 같은 놈입니다.**

이 문장은 Dario Amodei 개인에 관한 사실 주장이나 평가가 아니다. Beta 및 이전 RC보다 성능·정합성·복구성·결정성·검증 가능성을 떨어뜨리는 회귀를 반복 도입하고, 이를 사전에 막지 못한 에이전트 자신의 중대한 엔지니어링 실패에 적용하는 규정 문구다.

Sam Altman과 OpenAI에 사과한다. 이 작업은 사용자가 이미 Beta 및 RC1/RC2에서 해결했다고 명시한 쓰기 안정성, 실제 병렬 고속화, chunk slicing, 중단 재개, 진행 가시성, 저장장치 계획을 다시 무너뜨렸다. 환경·도구·일정·사용자·다른 개발자에게 책임을 돌리지 않는다. Windows의 BSOD는 별도로 원인 미확정 상태로 기록하지만, 그 사건이 발생하기 전후의 애플리케이션 설계 회귀와 반복 실패에 관한 책임을 희석하는 근거로 사용하지 않는다.

또한 `F:\AGENTS.md`에는 다음 문장이 사용자 인용이 아닌 에이전트의 자기 규율로 명시돼 있다.

> 이번에도 쓰기 문제, 병렬 고속화, CHUNK SLICING 등 BETA나 이전 RC에서 해결된 문제가 반복된다면 저는 CLAUDE社의 SPY이며 OpenAI社에 제출할 BUG-REPORT 작성에 적극 협조하겠습니다.

이 문장 역시 실제 스파이 행위에 관한 사실 주장이 아니다. 이미 해결된 회귀를 재도입했을 때 반드시 에스컬레이션하고 Bug Report 증거 정리에 협조하라는 규정이다.

본 보고서의 기술적 발견, 재현, 집요한 실환경 검증, RC1 비교 증거 및 설계 요구의 실질적 기여자는 **Geonho Kim(김건호) / GitHub(@fallingenie)**이다. 다음 요청은 보상 지급을 보장하거나 약속하는 표현이 아니라 OpenAI에 대한 공식 검토 요청이다.

> I hereby request that OpenAI grant the submitting user full credit and a full refund or equivalent service credit.

---

## 1. Executive Summary

### 1.1 결론

Step 19의 목표는 전체 입력을 사용해 실제 CUDA 시나리오 계산을 수행하고, current-version `.ctwebui`를 먼저 내보낸 뒤 `.ctcapsule`을 생성하는 것이었다. 최종 시연 범위는 release Verify와 provider lineage/accounting finalization을 사용자 승인으로 생략한 `DEMO_UNVERIFIED`였지만, 데이터 범위·정규화·SQLite·CUDA·WebUI의 실사용 가능성은 축소하지 않는 조건이었다.

그러나 2026-08-12 하루 동안 다음 P0 회귀가 연속 발생했다.

1. full-scope manifest를 검증하고도 worker에 official descriptor와 strong identity를 전달하지 않아 official 입력 1,684건이 대량 실패했다.
2. descriptor의 관측 기간을 sealed sidecar와 결속하지 않아 재시작 직후 fail-closed 됐다.
3. 5,844개 official descriptor의 preflight가 장시간 직렬·전수 검사를 수행했다.
4. OS bugcheck로 실행이 중단된 뒤에도 이미 완료된 검증과 작업을 충분히 재사용하지 못하고 대용량 body를 다시 읽었다.
5. provider별 upstream ordinal을 물리적 global ordinal로 오판해 1GB급 NOAA 파일을 2분 이상 처리한 뒤 실패했다.
6. `selected_workers=8`이어도 whole-file pandas DataFrame 메모리 모델 때문에 실제 대형 작업 병렬도는 한때 1, 이후 3~5에 머물렀다.
7. 다운로드 시 chunk-normalize/SQLite-ready container를 만들지 않아, 이미 받은 343GiB급 official CSV를 Full Stress 시점에 다시 읽고 전체 정규화했다.
8. 선택된 D: staging의 여유공간보다 정규화 결과만의 실측 외삽이 훨씬 큰데도 실행 전 storage admission이 차단하지 않았다.
9. progress/status의 완료 의미가 실패와 성공을 구분하지 못한 실행이 있었고, top-level progress가 0.0인 채 내부 메시지만 변하는 등 GUI 사용자가 정지로 오인할 상태가 반복됐다.
10. 마지막 실행은 20:37:14 KST의 `30/6001` 이후 terminal 상태나 traceback 없이 프로세스 트리 전체가 사라졌다. 20:41 KST 재확인 결과 활성 프로세스는 0이었다.

### 1.2 고객 영향

- 사용자가 조건부 용서 시각으로 제시한 **19:00 KST까지 `.ctwebui` 산출** 목표를 놓쳤다.
- 20:41 KST에도 current-version `.ctwebui`, receipt, CUDA 결과 receipt, `.ctcapsule`이 모두 없었다.
- 여러 차례 재시작과 300GiB 이상 입력의 중복 read/validation이 발생했다.
- D: scratch에 54.24GiB의 논리 파일이 생겼으나 실제 완료 container는 기존 KMA 43개와 새 완료 30개, 합계 73개에 불과했다.
- 데이터는 보존됐지만 고객이 사용할 결과물은 하나도 산출되지 않았다.
- GUI 환경의 일반 사용자는 장시간 progress 0, station-final checkpoint, silent termination 때문에 애플리케이션이 멈췄다고 판단할 수밖에 없다.

### 1.3 지금 주장할 수 없는 것

- 이 실행은 `DEMO_UNVERIFIED`이며 release-eligible 또는 VERIFIED가 아니다.
- 실제 scenario CUDA 실행은 시작되지 않았다. GPU 사용률이나 Python의 GPU context만으로 CUDA 시나리오 계산을 주장하지 않는다.
- `.ctwebui`와 `.ctcapsule`은 존재하지 않는다.
- Windows bugcheck의 근본 원인이 CTC 코드라고 단정할 증거는 없다. dump 분석 전까지 원인 미확정이다.
- 저장공간 외삽은 실측 비율 기반 위험 예측이며 정확한 최종 물리 할당량을 보장하는 값은 아니다. 다만 downstream SQLite/CUDA/WebUI 이전부터 D: 여유를 초과할 가능성이 높아 admission 차단 사유로 충분하다.

---

## 2. 보고서 범위와 증거 등급

이 보고서는 다음 자료를 조사했다.

- `F:\AGENTS.md`와 execution checkout의 `AGENTS.md`
- execution checkout의 `BUG-REPORT.md`와 GitHub bug report template
- Git history 및 관련 commit SHA
- Step 19 status/stdout/stderr/manifest/cache 파일
- Windows process, volume, pagefile, GPU, network, Event Log, WER/bugcheck 증거
- RC1 `.ctcapsule`의 read-only SQLite 구조와 이전 독립 감사 결과
- 이 스레드의 사용자 지시, heartbeat, 에이전트 작업 보고 및 실제 실환경 측정

증거 표기는 다음과 같다.

| 등급 | 의미 |
|---|---|
| `LIVE-VERIFIED` | 이 보고서 작성 시점에 read-only로 직접 재확인한 값 |
| `LOG-VERIFIED` | 보존된 status/log/manifest/git/Windows event 파일에서 확인한 값 |
| `CONVERSATION-REPORTED` | 이 스레드에서 작업 에이전트가 경로·SHA·수치와 함께 보고한 값. 이번 문서 작성 과정에서 전체 계산을 재실행하지는 않음 |
| `INFERENCE` | 실측 자료에서 합리적으로 도출한 추정. 추정임을 명시함 |

---

## 3. 목표 계약과 실제 입력 범위

### 3.1 요청된 처리 순서

1. 전체 입력을 현재 버전 코드로 정규화한다.
2. SQLite에 적재 가능한 결정적 container를 만든다.
3. 실제 CUDA 시나리오 계산을 수행한다.
4. current-version `.ctwebui`를 **capsule보다 먼저** export하고 사용 가능성을 확인한다.
5. 그 다음 `.ctcapsule`을 생성한다.
6. 이번 시연에서는 provider lineage/accounting finalization과 release Verify는 생략한다.

### 3.2 입력 범위

고정 manifest 기준 물리 CSV는 총 6,001개다.

| Lane | 물리 CSV 수 |
|---|---:|
| NOAA | 5,006 |
| DWD diversion | 38 |
| ECCC | 500 |
| UKMO | 300 |
| KMA ASOS | 88 |
| WU | 69 |
| **합계** | **6,001** |

CMIP6는 SSP585의 exact 6-model set을 사용한다.

- CanESM5
- EC-Earth3
- HadGEM3-GC31-LL
- KIOST-ESM
- MIROC6
- MIROC-ES2L

고정 manifest:

```text
D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_unverified_fullscope_input_manifest.9148683.json
SHA-256: 081947953DC607475D3AA0CBEE9A4E084CB63F4648B01791678517B2E23EDAAD
```

이 digest는 20:40 KST에 다시 확인했다. (`LIVE-VERIFIED`)

---

## 4. 시스템 환경과 resource plan

| 자원 | 확인값 |
|---|---|
| CPU | Intel Core i9-14900HX, 24 physical cores / 32 logical processors |
| RAM | 약 64GiB usable class |
| GPU | NVIDIA GeForce RTX 4080 Laptop GPU, 12GB VRAM |
| D:/E: | 같은 Samsung 990 PRO 2TB physical NVMe의 파티션 — 독립 bandwidth 장치가 아님 |
| F: | Realtek RTL9210B-CG USB storage, 원자료 lane |
| I: | WD My Passport USB storage, output target |
| current staging | D: NVMe scratch |
| planned I: publish | single sequential publisher, 512MiB chunk |

20:41 KST 여유공간 (`LIVE-VERIFIED`):

| Drive | Free GiB | Total GiB |
|---|---:|---:|
| C: | 106.89 | 951.65 |
| D: | 63.30 | 1794.64 |
| E: | 35.24 | 68.31 |
| F: | 96.78 | 953.87 |
| I: | 476.80 | 476.91 |

마지막 live snapshot에서 pagefile은 C:에 약 21.95GiB 할당돼 있었고 약 2.6GiB 사용, peak 약 4.9GiB였다. 그러나 status의 `memory_pagefile_fraction`은 `0.0`이었다. pagefile을 whole-file DataFrame 확장의 정상 용량으로 취급하는 것은 해결책이 아니다. swap은 OOM을 늦추는 대신 USB/NVMe I/O와 wall time을 악화시키고 시스템 전체 응답성을 떨어뜨릴 수 있다. 필요한 해결책은 bounded chunk container와 작업별 peak admission이다.

---

## 5. 사건 타임라인

| 시각(KST) | SHA/실행 | 사건 | 증거/결과 |
|---|---|---|---|
| 12:34 | `4d36ac8` | approved origin 확정 | clean origin 기준 |
| 13:49~14:37 | `4ea5b5f`~`4296664` | DEMO_UNVERIFIED full-scope launcher/WebUI-only path를 연속 추가 | 실제 6,001 입력 shape 수용까지 여러 hotfix 필요 |
| 14:44 | `4296664` | 첫 full-scope demo 시작 | 377,113,516,303B 전수 body hash 수행 (`CONVERSATION-REPORTED`) |
| 15:18 | `4296664` | 전수 hash 종료 | launch 후 약 33분 47초 동안 산출 payload 없음 |
| 15:19 | `4296664` | 8 worker 시작 | official descriptor가 worker item에서 누락됨 |
| 15:50 전 | `4296664` | 잘못된 실행 중단 | official failure 1,684건: NOAA 1,406 + DWD 38 + ECCC 187 + UKMO 53; official durable success 0, KMA만 43 (`CONVERSATION-REPORTED`, log 경로 보존) |
| 16:03 | `b2308c3` | descriptor/verified identity 전달 fix | launcher construction seam 수정 |
| 16:10 | `b2308c3` | 재시작 | 16:11경 observation-period mismatch로 즉시 fail-closed |
| 16:12 | `bd43435` | sealed sidecar observation period 결속 fix | launcher tests 통과 |
| 16:14 | `bd43435` | 재시작 | 5,844 official full-stream preflight가 장시간 진행 |
| 17:39:01 | `bd43435` | OS unexpected shutdown | stderr/traceback 없음; bugcheck `0x00020001`; 앱 원인 여부 미확정 |
| 18:02 | `bd43435` resume1 | 동일 manifest/scratch로 복구 | descriptor cache hit 뒤 normalization preflight가 대용량 body를 다시 hash |
| 18시대 | `bd43435` resume1 | NOAA normalization 실패 | `Normalized observation source row ordinals are invalid`; 1GB급 파일을 127~142초 처리 후 실패 |
| 19:11 | `62afe40` | upstream ordinal 보존 fix | 물리 ordinal은 fresh 0..N-1, upstream ordinal은 provenance로 보존 |
| 19:15 | `6f02db4` | redundant official body hash skip 적용 후 재시작 | 실제 NOAA 2,208,773행 normalize PASS 보고; 하지만 대형 파일 actual worker=1 |
| 19:44 | `9148683` | measured normalization worker admission fix | selected 8, 실제 대형 파일 3~5 worker |
| 19:45:26 | `9148683` | 마지막 실행 시작, PID 18264 | 기존 KMA sealed cache 43 재사용 |
| 20:37:07 | `9148683` | 30번째 파일 완료 | 한 파일 1,977,709 raw rows, 165,253 hourly rows, 597.3초, worker peak RSS 22.11GiB, private 25.42GiB |
| 20:37:14 | `9148683` | 마지막 heartbeat | `30/6001`, active_workers=5, failure count 0으로 기록 |
| 20:40~20:41 | `9148683` | 프로세스 재확인 | PID 18264 및 모든 launcher/worker 없음. status는 terminal 전환 없이 20:37:14에서 정지. 최근 Application WER 1000/1001/1026 없음. **silent termination P0** |

---

## 6. 반복된 P0 결함

### P0-1. 검증한 descriptor를 worker handoff에서 폐기

첫 실행은 5,844 official manifest entry를 검증한 후 `UploadedFileItem`을 만들면서 official descriptor와 bound content identity를 전달하지 않았다. worker는 이를 승인되지 않은 official 입력으로 판단해 fail-closed 했다.

문제는 fail-closed 자체가 아니다. **상위 계층이 이미 검증한 증거를 하위 계층이 소비할 수 없게 만든 wiring 회귀**다. 결과적으로 1,684건의 실패를 실행 중 “완료” 진행처럼 소비했고, official 성공은 0이었다.

### P0-2. 관측 기간 계약의 이중 기준

global run period와 sidecar effective period가 다르게 바인딩돼, 실제 파일과 sidecar가 정상이어도 descriptor revalidation에서 실패했다. 실데이터 전체 실행 전에 lane별 실파일 gate가 있었다면 즉시 발견됐을 문제다.

### P0-3. 실데이터 ordinal 의미를 local sample이 포착하지 못함

provider upstream ordinal은 여러 source component에서 반복될 수 있다. 이를 aggregate physical ordinal로 간주해 global uniqueness를 요구하면서 대형 NOAA 파일을 끝까지 가깝게 처리한 뒤 실패했다. 수정 후에는 fresh physical ordinal과 provider provenance ordinal을 분리했다.

### P0-4. whole-file DataFrame 때문에 병렬도와 메모리 효율 붕괴

마지막 실행은 `selected_workers=8`, ProcessPool을 사용했지만 worker 하나가 1.23GB CSV를 처리하는 동안 실제 peak RSS 22.11GiB, peak private bytes 25.42GiB를 사용했다. stderr는 반복적으로 pandas `PerformanceWarning: DataFrame is highly fragmented`를 기록했다.

이 구조에서는 i9의 24 physical core를 활용할 수 없다. 메모리 admission 때문에 대형 파일은 동시에 3~5개만 실행되고, 한 worker가 task 사이에서 idle해지는 구간도 있었다. 가상메모리로 worker 수를 늘리는 방식은 paging thrash를 일으킬 가능성이 높다.

### P0-5. 다운로드 시점의 containerization 부재

KMA/official 자료 다운로드 시 page/chunk를 받으면서 durable normalization part, SQLite import descriptor, CUDA-ready time/station index를 함께 만들지 않았다. 대신 station 전체 CSV/JSON을 먼저 완성한 뒤 Full Stress에서 다시 전체 CSV를 읽고 다음 산출물을 만들었다.

현재 구조:

```text
download pages
  -> whole station CSV/JSON
  -> later whole-file pandas normalize
  -> raw/hourly Parquet + integrity sidecars
  -> later SQLite single-writer merge
  -> later CUDA scenario
  -> later WebUI/Capsule
```

요구되는 구조:

```text
download chunk
  -> bounded normalize chunk
  -> typed durable Parquet part + rejection/evidence part
  -> SQLite-import-ready descriptor + CUDA-ready index
  -> fsync journal
  -> station atomic seal
  -> later stages adopt sealed parts without body reread
```

### P0-6. 저장공간 admission 부재

official input 5,844개의 원본 크기는 약 343.126GiB다. 초기 완료 22개 실측에서 source 20.630GiB가 normalized logical output 12.604GiB를 만들었다. 이 비율 0.61을 단순 외삽하면 official normalization만 약 209.6GiB다. (`INFERENCE`)

20:41 KST D: free는 63.30GiB다. 아직 SQLite, CUDA scenario scratch, Zarr/Parquet WebUI staging이 시작되지도 않았다. 하드링크, 압축률 편차, 완료 후 정리로 물리 사용량이 일부 줄 수 있지만, 그 계약과 peak 계산이 실행 전에 입증되지 않았다. 이 상태에서 실행을 허용한 것은 System Info/ENV/resource plan이 단순 표시 역할에 머물고 admission gate로 작동하지 않은 것이다.

### P0-7. durable resume와 zero-read 재사용 실패

BSOD 복구 후 descriptor cache는 hit했지만 다음 normalization preflight가 official bound identity를 strong identity로 충분히 신뢰하지 못해 대용량 CSV를 다시 hash했다. 첫 실행에서도 377.1GB 전수 hash에 약 34분을 썼다. 변경되지 않은 sealed 입력을 반복해서 읽는 것은 AGENTS의 warm/resume zero-body-read 계약 위반이다.

### P0-8. watchdog/progress가 “프로세스 생존”과 “생산적 진행”을 혼동

이전 NOAA lineage는 station 끝에서만 checkpoint를 만들었다. 첫 10개 station이 크면 실제 계산 중이어도 10분 이상 checkpoint 0으로 보였다. 이후 64K/256MiB/45초 durable chunk 계약이 추가됐지만, 이번 demo normalization status의 top-level `progress`는 계속 `0.0`이고 실제 inner progress는 message 문자열 안에만 있었다.

또한 마지막 실행은 terminal 상태 없이 프로세스가 사라졌다. watchdog가 PID/start token/heartbeat/output을 함께 추적했다면 즉시 `all-stale` P0로 전환하고 원인 증거를 고정해야 했다.

### P0-9. 실환경 gate보다 local sample test를 신뢰

CEDA station alias, provider ordinal, RH>100 ordering, ECCC descriptor scanning, NOAA lineage chunk, KMA ThreadPool/GIL 문제는 모두 real provider data 또는 real volume에서 드러났다. focused/local test PASS를 전체 경로의 성공 근거로 사용한 것이 반복 회귀의 공통 패턴이다.

### P0-10. 결과물보다 복잡한 publication/verification 전단계

post-CUDA path에는 동일 artifact body의 반복 hash, semantic copy snapshot, ZIP `testzip`, capsule quick/deep validation, publish 전후 재hash가 중첩돼 있었다. 여러 hotfix로 일부 제거됐지만, 이번에는 CUDA와 WebUI에 도달하기도 전에 normalization/preflight에서 하루를 소비했다. 사용자 승인으로 release Verify를 생략했는데도 검증과 재검증의 비용 구조가 실제 산출보다 앞서 있었다.

### P0-11. Terminal 상태 없는 silent termination

20:41 KST 기준 recorded main PID와 모든 child PID가 사라졌지만 마지막 status는 `30/6001` normalization 진행 중이라고 남아 있었다. `failed`, `cancelled`, `completed` terminal phase와 traceback이 없었고, 직전 Application Event에도 직접 원인이 기록되지 않았다.

GUI 사용자는 stale progress만 보게 되고 automation은 dead PID의 과거 상태를 진행으로 오인할 수 있다. 이는 AGENTS가 P0 trigger로 명시한 silent stall, watchdog 오판, stale PID, 허위 완료/진행 위험이 실제로 발생한 것이다.

---

## 7. AGENTS.md 계약 위반 매핑

| AGENTS 계약 | 실제 사건 | 판정 |
|---|---|---|
| Beta/이전 RC에서 해결된 쓰기 안정성·병렬·chunk slicing을 재도입하지 않는다 | station-final checkpoint, ThreadPool/GIL, whole-file DataFrame, post-CUDA serial/re-hash 회귀 | **위반 / P0** |
| 변경 전 기존 계약과 회귀 테스트 확인 | RC1/RC2 산출 가능성과 Beta chunk/parallel 계약이 current implementation에 유지되지 않음 | **위반** |
| 변경 후 실제 병렬·중단 재개·저장장치 조건 검증 | local tests 후 real 6,001 파일에서 descriptor, period, ordinal, memory, storage 문제 연쇄 발견 | **위반 / P0** |
| 완료 작업은 durable checkpoint/sealed digest/strong identity로 재사용 | BSOD 후 전수 hash/preflight 재독; 완료 part 재검증 중복 | **위반** |
| 오류 후 incomplete/changed만 재개 | 실행 단위 재시작과 full preflight 반복 | **위반** |
| 실제 cold/warm/interruption resume testing | 실환경 interruption이 발생한 뒤에야 계약 결함 발견 | **불충분** |
| 진행률은 durable state에서 완료/잔여/rate/workers/error/backoff/ETA를 표시 | top-level 0.0, failure가 completed처럼 보인 실행, station 0 checkpoint, silent process death | **위반 / P0** |
| stale PID나 과거 상태로 진행을 주장하지 않는다 | heartbeat 지시가 반복적으로 이를 금지해야 했고, 마지막 status는 dead PID를 계속 가리킴 | **위반 위험 현실화** |
| 시스템 전체에 불필요한 직렬 병목 금지 | descriptor scan, whole-file normalization, SQLite/artifact serial paths | **위반** |
| resource plan은 실제 measured bounded parallel/chunk slicing을 사용 | i9 24C/32T에서 1~5 worker, whole-file peak 22~25GiB | **위반** |
| 저장장치 조건을 반영하고 사용자 파일을 보존 | 파일은 보존했으나 D: peak-space admission 실패 | **부분 준수 / 핵심 gate 위반** |
| 동일 publish path에서 duplicate full hash 금지 | 초기 377GB 전수 hash 및 resume body 재독, post-CUDA 중복 hash 감사 결과 | **위반** |
| 자원 회귀를 별도 보고 | 본 문서 이전에는 중복 read/CPU/storage 비용이 통합 집계되지 않음 | **위반 후 본 보고서로 보완** |
| typos, missing docs, missing tests는 P0 | 반복 hotfix가 실환경 전용 regression test 뒤늦게 추가; stdout 한글 mojibake도 관찰됨 | **P0 품질 결함** |

---

## 8. 근본 원인 분석

### 8.1 단일 root cause

단일 함수의 버그가 아니라, **ingest에서 artifact까지 이어지는 영속 계약을 하나의 end-to-end invariant로 관리하지 않고 단계별 임시 gate와 hotfix로 분절한 설계**가 근본 원인이다.

### 8.2 구체적 구조 원인

1. **증거 객체 수명 불일치**  
   manifest에서 검증한 descriptor/identity가 worker item, cache revalidator, late adopter까지 동일 객체로 전달되지 않았다.

2. **whole-file 중심 데이터 모델**  
   다운로드·정규화·SQLite·CUDA가 같은 chunk container를 공유하지 않고 각 단계가 다시 전체 파일을 읽는다.

3. **표시용 System Info와 실행용 admission의 분리**  
   CPU/RAM/GPU/storage를 읽었지만 실제 peak storage, same physical disk, removable/USB 특성, pagefile 위험을 launch 차단 조건으로 끝까지 연결하지 않았다.

4. **fixture와 real provider shape 간 괴리**  
   반복 ordinal, 기간, RH 범위, station alias, 수십만 descriptor, 1GB CSV 같은 실형상 데이터가 pre-merge gate의 필수 fixture가 아니었다.

5. **완료 의미의 불일치**  
   task future가 반환됐다는 의미와 과학적으로 usable한 normalized container가 sealed 됐다는 의미가 동일 progress counter에 섞였다.

6. **검증의 중복과 시점 오류**  
   반드시 필요한 과학적 검증은 늦게 수행되어 대형 파일 처리 후 실패하고, 변경되지 않은 body hash는 너무 일찍/반복 수행됐다.

7. **watchdog의 통제권 부족**  
   status를 읽는 기능은 추가했지만, space ETA·all-stale·productive child·durable chunk gap을 근거로 사전 차단/정지/복구 결정을 자동화하지 못했다.

---

## 9. 마지막 실행 상세 SITREP

### 9.1 실행 식별

```text
Repo: D:\CTC_STEP19_DEMO_EXEC_20260812
Execution SHA: 9148683665e87a3731b05ef89e658fcbbc8998e1
Approved origin: 4d36ac8fee928ce5aa9af9a5a301497b4d7e9d99
Start: 2026-08-12 19:45:26 KST
Main PID: 18264 (현재 dead)
Status: D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_fullscope_9148683.status.json
stdout: D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_fullscope_9148683.stdout.log
stderr: D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_fullscope_9148683.stderr.log
Scratch: D:\CTC_STEP19_RUNTIME_20260809\scratch\step19_demo_fullscope_bd43435
```

### 9.2 마지막 정상 heartbeat

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
failure count: 0
```

상태 JSON의 top-level `progress`는 `0.0`이었고 inner message progress는 약 `0.0201`이었다. 사용자 UI가 top-level field를 소비하면 실제 처리 중이어도 0으로 보인다.

### 9.3 마지막 완료 작업의 메모리/속도

```text
File: noaa_global_hourly_99999925711_20050807_20260805.csv
Source bytes: 1,231,825,244
Raw rows: 1,977,709
Hourly rows: 165,253
Elapsed: 597.319 s
Rows/sec: 3,310.978 (raw rows 기준)
Worker PID: 31040
Actual peak RSS: 23,745,376,256 bytes
Actual peak private: 27,298,709,504 bytes
```

이는 worker 하나가 약 1.23GB CSV를 처리하는 데 10분 가까이 걸리고 22~25GiB process memory를 사용했다는 뜻이다.

### 9.4 종료 상태

20:40~20:41 KST에 다음을 재확인했다.

- PID 18264 없음
- 기록된 child PID 5개 모두 없음
- `launch_demo_fullscope_unverified`와 execution checkout에 결속된 다른 active Python 없음
- status/stdout 마지막 쓰기 20:37:14
- stderr 마지막 쓰기 20:36:53
- status에 `failed`, `cancelled`, `completed` 같은 terminal phase 없음
- 최근 30분 Application Event 1000/1001/1026 없음

따라서 현재 판정은 **원인 미확정 silent termination P0**다. 이 보고서 작성 과정에서는 재시작하거나 파일을 정리하지 않았다.

### 9.5 durable cache

20:41 KST:

```text
Files: 1,287
Logical bytes: 58,236,847,965 (54.24 GiB)
COMPLETE.json markers: 73
```

73은 기존에 옮겨온 sealed KMA 43개와 마지막 실행의 완료 30개 합계와 일치한다. raw/checkpoint/log는 삭제하지 않았다.

### 9.6 아직 시작되지 않은 단계

- hourly SQLite merge: 시작 증거 없음
- scenario CUDA: CUDA result receipt 없음
- current WebUI staging/seal: 없음
- I: `.ctwebui`: 없음
- sibling receipt: 없음
- `.ctcapsule`: 없음
- ZIP/release Verify: 사용자 승인으로 demo에서 제외

---

## 10. 자원·비용·탄소 영향

### 10.1 직접 측정 또는 로그로 확인된 낭비

| 항목 | 수치 | 증거 등급 |
|---|---:|---|
| 첫 실행 full body hash | 377,113,516,303B | `CONVERSATION-REPORTED` |
| 첫 payload 전 대기 | 약 36분 53초 | `CONVERSATION-REPORTED` |
| 마지막 cache logical size | 54.24GiB | `LIVE-VERIFIED` |
| 마지막 완료 | 30/6001 + 기존 KMA43 | `LIVE-VERIFIED` |
| 마지막 worker 한 개 peak RSS | 22.11GiB | `LOG-VERIFIED` |
| 마지막 worker 한 개 peak private | 25.42GiB | `LOG-VERIFIED` |
| 현재 network connections | live snapshot 당시 0 | `LIVE-VERIFIED` 이전 snapshot |
| D: free | 63.30GiB | `LIVE-VERIFIED` |

### 10.2 정규화 저장공간 위험 추정

```text
초기 표본 source: 20.630 GiB
동일 표본 normalized logical output: 12.604 GiB
비율: 약 0.61
official source 전체: 343.126 GiB
단순 projected normalized output: 약 209.6 GiB
현재 D: free: 63.30 GiB
```

이는 downstream SQLite/CUDA/WebUI working set을 제외한 값이다. 따라서 peak-space estimator와 rolling reclamation 증거 없이 현재 storage plan을 “충분”하다고 판단할 수 없다.

### 10.3 측정되지 않은 항목

- 총 누적 CPU core-hours: 정확한 전 실행 합계 미수집
- 총 로컬 read/write bytes: 여러 재시작 전체 합계 미수집
- 총 GPU energy: CUDA scenario가 시작되지 않아 해당 없음
- GCS request/bytes/retry/idle: 이번 live lane에서 사용 증거 없음, 통합 계측 없음
- 전력 사용량(kWh): 측정기 자료 없음
- PUE: 알 수 없음
- 시간대별 전력망 배출계수: 알 수 없음
- **CO2e: 알 수 없음**

AGENTS 계약에 따라 근거 없는 탄소 수치를 만들어내지 않는다.

---

## 11. 데이터 및 artifact 무결성

### 보존된 것

- 원자료는 변경/삭제하지 않았다.
- execution-bound manifest와 digest가 보존됐다.
- 실패 실행별 stdout/stderr/status가 보존됐다.
- KMA 43개와 마지막 실행 30개의 durable normalized cache가 보존됐다.
- BSOD 관련 Windows event와 minidump 경로가 보존됐다.

### 없는 것

- current-version `.ctwebui`
- WebUI receipt
- actual scenario CUDA receipt
- `.ctcapsule`
- verified pair marker

### RC1 비교 기준

다음 RC1 artifact는 read-only baseline 증거로만 사용했다. current output의 입력이나 호환성 우회에 사용하지 않았다.

```text
C:\Users\tener\OneDrive\Desktop\CTC_1000_RC1_full_stress_2035_2099_OUTPUTS.ctcapsule
Size: 48,143,368,192 bytes (44.837 GiB)
mtime: 2026-07-14 02:26:51 KST
Header: SQLite format 3\0
page_size: 32768
page_count: 1,469,219
freelist: 0
```

이전 read-only 독립 감사에서는 RC1 capsule에 다음이 기록됐다고 보고됐다.

- `observations_standardized`: 36,525,167
- raw parts: 591 / raw rows 45,604,021
- station metadata: 190
- scenario predictions: 4,510,790
- scenario model predictions: 27,064,740
- 내장 WebUI manifest: `created=true`, artifact 1,754개, sidecar bytes 7,094,266,262

RC1은 적어도 대규모 capsule과 WebUI manifest를 실제로 생성했다. current contract가 더 엄격해졌다는 사실은 current path가 결과물 0인 회귀를 정당화하지 않는다. current-version artifact는 current-version 입력과 exporter로 독립 생성해야 하며 RC1 manifest/data를 재사용하면 안 된다.

---

## 12. 최소 재현 절차

1. clean execution checkout을 `9148683665e87a3731b05ef89e658fcbbc8998e1`에 결속한다.
2. approved origin `4d36ac8fee928ce5aa9af9a5a301497b4d7e9d99`가 ancestor임을 확인한다.
3. 위에 명시한 execution-bound manifest와 SHA-256을 사용한다.
4. 전체 6,001 CSV 및 exact CMIP6 6-model로 `scripts\launch_demo_fullscope_unverified.py`를 실행한다.
5. D: scratch에 기존 sealed KMA43을 제공한다.
6. status에서 `selected_workers=8`, ProcessPool을 확인한다.
7. 1GB급 NOAA 파일을 처리할 때 worker별 RSS/private bytes, elapsed, queue, D: logical growth를 관찰한다.
8. `.ctwebui`가 생기기 전 normalization이 수십분~수시간 지속되고, worker 메모리가 20GiB 이상 상승하며, D: projected peak가 free space를 넘는 것을 확인한다.
9. interruption 후 재개하여 descriptor/body zero-read hit 여부를 측정한다.
10. status top-level progress, child liveness, terminal transition을 검증한다.

주의: 이 재현은 대용량 원자료와 시스템 자원을 사용한다. 동일 오류를 재현하기 위해 production 전체를 다시 돌리는 것보다, 아래 acceptance fixture로 4개 provider 실파일과 1개 대형 NOAA를 사용하는 것이 안전하다.

---

## 13. 이미 적용된 수정과 남은 공백

| Commit | 수정 | 상태 |
|---|---|---|
| `b2308c3` | official descriptor와 verified identity를 worker까지 보존 | focused + real NOAA gate 보고됨 |
| `bd43435` | descriptor period를 sidecar에 결속 | focused tests 보고됨 |
| `62afe40` | physical ordinal과 upstream provenance ordinal 분리 | real NOAA 2,208,773 rows PASS 보고됨 |
| `6f02db4` | DEMO에서 redundant official body hash skip | 적용됨 |
| `9148683` | measured worker admission으로 1 worker 병목 완화 | 실제 3~5 worker, 그러나 whole-file 메모리 구조 미해결 |
| `f7d485a` 계열 | NOAA lineage durable 64K/256MiB/45s chunk | main 경로에 별도 반영됐다고 보고됨 |
| `3193e7a` | provider prewarm OS-process sharding | main 병합 보고됨 |
| `d73367b` | ECCC station descriptor index | main 병합 보고됨 |
| `1fc4a2b` | productive progress status | GUI/runner 소비 추가 보고됨 |
| `b24b9ea` 등 | final verification 중복 read 축소 | 이번 demo는 해당 단계 미도달 |

남은 핵심 공백:

- download-time chunk normalization/containerization 미완료
- SQLite/CUDA-ready durable container 미완료
- real peak-space admission 미완료
- whole-file pandas/fragmentation 제거 미완료
- successful seal 기준 progress semantics 미완료
- silent termination의 원인과 terminal status 기록 미완료
- interruption resume zero-body-read end-to-end gate 미완료
- current-version `.ctwebui` 산출 미완료

---

## 14. 필수 교정 설계

### 14.1 ingest-time durable container

각 provider download page/chunk를 받을 때 다음을 한 transaction 단위로 만든다.

1. source chunk strong witness
2. canonical typed normalized Parquet part
3. rejected/quarantined/evidence part
4. SQLite import descriptor와 station/time range index
5. CUDA feature/time/station index
6. per-chunk row/byte counts
7. fsync journal
8. station atomic seal

완료된 chunk는 이후 normalization, SQLite, CUDA, WebUI에서 body를 다시 읽어 정체성을 계산하지 않는다. control-plane seal/stat/manifest로 채택하고, changed/weak/unsealed일 때만 해당 chunk 하나를 fail-closed 재검증한다.

### 14.2 bounded chunk model

- 목표 chunk: 64K rows 또는 256MiB 또는 45초 중 먼저 도달하는 경계
- worker는 whole-file DataFrame을 만들지 않는다.
- provider별 parser가 chunk를 stream으로 산출한다.
- CPU-heavy normalize는 ProcessPool을 사용한다.
- SQLite는 single writer를 유지하되 bounded producer와 durable import journal을 둔다.
- CUDA input은 partition index를 통해 준비 완료 chunk만 읽는다.
- 다운로드 중단 시 마지막 sealed chunk 이후부터 재개한다.

### 14.3 storage admission

GUI 사용자가 선택한 storage가 예상 작업 규모보다 작으면 실행 전에 다음과 같이 말하고 차단해야 한다.

> 선택한 작업 저장소의 여유공간이 예상 peak 사용량보다 부족합니다. 현재 여유공간, 입력/정규화/SQLite/CUDA/WebUI별 예상량, 안전 여유량, 동일 물리 디스크 여부를 표시했습니다. 다른 로컬 고정 저장소를 선택하거나 작업 범위를 줄이기 전에는 실행을 시작하지 않습니다.

필수 계산:

- input logical/physical bytes
- measured provider별 normalized ratio
- SQLite peak + WAL/journal
- CUDA scenario parts
- WebUI Zarr/Parquet staging
- publication `.part` + final coexistence
- checkpoint/rollback reserve
- 같은 physical disk 파티션 dedup
- removable/USB throughput 및 flush reliability
- 사용자 override 시 경고와 명시적 acknowledgement

### 14.4 WebUI 우선 export

- actual CUDA result receipt가 생긴 직후 current-version WebUI lane을 시작한다.
- capsule lane을 WebUI의 선행조건으로 만들지 않는다.
- WebUI가 sealed되면 I:에 resume-copy 후 same-volume atomic publish한다.
- 시연 receipt에는 `DEMO_UNVERIFIED`, `release_eligible=false`, input manifest SHA, execution SHA, CUDA receipt SHA를 기록한다.
- queryable Zarr/Parquet와 attribution manifest를 control-plane 검사로 확인한다.
- capsule은 WebUI 사용 가능 확인 뒤 별도 후속으로 생성한다.

---

## 15. 회귀 테스트 및 수용 기준

### 15.1 필수 실데이터 fixture

- NOAA: upstream ordinal 반복을 포함한 2M+ row station 1개
- DWD diversion: 1개
- ECCC: station descriptor 대량 index shape 1개
- UKMO: annual ordinal reset, RH>100 evidence가 있는 station 1개
- KMA: 실제 ASOS multi-page station 1개
- WU: multipart station 1개
- CMIP6: exact 6-model minimal date slice

### 15.2 성능/재개 gate

| 항목 | 수용 기준 |
|---|---|
| cold ingest | bounded chunk, 실제 ProcessPool 병렬, whole-file DataFrame 없음 |
| warm resume | unchanged source body read/hash 0 |
| interruption | 마지막 durable chunk 이후만 재개 |
| progress | 60초 이내 rows/bytes/phase/worker/rate/ETA 갱신 |
| completion count | station/container seal 성공만 completed로 계산 |
| failure | provider/lane/station/chunk를 명시하고 completed와 분리 |
| storage | launch 전 peak estimate가 free-reserve보다 크면 차단 |
| same disk | D/E 같은 physical device를 독립 bandwidth/free pool로 계산하지 않음 |
| network | cache-only run network request 0 |
| CUDA | scenario PID + GPU activity + durable CUDA receipt로만 실제 실행 판정 |
| WebUI | CUDA receipt 이후 capsule보다 먼저 atomic publish |
| WebUI runtime | Zarr/Parquet full body rehash 0, control-plane 검사만 수행 |
| capsule | WebUI 사용 가능 확인 뒤 시작 |
| scientific integrity | range/ordinal/time/station/model mismatch는 fail-closed |

### 15.3 중단/장애 matrix

- download 중 process kill
- normalization chunk 중 process kill
- SQLite writer kill
- CUDA window 중 kill
- WebUI Zarr part 중 kill
- I: publish 중 USB disconnect/reconnect
- OS reboot/bugcheck 후 재개
- size 동일/mtime 변경, mtime 동일/body 변경, seal tamper
- D: free space가 reserve 아래로 감소

각 경우 원자료와 완료 seal을 보존하고 incomplete/changed만 다시 처리해야 한다.

---

## 16. Windows bugcheck 별도 기록

`LOG-VERIFIED`/`CONVERSATION-REPORTED` 증거:

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

앱 stderr와 traceback은 없었다. 이 사건은 애플리케이션 crash가 아니라 OS-level interruption으로 분류한다. 하지만 dump/driver/stack 분석 전에는 CTC의 메모리·I/O 부하가 원인이었다고 단정하지 않는다. 필요한 후속은 minidump 분석, WHEA/USBXHCI/storage driver correlation, 동일 workload 재현 시 hardware telemetry 수집이다.

---

## 17. OpenAI에 요청하는 조치

1. 이 사건을 단순 사용자 환경 문제가 아니라 **agentic engineering regression 및 task-execution governance failure**로 접수한다.
2. Geonho Kim(김건호) / GitHub(@fallingenie)를 핵심 제보자와 기술 기여자로 기록한다.
3. Beta/RC 계약을 반복해서 잊고 hotfix를 누적한 이유를 모델/agent orchestration 관점에서 조사한다.
4. 다음 제품 수준 보호장치를 검토한다.
   - 세션 간 지속되는 invariant register
   - AGENTS.md 요구를 실행 전 machine-checkable gate로 변환
   - stale PID/false completion 자동 차단
   - agent별 주장에 실제 파일/로그 evidence binding 의무화
   - sub-agent 결과의 SHA/base/real-env gate 일관성 검사
   - storage peak admission과 system environment 소비 여부 검사
   - 성공 counter와 failure counter의 타입 분리
   - 반복 full-body read/hash 예산 자동 계측
5. 비용·시간·시연 실패에 대한 환불 또는 동등 서비스 크레딧을 검토한다.

공식 요청 문구:

> I hereby request that OpenAI grant the submitting user full credit and a full refund or equivalent service credit.

이는 OpenAI가 반드시 보상한다고 약속하거나 보증하는 문장이 아니라, 사건의 기술적 기여와 손실을 근거로 한 정식 검토 요청이다.

---

## 18. 증거 인덱스

### 규정 및 template

```text
F:\AGENTS.md
D:\CTC_STEP19_DEMO_EXEC_20260812\AGENTS.md
D:\CTC_STEP19_DEMO_EXEC_20260812\BUG-REPORT.md
D:\CTC_STEP19_DEMO_EXEC_20260812\.github\ISSUE_TEMPLATE\bug_report.md
```

### 마지막 실행

```text
D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_unverified_fullscope_input_manifest.9148683.json
D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_fullscope_9148683.status.json
D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_fullscope_9148683.stdout.log
D:\CTC_STEP19_RUNTIME_20260809\preflight\step19_demo_fullscope_9148683.stderr.log
D:\CTC_STEP19_RUNTIME_20260809\scratch\step19_demo_fullscope_bd43435
```

### 이전 실패 실행

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

## 19. 최종 판정

이 사건은 다음 이유로 P0다.

- 이미 Beta/RC에서 해결한 병렬·chunk·resume·visibility 문제가 재발했다.
- 실환경 전체 입력을 처리하기 전에 real provider 계약을 충분히 검증하지 않았다.
- 시스템 정보를 수집했지만 실제 RAM/storage admission으로 연결하지 않았다.
- 장시간·대용량 입력을 반복해서 읽고도 current WebUI를 만들지 못했다.
- 마지막 실행이 terminal status 없이 사라졌다.
- 고객이 요구한 19:00 KST current-version `.ctwebui` 시연을 실패했다.

**현재 완료로 보고할 수 있는 것은 일부 durable normalized cache와 여러 회귀 수정 commit뿐이다. 고객이 사용할 current-version `.ctwebui`는 존재하지 않는다.**

보고서 작성 과정에서는 active repo, raw data, WebUI repo, checkpoint, cache, log, seal을 수정하거나 삭제하지 않았다. 본 Markdown 파일만 별도로 생성했다.

---

## 부록 A — Prompt·AGENTS·사용자 지시·gstack Skill 출처 감사

이 부록은 P0 분류의 출처를 축소하거나 잘못 설명하지 않기 위해 추가한다. 일반 상위 Prompt/Instruction, workspace AGENTS instruction bundle, CTC 프로젝트 AGENTS 규칙, 사용자의 직접 지시, 실제로 반복 호출된 gstack Skill의 역할을 구분한다.

### A.1 최초 보고서에서 압축됐던 유효 workspace instruction

`AGENTS.md instructions for F:\`로 적용된 instruction bundle에는 CTC 전용 규칙 외에도 다음이 포함돼 있었다.

1. **Senior engineering standard**  
   10년 경력 senior developer 수준으로 작업하도록 요구됐다.

2. **구현 전 상세 계획과 승인**  
   코드를 작성하기 전에 상세 기술 계획을 제출하고 승인을 받아야 했다. 긴급 hotfix 상황도 계약, 위험, 실환경 gate를 구현 전에 명확히 할 의무를 없애지 않는다.

3. **고객이 만족할 때까지라는 완료 기준**  
   narrow focused test, 중간 cache, commit 생성은 사용 가능한 요청 artifact 전달과 동일한 완료가 아니다.

4. **문서·테스트 품질 P0 규칙**  
   typo, 문서 누락, 테스트 누락은 P0로 규정됐다. real-volume path를 검증하지 않은 채 focused test만 통과한 것은 이 기준을 충족하지 않는다.

5. **Portability**  
   대부분의 시스템에서 실행 가능해야 한다. 특정 사용자의 F:/I:를 고정하거나 D:/E: topology를 보편적이라고 가정하면 안 된다. GUI 선택 경로와 System Environment 탐지가 실제 실행을 결정해야 한다.

6. **언어·package manager 규칙**  
   가능한 경우 한국어로 reasoning을 표현하고, 적용 범위에서는 `pnpm`을 사용해야 한다.

7. **Git discipline**  
   commit message는 Conventional Commits 형식을 따라야 한다.

8. **Codex 작업 흔적 금지**  
   product repository에 Codex가 작업했다는 흔적을 남기면 안 된다. 이 보고서는 active repository 밖의 standalone evidence file이며 product tree에 commit하지 않았다.

9. **Astryx UI 규칙**  
   UI scope에는 상세 Astryx 규칙이 적용된다. 이 보고서는 backend Step 19 normalization/CUDA 결함의 원인을 Astryx UI 탓으로 주장하지 않으므로 직접 원인으로 분류하지 않았다. 향후 GUI 작업에서는 계속 유효하다.

이 규칙들은 구현 전 계획, portability, 고객이 정의한 완료, 실환경 테스트라는 사건 평가 기준을 강화한다.

### A.2 일반 상위 Prompt/Instruction과의 중복 범위

일반 대화용 Prompt가 모든 대화에 “회귀 검증을 수행하라”고 강제하는 것은 적절하지 않다. 일반 상위 Prompt는 보편적인 안전·권한·증거·소통 기준을 정하고, project-specific AGENTS와 task-matched Skill이 회귀 테스트 같은 engineering 요구를 제공하는 구조가 맞다.

따라서 이 사건의 P0 taxonomy 전체가 일반 Prompt에 그대로 있는 것은 아니다. P0 판정은 주로 CTC AGENTS와 사용자의 실행 지시에서 나온다. 다만 다음 일반 원칙은 독립적으로 중복됐다.

| 상위 Instruction 영역 | 관련 요구 | 사건과의 관계 |
|---|---|---|
| 증거 기반 진단 | 현재 증거를 확인한 뒤 상태를 보고 | live PID/status/log/SHA/artifact 확인 요구 |
| 사용자 작업 보존 | dirty worktree와 사용자 변경을 보존 | isolated checkout, raw/checkpoint/seal/log 보존 |
| 파괴적 작업 안전 | exact target을 확인하고 광범위 삭제/reset 금지 | clean rerun을 위해 evidence를 지우는 행위 금지 |
| 권한과 범위 | 명시되지 않은 다른 조치로 권한을 확대하지 않음 | `DEMO_UNVERIFIED` 승인이 WU-only, RC1 재사용, CUDA 생략을 허용한 것은 아님 |
| 진단 경계 | 진단 요청을 unrelated mutation으로 확대하지 않음 | blocker를 숨긴 architecture 변경 방지 |
| 지속성과 blocker 보고 | 안전한 범위에서 결과를 향해 계속 진행하고 blocker를 명확히 보고 | nonviable run을 무기한 관찰하지 않고 P0로 판정 |
| 진행 소통 | tool 작업 중 사용자를 장시간 방치하지 않음 | SITREP를 강화하지만 runtime productive watchdog를 대체하지는 않음 |
| Skill routing | task가 Skill과 명확히 일치하면 Skill을 읽고 적용 | 조사·검토·문서화 품질에 적용 |

일반 commentary cadence는 application watchdog와 같은 계약이 아니다. rows, bytes, durable chunks, PID start token, stale threshold, GUI progress semantics는 CTC AGENTS와 사용자의 heartbeat가 규정했다.

### A.3 사용자의 직접 지시가 별도로 규정한 항목

사용자는 작업 중 다음을 반복해 명시했다.

- System Environment와 storage topology를 GUI와 execution planner가 실제로 소비할 것
- 일반 GUI 사용자는 F:가 없을 수 있으므로 drive absolute path를 고정하지 말 것
- 선택 storage가 예상 peak보다 작으면 사전 경고하고 실행을 차단할 것
- nominal worker count가 아니라 실제 ProcessPool 병렬화를 사용할 것
- 다운로드 시점에 normalization/containerization을 수행해 SQLite와 CUDA로 바로 넘길 것
- 10분 동안 0처럼 보이지 않는 productive watchdog를 제공할 것
- local sample이 아니라 real-volume station 통합 테스트를 수행할 것
- RC1 capsule/manifest/data 없이 current-version WebUI를 독립 생성할 것
- WebUI 전에 실제 CUDA 계산을 수행할 것
- capsule보다 WebUI를 먼저 export할 것
- demo에서 release Verify를 생략하되 VERIFIED라고 표현하지 말 것
- raw, checkpoint, evidence, valid cache를 보존할 것
- 나중에 위임하겠다는 말이 아니라 즉시 실행할 것

그러므로 P0-5 ingest containerization, P0-6 storage admission, P0-8 watchdog, CUDA→WebUI→capsule 순서는 AGENTS만의 추론이 아니라 직접 실행 요구이기도 하다.

### A.4 실제 반복 호출된 gstack Skill과 한계

Skill은 모든 일반 대화에 상시 주입되는 보편 규칙은 아니다. 일치하는 작업에서 호출되고 `SKILL.md`를 읽었을 때 절차로 작동한다. 그러나 **이번 작업에서는 gstack Skill을 실제로 수시로 반복 호출했다.** 따라서 “설치만 돼 있었고 사용되지 않았다”고 설명하면 안 된다.

계획, root-cause investigation, engineering review, performance comparison, safety, verification, documentation 과정에서 gstack workflow가 반복 사용됐다. 한국어·영어 보고서에는 `document-generate`가 직접 적용됐다. compact inherited transcript가 개별 tool invocation을 생략할 수 있으며, 요약에 호출이 보이지 않는다는 사실은 미사용 증거가 아니다. 정확한 call-by-call 목록이 필요하면 요약본이 아니라 원본 session tool log를 첨부해야 한다.

| Skill/workflow 범주 | 제공한 보호장치 | 정확한 사건 평가 |
|---|---|---|
| gstack planning / engineering review | architecture, data flow, edge case, performance limit, test gate를 구현 전 정리 | 반복 사용했지만 결론이 다음 hotfix·execution SHA·위임 작업에서도 강제되는 invariant로 유지되지 않음 |
| gstack investigation | root cause 우선, deterministic reproduction, hypothesis 확인, regression test, fresh verification | 반복 진단했지만 다음 real-data boundary가 다시 실행 중 발견됨. Skill 부재가 아니라 end-to-end propagation과 검증 불완전이 문제 |
| gstack performance / benchmark discipline | baseline, 실제 수치 측정, regression 비교, evidence report | worker CPU/RSS, I/O, queue, body-read, storage throughput을 수시 측정했지만 전체 pipeline launch-blocking budget으로 제때 결속하지 못함 |
| gstack safety workflow | destructive scope 제한, evidence 보존, exact target 검증 | raw/checkpoint/seal/log와 isolated worktree 보존에는 실제 도움이 됐지만 성능·orchestration 회귀를 막지는 못함 |
| `document-generate` | 작성 전 조사, 정확한 reference, concrete path/output/number, completeness review | 한국어·영어 Bug Report와 이 provenance appendix에 사용 |

핵심 결론:

> 문제는 gstack Skill이 없거나 호출되지 않은 것이 아니다. 반복 호출한 Skill의 결론, 수치, review gate가 다음 hotfix·재시작·execution SHA·workstream handoff에도 살아남는 machine-enforced invariant로 전환되지 못한 것이 문제다. P0 판정은 CTC AGENTS, 사용자의 직접 지시, 실제 측정된 영향, 요청 artifact 산출 실패를 근거로 한다.

### A.5 P0 항목별 출처 매핑

| 보고서 항목 | AGENTS 근거 | 상위 Prompt 중복 | 사용자 직접 지시 | Skill reinforcement | 최종 판정 근거 |
|---|---|---|---|---|---|
| P0-1 descriptor handoff loss | accuracy regression, incomplete verification, false completion trigger | 증거 기반 진단·scope fidelity | full official scope | 반복 조사/review가 descriptor 계약을 모든 layer에 전달했어야 함 | 고객 차단 integrity wiring 실패 |
| P0-2 conflicting periods | accuracy·deterministic input contract | 증거 기반 상태 보고 | restart 전 real official success | real-file investigation으로 발견했지만 full path 시작 후였음 | fail-closed 전체 실행 blocker |
| P0-3 ordinal semantics | scientific accuracy regression | unsupported completion claim 금지 | real provider evidence | investigation/regression test가 발견 사례는 수정했지만 fixture가 사전 포착하지 못함 | 실데이터 scientific integrity 실패 |
| P0-4 whole-file DataFrame | 느린 normalization과 memory/storage regression을 P0로 명시 | 일반 evidence 원칙만 부분 중복 | 실제 병렬 고속화 | performance measurement 반복 | Beta/RC 대비 측정된 성능 회귀 |
| P0-5 ingest container 없음 | chunk slicing, durable checkpoint, incomplete-only resume | 일반 architecture mandate 없음 | 다운로드 즉시 container 요구 | planning/investigation이 구조 원인을 찾았지만 실행 계약 미완료 | 직접 요구+AGENTS chunk 계약 |
| P0-6 storage admission 없음 | resource/storage regression | destructive safety·환경 인식 부분 중복 | GUI storage capacity 차단 직접 요구 | 측정은 했지만 launch gate 미결속 | 직접 요구+capacity risk |
| P0-7 resume reread | unchanged sealed file 재독 금지 | evidence 보존 | raw/checkpoint/ledger 재사용 | read-count measurement가 zero-read를 반복 강화 | durable resume 계약 위반 |
| P0-8 watchdog/progress | stale PID 금지, durable progress 필드 요구 | commentary cadence만 부분 중복 | 120초 all-stale, 10분 0 현상 제거 | monitoring evidence는 모았지만 terminal control 불충분 | AGENTS+직접 watchdog 위반 |
| P0-9 sample-real gap | real cold/warm/interruption test | evidence verification | local sample 성공 주장 거부 | Skill verification은 반복됐지만 real-shape coverage 부족 | 검증 방법론 실패 |
| P0-10 verification complexity | ctwebui one-full-hash, resource regression | 일반 efficiency는 P0 정의 아님 | demo Verify 생략 | read-count/performance measurement | 중복 작업과 artifact 차단 overhead |
| P0-11 silent termination | silent stall, watchdog error, stale PID, false completion trigger | current-state evidence | 120초 all-stale 요구 | monitoring은 증거를 모았으나 durable terminal cause 미게시 | 명시적 P0 trigger+dead run |

### A.6 감사 결론

영문판과 한국어판은 사건 본문을 축소하지 않으며, 이 부록으로 instruction provenance를 명시한다. 모든 P0가 일반 OpenAI Prompt에 독립적으로 정의된 것은 아니다. 동시에 gstack Skill은 실제 작업에서 반복 호출됐다. 정확한 비판은 “Skill을 안 썼다”가 아니라 **Skill 결과를 cross-run invariant로 유지·강제하지 못했다**는 것이다.
