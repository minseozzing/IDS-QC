# 테스트 환경

## 구성도

```
[윈도우 PC]  ── curl / ping ──▶  [VM: ids-server]
192.168.56.1                      192.168.56.10
(트래픽 발생, Wireshark 캡처)      Suricata(IDS 모드) + nginx
```

외부망과 분리된 VirtualBox 호스트 전용 네트워크(실습망)에서만 테스트한다. 별도의 VM2는 만들지 않고 윈도우 호스트가 트래픽 발생 역할을 한다.

## 사양

| 항목 | 값 |
|---|---|
| 호스트 | Windows, VirtualBox 7.x |
| VM 이름 | ids-server |
| OS | Ubuntu Server 22.04.5 LTS (amd64) |
| 자원 | 2 vCPU / 4GB RAM / 25GB (LVM 미사용, `/` 약 25GB) |
| 어댑터 1 | NAT (패키지 설치용, enp0s3, 10.0.2.15) |
| 어댑터 2 | 호스트 전용 (실습망, enp0s8, 192.168.56.10/24 고정) |
| IDS | Suricata 8.0.8 (OISF 공식 PPA `ppa:oisf/suricata-stable`) |
| 웹 서버 | nginx 1.18.0 (HTTP 룰 테스트용 "보호 대상 사이트") |
| 보조 도구 | jq (eve.json 조회), Wireshark + Npcap (패킷 캡처) |
| 접속 | `ssh <user>@192.168.56.10` (자격 증명은 이 저장소에 기록하지 않음) |

## Suricata 설정 변경 사항

원본은 `suricata.yaml.bak`으로 백업한 뒤 `sed`로 5군데를 변경했다.

| 항목 | 변경 | 이유 |
|---|---|---|
| `HOME_NET` | `[192.168.56.0/24]` | 보호 대상 네트워크를 실습망으로 한정 |
| af-packet `interface` | `enp0s8` | 실습망 랜카드를 감시 |
| `checksum-validation` | `no` | VirtualBox 체크섬 오프로딩 때문에 HTTP 룰이 미탐되는 문제 예방 |
| `default-rule-path` | `/etc/suricata/rules` | 직접 만든 룰의 위치 |
| `rule-files` | `local.rules` | **공개 룰셋 대신 자체 룰만 로드** — 다른 룰이 섞이지 않게 테스트 결과를 통제 |

## 테스트 통제 원칙

- 모든 테스트는 가능하면 같은 스냅샷(`Day1-환경구축완료`)에서 시작해 재현성을 확보한다.
- 요청은 브라우저 대신 `curl.exe --path-as-is`로 보낸다. 브라우저는 주소를 임의로 정리(`/.` → `/`)하고 `favicon.ico` 요청을 몰래 추가하기 때문에 의도한 요청을 그대로 보낼 수 없다.
- 어떤 요청이 어떤 경보를 냈는지는 `eve.json`의 `http` 이벤트와 `alert` 이벤트를 **출발지 포트로 대조**해 확정한다. (`fast.log`에는 URL이 없다)
- VM은 `sudo poweroff`로 종료한다. (비정상 종료 시 eve.json 마지막 줄이 깨진 적이 있다 — [환경 이슈](04-environment-issues.md))

## 로그 조회 명령

```bash
# 모든 HTTP 요청의 출발지 포트와 URL (깨진 줄은 건너뜀)
sudo jq -rR 'fromjson? | select(.event_type=="http") | "\(.src_port) \(.http.url)"' /var/log/suricata/eve.json

# 경보가 난 요청의 출발지 포트와 룰 이름
sudo jq -rR 'fromjson? | select(.event_type=="alert") | "\(.src_port) \(.alert.signature)"' /var/log/suricata/eve.json
```
