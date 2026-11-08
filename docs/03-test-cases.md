# 테스트케이스 설계 및 실행 결과

대상 룰: `sid:1000002` (TEST Admin Access)
- **rev 1** (아래 "실행 결과"): `content:"/admin"` — `http.uri`에 `/admin`이 어디든 포함되면 경보 (nocase 없음, 경계 검사 없음). 실행일 2026-11-04, 요청↔경보 대조는 eve.json 출발지 포트 기준
- **rev 2** (아래 "회귀 테스트"): 결함 수정 후 같은 TC 10개 재실행. 실행일 2026-11-08

## 룰 목적과 TC 설계 기준

- 룰의 목적(= 요구사항): 관리자 페이지 접근 시도를 탐지한다.
- 기대 결과는 **사용자 의도** 기준으로 정했다. 예) `/blog/admin-tips`는 관리자 접근 의도가 없으므로 경보 X, `/%61dmin`은 서버가 `/admin`으로 해석하므로 경보 O.
- 실행 중 즉석에서 추가한 `/admin2`, `/dfwd`는 탐색적 테스트로 TC에 편입했다.

## 실행 결과

| TC ID | 분류 | 입력 | REQ | 기대 | 실제 | 출발지 포트 | 결과 | 결함 |
|---|---|---|---|---|---|---|---|---|
| TC-002-01 | 정탐 | `/admin` | REQ-01 | O | O | 5019 | Pass | |
| TC-002-02 | 정탐 (하위 경로) | `/admin/ids` | REQ-01 | O | O | 5020 | Pass | |
| TC-002-03 | 우회 (대소문자) | `/ADMIN` | REQ-01 | O | **X** | 5815 | **Fail** | DEF-001 |
| TC-002-04 | 정상 | `/` | REQ-02 | X | X | 5816 | Pass | |
| TC-002-05 | 정상 | `/index.html` | REQ-02 | X | X | 5818 | Pass | |
| TC-002-06 | 정상 (표기 변형) | `/.` | REQ-02 | X | X | 5024 | Pass | |
| TC-002-07 | 우회 (URL 인코딩) | `/%61dmin` | REQ-01 | O | O | 5023 | Pass | |
| TC-002-08 | 오탐 확인 | `/blog/admin-tips` | REQ-02 | X | **O** | 5819 | **Fail** | DEF-002 |
| TC-002-09 | 오탐 확인 | `/admin2` | REQ-02 | X | **O** | 5022 | **Fail** | DEF-002 |
| TC-002-10 | 대조군 (없는 페이지) | `/dfwd` | REQ-02 | X | X | 5025 | Pass | |

**요약: 10개 중 Pass 7 / Fail 3 (미탐 1, 오탐 2) → 결함 2건** (08·09는 원인이 같아 DEF-002 하나로 묶음)

### 해석 메모

- `/admin`도 404, `/dfwd`도 404인데 앞은 경보·뒤는 무경보 → 룰이 서버 응답이 아니라 **주소 문자열**을 본다는 것이 대조군으로 증명됨.
- `/.`는 nginx가 `/`로 해석해 메인 페이지를 반환했다. `/%61dmin`도 같은 원리로 서버가 `/admin`으로 해석하며, Suricata가 서버와 같은 방식으로 정규화하는지가 TC-002-07의 확인 포인트였다. (결과: 정규화 정상 동작)
- 로그에서 5021, 5817 등은 윈도우가 포트 번호를 건너뛴 것이며 요청 자체가 없었음을 eve.json 대조로 확인했다.
- 룰 추가 이전의 브라우저 접속 기록(포트 4914 `/admin`, `/favicon.ico`)은 본 테스트와 무관해 제외했다.

## 재현 명령 (윈도우 PowerShell)

```powershell
curl.exe --path-as-is http://192.168.56.10/admin            # TC-002-01
curl.exe --path-as-is http://192.168.56.10/admin/ids        # TC-002-02
curl.exe --path-as-is http://192.168.56.10/ADMIN            # TC-002-03
curl.exe --path-as-is http://192.168.56.10/                 # TC-002-04
curl.exe --path-as-is http://192.168.56.10/index.html       # TC-002-05
curl.exe --path-as-is http://192.168.56.10/.                # TC-002-06
curl.exe --path-as-is http://192.168.56.10/%61dmin          # TC-002-07
curl.exe --path-as-is http://192.168.56.10/blog/admin-tips  # TC-002-08
curl.exe --path-as-is http://192.168.56.10/admin2           # TC-002-09
curl.exe --path-as-is http://192.168.56.10/dfwd             # TC-002-10
```

## 추적 매트릭스

| 요구사항 | TC | 결함 |
|---|---|---|
| REQ-01 | TC-002-01, 02, 03, 07 | DEF-001 (TC-002-03) |
| REQ-02 | TC-002-04, 05, 06, 08, 09, 10 | DEF-002 (TC-002-08, 09) |

## 룰 rev1 증거 패킷

결함이 있는 상태의 트래픽을 `pcap/tc-002_rule-rev1.pcapng`으로 녹화했다 (정상 1 + 결함 3: `/admin`, `/ADMIN`, `/admin2`, `/blog/admin-tips`).
연결 과정(TCP)까지 전부 있어야 HTTP를 해석하므로 "표시된 패킷만 내보내기"가 아닌 전체 저장본이다. 이 파일을 Suricata로 재생하는 검증은 아직 하지 않았다.

## 룰 수정 (rev1 → rev2)

```
rev1: content:"/admin";
rev2: content:"/admin"; nocase; startswith; pcre:"/^\/admin(?:[\/?]|$)/i";
```

| 추가한 조건 | 해결하려는 것 | 근거 |
|---|---|---|
| `nocase` | DEF-001 (`/ADMIN` 미탐) | 대소문자를 무시하고 매칭 |
| `startswith` | DEF-002 (`/blog/admin-tips` 오탐) | `/admin`이 주소의 **시작 위치**에 있을 때만 매칭 |
| `pcre:"/^\/admin(?:[\/?]|$)/i"` | DEF-002 (`/admin2` 오탐) | `/admin` 뒤에는 끝, `/`, `?`만 허용하는 **경로 경계** |

변경 파일: `rules/local.rules` (rev2). 수정 전 룰은 `rules/local.rules.rev1`에 보관했다. sid 1000001(ping) 룰은 그대로다.

## 회귀 테스트 (룰 rev2, 2026-11-08)

TC 10개를 전체 재실행했다. rev2 경보만 구분하려고 eve.json에서 `alert.rev==2`인 경보만 필터했다 (수정 전 경보와 분리).

| TC | 입력 | 기대 | rev1 | rev2 | rev2 출발지 포트 |
|---|---|---|---|---|---|
| TC-002-01 | `/admin` | O | Pass | Pass | 1819 |
| TC-002-02 | `/admin/ids` | O | Pass | Pass | 1820 |
| TC-002-03 | `/ADMIN` | O | **Fail** | Pass | 1821 |
| TC-002-04 | `/` | X | Pass | Pass | 1822 |
| TC-002-05 | `/index.html` | X | Pass | Pass | 1823 |
| TC-002-06 | `/.` | X | Pass | Pass | 1824 |
| TC-002-07 | `/%61dmin` | O | Pass | Pass | 1825 |
| TC-002-08 | `/blog/admin-tips` | X | **Fail** | Pass | 1826 |
| TC-002-09 | `/admin2` | X | **Fail** | Pass | 1827 |
| TC-002-10 | `/dfwd` | X | Pass | Pass | 1828 |

**결과: rev1 Pass 7 / Fail 3 → rev2 Pass 10 / Fail 0.** 기존에 Pass였던 7개는 모두 그대로 Pass (회귀 없음).

- rev2 경보는 1819, 1820, 1821, 1825 네 건이며 각각 TC-002-01, 02, 03, 07이다. 나머지 6건(04, 05, 06, 08, 09, 10)은 경보 없음.
- DEF-001 해결: `/ADMIN`(1821)이 탐지됨
- DEF-002 해결: `/blog/admin-tips`(1826), `/admin2`(1827)에 경보 없음
- 룰 문법 검사: `suricata -T` 결과 "2 rules successfully loaded, 0 rules failed"
- 증거: `evidence/05_rev2_rule_loaded.png`, `evidence/06_rev2_regression_10TC.png`, `evidence/07_fastlog_rev1_vs_rev2.png`

## 추가 예정 TC

- REQ-03~06은 룰 작성 후 TC를 설계한다. ([PROGRESS](PROGRESS.md) 참고)
