# `@ModelAttribute`

`@RestController`에서 `@ModelAttribute`는 URL 쿼리 파라미터(Query Parameter)나 폼 데이터(Form Data)를 객체(DTO)에 자동으로 바인딩(mapping)하는 데 사용됩니다. JSON을 사용하는 `@RequestBody`와는 역할이 명확히 다릅니다.

## 1. `@ModelAttribute`의 핵심 역할: 파라미터 바인딩

`@RestController` 환경에서 `@ModelAttribute`의 주된 임무는 흩어져 있는 요청 파라미터들을 지정된 객체의 필드에 깔끔하게 담아주는 것입니다.

바인딩을 위해서는 기본 생성자와 각 필드에 대한 setter 메서드가 필요합니다.

### `@RequestBody`와의 차이점

이 둘의 차이를 이해하는 것이 가장 중요합니다.

- `@ModelAttribute`: `application/x-www-form-urlencoded` (일반 폼 전송) 또는 `multipart/form-data` (파일 업로드 포함 폼 전송), 그리고 GET 요청의 쿼리 파라미터(?name=...&city=...)를 처리합니다.
- `@RequestBody`: `application/json`, `application/xml` 등 요청 본문(Request Body)에 담긴 데이터를 처리합니다.

현대 REST API에서는 대부분의 POST, PUT 요청에 JSON을 사용하므로 `@RequestBody`를 훨씬 더 많이 사용하게 됩니다. `@ModelAttribute`는 주로 복잡한 검색 조건이 있는 GET 요청이나 파일 업로드가 있는 POST 요청에 유용합니다.
