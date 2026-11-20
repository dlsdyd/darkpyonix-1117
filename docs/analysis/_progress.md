# 분석 진행 기록
- 기준 커밋: b68d6ab04e12c1ebb15f7bb666ea54becfd1f3cb
- 시작일: 2026-11-17
- 현재 단계: 완료

## 단계 체크
- [x] Phase 0 준비
- [x] Phase 1 인벤토리
- [x] Phase 2 진입점
- [x] Phase 3 의존성 그래프
- [x] Phase 4 모듈 분석 (모듈별 체크는 아래)
- [x] Phase 5 기능 지도
- [x] Phase 6 이름 사전
- [x] Phase 7 흐름·데이터 모델
- [x] Phase 8 문제점
- [x] Phase 9 수정 가이드
- [x] Phase 10 검증
- [x] Phase 11 개요 문서 완성

## 모듈 분석 체크
- [x] kernel-infra
- [x] kernel-runtime-api
- [x] kernel-exec
- [x] kernel-collab
- [x] kernel-process
- [x] manager-core
- [x] manager-server
- [x] cli
- [x] hub-worker
- [x] hub-relay
- [x] packaging

## 메모 / 다음에 할 일
- 분석 범위: 프로덕션 코드 82파일(약 2만 줄) + `packaging/`. 테스트(`tests/`, `crates/*/tests`, `hub/*/test*`, 약 1.7만 줄)는 모듈별 "검증 테스트" 항목으로만 연결한다.
- 제외: `docs/설계초안/`(노션 내보내기 원본, 코드 아님), 락 파일, `target/`.
- 이 프로젝트는 화면(UI)이 없다. 기능 지도의 "화면" 자리는 CLI 명령, 매니저 HTTP API, 허브 HTTP API, 커널 와이어 연산(DKP), 노트북 런타임 API로 대체한다.
- 토큰 절약: 파일 전문 대신 docstring·시그니처 추출(`_raw/symbols.md`)과 핵심 함수 부분 읽기로 분석한다.

## Phase 10 검증 결과 (2026-11-17)
1. `경로:줄` 참조: 전체 자동 검증 (스크립트 `.scratch/analysis/verify.py`, 저장소 무시 영역).
   - 전체 경로 + 심볼 참조 127개 중 126개 확인, 일치 126 (해석 불가 1)
   - 단축형(`` `심볼` `:줄` ``) 212개 확인: 자동 일치 194, 불일치 18 → 모두 수동 대조 결과 스크립트가 같은 행의 다른 파일명을 잡은 오탐. 실제 오류 0
   - 파일 문맥 없어 자동 확인 못 한 단축형 64개
   - 합계 확인 338개, 오류율 0%. 검증 중 앞서 고친 줄 번호 4건(`api.rs:423-432` 등)은 Phase 5에서 반영
2. 이름 사전 136개 이름 중 코드에 없는 것 3개(`FINISHED_RUN_STATUSES`, `STREAM_KINDS`, `TOKEN_KINDS`) — 모두 "이름 정리 후보"의 제안 이름이라 정상
3. 문서 간 링크 깨짐 0
4. 비밀값 노출 0 (긴 문자열 탐지 결과는 경로와 커밋 해시뿐)
5. 남은 표시: `[추정]` 5개, `[확인 필요]` 9개
