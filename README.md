# dns-cost-lab

DNS 조회 비용을 0으로 가정하는 것이 얼마나 위험한지 실험으로 검증한다.

## 배경

마이크로서비스에서 서비스 간 호출 시 DNS 조회가 발생하지만, 대부분 비용을 0으로 가정한다.

- Docker 내부 DNS (127.0.0.11) 응답 시간은?
- Kubernetes CoreDNS의 응답 시간은?
- DNS 캐시 TTL이 0이면 매 요청마다 UDP 왕복
- DNS 조회 실패 시 retry/timeout이 p99 latency를 얼마나 끌어올리는가?

## 가설

| # | 가설 | 근거 |
|---|------|------|
| H1 | Docker 내부 DNS는 직접 IP 대비 0.5-2ms 추가될 것 | DNS 조회 UDP 왕복 |
| H2 | DNS TTL 0초는 TTL 30초 대비 5-10% throughput 감소 | 매 요청 DNS 조회 |
| H3 | DNS 서버 장애 시 요청 timeout이 5초+ 발생할 것 | DNS retry 기본값 |
| H4 | /etc/hosts 직접 기록이 DNS 대비 latency 0에 수렴할 것 | 커널 캐시, 네트워크 왕복 없음 |

## 실험 설계

### 시나리오
1. Docker 컨테이너에서 서비스명으로 호출 (DNS)
2. Docker 컨테이너에서 IP 직접 지정
3. DNS TTL 변경 (0 / 5 / 30 / 300초)
4. DNS 서버 다운 시뮬레이션 (iptables로 UDP 53 차단)

### 측정
- curl + `--write-out '%{time_namelookup}'`로 DNS 조회 시간 분리
- dig로 DNS 응답 시간 직접 측정
- k6로 전체 요청 latency에서 DNS 비중 산출
