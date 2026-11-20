# vibemap 진행 기록

대상: darkpyonix core (commit 253de56, 2026-11-17 fork 시점)
실행 위치: 저장소 루트에서 `python vibemap/scripts/<script>.py` (개인 fork 전용 폴더. upstream에는 올리지 않는다)
Windows 콘솔에서는 `PYTHONUTF8=1`을 켜고 실행한다 (cp949 출력 오류 방지).
제외: `vibemap/config.json` — `docs/설계초안`(노션 원본), `docs/analysis`(이전 분석 문서).

## 단계
- [x] 0. 추출 (파일 180개, 36,123줄, 모듈 후보 18개)
- [x] 1. 지도 (모듈 13개, 층 5개, 미배정 0)
- [ ] 2. 명함
- [ ] 3. 증상 지도
- [ ] 4. 흐름·이름
- [ ] 5. 문제점

## 명함 우선순위 (팬인 많은 곳 → 사용자 접점 순)
- [ ] kernel.base (`_protocol.py` 팬인 14: 커널 모듈 대부분이 사용)
- [ ] manager.server
- [ ] cli
- [ ] kernel.process
- [ ] kernel.exec
- [ ] kernel.collab
- [ ] manager.kernel
- [ ] hub
- [ ] kernel.runtime
- [ ] hub.db
- [ ] packaging
- [ ] tests
- [ ] docs

## 메모
- 크레이트 간 의존(Cargo path 의존)과 커널 패키지 안의 상대 import는 raw 모듈 연결선에 거의 안 잡힌다. 실제 의존은 `docs/analysis/_raw/imports.tsv`(b68d6ab 기준)와 흐름(4단계)으로 보완한다.
- 커널 폴더 하나(33파일)를 파일 목록으로 5개 모듈로 나눴다. 나눈 기준은 `docs/analysis/_raw/modules.md`와 같다.
- 같은 규약이 Python 커널과 Rust 매니저에 따로 구현되어 있다(kernel_id, 디스커버리 포트, 프레임 한도 등). 명함 cautions에 꼭 적는다.
- 2~5단계는 `docs/analysis/`(b68d6ab 기준) 내용을 재료로 쓸 수 있다. 단, 허브는 253de56에서 릴레이가 삭제되었으니(INTENT D23) 허브 위치는 반드시 새로 확인한다.
