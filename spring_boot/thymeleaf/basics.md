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
* Thymeleaf의 뷰 템플릿