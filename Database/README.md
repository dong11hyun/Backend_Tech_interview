
# 목차 (Table of Contents)

1. [데이터베이스](#데이터베이스)
   - [데이터베이스를 사용하는 이유](#데이터베이스를-사용하는-이유)
   - [데이터베이스의 특징](#데이터베이스의-특징)
   - [데이터베이스의 성능 이슈 (Disk I/O)](#데이터베이스의-성능-이슈-disk-io)
2. [Index (인덱스)](#index-인덱스)
   - [인덱스란 무엇인가?](#인덱스란-무엇인가)
   - [Index 구조와 원리](#index-구조와-원리)
   - [Index 성능과 최적화 (Cardinality & Covering Index)](#index-성능과-최적화-cardinality--covering-index)
   - [Clustered vs Non-Clustered Index](#clustered-vs-non-clustered-index)
3. [정규화와 반정규화](#정규화와-반정규화)
   - [정규화 (Normalization)](#정규화-normalization)
   - [반정규화 (De-normalization)](#반정규화-de-normalization)
4. [Transaction (트랜잭션)](#transaction-트랜잭션)
   - [트랜잭션이란?](#트랜잭션이란)
   - [ACID와 상태](#acid와-상태)
   - [트랜잭션의 상태](#트랜잭션의-상태)
   - [Transaction Isolation Level (격리 수준)](#transaction-isolation-level-격리-수준)
5. [동시성 제어 (Concurrency Control)](#동시성-제어-concurrency-control)
   - [Lock의 종류 (Optimistic vs Pessimistic)](#lock의-종류-optimistic-vs-pessimistic)
   - [MVCC (Multi-Version Concurrency Control)](#mvcc-multi-version-concurrency-control)
   - [교착상태 (Deadlock)](#교착상태-deadlock)
6. [DB 아키텍처와 확장](#db-아키텍처와-확장)
   - [Replication vs Clustering](#replication-vs-clustering)
   - [Sharding (샤딩)](#sharding-샤딩)
7. [Connection Pool](#connection-pool)
8. [Statement vs PreparedStatement](#statement-vs-preparedstatement)
9. [NoSQL](#nosql)
   - [CAP 이론 (분산 시스템의 3요소)](#cap-이론-분산-시스템의-3요소)
   - [NoSQL 분류와 사용 사례](#nosql-분류와-사용-사례)

---

## 데이터베이스
#### 데이터베이스를 사용하는 이유
> 과거의 파일 시스템은 데이터 종속성, 중복성, 무결성 유지의 어려움이 있었다. 데이터베이스(DBMS)는 이러한 문제를 해결하고 데이터를 체계적으로 관리하기 위해 등장했다.

#### 데이터베이스의 특징
1. **데이터의 독립성**:
    * **물리적 독립성**: 데이터베이스의 물리적 저장 구조(디스크 크기, 파일 경로 등)가 변경되어도 응용 프로그램에는 영향을 미치지 않는다.
    * **논리적 독립성**: 데이터베이스의 논리적 구조(스키마)가 변경되어도 응용 프로그램은 변경되지 않을 수 있다(View 등을 이용).

2. **데이터의 무결성 (Integrity)**:
    * 데이터의 유효성 검사, 제약 조건(Constraints)을 통해 잘못된 데이터의 진입을 막는다.

3. **데이터의 일관성 (Consistency)**:
    * 트랜잭션을 통해 작업의 전후가 논리적으로 모순이 없음을 보장한다.

4. **데이터의 보안성**:
    * 인가된 사용자만 접근하도록 권한(Permission)을 제어한다.

5. **데이터 중복 최소화**:
    * 통합 관리를 통해 자료의 중복 문제를 해결한다.


#### 데이터베이스의 성능 이슈 (Disk I/O)
> 데이터베이스 성능 튜닝의 90% 이상은 **Disk I/O를 줄이는 것**에 초점이 맞춰져 있다. 메모리(RAM) 접근 속도는 나노초(ns) 단위인 반면, 디스크 접근은 밀리초(ms) 단위로 수십만 배 느리기 때문이다.

* **Random I/O vs Sequential I/O**:
    * 디스크 헤드가 이동하며 데이터를 읽는 Random I/O가 성능의 병목이 된다.
    * 데이터베이스 쿼리 튜닝은 불필요한 Random I/O를 줄이고, 가능한 Sequential I/O를 유도하거나 메모리(Buffer Pool)에서 데이터를 처리하도록 하는 과정이다.
    * 순차 I/O가 빠르지만, 현실의 DB 작업은 대부분 랜덤 I/O이다.
    * 쿼리 튜닝은 랜덤 I/O를 줄이고, 가능한 순차 I/O로 유도하거나 메모리(Buffer Pool)를 활용하게 만드는 과정이다.

--- 

## Index (인덱스)
#### 인덱스란 무엇인가?
> 인덱스는 **데이터의 저장(Write) 성능을 희생하고 읽기(Read) 속도를 높이는 기능**이다. 책의 '색인'과 같다. 
* 인덱스가 없으면 DB는 테이블 전체를 뒤지는 **Full Table Scan**을 수행해야 한다.
* 하지만 INSERT, UPDATE, DELETE 시 인덱스도 함께 수정해야 하므로 저장 성능은 떨어진다.
* 무분별한 인덱스 생성은 오히려 역효과를 낸다.

#### Index 구조와 원리
##### B Tree (Balanced Tree 확장형)
> 대부분의 RDBMS(MySQL InnoDB, Oracle 등)가 채택한 구조이다.

```text
       [Root Node]
      /           \
 [Branch]       [Branch]
  /    \         /    \
[Leaf]--[Leaf]--[Leaf]--[Leaf]  <-- Linked List
(Data Pointer)

```
1. 실제 데이터의 포인터(혹은 Clustered Index의 Key)는 오직 **Leaf Node**에만 존재한다.
2. Leaf Node들은 **Linked List(연결 리스트)** 형태로 연결되어 있어, **범위 검색(Range Scan)**에 매우 유리하다.
3. 모든 Leaf Node는 같은 깊이(Depth)를 가지므로 검색 시간이 균일하다(O(log N)).

##### Hash Index
* 해시 함수를 이용해 O(1)의 속도로 데이터를 찾는다.
* **단점**: **범위 검색(Range Search, 부등호 `<, >`)이 불가능**하다. 따라서 범용 데이터베이스에서는 주력 인덱스로 사용되지 않는다. (Redis 같은 Key-Value 저장소나 메모리 DB에서 주로 사용)

##### Primary vs Secondary Index
- Primary Index (Clustered Index):
    - 테이블당 1개만 존재 (주로 Primary Key).
    - 인덱스 리프 노드에 실제 데이터 페이지가 저장된다. 물리적으로 데이터가 정렬되어 있어 검색 속도가 가장 빠르다.

- Secondary Index (Non-Clustered Index):
    - 테이블당 여러 개 생성 가능.
    - 리프 노드에는 **데이터가 위치한 주소(또는 PK 값)**가 저장된다. 실제 데이터를 얻기 위해 한 번 더 조회(Lookup)가 필요할 수 있다.

#### Index 성능과 최적화 (Cardinality & Covering Index)
> 단순히 인덱스를 많이 건다고 좋은 것이 아니다. 

##### 1. 카디널리티 (Cardinality)와 선택도 (Selectivity) 
* **카디널리티가 높다** = 중복된 값이 적다 (예: 주민등록번호, ID)
* **카디널리티가 낮다** = 중복된 값이 많다 (예: 성별, 사용여부)
* **전략**: 인덱스는 **카디널리티가 높은 컬럼**에 잡아야 효율적이다.
    * *예시*: '성별' 컬럼(남/여)에 인덱스를 걸면, 전체 데이터의 50%를 읽어야 하므로 인덱스 손익분기점을 넘어 Full Scan이 더 빠를 수 있다.

##### 2. 커버링 인덱스 (Covering Index)
* 쿼리에 필요한 모든 컬럼이 인덱스에 포함되어 있어, **실제 데이터 블록(테이블)에 접근하지 않고 인덱스 스캔만으로 쿼리를 끝내는 방식**.
* Random I/O를 획기적으로 줄일 수 있는 튜닝의 핵심 기법이다.
```sql
-- (email, nickname)으로 복합 인덱스가 생성되어 있을 때
SELECT nickname FROM member WHERE email = 'test@test.com';
-- 위 쿼리는 테이블을 읽지 않고 인덱스에서 nickname을 바로 반환한다.
```
##### 3. 복합 인덱스 (Composite Index)와 순서
- 두 개 이상의 컬럼을 합쳐 인덱스를 만들 때 순서가 매우 중요하다.
- 예: (Title, Author) 순서로 인덱스를 생성하면 Title 검색 시에는 효과가 있지만, Author만으로 검색할 때는 인덱스를 타지 못한다.

#### Clustered vs Non-Clustered Index
* **Clustered Index**:
    * 물리적인 데이터 정렬 순서와 인덱스 순서가 동일하다.
    * 테이블당 **1개**만 존재 (주로 Primary Key).
    * 리프 노드에 **실제 데이터 페이지**가 저장된다. 검색 속도가 가장 빠르다.

* **Non-Clustered Index (Secondary Index)**:
    * 물리적 정렬과 무관하게 별도의 인덱스 페이지를 생성한다.
    * 리프 노드에는 **데이터가 위치한 주소(또는 PK 값)**가 저장된다.
    * 검색 후 실제 데이터를 가져오기 위해 한 번 더 조회(Lookup)가 필요할 수 있다.

---

## 정규화와 반정규화
#### 정규화 (Normalization)
> 데이터의 중복을 줄이고, 이상 현상(Anomaly)을 방지하기 위해 테이블을 분리하는 과정.

* **제 1 정규형 (1NF)**: 모든 속성은 원자값(Atomic Value)을 가져야 한다.
* **제 2 정규형 (2NF)**: 부분 함수 종속 제거 (PK의 일부에만 종속되는 컬럼 분리).
* **제 3 정규형 (3NF)**: 이행 함수 종속 제거 (A->B, B->C 일 때 A->C가 성립하는 관계 분리).
* **BCNF**: 결정자가 후보키가 아닌 함수 종속 제거.

#### 반정규화 (De-normalization)
> 정규화는 데이터 무결성을 보장하지만, 테이블이 늘어나 **JOIN 연산**이 많아져 조회 성능이 저하될 수 있다. 
실무에서는 **읽기 성능(Read Performance)**을 최적화하기 위해 의도적으로 중복을 허용하고 테이블을 합치는 반정규화를 수행한다.

* **대상**: 통계 테이블, 자주 Join되는 테이블, 대량의 Range Scan이 필요한 경우.
* **주의**: 데이터 불일치(Inconsistency)가 발생할 수 있으므로 애플리케이션 레벨에서 동기화 관리가 필수적이다.

---

## Transaction (트랜잭션)
#### 트랜잭션이란?
> 데이터베이스의 논리적 작업 단위. "전부 되거나, 아예 안 되거나(All or Nothing)"를 보장한다.

#### ACID와 상태
1. **Atomicity (원자성)**: 트랜잭션 내의 작업은 모두 성공하거나 모두 실패해야 한다.
2. **Consistency (일관성)**: 트랜잭션 수행 전후에 DB의 제약조건(무결성)이 지켜져야 한다.
3. **Isolation (격리성)**: 동시에 실행되는 트랜잭션은 서로 영향을 주지 않아야 한다. (성능과 트레이드오프 관계)
4. **Durability (지속성)**: 성공한 트랜잭션은 영구적으로 저장되어야 한다.

#### 트랜잭션의 상태
![트랜잭션 상태 다이어그램](/Database/images/transaction_states.png)
- **Active**: 트랜잭션이 실행 중인 상태.
- **Failed**: 트랜잭션 실행 중 오류가 발생하여 중단된 상태.
- **Partially Committed**: 마지막 연산까지 실행했지만, Commit 연산이 실행되기 직전 상태.
- **Committed**: 트랜잭션이 성공적으로 완료되어 Commit된 상태.
- **Aborted**: 트랜잭션이 취소되어 실행 이전으로 롤백(Rollback)된 상태.

#### Transaction Isolation Level (격리 수준)
> 동시성 처리 성능과 데이터 무결성 사이의 균형을 맞추기 위한 설정이다. 레벨이 높을수록 데이터는 정확하지만 성능(동시성)은 떨어진다.

1. **READ UNCOMMITTED (Lv.0)**
    * 다른 트랜잭션이 커밋하지 않은 데이터(Dirty Read)를 읽을 수 있다.
    * 데이터 정합성에 심각한 문제가 발생할 수 있어 거의 사용하지 않는다.

2. **READ COMMITTED (Lv.1)** - *Oracle, PostgreSQL, SQL Server 기본값*
    * 커밋된 데이터만 읽을 수 있다. (Dirty Read 방지)
    * **Non-Repeatable Read 문제**: 하나의 트랜잭션 내에서 똑같은 SELECT를 두 번 했을 때, 그 사이에 다른 트랜잭션이 값을 수정/삭제하고 커밋하면 결과가 달라질 수 있다.

3. **REPEATABLE READ (Lv.2)** - *MySQL(InnoDB) 기본값*
    * 트랜잭션이 시작된 시점의 스냅샷을 기준으로 데이터를 읽는다. 트랜잭션 내에서 항상 동일한 조회 결과를 보장한다.
    * **Phantom Read 문제**: 다른 트랜잭션이 새로운 레코드를 INSERT 하면, 보이지 않던 유령 데이터(Phantom)가 생길 수 있다. (단, MySQL InnoDB는 Next-Key Lock과 MVCC를 통해 이를 어느 정도 방지한다.)

4. **SERIALIZABLE (Lv.3)**
    * 가장 엄격한 수준. 트랜잭션을 순차적으로 처리하는 것과 같은 효과.
    * 동시 처리 성능이 급격히 떨어지므로 특수한 경우에만 사용한다.
---

## 동시성 제어 (Concurrency Control)
#### Lock의 종류 (Optimistic vs Pessimistic)
1. **비관적 락 (Pessimistic Lock)**:
    * "충돌이 발생할 것이다"라고 가정하고 데이터를 읽을 때 아예 Lock(X-Lock, S-Lock)을 걸어버린다.
    * 데이터 무결성은 확실하지만, 성능 저하와 교착상태(Deadlock) 위험이 있다.
    * 예: `SELECT ... FOR UPDATE`

2. **낙관적 락 (Optimistic Lock)**:
    * "충돌이 나지 않을 것이다"라고 가정하고 Lock 없이 작업을 수행한다.
    * 업데이트 시점에 버전(Version) 번호나 타임스탬프를 확인하여, 내가 읽은 이후에 수정된 내역이 있는지 검사한다.
    * 충돌 발생 시 재시도 로직을 애플리케이션에서 직접 구현해야 한다.


#### MVCC (Multi-Version Concurrency Control)
> 최신 DB(MySQL InnoDB, PostgreSQL 등)가 Lock을 사용하지 않고도 일관된 읽기(Consistent Read)를 제공하는 핵심 기술이다.

* **원리**: 데이터를 업데이트할 때, 기존 데이터를 덮어쓰는 것이 아니라 **Undo Log**에 이전 버전을 보관한다.
* **효과**:
    * 트랜잭션 A가 데이터를 수정 중(Lock 획득)이라도, 트랜잭션 B는 Undo Log에 있는 **이전 버전의 데이터**를 읽음으로써 대기 없이 조회 가능하다.
    * 이를 통해 **Read와 Write가 서로를 방해하지 않는다.**

#### 교착상태 (Deadlock)
> 두 개 이상의 트랜잭션이 서로가 가진 자원의 잠금을 획득하려고 무한정 대기하는 상태.

```sql
-- Transaction 1
START TRANSACTION;
INSERT INTO B VALUES(1); -- B 테이블 Lock 획득
INSERT INTO A VALUES(1); -- A 테이블 Lock 대기...

-- Transaction 2
START TRANSACTION;
INSERT INTO A VALUES(1); -- A 테이블 Lock 획득
INSERT INTO B VALUES(1); -- B 테이블 Lock 대기... (Deadlock 발생!)
```
**해결방법**:
1. **순서 통일**: 트랜잭션들이 테이블에 접근하는 순서를 동일하게 맞춘다.
2. **인덱스**: 인덱스가 없으면 잠금 범위가 테이블 전체로 확대될 수 있으므로 적절한 인덱스를 사용한다.
3. **타임아웃**: 트랜잭션 대기 시간을 설정하여 일정 시간이 지나면 롤백시킨다.

---
## DB 아키텍처와 확장
> 서비스가 커지면 한 대의 DB 서버로 감당할 수 없다. 이때 Scale-out 전략이 필요하다.

#### Replication vs Clustering
1. **Replication (복제) - Master/Slave 구조**:
    * **Master**: 쓰기(INSERT, UPDATE, DELETE) 전용. 변경사항을 Binlog 등에 기록하여 Slave로 전파.
    * **Slave**: 읽기(SELECT) 전용. Master의 로그를 받아 데이터 동기화.
    * **장점**: 읽기 트래픽 분산.
    * **단점**: Master와 Slave 간 동기화 지연(Lag)으로 인한 데이터 불일치 가능성.


2. **Clustering (클러스터링)**:
    * 여러 DB 서버가 하나의 스토리지(또는 동기화된 스토리지)를 공유하며 Active-Active 형태로 동작.
    * 한 서버가 죽어도 서비스가 중단되지 않는 고가용성(HA) 확보가 주 목적.



#### Sharding (샤딩)
> 데이터를 여러 DB 서버에 **분할 저장**하는 기술. (수평적 파티셔닝)

* **원리**: 샤딩 키(Sharding Key)를 기준으로 데이터를 나눈다. (예: user_id % 3)
* **장점**: 데이터 용량의 한계를 극복하고 쓰기 성능을 분산시킬 수 있다.
* **단점**: 관리가 매우 복잡하며, 여러 샤드에 걸친 조회(Cross-Partition Join)나 트랜잭션 처리가 어렵다.

---

## Connection Pool
> DB 연결(Connection)은 TCP/IP 핸드쉐이크 등 비용이 매우 비싼 작업이다. 요청마다 연결을 맺고 끊으면 성능이 급격히 저하된다.

* **개념**: 미리 일정 수의 Connection 객체를 만들어 **Pool**에 보관해두고, 요청이 오면 빌려주고 작업이 끝나면 반납받는 방식.
* **대표 라이브러리**: **HikariCP** (Spring Boot 2.0부터 기본).
* **설정 주의**: Pool Size가 너무 작으면 대기 시간이 길어지고, 너무 크면 메모리 낭비 및 컨텍스트 스위칭 오버헤드가 발생한다.

---

## Statement vs PreparedStatement
> 단순 속도 차이보다 **보안**과 **캐싱** 관점이 더 중요하다.

1. **Statement**:
    * 쿼리에 변수를 문자열 더하기로 직접 넣는다.
    * 매번 쿼리를 파싱하고 최적화 단계를 거친다.
    * **SQL Injection 취약점**: 사용자가 입력값에 `' OR 1=1 --` 등을 넣어 조작 가능.

2. **PreparedStatement**:
    * `?`를 사용하여 파라미터를 바인딩한다.
    * **SQL Injection 예방**: 특수문자를 자동으로 이스케이프 처리한다.
    * **Pre-compilation**: DB가 미리 쿼리 실행 계획을 캐싱(Caching)해두고 재사용하므로, 반복 실행 시 성능이 우수하다.

---

## NoSQL
> RDBMS의 경직된 스키마와 확장성(Scale-out) 한계를 극복하기 위해 등장했다.

#### CAP 이론 (분산 시스템의 3요소)
분산 시스템은 아래 3가지를 모두 만족할 수 없으며, 보통 2가지를 선택한다. (CP 또는 AP)

1. **Consistency (일관성)**: 모든 노드가 동시에 같은 데이터를 보여준다.
2. **Availability (가용성)**: 일부 노드가 실패해도 항상 응답을 받을 수 있다.
3. **Partition Tolerance (분할 허용성)**: 네트워크 단절이 발생해도 시스템이 동작한다. (분산 시스템에서는 필수 전제)

#### NoSQL 분류와 사용 사례
1. **Key-Value (Redis, Memcached)**:
    * 가장 빠르다. 단순한 Get/Set.
    * **용도**: 캐싱(Caching), 세션 관리, 리더보드, 실시간 재고 관리.

2. **Document (MongoDB)**:
    * JSON 유사 형식(BSON) 저장. 스키마가 유연하다.
    * **용도**: 로그 저장, 콘텐츠 관리 시스템(CMS), 비정형 데이터.

3. **Column Family (Cassandra, HBase)**:
    * 대량의 데이터를 쓰는데 특화. 쓰기 성능이 강력하다.
    * **용도**: 실시간 분석, 시계열 데이터, 센서 데이터.

4. **Graph (Neo4j)**:
    * 데이터 간의 '관계' 탐색에 특화.
    * **용도**: 소셜 네트워크 추천, 사기 탐지, 경로 찾기.

