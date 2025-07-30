---
layout: post
title: "chat toner fastapi & swagger 구현"
---

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
