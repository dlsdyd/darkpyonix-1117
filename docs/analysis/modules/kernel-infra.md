# 모듈: kernel-infra

- 경로: `darkpyonix/kernel/` 중 `_protocol.py`, `_home.py`, `_peer.py`, `_winpipe.py`, `_lock.py`, `_host.py`, `_hostctx.py`, `_events.py`, `_model.py`
- 규모: 파일 9개 / 약 1,290줄
- 레이어: 인프라 / 공용
- 한 줄 책임: 커널·클라이언트·매니저가 공유하는 와이어 상수와 프레임, 런타임 홈 경로, 파일 잠금, 피어 계정 확인, 이벤트 링, 실행 레코드 모델
- 의존하는 모듈: 없음 (표준 라이브러리만)
- 이 모듈을 쓰는 모듈: [kernel-runtime-api](kernel-runtime-api.md), [kernel-exec](kernel-exec.md), [kernel-collab](kernel-collab.md), [kernel-process](kernel-process.md). Rust 쪽 [manager-core](manager-core.md)가 같은 규약을 **별도로 다시 구현**한다.

## 공개 인터페이스

| 이름 | 종류 | 위치 | 설명 |
|---|---|---|---|
| `DKPError` | 클래스 | `_protocol.py:52` | `{"ok": false, "error": {...}}`로 전송되는 오류 |
| `encode`, `recv_frame`, `FrameDecoder` | 함수/클래스 | `_protocol.py:91`, `:113`, `:124` | 길이 접두 JSON 프레임 인코딩·디코딩 |
| `canonical_path`, `kernel_id_for` | 함수 | `_protocol.py:75`, `:83` | 파일 경로 → `k_` + SHA-256 앞 20자리 (FR-K2) |
| `now_iso`, `run_start_iso`, `new_run_id` | 함수 | `_protocol.py:150`, `:160`, `:179` | 시각·run id 생성 |
| `home`, `subdir`, `control_path`, `user_tag` | 함수 | `_home.py:16`, `:22`, `:133`, `:157` | `DARKPYONIX_HOME` 아래 경로 |
| `expose_package` | 함수 | `_home.py:73` | 자식 인터프리터에서 `import darkpyonix`가 되게 함 |
| `FileLock`, `lock_path` | 클래스/함수 | `_lock.py:19`, `:14` | 파일당 커널 1개 (FR-K3) |
| `same_account` | 함수 | `_peer.py:38` | 소켓 상대가 같은 OS 계정인지 |
| `PipeListener`, `connect_client`, `pipe_name` | 클래스/함수 | `_winpipe.py:407`, `:387`, `:23` | Windows named pipe 제어 채널 |
| `EventLog` | 클래스 | `_events.py:17` | 번호 붙은 이벤트 링 (PR-3) |
| `RunRequest`, `CellRecord`, `Run` | 클래스 | `_model.py:14`, `:35`, `:56` | 실행 요청·셀 결과·실행 레코드 |
| `in_kernel`, `emit_display`, `set_active`, `clear` | 함수 | `_hostctx.py:16`, `:26`, `:69`, `:83` | 실행기 ↔ 런타임 API 연결 |
| `hostctx`, `active`, `current_params` | 함수 | `_host.py:11`, `:23`, `:34` | `_hostctx`의 방어적 접근 |

## 파일별 요약

### `_protocol.py` (198줄) — 팬인 14 (가장 많이 쓰임)
- **목적:** DKP/1 와이어 규약의 단일 출처 (`docs/PROTOCOL.md`)
- **주요 상수:**
  | 이름 | 줄 | 값 |
  |---|---|---|
  | `DKP_VERSION` | 20 | `1` |
  | `DISCOVERY_GROUP` / `DISCOVERY_PORT` / `DISCOVERY_INTERFACE` | 24-26 | `'239.255.68.80'` / `46880` / `'127.0.0.1'` |
  | `ANNOUNCE_INTERVAL`, `STALE_AFTER`, `QUERY_TIMEOUT` | 27-29 | `5.0`, `15.0`, `0.2` 초 |
  | `MAX_FRAME` | 32 | 64 MiB |
  | `EVENT_RING_MAX_EVENTS` / `_BYTES` | 36-37 | `10000` / 16 MiB |
  | `EXIT_ALREADY_RUNNING` | 41 | `3` |
  | `ERROR_CODES` | 43 | `'bad_request'`, `'unknown_method'`, `'busy'`, `'not_found'`, ... |
  | `KERNEL_STATUSES` | 49 | `'starting'`, `'idle'`, `'busy'`, `'stopping'` |
- **주의점:** 같은 값이 Rust에도 있다 — `dpx-kernel/src/discovery.rs:30-34`, `dkp.rs:21-22`. 한쪽만 바꾸면 매니저가 커널을 못 찾는다.

### `_home.py` (159줄)
- **목적:** 런타임 홈(`~/.darkpyonix`) 하위 경로와 권한(0700/0600) 관리
- **부수효과:** 디렉터리 생성, `write_private`(`:102`)·`write_public`(`:114`) 원자적 쓰기
- **주의점:** `control_path`(`:133`)는 소켓 경로가 `_MAX_SOCKET_PATH`(`:130`, 100바이트)를 넘으면 `<tmp>/darkpyonix-<uid>/`로 옮긴다(`:144-149`, POSIX만). Rust 사본 `dpx-kernel/src/home.rs`와 일치해야 한다.

### `_lock.py` (70줄)
- **목적:** `<home>/locks/<kernel_id>.lock`에 비차단 배타 잠금. 프로세스가 죽으면 OS가 해제 (헤더 docstring)
- **사용처:** `__main__.py:49`, `:190`

### `_peer.py` (57줄) · `_winpipe.py` (516줄)
- **목적:** 제어 채널 접근을 "같은 OS 계정"으로만 제한 (FR-A1). POSIX는 `SO_PEERCRED`류(`_peer.py:21`), Windows는 SID와 DACL(`_winpipe.py:32` `dacl_sddl`, `:208` `sid_of_process`)
- **주의점:** `_winpipe.py`는 ctypes로 Win32를 직접 호출한다(`_Win32` `:141`). 오류 코드 상수 `:127-138`. 수정 시 Windows 테스트(`tests/test_fr_a1_windows_pipe.py`)로만 검증 가능.

### `_events.py` (77줄)
- **목적:** 개수·바이트 한도의 스레드 안전 이벤트 링. `since(seq)`(`:58`)로 재전송, 잘린 경우 `oldest_seq` 반환
- **사용처:** `__main__.py:52`, `_server.py:27`

### `_model.py` (86줄)
| 이름 | 줄 | 설명 |
|---|---|---|
| `RUN_STATUSES` | 10 | `'queued'`, `'running'`, `'ok'`, `'error'`, `'interrupted'`, `'cancelled'`, `'crashed'` |
| `CELL_STATUSES` | 11 | `'ok'`, `'error'`, `'interrupted'` |
| `Run.summary()` | 79 | OpenAPI `RunSummary` 형태 |

### `_hostctx.py` (85줄) · `_host.py` (38줄)
- **목적:** 실행기가 모듈 전역을 설정하고, 런타임 API가 그것을 읽는 다리. 커널 밖에서는 모든 함수가 "plain python" 동작 (FR-F5)
- **주의점:** `_host.py`는 import 실패까지 삼키는 방어 계층이다. 런타임 API는 `_hostctx`를 직접 import하지 말고 `_host`를 거친다(`_markdown.py`, `_params.py`, `_misc.py`가 이렇게 함).

## 이 모듈을 수정할 때 주의할 점
- **Python 3.8 문법·표준 라이브러리만** (AGENTS.md 하드 제약). `tests/test_nfr_k2_stdlib_only.py`가 검사한다.
- 와이어 상수·kernel_id 계산·홈 경로는 Rust에 사본이 있다. [manager-core](manager-core.md) "Python 사본 대응표" 참조.
