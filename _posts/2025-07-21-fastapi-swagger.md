---
layout: post
title: "chat toner fastapi & swagger 구현"
---

# 초보의 FastAPI 개발기

- 일반적으로 main.py 에는 다음이 들어간다.

1. 필수 패키지 import와 예외 처리 (optionally)

2. FastAPI 인스턴스 생성 (앱 메타데이터/문서화/옵션 등)
3. 핵심 서비스 인스턴스 초기화 (데이터베이스, 외부 서비스 등)
4. 미들웨어(CORS 등) 적용

5. 주요 라우팅(엔드포인트) 등록

6. 필요시 메인함수(main) 정의 및 실행

신경쓴 점은 반드시 라우터·비즈니스 로직·모델·서비스로 분리했다는 것이다. 엔드포인트가 많진 않지만 프롬프트랑 합쳐지다 보니까 main.py 코드가 길어지면서 재사용성이 떨어졌다. 따라서 중요 라우터들은 api 폴더에 위치해놓았다.

### main.py

또 바뀔 수도 있지만, 잘 모르는 상태에서 main.py를 구현한 상태를 보니 다음과 같은 문제가 있었다 .
현재 main.py의 문제점들:

- 단일 파일에 모든 엔드포인트가 있음 (Single Responsibility Principle 위배)
- 의존성 주입 패턴을 제대로 활용하지 못함
- 글로벌 변수를 사용한 서비스 초기화
- 라우터 분리가 불완전함
- 설정 관리가 분산되어 있음
- 예외 처리가 일관성 없음

### Swagger_config.py

```python

"""
FastAPI Swagger UI 및 OpenAPI 설정
"""

from fastapi import FastAPI
from fastapi.openapi.utils import get_openapi
from typing import Dict, Any

# fastapi 인스턴스를 인자로 받아
# swagger/OpenAPI 문서를 커스터마이즈 하는 함수
def configure_swagger(app: FastAPI) -> None:
    """Swagger 및 OpenAPI 설정"""

    def custom_openapi() -> Dict[str, Any]:
        if app.openapi_schema:
            return app.openapi_schema
            # 스키마가 있으면 바로 그 값을 반환

        # Fastapi 의 get_openapi() .함수 호출로 openapi 명세서를 만듦( 제목, 설명, 버전, 경로 정보 )
        schema = get_openapi(
            title="Chat Toner swagger API",
            version="1.0.0",
            description="Chat Toner는 AI 기반 한국어 텍스트 스타일 변환 서비스입니다.",
            routes=app.routes,
        )
        # 태그 커스터마이즈
        # 스웨거 ui 문서에서 api 그룹핑/ 설명에 쓰일 태그 정의 ( 상위 분류 및 상세 설명 제공 )
        schema["tags"] = [
            {"name": "Health Check", "description": "서버 상태 및 연결 확인"},
            {"name": "Text Conversion", "description": "AI 기반 텍스트 스타일 변환"},
            {"name": "User Profile", "description": "사용자 개인화 프로필 관리"},
            {"name": "Preferences", "description": "네거티브 프롬프트 및 스타일 선호도"},
            {"name": "Advanced AI", "description": "파인튜닝 및 고급 프롬프트 기능"},
        ]

        # 보안 스키마 준비
        # openapi 명세에 jwt 인증 방식을 등록하는 코드
        # 모든 경로의 시큐리티 항목에 bearer 을 기본 적용해서 swagger ui 에서 jwt 인증 테스트를 가능케함
        schema["components"] = schema.get("components", {})
        schema["components"]["securitySchemes"] = {
            "BearerAuth": {
                "type": "http",
                "scheme": "bearer",
                "bearerFormat": "JWT"
            }
        }

        for path_data in schema["paths"].values():
            for operation in path_data.values():
                if "security" not in operation:
                    operation["security"] = [{"BearerAuth": []}]

        app.openapi_schema = schema
        return schema

    app.openapi = custom_openapi
    # 내부적으로 사용한느 openapi 함수를 custom_openapi 로 오버라이드
    # 이제 /openapi.json 이나 swagger UI 진입시 기존 기본 스키마가 아니라 커스텀 설정이 반영됨

def get_swagger_ui_parameters() -> Dict[str, Any]:
    """Swagger UI 커스터마이징 파라미터
    - 스웨거 제어옵션
    - 이 함수의 결과값을 FastAPI 인스턴스 생성시 파라미터로 넘기는 것
    """
    return {
        "swagger_ui_parameters": {
            "deepLinking": True,
            # URL에서 특정 경로 바로 하이라이트
            "displayRequestDuration": True,
             # 요청-응답 시간 표시
            "docExpansion": "list",
            # 초기 문서 펼침 방식(list=전체는 접고 그룹만 펼침)
            "operationsSorter": "method",
             # 메서드별 정렬
            "filter": True,
             # API 필터(검색) 버튼 활성화
            "tagsSorter": "alpha",
             # 태그 이름 알파벳 순 정렬
            "tryItOutEnabled": True,
            # Try it out(실제 요청) 기본활성화
            "layout": "BaseLayout", # 기본 UI 레이아웃
            "defaultModelsExpandDepth": 2,
            "defaultModelExpandDepth": 2,
            "showExtensions": True,
            "showCommonExtensions": True,
        }
    }



```
