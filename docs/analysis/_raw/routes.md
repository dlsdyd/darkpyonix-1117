# 라우트

이 프로젝트는 화면 라우트가 없다. 사용자 접점은 아래 5종이다.

## 1. 매니저 HTTP API (`docs/api/manager.openapi.yaml`)

등록: `darkpyonix/manager/crates/dpx-server/src/lib.rs:358-389`. 핸들러는 `dpx-server/src/api.rs`.

| 메서드 | 경로 | 핸들러 |
|---|---|---|
| GET | `/health` | `get_health` `api.rs:315` |
| GET | `/api/manager` | `get_manager` `api.rs:319` |
| GET | `/api/auth` | `login` `api.rs:1049` |
| PUT | `/api/auth/password` | `set_password` `api.rs:1063` |
| PUT | `/api/auth/tokens/master` | `reset_master_token` `api.rs:1086` |
| PUT | `/api/auth/tokens/shared/{token_type}` | `reset_shared_token` `api.rs:1095` |
| POST | `/api/kernels/{kernel_id}/tokens/initial` | `create_initial_token` `api.rs:1146` |
| POST | `/api/kernels/{kernel_id}/tokens/auth` | `kernel_login` `api.rs:1167` |
| GET | `/api/kernels/{kernel_id}/tokens/{token}/verify` | `verify_token` `api.rs:1187` |
| GET | `/api/kernels` | `list_kernels` `api.rs:343` |
| POST | `/api/kernels` | `start_kernel` `api.rs:365` |
| GET | `/api/kernels/{kernel_id}` | `get_kernel` `api.rs:390` |
| DELETE | `/api/kernels/{kernel_id}` | `shutdown_kernel` `api.rs:403` |
| POST | `/api/kernels/{kernel_id}/interrupt` | `interrupt_kernel` `api.rs:442` |
| POST | `/api/kernels/{kernel_id}/restart` | `restart_kernel` `api.rs:470` |
| GET | `/api/kernels/{kernel_id}/namespace` | `get_namespace` `api.rs:510` |
| GET | `/api/kernels/{kernel_id}/document` | `get_kernel_document` `api.rs:637` |
| GET | `/api/documents` | `get_document` `api.rs:661` |
| GET | `/api/kernels/{kernel_id}/runs` | `list_runs` `api.rs:684` |
| POST | `/api/kernels/{kernel_id}/runs` | `start_run` `api.rs:728` |
| GET | `/api/kernels/{kernel_id}/runs/{run_ref}` | `get_run` `api.rs:804` |
| DELETE | `/api/kernels/{kernel_id}/runs/{run_ref}` | `cancel_run` `api.rs:838` |
| GET | `/api/kernels/{kernel_id}/events` | `stream_events` `api.rs:861` |
| GET | `/api/ws/kernels/{kernel_id}` | `sync_ws` `api.rs:923` |
| POST | `/api/kernels/{kernel_id}/tokens/share` | `create_share_token` `api.rs:1336` |
| GET | `/api/kernels/{kernel_id}/shares` | `list_shares` `api.rs:1258` |
| POST | `/api/kernels/{kernel_id}/shares` | `create_share` `api.rs:1298` |
| DELETE | `/api/kernels/{kernel_id}/shares/{share_id}` | `revoke_share` `api.rs:1390` |
| POST | `/api/kernels/{kernel_id}/cells` | `create_cell` `api.rs:1482` |
| PATCH | `/api/kernels/{kernel_id}/cells/{cell_id}` | `update_cell` `api.rs:1512` |
| DELETE | `/api/kernels/{kernel_id}/cells/{cell_id}` | `delete_cell` `api.rs:1537` |
| POST | `/api/kernels/{kernel_id}/cells/{cell_id}/move` | `move_cell` `api.rs:1568` |
| PUT | `/api/kernels/{kernel_id}/cells/{cell_id}/lock` | `lock_cell` `api.rs:1602` |
| DELETE | `/api/kernels/{kernel_id}/cells/{cell_id}/lock` | `unlock_cell` `api.rs:1622` |
| PUT | `/api/kernels/{kernel_id}/presence` | `update_presence` `api.rs:1696` |
| DELETE | `/api/kernels/{kernel_id}/presence` | `leave_presence` `api.rs:1750` |
| GET | `/api/kernels/{kernel_id}/runs/{run_ref}/wait` | `wait_run` `api.rs:1806` |
| GET | `/docs` | `redirect` `docs.rs` |
| GET | `/docs/` | `index` `docs.rs` |
| GET | `/docs/{name}` | `file` `docs.rs` |

## 2. 허브 API (`docs/api/hub.openapi.yaml`)

등록: `darkpyonix/hub/worker/src/app.ts:105-147`. 릴레이 계열은 `darkpyonix/hub/server/src/routes.rs:10-17`.

| 메서드 | 경로 | 핸들러 |
|---|---|---|
| GET | `/health` | `health` `app.ts:50` |
| GET | `/config` | `config` `app.ts:55` |
| GET | `/.well-known/org.flathub.VerifiedApps.txt` | `flathubVerifiedApps` `app.ts:75` |
| GET | `/auth/login` | `login` `github.ts:41` |
| GET | `/auth/callback` | `callback` `github.ts:160` |
| POST | `/auth/logout` | `logout` `github.ts:200` |
| GET | `/me` | `me` `devices.ts:151` |
| POST | `/me/resolve-token` | `rotateResolveToken` `devices.ts:435` |
| GET | `/link` | `linkLanding` `app.ts:85` |
| POST | `/device-links` | `createLink` `devices.ts:167` |
| GET | `/device-links/{link_id}` | `getLink` `devices.ts:316` |
| POST | `/device-links/{link_id}/token` | `claimLink` `devices.ts:338` |
| GET | `/link-codes/{user_code}` | `getLinkCode` `devices.ts:247` |
| POST | `/link-codes/{user_code}` | `decideLinkCode` `devices.ts:265` |
| GET | `/devices` | `listDevices` `devices.ts:450` |
| GET | `/devices/{endpoint_id}` | `getDevice` `devices.ts:490` |
| PATCH | `/devices/{endpoint_id}` | `updateDevice` `devices.ts:504` |
| DELETE | `/devices/{endpoint_id}` | `removeDevice` `devices.ts:597` |
| POST | `/devices/{endpoint_id}/readmit` | `readmitDevice` `devices.ts:539` |
| GET | `/removed-devices` | `listRemovedDevices` `devices.ts:559` |
| GET | `/devices/{endpoint_id}/addresses` | `deviceAddresses` `directory.ts:76` |
| PUT | `/pkarr/{key}` | `pkarrPut` `directory.ts:16` |
| GET | `/pkarr/{key}` | `pkarrGet` `directory.ts:55` |
| POST | `/internal/relay/admit` | `admit` `relay.ts:20` |
| POST | `/internal/relay/presence` | `presence` `relay.ts:48` |
| POST | `/shares` | `publishShare` `shares.ts:28` |
| GET | `/shares/{share_id}` | `resolveShare` `shares.ts:50` |
| DELETE | `/shares/{share_id}` | `unpublishShare` `shares.ts:75` |
| GET | `/s/{share_id}` | `sharePage` `shares.ts:98` |
| GET | `/ash/` | `ashPage` `shares.ts:111` |
| GET | `/notebooks` | `listMine` `notebooks.ts:78` |
| POST | `/notebooks` | `createNotebook` `notebooks.ts:68` |
| GET | `/users/{github_login}/notebooks` | `listPublic` `notebooks.ts:83` |
| GET | `/notebooks/{notebook_id}` | `getNotebook` `notebooks.ts:89` |
| PATCH | `/notebooks/{notebook_id}` | `updateNotebook` `notebooks.ts:94` |
| DELETE | `/notebooks/{notebook_id}` | `deleteNotebook` `notebooks.ts:165` |
| POST | `/notebooks/{notebook_id}/report` | `reportNotebook` `notebooks.ts:188` |
| POST | `/admin/notebooks/{notebook_id}/takedown` | `takeDownNotebook` `notebooks.ts:211` |
| DELETE | `/admin/notebooks/{notebook_id}/takedown` | `restoreNotebook` `notebooks.ts:212` |
| POST | `/notebooks/{notebook_id}/versions` | `pushVersion` `notebooks.ts:108` |
| GET | `/notebooks/{notebook_id}/versions/{version}` | `readVersion` `notebooks.ts:145` |
| DELETE | `/notebooks/{notebook_id}/versions/{version}` | `deleteVersion` `notebooks.ts:159` |
| GET | `/notebooks/{notebook_id}/versions/{version}/source` | `readSource` `notebooks.ts:157` |
| GET | `/notebooks/{notebook_id}/versions/{version}/run` | `readRun` `notebooks.ts:158` |
| GET | `/names` | `listNames` `names.ts:31` |
| PUT | `/names/{name}` | `reserveName` `names.ts:40` |
| DELETE | `/names/{name}` | `releaseName` `names.ts:67` |
| PUT | `/names/{name}/acme-challenge` | `setAcmeChallenge` `names.ts:87` |
| DELETE | `/names/{name}/acme-challenge` | `clearAcmeChallenge` `names.ts:104` |
| GET | `/relay` | 릴레이 서버 `hub/server/src/serve.rs` (WebSocket 업그레이드, `serve.rs:1` 주석) |
| GET | `/health`, `/ping`, `/generate_204` | 릴레이 서버 `hub/server/src/routes.rs:10-11` |
| POST | `/admin/disconnect` | 릴레이 서버 `routes.rs:12` (`authorize` 미들웨어) |
| GET | `/ash/*` 정적 자산 | `app.ts:163` `env.ASSETS.fetch` |

## 3. CLI 명령

정의 `darkpyonix/manager/crates/darkpyonix/src/args.rs:25-52`, 디스패치 `commands.rs:117-238`.

| 명령 | 설명 (doc 원문) | 디스패치 |
|---|---|---|
| `run` | Run the file (or selected cells) and follow its output. Ctrl-C interrupts the run; a second Ctrl-C detaches | `commands.rs:117` `run()` |
| `stop` | Interrupt the running cell (KeyboardInterrupt) | `commands.rs:118` → `POST .../interrupt` |
| `status [FILE]` | Kernel status of a file, or the manager and its kernels without FILE | `commands.rs:133`, `:138` |
| `ps` | List running kernels | `commands.rs:155` |
| `logs` | Print a run log's outputs | `commands.rs:161` `logs()` |
| `attach` | Follow the kernel's live events until Ctrl-C | `commands.rs:162` |
| `vars` | Show the kernel's user variables | `commands.rs:176` |
| `restart` | Restart the kernel (clears the namespace) | `commands.rs:182` |
| `shutdown` | Shut the kernel down. Only --force kills the process | `commands.rs:188` |
| `kernel` | Start the kernel for a file without running anything | `commands.rs:195` |
| `share` | Create a share token for a file's kernel (dedicated manager only) | `commands.rs:211` |
| `manager` | Run a kernel manager in the foreground | `main.rs:19` → `manager.rs:48` `run_manager` |

## 4. 커널 와이어 연산 (DKP, `docs/PROTOCOL.md`)

| 연산 | 핸들러 등록 |
|---|---|
| `status`, `run`, `cancel`, `interrupt`, `restart`, `shutdown`, `namespace`, `runs.list`, `runs.get`, `runs.wait` | `darkpyonix/kernel/__main__.py:164` `Kernel.handlers()` |
| `doc.snapshot`, `doc.cell.create`, `doc.cell.update`, `doc.cell.delete`, `doc.cell.move`, `doc.lock`, `doc.unlock`, `presence.update`, `presence.leave` | `darkpyonix/kernel/_collab.py:276` `DocumentState.handlers()` |
| `tokens.issue`, `tokens.verify`, `tokens.revoke`, `tokens.list` | `darkpyonix/kernel/_tokens.py:87` `TokenStore.handlers()` |

## 5. 런타임 API (노트북 코드가 import)

`darkpyonix/kernel/__init__.py:14` `_EXPORTS`: `markdown`, `params`, `binding`, `run_command`, `uv`, `pip`, `display`, `run_parallel`, `run_cinterop`, `run_cppinterop`, `run_rustinterop`.
