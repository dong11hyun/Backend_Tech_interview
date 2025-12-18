# 목차
1. [📁Django (The Framework)](#django-the-framework)
   - Django란 무엇인가?
   - MVT 패턴과 철학
   - WSGI vs ASGI (Django의 진화)
2. [📁Django Channels (The Extension)](#django-channels-the-extension)
   - Channels란 무엇인가?
   - 핵심 개념: Consumer & Scope
   - Channel Layer (Redis의 역할)
3. [📁Daphne (The Interface Server)](#daphne-the-interface-server)
   - Daphne란 무엇인가?
   - Gunicorn vs Daphne
4. [📁DRF (Django REST Framework) 란 무엇인가?](#drf-django-rest-framework-란-무엇인가)
---
## 📁Django (The Framework)

#### Django란 무엇인가?
> **"Batteries Included"** 철학을 가진 파이썬 기반의 대표적인 웹 프레임워크. 웹 개발에 필요한 인증, 관리자 패널, ORM, 보안 기능 등을 기본적으로 모두 제공하여 **빠른 생산성**을 보장

#### MVT 패턴과 철학
> MVC(Model-View-Controller)와 매핑하여 이해해야 한다.
* **Model**: 데이터베이스 구조(Schema)를 정의. ORM을 통해 DB를 객체로 조작.
* **View (Controller 역할)**: 비즈니스 로직을 처리하고 데이터를 가공하여 템플릿에 전달.
* **Template (View 역할)**: 사용자에게 보여질 UI(HTML)를 정의.

#### WSGI vs ASGI (Django의 진화)
> Django의 역사에서 가장 큰 분기점

1. **WSGI (Web Server Gateway Interface)**:
    * **동기(Synchronous)** 방식. 요청(Request) 하나당 하나의 스레드/프로세스가 점유된다.
    * **한계**: 짧은 HTTP 요청 처리에 최적화되어 있어, 연결을 오래 유지해야 하는 **WebSocket, 채팅, 스트리밍** 구현에 부적합하다.

2. **ASGI (Asynchronous Server Gateway Interface)**:
    * **비동기(Asynchronous)** 방식. `async/await` 문법을 지원한다.
    * 하나의 스레드가 대기 시간(I/O Waiting) 동안 다른 요청을 처리할 수 있어, **WebSocket과 같은 롱 폴링 연결**을 효율적으로 처리한다.

---

## 📁Django Channels (The Extension)

#### Channels란 무엇인가?

> Django를 **동기식 HTTP 프레임워크**에서 **비동기식 이벤트 기반(Event-driven) 프레임워크**로 확장시켜주는 공식 라이브러리이다.

* 기존 Django는 HTTP 요청만 처리할 수 있었지만, Channels를 통해 **WebSocket, IoT 프로토콜(MQTT)** 등 연결 지향형 프로토콜을 처리할 수 있게 되었다.
* **핵심**: 기존의 Django 뷰(View) 작성 방식과 유사하게 비동기 코드를 작성할 수 있게 해준다.

#### 핵심 개념: Consumer & Scope

**Consumer (소비자)**
* 기존 Django의 **View**에 해당한다.
* WebSocket 연결이 맺어지거나, 메시지가 도착하는 **'이벤트'**를 처리한다.
* `SyncConsumer`와 `AsyncConsumer`를 구분해서 사용할 수 있다. (DB 접근이 많으면 Sync, 외부 API 호출 등 I/O가 많으면 Async 유리)

**Scope (스코프)**
* 기존 Django의 **Request** 객체에 해당한다.
* 현재 연결된 사용자의 정보(`scope['user']`), URL 인자(`scope['url_route']`) 등 연결의 문맥(Context)을 담고 있다.

#### Channel Layer (Redis의 역할)

**문제**
- Django 프로세스는 보통 여러 개(Worker)로 실행된다. 프로세스 A에 연결된 사용자와 프로세스 B에 연결된 사용자는 서로 메모리를 공유하지 못해 대화할 수 없다.

**해결**
- **Channel Layer**라는 외부 저장소(주로 **Redis**)를 통해 프로세스 간 통신을 수행한다.
* `group_add`: 사용자를 특정 '방'에 넣는다. (Redis에 기록)
* `group_send`: 특정 '방'에 메시지를 쏜다. (Redis Pub/Sub을 통해 모든 프로세스에 전파)

---
## 📁Daphne (The Interface Server)

#### Daphne란 무엇인가?

> Django Channels를 위해 만들어진 **HTTP 및 WebSocket 프로토콜을 지원하는 ASGI 서버**이다. (Twisted 기반)

**역할**
- 웹 브라우저와 Django 코드 사이의 **통역사(Interface)** 이다.
- 클라이언트의 요청(HTTP/WebSocket)을 받아서, 이를 Django가 이해할 수 있는 Python 객체(Scope)로 변환하여 Channels로 넘겨준다.

#### Gunicorn vs Daphne

* **Gunicorn**: 
    - WSGI 서버. **동기 처리**에 강력하고 안정적이다. 일반적인 HTTP 요청 처리에 표준처럼 쓰인다.
- **Daphne** 
    - ASGI 서버. **비동기 및 WebSocket** 처리에 특화되어 있다.
* **프로덕션 권장 구성**:
    * 과거에는 Gunicorn(HTTP) + Daphne(WS)를 분리했으나,
    * 최근에는 **Gunicorn 뒤에 Uvicorn/Daphne 워커를 붙여서** 한 번에 처리하거나,
    * Daphne 단독으로 사용하여 HTTP/2와 WebSocket을 모두 처리하기도 한다. (NeighborBid 프로젝트는 이 방식)

---
## 📁DRF (Django REST Framework) 란 무엇인가?
> RESTful API를 빠르고 표준화된 방식으로 구축하기 위한 강력한 라이브러리(Toolkit)
> 장고로 백엔드 API 서버 구축할 때, 데이터의 변환(Serialization)과 요청 처리의 표준(REST)를 잡아주는 프레임워크

1. **Serialization(직렬화)의 추상화**
    - JSON/XML 데이터 간의 변환을 `Serializer` 클래스를 통해 체계적으로 처리. 
    - 이는 데이터 검증(Validation)과 포맷 변환을 일원화하여 유지보수성을 극대화.
2. API 서버에 필수적인 기능(인증, 권한, 페이지네이션, 쓰로틀링...) 들이 **모듈화**되어 있음
3. 유연한 아키텍쳐

```
[ Client (Frontend/Mobile) ]
          ^  |
          |  |  1. Request (JSON data)
          |  v
+---------|--------------------------------------------------+
| Django  |  2. URL Dispatching (urls.py)                    |
| Project |          |                                       |
|         |          v                                       |
|    +----|---------------------------------------------+    |
|    | DRF View Layer (Views/ViewSets)                  |    |
|    |  - Authentication (로그인 확인)                  |    |
|    |  - Permission (권한 확인)                        |    |
|    |  - Throttling (요청 제한)                        |    |
|    |          |                                       |    |
|    |          v                                       |    |
|    |    3. Serializer (직렬화/역직렬화)               |    |
|    |       [ Python Object <---> JSON ]               |    |
|    |          ^                                       |    |
|    +----------|---------------------------------------+    |
|               |                                            |
|          4. Django ORM                                     |
|               v                                            |
+---------------|--------------------------------------------+
| Database      |                                            |
| (MySQL, etc.) v                                            |
+------------------------------------------------------------+
```
