# 모듈: manager-server

- 경로: `darkpyonix/manager/crates/dpx-server/src/`
- 규모: 파일 10개 / 약 4,500줄 (Rust). `api.rs` 한 파일이 1,979줄이라 하위 분할 불가 → 문서 안에서 "API"와 "인프라"로 나눔
- 레이어: API (HTTP) + 인증
- 한 줄 책임: `docs/api/manager.openapi.yaml`의 모든 연산을 `KernelBackend` 위에서 제공. 인증·권한·토큰·비밀번호, 긴 스트림을 커널에 인계, 리버스 프록시, TLS
- 의존하는 모듈: [manager-core](manager-core.md) (`dpx-core`)
- 이 모듈을 쓰는 모듈: [cli](cli.md) (`manager.rs`)

## 공개 인터페이스

| 이름 | 종류 | 위치 | 설명 |
|---|---|---|---|
| `serve` | 함수 | `lib.rs:461` | HTTP 서버 기동, 연결 수락 상태가 되면 반환 |
| `run_until_signal` | 함수 | `lib.rs:599` | SIGINT/SIGTERM 또는 유휴 종료까지 |
| `router` | 함수 | `lib.rs:355` | 라우트 표 (`:358-389`, `_raw/routes.md`) |
| `ServerConfig`, `Mode` | 구조체/enum | `lib.rs:83`, `:68` | `Ephemeral` / `Dedicated` |
| `AppState` | 구조체 | `api.rs:29` | 핸들러 공유 상태 |
| `Auth`, `Principal`, `Caps`, `Capability` | 구조체/enum | `auth.rs:596`, `:106`, `:53`, `:20` | 인증 주체·능력 |
| `ApiError`, `ApiResult` | 구조체/타입 | `error.rs:10`, `:130` | `{"error": {code, message, data?}}` |

## 주요 상수 (`lib.rs:55-66`)
| 이름 | 값 | 의미 |
|---|---|---|
| `DEFAULT_IDLE_TIMEOUT` | 120초 | 임시(ephemeral) 매니저 유휴 종료 |
| `DEFAULT_DEDICATED_PORT` | `46881` | 전용 매니저 포트 |
| `DEFAULT_SHARE_BASE` | `"https://darkpyonix.dev/s"` | 공유 링크 `<base>/<share_id>#<token>` |
| `PRESENCE_HEARTBEAT` | 10초 | 이벤트 스트림 열린 동안 presence 갱신 (커널은 30초 후 만료) |
| `MAX_BODY` | 64 MiB | 요청 본문 한도 |

## 파일별 요약 — API

### `api.rs` (1,979줄)
- **목적:** OpenAPI 연산 전부 + 연산별 필요 능력(FR-A3, FR-S8). 공동 편집 연산은 커널에 그대로 전달(얇음)
- **핸들러 위치:** `_raw/routes.md` §1 표
- **공통 헬퍼:** `require` `:146`(능력 검사), `check_user_permission` `:156`, `call` `:185`(DKP 호출), `parse_json` `:105`(검증 실패는 항상 400, 422 아님), `check_cell` `:297`, `client_value` `:239`, `with_rid` `:1454`(`X-DarkPyonix-Request` → `request_id`)
- **인계:** `plan_handoff` `:1903`, `HandoffKind` `:1872`(events/ws/wait)
- **비밀번호:** `check_pw` `:1018` (PBKDF2를 blocking 스레드에서), `check_basic` `:1041`
- **주의점:** 2025 호환 API(`create_share_token` `:1336`, `user_permission` 인자, `sync_ws` `:923`)와 현행 API가 공존.

### `error.rs` (130줄) · `sse.rs` (97줄) · `util.rs` (164줄) · `docs.rs` (29줄)
| 파일 | 요약 |
|---|---|
| `error.rs` | `From<DpxError> for ApiError` (`:68`) — DKP 코드 → HTTP 상태 매핑 |
| `sse.rs` | SSE 프레임 `id/event/data`. `replay_truncated`는 `id` 없음 (`frame` `:19`) |
| `util.rs` | id 정규식 `is_kernel_id` `:31`(`^k_[0-9a-f]{20}$`), `is_share_id` `:36`, `is_run_id` `:41`, `is_client_id` `:116`, `is_cell_id` `:121`; `write_private_atomic` `:76` |
| `docs.rs` | `/docs/` API 문서 페이지와 OpenAPI 파일을 바이너리에 내장 (계약 밖, NFR-M3 예외) |

## 파일별 요약 — 인프라

### `lib.rs` (618줄)
- **목적:** 서버 조립. 인증을 라우팅 **전에** 수행(`auth_gate` `:290`) → 익명은 항상 401
- **인증 예외 경로:** `is_open_auth_path` `:237` (로그인 등 자체 인증 경로)
- **토큰 위치:** Bearer 헤더, 단 events/WebSocket은 브라우저 제약으로 `?token=` 허용(`query_token` `:229`)
- **커널 토큰:** `kernel_principal` `:264` → 커널에 `tokens.verify` 문의
- **유휴 감시:** `Activity` `:145`, `ActivityGuard` `:171`; 인계된 스트림은 유휴 카운트 제외(`handoff.rs:25`)

### `auth.rs` (678줄)
- **목적:** 마스터 토큰(프로세스마다 랜덤, 설정 가능), 공유 토큰(전용 매니저만, SQLite `<home>/manager.db`), 비밀번호
- **주요 심볼:** `CAPABILITIES` `:87`, `parse_token_type` `:91`(`read+history` 형식, 정해진 순서여야 함), `hash_password` `:211`(`pbkdf2-sha256$<iter>$<salt>$<hash>`), `AttemptLimiter` `:260`(3회 무료 후 1초부터 두 배, 최대 5분), `ShareStore` `:317`
- **DB 테이블:** `shares`, `shares_revoked`, `manager_auth`, `manager_shared` (`:329-359`) → [04_DATA_MODEL.md](../04_DATA_MODEL.md)
- **주의점:** 토큰 원문은 저장하지 않음(SHA-256). PBKDF2·HMAC를 직접 구현(`HmacSha256` `:158`, `pbkdf2_sha256` `:195`).

### `handoff.rs` (487줄)
- **목적:** 긴 스트림을 커널에 넘김. 평문 TCP는 `intercept`로 요청 머리를 엿보고 소켓 자체를 넘김(PROTOCOL §3.7), TLS 등은 `pump_response` `:142`로 중계
- **주요 심볼:** `parse_head` `:45`, `token_of` `:89`, `build_request` `:107`(토큰 제거 후 커널에 전달)

### `proxy.rs` (247줄) · `tls.rs` (76줄)
- `proxy.rs`: nginx 대체 리버스 프록시(VS Code serve-web, ember 앞단). 계약 밖
- `tls.rs`: rustls + ALPN HTTP/2. `TlsConfig` `:18`의 ACME 변형은 **TODO: 미구현, `serve`가 거부** (`tls.rs:22`)

## 이 모듈을 수정할 때 주의할 점
- 엔드포인트 추가·변경 = `docs/api/manager.openapi.yaml` 먼저 (AGENTS.md: OpenAPI에 없는 엔드포인트는 존재하지 않음). 테스트가 제공 스키마와 파일을 비교한다.
- 능력 이름은 Python 커널 3곳과 중복 ([kernel-collab](kernel-collab.md)).
- 검증 테스트: `crates/dpx-server/tests/api.rs`, `auth.rs`, `collab.rs`, `handoff.rs`.
