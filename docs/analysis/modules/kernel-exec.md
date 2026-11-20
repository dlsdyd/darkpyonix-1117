# 모듈: kernel-exec

- 경로: `darkpyonix/kernel/_executor.py`, `_capture.py`, `_runs.py`, `_document.py`
- 규모: 파일 4개 / 약 2,100줄
- 레이어: 비즈니스 로직 (셀 실행) + 데이터 접근 (실행 기록 파일)
- 한 줄 책임: 메인 스레드에서 셀을 실행하고, 출력을 캡처해 이벤트로 내보내며, `__runs__/`에 nbformat 실행 기록을 남긴다
- 의존하는 모듈: [kernel-infra](kernel-infra.md), [kernel-runtime-api](kernel-runtime-api.md) (`format`, `_display`, `_figures`), [kernel-collab](kernel-collab.md) (`by_of`만)
- 이 모듈을 쓰는 모듈: [kernel-process](kernel-process.md) (`__main__.py`), 매니저(`_document.build_document`를 Python 서브프로세스로 실행, `dpx-kernel/src/launch.rs:229`)

## 공개 인터페이스

| 이름 | 종류 | 위치 | 설명 |
|---|---|---|---|
| `Executor` | 클래스 | `_executor.py:66` | 한 파일의 한 네임스페이스에서 실행 |
| `Executor.submit` | 메서드 | `_executor.py:128` | DKP `run` — 요청을 큐에 넣음 |
| `Executor.cancel` / `interrupt` / `restart` / `shutdown` | 메서드 | `:206` / `:219` / `:245` / `:262` | 큐 취소 / KeyboardInterrupt / soft·hard 재시작 / 종료 |
| `Executor.namespace` | 메서드 | `:286` | 사용자 변수 목록. busy일 때는 이름·타입만 |
| `Executor.run_forever` | 메서드 | `:377` | **메인 스레드에서만** 호출 |
| `RunStore` | 클래스 | `_runs.py:274` | 실행 기록 쓰기·목록·조회·충돌 복구 |
| `runs_dir_for` | 함수 | `_runs.py:58` | `<파일 폴더>/__runs__/<파일명>/` |
| `RunsMagic` | 클래스 | `_runs.py:683` | 커널 네임스페이스의 `__runs__` 객체 |
| `build_document` | 함수 | `_document.py:76` | 파일 + 최신 출력 → OpenAPI `Document` |
| `OutputRouter`, `StreamCapture`, `FdCapture` | 클래스/함수 | `_capture.py:33`, `:324`, `:346` | 출력 캡처 |

## 파일별 요약

### `_executor.py` (714줄)
- **목적:** 셀 실행 엔진. 스레드 모델은 헤더 docstring 참조(INTENT D12): 실행은 메인 스레드, 제어는 다른 스레드
- **주요 상수:** `INJECTED_NAMES` (`:48`) — 네임스페이스에 주입되는 이름(`__runs__`, `darkpyonix`, `__builtins__`, ...). `namespace()`가 이 이름과 `_`로 시작하는 이름을 숨긴다(`:302`); `REPR_LIMIT = 200` (`:54`)
- **입력 → 출력:** `RunRequest`(`_model.py`) → `Run` 레코드 + `run.*`/`output` 이벤트(`emit`)
- **의존:** `_capture`, `_collab.by_of`, `_display`, `_figures`, `_hostctx`, `_model`, `_protocol`, `format` (`:34-40`)
- **사용처:** `__main__.py:209`
- **부수효과:** 사용자 코드 실행, `sys.stdout/stderr`·fd 1/2 교체, `_hostctx` 전역 설정, `RunStore` 쓰기
- **주의점:** hard restart는 `hard_restart_requested`를 세우고 루프를 끝낸 뒤 `__main__.py:232-234`에서 `os.execv`로 프로세스를 갈아끼운다.

### `_capture.py` (510줄)
- **목적:** 출력 캡처. 생산자(메인·사용자 스레드·fd 리더)는 deque에 넣기만, `OutputRouter`의 데몬 스레드 하나가 기록·발행
- **주요 심볼:** `begin_cell`/`end_cell` (`:120`/`:126`), `flush` (`:132`, 큐 비울 때까지 블록), `FdCapture.sync` (`:457`)
- **상수:** `_POLL = 0.02`, `_IDLE_POLL = 0.1` (`:29-30`)
- **주의점:** `_loop`(`:142`)가 110줄짜리 단일 함수. 출력 순서·코얼레싱이 여기 모여 있다.

### `_runs.py` (748줄)
- **목적:** 실행 기록 `<dir>/__runs__/<file>/<run_id>.ipynb` + `index.json` (헤더 docstring, INTENT D8)
- **주요 상수:**
  | 이름 | 줄 | 값 | 의미 |
  |---|---|---|---|
  | `RUNS_DIRNAME` | 34 | `'__runs__'` | 폴더명 |
  | `INDEX_NAME` | 35 | `'index.json'` | |
  | `WRITE_INTERVAL` | 39 | `0.8` | 진행 중 기록 재작성 주기(초) |
  | `DEFAULT_OUTPUT_LIMIT` | 45 | 16 MiB | 넘는 스트림은 사이드카 `<run_id>.cell<i>.log`로 (FR-R5, `sidecar_name` `:180`) |
  | `OUTPUT_LIMIT_ENV` | 46 | `'DARKPYONIX_RUN_OUTPUT_LIMIT'` | 위 한도 환경 변수 |
  | `_RUN_ID_RE` | 49 | `YYYYMMDD-HHMMSS-xxxx` | |
- **주요 메서드:** `begin` `:301`, `update` `:309`, `finish` `:319`, `flush` `:330`, `list` `:439`, `get` `:459` (`"latest"`/`"current"` 지원), `recover_crashed` `:487`
- **부수효과:** 원자적 파일 쓰기(`_write_atomic` `:240`)
- **주의점:** run id 정규식이 Rust `dpx-server/src/util.rs:41` `is_run_id`와 중복.

### `_document.py` (123줄)
- **목적:** 매니저 `/document` 엔드포인트용. `__runs__/`를 직접 읽으므로 커널이 없어도 동작
- **매칭 규칙:** `_map_run` (`:42`) id → `source_sha256` → index 순. `MAX_RUNS_SCANNED = 20` (`:23`)
- **사용처:** 매니저가 Python으로 실행 (`dpx-kernel/src/launch.rs:229`), `_wsync.py:152` `match_run_cells`

## 이 모듈을 수정할 때 주의할 점
- 실행 기록 형식(`metadata.darkpyonix`)은 매니저 `api.rs:780` `run_summary`가 직접 파싱한다. 필드 이름을 바꾸면 Rust도 바꿔야 한다.
- 출력 캡처 변경은 Windows/POSIX 양쪽 fd 동작을 확인 (`tests/test_fr_x_executor.py`).
