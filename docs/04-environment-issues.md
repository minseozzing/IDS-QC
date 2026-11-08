# 환경 이슈 기록

환경 구축과 테스트 중 겪은 문제와 조치. 결과보고서 "환경 이슈"와 회고의 재료다.

| # | 현상 | 원인 | 조치 / 교훈 |
|---|---|---|---|
| 1 | VirtualBox에서 HTTP 룰 미탐 가능성 | 체크섬 오프로딩 | `checksum-validation: no` 로 비활성화 (예방 조치) |
| 2 | 공개 룰셋과 섞이면 결과 해석이 어려움 | — | `rule-files`에 자체 `local.rules`만 로드해 테스트 환경 통제 |
| 3 | `unix-manager: failed to create socket directory /var/run/suricata` 에러 | 서비스 계정(suricata)이 `/var/run/suricata` 생성 권한 없음 → 런타임 제어 소켓(suricatasc) 생성 실패. 탐지에는 영향 없음 | `/etc/tmpfiles.d/suricata.conf`에 `d /run/suricata 0755 suricata suricata -` 등록 (**미적용 — 자동화 단계에서 룰 재적재가 필요하면 적용**) |
| 4 | 여러 줄을 한 번에 붙여넣자 입력이 꼬이고 `>` 프롬프트가 계속 나옴 | 중간에 sudo 비밀번호 입력이 끼어 따옴표가 닫히지 않음 | Ctrl+C 후 한 줄씩 실행. 먼저 `sudo` 한 줄로 인증해 두면 약 15분간 재입력 불필요 |
| 5 | `sudo` 없이 `grep`/`cat` 시 Permission denied | Suricata 설정·룰 파일은 보안상 관리자만 읽을 수 있음 | 앞에 `sudo` (또는 `sudo !!`). 보안 장비 설정 파일 권한 설계의 한 예 |
| 6 | fast.log 출력이 멈춘 것처럼 보임 | 윈도우 콘솔을 클릭하면 "선택 모드"로 출력이 일시 정지됨 | Esc 또는 Ctrl+C로 해제. tail 창은 클릭하지 않기 |
| 7 | `jq`가 eve.json을 읽다 중간에 멈춤 (2495번째 줄) | VM을 정상 종료하지 않아 로그 마지막 줄이 반쯤만 기록됨 | `jq -R 'fromjson? ...'`로 깨진 줄 건너뛰기. 앞으로 `sudo poweroff` 사용. 자동화 스크립트는 깨진 줄을 건너뛰고 **건너뛴 줄 수를 기록**하도록 설계 |
| 8 | fast.log만으로는 어떤 요청이 경보를 냈는지 알 수 없음 | fast.log에 URL이 없고 출발지 포트만 있음 | eve.json의 http·alert 이벤트를 출발지 포트로 대조. **추적성(traceability)** 확보. 자동화 시 TC마다 시각·포트를 기록하도록 설계 근거로 사용 |
| 9 | 브라우저로 보낸 요청이 의도와 다름 | 브라우저가 `/.`→`/` 정리, `favicon.ico` 자동 추가 | 요청은 `curl.exe --path-as-is`로 전송 |
| 10 | PowerShell에서 `curl`이 curl이 아님 | `Invoke-WebRequest` 별칭 | `curl.exe`로 호출 |

## 테스트 중 발견한 부가 이슈 (Suricata 룰과 별개)

- nginx 404 페이지에 `nginx/1.18.0 (Ubuntu)`가 그대로 노출됨 → 서버 버전 정보 노출. 보안 점검에서 흔히 지적되는 항목이라 보고서에 한 줄 기재할 것 (`server_tokens off` 권장).
