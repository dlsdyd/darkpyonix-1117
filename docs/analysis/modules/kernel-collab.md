# 모듈: kernel-collab

- 경로: `darkpyonix/kernel/_collab.py`, `_wsync.py`, `_streams.py`, `_tokens.py`
- 규모: 파일 4개 / 약 2,000줄
- 레이어: 비즈니스 로직 (공동 편집) + 인프라 (매니저가 넘긴 스트림 처리)
- 한 줄 책임: 커널이 쥐고 있는 공유 문서(셀·잠금·presence), 커널 접근 토큰, 매니저가 넘겨준 SSE/WebSocket/long-poll 스트림을 직접 응답
- 의존하는 모듈: [kernel-infra](kernel-infra.md), [kernel-runtime-api](kernel-runtime-api.md) (`format`), [kernel-exec](kernel-exec.md) (`_runs`, `_document`)
- 이 모듈을 쓰는 모듈: [kernel-process](kernel-process.md) (`__main__.py:200-219`, `_server.py:29`), [kernel-exec](kernel-exec.md) (`by_of`)

## 공개 인터페이스

| 이름 | 종류 | 위치 | 설명 |
|---|---|---|---|
| `DocumentState` | 클래스 | `_collab.py:162` | 파싱된 파일을 메모리에 두고 PROTOCOL §4 연산 제공 |
| `DocumentState.handlers` | 메서드 | `_collab.py:276` | `doc.*`, `presence.*` DKP 연산 등록 |
| `by_of` | 함수 | `_collab.py:70` | `{client_id, user, nickname}` 실행 주체 표기 |
| `TokenStore` | 클래스 | `_tokens.py:54` | `tokens.issue/verify/revoke/list` |
| `StreamHub` | 클래스 | `_streams.py:124` | 매니저가 넘긴 소켓을 `adopt`(`:138`)해서 응답 |
| `WsSync` | 클래스 | `_wsync.py:186` | WebSocket 동기화 메시지 (PROTOCOL §5, "2025" 프로토콜) |
| `match_run_cells` | 함수 | `_wsync.py:152` | 실행 기록 ↔ 스냅샷 셀 매칭 |

## 파일별 요약

### `_collab.py` (890줄)
- **목적:** 공유 문서 상태. 편집은 디바운스 저장, 외부 파일 변경은 폴링으로 감지(`tick` `:253`, `_reload` `:760`)
- **주요 심볼:**
  | 이름 | 줄 | 설명 |
  |---|---|---|
  | `CAPABILITY_NAMES` | 63 | `('read','history','execute','edit','manage')` |
  | `CURSOR_MIN_INTERVAL` | 65 | `1/20`초 — 커서 이벤트 합치기 |
  | `snapshot` | 411 | `doc.snapshot` |
  | `cell_create/update/delete/move` | 423/499/526/553 | 셀 편집 (`base_version` 충돌 검사는 update) [추정] |
  | `lock` / `unlock` | 575 / 592 | 셀 잠금 |
  | `presence_update` / `presence_leave` | 666 / 706 | 참여자·커서 |
  | `flush` | 224 | 실행 전에 편집 내용을 파일에 씀 |
  | `_client` | 312 | 요청 `params`의 클라이언트 검증 (여러 핸들러에서 사용) |
- **부수효과:** 노트북 파일 쓰기, 이벤트 발행(`events.append`)
- **주의점:** `_reload`(`:760`)가 90줄. 외부 편집과 공동 편집이 충돌할 때의 규칙이 여기에 있다.

### `_tokens.py` (193줄)
- **목적:** 커널이 자기 토큰을 발급·검증·폐기·저장 (FR-K10). 매니저는 `tokens.*`로 묻기만
- **상수:** `CAPABILITIES` (`:25`), `KINDS = ('initial','login','share')` (`:26`), `STORE_NAME = 'tokens.json'` (`:27`)
- **저장:** 토큰은 SHA-256 해시(`_hash` `:30`)로만 저장
- **주의점:** 노트북 파일이 사라지면 저장소 삭제(`check_file` `:173`). 공유 폐기 시 `close_streams`로 열린 스트림을 닫는다(`__main__.py:216`에서 연결).

### `_streams.py` (469줄)
- **목적:** 매니저가 인증·인가 후 넘긴 연결(SSE, WebSocket, run-wait)을 커널이 직접 응답 (PROTOCOL §3.7)
- **상수:** `KINDS = ('events','ws','wait')` (`:29`), `KEEPALIVE = 15.0` (`:33`), `PRESENCE_HEARTBEAT = 10.0` (`:34`), `MAX_POLLS = 100` (`:36`), `_CAPABILITY_ORDER` (`:42`)
- **주의점:** `FINISHED` (`:40`)가 `__main__.py:34`와 중복. HTTP 상태 매핑 `_STATUS` (`:111`).

### `_wsync.py` (453줄)
- **목적:** RFC 6455 프레이밍(`FrameReader` `:62`, `frame` `:42`)과 2025 WebSocket 메시지 처리(`WsSync`)
- **상수:** `PING_INTERVAL = 30.0`, `IDLE_TIMEOUT = 60.0`, `MAX_MESSAGE = 1 MiB` (`:23-25`)
- **주의점:** `read` 권한이 없으면 셀 코드를 비운 이벤트를 보낸다(`_blank_sources` `:175`).

## 이 모듈을 수정할 때 주의할 점
- 능력(capability) 이름 5개가 4곳에 정의됨: `_collab.py:63`, `_tokens.py:25`, `_streams.py:42`, Rust `dpx-server/src/auth.rs:87`. 추가·변경 시 전부 + OpenAPI.
- 연산 이름 변경은 Rust 매니저 `api.rs`의 `call(...)` 문자열도 바꿔야 한다.
