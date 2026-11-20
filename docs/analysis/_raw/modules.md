# 모듈 목록 (분석 순서 = 의존 그래프의 잎부터)

| 순서 | 모듈 | 책임 | 파일 수 | 줄 | 문서 |
|---|---|---|---|---|---|
| 1 | `kernel-infra` | Python 커널 공용 기반 (프로토콜·경로·잠금·IPC·모델) | 9 | 1286 | [modules/kernel-infra.md](../modules/kernel-infra.md) |
| 2 | `kernel-runtime-api` | 노트북이 import하는 런타임 API + 포맷 파서 | 11 | 1188 | [modules/kernel-runtime-api.md](../modules/kernel-runtime-api.md) |
| 3 | `kernel-exec` | 셀 실행·출력 캡처·실행 기록 | 4 | 2095 | [modules/kernel-exec.md](../modules/kernel-exec.md) |
| 4 | `kernel-collab` | 공동 편집 문서 상태·WS 동기화·스트림·토큰 | 4 | 2005 | [modules/kernel-collab.md](../modules/kernel-collab.md) |
| 5 | `kernel-process` | 커널 프로세스 수명·제어 서버·디스커버리·런처·클라이언트 | 5 | 1539 | [modules/kernel-process.md](../modules/kernel-process.md) |
| 6 | `manager-core` | dpx-core 공용 타입 + dpx-kernel (커널 탐색·실행·DKP 클라이언트) | 9 | 2770 | [modules/manager-core.md](../modules/manager-core.md) |
| 7 | `manager-server` | dpx-server: HTTP API·인증·핸드오프·프록시 | 10 | 4505 | [modules/manager-server.md](../modules/manager-server.md) |
| 8 | `cli` | darkpyonix CLI 바이너리 (명령·매니저 기동·출력) | 8 | 1823 | [modules/cli.md](../modules/cli.md) |
| 9 | `hub-worker` | darkpyonix.dev API (Cloudflare Worker, D1) | 17 | 2541 | [modules/hub-worker.md](../modules/hub-worker.md) |
| 10 | `hub-relay` | 릴레이 호스트 (Rust) | 5 | 524 | [modules/hub-relay.md](../modules/hub-relay.md) |
| 11 | `packaging` | PyPI 빌드 훅·검증 스크립트 | 3 | 327 | [modules/packaging.md](../modules/packaging.md) |

## 포함 파일

- `kernel-infra`: `darkpyonix/kernel/_protocol.py`, `darkpyonix/kernel/_home.py`, `darkpyonix/kernel/_peer.py`, `darkpyonix/kernel/_winpipe.py`, `darkpyonix/kernel/_lock.py`, `darkpyonix/kernel/_host.py`, `darkpyonix/kernel/_hostctx.py`, `darkpyonix/kernel/_events.py`, `darkpyonix/kernel/_model.py`
- `kernel-runtime-api`: `darkpyonix/kernel/__init__.py`, `darkpyonix/kernel/_markdown.py`, `darkpyonix/kernel/_params.py`, `darkpyonix/kernel/_binding.py`, `darkpyonix/kernel/_command.py`, `darkpyonix/kernel/_misc.py`, `darkpyonix/kernel/_display.py`, `darkpyonix/kernel/_figures.py`, `darkpyonix/kernel/_mplbackend.py`, `darkpyonix/kernel/format/__init__.py`, `darkpyonix/kernel/format/_parser.py`
- `kernel-exec`: `darkpyonix/kernel/_executor.py`, `darkpyonix/kernel/_capture.py`, `darkpyonix/kernel/_runs.py`, `darkpyonix/kernel/_document.py`
- `kernel-collab`: `darkpyonix/kernel/_collab.py`, `darkpyonix/kernel/_wsync.py`, `darkpyonix/kernel/_streams.py`, `darkpyonix/kernel/_tokens.py`
- `kernel-process`: `darkpyonix/kernel/__main__.py`, `darkpyonix/kernel/_server.py`, `darkpyonix/kernel/_discovery.py`, `darkpyonix/kernel/_launcher.py`, `darkpyonix/kernel/_client.py`
- `manager-core`: `darkpyonix/manager/crates/dpx-core/src/lib.rs`, `darkpyonix/manager/crates/dpx-kernel/build.rs`, `darkpyonix/manager/crates/dpx-kernel/src/discovery.rs`, `darkpyonix/manager/crates/dpx-kernel/src/dkp.rs`, `darkpyonix/manager/crates/dpx-kernel/src/home.rs`, `darkpyonix/manager/crates/dpx-kernel/src/ident.rs`, `darkpyonix/manager/crates/dpx-kernel/src/launch.rs`, `darkpyonix/manager/crates/dpx-kernel/src/lib.rs`, `darkpyonix/manager/crates/dpx-kernel/src/process.rs`
- `manager-server`: `darkpyonix/manager/crates/dpx-server/src/api.rs`, `darkpyonix/manager/crates/dpx-server/src/auth.rs`, `darkpyonix/manager/crates/dpx-server/src/docs.rs`, `darkpyonix/manager/crates/dpx-server/src/error.rs`, `darkpyonix/manager/crates/dpx-server/src/handoff.rs`, `darkpyonix/manager/crates/dpx-server/src/lib.rs`, `darkpyonix/manager/crates/dpx-server/src/proxy.rs`, `darkpyonix/manager/crates/dpx-server/src/sse.rs`, `darkpyonix/manager/crates/dpx-server/src/tls.rs`, `darkpyonix/manager/crates/dpx-server/src/util.rs`
- `cli`: `darkpyonix/manager/crates/darkpyonix/src/args.rs`, `darkpyonix/manager/crates/darkpyonix/src/client.rs`, `darkpyonix/manager/crates/darkpyonix/src/commands.rs`, `darkpyonix/manager/crates/darkpyonix/src/discovery.rs`, `darkpyonix/manager/crates/darkpyonix/src/main.rs`, `darkpyonix/manager/crates/darkpyonix/src/manager.rs`, `darkpyonix/manager/crates/darkpyonix/src/render.rs`, `darkpyonix/manager/crates/darkpyonix/src/sse.rs`
- `hub-worker`: `darkpyonix/hub/worker/src/app.ts`, `darkpyonix/hub/worker/src/devices.ts`, `darkpyonix/hub/worker/src/directory.ts`, `darkpyonix/hub/worker/src/dns.ts`, `darkpyonix/hub/worker/src/ed25519.ts`, `darkpyonix/hub/worker/src/env.ts`, `darkpyonix/hub/worker/src/github.ts`, `darkpyonix/hub/worker/src/http.ts`, `darkpyonix/hub/worker/src/index.ts`, `darkpyonix/hub/worker/src/names.ts`, `darkpyonix/hub/worker/src/notebooks.ts`, `darkpyonix/hub/worker/src/pages.ts`, `darkpyonix/hub/worker/src/pkarr.ts`, `darkpyonix/hub/worker/src/relay.ts`, `darkpyonix/hub/worker/src/shares.ts`, `darkpyonix/hub/worker/src/util.ts`, `darkpyonix/hub/worker/vitest.config.ts`
- `hub-relay`: `darkpyonix/hub/server/src/lib.rs`, `darkpyonix/hub/server/src/main.rs`, `darkpyonix/hub/server/src/routes.rs`, `darkpyonix/hub/server/src/serve.rs`, `darkpyonix/hub/server/src/state.rs`
- `packaging`: `packaging/hatch_build.py`, `packaging/check_versions.py`, `packaging/smoke.py`

미배정 파일: 없음

참고: `manager-server`(약 4,500줄)는 3,000줄 기준을 넘지만 `api.rs` 1개가 1,979줄이라 하위 폴더 분할이 불가능하여 한 모듈로 둔다. 문서 안에서 API/인프라 절로 나눈다.
