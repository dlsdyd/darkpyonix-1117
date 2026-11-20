# vibemap 진행 기록

대상: darkpyonix core (commit 253de56, 2026-11-17 fork 시점)
실행 위치: 저장소 루트에서 `python vibemap/scripts/<script>.py` (개인 fork 전용 폴더. upstream에는 올리지 않는다)
Windows 콘솔에서는 `PYTHONUTF8=1`을 켜고 실행한다 (cp949 출력 오류 방지).
제외: `vibemap/config.json` — `docs/설계초안`(노션 원본), `docs/analysis`(이전 분석 문서).

## 단계
- [x] 0. 추출 (파일 180개, 36,123줄, 모듈 후보 18개)
- [x] 1. 지도 (모듈 13개, 층 5개, 미배정 0)
- [x] 2. 명함 (13/13. verified 3, carded 10)
- [x] 3. 증상 지도 (증상 18개, 범주 7개)
- [x] 4. 흐름·이름 (흐름 5개, 이름 26개)
- [x] 5. 문제점 (이슈 16개: high 0, mid 6, low 10)
- [x] 6. 마무리 (verify 오류 0·경고 0, 위치 75개 파일 확인, build 완료) — 2026-11-20

## 명함 우선순위 (팬인 많은 곳 → 사용자 접점 순)
- [x] kernel.base (`_protocol.py` 팬인 14: 커널 모듈 대부분이 사용) — verified
- [x] manager.server — verified
- [x] cli — verified
- [x] kernel.process
- [x] kernel.exec
- [x] kernel.collab
- [x] manager.kernel
- [x] hub
- [x] kernel.runtime
- [x] hub.db
- [x] packaging
- [x] tests
- [x] docs

carded 10개는 verify.py(이름 ±3줄)와 증상 위치 직접 확인을 통과했다. 시작점을 하나씩 열어 읽으면 verified로 올린다.

## 메모
- 크레이트 간 의존(Cargo path 의존)과 커널 패키지 안의 상대 import는 raw 모듈 연결선에 거의 안 잡힌다. 실제 의존은 `docs/analysis/_raw/imports.tsv`(b68d6ab 기준)와 흐름(4단계)으로 보완한다.
- 커널 폴더 하나(33파일)를 파일 목록으로 5개 모듈로 나눴다. 나눈 기준은 `docs/analysis/_raw/modules.md`와 같다.
- 같은 규약이 Python 커널과 Rust 매니저에 따로 구현되어 있다(kernel_id, 디스커버리 포트, 프레임 한도 등). 명함 cautions에 적었다.
- 2~5단계 재료는 `docs/analysis/`(b68d6ab 기준). 커널·매니저 코드는 253de56까지 버전 문자열 외 변경이 없어 위치가 그대로 맞았고, 허브는 253de56 코드에서 새로 확인했다.
- 이슈 I1(CLI kernel_id)은 이전 분석에서 "높음 [확인 필요]"였으나, Windows 재현 전이라 mid로 적었다.
- `docs/analysis`의 해소 이슈 2건(테스트 더미 RELAY_SHARED_SECRET, hub/server 크레이트 설명)은 릴레이 삭제(D23)로 없어져 넣지 않았다. 새 이슈 I14(허브 버전 0.3.0 잔류).

## 다음에 할 일
- 사용자 테스트 피드백이 생기면 `vibemap/data/feedback.md`에 적고 3단계를 다시 한다.
- 코드가 바뀌면 README의 "코드가 바뀐 뒤" 절차 (extract → verify → 오류 위치만 고침).
