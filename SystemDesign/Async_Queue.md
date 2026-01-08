# 📁비동기 큐 (Async Task Queue)

> "FastAPI의 `async`와 Celery의 `async`는 다르다."
> 비동기 I/O와 백그라운드 작업의 차이를 이해하는 것이 핵심이다.

---

## 1. Celery 란 무엇인가?

#### 1) 정의
Python으로 작성된 **분산 메시지 전달 기반의 비동기 작업 큐(Task Queue)**.

#### 2) 왜 필요한가? (VS FastAPI async)
* **FastAPI `async`**:
    * **I/O Bound** 작업(DB 조회, 외부 API 대기)이 끝날 때까지 기다리지 않고, 다른 가벼운 요청을 처리하는 것.
    * 하지만 CPU를 많이 쓰는 작업(이미지 변환)이나 **오래 걸리는 작업(10분)**을 `async`로 돌리면, 그동안 서버가 멈출(Blocking) 위험이 있다.
* **Celery**:
    * 웹 서버와는 **완전히 분리된 별도의 프로세스(Worker)**에서 작업을 수행한다.
    * 사용자가 "이메일 보내줘" 요청하면, 서버는 "알았어(202 Accepted)" 하고 바로 응답을 준 뒤, 실제 발송 작업은 뒤단에서 Celery가 천천히 처리한다.

---

## 2. 동작 아키텍처 (Producer - Broker - Consumer)

```text
[ Django/FastAPI (Web Server) ]  ---> 1. 메시지 발행 (Produce)
                                       |
                                       v
                                [ Message Broker ]
                                (Redis / RabbitMQ)
                                       |
                                       v
                             2. 메시지 수신 (Consume)
[ Celery Worker 1 ]   [ Celery Worker 2 ]   [ Celery Worker 3 ]
```

1. **Producer (Web Server)**: 무거운 작업을 큐(Broker)에 던지고 즉시 사용자에게 응답한다.
2. **Broker (Redis/RabbitMQ)**: 작업을 쌓아두는 대기열. 누가 작업을 가져갈 때까지 보관한다.
3. **Consumer (Worker)**: 대기열에서 작업을 하나씩 꺼내와 실제로 처리한다. 처리가 끝나면 결과 백엔드에 저장할 수도 있다.

---

## 3. RabbitMQ vs Redis (Broker 선택)

| 구분 | RabbitMQ | Redis |
| :--- | :--- | :--- |
| **전문성** | 메시지 브로커 전용 소프트웨어 | 인메모리 DB (브로커 기능도 함) |
| **신뢰성** | **높음** (메시지 영속성, 실패 처리 강력) | 메모리 기반이라 유실 위험 있음 |
| **속도** | Redis보다 약간 느림 | **매우 빠름** |
| **추천** | 데이터 유실이 치명적인 결제/주문 시스템 | 가벼운 캐시도 쓰고 큐도 쓰고 싶을 때 |

---

## 4. 실무 패턴
* **이메일/알림 발송**: 회원가입 축하 메일 (사용자를 3초간 대기시키면 안 됨).
* **리포트 생성**: "월간 통계 다운로드" 클릭 시, "준비되면 알림 드릴게요" 하고 백그라운드 생성.
* **외부 API 연동**: 타사 API가 느리거나 장애가 났을 때, 재시도(Retry) 로직을 Celery가 담당.
