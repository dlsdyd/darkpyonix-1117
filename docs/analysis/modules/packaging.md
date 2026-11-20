# 모듈: packaging

- 경로: `packaging/` (`hatch_build.py`, `check_versions.py`, `smoke.py`, `darkpyonix-unsupported`)
- 규모: 파일 4개 / 약 360줄
- 레이어: 인프라 (빌드·배포)
- 한 줄 책임: PyPI wheel 빌드 훅, 버전 3곳 일치 검사, 설치된 wheel 스모크 테스트
- 의존하는 모듈: 없음 (빌드 시 Rust 바이너리를 외부에서 받음)
- 이 모듈을 쓰는 모듈: `pyproject.toml`(hatch 빌드 훅), `.github/workflows/publish-pypi.yml`

## 파일별 요약
| 파일 | 줄 | 요약 |
|---|---|---|
| `hatch_build.py` | 50 | `CustomBuildHook` `:25`. `DARKPYONIX_BINARY`(Rust 바이너리 경로)와 `DARKPYONIX_WHEEL_TAG`가 있으면 플랫폼 wheel, 없으면 `FALLBACK_SCRIPT = 'packaging/darkpyonix-unsupported'` (`:22`)를 넣은 범용 wheel |
| `check_versions.py` | 55 | 정본 `darkpyonix/kernel/__init__.py` `__version__`과 `darkpyonix/manager/Cargo.toml` (`--binary`면 바이너리 출력까지) 비교 |
| `smoke.py` | 222 | venv에 설치 후 FR-C3 수용 기준 1~5 확인. `TIMEOUT = 90` (`:22`), `main` `:119` 99줄 |
| `darkpyonix-unsupported` | 33 | 지원 안 되는 플랫폼에서 실행되는 대체 스크립트 |

## 이 모듈을 수정할 때 주의할 점
- 버전 올리기 = `__init__.py:12`, `Cargo.toml` `[workspace.package] version`, `hub/worker/package.json` `version`(현재 0.3.0). `check_versions.py`는 앞의 두 곳만 검사하고 `package.json`은 보지 않는다(파일 내 `package.json`·`hub` 언급 0건).
