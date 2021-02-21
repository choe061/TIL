## 목차

[1. DomainClassConverter](#DomainClassConverter)

[2. HandlerMethodArgumentResolver](#HandlerMethodArgumentResolver)

## [DomainClassConverter](https://docs.spring.io/spring-data/jpa/docs/2.2.7.RELEASE/reference/html/#core.web.basic)
* request parameters or path variables 을 repository 로 domain 객체로 풀어줌
* 따로 Converter<S, T> 또는 HandlerMethodArgumentResolver 를 따로 구현하지 않아도 Spring DATA-JPA 가 PathVariable 또는 RequestParam 으로 들어온 값을 entity 의 id 로 판단하여 해당 domain 의 repository 로 조회
  * @PathVariable("id"), @RequestParam("id") → id 타입으로 들어온 값을 이용
  * Repository 는 CurdRepository 를 구현해야 하고, findById 로 도메인을 조회

## HandlerMethodArgumentResolver
* DomainClassConverter 는 조회의 기능만 가능하고, 추가적인 핸들링이 불가능하다. 무언가 추가 작업을 하고싶다면, HandlerMethodArgumentResolver 를 구현해야 한다.
