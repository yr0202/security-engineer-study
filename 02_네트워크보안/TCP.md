# TCP (Transmission Control Protocol)

## 1. TCP 기본 특징

- **연결 지향 (Connection-Oriented)**
- **신뢰성 보장** — ACK, 재전송, 순서 보장
- **흐름 제어** — 슬라이딩 윈도우
- **혼잡 제어** — Slow Start, 혼잡 회피
- **전송 단위:** 세그먼트(Segment)

---

## 2. TCP 헤더 구조 (기본 20byte)

| 필드 | 크기 | 핵심 내용 |
|------|------|-----------|
| Source / Dest Port | 각 16bit | 출발지 / 목적지 포트 |
| Sequence Number | 32bit | 송신 데이터 순서 번호 |
| Acknowledgment Number | 32bit | 다음에 받을 바이트 번호 (ACK=1일 때 유효) |
| Data Offset (HLEN) | 4bit | 헤더 길이, 32bit 워드 단위. 최솟값 5 = 20byte |
| Control Flags | 6bit | URG ACK PSH RST SYN FIN |
| Window Size | 16bit | 수신 버퍼 크기 (흐름 제어용) |
| Checksum | 16bit | 오류 검출 |
| Urgent Pointer | 16bit | URG=1일 때만 유효 |

---

## 3. TCP 제어 플래그

| 플래그 | 역할 |
|--------|------|
| SYN | 연결 수립 요청, ISN 교환 |
| ACK | 수신 확인, 2단계부터 모든 패킷에 포함 |
| FIN | 정상 연결 종료 요청 |
| RST | 강제 연결 초기화, 포트 닫힘 응답 |
| PSH | 수신 즉시 상위 계층 전달 |
| URG | 긴급 데이터 존재, Urgent Pointer와 함께 사용 |

---

## 4. 3-way Handshake (연결 수립)

```
클라이언트                          서버
    |  --- SYN (Seq=ISN_c) ------->  |  LISTEN → SYN_RECEIVED
    |  <-- SYN+ACK (Seq=ISN_s,    -- |
    |       Ack=ISN_c+1)             |
    |  --- ACK (Ack=ISN_s+1) ----->  |  → ESTABLISHED
ESTABLISHED
```

> **SYN Flood 취약점:** 3단계 ACK를 안 보내면 서버가 SYN_RECEIVED 상태로 누적 → 백로그 큐 소진
> **대응:** SYN Cookie, 방화벽 임계값 설정

---

## 5. 4-way Handshake (연결 종료)

```
클라이언트                          서버
    |  --- FIN ------------------>   |  → CLOSE_WAIT
    |  <-- ACK ------------------    |
    |  <-- FIN ------------------    |  → LAST_ACK
    |  --- ACK ------------------>   |  → CLOSED
TIME_WAIT (2MSL 대기) → CLOSED
```

> **4-way인 이유:** 서버가 ACK 후 남은 데이터 전송 완료 후 별도 FIN 전송
> **TIME_WAIT:** 마지막 ACK 유실 대비, 2MSL 동안 대기

---

## 6. TCP 상태 전이 (자주 출제)

| 상태 | 설명 |
|------|------|
| LISTEN | 서버 대기 중 |
| SYN_SENT | 클라이언트가 SYN 전송 후 대기 |
| SYN_RECEIVED | SYN 받고 SYN+ACK 전송 후 대기 → SYN Flood 시 누적 |
| ESTABLISHED | 연결 완료 |
| FIN_WAIT_1 | FIN 전송 후 대기 |
| FIN_WAIT_2 | ACK 받고 FIN 대기 |
| CLOSE_WAIT | FIN 받고 ACK 전송, 로컬 종료 대기 |
| LAST_ACK | FIN 보내고 마지막 ACK 대기 |
| TIME_WAIT | 2MSL 대기 후 CLOSED |

---

## 7. 포트 스캔 유형 (플래그 활용)

| 스캔 방식 | 전송 패킷 | 열린 포트 | 닫힌 포트 |
|-----------|-----------|-----------|-----------|
| TCP Connect | SYN | SYN+ACK (3-way 완료) | RST |
| SYN (Half-Open) | SYN | SYN+ACK → RST 전송 | RST |
| FIN | FIN | 무응답 | RST |
| NULL | 플래그 없음 | 무응답 | RST |
| XMAS | URG+PSH+FIN | 무응답 | RST |

> FIN / NULL / XMAS 스캔은 **Unix 계열에서만 유효**, Windows는 RST로 응답

---

## 8. TCP 기반 주요 공격

### SYN Flood
- 위조 IP로 SYN만 대량 전송 → SYN_RECEIVED 상태 누적 → 자원 고갈
- **대응:** SYN Cookie, 백로그 큐 확대, 방화벽

### TCP 세션 하이재킹
- Seq/Ack 번호 예측·스니핑 → 기존 세션 탈취
- **대응:** TLS 암호화, 랜덤 ISN 생성

### Land Attack
- 출발지 IP = 목적지 IP, 출발지 포트 = 목적지 포트로 전송 → 자기 자신에게 응답 반복
- **대응:** Ingress Filtering

---

## 9. 숫자 암기

| 항목 | 값 |
|------|----|
| TCP 헤더 최솟값 | 20byte |
| UDP 헤더 크기 | 8byte (고정) |
| Sequence Number | 32bit |
| Window Size | 16bit |
| TIME_WAIT 대기 | 2MSL |
| Well-known 포트 범위 | 0 ~ 1023 |

---

## 10. 주요 포트 번호

| 서비스 | 포트 |
|--------|------|
| FTP (데이터/제어) | 20 / 21 |
| SSH | 22 |
| Telnet | 23 |
| SMTP | 25 |
| DNS | 53 (UDP/TCP) |
| HTTP | 80 |
| HTTPS | 443 |
| POP3 | 110 |
| IMAP | 143 |
| RDP | 3389 |
