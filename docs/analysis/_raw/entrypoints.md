# 진입점

| 진입점 | 위치 | 실행 방법 |
|---|---|---|
| 커널 프로세스 | `darkpyonix/kernel/__main__.py:252` `main()` | `python -m darkpyonix --file X.py` (매니저 `dpx-kernel/src/launch.rs`가 띄움) |
| 런타임 API | `darkpyonix/kernel/__init__.py:14` `_EXPORTS` + `__getattr__` (PEP 562 지연 로딩) | 노트북 파일에서 `import darkpyonix` |
| CLI / 매니저 | `darkpyonix/manager/crates/darkpyonix/src/main.rs:17` `main()` | `darkpyonix <command>`; `darkpyonix manager`만 `manager::run_manager` (`manager.rs:48`) |
| 매니저 HTTP 서버 | `darkpyonix/manager/crates/dpx-server/src/lib.rs:461` `serve()` | `run_manager`에서 호출 |
| 허브 API | `darkpyonix/hub/worker/src/index.ts` → `app.ts:172` `handle()` | Cloudflare Worker (`wrangler deploy`) |
| 허브 릴레이 | `darkpyonix/hub/server/src/main.rs:52` `main()` | relay.darkpyonix.dev 호스트 |
| 빌드 훅 | `packaging/hatch_build.py`, `dpx-kernel/build.rs` (Python 소스를 바이너리에 내장) | 빌드 시 |

## 커널 초기화 순서 (`Kernel.run`, `__main__.py:189`)
1. `FileLock.acquire()` 실패 시 기존 커널을 discovery로 찾아 stderr에 JSON 출력 후 `EXIT_ALREADY_RUNNING` (`:190-197`) — "파일당 커널 1개" 강제
2. `_home.expose_package()` (`:199`)
3. `DocumentState` 생성 → collab 핸들러 등록 (`:200-202`)
4. `TokenStore` → 토큰 핸들러 등록 (`:203-204`)
5. `RunStore` + `recover_crashed()` (`:205-208`)
6. `Executor` 생성 (`:209-212`)
7. `ControlServer` 생성(지연 import) → `collab.start()`, `tokens.start()`, `server.start()` (`:214-219`)
8. `DiscoveryResponder.start()` (`:220-221`)
9. SIGTERM → `request_shutdown`, SIGHUP 무시 (`:223-225`)
10. `executor.run_forever()` 메인 스레드 점유 (`:228`)
11. 종료: `_teardown()` 순서 store.flush → discovery.stop(bye) → server.stop → collab.stop → tokens.stop → lock.release (`:237-250`). hard restart면 `os.execv`로 재실행 (`:232-234`)

## CLI 초기화 순서 (`main.rs:17`)
1. clap 파싱 (`args::Cli::parse`)
2. `manager` 서브커맨드면 `run_manager` 후 종료
3. 그 외: current-thread tokio 런타임 → `commands::dispatch` (`commands.rs`) → 매니저 HTTP API 호출
