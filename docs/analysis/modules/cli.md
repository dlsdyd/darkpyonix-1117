# 모듈: cli

- 경로: `darkpyonix/manager/crates/darkpyonix/src/`
- 규모: 파일 8개 / 약 1,820줄 (Rust)
- 레이어: UI (터미널)
- 한 줄 책임: 사람과 에이전트가 부르는 `darkpyonix` 명령. 매니저 HTTP API만 사용하며, 매니저가 없으면 임시 매니저를 띄운다. `darkpyonix manager`는 같은 바이너리의 매니저 모드
- 의존하는 모듈: [manager-server](manager-server.md), [manager-core](manager-core.md) (매니저 모드에서만)
- 이 모듈을 쓰는 모듈: 없음 (진입점 `main.rs:17`)

## 공개 인터페이스 (명령)
`_raw/routes.md` §3 표 참조. 정의 `args.rs:25-52`, 실행 `commands.rs:89` `dispatch_inner`.

## 파일별 요약

### `commands.rs` (573줄)
- **목적:** 각 명령 = 몇 번의 HTTP 호출
- **주요 심볼:**
  | 이름 | 줄 | 설명 |
  |---|---|---|
  | `dispatch` | 79 | 종료 코드 반환 |
  | `dispatch_inner` | 89 | 명령별 분기(154줄 단일 함수) |
  | `ensure_kernel` | 243 | `POST /api/kernels`, 201이면 새로 띄운 것 |
  | `run` | 252 | 실행 요청 후 이벤트 따라가기 |
  | `busy` | 300 | 이미 실행 중이면 현재 run 출력 후 exit 75 |
  | `logs` | 321 | 실행 기록 출력 |
  | `CtrlC` | 397 | 첫 Ctrl-C = `POST /interrupt`, 두 번째 = 분리(detach) |
  | `follow` | 438 | SSE 이벤트 스트림 추적 |
  | `handle_event` | 517 | 이벤트 1개 출력, run 종료 시 종료 코드 |
- **사용자에게 보이는 문구:** `"no kernel is running for {label}"` (`:110`)

### `discovery.rs` (264줄)
- **목적:** 매니저 찾기(FR-C1): `DARKPYONIX_MANAGER_URL` + `DARKPYONIX_TOKEN` 환경 변수, 없으면 `darkpyonix manager --ephemeral`을 별도 세션으로 기동(`spawn_ephemeral` `:114`)
- **상수:** `SELF_EXE_ENV` `:13`, `SPAWN_TIMEOUT` `:15`, `URL_ENV` `:18`, `TOKEN_ENV` `:19`, `MANAGER_LOG_LIMIT` `:84`(넘으면 `manager.log.1`로 회전)
- **주의점:** `resolve_file` `:189`이 `std::fs::canonicalize` + 자체 `kernel_id_for` `:196`을 쓴다. 매니저·커널은 `realpath+normcase` 규칙(`dpx-kernel/src/ident.rs:24`)이라 **Windows에서 id가 달라질 수 있음** → [05_ISSUES.md](../05_ISSUES.md) I-001.

### `render.rs` (309줄)
- **종료 코드:** `EXIT_OK` `:5`, `EXIT_ERROR` `:6`, `EXIT_BUSY` `:8`(EX_TEMPFAIL 75), `EXIT_INTERRUPTED` `:10`(128+SIGINT=130); `exit_code_for_run_status` `:13`
- **출력:** `render_output` `:49`(nbformat → 터미널), `strip_ansi` `:91`, `kernels_table` `:144`(`ps`), `variables_table` `:164`(`vars`), `kernel_details` `:187`(`status`)

### 한 줄 요약
| 파일 | 줄 | 요약 |
|---|---|---|
| `main.rs` | 33 | `manager`만 별도, 나머지는 current-thread tokio로 `dispatch` |
| `args.rs` | 267 | clap 정의. `parse_cells` `:154`(`1,3`/`2-4`), `parse_param` `:178`(`k=v`, v는 JSON 우선) |
| `client.rs` | 132 | `Api` `:50`(마스터 토큰으로 매니저 호출), `CliError` `:10` |
| `manager.rs` | 140 | `run_manager` `:48`, `startup_line` `:88`(stdout JSON 1줄), `DEFAULT_IDLE_TIMEOUT` `:24` |
| `sse.rs` | 105 | SSE 증분 파서 |

## 이 모듈을 수정할 때 주의할 점
- 명령 추가 = `args.rs` enum + `commands.rs` 두 match(`:92-100` 파일 인자 추출, `:117-238` 실행) + SPEC FR-C2.
- 종료 코드는 에이전트가 의존하는 계약(FR-C2). 바꾸지 말 것.
- 검증 테스트: `crates/darkpyonix/tests/cli.rs`, `share_views.rs`, `manager_latency.rs`.
