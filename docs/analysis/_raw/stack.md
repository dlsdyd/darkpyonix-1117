# 기술 스택

| 영역 | 언어/런타임 | 주요 라이브러리 | 근거 |
|---|---|---|---|
| 커널 + 런타임 API (`darkpyonix/kernel/`) | Python 3.8+ | 표준 라이브러리만 (`dependencies = []`) | `pyproject.toml` |
| 매니저 + CLI (`darkpyonix/manager/`) | Rust 2021, 워크스페이스 4 크레이트 | tokio, serde, axum `[추정: route()/get() 사용]`, clap | `darkpyonix/manager/Cargo.toml`, `crates/darkpyonix/src/args.rs:5` |
| 허브 API (`darkpyonix/hub/worker/`) | TypeScript, Cloudflare Workers | wrangler, D1(SQLite), vitest | `darkpyonix/hub/worker/package.json`, `wrangler.toml` |
| 허브 릴레이 (`darkpyonix/hub/server/`) | Rust | axum, reqwest `[추정]` | `darkpyonix/hub/server/Cargo.toml` |
| 패키징 | hatchling | `packaging/hatch_build.py` | `pyproject.toml` |

## 명령

| 목적 | 명령 | 비고 |
|---|---|---|
| Python 테스트 | `pytest` (루트) | `tests/conftest.py`가 `.scratch/src/darkpyonix` 링크 생성 |
| Rust 빌드/테스트 | `cargo test` (`darkpyonix/manager/`) | AGENTS.md: 리더가 직접 빌드 금지, builder 서브에이전트 1개, `CARGO_BUILD_JOBS=2` |
| 허브 워커 | `npm test`, `npm run typecheck`, `npm run dev` (`darkpyonix/hub/worker/`) | `package.json` scripts |
| 허브 DB 마이그레이션 | `npm run migrate:local` / `migrate:remote` | `migrations/0001~0009*.sql` |
| 커널 직접 실행 | `python -m darkpyonix --file <notebook.py>` | `darkpyonix/kernel/__main__.py:252` |
| CI | `.github/workflows/test.yml`, `publish-pypi.yml`, `release-sync.yml`, `pages.yml` | |

## 규모 (프로덕션 코드)
| 영역 | 파일 | 줄 |
|---|---|---|
| Python 커널 (`darkpyonix/kernel/`) | 31 | 약 8,150 |
| Rust 매니저 (`darkpyonix/manager/crates/*/src`, build.rs) | 28 | 약 9,700 |
| 허브 워커 TS | 16 | 약 2,540 |
| 허브 릴레이 Rust | 5 | 약 520 |
| 합계 (`_raw/prod.tsv`) | 82 | 20,276 |
| 테스트 (전 언어) | 72 | 16,721 |
| 전체 추적 파일 (설계초안 제외) | 225 | 약 5만 (문서·OpenAPI 포함) |
