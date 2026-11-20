# 이름 사전

> 도메인별로 묶었다. Ctrl+F로 이름을 검색하세요. 경로 약어는 [01_FEATURE_MAP.md](01_FEATURE_MAP.md)와 같다: `cli/`, `srv/`, `kern/`, `hub/`. `dpx-kernel/`은 `darkpyonix/manager/crates/dpx-kernel/src/`.

## 식별자·경로
| 이름 | 종류 | 정의 위치 | 의미 | 값/형태 | 주의 |
|---|---|---|---|---|---|
| `kernel_id` | 문자열 | `kern/_protocol.py:83` `kernel_id_for` | 파일 하나의 커널 id | `k_` + SHA-256(정규 경로) 앞 20 hex | Rust 사본 2개: `dpx-kernel/ident.rs:13`, `cli/discovery.rs:196`(정규화 방식 다름, I-001) |
| `canonical_path` | 함수 | `kern/_protocol.py:75` | `realpath(abspath())` + Windows `normcase` | 문자열 | Rust `dpx-kernel/ident.rs:24` |
| `run_id` | 문자열 | `kern/_protocol.py:179` `new_run_id` | 실행 1회 id | `YYYYMMDD-HHMMSS-xxxx` (UTC) | 검증 정규식 2곳: `kern/_runs.py:49`, `srv/util.rs:41` |
| `run_ref` | 경로 인자 | `srv/api.rs:771` `check_run_ref` | 실행 지정 | `run_id` 또는 `"latest"`, `"current"` | |
| `share_id` | 문자열 | `srv/util.rs:36` | 공유 id | `s_` + 16 hex | 허브 `hub/util.ts:93` `SHARE_ID_RE` |
| `client_id` | 헤더 `X-DarkPyonix-Client` | `srv/util.rs:116` | 공동 편집 참여자 | `^[A-Za-z0-9_-]{8,64}$` | |
| `cell_id` | 문자열 | `srv/util.rs:121` | 셀 id | `^[A-Za-z0-9_.:-]{1,64}$` | |
| `user_tag` | 문자열 | `kern/_home.py:157` | OS 계정 해시 (디스커버리에서 같은 사용자 판별) | SHA-256(account) 앞 16 hex | Rust `dpx-kernel/home.rs:141` |
| `account` | 함수 | `kern/_peer.py:46` (`kern/_home.py:151`은 이를 호출) | OS 계정 | `uid:<n>` / `sid:<S-1-5-...>` | Rust `dpx-kernel/home.rs:53` |
| `__runs__` | 폴더/객체 | `kern/_runs.py:34`, `:683` | 실행 기록 폴더, 네임스페이스 매직 | `<파일 폴더>/__runs__/<파일명>/` | Rust `srv/util.rs:126` |
| `DARKPYONIX_HOME` 하위 | 폴더 | `kern/_home.py` | `locks/`, `sockets/`, `kernels/`, `managers/`, `runtime/<tag>/`, `manager.db`, `manager.log` | | `_home.py:90-125`, `launch.rs:32` |

## 상태값
| 이름 | 정의 위치 | 값 | 주의 |
|---|---|---|---|
| `KERNEL_STATUSES` | `kern/_protocol.py:49` | `'starting'`, `'idle'`, `'busy'`, `'stopping'` | 런처는 `idle`/`busy`가 되면 준비 완료로 봄 (`_launcher.py:29` `READY_STATUSES`) |
| `RUN_STATUSES` | `kern/_model.py:10` | `'queued'`, `'running'`, `'ok'`, `'error'`, `'interrupted'`, `'cancelled'`, `'crashed'` | 종료 상태 `FINISHED`가 `__main__.py:34`, `_streams.py:40`에 중복 |
| `CELL_STATUSES` | `kern/_model.py:11` | `'ok'`, `'error'`, `'interrupted'` | |
| `ERROR_CODES` | `kern/_protocol.py:43` | `'bad_request'`, `'unknown_method'`, `'busy'`, `'not_found'`, ... | HTTP 매핑 `srv/error.rs:68`, `kern/_streams.py:111` `_STATUS` |
| `on_busy` | 실행 요청 필드 | `srv/api.rs:706` `OnBusy` | `"reject"` / `"queue"` | CLI `--queue` (`cli/commands.rs:255`) |
| `mode` | 실행 요청 필드 | `srv/api.rs:699` `RunMode` | `"all"` / `"cells"` | |
| `Mode` | enum | `srv/lib.rs:68` | `Ephemeral`(CLI가 띄움, 127.0.0.1, 유휴 종료) / `Dedicated`(전용, 공유 토큰 가능) | |
| `KINDS` (토큰) | `kern/_tokens.py:26` | `'initial'`, `'login'`, `'share'` | |
| `KINDS` (스트림) | `kern/_streams.py:29` | `'events'`, `'ws'`, `'wait'` | Rust `srv/api.rs:1872` `HandoffKind` |
| `ROLES` (기기) | `hub/devices.ts:98` | 기기 역할 (`client` 등) | 계정당 main server 1개 |
| 셀 타입 `KNOWN_TYPES` | `kern/format/__init__.py:19` | `'preamble'`, `'code'`, `'markdown'`, `'argparse'`, `'binding'`, ... | 별칭 `TYPE_ALIASES` `:24` |

## 권한 (capability)
| 이름 | 정의 위치 | 값 | 주의 |
|---|---|---|---|
| `CAPABILITIES` / `CAPABILITY_NAMES` | `srv/auth.rs:87`, `kern/_collab.py:63`, `kern/_tokens.py:25`, `kern/_streams.py:42` | `read`, `history`, `execute`, `edit`, `manage` | **4곳 중복**. 순서가 정규 순서(`parse_token_type` `auth.rs:91`이 순서 검사) |
| `Principal` | `srv/auth.rs:106` | 호출 주체 (마스터 = 전부, 또는 커널 하나에 묶인 토큰) | |
| `require` | `srv/api.rs:146` | 핸들러의 능력 검사 | 엔드포인트별 능력은 기능 지도 표 |
| `user_permission` | `srv/api.rs:156` | 2025 API 인자, 연산이 요구할 능력 지정 | |

## 커널 핵심 객체
| 이름 | 종류 | 위치 | 의미 |
|---|---|---|---|
| `Kernel` | 클래스 | `kern/__main__.py:45` | 커널 프로세스 조립체 |
| `Executor` | 클래스 | `kern/_executor.py:66` | 셀 실행 (메인 스레드) |
| `ControlServer` | 클래스 | `kern/_server.py:103` | DKP/1 서버 |
| `DocumentState` | 클래스 | `kern/_collab.py:162` | 공유 문서 |
| `TokenStore` | 클래스 | `kern/_tokens.py:54` | 커널 토큰 |
| `StreamHub` | 클래스 | `kern/_streams.py:124` | 인계받은 스트림 |
| `RunStore` | 클래스 | `kern/_runs.py:274` | 실행 기록 |
| `EventLog` | 클래스 | `kern/_events.py:17` | 이벤트 링 |
| `DiscoveryResponder` | 클래스 | `kern/_discovery.py:97` | 디스커버리 응답 |
| `OutputRouter` | 클래스 | `kern/_capture.py:33` | 출력 라우팅 스레드 |
| `FileLock` | 클래스 | `kern/_lock.py:19` | 파일당 커널 1개 잠금 |
| `DKPError` | 클래스 | `kern/_protocol.py:52` | 와이어 오류 |

## 매니저 핵심 객체
| 이름 | 종류 | 위치 | 의미 |
|---|---|---|---|
| `KernelBackend` | 트레이트 | `dpx-core/src/lib.rs:119` | HTTP ↔ 커널 계약 |
| `RealBackend` | 구조체 | `dpx-kernel/lib.rs:77` | 실구현 |
| `Connection` | 구조체 | `dpx-kernel/dkp.rs:424` | 커널당 공유 DKP 연결 |
| `AppState` | 구조체 | `srv/api.rs:29` | 핸들러 공유 상태 |
| `Auth` | 구조체 | `srv/auth.rs:596` | 토큰 검사 (마스터 → 공유 → 커널) |
| `ShareStore` | 구조체 | `srv/auth.rs:317` | `manager.db` |
| `AttemptLimiter` | 구조체 | `srv/auth.rs:260` | 비밀번호 오답 지연 |
| `Api` | 구조체 | `cli/client.rs:50` | CLI의 매니저 클라이언트 |
| `CtrlC` | 구조체 | `cli/commands.rs:397` | Ctrl-C 처리 |

## 기능 지도 핸들러·서비스 함수
매니저 핸들러(`srv/api.rs`의 `get_health`…`wait_run`)와 허브 핸들러(`hub/*.ts`)는 이름·줄이 [_raw/routes.md](_raw/routes.md)에 전부 있다. CLI 함수: `dispatch` `cli/commands.rs:79`, `ensure_kernel` `:243`, `run` `:252`, `busy` `:300`, `logs` `:321`, `follow` `:438`, `handle_event` `:517`, `connect` `cli/discovery.rs:50`, `spawn_ephemeral` `:114`, `resolve_file` `:189`, `run_manager` `cli/manager.rs:48`.

## 설정 상수 (바꿀 법한 것)
| 이름 | 위치 | 값 | 의미 | 함께 바꿀 곳 |
|---|---|---|---|---|
| `DISCOVERY_GROUP`, `DISCOVERY_PORT` | `kern/_protocol.py:24-25` | `239.255.68.80`, `46880` | 디스커버리 멀티캐스트 | `dpx-kernel/discovery.rs:30-31` |
| `ANNOUNCE_INTERVAL`, `STALE_AFTER` | `kern/_protocol.py:27-28` | 5초, 15초 | announce 주기, 만료 | `dpx-kernel/discovery.rs:34` |
| `MAX_FRAME` | `kern/_protocol.py:32` | 64 MiB | 프레임 한도 | `dpx-kernel/dkp.rs:21` |
| `EXIT_ALREADY_RUNNING` | `kern/_protocol.py:41` | 3 | 같은 파일 커널이 이미 있음 | 커널 `__main__.py:196`만 반환. Rust 쪽에는 이 코드를 해석하는 곳이 없음(grep 0건) — 매니저는 디스커버리 announce로 판단 |
| `WRITE_INTERVAL` | `kern/_runs.py:39` | 0.8초 | 실행 기록 재작성 주기 | |
| `DEFAULT_OUTPUT_LIMIT` | `kern/_runs.py:45` | 16 MiB | 셀 출력 상한(넘으면 사이드카) | 환경 변수 `DARKPYONIX_RUN_OUTPUT_LIMIT` |
| `INTERRUPT_GRACE` | `kern/_command.py:19` | 5초 | `run_command` 인터럽트 유예 | |
| `TEXT_PLAIN_LIMIT` | `kern/_display.py:14` | 100000 | `text/plain` 길이 | |
| `HANDLER_WORKERS` | `kern/_server.py:36` | 4 | DKP 핸들러 스레드 | |
| `DEFAULT_IDLE_TIMEOUT` | `srv/lib.rs:56`, `cli/manager.rs:24` | 120초 | 임시 매니저 유휴 종료 | 두 곳 |
| `DEFAULT_DEDICATED_PORT` | `srv/lib.rs:57` | 46881 | 전용 매니저 포트 | |
| `DEFAULT_SHARE_BASE` | `srv/lib.rs:58` | `https://darkpyonix.dev/s` | 공유 링크 | 허브 `/s/{id}` |
| `PRESENCE_HEARTBEAT` | `srv/lib.rs:62`, `kern/_streams.py:34` | 10초 | presence 갱신 | 두 곳 |
| `MAX_BODY` | `srv/lib.rs:63` | 64 MiB | HTTP 본문 한도 | |
| `EXIT_BUSY` / `EXIT_INTERRUPTED` | `cli/render.rs:8`, `:10` | 75 / 130 | CLI 종료 코드 | SPEC FR-C2 |
| `LIMITS` | `hub/notebooks.ts:9` | 객체 | 허브 노트북 한도 | 분당 한도는 `wrangler.toml` |
| `LINK_TTL_SECS` | `hub/devices.ts:40` | | 기기 연결 승인 유효 시간 | |
| `SESSION_TTL_SECS` | `hub/github.ts:27` | | 로그인 세션 유효 시간 | |
| `__version__` | `kern/__init__.py:12` | `"0.3.0"` | 버전 정본 | `darkpyonix/manager/Cargo.toml`, `hub/worker/package.json` |

## 환경 변수
| 이름 | 사용 위치 | 용도 | 필수 여부 |
|---|---|---|---|
| `DARKPYONIX_HOME` | `kern/_home.py:17`, `dpx-kernel/home.rs:13`, `cli/discovery.rs:32`, `srv/lib.rs:134` | 런타임 홈 (기본 `~/.darkpyonix`) | 선택. 테스트는 `.scratch/`로 지정 |
| `DARKPYONIX_PYTHON` | `dpx-kernel/launch.rs:94`, `dpx-kernel/lib.rs:48` | 커널을 띄울 인터프리터 | 선택 (없으면 PATH의 `python3`/`python`) |
| `DARKPYONIX_MANAGER_URL` + `DARKPYONIX_TOKEN` | `cli/discovery.rs:18-19`, `:47` | CLI가 쓸 기존 매니저 | 선택 (없으면 임시 매니저 기동) |
| `DARKPYONIX_MANAGER_TOKEN` | `cli/manager.rs:65` | 매니저의 마스터 토큰 고정 | 선택 (없으면 매번 랜덤). **CLI 쪽 이름(`DARKPYONIX_TOKEN`)과 다름** |
| `DARKPYONIX_RUN_OUTPUT_LIMIT` | `kern/_runs.py:46`, `:65` | 셀 출력 상한(바이트) | 선택 |
| `DARKPYONIX_SELF_EXE` | `cli/discovery.rs:13` | 임시 매니저로 띄울 실행 파일 (테스트·래퍼) | 선택 |
| `DARKPYONIX_BINARY`, `DARKPYONIX_WHEEL_TAG` | `packaging/hatch_build.py:27-28` | 플랫폼 wheel 빌드 | 빌드 시 |
| `MPLBACKEND` | `kern/_figures.py:22` | 사용자가 정하면 커널 백엔드를 쓰지 않음 | 선택 |
| 허브 `API_URL`, `FRONT_URL`, `ZONE`, `RELAY_URL`, `RELAY_ADMIN_URL`, `GITHUB_ALLOWED_IDS` | `darkpyonix/hub/worker/wrangler.toml:14-19`, `hub/env.ts:15-30` | 허브 설정 | 필수(앞 4개) |
| 허브 비밀 `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`, `RELAY_SHARED_SECRET`, `CF_API_TOKEN`, `CF_ZONE_ID`, `OPERATOR_SECRET`, `FLATHUB_VERIFICATION_TOKEN` | `hub/env.ts:13-47` | 외부 연동 비밀 (wrangler secret) | 기능별 선택 (없으면 해당 기능 비활성 [추정: `?` 선택 타입]) |

## 이름 정리 후보
| 현재 이름 | 위치 | 실제 의미 | 제안 |
|---|---|---|---|
| `KINDS` (2개) | `kern/_tokens.py:26`, `kern/_streams.py:29` | 토큰 종류 / 스트림 종류 | `TOKEN_KINDS`, `STREAM_KINDS` |
| `FINISHED` (2개) | `kern/__main__.py:34`, `kern/_streams.py:40` | 종료된 실행 상태 | `_model.py`에 하나로 (`FINISHED_RUN_STATUSES`) |
| `home` / `default_home` (Rust 3개) | `cli/discovery.rs:31`, `srv/lib.rs:134`, `dpx-kernel/home.rs:12` | 같은 함수 | `dpx-kernel::home::default_home` 하나로 |
| `kernel_id_for` (CLI) | `cli/discovery.rs:196` | 정규화 없는 사본 | `dpx-kernel::ident::kernel_id_for` 사용 |
| `concorrunt` | `kern/format/__init__.py:24` | `concurrent` 오타 별칭 | 유지 (예시 파일 호환, `docs/FORMAT.md:75`) |
