# Section 05. DI와 테스트, 디자인 패턴

## 테스트 코드를 이용한 테스트

지금까진 작성한 코드가 제대로 동작하는지 확인하기 위해서  
서버를 동작시키고, 요청을 보내서 원하는 응답을 받는지 확인했음  
이렇게 테스트하는건 번거로움. 그래서 테스트코드를 활용해서 테스트를 진행해봄

SpringBoot를 생성하며 만들어진 TobyApplicationTest는  
애플리케이션이 기본적으로 SpringBoot 기반으로 동작하는 것을 전제로 하기 때문에 해당 테스트를 사용하지 않고 제거

기존에 만든 Controller, Service 를 모두 한 파일에서 테스트할 예정이므로 HelloApiTest 새로 생성

```java
public class HelloApiTest {
    @Test
    void helloApi() {
        TestRestTemplate rest = new TestRestTemplate();

        ResponseEntity<String> res =
            rest.getForEntity("http://localhost:8080/hello?name={name}", String.class, "Spring");

        assertThat(res.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(res.getHeaders().getFirst(HttpHeaders.CONTENT_TYPE)).startsWith(MediaType.TEXT_PLAIN_VALUE);
        assertThat(res.getBody()).isEqualTo("Hello, Spring!");
    }
}
```
- 기존에 요청 보내는 `http localhost:8080/hello?name=Spring` 요청을 TestRestTemplate을 활용하여 테스트
  - RestTemplate을 사용하면, 200 Ok 응답은 문제가 없지만 400, 500 응답일 경우에는 예외를 던짐
- 파라미터 name은 변동될 여지가 많기 때문에 플레이스홀더(`{}`)를 사용
- 응답은 AssertJ 라는 라이브러리를 활용해서 검증
  - `Assertions.assertThat().isEqualTo()`
  - 테스트를 진행할 때는 예상값과 결과값의 타입이 동일한지를 주의해야함
  - Content-Type 검증시에는 인코딩 정보가 포함되어 있기 때문에 `isEqualTo()`가 아닌 `startsWith()`를 활용

다만 위 방법은 서버가 실행중이어야 함

## DI와 단위 테스트

## DI를 이용한 Decorator, Proxy 패턴
