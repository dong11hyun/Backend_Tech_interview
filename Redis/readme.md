# 목차 (Table of Contents)

* [📁Redis란 무엇인가?](#redis란-무엇인가)
* [📁왜 Redis는 빠른가? (Single Thread & Event Loop)](#왜-redis는-빠른가-single-thread--event-loop)
* [📁주요 자료구조 (Data Structures)](#주요-자료구조-data-structures)
* [📁Redis Pub/Sub (메시지 브로커)](#redis-pubsub-메시지-브로커)
* [📁데이터 영속성 (Persistence: RDB vs AOF)](#데이터-영속성-persistence-rdb-vs-aof)
* [📁캐싱 전략 (Caching Strategies)](#캐싱-전략-caching-strategies)
* [📁Redis 운영 시 주의사항 (O(N) 명령어)](#redis-운영-시-주의사항-on-명령어)
* [📁Redis vs Memcached](#redis-vs-memcached)
---


## 📁Redis란 무엇인가?

> **RE**mote **DI**ctionary **S**erver의 약자로, **모든 데이터를 메모리(RAM)에 상주**시켜 처리하는 고성능 키-값(Key-Value) 데이터 저장소이다.

- Redis는 단순한 캐시가 아니라, 
- **다양한 자료구조를 지원하는 인메모리 NoSQL 데이터베이스**이자, 
- **메시지 브로커**, **스트리밍 엔진**으로도 활용 가능한 미들웨어이다.

---

## 📁왜 Redis는 빠른가? (Single Thread & Event Loop)

> Redis는 초당 수만~수십만 건의 요청을 처리한다. 그 비결은 아키텍처에 있다.

1. **In-Memory Storage**:
    - 디스크 I/O 없이 메모리에서 바로 데이터를 읽기 때문에 백만 배 이상 빠르다. 
    - (`Disk 접근` : ms 단위 vs `RAM 접근`: ns 단위)

2. **Single Threaded Architecture**:
    * **멀티 스레드가 더 빠른 거 아닌가?**
    * Redis의 병목은 CPU가 아니라 **메모리와 네트워크 대역폭**이다.
    * 싱글 스레드를 사용함으로써 **Context Switching(문맥 교환) 비용**과 **Lock(동기화) 오버헤드**를 제거
    * 비동기 논블로킹 I/O (Event Loop) 모델을 사용하여 단일 스레드로도 다중 클라이언트 요청을 효율적으로 처리

---

## 📁주요 자료구조 (Data Structures)

> Redis가 강력한 이유는 단순 String뿐만 아니라 **복잡한 자료구조**를 지원하기 때문이다. 이를 통해 개발 복잡도를 낮출 수 있다.

* **String**: 기본 키-값 (최대 512MB). 캐싱, 세션 저장.
* **Hash**: 필드와 값이 있는 객체 형태. 사용자 프로필 저장.
* **List**: 양방향 연결 리스트. 메시지 큐(Queue) 구현.
* **Set**: 중복 없는 집합. 고유 방문자 수 집계.
* **Sorted Set (ZSet)**: 점수(Score)를 기반으로 정렬된 집합. **실시간 리더보드/랭킹 시스템** 구현의 핵심.

---

## 📁Redis Pub/Sub (메시지 브로커)

> Redis는 데이터 저장뿐만 아니라, **실시간 메시지 전달**을 위한 발행/구독(Publish/Subscribe) 패턴을 지원한다.

1.  **구조**:
    * **Publisher**: 특정 채널(Channel)에 메시지를 발행한다.
    * **Subscriber**: 채널을 구독하고 있다가, 메시지가 오면 즉시 받는다.
2.  **특징 (Fire-and-Forget)**:
    * 메시지를 보관하지 않는다. 구독자가 없을 때 발행된 메시지는 사라진다.
    * 실시간 채팅, 실시간 알림 등 **즉시성이 중요한 서비스**에 사용된다.
3.  **Redis Streams (v5.0+)**:
    * Pub/Sub의 단점(메시지 휘발성)을 보완하기 위해 나온 자료구조.
    * 로그(Log) 형태로 메시지를 저장하므로, 소비자가 나중에 읽을 수 있다 (Kafka와 유사).

---

## 📁데이터 영속성 (Persistence: RDB vs AOF)

> 메모리는 휘발성이므로 서버가 꺼지면 데이터가 날아간다. Redis는 이를 방지하기 위해 디스크에 저장하는 옵션을 제공.

1. **RDB (Redis Database) - 스냅샷**:
* 특정 간격마다 메모리의 **스냅샷**을 파일로 저장한다.
* **장점**: 로딩 속도가 빠르고 파일 크기가 작다. 백업에 용이하다.
* **단점**: 스냅샷 생성 이후의 데이터는 유실될 수 있다.

2. **AOF (Append Only File) - 로그 기록**:
* 모든 쓰기 명령(Command)을 로그 파일에 기록한다.
* **장점**: 데이터 유실 가능성이 매우 적다.
* **단점**: 파일 크기가 커지고, 재시작 시 로그를 다시 실행해야 하므로 로딩 속도가 느리다.

> 데이터의 중요도에 따라 두 방식을 혼용한다. 성능이 최우선이고 일부 유실이 허용되는 캐시 용도라면 둘 다 끄는 것도 전략이다.

---

## 📁캐싱 전략 (Caching Strategies)
> Redis를 캐시로 사용할 때의 아키텍처 패턴이다.

1. **Look-aside (Lazy Loading) - 가장 일반적**:
    * 앱이 데이터를 찾을 때 **Cache를 먼저 확인**한다.
    * 있으면(Cache Hit) 반환, 없으면(Cache Miss) **DB에서 조회 후 Cache에 저장**하고 반환.
    * **장점**: Redis가 죽어도 DB로 서비스가 가능하다(물론 DB 부하는 급증함).

2. **Write-through**:
    * 데이터를 쓸 때 **Cache와 DB에 동시에** 쓴다.
    * **장점**: 항상 최신 데이터를 보장(일관성).
    * **단점**: 쓰기 시간이 길어지고, 잘 안 쓰는 데이터도 캐시에 저장되는 리소스 낭비.

---

## 📁Redis 운영 시 주의사항 (O(N) 명령어)

> 싱글 스레드 아키텍처의 치명적인 약점

* **문제**: Redis는 한 번에 하나의 명령만 처리한다. 만약 처리 시간이 긴 명령어를 실행하면, **그 명령이 끝날 때까지 뒤에 있는 모든 요청이 대기(Stop-the-World)** 상태에 빠진다.
* **절대 금지 명령어 (운영 환경)**:
* `KEYS *`: 모든 키를 뒤지는 O(N) 명령어. 데이터가 많으면 수 초간 서버가 멈춘다. 
    - **`SCAN`** 명령어로 대체해야 한다.
* `FLUSHALL`, `FLUSHDB`: 모든 데이터 삭제.

---

## 📁Redis vs Memcached

| 비교 항목 | Redis | Memcached |
| --- | --- | --- |
| **자료구조** | String, Hash, List, Set, Sorted Set 등 다양함 | 오직 String (Key-Value) |
| **스레드 모델** | **Single Thread** | Multi Thread |
| **데이터 영속성** | **지원 (RDB, AOF)** | 미지원 (메모리 날아가면 끝) |
| **확장성** | Redis Cluster 지원 | 클라이언트 측 분산 처리 필요 |

**결론**: 단순 캐싱만 필요하고 멀티 스레드의 이점을 살려야 한다면 Memcached가 빠를 수 있으나, **대부분의 현대 아키텍처에서는 다양한 기능과 편의성을 갖춘 Redis를 표준으로 사용**한다.