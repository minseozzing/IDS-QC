# 증거 캡처

| 파일 | 내용 | 관련 문서 |
|---|---|---|
| `01_smoke_test_ping.png` | 환경 구축 후 ping 4회 → 경보 4건 (스모크 테스트) | [환경](../docs/02-test-environment.md) |
| `02_env_issue_eve_parse_error.png` | VM 비정상 종료로 eve.json이 손상되어 jq 파싱 실패 | [환경 이슈 #7](../docs/04-environment-issues.md) |
| `03_rev1_request_tracking.png` | rev1 결과. 요청 주소와 출발지 포트를 대조 (DEF-001·002 근거) | [DEF-001](../docs/defects/DEF-001.md), [DEF-002](../docs/defects/DEF-002.md) |
| `04_DEF-001_wireshark_GET_ADMIN.png` | IDS가 놓친 `GET /ADMIN` 패킷이 실제로 전송된 증거 | [DEF-001](../docs/defects/DEF-001.md) |
| `05_rev2_rule_loaded.png` | rev2 룰 문법 검사 통과 (2 rules loaded, 0 failed) | [TC](../docs/03-test-cases.md) |
| `06_rev2_regression_10TC.png` | 회귀 테스트 Pass 10 / Fail 0 | [TC](../docs/03-test-cases.md) |
| `07_fastlog_rev1_vs_rev2.png` | fast.log에서 rev1 경보와 rev2 경보 구분 | [TC](../docs/03-test-cases.md) |

캡처에 비밀번호, 개인 계정 정보가 보이지 않는지 확인한 뒤 커밋한다.
