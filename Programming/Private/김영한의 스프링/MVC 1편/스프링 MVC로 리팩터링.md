---
title: 
tags:
  - java
  - programming
  - spring
  - mvc
publish: true
date: 2024-11-22 00:01
---

## 스프링 MVC - 시작하기

스프링이 제공하는 컨트롤러는 어노테이션 기반으로 동작해서, 매우 유연하고 실용적이다. 과거에는 자바 언어에 어노테이션이 없기도 했고, 스프링도 처음부터 이런 유연한 컨트롤러를 제공한 것은 아니다.

**@RequestMapping**

스프링은 어노테이션을 활용한 매우 유연하고 실용적인 컨트롤러를 만들었는데, 이것이 바로 `@RequestMapping`이다. 과거에는 스프링 프레임워크가 MVC 부분이 약해서 스프링을 사용하더라도 MVC 웹 기술은 스트럿츠 같은 다른 프레임워크를 사용했었다. 그런데 `@RequestMapping` 기반의 어노테이션 컨트롤러가 등장하면서 MVC 부분도 스프링의 완승으로 끝이 났다.

지금까지 만들었던 프레임워크에서 사용했던 컨트롤러를 `@RequestMapping` 기반의 스프링 MVC 컨트롤러로 변경해보면서 본격적으로 어노테이션 기반의 컨트롤러를 사용해본다.

```java title="SpringMemberFormControllerV1.java"
@Controller
public class SpringMemberFormControllerV1 {
    @RequestMapping("/springmvc/v1/members/new-form")
    public ModelAndView process() {
        return new ModelAndView("new-form");
    }
}
```

- `@Controller`
  - 스프링이 자동으로 스프링 빈으로 등록한다. 관련 참고 문서는 [[컴포넌트 스캔과 의존관계 자동 주입#컴포넌트 스캔 기본 대상|컴포넌트 스캔 기본 대상]]을 참고하면 된다. (내부에 @Component 어노테이션이 있다.)
  - 스프링 MVC에서 어노테이션 기반 컨트롤러로 인식한다.
- `@RequestMapping`
  - 요청 정보를 매핑한다. 해당 URL이 호출되면 이 메서드가 호출된다. 어노테이션을 기반으로 동작하기 때문에, 메서드의 이름은 임의로 지으면 된다.
- `ModelAndView`
  - 모델과 뷰 정보를 담아서 반환하면 된다.

```java title="SpringMemberSaveControllerV1.java"
@Controller
public class SpringMemberSaveControllerV1 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/springmvc/v1/members/save")
    public ModelAndView process(HttpServletRequest request, HttpServletResponse
            response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));
        Member member = new Member(username, age);

        memberRepository.save(member);

        ModelAndView mv = new ModelAndView("save");
        mv.addObject("member", member);
        return mv;
    }
}
```

```java title="SpringMemberListControllerV1.java"
@Controller
public class SpringMemberListControllerV1 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/springmvc/v1/members")
    public ModelAndView process() {
        List<Member> members = memberRepository.findAll();
        ModelAndView mv = new ModelAndView("members");

        mv.addObject("members", members);
        return mv;
    }
}
```

> [!summary] 핵심
> `@Controller` 어노테이션이 클래스 레벨에 적용되어 있으면 스프링 빈으로 인식되고, `RequestMappingHandlerMapping`에 등록된다.
>
> 그리고 `RequestMappingHandlerAdapter`에서 해당 컨트롤러를 찾아서 호출하는 것이다.

## 스프링 MVC - 컨트롤러 통합

`@RequestMapping`을 살펴보면 클래스 단위가 아니라 메서드 단위에 적용된 것을 확인할 수 있다. 따라서 컨트롤러 클래스를 유연하게 하나로 통합할 수 있다.

```java
@Controller
@RequestMapping("springmvc/v2/members")
public class SpringMemberControllerV2 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/new-form")
    public ModelAndView newForm() {
        return new ModelAndView("new-form");
    }

    @RequestMapping("/save")
    public ModelAndView save(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelAndView mv = new ModelAndView("save");
        mv.addObject("member", member);
        return mv;
    }

    @RequestMapping
    public ModelAndView members() {
        List<Member> members = memberRepository.findAll();

        ModelAndView mv = new ModelAndView("members");
        mv.addObject("members", members);

        return mv;
    }
}
```

클래스 레벨의 `@RequestMapping`과 메서드 레벨의 `@RequestMapping`을 조합하여 중복을 제거하고 컨트롤러를 하나로 통합했다.

## 스프링 MVC - 실용적인 방식

MVC 프레임워크 만들기에서 v3는 `ModelView`를 개발자가 직접 생성해서 반환했기 때문에, 불편했던 기억이 날 것이다.

스프링 MVC는 개발자가 편리하게 개발할 수 있도록 수 많은 편의 기능을 제공한다. **실무에서는 지금부터 설명하는 방식을 주로 사용한다.**

```java
@Controller
@RequestMapping("/springmvc/v3/members")
public class SpringMemberControllerV3 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @GetMapping("/new-form")
    public String newForm() {
        return "new-form";
    }

    @PostMapping("/save")
    public String save(
            @RequestParam("username") String username,
            @RequestParam("age") int age,
            Model model
    ) {
        Member member = new Member(username, age);
        memberRepository.save(member);

        model.addAttribute("member", member);
        return "save";
    }

    @GetMapping
    public String members(Model model) {
        List<Member> members = memberRepository.findAll();
        model.addAttribute("members", members);

        return "members";
    }
}
```

어노테이션 기반 컨트롤러 기능들을 활용하면 지금까지 구현했던 어떤 컨트롤러보다도 깔끔하다.

- `Model`
  - `save()`, `members()`를 보면 `Model`을 파라미터로 받는 것을 확인할 수 있다. 스프링 MVC도 이런 편의기능을 제공한다.
- `ViewName 직접 반환`
  - 뷰의 논리 이름을 메서드에서 편리하게 직접 반환할 수 있다.
- `@RequestParam`
  - 스프링은 HTTP 요청 파라미터를 `@RequestParam` 어노테이션으로 받을 수 있다.
  - `@RequestParam("username")`은 `request.getParameter("username")`과 거의 같은 코드라고 생각하면 된다.
  - 물론 `GET 쿼리 파라미터`, `POST Form` 방식 모두 지원한다.
- `@RequestMapping` -> `@GetMapping`, `@PostMapping`
  - `@RequestMapping`은 URL만 매칭하는 것이 아니라 HTTP Method도 함께 구분할 수 있다.
  - 이것을 `@GetMapping`, `@PostMapping`등의 HTTP 메소드 어노테이션으로 편리하게 사용할 수 있다.
  - `@GetMapping` 어노테이션을 확인해보면 다음과 같이 `@RequestMapping` 어노테이션을 내부에 갖고 있다.

---

References: 김영한의 스프링 MVC 1편

Links to this page: [[컴포넌트 스캔과 의존관계 자동 주입]]
