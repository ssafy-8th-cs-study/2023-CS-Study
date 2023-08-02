숨쉬듯 사용해온 @RequestParam, @RequestBody, @AuthenticationPrincipal 등등.
우리에게 편리함을 안겨주는, 파라미터로 받는 이 Annotation들은 어째서 사용이 가능한걸까?

Spring Boot에는 HandlerMethodArgumentResolver를 implements한 많은 구현체가 존재하기 때문이다.

예시)
@RequestParam: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/method/annotation/RequestParamMethodArgumentResolver.html
@RequestBody: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/result/method/annotation/RequestBodyMethodArgumentResolver.html

http 요청 파라미터나, form 데이터 등을 간편하게 받아쓸 수 있게 해주는 interface, WebArgumentResolver와 HandlerMethodArgumentResolver를 알아보자.

1편: WebArgumentResolver

https://velog.io/@l_dh/WebArgumentResolver
