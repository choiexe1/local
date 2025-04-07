---
title: 
tags:
  - java
  - programming
  - servlet
  - spring
  - mvc
publish: true
date: 2024-11-20 00:01
---

## 서블릿 (Servlet)

최초의 MVC 기술부터 현재 사용하는 스프링 MVC 기술까지 어떻게, 그리고 왜 발전 해왔는지 수강생에게 알려준다. 그 시작이 서블릿이다.

### Hello 서블릿

백문이 불여일타다. 스프링 부트 환경에서 서블릿을 등록하고 사용해본다.

> [!tip] 톰캣과 서블릿
> 서블릿은 톰캣 같은 웹 어플리케이션 서버를 직접 설치하고 그 위에 서블릿 코드를 클래스 파일로 빌드해서 올린 다음, 톰캣 서버를 실행하면 된다. 하지만 이 과정은 매우 번거롭다.
>
> 스프링 부트는 톰캣 서버를 내장하고 있으므로, 톰캣 서버 설치 없이 편리하게 서블릿 코드를 실행할 수 있다.

```java
@ServletComponentScan
@SpringBootApplication
public class ServletApplication {

    public static void main(String[] args) {
       SpringApplication.run(ServletApplication.class, args);
    }

}
```

- 스프링 부트는 서블릿을 직접 등록해서 사용할 수 있도록 `@ServletComponentScan`을 지원한다.

```java
@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("HelloServlet.service");
    }
}
```

- `@WebServlet` 서블릿 어노테이션
  - name: 서블릿 이름
  - urlPatterns: URL 매핑
- 서블릿 객체로 사용하기 위해서는 `HttpServlet`를 상속 받아야한다.
- 그리고 `protected` 접근 제어자가 사용된 `service()`를 오버라이드 해야한다. 만약 `public`이 적용된 `service()`를 오버라이드하면 호출되지 않는다.

HTTP 요청을 통해 매핑된 URL이 호출되면 서블릿 컨테이너는 `service()` 메서드를 실행한다.

```java
@Override
protected void service(HttpServletRequest req, HttpServletResponse res) throws ServletException, IOException {
    System.out.println("HelloServlet.service");
    System.out.println("req = " + req);
    System.out.println("res = " + res);

    String username = req.getParameter("username");
    System.out.println("username = " + username);

    res.setContentType("text/plain");
    res.setCharacterEncoding("utf-8");
    res.getWriter().write("hello " + username);
}
```

- `HttpServletRequest`, `HttpServletResponse`는 서블릿에서 구현해놓은 HTTP 기본 스펙이다. 이 두 객체를 이용해 다양한 기능을 구현한다.
- `req.getParameter`: 쿼리 파라미터를 쉽게 가져올 수 있는 메서드
- `res.setContentType`: 응답 헤더의 컨텐츠 타입을 설정하는 메서드
- `res.setCharacterEncoding`: 응답 헤더의 인코딩 유형을 설정하는 메서드
- `res.getWriter().write()`: 응답 바디를 설정하는 메서드

## HttpServletRequest

HTTP 요청 메시지를 개발자가 직접 파싱해서 사용해도 되지만, 불편할 것이다. 서블릿은 개발자가 HTTP 요청 메시지를 편리하게 사용할 수 있도록 개발자 대신에 HTTP 요청 메시지를 파싱한다.

그리고 그 결과를 `HttpServletRequest` 객체에 담아서 제공한다.

```http title="HTTP 요청 메시지"
POST /save HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded

username=kim&age=20
```

HTTP 요청 메시지는 다음과 같은 구조로 이루어져 있다.

- HTTP 시작 줄 (start line)
  - HTTP 메서드 정보
  - URL
  - 사용된 HTTP 프로토콜 버전
- HTTP 헤더 (header)
  - 헤더 조회
- HTTP 바디 (body)
  - form 파라미터 형식 조회
  - message body 데이터 직접 조회

**임시 저장소 기능**

- 해당 HTTP 요청이 시작부터 끝날 때 까지 유지되는 임시 저장소 기능
- 저장: `req.setAttribute(name, value)`
- 조회: `req.getAttribute(name)`

**세션 관리 기능**

- `req.getSession(create: true)`

> [!note] 중요
> HttpServletRequest, HttpServletResponse를 사용할 때 가장 중요한 점은 이 객체들이 HTTP 요청 메시지, HTTP 응답 메시지를 편리하게 사용하도록 도와주는 객체라는 점이다.
>
> 따라서 이 기능에 대해서 깊이있는 이해를 하려면 **HTTP 스펙이 제공하는 요청, 응답 메시지 자체를 이해해야 한다.**

## HTTP 요청 데이터

HTTP 요청 메시지를 통해 클라이언트에서 서버로 데이터를 전달하는 방법을 알아본다.

주로 다음 3가지의 방법을 사용한다.

`GET - 쿼리 파라미터`

- /url?username=hello&age=20
- 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
- 예) 검색, 필터, 페이징등에서 많이 사용하는 방식

`POST - HTML Form`

- content-type: application/x-www-form-urlencoded
- 메시지 바디에 쿼리 파라미터 형식으로 전달, username=hello&age=20
- 예) 회원 가입, 상품 주문, HTML Form 사용

`HTTP message body`에 데이터를 직접 담아서 요청

- HTTP API에서 주로 사용, JSON, XML, TEXT
- 데이터 형식은 주로 JSON 사용

---

References: 김영한의 스프링 MVC 1편

Links to this page:
