# 문제점

심각도: **높음** = 보안·데이터 손상·주요 기능 오동작 가능, **중간** = 버그 유발이나 수정 시 혼란, **낮음** = 정리하면 좋은 것.
경로 약어는 [01_FEATURE_MAP.md](01_FEATURE_MAP.md)와 같다.

## 요약
- 비밀값 하드코딩: 없음 (테스트 설정의 더미 값 1건만, I-012)
- 하드 제약 위반: 발견 못 함. `eval`은 `kern/_executor.py:675` 1건 — 사용자 셀의 마지막 식을 평가하는 것이라 와이어 입력 eval이 아님. `pickle` 0건
- 순환 의존: 없음 (`_raw/cycles.md`)
- TODO: 1건 (I-010)
- 가장 큰 위험: **Python과 Rust에 같은 규약이 여러 번 구현됨** (I-001~I-004)

| ID | 분류 | 심각도 | 위치 | 내용 | 권장 조치 |
|---|---|---|---|---|---|
| I-001 | 불일치 | 높음 [확인 필요] | `cli/discovery.rs:189-200` vs `dpx-kernel/ident.rs:24`, `kern/_protocol.py:75` | CLI는 `std::fs::canonicalize` + 자체 해시로 kernel_id를 만든다. 매니저·커널은 Python `realpath`+`normcase` 규칙. Windows에서는 `canonicalize`가 `\\?\C:\...` 형태와 원래 대소문자를 돌려주므로 id가 달라져 `stop`/`status`/`logs`/`vars`가 `no kernel is running for ...`를 낼 수 있음 | CLI가 이미 의존하는 `dpx_kernel::ident::kernel_id_for`(`darkpyonix/Cargo.toml:17` 의존성 있음)를 쓰도록 교체. Windows CLI 테스트로 먼저 재현 |
| I-002 | 중복 | 중간 | `kern/_protocol.py:24-33` ↔ `dpx-kernel/discovery.rs:30-34`, `dkp.rs:21-22` | 디스커버리 그룹·포트·타임아웃·프레임 한도가 두 언어에 각각 정의 | 바꿀 때 양쪽 동시 수정. 장기적으로 하나에서 생성하거나 테스트로 값 일치 검사 |
| I-003 | 중복 | 중간 | `srv/auth.rs:87`, `kern/_collab.py:63`, `kern/_tokens.py:25`, `kern/_streams.py:42` | 능력 이름 5개가 4곳에 정의 | Python 3곳은 `_protocol.py`로 모으기 |
| I-004 | 중복 | 낮음 | `cli/discovery.rs:31` `home`, `srv/lib.rs:134` `default_home`, `dpx-kernel/home.rs:12` `default_home` | Rust 안에서 같은 홈 경로 함수 3개 (`dpx-kernel` 것만 절대경로화) | `dpx-kernel::home::default_home` 하나로 |
| I-005 | 중복 | 낮음 | `kern/__main__.py:34`, `kern/_streams.py:40` | `FINISHED` 튜플 중복 (`_model.RUN_STATUSES`의 부분집합) | `_model.py`로 이동 |
| I-006 | 중복 | 낮음 | `kern/_runs.py:49` ↔ `srv/util.rs:41`; `kern/_runs.py:58` ↔ `srv/util.rs:126`; `now_iso` 3곳 | run id 정규식, runs 폴더 경로, 시각 포맷 중복 | 변경 시 동시 수정 |
| I-007 | 불일치 | 중간 | `cli/discovery.rs:19` `DARKPYONIX_TOKEN` vs `cli/manager.rs:65` `DARKPYONIX_MANAGER_TOKEN` | 클라이언트가 읽는 토큰 변수와 매니저가 읽는 토큰 변수 이름이 다름. 매니저 기동 시 같은 이름을 기대하면 혼란 | 의도라면 문서화(README/SPEC), 아니면 통일 |
| I-008 | 죽은 코드 | 낮음 | `kern/_home.py:98` `managers_dir` | 호출처 0건 (grep) | 제거 또는 사용처 확인 |
| I-009 | 죽은 코드 후보 | 낮음 | `kern/_client.py` (215줄) | 헤더는 "CLI, manager, tests가 사용"이라 하지만 프로덕션 import 0건, 테스트만 사용 | 헤더 문구 수정 또는 `tests/helpers/`로 이동 검토 (패키지 노출 여부 결정 필요) |
| I-010 | 가짜 구현 | 낮음 | `srv/tls.rs:22` | ACME `TlsConfig` 변형: "TODO: not implemented; `serve` refuses it" | SPEC 상태 확인, 미구현 명시 유지 |
| I-011 | 가짜 구현 | 낮음 | `kern/_misc.py:55-66` `_reserved` | `run_cinterop`, `run_cppinterop`, `run_rustinterop`는 공개 export지만 호출하면 `NotImplementedError("darkpyonix.X is reserved: issue #5")` | 의도된 예약(issue #5). 문서에 v1 미지원 명시 유지 |
| I-012 | 하드코딩 | 낮음 | `darkpyonix/hub/worker/vitest.config.ts:16` | `RELAY_SHARED_SECRET: "relay-secret"` — 테스트 전용 더미 값 | 문제 없음. 실제 비밀은 `wrangler secret` |
| I-013 | 문서 불일치 | 낮음 | `darkpyonix/hub/server/src/main.rs:18`, `lib.rs:1-7` | 크레이트 설명은 "device registry, address directory, iroh relay, share links, HTTPS names"인데 실제 담당은 릴레이뿐 (D15) | `Args` doc 문구 갱신 |
| I-014 | 불일치 | 낮음 | `srv/error.rs:10` vs `hub/http.ts:10` | 오류 JSON 모양이 매니저 `{"error": {code, message}}`, 허브 `{"error": "...", "code"?}`로 다름 | 클라이언트(ember/ash)가 둘 다 처리하는지 확인. 의도된 것이면 문서화 |
| I-015 | 불일치 | 낮음 | `packaging/check_versions.py` | 버전 검사가 `__init__.py`와 `Cargo.toml`만 본다. `hub/worker/package.json` `version`(0.3.0)은 검사 밖 | 검사 대상에 추가하거나 의도 명시 |
| I-016 | 하드코딩 | 낮음 | `hub/notebooks.ts:9` `LIMITS` | 모든 한도가 "provisional" (주석) | 실사용 후 조정. 이 객체 한 곳에서만 바꿀 것 |
| I-017 | 거대 파일 | 중간 | `srv/api.rs` (1,979줄) | 모든 엔드포인트가 한 파일 | 도메인별(kernels/runs/collab/auth/shares) 모듈 분할 검토 |
| I-018 | 거대 파일 | 낮음 | `dpx-kernel/dkp.rs` (995), `kern/_collab.py` (890), `kern/_runs.py` (748), `kern/_executor.py` (714), `srv/auth.rs` (678), `hub/devices.ts` (622), `srv/lib.rs` (618) | 500줄 초과 | 당장 조치 불필요. 수정 시 모듈 문서의 줄 번호 참고 |
| I-019 | 거대 함수 | 중간 | `cli/commands.rs:89` `dispatch_inner` (~154줄), `kern/_capture.py:142` `_loop` (110), `srv/handoff.rs:320` `pump_response` (~117), `kern/_collab.py:760` `_reload` (90), `packaging/smoke.py:119` `main` (99) | 100줄 안팎 단일 함수 | 수정 시 회귀 테스트 필수 |
| I-020 | 외부 의존 | 낮음 | `srv/proxy.rs:14-15` | `hyper_util::client::legacy` 사용 (hyper-util의 legacy 클라이언트) | 업그레이드 시 확인 |
| I-021 | 안 쓰는 의존성 | — | 전체 | 도구(`cargo udeps`, `knip`)를 돌리지 않음 (토큰·빌드 규칙상 생략). 커널은 의존성 0 | 필요 시 builder 서브에이전트로 실행 |

## 잔해 파일
`old`, `backup`, `copy`, `v2`, `temp` 이름의 소스 파일 없음. `docs/설계초안/`은 노션 내보내기 원본(HTML·CSV 80여 개)으로 코드가 아니지만 저장소 크기를 차지한다 — 보관 의도인지 확인 [확인 필요].
