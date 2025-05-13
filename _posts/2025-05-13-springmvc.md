---
layout: post
title: "Spring MVC 4주차 "
date: 2025-05-13
categories: [spring-study] # 또는 [research, nlp] 또는 [research, computer-vision]
use_math: true
---

### MVC 패턴 - 한계

MVC 패턴을 적용하면 컨트롤러의 역할과 뷰를 렌더링하는 역할을 명확하게 구분할 수 있다. 특히 뷰는 화면을 그리는 역할에 충실하여 코드가 깔끔하고 직관적이다. 단순히 모델에서 필요한 데이터를 꺼내서 화면을 만들면 된다.

그러나 컨트롤러에는 중복이 많고 불필요한 코드들이 많이 보인다.

MVC 컨트롤러의 단점:

- 포워드 중복
- View로 이동하는 코드가 항상 중복 호출되어야 한다. 이 부분을 메서드로 공통화할 수 있지만, 해당 메서드도 항상 직접 호출해야 한다.

```java
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response)
```

ViewPath에 중복

```
String viewPath = "/WEB-INF/views/new-form.jsp";

```

그리고 만약 jsp가 아닌 thymeleaf 같은 다른 뷰로 변경한다면 전체 코드를 다 변경해야 한다.
**사용하지 않는 code**
다음 코드를 사용할 때도 있고, 사용하지 않을 때도 있다. 특히 response는 현재 코드에서 사용되지 않는다.

또한 HttpServletRequest, HttpServletResponse를 사용하는 코드는 테스트 케이스 작성이 어렵다.

**공통 처리의 어려움**
기능이 복잡해질수록 컨트롤러에서 공통으로 처리해야 하는 부분이 증가한다. 공통 기능을 메서드로 분리할 수 있지만, 이 메서드를 항상 호출해야 하며 호출 자체가 중복이다. 실수로 호출하지 않으면 문제가 발생할 수 있다.

이러한 문제를 해결하려면 컨트롤러 호출 전에 공통 기능을 처리하는 수문장 역할이 필요하다. 프론트 컨트롤러(Front Controller) 패턴을 도입하면 이런 문제를 효과적으로 해결할 수 있다(입구를 하나로!). 스프링 MVC의 핵심도 바로 이 프론트 컨트롤러에 있다

---

### FrontController 패턴 특징

**프론트 컨트롤러 서블릿 하나로 클라이언트의 요청을 받음,프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출**

- 입구를 하나로
- 공통 처리 가능
- 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 됨

### 스프링 웹 MVC와 프론트 컨트롤러

- 스프링 웹 MVC의 핵심도 바로 FrontController
- 스프링 웹 MVC의 DispatcherServlet이 FrontController 패턴으로 구현되어 있음

－ controllerV1 : 다형성을 사용하면서 편리하게 호출

- 모든 요청을 한 서블릿이 받는 이유 -> 공통 처리 로직을 한 곳에 모으기 위해

### 1. `ControllerV1` 인터페이스

- `HttpServletRequest`, `HttpServletResponse`를 받아서 처리하는 **통일된 인터페이스**.
- 모든 컨트롤러는 이를 구현함으로써 프론트 컨트롤러에서 **일관되게 호출 가능**.

### 2. 각 컨트롤러 클래스 (`MemberFormControllerV1`, `MemberSaveControllerV1`, `MemberListControllerV1`)

- `ControllerV1`을 구현한 실제 로직 담당 클래스.
- 각각 회원 등록 폼, 회원 저장, 회원 목록 조회 기능을 수행.
- 내부 구현은 기존 서블릿 방식과 동일.

### 3. `FrontControllerServletV1`

- **URL 경로별로 컨트롤러를 맵으로 관리**하고, 요청에 맞는 컨트롤러를 찾아 실행.
- `/front-controller/v1/*` 경로에 매핑되어, 해당 경로의 모든 요청을 처리.
- **프론트 컨트롤러 패턴의 핵심 구현부**.

### 실행 흐름 요약

1. 사용자가 `/front-controller/v1/members/new-form`에 요청을 보냄
2. `FrontControllerServletV1`가 요청을 가로채고, URL에 해당하는 컨트롤러(`MemberFormControllerV1`)를 찾아 호출
3. 해당 컨트롤러는 JSP 뷰(`/WEB-INF/views/new-form.jsp`)로 forward
4. 사용자는 HTML form을 보고, 입력 후 `/save`로 요청
5. 같은 흐름으로 저장 컨트롤러가 호출되고, 데이터가 저장됨

MVC 패턴 구조를 분리해서 적용하기 위해 프론트 컨트롤러는 오직 요청 분배 역할, 컨트롤러는 비즈니스 처리 역할, JSP는 뷰 렌더링 역할을 함. 이런 식으로 역할을 나누면 구조가 명확하고, 테스트나 확장이 쉬워짐
![alt text](images/pj/z1.png)
이 부분은 nodejs 랑 똑같다

### 4. V2

ControllerV1 구조에서의 중복된 뷰 처리 코드를 제거하고, 이를 **전담하는 뷰 객체(MyView)**로 분리하여, 보다 깔끔하고 확장 가능한 **프론트 컨트롤러 구조(V2)**를 구현

| 요소            | V1 구조                                            | V2 구조                                  |
| --------------- | -------------------------------------------------- | ---------------------------------------- |
| 컨트롤러 반환값 | void (직접 forward 처리)                           | `MyView` 객체 반환                       |
| 뷰 이동 처리    | 컨트롤러 내부에서 `dispatcher.forward()` 직접 수행 | 프론트 컨트롤러가 `MyView.render()` 수행 |
| 뷰 처리 역할    | 컨트롤러가 직접 처리                               | `MyView`가 전담                          |

MyView 클래스
뷰 이름(viewPath)만 받아서 dispatcher.forward()를 실행하는 뷰 전용 객체.

각 컨트롤러는 forward 처리 대신, 단지 new MyView("/some/view.jsp")를 반환하기만 하면 됨.

ControllerV2 인터페이스
기존과 동일하게 request, response를 받지만, 반환값이 MyView.

- 프론트 컨트롤러가 뷰 렌더링을 일괄적으로 처리할 수 있게 구조화됨.

각 컨트롤러 (MemberFormControllerV2, MemberSaveControllerV2, MemberListControllerV2)

- 로직은 그대로 두고, 뷰 포워딩 코드를 제거하고 MyView만 반환.

FrontControllerServletV2

#### V2 실행 흐름

- 모든 요청을 받아 controllerMap에서 알맞은 컨트롤러를 찾아 호출.
- 컨트롤러가 반환한 MyView 객체를 받아 .render() 호출 → JSP forward 실행.
  사용자가 /front-controller/v2/members/new-form 요청

FrontControllerServletV2가 해당 URL을 controllerMap에서 찾아 MemberFormControllerV2 실행

해당 컨트롤러는 MyView("/WEB-INF/views/new-form.jsp")를 반환

프론트 컨트롤러는 view.render(request, response) 호출 → 뷰 forward 실행
| 포인트 | 설명 |
| --------------------- | --------------------------------------------------------------- |
| **중복 제거** | 모든 컨트롤러에 있던 `dispatcher.forward()` 코드 제거 |
| **역할 분리** | 컨트롤러는 로직만, 뷰 이동은 `MyView`에 위임 |
| **일관된 렌더링 처리** | `view.render()`가 프론트 컨트롤러에서 호출되므로 전처리/후처리 코드 삽입 가능 |
| **향후 리팩토링 기반 마련** | 템플릿 처리, 뷰 경로 자동화, Model 전달 등 더 발전된 프레임워크 구조로 확장 가능 |
| **Spring MVC의 철학 체험** | 실제 Spring에서도 이런 구조(Controller → ViewResolver → View)를 내부적으로 사용함 |

> "모든 컨트롤러가 똑같이 반복하는 코드 → 공통화하여 깔끔하게 분리"

### Node.js vs Spring MVC 구조 비교

| 역할 | Node.js(Express)  
| Spring MVC |
| ---------------------- | -------------------------------------- | ------------------------------------------ |
| **요청 매핑** | `router` (`app.get`, `app.post`, etc.) | `@RequestMapping`, `@GetMapping` 등 |
| **요청 처리 컨트롤러** | `controller` | `@Controller` 클래스의 메서드 |
| **비즈니스 로직** | `service` | `@Service` 클래스 |
| **데이터베이스 처리** | `repository` 또는 직접 SQL | `@Repository` 클래스 + JPA or JdbcTemplate |

### V4

| 항목            | V3                                         | V4                                                     |
| --------------- | ------------------------------------------ | ------------------------------------------------------ |
| 컨트롤러 반환   | `ModelView` (뷰 이름 + 모델을 객체로 래핑) | `String` (뷰 이름만 반환)                              |
| 모델 전달       | `ModelView.getModel().put(...)`            | `Map<String, Object> model`을 컨트롤러 파라미터로 전달 |
| 프레임워크 처리 | ModelView → viewResolver                   | **뷰 이름 추출 + 모델 전달 모두 프레임워크가 처리**    |
| 코드 간결성     | 중간 단계 필요                             | 훨씬 간결                                              |

1. 컨트롤러 조회
   요청 URL을 기반으로 controllerMap에서 해당 컨트롤러를 찾음

2. 컨트롤러 호출
   프론트 컨트롤러가 paramMap과 비어 있는 model(Map)을 만들어 컨트롤러에 넘겨줌

3. 뷰 이름 반환
   컨트롤러는 로직 처리 후, 단지 "save-result"와 같은 **논리적 뷰 이름(String)**만 반환

4. 뷰 리졸버 호출
   반환된 "save-result"를 기반으로 /WEB-INF/views/save-result.jsp와 같은 물리 경로로 변환

5. MyView 반환
   뷰 객체 생성 (new MyView("/WEB-INF/views/save-result.jsp"))

6. 뷰 렌더링
   view.render(model, request, response) 호출

model 데이터를 request.setAttribute()로 넣고 forward 실행

**V4에서는 서블릿 기술은 물론, ModelView 객체 생성도 필요 없음**

### V5

V5 구조는 다양한 컨트롤러 방식(예: `ControllerV3`, `ControllerV4`)을 **유연하게 처리**할 수 있도록 **어댑터 패턴(Adapter Pattern)**을 도입한 구조이다.

이 구조를 통해 프론트 컨트롤러는 **하나의 인터페이스에 종속되지 않고**, 다양한 형태의 컨트롤러를 처리할 수 있게 된다.

```text
[요청] → FrontControllerServletV5
↓
핸들러 조회 (Map)
↓
어댑터 목록에서 적합한 어댑터 찾기
↓
어댑터가 핸들러 실행 → ModelView 반환
↓
뷰 리졸버로 논리 → 물리 뷰 변환
↓
MyView.render(model) 호출 → JSP 렌더링
```

```java
public interface MyHandlerAdapter {
    boolean supports(Object handler);
    ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
        throws ServletException, IOException;
}
```

supports() → 이 어댑터가 해당 핸들러를 처리할 수 있는지 여부

handle() → 실제 핸들러(컨트롤러) 실행 및 ModelView 반환

## code insight

1. 다형성 기반의 유연한 구조
   Object형 handler를 어댑터가 적절히 처리 → 다양한 컨트롤러 지원 가능

2. Controller와 FrontController 완전 분리
   프론트 컨트롤러는 핸들러가 어떤 형식인지 몰라도 동작

어댑터가 책임을 떠맡음

3. 확장에 강한 구조
   새로운 컨트롤러가 등장해도, 해당 컨트롤러용 어댑터만 추가하면 됨

OCP (Open-Closed Principle) 실현

### 핵심 코드

프론트 컨트롤러: FrontControllerServletV5.java

```java
Object handler = getHandler(request);
MyHandlerAdapter adapter = getHandlerAdapter(handler);
ModelView mv = adapter.handle(request, response, handler);
MyView view = viewResolver(mv.getViewName());
view.render(mv.getModel(), request, response);

```

_어댑터 판별 로직_

```java
private MyHandlerAdapter getHandlerAdapter(Object handler) {
    for (MyHandlerAdapter adapter : handlerAdapters){
        if (adapter.supports(handler)) {
        return adapter;
    }
}
    throw new IllegalArgumentException("HandlerAdapter를 찾을 수 없습니다.");
}
```
