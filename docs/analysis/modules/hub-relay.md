# 모듈: hub-relay

- 경로: `darkpyonix/hub/server/src/`
- 규모: 파일 5개 / 약 520줄 (Rust)
- 레이어: 인프라
- 한 줄 책임: `relay.darkpyonix.dev`의 iroh 릴레이 호스트. Workers가 못 하는 릴레이·QUIC 주소 발견(UDP 7842)과 프로브만 담당 (`lib.rs:3-7`)
- 의존하는 모듈: [hub-worker](hub-worker.md) (HTTP: `/internal/relay/admit`, `/internal/relay/presence`)
- 이 모듈을 쓰는 모듈: 없음 (진입점 `main.rs:52`)

## 파일별 요약
| 파일 | 줄 | 요약 |
|---|---|---|
| `lib.rs` | 206 | `HubConfig` `:34`, `start` `:118`(모든 리스너 바인드 후 반환), `RunningHub` `:69`(drop 시 정지). 헤더: "Partly superseded (INTENT D15, 2026-10-03)" |
| `main.rs` | 84 | 인자 `Args` `:18`, TLS 로드 `load_tls` `:45` |
| `serve.rs` | 92 | 단일 리스너: TLS(선택) → HTTP/1.1. `GET /relay`는 iroh 릴레이로, 나머지는 axum 라우터로 |
| `routes.rs` | 51 | `/health`, `/ping`, `/generate_204`, `POST /admin/disconnect`(`authorize` `:34`) |
| `state.rs` | 91 | 릴레이 입장 허가(`State` `:9`, 캐시), presence 보고(`:26` worker URL), `HubAccess` `:80`(iroh `AccessControl`) |

## 이 모듈을 수정할 때 주의할 점
- 헤더 문서와 크레이트 설명(`main.rs:18` "device registry, address directory, iroh relay, share links, HTTPS names")은 D15 이전 기능 범위를 말한다. 실제 담당은 릴레이뿐 → [05_ISSUES.md](../05_ISSUES.md) 문서 불일치 항목.
- `/admin/disconnect`는 `hub-worker` `devices.ts:586` `disconnectFromRelay`가 부른다.
