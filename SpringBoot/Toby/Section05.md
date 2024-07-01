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

좀 더 간편하게 직접적으로 코드를 테스트하는 방법  
SimpleHelloService는 @Service가 붙어있긴 하지만, 그 안의 로직을 검증하는 코드를 만드는 건 어렵지 않음  
그리고 이 때는 이전에 테스트를 했을 때처럼 서버를 실행시킬 필요가 없음

HelloServiceTest 생성

```java
// HelloServiceTest.java
public class HelloServiceTest {
    @Test
    void simpleHelloService() {
        SimpleHelloService helloService = new SimpleHelloService();

        String res = helloService.sayHello("Spring");

        assertThat(res).isEqualTo("Hello, Spring!");
    }
}
```
- 오브젝트를 생성하고 메소드를 바로 호출하여 결과값 검증

이전 helloApiTest는 네트워크 구간을 통해서 HTTP 요청이 날라가고 응답을 받아서 파싱, 바인딩 작업들이 다 진행되어야 함  
테스트가 간결하지만, 수행속도가 오래 걸릴 수 있음  
그에 비해, HelloServiceTest는 Java 인스턴스 하나 만들고 메소드 호출하고 검증 끝이라 작업이 간결하고 시간이 짧게 걸림  
당연히 서버가 띄워져있을 필요도 없음

이런 방식의 테스트는 고립된 테스트가 가능하다는 장점이 있음  
예를 들기 위해 HelloControllerTest 를 생성하여 비교해봄

```java
// HelloControllerTest.java
public class HelloControllerTest {
    @Test
    void helloController() {
        HelloController controller = new HelloController(name -> name);

        String res = controller.hello("Test");

        assertThat(res).isEqualTo("Test");
    }
}
```
- HelloController를 테스트하려면 HelloService가 필요함
  - HelloController 오브젝트를 생성하려면 HelloService가 필요하기 때문
  - HelloService 의존성을 제거하여 고립된 테스트를 진행
- HelloService가 호출되는 구간을 간단하게 만들어서 HelloController의 파라미터로 전달
  - 이렇게 HelloService를 간단하게 만든 것을 Test Stub이라고 함
  - 여기서는 HelloService의 메소드가 하나이므로 람다표현식으로 진행

비정상적인 상황도 가정해봐야 함  
HelloController의 경우 name 파라미터로 null이 넘어오는 경우가 있으므로 해당 상황을 검증해 봄

```java
// HelloControllerTest.java
@Test
void failHelloController() {
    HelloController controller = new HelloController(name -> name);

    assertThatThrownBy(() -> controller.hello(null))
        .isInstanceOf(NullPointerException.class);
}
```
- null이 들어가서 예외가 발생하면 테스트가 성공하고, 예외가 발생하지 않으면 테스트가 실패하도록 구현
  - `Assertions.assertThatThrownBy()` 활용
  - `isInstanceOf()`로 Exception의 타입도 검증
- HelloController의 null 검증하는 부분 `Objects.requireNonNull()`을 제거하고 테스트해보면 테스트가 실패

null이 아닌 emptyString ""을 파라미터로 전달하면 테스트가 성공  
emptyString의 경우 정상적인 상황이 아니므로 HelloController에서 emptyString도 예외가 발생하도록 코드를 수정

```java
// HelloController.java
@GetMapping("/hello")
public String hello(String name) {
    if (name == null || name.isBlank()) {
        throw new IllegalArgumentException();
    }

    // return service.sayHello(Objects.requireNonNull(name));
    return service.sayHello(name);
}
```

위 코드를 수정한 후 테스트 코드에서도 emptyString을 전달했을 때 예외가 발생하는지 검증 추가

```java
// HelloControllerTest.java
@Test
void failHelloController() {
    HelloController controller = new HelloController(name -> name);

    assertThatThrownBy(() -> controller.hello(null))
        .isInstanceOf(IllegalArgumentException.class);

    assertThatThrownBy(() -> controller.hello(""))
        .isInstanceOf(IllegalArgumentException.class);
}
```

이런 식의 테스트를 Unit Test 혹은 단위 테스트라고 함  
어떤 환경에 의존하지 않고 하나의 대상, 주로 하나의 클래스 정도로 테스트를 빠르게 진행할 수 있도록 만드는 것

HTTP 요청을 직접 보내서 응답을 받는 API 테스트 레벨에서도 예외 상황에 대한 테스트가 가능

```java
// HelloApiTest.java
@Test
void failHelloApi() {
    TestRestTemplate rest = new TestRestTemplate();

    ResponseEntity<String> res =
        rest.getForEntity("http://localhost:8080/hello?name=", String.class);

    assertThat(res.getStatusCode()).isEqualTo(HttpStatus.INTERNAL_SERVER_ERROR);
}
```
- API 요청을 보낼때 name 파라미터를 전달하지 않음
- 일반적인 예외가 발생하면 서블릿 컨테이너는 응답코드 500 INTERNAL_SERVER_ERROR로 매칭해줌
  - 예외를 잡지는 않고 응답코드 500을 받았는지 검증
- 웹 클라이언트가 요청을 잘못 준 경우는 500이 아닌 400 응답을 주로 함
  - 400 응답을 주기 위해서는 Controller를 수정하거나, 400으로 응답하라고 스프링 컨테이너에게 알려줘야 함 (추후 학습)

## DI를 이용한 Decorator, Proxy 패턴
