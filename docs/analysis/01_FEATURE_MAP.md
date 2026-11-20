# 기능 지도

## 사용법
이 프로젝트에는 화면(UI)이 없다. 사용자가 만나는 접점은 **CLI 명령**, **매니저 HTTP API**(IDE·에이전트·ash가 호출), **노트북 런타임 API**, **허브 API**(darkpyonix.dev)다.
피드백에 나온 명령어, 엔드포인트, 에러 문구를 이 문서에서 검색한 뒤 "핸들러"부터 오른쪽으로 따라가면 수정할 위치가 나온다.

경로 약어: `cli/` = `darkpyonix/manager/crates/darkpyonix/src/`, `srv/` = `darkpyonix/manager/crates/dpx-server/src/`, `kern/` = `darkpyonix/kernel/`, `hub/` = `darkpyonix/hub/worker/src/`.

공통 경로: CLI → `cli/client.rs` `Api` → 매니저 `srv/lib.rs:290` `auth_gate` → `srv/api.rs` 핸들러 → `srv/api.rs:185` `call` → `dpx-kernel` `Connection`(DKP/1) → 커널 `kern/_server.py` → 연산 핸들러.

---

## 접점 1: CLI (`darkpyonix <명령>`)

진입 `cli/main.rs:17`, 분기 `cli/commands.rs:89` `dispatch_inner`. 매니저 연결 `cli/discovery.rs:50` `connect`(없으면 임시 매니저 기동 `:114`).

| 사용자 동작 | 명령 | 핸들러 | 매니저 API | 서버 핸들러 | 커널 연산 / 로직 | 데이터 |
|---|---|---|---|---|---|---|
| 파일 실행 | `darkpyonix run FILE [--cells 1,3] [--param k=v] [--queue] [--detach]` | `run` `commands.rs:252` | ① `POST /api/kernels` ② `GET .../events` ③ `POST .../runs` | `start_kernel` `api.rs:365` → `start_run` `api.rs:728` | `RealBackend::ensure` → `run` → `Executor.submit` `_executor.py:128` | `__runs__/<file>/<run_id>.ipynb`, `index.json` |
| 실행 중 Ctrl-C | (run 중) | `CtrlC` `commands.rs:397`, `follow` `:438` | `POST .../interrupt` | `interrupt_kernel` `api.rs:442` | `interrupt` → `Executor.interrupt` `:219` | 실행 상태 `interrupted` |
| 두 번째 Ctrl-C | (run 중) | `follow` `commands.rs:438` | 없음 (분리) | — | 실행은 계속 | — |
| 셀 멈추기 | `darkpyonix stop FILE` | `commands.rs:118` | `POST .../interrupt` | `interrupt_kernel` `api.rs:442` | `Executor.interrupt` | — |
| 상태 보기 | `darkpyonix status [FILE]` | `commands.rs:133`, `:138` | `GET /api/kernels/{id}` / `GET /api/manager` | `get_kernel` `api.rs:390` / `get_manager` `:319` | 디스커버리 announce + `status` | — |
| 커널 목록 | `darkpyonix ps` | `commands.rs:155` → `render.rs:144` | `GET /api/kernels` | `list_kernels` `api.rs:343` | UDP 멀티캐스트 질의 | — |
| 실행 기록 출력 | `darkpyonix logs FILE` | `logs` `commands.rs:321` | `GET .../runs/{ref}` | `get_run` `api.rs:804` | `runs.get` → `RunStore.get` `_runs.py:459` | `.ipynb` 실행 기록 |
| 실시간 이벤트 | `darkpyonix attach FILE` | `commands.rs:162` | `GET .../events` (SSE) | `stream_events` `api.rs:861` | 커널에 인계 → `StreamHub.adopt` `_streams.py:138` | 이벤트 링 |
| 변수 보기 | `darkpyonix vars FILE` | `commands.rs:176` → `render.rs:164` | `GET .../namespace` | `get_namespace` `api.rs:510` | `namespace` → `Executor.namespace` `:286` | — |
| 재시작 | `darkpyonix restart FILE [--hard]` | `commands.rs:182` | `POST .../restart` | `restart_kernel` `api.rs:470` | `Executor.restart` `:245` (hard는 `os.execv` `__main__.py:234`) | 네임스페이스 초기화 |
| 종료 | `darkpyonix shutdown FILE [--force]` | `commands.rs:188` | `DELETE /api/kernels/{id}[?force]` | `shutdown_kernel` `api.rs:403` | `shutdown` → `Executor.shutdown` `:262`; `--force`면 유예 후 `backend.kill`(`api.rs:423-432`, **매니저가 커널을 죽이는 유일한 곳**) | 잠금 해제 |
| 커널만 기동 | `darkpyonix kernel FILE` | `commands.rs:195` | `POST /api/kernels` | `start_kernel` `api.rs:365` | `spawn_kernel` `launch.rs:135` | `<home>/runtime/<tag>/` |
| 공유 토큰 | `darkpyonix share FILE` | `commands.rs:211` | `POST .../tokens/share` | `create_share_token` `api.rs:1336` | `tokens.issue` `_tokens.py:95` | `tokens.json` |
| 매니저 실행 | `darkpyonix manager [--ephemeral]` | `main.rs:19` | — | `run_manager` `manager.rs:48` → `serve` `srv/lib.rs:461` | — | stdout JSON 1줄 `startup_line` `manager.rs:88` |

### CLI 종료 코드 (`cli/render.rs:5-10`)
| 코드 | 이름 | 의미 |
|---|---|---|
| 0 | `EXIT_OK` | 성공 |
| 1 | `EXIT_ERROR` | 오류 |
| 75 | `EXIT_BUSY` | 다른 실행이 진행 중 (`busy` `commands.rs:300`) |
| 130 | `EXIT_INTERRUPTED` | 인터럽트 또는 Ctrl-C 분리 |

---

## 접점 2: 매니저 HTTP API (`docs/api/manager.openapi.yaml`)

등록 `srv/lib.rs:358-389`. 모든 핸들러는 `srv/api.rs`. "능력"은 핸들러 안의 `require(..., Capability::X)` 기준.

### 커널 수명
| 동작 | 엔드포인트 | 핸들러 | 능력 | 커널 연산 / 백엔드 |
|---|---|---|---|---|
| 목록 | `GET /api/kernels` | `list_kernels` `:343` | Read | `backend.list` |
| 기동(있으면 그대로) | `POST /api/kernels` | `start_kernel` `:365` | Execute | `backend.ensure` (200 기존 / 201 새로) |
| 조회 | `GET /api/kernels/{id}` | `get_kernel` `:390` | Read | — |
| 종료 | `DELETE /api/kernels/{id}` | `shutdown_kernel` `:403` | Execute | `shutdown`, `force`면 kill |
| 인터럽트 | `POST .../interrupt` | `interrupt_kernel` `:442` | Execute | `interrupt` |
| 재시작 | `POST .../restart` | `restart_kernel` `:470` | Execute | `restart` |
| 변수 | `GET .../namespace` | `get_namespace` `:510` | History | `namespace` |

### 실행
| 동작 | 엔드포인트 | 핸들러 | 능력 | 커널 연산 |
|---|---|---|---|---|
| 실행 시작 | `POST .../runs` | `start_run` `:728` | Execute | `run` |
| 실행 목록 | `GET .../runs` | `list_runs` `:684` | History | `runs.list` |
| 실행 조회 | `GET .../runs/{run_ref}` (`latest`, `current` 가능) | `get_run` `:804` | History | `runs.get` |
| 실행 취소(대기열) | `DELETE .../runs/{run_ref}` | `cancel_run` `:838` | Execute | `cancel` |
| 완료 대기(long-poll) | `GET .../runs/{run_ref}/wait` | `wait_run` `:1806` | History | `runs.wait` / 커널 인계 |
| 이벤트 스트림 | `GET .../events` (SSE) | `stream_events` `:861` | Read(+History) | 커널 인계 `_streams.py` |

### 문서·공동 편집 (커널이 상태를 가짐, `kern/_collab.py`)
| 동작 | 엔드포인트 | 핸들러 | 능력 | 커널 연산 |
|---|---|---|---|---|
| 문서+출력 | `GET .../document` | `get_kernel_document` `:637` | Read(+History) | `doc.snapshot` + `build_document` |
| 파일 문서(커널 없이) | `GET /api/documents?path=` | `get_document` `:661` | Read | `launch.rs:229` → `_document.py:76` |
| 셀 추가 | `POST .../cells` | `create_cell` `:1482` | Edit | `doc.cell.create` `_collab.py:423` |
| 셀 수정 | `PATCH .../cells/{cell_id}` | `update_cell` `:1512` | Edit | `doc.cell.update` `:499` |
| 셀 삭제 | `DELETE .../cells/{cell_id}` | `delete_cell` `:1537` | Edit | `doc.cell.delete` `:526` |
| 셀 이동 | `POST .../cells/{cell_id}/move` | `move_cell` `:1568` | Edit | `doc.cell.move` `:553` |
| 셀 잠금/해제 | `PUT`/`DELETE .../cells/{cell_id}/lock` | `lock_cell` `:1602` / `unlock_cell` `:1622` | Edit | `doc.lock` `:575` / `doc.unlock` `:592` |
| 참여·커서 | `PUT`/`DELETE .../presence` | `update_presence` `:1696` / `leave_presence` `:1750` | Read | `presence.update` `:666` / `presence.leave` `:706` |
| WebSocket 동기화(2025) | `GET /api/ws/kernels/{id}` | `sync_ws` `:923` | Read | 커널 인계 → `WsSync` `_wsync.py:186` |

### 인증·토큰·공유
| 동작 | 엔드포인트 | 핸들러 | 능력 | 저장 위치 |
|---|---|---|---|---|
| 매니저 로그인 (Basic 비밀번호) | `GET /api/auth` | `login` `:1049` | 자체 인증 | `manager.db` `manager_auth` |
| 비밀번호 설정 | `PUT /api/auth/password` | `set_password` `:1063` | 마스터 | `manager_auth` |
| 마스터 토큰 재발급 | `PUT /api/auth/tokens/master` | `reset_master_token` `:1086` | Basic | `Auth::rotate_master` (`api.rs:1089`) |
| 공유 토큰 재발급 | `PUT /api/auth/tokens/shared/{token_type}` | `reset_shared_token` `:1095` | 마스터 | `manager_shared` |
| 커널 초기 토큰 | `POST .../tokens/initial` | `create_initial_token` `:1146` | — | 커널 `tokens.json` |
| 커널 로그인 | `POST .../tokens/auth` | `kernel_login` `:1167` | — | 커널 `tokens.json` |
| 토큰 검증 | `GET .../tokens/{token}/verify` | `verify_token` `:1187` | — | 커널 `tokens.verify` |
| 공유 목록/생성/폐기 | `GET`/`POST .../shares`, `DELETE .../shares/{share_id}` | `list_shares` `:1258`, `create_share` `:1298`, `revoke_share` `:1390` | Manage | `manager.db` `shares` + 커널 토큰 |
| 공유 토큰(2025) | `POST .../tokens/share` | `create_share_token` `:1336` | Manage | 커널 `tokens.json` |
| 상태 확인 | `GET /health`, `GET /api/manager` | `get_health` `:315`, `get_manager` `:319` | 없음 / — | — |

### 계약 밖 경로 (OpenAPI에 없음, 의도적)
| 경로 | 위치 | 비고 |
|---|---|---|
| `/docs`, `/docs/`, `/docs/{name}` | `srv/docs.rs` | API 문서 페이지 (NFR-M3 예외라고 `docs.rs` 헤더에 명시) |
| 프록시 경로 | `srv/proxy.rs` | 설정된 `ProxyRoute`만 |

---

## 접점 3: 노트북 런타임 API (`import darkpyonix`)

| 사용자 코드 | 정의 | 커널 안 동작 | 커널 밖(`python file.py`) |
|---|---|---|---|
| `darkpyonix.markdown(text)` | `kern/_markdown.py:12` | `text/markdown` 출력 | 아무것도 안 함 |
| `darkpyonix.params.get(name, default, choices=...)` | `kern/_params.py:111` | 실행 요청 `params` → `--name` → 기본값 | `--name` → 기본값 |
| `@darkpyonix.binding` | `kern/_binding.py:89` | 정의 시점 전역으로 재평가 | 같음 |
| `darkpyonix.run_command(cmd)` | `kern/_command.py:70` | 출력 줄 단위 스트리밍 | 같음 |
| `darkpyonix.pip.install(...)`, `darkpyonix.uv.add(...)` | `kern/_command.py:167`, `:192` | 현재 인터프리터에 설치 | 같음 |
| `darkpyonix.display(obj)` | `kern/_misc.py:9` → `_display.py:65` | `display_data` | `print(repr(obj))` |
| `darkpyonix.run_parallel(*aws)` | `kern/_misc.py:28` | 동시 실행 | 같음 |
| `darkpyonix.run_cinterop` 등 | `kern/_misc.py:55` | **예약(미구현)** | 같음 |
| `__runs__.latest`, `__runs__.current`, `__runs__.list()` | `kern/_runs.py:683` `RunsMagic` | 실행 기록 접근 | 없음 |
| `import matplotlib.pyplot` | `kern/_figures.py:34` | 셀 끝에 PNG 출력 | 일반 matplotlib |

---

## 접점 4: 허브 API (`darkpyonix.dev`, `docs/api/hub.openapi.yaml`)

등록 `hub/app.ts:105-147`. 전체 표: `_raw/routes.md` §2.

| 사용자 동작 | 화면/문구 | 엔드포인트 | 핸들러 | 데이터 |
|---|---|---|---|---|
| GitHub 로그인 | — | `GET /auth/login` → `GET /auth/callback` | `login` `github.ts:41`, `callback` `:160` | `oauth_transactions`, `accounts`, `sessions` |
| 기기 연결 요청 | — | `POST /device-links` | `createLink` `devices.ts:167` | `device_links` |
| 기기 승인 페이지 | `/link?code=XXXX-XXXX` | `GET /link` | `linkLanding` `app.ts:85` → `linkPage` `pages.ts:41` | — |
| 승인/거절 | 승인 페이지 버튼 | `POST /link-codes/{user_code}` | `decideLinkCode` `devices.ts:265` | `device_links`, `devices` |
| 기기 토큰 수령 | — | `POST /device-links/{id}/token` | `claimLink` `devices.ts:338` | `devices` |
| 기기 목록(롱폴) | — | `GET /devices[?wait=]` | `listDevices` `devices.ts:450` | `devices`, `accounts.devices_version` |
| 기기 제거/재허용 | — | `DELETE /devices/{id}`, `POST .../readmit` | `removeDevice` `:597`, `readmitDevice` `:539` | `devices` + 릴레이 끊기 |
| 주소 게시/조회 | — | `PUT`/`GET /pkarr/{key}` | `pkarrPut` `directory.ts:16`, `pkarrGet` `:55` | `records` |
| 공유 링크 열기 | ash 뷰어 | `GET /s/{share_id}` | `sharePage` `shares.ts:98` | `shares` |
| 공유 게시/해석 | — | `POST /shares`, `GET /shares/{id}` | `publishShare` `shares.ts:28`, `resolveShare` `:50` | `shares`, `relay_passes` |
| HTTPS 이름 | — | `PUT /names/{name}`, `PUT .../acme-challenge` | `reserveName` `names.ts:40`, `setAcmeChallenge` `:87` | `names`, Cloudflare DNS |
| 노트북 게시 | — | `POST /notebooks`, `POST .../versions` | `createNotebook` `notebooks.ts:68`, `pushVersion` `:108` | `notebooks`, `notebook_versions`, R2 |
| 노트북 신고/내림 | — | `POST .../report`, `POST /admin/notebooks/{id}/takedown` | `reportNotebook` `:188`, `takeDownNotebook` `:211` | `notebook_reports`, `notebooks.taken_down` |

---

## 사용자에게 보이는 에러 메시지

### CLI (`cli/`)
| 메시지 원문 | 위치 | 발생 조건 |
|---|---|---|
| `no kernel is running for {label}` | `commands.rs:110` | 매니저가 `not_found`를 돌려줌 (커널 없음, 또는 kernel_id 불일치 — I-001) |
| `no run log \`{which}\` for {label}` | `commands.rs:371` | `logs`에서 해당 실행 기록 없음 |
| `the kernel of {label} is gone` | `commands.rs:505` | 따라가던 중 커널이 사라짐 |
| `use --queue or \`darkpyonix stop {label}\`` | `commands.rs:301` | 바쁜 파일에 두 번째 `run` (exit 75) |
| `run {run_id} is queued at position {n}; waiting for its turn` | `commands.rs:291` | `--queue`로 대기열에 들어감 |
| `cannot reach the manager at {base}: {e}` | `client.rs:121` | 매니저 연결 실패 |
| `cannot locate the darkpyonix executable: {e}` | `discovery.rs:79` | 임시 매니저를 띄울 실행 파일을 못 찾음 |
| `cannot handle Ctrl-C: {e}` | `commands.rs:407` | 시그널 핸들러 설치 실패 |
| `darkpyonix manager: {msg}` | `main.rs:23` | 매니저 기동 실패 |

### 매니저 API (`srv/api.rs`, 예시)
| 메시지 원문 | 위치 | 발생 조건 |
|---|---|---|
| `cells, cell_ids: give one of them, not both` | `api.rs:741` | 실행 요청에 둘 다 지정 |
| `cells: mode 'cells' needs a non-empty 'cells' or 'cell_ids' list` | `api.rs:745` | `mode=cells`인데 셀 목록 없음 |
| 오류 코드 → HTTP 상태 | `error.rs:68` | DKP 코드(`bad_request`, `busy`, `not_found`, ...) 매핑 |

### 커널 (`kern/`)
| 메시지 원문 | 위치 | 발생 조건 |
|---|---|---|
| `darkpyonix-kernel: no such file: {path}` | `__main__.py:257` | `--file`이 없는 파일 |
| `no cell {cid!r}` | `_collab.py:296`, `:337` | 없는 셀 id |
| `kind must be one of initial, login, share` | `_tokens.py:99` | 잘못된 토큰 종류 |

---

## 연결되지 않은 API
CLI가 직접 호출하지 않는 매니저 API(IDE·ash·ember용): `/api/documents`, `.../document`, `.../cells*`, `.../presence`, `/api/ws/kernels/{id}`, `.../runs/{ref}/wait`, `/api/auth*`, `.../tokens/initial`, `.../tokens/auth`, `.../tokens/{token}/verify`, `.../shares*`. 호출자는 sibling 저장소(`darkpyonix-ember`, `darkpyonix-ash`, `vscode-darkpyonix`)이며 이 저장소에서 확인 불가 [확인 필요].
