# 📁DRF Advanced (실무 심화)

기본적인 CRUD(Create, Read, Update, Delete)를 넘어, 상용 서비스를 위한 DRF 심화 기술입니다.

---

## 1. ViewSet & Router

#### 1) APIView vs ViewSet
* **APIView**: `get`, `post` 메서드를 직접 정의. 세밀한 제어가 가능하지만 반복 코드가 많음.
* **ViewSet**: `list`, `create`, `retrieve`, `update`, `destroy` 같은 **행위(Action)** 기반으로 추상화됨. 
    * `ModelViewSet`을 상속받으면 CRUD 로직을 단 한 줄도 안 짜고 구현 가능.

#### 2) Router의 마법
* ViewSet은 URL Conf를 일일이 작성할 필요 없이, **Router**가 자동으로 URL을 매핑해준다.
    ```python
    router = DefaultRouter()
    router.register(r'users', UserViewSet) # /users/, /users/{id}/ 자동 생성
    ```

#### 3) @action 데코레이터 (Custom Endpoint)
* **문제**: ViewSet은 기본 CRUD URL만 만들어준다. "비밀번호 변경"이나 "최근 접속일 갱신" 같은 커스텀 기능은?
* **해결**: `@action` 데코레이터를 사용하면 ViewSet 내부에 함수를 만들어도 URL이 자동 생성된다.
    ```python
    class UserViewSet(ModelViewSet):
        # URL: POST /users/{id}/change_password/
        @action(detail=True, methods=['post'])
        def change_password(self, request, pk=None):
            user = self.get_object()
            # ... 비밀번호 변경 로직 ...
            return Response({'status': 'password set'})
            
        # URL: GET /users/recent_users/ (detail=False는 목록 레벨)
        @action(detail=False)
        def recent_users(self, request):
            recent_users = User.objects.filter(last_login__gte=...)
            serializer = self.get_serializer(recent_users, many=True)
            return Response(serializer.data)
    ```

---

## 2. API Versioning (버전 관리)

서비스를 운영하다 보면 API 구조를 바꿔야 할 때가 온다. 기존 앱 사용자들을 위해 **하위 호환성**을 유지하려면 버저닝이 필수다.

| 방식 | URL 예시 | 장점 | 단점 |
| :--- | :--- | :--- | :--- |
| **URL Path** | `/api/v1/users` | 가장 직관적. 브라우저에서 테스트 쉬움. | URL이 지저분해짐. 리소스를 버전별로 분리하는 느낌. |
| **Namespace** | `/api/users (namespace='v1')` | URL 깔끔함. Django 내부 라우팅으로 처리. | 설정이 약간 복잡함. |
| **Query Param** | `/api/users?version=1` | 구현이 쉬움. | REST스럽지 않다는 비판 있음 (자원과 버전은 분리해야 함). |
| **Accept Header** | `Accept: application/json; version=1.0` | **가장 RESTful한 방식**. URL 변경 없음. | 클라이언트(프론트)에서 헤더 설정을 일일이 해야 함. 테스트 불편. |

> **실무 추천**: 초기에 명확한 **URL Path (`/api/v1/...`)** 방식이 관리하기 편하고 실수할 확률이 적다.

---

## 3. Test Code (통함 테스트)

서버 개발자의 의무는 **"내 코드가 기존 기능을 망가뜨리지 않음"**을 증명하는 것이다.

#### 1) APIClient
* Django의 기본 `Client`를 확장하여 DRF 기능(JSON 처리, 인증 등)을 쉽게 쓰게 해주는 도구.

#### 2) APITestCase 작성 예시
```python
from rest_framework.test import APITestCase
from rest_framework import status
from django.urls import reverse

class PostAPITest(APITestCase):
    def setUp(self):
        # 각 테스트 실행 전 초기화 (유저 생성 등)
        self.user = User.objects.create_user(username='testuser', password='123')
        self.client.force_authenticate(user=self.user) # 강제 로그인

    def test_create_post(self):
        url = reverse('post-list') # URL Name으로 주소 찾기
        data = {'title': 'New Post', 'content': 'Hello World'}
        
        response = self.client.post(url, data, format='json')
        
        # 검증 (Assertion)
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(Post.objects.count(), 1)
        self.assertEqual(Post.objects.get().title, 'New Post')
```

#### 3) 테스트 원칙 (FIRST)
* **F**ast: 빨라야 한다.
* **I**ndependent: 각 테스트는 서로 의존하면 안 된다.
* **R**epeatable: 언제 어디서 돌려도 같은 결과여야 한다.
* **S**elf-validating: 성공/실패가 자체적으로 검증되어야 한다 (print로 확인 X).
* **T**imely: 구현하기 직전(TDD) 혹은 구현 직후에 짜야 한다.

---

## 4. API 문서화 (drf-spectacular)

FastAPI는 Swagger가 자동이지만, Django는 별도 설정이 필요하다. 과거엔 `drf-yasg`를 썼으나, 최근엔 **`drf-spectacular`**가 표준(`OpenAPI 3.0` 지원)으로 자리 잡았다.

#### 1) 설치 및 설정
```python
# settings.py
INSTALLED_APPS = [ ..., 'drf_spectacular' ]

REST_FRAMEWORK = {
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}
```

#### 2) 코드에 문서 정보 추가 (@extend_schema)
Serializer만 잘 짜도 대부분 자동화되지만, 추가 설명이 필요할 때 사용한다.

```python
from drf_spectacular.utils import extend_schema

class UserViewSet(ModelViewSet):
    @extend_schema(
        summary="회원가입",
        description="이메일과 비밀번호를 받아 회원을 생성합니다.",
        responses={201: UserSerializer}
    )
    def create(self, request, *args, **kwargs):
        return super().create(request, *args, **kwargs)
```

