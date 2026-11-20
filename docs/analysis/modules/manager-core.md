# 모듈: manager-core

- 경로: `darkpyonix/manager/crates/dpx-core/`, `darkpyonix/manager/crates/dpx-kernel/`
- 규모: 파일 9개 / 약 2,770줄 (Rust)
- 레이어: 공용 계약 + 인프라 (커널 접근)
- 한 줄 책임: `KernelBackend` 트레이트(계약)와 그 실구현 `RealBackend` — 디스커버리, 커널 기동, DKP/1 연결 공유, 이벤트 팬아웃
- 의존하는 모듈: 없음 (Python 커널과는 와이어로만 통신, 소스는 `build.rs`로 내장)
- 이 모듈을 쓰는 모듈: [manager-server](manager-server.md), [cli](cli.md)

## 공개 인터페이스

| 이름 | 종류 | 위치 | 설명 |
|---|---|---|---|
| `KernelBackend` | 트레이트 | `dpx-core/src/lib.rs:119` | HTTP 계층이 커널에 요구하는 전부 |
| `DpxError`, `Result` | 구조체/타입 | `dpx-core/src/lib.rs:22`, `:41` | OpenAPI `Error` + DKP 오류 코드 |
| `KernelInfo` | 구조체 | `dpx-core/src/lib.rs:45` | OpenAPI `Kernel` |
| `StartKernel`, `Ensured` | 구조체 | `:71`, `:83` | `POST /api/kernels` 본문 / 200 vs 201 |
| `KernelEvent`, `AdoptRequest` | 구조체 | `:90`, `:105` | 이벤트 / 스트림 인계 |
| `RealBackend` | 구조체 | `dpx-kernel/src/lib.rs:77` | 실구현 (`impl KernelBackend` `:346`) |
| `Config::from_env` | 함수 | `dpx-kernel/src/lib.rs:54` | `DARKPYONIX_HOME`, `DARKPYONIX_PYTHON` |
| `START_TIMEOUT` | 상수 | `dpx-kernel/src/lib.rs:34` | 기동한 커널의 announce 대기 시간 (FR-M2) |

## 파일별 요약

### `dpx-kernel/src/dkp.rs` (995줄)
- **목적:** DKP/1 클라이언트. 커널당 `Connection`(`:424`) 하나를 모든 요청·이벤트 구독자가 공유
- **주요 심볼:** `encode` `:48`, `read_frame` `:63`, `open_stream`(Unix `:88` / Windows `:120`), `require_same_account` `:106`, `require_same_sid` `:157`, `adopt`(`:746` Unix fd 전달 / `:806` Windows), `base64_encode` `:688`
- **상수:** `MAX_FRAME` `:21`, `HANDSHAKE_TIMEOUT` `:22`, `REQUEST_TIMEOUT` `:23`, `RING_MAX_BYTES` `:24`
- **주의점:** 커널 계정이 다르면 아무것도 보내지 않는다(FR-A1).

### `dpx-kernel/src/lib.rs` (484줄)
- **목적:** `RealBackend` — `ensure`(있으면 200, 없으면 기동 후 201), 목록, 이벤트, 인계 등
- **사용처:** `crates/darkpyonix/src/manager.rs:48` `run_manager`

### `dpx-kernel/src/launch.rs` (278줄)
- **목적:** 내장 Python 패키지를 `<home>/runtime/<tag>/`에 한 번 풀고(`extract_runtime` `:32`), 인터프리터 선택(`resolve_python` `:95`), 분리 기동(`spawn_kernel` `:135`), `build_document` (`:229`, Python 서브프로세스로 `_document.build_document` 실행, `DOCUMENT_TIMEOUT` `:18`)
- **주의점:** `runtime_tag` (`:25`) = `<crate version>-<content hash>`. 커널 소스가 바뀌면 새 폴더에 풀린다.

### `dpx-kernel/build.rs` (73줄)
- **목적:** `darkpyonix/kernel/` 소스를 바이너리에 내장(`$OUT_DIR/embedded.rs`) — Python 수정이 Rust 재빌드를 유발

### 한 줄 요약
| 파일 | 줄 | 요약 |
|---|---|---|
| `dpx-core/src/lib.rs` | 146 | 계약 타입 + `KernelBackend` |
| `dpx-kernel/src/discovery.rs` | 318 | UDP 멀티캐스트 질의 + 백그라운드 announce 캐시 (`Discovery` `:46`) |
| `dpx-kernel/src/home.rs` | 143 | `default_home` `:12`, `account` `:53`, `user_tag` `:141` |
| `dpx-kernel/src/ident.rs` | 235 | Python `posixpath.realpath`/`ntpath` 이식, `kernel_id_for` `:13`, `canonical_path` `:24` |
| `dpx-kernel/src/process.rs` | 98 | `pid_alive` `:4`, `kill_pid` `:12` (SIGKILL/TerminateProcess) |

## Python 사본 대응표 (한쪽만 바꾸면 깨지는 곳)

| 개념 | Python | Rust |
|---|---|---|
| 디스커버리 그룹·포트·타임아웃 | `_protocol.py:24-29` | `dpx-kernel/src/discovery.rs:30-34` |
| 프레임 한도·핸드셰이크 | `_protocol.py:32-33` | `dpx-kernel/src/dkp.rs:21-22` |
| kernel_id, canonical path | `_protocol.py:75`, `:83` | `dpx-kernel/src/ident.rs:13`, `:24`; **CLI는 별도 구현** `crates/darkpyonix/src/discovery.rs:189-200` |
| 홈 경로 | `_home.py:16` | `dpx-kernel/src/home.rs:12`, `dpx-server/src/lib.rs:134`, `crates/darkpyonix/src/discovery.rs:31` |
| account / user_tag | `_home.py:151`, `:157`, `_peer.py:46` | `dpx-kernel/src/home.rs:53`, `:141` |
| 부트스트랩 문자열 | `_launcher.py:22-27` | `dpx-kernel/src/launch.rs:67` |
| `now_iso` | `_protocol.py:150` | `dpx-kernel/src/dkp.rs:250`, `dpx-server/src/util.rs:14` |
| runs 폴더 | `_runs.py:58` | `dpx-server/src/util.rs:126` |

## 이 모듈을 수정할 때 주의할 점
- 빌드는 AGENTS.md 규칙대로 builder 서브에이전트에서만, `CARGO_BUILD_JOBS=2`.
- 커널을 kill하는 경로는 `process.rs:12` 하나뿐이어야 한다("정지 = 인터럽트").
