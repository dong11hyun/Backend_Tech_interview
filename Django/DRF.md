
<details>
<summary> 📁DRF (Django REST Framework) 란 무엇인가?</summary>

**- RESTful API를 빠르고 표준화된 방식으로 구축하기 위한 강력한 라이브러리(Toolkit)**
**- 장고로 백엔드 API 서버 구축할 때, 데이터의 변환(Serialization)과 요청 처리의 표준(REST)를 잡아주는 프레임워크**
1. Serialization(직렬화)의 추상화
    - JSON/XML 데이터 간의 변환을 `Serializer` 클래스를 통해 체계적으로 처리. 
    - 이는 데이터 검증(Validation)과 포맷 변환을 일원화하여 유지보수성을 극대화.
2. API 서버에 필수적인 기능(인증, 권한, 페이지네이션, 쓰로틀링...) 들이 모듈화되어 있음
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
</details>

-ㅇㅇㅇㅇ