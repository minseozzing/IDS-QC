# IDS-QC — 네트워크 침입 탐지 룰 품질 검증 프로젝트

> **목표:** 탐지 룰이 요구사항대로 동작하는지 검증하고, 룰이 바뀔 때마다 자동으로 재검증할 수 있는 테스트 체계를 만든다.

- 구분: 개인 프로젝트 (QA/QC 포트폴리오)
- 테스트 대상: 오픈소스 IDS **Suricata 8.0.8** 위에 직접 작성한 **탐지 룰(local.rules)**
- 시나리오(가정): 가상의 지역은행 **B은행**의 인터넷뱅킹 웹 서버·직원용 관리자 페이지 보호
  - 실제 은행 시스템이 아닌 **가상 시나리오**이며, 요구사항의 근거는 2026년 하반기 국내 금융권 실제 침해 사고 보도를 참고했습니다.
  - 모든 테스트는 외부와 분리된 본인 PC의 VirtualBox 실습망 안에서만 수행합니다.

## 무엇을 테스트하나

Suricata 엔진 자체의 버그가 아니라, 엔진 위에 올린 **룰이 의도대로 동작하는지**를 블랙박스로 검증합니다.

| 관점 | 확인하는 것 |
|---|---|
| 정탐 | 잡아야 할 요청을 잡는가 |
| 오탐 | 정상 요청을 잘못 잡지 않는가 |
| 미탐 | 대소문자·URL 인코딩 같은 우회에도 잡는가 |
| 회귀 | 룰을 고친 뒤 기존 동작이 깨지지 않는가 |

## 현재까지의 결과 (REQ-01·02, 룰 rev1)

- TC 10개 실행 → **Pass 7 / Fail 3**
- 결함 2건 발견
  - [DEF-001](docs/defects/DEF-001.md) 미탐 (Major): `/ADMIN` 대문자 우회를 놓침
  - [DEF-002](docs/defects/DEF-002.md) 오탐 (Minor, IPS 차단 모드 시 Major): `/admin2`, `/blog/admin-tips`에 경보 발생

## 문서 구성

| 문서 | 내용 |
|---|---|
| [docs/PROGRESS.md](docs/PROGRESS.md) | 진행 현황, 일정, 다음 할 일 |
| [docs/01-requirements.md](docs/01-requirements.md) | 시나리오, 요구사항(REQ-01~06), 기준값 근거 |
| [docs/02-test-environment.md](docs/02-test-environment.md) | VM·네트워크·버전 구성 |
| [docs/03-test-cases.md](docs/03-test-cases.md) | TC 설계·실행 결과·추적 매트릭스 |
| [docs/defects/](docs/defects) | 결함 리포트 |
| [docs/04-environment-issues.md](docs/04-environment-issues.md) | 환경 구축 중 겪은 이슈와 조치 |
| [docs/05-linux-commands.md](docs/05-linux-commands.md) | 사용한 리눅스 명령어와 이유 |
| [rules/local.rules](rules/local.rules) | 테스트 대상 룰 |
| [evidence/](evidence) | 증거 캡처 (eve.json, Wireshark) |
| [pcap/](pcap) | 회귀 테스트용 패킷 녹화 파일 |

## AI 활용 범위

- AI와 협업한 부분: 환경 구축 명령어, Suricata 문법 학습, 문서 서식 정리, (예정) 수집기·API 코드
- 직접 판단한 부분: 요구사항 내용, 기대 결과 설계, Pass/Fail 판정, 결함 여부와 심각도 판단, 기준값 설정

## 결과 재현 방법

[docs/03-test-cases.md](docs/03-test-cases.md)의 TC별 `curl.exe --path-as-is` 명령으로 재현합니다.
자동화 스크립트(pcap 재생 → eve.json 판정)는 `scripts/`에 추가 예정입니다.
