# 모듈: kernel-runtime-api

- 경로: `darkpyonix/kernel/__init__.py`, `_markdown.py`, `_params.py`, `_binding.py`, `_command.py`, `_misc.py`, `_display.py`, `_figures.py`, `_mplbackend.py`, `format/`
- 규모: 파일 11개 / 약 1,190줄
- 레이어: 공용 (노트북 사용자 API) + 포맷 파서
- 한 줄 책임: 노트북 파일이 `import darkpyonix`로 쓰는 API와 `.py` 노트북 포맷 파서. 커널 밖에서도 `python file.py`와 똑같이 동작해야 함 (FR-F5)
- 의존하는 모듈: [kernel-infra](kernel-infra.md) (`_host`, `_hostctx`)
- 이 모듈을 쓰는 모듈: 노트북 사용자 코드, [kernel-exec](kernel-exec.md) (`format`, `_display`, `_figures`), [kernel-collab](kernel-collab.md) (`format`)

## 공개 인터페이스 (`__init__.py:14` `_EXPORTS`, PEP 562 지연 로딩 `__getattr__` `:31`)

| 이름 | 위치 | 설명 |
|---|---|---|
| `darkpyonix.markdown(text, silent=...)` | `_markdown.py:12` | 커널에서 Markdown 렌더, 밖에서는 아무것도 안 함 |
| `darkpyonix.params` (`Params.get/all/spec`) | `_params.py:104`, `:111`, `:189`, `:193` | 값 우선순위: 실행 요청 `params` → 명령줄 `--name` → `default` |
| `@darkpyonix.binding` | `_binding.py:89` | 정의 시점 전역 스냅샷에서 def/class를 다시 평가 |
| `run_command`, `pip`, `uv` | `_command.py:70`, `:154` `_Pip`, `:186` `_Uv` | 서브프로세스 출력 줄 단위 스트리밍 |
| `display` | `_misc.py:9` → `_display.py:125` | `_repr_*_` → MIME 번들 |
| `run_parallel` | `_misc.py:28` | 코루틴 동시 실행, 인자 순서대로 결과 |
| `run_cinterop`, `run_cppinterop`, `run_rustinterop` | `_misc.py:55` `_reserved` | **예약만 됨**(구현 없음) |
| `format.parse/load/serialize` | `format/__init__.py:74`, `:100`, `:110` | 노트북 텍스트 ↔ `NotebookDocument` |

## 파일별 요약

### `format/__init__.py` (128줄) · `format/_parser.py` (119줄)
- **목적:** `# %%` 마커 기반 셀 분리. `serialize(parse(x)) == x` 바이트 동일 (`:110`)
- **주요 심볼:**
  | 이름 | 줄 | 설명 |
  |---|---|---|
  | `KNOWN_TYPES` | `__init__.py:19` | `'preamble'`, `'code'`, `'markdown'`, `'argparse'`, `'binding'`, ... |
  | `TYPE_ALIASES` | `__init__.py:24` | `{'concorrunt': 'concurrent'}` (오타 별칭, `docs/FORMAT.md:75`) |
  | `Cell`, `NotebookDocument` | `:27`, `:54` | `cells[0]`은 항상 preamble |
  | `MARKER_RE`, `METADATA_RE` | `_parser.py:25`, `:30` | `# %% 제목 [타입]`, `# @key: value` |
  | `source_sha256` | `_parser.py:60` | 셀 본문 해시 (`\r\n`→`\n`, 끝 공백줄 제거) — 실행 결과를 셀에 다시 매칭할 때 쓰임 |
- **사용처:** `_collab.py:55`, `_document.py:19`, `_executor.py:34`
- **주의점:** `source_sha256` 정규화 규칙을 바꾸면 기존 `__runs__` 기록이 셀에 매칭되지 않는다(`_document.py:42` `_map_run`).

### `_params.py` (201줄)
- **목적:** 노트북 파라미터 선언·해석. 불리언 문자열 `_TRUE`/`_FALSE`(`:15-16`)
- **주의점:** `_argv_value`(`:64`)는 argv를 소비하지 않는다(마지막 값 우선).

### `_binding.py` (134줄)
- **목적:** 데코레이터 대상의 소스를 찾아(`_source_for` `:69`) 정의 시점 전역 스냅샷에서 재컴파일
- **주의점:** 소스를 못 찾으면 [확인 필요: 실패 동작] — `binding` `:89` 본문 확인 필요.

### `_command.py` (203줄)
- **목적:** 서브프로세스 실행, 출력은 호출 스레드에서 한 줄씩 기록 (커널 셀 출력에 섞이지 않게)
- **상수:** `INTERRUPT_GRACE = 5.0` (`:19`) — 인터럽트 후 프로세스 그룹 종료까지 유예

### `_display.py` (132줄) · `_figures.py` (80줄) · `_mplbackend.py` (53줄)
- `_display.py`: `format_bundle`(`:65`)이 `_repr_html_` 등을 MIME 번들로. `TEXT_PLAIN_LIMIT = 100000` (`:14`)
- `_figures.py`: `sys.meta_path` 훅(`_PyplotHook` `:34`)으로 pyplot import 시 백엔드를 `BACKEND = 'module://darkpyonix._mplbackend'`(`:17-18`)로 지정. **커널은 matplotlib을 import하지 않는다.**
- `_mplbackend.py`: matplotlib이 문자열로 로드 (정적 import 없음 → `_raw/orphans.md`에서 사용됨으로 판정). `flush_figures` `:39`

### 한 줄 요약
| 파일 | 줄 | 요약 |
|---|---|---|
| `__init__.py` | 43 | `__version__ = "0.3.0"` (`:12`, 버전 단일 출처), 지연 export |
| `_markdown.py` | 29 | `_host.active()`가 있으면 `text/markdown` 번들 emit |
| `_misc.py` | 66 | `display`, `run_parallel`, 예약 interop |

## 이 모듈을 수정할 때 주의할 점
- `import darkpyonix`는 가벼워야 하고 커널 프로세스 모듈(`_server`, `_executor`)을 import하면 안 된다 (`__init__.py:3-5`). `tests/test_fr_f_runtime.py:62`가 모듈 목록을 검사한다.
- 새 export 추가 = `_EXPORTS` 한 줄 + 모듈 구현 + `docs/FORMAT.md` §4.
- 커널 밖 동작(plain python) 경로를 반드시 유지 (FR-F5).
