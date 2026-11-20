# 사용되지 않는 파일 후보

`_raw/imports.tsv`(정적 import) 기준 팬인 0인 파일. 진입점과 문자열·빌드로 로드되는 파일을 구분했다.

| 파일 | 판정 | 근거 |
|---|---|---|
| `darkpyonix/kernel/__init__.py`, `__main__.py` | 진입점 | `_raw/entrypoints.md` |
| `darkpyonix/kernel/_client.py` | 테스트 전용 클라이언트 (프로덕션 사용처 없음) | `tests/kernel_procs.py:71`, `tests/test_fr_k9_handoff.py:24` 등에서만 import |
| `darkpyonix/kernel/_mplbackend.py` | 사용됨 (문자열 로드) | `_figures.py:17` `BACKEND_MODULE = "darkpyonix._mplbackend"` → matplotlib이 `module://`로 로드 |
| `manager/crates/*/src/main.rs`, `lib.rs`, `build.rs` | 진입점·크레이트 루트 | Cargo 규약 |
| `hub/worker/src/index.ts`, `vitest.config.ts` | 진입점·설정 | `wrangler.toml` main, vitest |
| `hub/server/src/main.rs`, `lib.rs` | 진입점·크레이트 루트 | Cargo 규약 |
