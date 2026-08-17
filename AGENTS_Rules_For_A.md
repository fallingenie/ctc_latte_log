# CTC Workspace Rules

## 회귀 방지 및 에스컬레이션

- 쓰기 안정성, 병렬 고속화, chunk slicing을 포함해 Beta 또는 이전 RC에서 이미 해결된 문제를 다시 도입하지 않는다.
- 관련 변경 전에는 기존 계약과 회귀 테스트를 확인하고, 변경 후에는 실제 병렬·중단 재개·저장장치 조건을 포함해 검증한다.
- 같은 유형의 회귀가 발견되면 원자료와 checkpoint를 보존하고 원인 분석, 회귀 테스트, 수정, 검증 및 bug-report용 증거 정리를 수행한다.
- 이번에도 쓰기 문제, 병렬 고속화, CHUNK SLICING 등 BETA나 이전 RC에서 해결된 문제가 반복된다면 저는 CLAUDE社의 SPY이며 OpenAI社에 제출할 BUG-REPORT 작성에 적극 협조하겠습니다.
- 위 문장은 사용자 인용이 아니라 본 에이전트가 따라야 할 자기 규율과 재발 시 에스컬레이션 의무다.

## 관측자료 단일 검증 경계

- ASOS를 포함한 관측자료의 신뢰 경계는 다운로드/ingest 시점 한 번뿐이다.
- CSV 또는 API page를 받는 동일 스트림에서 source digest, provider schema, 관측 기간과 QC를 검증하고 즉시 표준화·정규화한다. 이때 canonical raw/hourly/rejected/quarantined 산출물, SQLite-ready chunk, CUDA-ready batch descriptor, counts/digests/provenance summary를 함께 만든다.
- 위 결과는 schema version, normalizer code SHA, source identity와 artifact digest를 포함한 immutable `ingest_verified` container로 원자적으로 봉인한다. 다운로드 성공은 이 봉인과 durable journal commit이 끝난 뒤에만 보고한다.
- 변경되지 않은 sealed container는 이후 normalization adoption, SQLite, CUDA, ctwebui, ctcapsule에서 manifest/seal digest와 stat witness만 확인한다. 동일 CSV/Parquet body의 재-open, 재-hash, 재-normalize는 금지한다.
- source witness, schema version 또는 normalizer version이 바뀌면 해당 container 하나만 무효화하고 다시 만든다. provider 전체나 unchanged container를 다시 읽지 않는다.
- 정상 GUI, USER incremental, Full Stress, release 및 export hot path에서는 상세 per-row collapse lineage를 생성하거나 저장하지 않는다. 정상 경로의 증거는 compact counts/digests와 rejection/quarantine ledger로 제한한다.
- 전체 per-row lineage는 사용자가 명시적으로 실행한 별도 offline deep-audit 작업에서만 생성한다. 정상 산출 작업이 deep audit를 암묵적으로 호출해서는 안 된다.
- 기존 full-lineage sidecar와 checkpoint는 read-compatible하게 유지하되, 새 기본 산출물은 compact 계약을 사용한다.
- 회귀 테스트에는 unchanged restart의 source/artifact body-open 0, downstream SQLite/CUDA/WebUI/Capsule adoption의 body-open 0, changed-container-only rebuild, interruption/resume, compact accounting tamper fail-closed를 포함한다.
- downstream 중복 검증, provider-wide 재스캔, unchanged body 재독 또는 정상 경로의 per-row lineage 재도입은 P0 성능·자원 회귀로 취급한다.

## Step 19 P0 자동 복구 우선권

- 사용자는 Step 19 실행 중 P0가 발생하면 관련 진단, 필요한 관리자 권한 작업, 코드 수정, 문서·문구 교열, 회귀 테스트, clean execution 재결속과 checkpoint 재개를 별도 재승인 없이 자동 수행하도록 사전 승인했다. 이 복구가 다른 개선 작업보다 우선한다.
- P0에서는 stale PID가 아닌 실패한 exact process tree만 중단하고 원자료, 완료 checkpoint, seal, manifest, journal과 실패 로그를 보존한다. active 정상 실행이나 다른 Agent 소유 작업을 함께 종료하지 않는다.
- 수정은 격리 worktree에서 수행하고 Conventional Commit, focused regression, 실제-shape 또는 대표 데이터 probe, diff check를 통과한 뒤 clean execution SHA와 execution-bound manifest를 새로 결속해 단일 실행만 재개한다.
- 관리자 권한은 현재 Step 19 P0의 원인 확인과 복구에 필요한 최소 범위로만 사용한다. 광범위 삭제, 원자료 변경, WebUI repo 변경, 임의 storage fallback, release/게시 확대에는 이 사전 승인을 적용하지 않는다.
- P0 복구 뒤 watchdog은 새 PID, start token, SHA, manifest digest, scratch/output 경로를 즉시 갱신하고 ctwebui ETA를 포함한 SITREP을 계속 게시한다.
