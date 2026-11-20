# 데이터 모델

이 프로젝트의 영속 데이터는 4종: (1) 노트북 파일과 `__runs__/` 실행 기록, (2) 런타임 홈 `~/.darkpyonix/`, (3) 매니저 SQLite `manager.db`, (4) 허브 D1 + R2.

## 1. 노트북 파일 (`docs/FORMAT.md`)
- 형식: 유효한 Python. `# %% 제목 [타입]` 마커로 셀 분리, `# @key: value` 메타데이터 (`kern/format/_parser.py:25`, `:30`)
- 파싱 결과: `NotebookDocument.cells[0]` = preamble, 이후 `Cell(index, type, title, metadata, header, source)` (`kern/format/__init__.py:27`, `:54`)
- 셀 해시: `source_sha256` (`_parser.py:60`) — 실행 결과를 셀에 다시 붙일 때 사용

## 2. 실행 기록 `__runs__/` (`kern/_runs.py`, INTENT D8)
```
<노트북 폴더>/__runs__/<파일명>/
  index.json                      # 실행 요약 목록
  <run_id>.ipynb                  # 실행 1회 = nbformat 4.5 노트북
  <run_id>.cell<i>.log            # 출력 한도(16 MiB) 초과분 (FR-R5)
  tokens.json                     # 커널 접근 토큰(SHA-256 해시, 능력) — kern/_tokens.py:61
```
- `metadata.darkpyonix`: 실행 필드 (`to_notebook_meta` `_runs.py:535`). 매니저 `srv/api.rs:780` `run_summary`가 직접 읽음
- `RunSummary` (`kern/_model.py:79`): OpenAPI 스키마와 같은 모양

## 3. 런타임 홈 `~/.darkpyonix/` (`DARKPYONIX_HOME`, PROTOCOL §1)
| 경로 | 쓰는 곳 | 내용 |
|---|---|---|
| `locks/<kernel_id>.lock` | `kern/_lock.py:14` | 커널 1개 잠금 |
| `sockets/<kernel_id>.sock` | `kern/_home.py:133` | POSIX 제어 소켓 (길면 `<tmp>/darkpyonix-<uid>/`) |
| `\\.\pipe\darkpyonix-<user_tag>-<kernel_id>` | `kern/_winpipe.py:23` | Windows 제어 파이프 (파일 아님) |
| `kernels/<kernel_id>.log` | `kern/_launcher.py:41` (`kernels_dir` `_home.py:90`) | Python 런처로 띄운 커널 로그 |
| `managers/` | `kern/_home.py:98` `managers_dir` | Python 쪽 호출처 없음 (I-008) |
| `runtime/<version>-<hash>/darkpyonix/` | `dpx-kernel/launch.rs:32` | 매니저가 푼 커널 소스 |
| `manager.db` | `srv/auth.rs:317` | 아래 4 |
| `manager.log`, `manager.log.1` | `cli/discovery.rs:88` | 임시 매니저 stderr (회전) |


## 4. 매니저 DB `manager.db` (SQLite, `srv/auth.rs:329-359`)
```mermaid
erDiagram
  shares { TEXT share_id PK; TEXT kernel_id; TEXT permission; TEXT label; TEXT created_at; TEXT expires_at; INTEGER expires_unix; TEXT token_sha256 UK }
  shares_revoked { TEXT share_id PK; TEXT kernel_id; TEXT token_sha256 UK; INTEGER revoked_unix }
  manager_auth { TEXT key PK; TEXT value }
  manager_shared { TEXT share_id PK; TEXT token_type; TEXT capabilities; TEXT token_sha256 UK; TEXT created_at; TEXT revoked_at }
```
- `manager_auth`: 비밀번호 해시(`pbkdf2-sha256$...`, `auth.rs:211`) 등 키-값
- 전용(dedicated) 매니저에서만 공유 토큰 사용. 토큰 원문은 저장하지 않음

## 5. 허브 D1 (`darkpyonix/hub/worker/migrations/0001~0009`)
```mermaid
erDiagram
  accounts ||--o{ sessions : has
  accounts ||--o{ devices : owns
  accounts ||--o{ names : owns
  accounts ||--o{ notebooks : owns
  devices ||--o| records : publishes
  devices ||--o{ shares : hosts
  devices ||--o{ names : serves
  notebooks ||--o{ notebook_versions : has
  shares ||--o{ relay_passes : grants
  accounts { TEXT account_id PK; INTEGER github_id UK; TEXT github_login; INTEGER created_at; INTEGER devices_version }
  sessions { TEXT session_hash PK; TEXT account_id FK; INTEGER expires_at }
  oauth_transactions { TEXT state PK; TEXT code_verifier; TEXT return_to; INTEGER expires_at }
  device_links { TEXT link_id PK; TEXT user_code UK; TEXT endpoint_id; TEXT role; TEXT status; TEXT account_id FK; TEXT replace_endpoint_id }
  devices { TEXT endpoint_id PK; TEXT account_id FK; TEXT name; TEXT role; TEXT token_hash UK; INTEGER online; INTEGER revoked_at; TEXT resolve_token_hash; TEXT app; INTEGER readmit_until }
  records { TEXT endpoint_id PK; TEXT payload; INTEGER timestamp_us }
  shares { TEXT share_id PK; TEXT endpoint_id FK; INTEGER created_at }
  relay_passes { TEXT token_hash PK; TEXT share_id; INTEGER expires_at }
  names { TEXT name PK; TEXT endpoint_id FK; TEXT account_id FK }
  notebooks { TEXT notebook_id PK; TEXT account_id FK; TEXT title; TEXT visibility; TEXT share_id FK; INTEGER taken_down }
  notebook_versions { TEXT notebook_id PK; INTEGER version PK; TEXT source_sha256; TEXT run_id }
  notebook_reports { INTEGER report_id PK; TEXT notebook_id; TEXT ip_hash; TEXT reason }
```
- 상태값: `device_links.status` = `pending | approved | denied | claimed`; `devices.role` = `main_server | computer` (마이그레이션 주석)
- 노트북 본문은 R2 `NOTEBOOKS`, 키 `nb/<account>/<notebook>/<version>/<part>` (`hub/notebooks.ts:27`)
- 시간은 모두 Unix 초(INTEGER). 매니저 DB는 ISO 문자열 + 일부 Unix — 형식이 다름

## 6. 주고받는 주요 타입
| 타입 | 정의 | 사용 |
|---|---|---|
| OpenAPI `Kernel` | `dpx-core/src/lib.rs:45` `KernelInfo`, `srv/api.rs:190` `kernel_json` | 커널 목록·조회 |
| `RunSummary` | `kern/_model.py:79`, `srv/api.rs:780` | 실행 목록 |
| `Document` | `kern/_document.py:76` + `srv/api.rs:552` `merge_snapshot` | 문서+출력 |
| `Error` | `srv/error.rs:10` `{"error": {code, message, data?}}`; 허브는 `{"error", "code"?}` (`hub/http.ts:10`) | **매니저와 허브의 오류 모양이 다름** |
| DKP 프레임 | `kern/_protocol.py:91` | 길이 접두 JSON, `{"dkp":1, ...}` |
| 이벤트 | `{seq, type, time, data}` (`dpx-core/src/lib.rs:90`) | `run.*`, `cell.*`, `output`, `output.clear`, `kernel.status`, `doc.*`, `presence.*` |

## 7. 외부 연동
| 서비스 | 호출 위치 | 필요 설정 | 실패 처리 |
|---|---|---|---|
| GitHub OAuth | `hub/github.ts:70` `exchangeCode`, `:95` `fetchUser`, `:112` `revokeToken` | `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET` | 토큰 폐기는 best effort |
| Cloudflare DNS API | `hub/dns.ts:32` `CloudflareDns` | `CF_API_TOKEN`, `CF_ZONE_ID` | `DnsError` → `dnsFailure` `hub/names.ts:81` |
| 릴레이 호스트 | `hub/devices.ts:586` `disconnectFromRelay` → `/admin/disconnect` | `RELAY_ADMIN_URL`, `RELAY_SHARED_SECRET` | best effort |
| 허브 Worker (릴레이에서) | `darkpyonix/hub/server/src/state.rs:26`, `:41` | `RELAY_SHARED_SECRET` | [확인 필요] |
| iroh 릴레이 | `darkpyonix/hub/server/src/serve.rs` | TLS 인증서 | — |
| Python 인터프리터 | `dpx-kernel/launch.rs:135`, `:229` | `DARKPYONIX_PYTHON` | `DOCUMENT_TIMEOUT` `:18` |
