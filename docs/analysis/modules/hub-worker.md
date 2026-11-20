# 모듈: hub-worker

- 경로: `darkpyonix/hub/worker/` (`src/`, `migrations/`, `wrangler.toml`)
- 규모: 소스 16개 + 설정 1개 / 약 2,540줄 (TypeScript), D1 마이그레이션 9개
- 레이어: API (Cloudflare Worker) + 데이터 접근 (D1 SQLite, R2)
- 한 줄 책임: `darkpyonix.dev` 허브 API — GitHub 로그인, 기기 연결·목록, 주소 디렉터리(pkarr), 공유 링크, HTTPS 이름(ACME DNS-01), 노트북 게시
- 의존하는 모듈: 없음 (Rust 쪽과 HTTP로만). [hub-relay](hub-relay.md)가 이 API의 `/internal/relay/*`를 호출
- 이 모듈을 쓰는 모듈: 외부 클라이언트(ember, ash, 매니저 기기), [hub-relay](hub-relay.md)

## 공개 인터페이스
라우트 표는 `_raw/routes.md` §2 (`app.ts:105-147` `ROUTES`). 진입 `index.ts` → `handle` (`app.ts:172`), 크론 `cleanup` (`app.ts:193`, 매시간 만료 행 삭제).

## 파일별 요약 (팬인 순)

### `env.ts` (62줄) — 팬인 10
- `Env` (`:7`): 바인딩 `DB`(D1), `NOTEBOOKS`(R2), `ASSETS`, 레이트리미터 4개(`WRITE_`/`PUSH_`/`CREATE_`/`READ_LIMITER`), 비밀값 `GITHUB_CLIENT_SECRET`, `RELAY_SHARED_SECRET`, `CF_API_TOKEN`, `OPERATOR_SECRET` 등 → [02_GLOSSARY.md](../02_GLOSSARY.md) 환경 변수
- `Deps` (`:51`): 테스트가 바꿔 끼우는 네트워크·시계·DNS

### `util.ts` (134줄) — 팬인 9
- 토큰: `newToken` `:62`, `hashToken` `:72`(SHA-256 hex로 저장), `ctEq` `:77`
- 검증 정규식: `ENDPOINT_ID_RE` `:92`, `SHARE_ID_RE` `:93`, `ACME_VALUE_RE` `:94`, `USER_CODE_RE` `:109`; `RESERVED_NAMES` `:98`; `safeReturnTo` `:131`(오픈 리다이렉트 방지)

### `http.ts` (208줄) — 팬인 6
- `ApiError` `:10`(`{"error", "code"?}`), `json`/`html`/`redirect`, 쿠키(`SESSION_COOKIE` `:88`, `__Host-` 접두)
- 인증 주체: `principal` `:128`(기기 토큰 Bearer 또는 브라우저 세션), `requireAccountAdmin` `:170`, `resolvePrincipal` `:182`(pkarr GET 전용 resolve 토큰), `requireDevice` `:200`, `corsOrigin` `:206`

### `devices.ts` (622줄)
- **목적:** RFC 8628(디바이스 인가) 방식 + 키 소유 증명으로 기기를 계정에 연결 (FR-H1)
- **흐름:** `createLink` `:167` → 사람이 `/link?code=` 승인(`decideLinkCode` `:265`) → 기기가 서명으로 토큰 수령(`claimLink` `:338`)
- **상수:** `LINK_TTL_SECS` `:40`, `LINK_POLL_INTERVAL_SECS` `:42`, `MAX_WAIT_SECS` `:122`, `ROLES` `:98`, `MAX_REMOVED_LISTED` `:556`
- **주의점:** 계정당 main server 1개(`currentMainServer` `:228`, 마이그레이션 `0006_one_main_server.sql`). 기기 목록 변경 시 `bumpDevicesVersion` `:127` 호출 필수(롱폴·ETag가 이것을 봄).

### `notebooks.ts` (212줄)
- **목적:** 노트북 게시·버전·신고·관리자 내림 (FR-H17~H19)
- **한도:** `LIMITS` `:9` — `sourceBytes` 1 MiB, `runBytes` 16 MiB, `notebooksPerAccount` 200, `versionsPerNotebook` 100, `storedBytesPerAccount` 512 MiB, `reportsPerAddress` 10/시간. 분당 속도 한도는 `wrangler.toml` (주석 `:16`)
- **저장:** 메타데이터 D1, 본문 R2 키 `nb/<account>/<notebook>/<version>/<part>` (`key` `:27`)
- **주의점:** 주석에 "provisional" — 값은 여기서만 바꾼다고 명시.

### 한 줄 요약
| 파일 | 줄 | 요약 |
|---|---|---|
| `app.ts` | 202 | 라우팅, `/config`(`:55`), Flathub 검증 파일(`:75`), `/link` 랜딩(`:85`) |
| `github.ts` | 210 | OAuth + PKCE, 스코프 없음. `GITHUB_ALLOWED_IDS`로 허용 목록(`allowed` `:128`), `SESSION_TTL_SECS` `:27` |
| `directory.ts` | 87 | `PUT/GET /pkarr/{key}`, `/devices/{id}/addresses` |
| `pkarr.ts` | 242 | iroh-dns 호환 서명 패킷 검증·DNS 응답 파싱 |
| `names.ts` | 109 | `<name>.darkpyonix.dev` 예약, ACME TXT 설정 |
| `dns.ts` | 106 | `CloudflareDns`(실), `MemoryDns`(테스트) |
| `shares.ts` | 115 | 공유 게시·해석(게스트 릴레이 패스 발급, `RELAY_PASS_TTL_SECS` `:10`), `/s/{id}` ash 뷰어 |
| `relay.ts` | 68 | 릴레이 호스트 전용 `admit`/`presence`, 공유 비밀 필요, `ADMIT_CACHE_SECS` `:9` |
| `pages.ts` | 98 | 서버 렌더 HTML 2개(공유 대기 페이지, 기기 승인 페이지), 입력은 `escape` `:3` |
| `ed25519.ts` | 18 | Web Crypto 서명 검증 |
| `index.ts` | 24 | Worker 진입 |
| `vitest.config.ts` | 24 | 테스트 설정. `RELAY_SHARED_SECRET: "relay-secret"`은 테스트 전용 값 (`:16`) |

## 이 모듈을 수정할 때 주의할 점
- 엔드포인트는 `docs/api/hub.openapi.yaml`이 정본 (`app.ts` 헤더 주석).
- DB 변경은 새 마이그레이션 파일(`migrations/00NN_*.sql`)로만. 기존 파일 수정 금지 [추정: D1 마이그레이션 관례].
- 검사: `npm test`, `npm run typecheck` (`darkpyonix/hub/worker/`).
