---
title:
tags:
  - java
  - programming
  - spring
  - mvc
publish: true
date: 2024-11-24
---

## PRG Post, Redirect, Get

다음의 상품 등록 처리 컨트롤러는 심각한 문제가 있다. 상품 등록을 완료하고 웹 브라우저의 새로고침 버튼을 클릭해보면 상품이 계속해서 중복 등록 되는 것을 알 수 있다.

```java
@Controller
@RequestMapping("/basic/items")
@RequiredArgsConstructor
public class BasicItemController {
    private final ItemRepository itemRepository;

    @GetMapping
    public String items(Model model) {
        List<Item> items = itemRepository.findAll();
        model.addAttribute("items", items);
        return "basic/items";
    }

    @GetMapping("/{itemId}")
    public String item(@PathVariable long itemId, Model model) {
        Item item = itemRepository.findById(itemId);
        model.addAttribute("item", item);
        return "basic/item";
    }

    @GetMapping("/add")
    public String addForm() {
        return "basic/addForm";
    }

    @PostMapping("/add")
    public String save(Item item) {
        itemRepository.save(item);
        return "basic/item";
    }

    @GetMapping("/{itemId}/edit")
    public String editForm(@PathVariable long itemId, Model model) {
        Item item = itemRepository.findById(itemId);

        model.addAttribute("item", item);
        return "basic/editForm";
    }

    @PostMapping("/{itemId}/edit")
    public String edit(@PathVariable long itemId, Item item) {
        itemRepository.update(itemId, item);
        return "redirect:/basic/items/{itemId}";
    }

    @PostConstruct
    public void init() {
        itemRepository.save(new Item("itemA", 10000, 10));
        itemRepository.save(new Item("itemB", 20000, 20));
    }
}
```

### 새로 고침 중복 등록 문제

![[prg-1.png]]

웹 브라우저의 새로고침은 마지막에 서버에 전송한 데이터를 다시 전송한다.

상품 등록 폼에서 데이터를 입력하고 저장을 선택하면 `POST /add`로 상품 데이터를 서버로 전송한다.
이 상태에서 새로고침을 또 선택하면 마지막에 전송한 `POST /add`로 다시 데이터를 전송한다. 그래서 내용은 같고 ID만 다른 상품 데이터가 계속 쌓이게 된다.

이 문제를 어떻게 해결할 수 있을까?

![[prg-2.png]]

새로 고침 문제를 해결하려면 상품 저장 후에 뷰 템플릿으로 이동하는게 아니라, 상품 상세 화면으로 리다이렉트를 호출해주면 된다.

웹 브라우저는 리다이렉트의 영향으로 상품 저장 후에 실제 상품 상세 화면으로 다시 이동한다. 따라서 마지막에 호출한 내용이 상품 상세 화면인 `GET /items/{id}`가 되는 것이다. 이후엔 새로고침을 해도 상품 상세 화면으로 이동하게 되므로 새로 고침 중복 등록 문제를 해결할 수 있다.

상품 등록 처리 이후에 뷰 템플릿이 아니라 상품 상세화면으로 리다이렉트 하도록 코드를 수정한다. 이런 문제 해결 방식을 PRG Post, Redirect, Get이라 한다.

- POST 요청을 리다이렉트해서, GET 요청으로 보낸다.

```java
@PostMapping("/add")
public String save(Item item) {
    itemRepository.save(item);

    return "redirect:/basic/items/" + item.getId();
}
```

이제 문제가 있던 코드를 PRG를 통해 해결했다. 그런데 이 코드엔 문제가 있다. redirect에서 `+ item.getId()`처럼 URL에 변수를 더하고 있다. 이렇게 하면 URL 인코딩이 안되기 때문에 위험하다. 따라서 `RedirectAttribute`를 이용해야 한다.

## RedirectAttributes

상품을 저장하고 상품 상세 화면으로 리다이렉트 한 것 까지는 좋았다. 그런데 고객 입장에서 저장이 잘 된 것인지 안 된 것인지 확신이 들지 않는다. 그래서 저장이 잘 되었으면 상품 상세 화면에 "저장되었습니다"라는 메세지를 보여달라는 요구사항이 왔다. 간단하게 해결해본다.

```java
@PostMapping("/add")
public String save(Item item, RedirectAttributes redirectAttributes) {
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);

    return "redirect:/basic/items/{itemId}";
}
```

`RedirectAttribute`를 사용하면 리다이렉트 할 때 위의 예제처럼 여러 속성을 추가할 수 있다.

- `itemId`: 아이템 id를 리다이렉트 시 사용하기 위해 추가한다.
- `status`: 뷰 템플릿에서 이 값이 있으면 저장되었습니다. 하는 메세지를 출력하기 위해 추가한 속성이다.

다시 아이템 저장을 해보면 다음과 같은 리다이렉트 결과가 나온다.

`http://localhost:8080/basic/items/3?status=true`

### RedirectAttribute

`RedirectAttribute`를 사용하면 URL 인코딩도 해주고, `pathVariable`, `쿼리 파라미터`까지 처리해준다.

- redirect:/basic/items/{itemId} 는 다음과 같이 변경된다.
  - pathVariable 바인딩: `{itemId}`
  - 나머지는 쿼리 파라미터로 처리: `?status=true`

### 뷰 템플릿 메세지 추가

```html
<div class="py-5 text-center">
  <h2>상품 상세</h2>
</div>

<h2 th:if="${param.status}" th:text="'저장 완료!'"></h2>
```

- `th:if`: 해당 조건이 참이면 실행
- `${param.status}`: 타임리프에서 쿼리 파라미터를 편리하게 조회하는 기능
  - 원래는 컨트롤러에서 모델에 직접 담고 값을 꺼내야한다. (EJS의 경우를 떠올려보자) 그런데 쿼리 파라미터는 자주 사용해서 타임리프에서 직접 지원한다.

뷰 템플릿에 메세지를 추가하고 실행해보면 "저장 완료!"라는 메세지가 나오는 것을 확인할 수 있다. 물론 상품 목록에서 상품 상세로 이동할 경우에는 해당 메세지가 출력되지 않는다.

---

References: 김영한의 스프링 MVC 1편

Links to this page:
