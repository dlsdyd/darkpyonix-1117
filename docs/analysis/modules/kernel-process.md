# 모듈: kernel-process

- 경로: `darkpyonix/kernel/__main__.py`, `_server.py`, `_discovery.py`, `_launcher.py`, `_client.py`
- 규모: 파일 5개 / 약 1,540줄
- 레이어: 인프라 (프로세스 수명, IPC 서버)
- 한 줄 책임: 커널 프로세스를 띄우고, 제어 채널(DKP/1) 서버와 UDP 멀티캐스트 디스커버리 응답기를 돌린다
- 의존하는 모듈: [kernel-infra](kernel-infra.md), [kernel-exec](kernel-exec.md), [kernel-collab](kernel-collab.md)
- 이 모듈을 쓰는 모듈: 없음(진입점). `_client.py`는 테스트만 사용 (`_raw/orphans.md`)

## 공개 인터페이스

| 이름 | 종류 | 위치 | 설명 |
|---|---|---|---|
| `main` | 함수 | `__main__.py:252` | `--file` 인자, 파일 없으면 exit 2 |
| `Kernel` | 클래스 | `__main__.py:45` | 커널 프로세스 전체 조립 |
| `Kernel.handlers` | 메서드 | `__main__.py:164` | 기본 DKP 연산 표 |
| `ControlServer` | 클래스 | `_server.py:103` | selector 스레드 1개로 모든 클라이언트 처리 |
| `DiscoveryResponder` / `DiscoveryClient` / `discover` | 클래스/함수 | `_discovery.py:97` / `:222` / `:318` | UDP 멀티캐스트 |
| `launch`, `bootstrap_command`, `wait_for_announce` | 함수 | `_launcher.py:44`, `:35`, `:130` | 분리(detached) 실행 |
| `KernelClient` | 클래스 | `_client.py:31` | 블로킹 DKP 클라이언트 (헤더는 "CLI, manager, tests"라 하지만 프로덕션 사용처 없음) |

## 파일별 요약

### `__main__.py` (263줄)
- **목적:** 초기화·종료 순서는 `_raw/entrypoints.md` 참조
- **DKP 연산 → 처리:** `status`→`_status` `:95`, `run`→`_run` `:141`, `cancel/interrupt/restart/namespace`→`Executor`, `shutdown`→`_shutdown` `:160`, `runs.list/get/wait`→`:100/:104/:110`
- **주의점:** `_teardown`(`:237`)은 단계별 예외를 삼켜서 **잠금 해제가 반드시 실행**되게 한다. 순서를 바꾸지 말 것.

### `_server.py` (584줄)
- **목적:** DKP/1 제어 채널. 클라이언트별 송신 버퍼 → 느린 리더가 다른 클라이언트를 막지 않음. 핸들러는 워커 풀에서 실행
- **상수:** `MAX_OUTBUF = 64 MiB`, `MAX_PREAUTH_FRAME = 64 KiB`, `HANDLER_WORKERS = 4`, `MAX_PENDING_FDS = 4` (`:33-37`)
- **주의점:** POSIX는 Unix 소켓 + `SCM_RIGHTS`로 fd 수신(`_recv` `:69`), Windows는 `_winpipe`로 분기(`:133`). 같은 계정 검사(`_peer`).

### `_discovery.py` (327줄)
- **목적:** 루프백 UDP 멀티캐스트만으로 발견 (레지스트리 파일 없음, INTENT D4). `announce_now` (`:139`)는 상태 변화 시 호출
- **주의점:** Rust 사본 `dpx-kernel/src/discovery.rs`. 데이터그램 형식(`_encode` `:38`) 변경 시 양쪽 수정.

### `_launcher.py` (150줄)
- **목적:** DarkPyonix가 설치되지 않은 아무 인터프리터에서 커널 기동. `BOOTSTRAP_PRELUDE` (`:22`)가 이 파일 폴더에서 패키지를 로드
- **주의점:** Rust `dpx-kernel/src/launch.rs:67` `bootstrap`이 같은 부트스트랩 문자열을 만든다 (주석 "launcher.py `BOOTSTRAP`").

### `_client.py` (215줄)
- **목적:** 응답(요청 id별)과 이벤트(큐)를 백그라운드 리더가 분리하는 블로킹 클라이언트
- **사용처:** `tests/kernel_procs.py:71`, `tests/test_fr_k9_handoff.py:24` 등 테스트만

## 이 모듈을 수정할 때 주의할 점
- 하드 제약: 커널은 매니저 없이도 살아야 하고, 파일당 커널 1개는 **커널이** 강제(`__main__.py:190`).
- "정지 = 인터럽트". `shutdown`도 실행 중 셀을 인터럽트한 뒤 끝낸다(`_executor.py:262`). 프로세스 kill은 매니저의 `--force`만.
