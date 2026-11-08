# 패킷 녹화 파일 (회귀 테스트용)

| 파일 | 내용 |
|---|---|
| `tc-002_rule-rev1.pcapng` | 룰 rev1(결함 있는 상태)에서 녹화. `/admin`(정상 탐지), `/ADMIN`(DEF-001), `/admin2`, `/blog/admin-tips`(DEF-002) 요청 4개와 404 응답 4개 포함. 총 199패킷 (Windows 호스트의 mDNS·SSDP 등 배경 트래픽도 일부 섞여 있음) |

저장 형식은 Wireshark 기본값인 pcapng다. Suricata(libpcap)는 pcapng도 읽을 수 있다. 만약 `suricata -r`가 읽지 못하면 Wireshark에서 File → Save As → "Wireshark/tcpdump/... - pcap"으로 다시 저장한다.
"표시된 패킷만 내보내기"는 TCP 연결 과정이 빠져 Suricata가 HTTP를 해석하지 못하므로 사용하지 않는다. (이 파일은 전체 저장본)

rev2 회귀 테스트(2026-11-08)는 TC 10개를 실제로 다시 전송해 수행했다. 이 파일을 `suricata -r`로 재생하는 방식은 아직 시도하지 않았으며, 자동화 단계에서 적용할 예정이다.
