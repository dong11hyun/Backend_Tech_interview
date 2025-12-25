# 목차 (Table of Contents)
1. [📁FastAPI란 무엇인가?](#fastapi란-무엇인가)
2. [📁비동기 처리 (Async & Concurrency)](#비동기-처리-async--concurrency)
3. [📁데이터 검증 (Pydantic)](#데이터-검증-pydantic)
4. [📁의존성 주입 (Dependency Injection)](#의존성-주입-dependency-injection)
5. [📁표준 문서화 (OpenAPI & Swagger)](#표준-문서화-openapi--swagger)
6. [📁실행 환경 (ASGI vs WSGI)](#실행-환경-asgi-vs-wsgi)
7. [📁Django vs FastAPI (Selection Guide)](#django-vs-fastapi-selection-guide)

---

## 📁FastAPI란 무엇인가?

> Python 3.6+의 **Type Hint**를 기반으로 한, 현대적이고 빠른 웹 API 구축 프레임워크.
> **빠른 개발 속도**와 **"Node.js/Go에 준하는 성능"** 두 마리 토끼를 잡기 위해 탄생.

**핵심 가치**
* **고성능**: 내부적으로 `Starlette`(라우팅)과 `Pydantic`(데이터)을 사용하여 현존하는 파이썬 프레임워크 중 가장 빠름.
* **직관성**: 파이썬 기본 문법(타입 힌트)만 알면 별도의 문법 학습 없이 바로 사용 가능.
* **생산성**: 자동완성 지원이 강력하여 디버깅 시간을 획기적으로 단축.

[목차🔝](#목차-table-of-contents)

---

## 📁비동기 처리 (Async & Concurrency)

> **핵심 차별점**: Django(WSGI)가 멈춰있을 때, FastAPI(ASGI)는 다른 일을 한다.

1. **Native Async Support**:
    * `async def` 구문을 통해 **비동기(Asynchronous) 코드**를 네이티브로 지원.
    * I/O Bound 작업(DB 조회, 외부 API 호출 등)에서 블로킹 없이 다른 요청을 처리.


2. **Concurrency (동시성)**:
    * Node.js와 유사한 이벤트 루프 방식을 사용하여 적은 리소스로 대량의 트래픽 처리 가능.
    * **주의점**: `async` 함수 안에서 `time.sleep()` 같은 동기 함수를 쓰면 전체 서버가 멈춤. (`await asyncio.sleep()` 사용 필수)

[목차🔝](#목차-table-of-contents)

---

## 📁데이터 검증 (Pydantic)

> 데이터 유효성 검사(Validation)와 직렬화(Serialization)를 위한 **강력한 엔진**.

**작동 원리**
* 별도의 Form 클래스나 Serializer(DRF)를 만들 필요 없이, 파이썬 클래스(`BaseModel`)로 데이터 구조 정의.
* 런타임에 타입을 강제하고, 틀린 데이터가 들어오면 명확한 에러 메시지 자동 반환.


**시니어의 관점**
* 단순한 검증 도구가 아니라, **데이터 파싱(Parsing)** 도구로 이해해야 함.
* JSON 문자열을 Python 객체로 변환하거나 그 반대의 과정을 Type-safe 하게 보장함.

[목차🔝](#목차-table-of-contents)

---

## 📁의존성 주입 (Dependency Injection)

> FastAPI 아키텍처의 **꽃(Masterpiece)**. 결합도는 낮추고 테스트 용이성은 높인다.

**Dependency Injection (DI) 시스템**
* `Depends()`를 사용하여 필요한 로직(DB 세션, 인증 유저 확인, 설정 로드 등)을 함수 파라미터로 주입받음.
* **계층 분리**: 컨트롤러(Router) 코드와 비즈니스 로직, 인프라 로직을 깔끔하게 분리할 수 있음.
* **Testability**: 테스트 시 `app.dependency_overrides`를 통해 실제 DB 대신 Mock DB로 손쉽게 교체 가능.


[목차🔝](#목차-table-of-contents)

---

## 📁표준 문서화 (OpenAPI & Swagger)

> "API 문서는 코드를 짜면 알아서 만들어지는 것."

**자동화된 문서**
* 코드를 작성하면 **OpenAPI(구 Swagger) 사양**을 따르는 문서를 자동으로 생성 (`/docs`).
* 프론트엔드 개발자와의 소통 비용이 0에 수렴.


**Client Code Generation**:
* OpenAPI 표준을 따르기 때문에, 프론트엔드(TypeScript, Swift 등)에서 API 호출 코드를 **자동 생성(Codegen)** 할 수 있음.


[목차🔝](#목차-table-of-contents)

---

## 📁실행 환경 (ASGI vs WSGI)

> FastAPI를 돌리기 위해선 **ASGI 서버**가 필요하다.

* **WSGI (Web Server Gateway Interface)**:
    * 동기 방식. Django, Flask가 사용. 요청 하나당 스레드/프로세스 점유. (예: Gunicorn)


* **ASGI (Asynchronous Server Gateway Interface)**:
    * 비동기 방식. FastAPI가 사용. 단일 스레드 내에서 비동기 이벤트 루프 처리. (예: **Uvicorn**, Hypercorn)


* 실제 운영 환경에서는 **Gunicorn**(프로세스 매니저) 위에 **Uvicorn**(워커 클래스)을 얹어서 사용하는 사례


[목차🔝](#목차-table-of-contents)

---

## 📁Django vs FastAPI (Selection Guide)

| 비교 항목 | Django (DRF) | FastAPI |
| --- | --- | --- |
| **철학** | **Batteries Included** (모든 게 다 있음) | **Micro Framework** (필요한 것만 조립) |
| **성능** | 무거움 (상대적으로 느림) | **매우 빠름** (Golang 수준에 근접) |
| **개발 속도** | 이미 만들어진 기능(Admin, Auth) 활용 시 빠름 | API 자체 구현 속도는 빠르나, 부가 기능은 직접 구현 필요 |
| **비동기 지원** | 3.1부터 지원하나 생태계가 완전치 않음 | **설계부터 비동기 중심 (Native Async)** |
| **추천 대상** | CMS, 전통적 웹사이트, 빠른 MVP (Admin 필요 시) | **MSA**, 고성능 API 서버, ML 모델 서빙, 실시간 채팅 |

> **결론**:
> 기존 레거시나 Admin 패널이 중요하다면 **Django**.
> 마이크로서비스(MSA) 구조, 높은 트래픽 처리, AI/ML 모델 서빙이 목적이라면 **FastAPI**가 압도적으로 유리합니다.

[목차🔝](#목차-table-of-contents)

---
