---
title: 
tags:
  - java
  - programming
  - spring
  - mvc
  - logging
publish: true
date: 2024-11-22 00:02
---

## 로깅 간단히 알아보기

운영 시스템에서는 `System.out.println()`같은 시스템 콘솔을 사용해서 필요한 정보를 출력하지 않고, 별도의 로깅 라이브러리를 사용해서 로그를 출력한다.

참고로 로그 관련 라이브러리도 많고, 깊게 들어가면 끝이 없기 때문에 여기서는 최소한의 사용 방법만 알아본다.

**로깅 라이브러리**

스프링 부트 라이브러리를 사용하면 스프링 부트 로깅 라이브러리인 `spring-boot-starter-logging`이 함께 포함된다. 스프링 부트 로깅 라이브러리는 기본으로 다음 로깅 라이브러리를 사용한다.

- [SLF4J](http://www.slf4j.org)
- [Logback](http://logback.qos.ch)

로그 라이브러리는 `Logback`, `Log4J`, `Log4J2` 등등 수 많은 라이브러리가 있는데, 그것을 통합해서 인터페이스로 제공하는 것이 바로 `SLF4J` 라이브러리다.

쉽게 이야기해서 `SLF4J`는 인터페이스고, 구현체로 `Logback` 같은 로그 라이브러리를 선택하면 된다.
실무에서는 스프링 부트가 기본으로 제공하는 `Logback`을 대부분 사용한다.

```java
@RestController
public class LogTestController {
    private Logger log = LoggerFactory.getLogger(getClass());

    @GetMapping("/log-test")
	public String logTest() {
	    String name = "Spring";

	    log.trace("trace log={}", name);
	    log.debug("debug log={}", name);
	    log.info("info log={}", name);
	    log.warn("warn log={}", name);
	    log.error("error log={}", name);

	    return "ok";
	}
}
```

- `@RestController`
  - `@Controller` 는 반환 값이 `String` 이면 뷰 이름으로 인식된다. 그래서 **뷰를 찾고, 뷰가 렌더링 된다.**
  - RestController는 반환 값으로 뷰를 찾는 것이 아니라, HTTP 메시지 바디에 바로 입력한다. 따라서 실행 결과로 ok 메시지를 받을 수 있다. `@ResponseBody`와 관련이 있는데, 뒤에서 따로 설명한다.
- `private Logger log = LoggerFactory.getLogger(getClass())`
  - 로거를 선언하는 부분이다. 주의할 점은 `Logger`의 인터페이스인데, `SLF4J`가 로그 라이브러리의 통합 인터페이스이기 때문에 인터페이스를 의존해야한다.
  - 클래스 레벨에 `@Slf4j` 어노테이션을 적용하면 롬복 라이브러리를 통해 로거를 선언할 수 있다.

#### 로그 레벨 설정

로그 레벨은 `application.properties`에서 다음과 같이 설정할 수 있다.

```
# 전체 로그 레벨 설정 (기본은 info)
logging.level.root=info

# hello.springmvc 패키지와 그 하위 로그 레벨 설정
# 해당 패키지 경로에서는 아래 설정이 더 우선권을 가짐
logging.level.hello.springmvc=trace
```

**레벨의 단계**

- `TRACE` > `DEBUG` > `INFO` > `WARN` > `ERROR`

일반적으로 개발 서버는 `DEBUG` 레벨의 로그를 출력하고, 운영 서버는 `INFO` 레벨의 로그를 출력한다.

#### 올바른 로그 사용법

- `log.debug("data = " + data)`
  - 로그 출력을 `info`로 설정해도 해당 코드에 있는 `"data = " + data` 가 실제로 실행이 되어버린다. 결과적으로 문자 더하기 연산이 발생한다.
- `log.debug("data = {}", data)`
  - 로그 출력 레벨은 `info`로 설정하면 아무 일도 발생하지 않는다. 따라서 앞과 같은 의미 없는 연산이 발생하지 않는다.

핵심은 연산이 발생해서 컴퓨터 리소스를 사용하냐, 안하냐의 차이인 것이다.

#### 로그 사용시 장점

- 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있고, 출력 모양을 조정할 수 있다.
- 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력하고, 운영 서버에서는 출력하지 않는 등 로그를 상황에 맞게 조절할 수 있다.
- 시스템 아웃 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등 로그를 별도의 위치에 남길 수 있다. 특히 파일로 남길 때는 일별, 특정 용량에 따라 로그를 분할하는 것도 가능하다.
- 성능도 일반 `System.out`보다 좋다. (내부 버퍼링, 멀티 쓰레드 등등) 그래서 실무에서는 꼭 로그를 사용해야 한다.

## 요청 매핑

회원 관리를 HTTP API로 만든다 가정하고 매핑을 어떻게 하는지 알아본다.

#### 일반적인 매핑

```java
@RestController
@Slf4j
@RequestMapping("/mapping/users")
public class MappingController {
    @GetMapping("/hello-basic")
    public String helloBasic() {
        log.info("helloBasic");
        return "ok";
    }
}
```

#### 경로 변수 (Path Variable) 매핑

최근 HTTP API는 리소스 경로에 식별자를 넣는 스타일을 선호한다. 쉽게 말해 URL 경로에 변수를 사용하는 경우다. 이것을 `Path Variable`, `경로 변수`라 한다. 스프링 MVC는 이런 경로 변수도 편리하게 사용할 수 있도록 기능을 지원한다.

```java
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable("userId") String userId) {
    log.info("mappingPath userId={}", userId);

    return "ok";
}
```

- `@GetMapping("/mapping/{userId}")`
  - `/mapping/{userId}`에 GET 요청을 올 때 이 컨트롤러를 호출한다.
  - 이 때 경로 변수는 컨트롤러 내부에서 `@PathVariable("userId")`로 파라미터로 입력 받을 수 있다.
- 경로 변수의 이름과 변수 명을 동일하게 하면 `("userId")` 부분을 생략할 수 있다.

#### 파라미터로 조건부 매핑

파라미터로 조건부 매핑을 할 수 있다. 다음의 코드를 분석해본다.

```java
@GetMapping(value = "/mapping-param", params = "mode=debug")
public String mappingParam(@RequestParam("mode") String mode) {
    log.info("mappingParam");
    log.info("param = {}", mode);
    return "ok";
}
```

- `/mapping-param` 경로에 쿼리 파라미터로 `mode=debug`가 오는 경우에만 이 컨트롤러가 호출된다.
- 만약 쿼리 파라미터 `mode`의 값이 `debug`가 아니라 `info`라면 컨트롤러가 매핑되지 않는다.

다음과 같은 경우에는 어떻게 되는지 확인해본다.

```java
@GetMapping(value = "/mapping-param", params = "mode")
public String mappingParam(@RequestParam("mode") String mode) {
    log.info("mappingParam");
    log.info("param = {}", mode);
    log.info("is empty string? = {}", mode.isEmpty());
    return "ok";
}
```

- 요청 쿼리 파라미터에 `mode`가 없으면 이 컨트롤러가 호출되지 않는다.
- 단, 키와 값으로 이루어진 쿼리 파라미터가 아니어도 호출이 된다.
- 예를 들어 `/mapping-param?mode` 일 경우에 값이 빈 문자열로 값이 들어온다.

다음과 같은 다양한 방식의 조건부 매핑이 가능하다.

- `params = "mode"`: mode가 debug면 매핑
- `params = "!mode"`: mode가 없으면 매핑
- `params = "mode=debug"`: mode의 값이 debug면 매핑
- `params = "mode!=debug"`: mode의 값이 debug가 아니면 매핑
- `params = {"mode=debug", "data=good"}` : mode가 debug 또는 data가 good일 경우 매핑

#### 특정 헤더로 조건부 매핑

```java title="헤더 조건부 매핑"
@GetMapping(value = "/mapping-header", headers = "mode=debug")
public String mappingHeader() {
	log.info("mappingHeader");
	return "ok";
}
```

```java title="Content-Type 조건부 매핑"
@PostMapping(value = "/mapping-consume", consumes = "application/json")
public String mappingConsumes() {
	log.info("mappingConsumes");
	return "ok";
}
```

```java title="Accept 조건부 매칭"
@PostMapping(value = "/mapping-produce", produces = "text/html")
public String mappingProduces() {
	log.info("mappingProduces");
	return "ok";
}
```

파라미터 매핑과 비슷하지만 HTTP 헤더를 사용한다.

- **headers**: 특정 헤더로 조건부 매핑, 파라미터 매핑과 유사하다.
- **consume**: Content-Type으로 미디어 타입 조건부 매핑
  - `"application/json"`과 같은 문자열로 구분 가능하다.
  - `org.springframework.http.MediaType`으로도 구분 가능하다.
  - 예) `consume=MediaType.APPLICATION_JSON_VALUE`
- **produces**: HTTP 헤더의 Accept를 기반으로 미디어 타입으로 매핑한다.
  - 만약 맞지 않으면 HTTP 406 상태코드(Not Acceptable)을 반환한다.

## HTTP 요청 - 기본, 헤더 조회

어노테이션 기반의 스프링 컨트롤러는 다양한 파라미터를 지원한다.

```java
@RequestMapping("/headers")
public String headers(
        HttpServletRequest req,
        HttpServletResponse res,
        HttpMethod httpMethod,
        Locale locale,
        @RequestHeader MultiValueMap<String, String> headerMap,
        @RequestHeader("host") String host,
        @CookieValue(value = "myCookie", required = false) String cookie
) {

    log.info("request={}", req);
    log.info("response={}", res);
    log.info("httpMethod={}", httpMethod);
    log.info("locale={}", locale);
    log.info("headerMap={}", headerMap);
    log.info("header host={}", host);
    log.info("myCookie={}", cookie);

    return "ok";
}
```

- `HttpServletRequest`
- `HttpServletResponse`
- `HttpMethod` : HTTP 메서드를 조회한다. `org.springframework.http.HttpMethod`
- `Locale` : Locale 정보를 조회한다.
- `@RequestHeader MultiValueMap<String, String> headerMap`
  - 모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다.
- `@RequestHeader("host") String host`
  - 특정 HTTP 헤더를 조회한다.
  - 속성
    - 필수 값 여부: `required`
    - 기본 값 속성: `defaultValue`
- `@CookieValue(value = "myCookie", required = false) String cookie`
  - 특정 쿠키를 조회한다.
  - 속성
    - 필수 값 여부: `required`
    - 기본 값: `defaultValue`

**MultiValueMap**

- MAP과 유사, 하나의 키에 여러 값을 받을 수 있는 자료 구조
- HTTP Header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용
  - 예) `keyA=value1&keyA=value2`

```java
MultiValueMap<String, String> map = new LinkedMultiValueMap();
map.add("keyA", "value1");
map.add("keyA", "value2");

List<String> values = map.get("keyA");

// values = [value1, value2] 출력
```

> [!tip] 스프링이 지원하는 `@Controller`의 파라미터 목록과 응답 값 목록
>
> - [스프링 공식 문서 파라미터 목록](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/arguments.html)
> - [스프링 공식 문서 반환 타입](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/return-types.html)

## HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form

서블릿을 이용할 때는 `request.getParameter()` 메서드를 통해서 쿼리 파라미터와 HTML Form을 획득할 수 있었다. 스프링 컨트롤러를 사용하면 쿼리 파라미터 획득과 HTML Form 획득이 얼마나 편해지는지 살펴본다.

### V1

```java
@RequestMapping("/request-param-v1")
public void requestParamV1(HttpServletRequest req, HttpServletResponse res) throws IOException {
    String username = req.getParameter("username");
    int age = Integer.parseInt(req.getParameter("age"));

    log.info("username = {}, age = {}", username, age);

    res.getWriter().write("ok");
}
```

서블릿만을 이용해서 파라미터들을 획득했다. 다음은 `@RequestParam`을 이용해서 파라미터를 직접 획득해본다.

### V2

```java
@ResponseBody
@RequestMapping("/request-param-v2")
public String requestParamV2(
        @RequestParam("username") String username,
        @RequestParam("age") int age
) {
    log.info("username = {}, age = {}", username, age);

    return "ok";
}
```

`@RequestParam` 어노테이션을 사용하면 서블릿만을 이용해서 획득했던 방식과는 다르게, 깔끔하고 편하게 파라미터를 획득할 수 있다.

참고로 파라미터 이름과 변수명을 동일하게 하면 `@RequestParam`으로 문자열을 명시하지 않고 생략할 수 있다.

### V2

```java
@ResponseBody
@RequestMapping("/request-param-v2")
public String requestParamV2(
        @RequestParam String username,
        @RequestParam int age
) {
    log.info("username = {}, age = {}", username, age);
    return "ok";
}
```

코드를 줄이기 위한 개발자의 욕심은 끝이 없다. 생략 방법은 한 가지 더 있다. 다음과 같이 작성할 경우 `@RequestParam` 마저도 생략할 수 있다.

다만, 입력 받는 파라미터의 유형이 `String`, `int`, `Integer`등 단순 타입이면 생략 가능하다.

### V3

```java
@ResponseBody
@RequestMapping("/request-param-v3")
public String requestParamV3(
        String username,
        int age
) {
    log.info("username = {}, age = {}", username, age);
    return "ok";
}
```

### 파라미터의 필수 여부 설정 - required

```java
@ResponseBody
@RequestMapping("/request-param-required")
public String requestParamRequired(
        @RequestParam(required = true) String username,
        @RequestParam(required = false) Integer age
) {
    log.info("username = {}, age = {}", username, age);
    return "ok";
}
```

- `@RequestParam.required`
  - 파라미터 필수 여부를 boolean으로 설정할 수 있다.
  - 기본값이 true로 되어 있다.

위 컨트롤러의 `username`은 필수 값이다. 따라서 요청시 해당 파라미터가 존재하지 않으면 클라이언트에 400 에러가 발생한다.

#### null != ""

주의할 점으로는 `username` 파라미터만 존재하고, 값은 빈 문자열(Empty String)일 경우에는 작동한다는 점이다. `null`과 `빈 문자열("")`은 다르기 때문이다.

#### primitive 타입과 null

위 코드를 살펴보면 `age`의 유형이 `Integer`인 것을 확인할 수 있는데, 이는 기본형인 `int`의 경우에는 `null`을 입력하는 것이 불가능하기 때문이다. 따라서 `null`을 입력 받을 수 있는 `Integer` 참조형을 사용한다.

어짜피 메서드 바디에서 사용할 땐 [[Wrapper 클래스#오토 박싱 (Auto-Boxing), 오토 언박싱 (Auto-Unboxing)|오토 언박싱]]을 이용하면 딱히 불편한 점도 없다.

이에 대한 내용은 [[Wrapper 클래스]] 문서를 참고하면 된다.

### 파라미터 기본 값 설정 - defaultValue

```java
@ResponseBody
@RequestMapping("/request-param-default")
public String requestParamDefault(
        @RequestParam(required = true, defaultValue = "guest") String username,
        @RequestParam(required = false, defaultValue = "-1") int age) {
    log.info("username={}, age={}", username, age);
    return "ok";
}
```

`defaultValue`를 설정하면 값이 존재할 경우엔 해당 값을 사용하지만 값이 `빈 문자열`이거나 `null`일 경우 `deafultValue`에 명시되어 있는 값을 사용한다.

따라서 기본 값 설정을 한 경우엔 `required`가 의미가 없다.

### 파라미터 Map으로 조회 - requestParamMap

```java
@ResponseBody
@RequestMapping("/request-param-map")
public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
    log.info("username={}, age={}", paramMap.get("username"),
            paramMap.get("age"));
    return "ok";
}
```

파라미터를 `Map`, `MultiValueMap`으로 조회 할 수 있다. `MultiValueMap`은 하나의 키에 다수의 값이 들어가는 맵과 유사한 구조를 가진다.

- **RequestParam Map**
  - `Map(key=value)`
- **@RequestParam MultiValueMap**
- 예) `(key=userIds, value=[id1,id2])`

## HTTP 요청 파라미터 - @ModelAttribute

실제 개발을 하면 요청 파라미터를 받아서 필요한 객체를 만들고, 그 객체에 값을 넣어주어야 한다. 보통 다음과 같이 코드를 작성할 것이다.

```java
@RequestParam String username;
@RequestParam int age;

HelloData data = new HelloData();
data.setUsername(username);
data.setAge(age);
```

스프링은 이 과정을 완전히 자동화해주는 `@ModelAttribute` 기능을 제공한다. 먼저 요청 파라미터를 바인딩 받을 객체를 만든다.

```java
@Data
public class HelloData {
    private String username;
    private int age;
}
```

- 롬복의 `@Data`
  - `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode`, `@RequiredArgsContructor`를 자동으로 적용해준다.

```java
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData) {
    log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
    log.info("helloData = {}", helloData);
    return "ok";
}
```

마치 마법처럼 `HelloData` 객체가 생성되고, 요청 파라미터의 값도 모두 들어가 있다. 스프링 MVC는 `@ModelAttribute`가 있으면 다음을 실행한다.

- `HelloData` 객체를 생성한다.
- 요청 파라미터의 이름으로 `HelloData` 객체의 프로퍼티를 찾는다. 그리고 해당 프로퍼티의 setter를 호출해서 파라미터의 값을 입력(바인딩)한다.
  - 예) 파라미터의 이름이 `username`이면 `setUsername()` 메서드를 찾아서 호출하면서 값을 바인딩
- 만약 `@ModelAttribute`의 `value` 속성이 생략되면 해당 파라미터의 유형의 첫 글자만 소문자로 변경한 문자열을 키 값으로 모델에 추가한다.
  - 예) `HelloData 유형이면 -> model.addAttribute("helloData", helloData)`

`@ModelAttribute` 어노테이션 또한 생략 할 수 있다. 단순히 어노테이션을 생략하고 객체 유형만 다음과 같이 입력하면 된다.

```java
modelAttribute(HelloData helloData) { ... }
```

그런데 `@RequestParam`도 생략할 수 있으니 혼란이 발생할 수 있다. 스프링은 파라미터의 유형별로 다음의 규칙을 적용한다.

- `String`, `int`, `Integer` 같은 단순 타입 = `@RequestParam` 적용
- 나머지는 모두 `@ModelAttribute` 적용 (argument resolver로 지정해둔 타입 제외)

#### 바인딩 오류

`age=abc`처럼 숫자가 들어가야 할 곳에 문자를 넣으면 `BindException`이 발생한다. 이런 바인딩 오류를 처리하는 방법은 검증 부분에서 다룬다.

## HTTP 요청 메시지 - 단순 텍스트

요청 파라미터와 다르게 HTTP 메시지 바디를 통해 데이터가 직접 넘어오는 경우는 `@RequestParam` 또는 `@ModelAttribute`를 사용할 수 없다. (HTML Form 형식으로 전달되는 경우는 요청 파라미터로 인정)

- 먼저 가장 단순한 텍스트 메시지를 HTTP 메시지 바디에 담아서 전송하고, 읽어본다.
- HTTP 요청 메시지 바디의 데이터를 `InputStream`을 사용해서 직접 읽을 수 있다.

### V1 - Servlet

```java
@Controller
@Slf4j
public class RequestBodyStringController {
    @PostMapping("/request-body-string-v1")
    public void requestBodyString(HttpServletRequest req, HttpServletResponse res) throws IOException {
        ServletInputStream inputStream = req.getInputStream();
        String message = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("message = {}", message);

        res.getWriter().write("ok");
    }
}
```

- 서블릿을 이용한 방법이다.
- `InputStream`을 통해 메시지 바디를 읽는다.
- `StreamUtils.copyToString(InputStream, Charset)` 메서드를 이용해 스트림을 문자열로 변환한다.

### V2 - InputStream, Writer

```java
@PostMapping("/request-body-string-v2")
public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {
    String message = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
    log.info("message = {}", message);

    responseWriter.write("ok");
}
```

스프링 MVC는 다음 파라미터를 지원한다.

- **InputStream(Reader)**: HTTP 요청 메세지 바디의 내용을 직접 조회
- **OutputStream(Writer)**: HTTP 응답 메세지의 바디에 직접 결과 출력

### V3 - HttpEntity

```java
@PostMapping("/request-body-string-v3")
public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
    String message = httpEntity.getBody();

    log.info("message = {}", message);

    return new HttpEntity<>("ok");
}
```

스프링 MVC는 다음 파라미터를 지원한다.

- **HttpEntity**: HTTP Header, Body 정보를 편리하게 조회
  - 메시지 바디 정보를 직접 조회
  - 요청 파라미터를 조회하는 기능과 관계 없음 - `@RequestParam` X, `@ModelAttribute` X
- **HttpEntity**는 응답에도 사용 가능
  - 메시지 바디 정보 직접 반환
  - 헤더 정보 포함 가능
  - view 조회 X

`HttpEntity`를 상속받은 다음 객체들은 같은 기능에 더 해, 추가 기능을 제공한다.

- **RequestEntity**
  - HttpMethod, url 정보, 요청에서 사용
- **ResponseEntity**
  - Http 상태 코드 설정 가능, 응답에서 사용
  - `return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED)`

### V4 - @RequestBody, @ResponseBody

어노테이션 기반으로 동작하여 편리하게 HTTP 요청 메세지 바디를 조회할 수 있다. 참고로 헤더 정보가 필요하면 `HttpEntity`를 사용하거나 `@RequestHeader` 어노테이션을 사용하면 된다.

- 요청: `@RequestBody`
- 응답: `@ResponseBody`

```java
@ResponseBody
@PostMapping("/request-body-string-v4")
public String requestBodyStringV4(@RequestBody String message) {
    log.info("message = {}", message);

    return "ok";
}
```

앞서 살펴보았던 방법들과 비교하면 굉장히 편리하게 사용할 수 있다. 따라서 실무에서 가장 많이 사용하는 방법이다.

## HTTP 요청 메시지 - JSON

이번에는 HTTP API에서 주로 사용하는 JSON 데이터 형식을 조회해본다.

### V1 - Servlet

```java
@Slf4j
@Controller
public class RequestBodyJsonController {

    private ObjectMapper mapper = new ObjectMapper();

    @PostMapping("/request-body-json-v1")
    public void requestBodyJsonV1(HttpServletRequest req, HttpServletResponse res) throws IOException {
        ServletInputStream inputStream = req.getInputStream();
        String message = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("message = {}", message);
        HelloData helloData = mapper.readValue(message, HelloData.class);
        log.info("username = {}, age = {}", helloData.getUsername(), helloData.getAge());

        res.getWriter().write("ok");
    }
}
```

- HttpServletRequest를 사용해서 직접 HTTP 메시지 바디에서 데이터를 읽어오고, 문자로 변환한다.
- 문자로 된 JSON 데이터를 Jackson 라이브러리인 `objectMapper`를 사용해서 자바 객체로 변환한다.

### V2 - @RequestBody, @ResponseBody

```java
@ResponseBody
@PostMapping("/request-body-json-v2")
public String requestBodyJsonV2(@RequestBody String message) throws IOException {
    log.info("message = {}", message);
    HelloData helloData = mapper.readValue(message, HelloData.class);

    log.info("username = {}, age = {}", helloData.getUsername(), helloData.getAge());

    return "ok";
}
```

### V3 - @RequestBody 객체 변환

```java
@ResponseBody
@PostMapping("/request-body-json-v3")
public String requestBodyJsonV3(@RequestBody HelloData helloData) {
    log.info("username = {}, age = {}", helloData.getUsername(), helloData.getAge());
    return "ok";
}
```

- `@RequestBody HelloData data`
- `@RequestBody`에 직접 만든 객체를 지정할 수 있다.

`HttpEntity` 또는 `@RequestBody`를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.

HTTP 메시지 컨버터는 문자 뿐만 아니라 JSON도 객체로 변환해주는데, 방금 V2에서 했던 작업을 대신 처리해준다.

> [!warning] @RequestBody 생략 불가
> JSON 형식의 HTTP 요청 메세지 바디를 `@RequestBody`를 통해 자바 객체로 변환할 경우에 `@RequestBody` 어노테이션을 생략할 수 없다.
>
> 만약 `@RequestBody` 어노테이션을 생략해버릴 경우에 `@ModelAttribute` 어노테이션이 자동으로 적용되기 때문이다. 따라서 요청 파라미터가 아닌 JSON의 경우에는 생략할 수 없다.
>
> 이 경우 HelloData에 `@RequestBody` 를 생략하면 `@ModelAttribute` 가 적용되어버린다.
>  따라서 생략하면 HTTP 메시지 바디가 아니라 요청 파라미터를 처리하게 된다.

> [!warning] HTTP 요청의 content-type
> HTTP 요청 시 content-type이 application/json인지 확인해야 한다. 그래야 JSON을 처리할 수 있는 HTTP 메시지 컨버터가 실행된다.
>
> 만약 컨텐츠 타입이 올바르지 않다면 415 에러가 발생한다. (Unsupported Media Type)

### V4 - HttpEntity

```java
@ResponseBody
@PostMapping("/request-body-json-v4")
public String requestBodyJsonV4(HttpEntity<HelloData> helloData) {
    log.info("username = {}, age = {}", helloData.getBody().getUsername(), helloData.getBody().getAge());
    return "ok";
}
```

앞서 학습했던 HttpEntity를 이용해서 메시지 바디를 획득하여 사용한다.

### V5 - @ResponseBody JSON 반환

```java
@ResponseBody
@PostMapping("/request-body-json-v5")
public HelloData requestBodyJsonV5(@RequestBody HelloData helloData) {
    log.info("username = {}, age = {}", helloData.getUsername(), helloData.getAge());
    return helloData;
}
```

응답의 경우에도 `@ResponseBody`를 사용하면 해당 객체를 HTTP 메세지 바디에 직접 넣어줄 수 있다. 물론 이 경우에도 `HttpEntity`를 사용해도 된다. 다만 이 방법이 편리하다.

- 요청 (@RequestBody): JSON 요청 -> HTTP 메시지 컨버터 -> 객체
- 응답 (@ResponseBody): 객체 -> HTTP 메시지 컨버터 -> JSON 응답

> [!warning] HTTP 요청 헤더 Accept의 영향
> HTTP 요청 헤더의 Accept가 application/json이 아니면 @ResponseBody에서 어떤 HTTP 메시지 컨버터가 선택될 지 정할 수 없으므로 영향을 끼친다.

## HTTP 응답 - 정적 리소스, 뷰 템플릿

응답 데이터는 앞서 일부 다룬 내용들이지만, 응답 부분에 초점을 맞추어 정리한다. 스프링에서 응답 데이터를 만드는 방법은 크게 3가지이다.

- 정적 리소스
  - 예) 웹 브라우저에 정적인 HTML, CSS, JS를 제공할 때는 정적 리소스를 사용
- 뷰 템플릿
  - 예) 웹 브라우저에 동적인 HTML을 제공할 때는 뷰 템플릿 사용
- HTTP 메시지
  - HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달하므로, HTTP 메시지 바디에 JSON 같은 형식으로 데이터를 실어 보낸다.

### 정적 리소스

스프링 부트는 클래스패스의 다음 디렉토리에 있는 정적 리소스를 제공한다.

- `/static` , `/public` , `/resources` ,`/META-INF/resources`

`src/main/resources` 는 리소스를 보관하는 곳이고, 또 클래스패스의 시작 경로이다. 따라서 다음 디렉토리에 리소스를 넣어두면 스프링 부트가 정적 리소스로 서비스를 제공한다.

#### 정적 리소스 경로

- `src/main/resources/static`

다음 경로에 파일이 들어있으면
`src/main/resources/static/basic/hello-form.html`

웹 브라우저에서 다음과 같이 요청을 보내면 정적 리소스를 서비스한다.
`http://localhost:8080/basic/hello-form.html`

### 뷰 템플릿

뷰 템플릿을 거쳐 HTML이 생성되고, 뷰가 응답을 만들어서 반환한다.

일반적으로 HTML을 동적으로 생성하는 용도로 사용하지만, 다른 것들도 가능하다. 뷰 템플릿이 만들 수 있는 것이라면 뭐든지 가능하다.

스프링 부트는 기본 뷰 템플릿 경로를 제공한다.

#### 뷰 템플릿 경로

`src/main/resources/templates`

#### 뷰 템플릿 생성

```html
<!doctype html>
<html lang="ko" xmlns:th="http://www.thymeleaf.org">
  <head>
    <meta
      name="viewport"
      content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0"
    />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Title</title>
  </head>
  <body>
    <p th:text="${data}">Empty</p>
  </body>
</html>
```

### V1 - ModelAndView

```java
@RequestMapping("/response-view-v1")
public ModelAndView responseViewV1() {
    ModelAndView mv = new ModelAndView("response/hello")
            .addObject("data", "hello!");

    return mv;
}
```

### V2 - Model

```java
@RequestMapping("/response-view-v2")
public String responseViewV2(Model model) {
    model.addAttribute("data", "hello!");

    return "response/hello";
}
```

**String을 반환하는 경우의 반환**

- View
- HTTP 메세지

`@ResponseBody`가 없으면 `String` 타입의 `response/hello`로 뷰 리졸버가 실행되어서 뷰를 찾고, 렌더링 한다.

`@ResponseBody`가 있으면 뷰 리졸버를 실행하지 않고, HTTP 메세지 바디에 직접 `response/hello`라는 문자가 입력된다.

**void를 반환하는 경우**

- `@Controller`를 사용하고, `HttpServletResponse`, `OutputStream(Writer)`같은 HTTP 메세지 바디를 처리하는 파라미터가 없으면 요청 URL을 참고해서 논리 뷰 이름으로 사용한다.
  - 요청 URL: `/response/hello`
  - 실행: `templates/response/hello.html`
- 이 방식은 명시성이 너무 떨어지고, 이렇게 딱 맞는 경우도 많이 없어서 권장하지 않는다.

**HTTP 메세지**

`@ResponseBody`, `HttpEntity`를 사용하면 뷰 템플릿을 사용하는게 아니라, HTTP 메세지 바디에 직접 응답 데이터를 출력할 수 있다.

## Thymeleaf 스프링 부트 설정

타임리프 라이브러리를 추가하면 스프링 부트가 자동으로 `ThymeleafViewResolver`와 필요한 스프링 빈들을 등록한다. 그리고 다음 설정도 사용한다. 이 설정은 기본 값이기 때문에 변경이 필요할 때만 설정하면 된다.

```properties title="application.properties"
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
```

---

References: 김영한의 스프링 MVC 1편

Links to this page: [[Wrapper 클래스]]
