# 간단 사용법

## 사용 선언
```html
<html xmlns:th="http://www.thymeleaf.org">
```

## 속성 변경 `th:href`
```html
th:href="@{/css/bootstrap.min.css}"
```
* `href`의 값을 `th:href`의 값으로 변경한다.
* Thymeleaf의 뷰 템플릿을 거치게 되면 원래 값을 `th:xxx` 값으로 변경한다. 원래 값이 없다면 새로 생성ㅎ나다.
* HTML을 그대로 볼땐 `href`가, 뷰 템플릿을 거치면 `th:href` 값이 `href`를 대체한다.
* 대부분의 HTML 속성을 `th:xxx`로 변경할 수 있다.

## Thymeleaf의 핵심
* 