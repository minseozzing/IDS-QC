# 사용한 리눅스 명령어와 이유

| 명령 | 의미 | 쓴 이유 |
|---|---|---|
| `sudo` | super user do, 관리자 권한으로 실행 (본인 비밀번호 사용, 한 번 입력하면 약 15분 유지) | 시스템 폴더(/etc, /usr) 수정·설치에 필요 |
| `apt update` / `apt upgrade` / `apt install` | 패키지 목록 갱신 / 설치된 것 업데이트 / 새로 설치 | 프로그램 설치 관리 (update는 목록만 받음) |
| `&&` | 앞 명령이 성공했을 때만 뒤 명령 실행 | update 실패 시 upgrade 방지 |
| `-y` | 설치 중 질문에 자동으로 yes | 무인 진행 |
| `add-apt-repository ppa:oisf/suricata-stable` | 저장소 추가 | 기본 저장소보다 최신인 Suricata 설치 |
| `cp A A.bak` | 파일 복사 | 설정 변경 전 백업 |
| `sed -i 's/찾을것/바꿀것/'` | 파일 안 글자 치환 후 저장 | suricata.yaml 5군데를 정확히 수정 (경로에 `/`가 있으면 구분자로 `#` 사용) |
| `grep -nE "a\|b"` | 파일에서 해당 줄 검색, `-n` 줄 번호, `-E` 여러 패턴 | 변경이 반영됐는지 확인 |
| `mkdir -p` | 폴더 생성, 이미 있으면 넘어감 | 룰 폴더 생성 |
| `echo '...' \| sudo tee [-a] 파일` | 내용을 관리자 권한으로 파일에 저장, `-a`는 이어쓰기 | 룰 추가 (`-a` 없으면 기존 룰이 덮어써짐) |
| `cat 파일` | 파일 내용 출력 | 룰 파일 확인 |
| `suricata -T -c 설정파일 -v` | 설정·룰 문법 검사만 수행 | 적용 전 오류 확인 ("N rules successfully loaded") |
| `systemctl restart/status/enable suricata` | 서비스 재시작 / 상태 / 부팅 시 자동 실행 (ctl = control) | 룰 변경 적용 |
| `tail -f 파일` | 파일 끝을 실시간으로 보여줌 (Ctrl+C로 종료) | fast.log 실시간 확인 |
| `jq` | JSON 필터·출력 도구 | eve.json에서 http·alert 이벤트 추출 |
| `jq -rR 'fromjson? \| select(...) \| "\(.a) \(.b)"'` | `-r` 따옴표 없이, `-R` 줄 단위로 읽기, `fromjson?` 깨진 줄 건너뛰기, `select` 조건 필터, `\(...)` 값 끼워넣기 | 깨진 로그 줄 무시하며 포트·URL·룰 이름 대조 |
| `curl.exe --path-as-is URL` | 주소를 고치지 않고 그대로 HTTP 요청 | 의도한 요청 그대로 전송 |
| `ssh 사용자@IP` | 원격 접속 | 윈도우에서 복사·붙여넣기 편하게 작업 |
| `sudo poweroff` | 프로그램을 정상 종료하고 전원 끄기 | 로그 손상 방지 |
| `ip a` | 네트워크 인터페이스 확인 | IP 고정 확인 |
| `ipconfig` (윈도우) | 어댑터·IP 확인 | 실습망 어댑터(192.168.56.1) 식별 |

터미널 단축키: `Tab` 자동완성, `↑/↓` 이전 명령, `Ctrl+R` 명령 검색, `Ctrl+C` 중단, `sudo !!` 직전 명령을 sudo로 재실행.
