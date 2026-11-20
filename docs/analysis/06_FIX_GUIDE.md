# 수정 가이드

경로 약어는 [01_FEATURE_MAP.md](01_FEATURE_MAP.md)와 같다: `cli/`, `srv/`, `kern/`, `hub/`, `dpx-kernel/`.

## 1. 수정 절차 (항상 이 순서로)
1. **위치 찾기:** [01_FEATURE_MAP.md](01_FEATURE_MAP.md)에서 명령어·엔드포인트·에러 문구를 검색 → 핸들러와 커널 연산 위치 확인
2. **이름 확인:** [02_GLOSSARY.md](02_GLOSSARY.md)에서 관련 이름의 의미와 "함께 바꿀 곳" 확인
3. **SPEC 확인:** 이 저장소는 SPEC 주도 개발이다(AGENTS.md). `docs/SPEC.md`에서 요구사항 ID(FR-*, NFR-*)를 찾는다. 동작을 바꾸면 **SPEC 먼저**, 엔드포인트를 바꾸면 **`docs/api/*.openapi.yaml` 먼저**
4. **영향 범위 확인:** 고칠 함수 이름을 저장소 전체에서 검색. 특히 아래 "Python·Rust 짝" 표에 있으면 양쪽 다
   - 사용처 1곳: 바로 수정
   - 여러 곳: 시그니처를 바꾸지 말고 옵션 인자나 새 함수 추가
5. **재현 테스트 먼저:** 실패하는 테스트를 요구사항 이름으로 작성 (`test_fr_k3_...`). 테스트와 수정은 같은 커밋
6. **작게 수정:** 한 커밋에 한 문제. 커밋 제목 `Fix: ...`, 커밋 직후 push
7. **검사 실행:**
   | 대상 | 명령 | 비고 |
   |---|---|---|
   | Python 커널 | `pytest` (저장소 루트) | 가능한 모든 인터프리터에서 실행됨(NFR-K1). `DARKPYONIX_HOME`은 `.scratch/` |
   | Rust 매니저·CLI | `cargo test` (`darkpyonix/manager/`) | **직접 빌드 금지** — builder 서브에이전트 1개, `CARGO_BUILD_JOBS=2`, 또는 push 후 CI(`test.yml`) |
   | 허브 Worker | `npm test`, `npm run typecheck` (`darkpyonix/hub/worker/`) | |
   | 표준 라이브러리 제약 | `tests/test_nfr_k2_stdlib_only.py` | 커널 수정 시 자동 포함 |
8. **검증:** 원래 문제 해결 + 4단계 사용처 확인 + 아래 기본 점검 목록

## 2. 기본 점검 목록 (수정 후 매번, [03_FLOWS.md](03_FLOWS.md)와 일치)
- [ ] 흐름 1: `darkpyonix run 예시.py`가 출력을 보여주고 exit 0, `__runs__/`에 `.ipynb` 생성
- [ ] 흐름 1: 실행 중 Ctrl-C 한 번 → 셀 인터럽트, exit 130. 커널은 살아 있음(`darkpyonix ps`)
- [ ] 흐름 1: 같은 파일 두 번째 `run` → exit 75와 안내 문구
- [ ] 흐름 2: 같은 파일로 커널을 두 번 띄워도 하나만 남음 (`darkpyonix ps`)
- [ ] 흐름 3: 셀 PATCH 후 파일 내용이 바뀌고 이벤트 스트림에 `doc.cell.updated`
- [ ] 흐름 4: `darkpyonix attach FILE`이 이벤트를 받음
- [ ] 흐름 5 (허브 수정 시): `npm test` 통과 (기기 연결 시나리오 `hub/worker/test/devices.test.ts`)
- [ ] `python 예시.py`로 실행해도 똑같이 동작 (FR-F5, 노트북은 유효한 Python)

## 3. 기능별 수정 포인트

### CLI
| 바꾸고 싶은 것 | 고칠 위치 | 함께 고쳐야 하는 곳 | 주의 |
|---|---|---|---|
| 명령 추가 | `cli/args.rs:26` `Command` | `cli/commands.rs:92-100`(파일 인자 match), `:117-238`(실행 match), SPEC FR-C2 | match 두 곳 중 하나를 빠뜨리면 컴파일 오류 |
| 에러 문구 | `cli/commands.rs:110`, `:371`, `:505` 등 | 테스트 `crates/darkpyonix/tests/cli.rs`의 기대 문구 [확인 필요] | |
| 종료 코드 | `cli/render.rs:5-10` | SPEC FR-C2 | 에이전트가 의존하는 계약. 바꾸지 말 것 |
| `ps`·`vars` 표 모양 | `cli/render.rs:144`, `:164` | 없음 | |
| `--cells` 문법 | `cli/args.rs:154` `parse_cells` | 없음 | |
| 매니저 로그 회전 크기 | `cli/discovery.rs:84` `MANAGER_LOG_LIMIT` | 없음 | |
| Windows kernel_id (I-001) | `cli/discovery.rs:189-200` | `dpx-kernel/ident.rs:13`로 교체 | Windows CI에서 확인 |

### 매니저 HTTP API
| 바꾸고 싶은 것 | 고칠 위치 | 함께 고쳐야 하는 곳 | 주의 |
|---|---|---|---|
| 엔드포인트 추가 | `srv/api.rs` 핸들러 + `srv/lib.rs:358-389` | **`docs/api/manager.openapi.yaml` 먼저**, `dpx-core` 트레이트(필요 시), 커널 연산(필요 시) | 스키마 비교 테스트가 있음 |
| 엔드포인트 필요 능력 | 해당 핸들러의 `require(..., Capability::X)` | OpenAPI 설명, SPEC FR-A3 | |
| 임시 매니저 유휴 시간 | `srv/lib.rs:56` | `cli/manager.rs:24` (같은 기본값 2곳) | |
| 전용 매니저 포트 | `srv/lib.rs:57` | 문서(README/SPEC) | |
| 공유 링크 주소 | `srv/lib.rs:58` `DEFAULT_SHARE_BASE` | 허브 `/s/{id}` (`hub/shares.ts:98`) | |
| 비밀번호 잠금 정책 | `srv/auth.rs:253-254` | `ServerConfig` 기본값 `srv/lib.rs` [확인 필요] | |
| 요청 본문 한도 | `srv/lib.rs:63` `MAX_BODY` | 커널 `MAX_FRAME`보다 크면 커널에서 거부 | |
| 오류 → HTTP 상태 | `srv/error.rs:68` | 커널 `_streams.py:111` `_STATUS`(인계 스트림 오류) | 두 매핑이 따로 있음 |

### 커널 실행
| 바꾸고 싶은 것 | 고칠 위치 | 함께 고쳐야 하는 곳 | 주의 |
|---|---|---|---|
| 실행 상태 추가 | `kern/_model.py:10` | `FINISHED` 2곳(`__main__.py:34`, `_streams.py:40`), `cli/render.rs:13` 종료 코드, OpenAPI enum | 빠지면 CLI가 종료를 감지 못 함 |
| 셀 출력 상한 | `kern/_runs.py:45` 또는 env `DARKPYONIX_RUN_OUTPUT_LIMIT` | 없음 | |
| 실행 기록 쓰기 주기 | `kern/_runs.py:39-44` | 없음 | |
| 실행 기록 형식(`metadata.darkpyonix`) | `kern/_runs.py:535` | `srv/api.rs:780` `run_summary`, OpenAPI `RunSummary` | Rust가 직접 파싱 |
| `vars`에 숨길 이름 | `kern/_executor.py:48` `INJECTED_NAMES` | 없음 | |
| matplotlib 처리 | `kern/_figures.py`, `kern/_mplbackend.py` | 없음 | 커널은 matplotlib을 import하지 않아야 함 |

### 노트북 포맷·런타임 API
| 바꾸고 싶은 것 | 고칠 위치 | 함께 고쳐야 하는 곳 | 주의 |
|---|---|---|---|
| 셀 타입 추가 | `kern/format/__init__.py:19` | `docs/FORMAT.md` 표, 실행기의 타입 분기(`_executor.py`) [확인 필요] | |
| 마커 문법 | `kern/format/_parser.py:25` | `serialize` 바이트 동일성 테스트 `tests/test_fr_f1_format.py` | |
| 셀 해시 규칙 | `kern/format/_parser.py:60` | **기존 `__runs__` 기록 매칭이 깨짐** (`_document.py:42`, `_wsync.py:152`) | 사실상 바꾸지 말 것 |
| 런타임 API 추가 | `kern/__init__.py:14` `_EXPORTS` + 새 모듈 | `docs/FORMAT.md` §4, `tests/test_fr_f_runtime.py:62` 모듈 목록 | 커널 밖 동작 필수 |
| params 불리언 문자열 | `kern/_params.py:15-16` | 없음 | |

### 공동 편집·토큰
| 바꾸고 싶은 것 | 고칠 위치 | 함께 고쳐야 하는 곳 | 주의 |
|---|---|---|---|
| 능력 이름 추가 | `srv/auth.rs:87` | `kern/_collab.py:63`, `kern/_tokens.py:25`, `kern/_streams.py:42`, OpenAPI, SPEC FR-A3 | **4곳** (I-003) |
| 커서 이벤트 빈도 | `kern/_collab.py:65` | 없음 | |
| presence 주기 | `srv/lib.rs:62` | `kern/_streams.py:34` | 커널 만료(30초)보다 짧아야 함 |
| 새 문서 연산 | `kern/_collab.py:276` `handlers` | `srv/api.rs` 핸들러, `docs/PROTOCOL.md` §4 | |

### 디스커버리·프로세스
| 바꾸고 싶은 것 | 고칠 위치 | 함께 고쳐야 하는 곳 | 주의 |
|---|---|---|---|
| 멀티캐스트 주소·포트 | `kern/_protocol.py:24-26` | `dpx-kernel/discovery.rs:30-32`, `docs/PROTOCOL.md` | 불일치 시 매니저가 커널을 못 찾음 |
| announce 주기·만료 | `kern/_protocol.py:27-29` | `dpx-kernel/discovery.rs:33-34` | |
| 프레임 한도 | `kern/_protocol.py:32` | `dpx-kernel/dkp.rs:21` | |
| 홈 경로 규칙 | `kern/_home.py:16` | `dpx-kernel/home.rs:12`, `srv/lib.rs:134`, `cli/discovery.rs:31` | Rust 3곳 |
| 부트스트랩 | `kern/_launcher.py:22-27` | `dpx-kernel/launch.rs:67` | |

### 허브 (darkpyonix.dev)
| 바꾸고 싶은 것 | 고칠 위치 | 함께 고쳐야 하는 곳 | 주의 |
|---|---|---|---|
| 노트북 한도 | `hub/notebooks.ts:9` `LIMITS` | 분당 한도는 `wrangler.toml` | 주석: 여기서만 바꿀 것 |
| 기기 연결 유효 시간 | `hub/devices.ts:40` | `hub/pages.ts` 안내 문구 [확인 필요] | |
| DB 컬럼 추가 | 새 `migrations/00NN_*.sql` | 해당 `*.ts`의 SELECT/타입, `hub.openapi.yaml` | 기존 마이그레이션 수정 금지 |
| 기기 목록 변경 동작 | `hub/devices.ts` | `bumpDevicesVersion` `:127` 호출 | 빠지면 롱폴 클라이언트가 변경을 못 봄 |
| 엔드포인트 추가 | `hub/app.ts:104` `ROUTES` | `docs/api/hub.openapi.yaml` 먼저 | |

## 4. 건드리면 위험한 곳
| 위치 | 이유 | 수정이 필요할 때 |
|---|---|---|
| `kern/__main__.py:237` `_teardown` | 예외를 삼켜 **잠금 해제를 보장**. 순서 변경 시 잠금이 남아 재기동 불가 | 새 단계는 기존 람다 사이에 추가, 예외 처리 유지 |
| `kern/_lock.py` | 파일당 커널 1개 (하드 제약) | 매니저가 대신 강제하게 바꾸지 말 것 |
| `kern/_executor.py` 스레드 모델 | 실행은 메인 스레드만 (`run_forever` `:377`) | INTENT D12 확인 |
| `kern/_capture.py:142` `_loop` | 출력 순서·누락에 직결 | `tests/test_fr_x_executor.py` 전체 통과 확인 |
| `srv/api.rs:423-432` 강제 kill | 매니저가 커널을 죽이는 유일한 곳 (INTENT D9) | 다른 곳에 kill 추가 금지 |
| `srv/lib.rs:290` `auth_gate` | 라우팅 전 인증. 순서를 바꾸면 익명에게 400/404가 새어 나감 | |
| `srv/handoff.rs` + `dpx-kernel/dkp.rs:705-806` | 소켓/fd 인계, OS별 분기 | Linux·macOS·Windows CI 모두 확인 |
| `kern/_winpipe.py`, `kern/_peer.py` | 접근 제어(같은 계정만) | 보안 회귀. Windows 테스트 필수 |
| `kern/format/_parser.py:60` | 셀 해시 | 바꾸면 과거 실행 기록이 셀에 안 붙음 |
| `hub/http.ts:128` `principal`, `hub/util.ts:131` `safeReturnTo` | 인증·오픈 리다이렉트 방지 | |
| 커널 코드 전체 | Python 3.8 + 표준 라이브러리만 | 3.9+ 문법(`match`, `X | Y` 타입 등) 금지 |
