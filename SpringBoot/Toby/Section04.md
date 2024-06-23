# Section 04. 독립 실행형 스프링 애플리케이션

## 스프링 컨테이너 사용

지금까진 서블릿 컨테이너를 띄웠고, 서블릿 컨테이너 내에 Front Controller를 만들고  
외부에 Hello Controller를 Front Controller와 만들어서 연결했음.

이제 앞서 만든 Hello Controller를 스프링 컨테이너 내에 추가하고  
Front Controller가 직접 Hello Controller를 생성하고 변수에 담아뒀다가 사용하는 대신  
스프링 컨테이너를 이용하는 방식으로 변경

스프링 컨테이너는 서블릿 컨테이너와 작업하는 방식이 다름  
스프링 컨테이너는 비즈니스 오브젝트(POJO)와 만들어진 코드들을 어떤 식으로 구성할 지 구성 정보를 담고있는 Configuration Metatdata가 필요

Front Controller의 코드 중 필요없는 부분 우선 수정  
```java
WebServer webServer = serverFactory.getWebServer(servletContext -> {
    servletContext.addServlet("hello", new HttpServlet() {
        @Override
        protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            if (req.getRequestURI().equals("/hello") && req.getMethod().equals(HttpMethod.GET.name())) {
                String name = req.getParameter("name");

                // String ret = helloController.hello(name);

                resp.setContentType(MediaType.TEXT_PLAIN_VALUE);
                resp.getWriter().println(ret);
            } else {
                resp.setStatus(HttpStatus.NOT_FOUND.value());
            }
        }
    }).addMapping("/*");
});
```
- 사용하지 않는 else if 문 제거
- HttpStatus 값은 따로 설정하지 않으면 200 Ok로 응답
- Content-Type을 Header가 아니라 setContentType으로 설정

스프링 컨테이너를 사용하도록 코드 수정

우선 HelloController 생성하는 부분을 스프링 컨테이너를 생성하도록 수정
```java
// 기존 - HelloController 사용
// HelloController helloController = new HelloController();

// 변경 - 스프링 컨테이너 사용
GenericApplicationContext context = new GenericApplicationContext();
context.registerBean(HelloController.class);
context.refresh();
```
- 스프링 컨테이너 = Application Context
  - 애플리케이션이라면 필요한 많은 작업들을 수행해주는 기능을 담고 있는 오브젝트들을 구현해야 하는 것이 애플리케이션 컨텍스트
  - GenericApplicationContext 코드에 의해서 손쉽게 만들수 있도록 만들어진 Class
- 스프링은 어떤 클래스를 이용해서 Bean 오브젝트를 생성할 것인가 메타정보를 넣어주는 방식으로 구성
  - Bean 등록하는데 일반적인 방법은 Bean 클래스를 지정하는 방법으로 사용
- 자기가 가지고 있는 구성 정보를 이용해서 컨테이너를 초기화하는 작업을 refresh()로 호출

서블릿 컨테이너에서 스프링 컨테이너의 어떤 Bean을 호출하는지 선언하고 변수에 할당  
나머지 작업은 동일 (바인딩, 메소드 호출, 응답 등)
```java
WebServer webServer = serverFactory.getWebServer(servletContext -> {
    servletContext.addServlet("hello", new HttpServlet() {
        @Override
        protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            if (req.getRequestURI().equals("/hello") && req.getMethod().equals(HttpMethod.GET.name())) {
                String name = req.getParameter("name");

                HelloController helloController = context.getBean(HelloController.class);
                String ret = helloController.hello(name);

                resp.setContentType(MediaType.TEXT_PLAIN_VALUE);
                resp.getWriter().println(ret);
            } else {
                resp.setStatus(HttpStatus.NOT_FOUND.value());
            }
        }
    }).addMapping("/*");
});
```

## 의존 오브젝트 추가

위 방법은 FrontController가 직접 HelloController를 생성해서 사용하는 방식과 크게 다르지 않음. 다만 스프링의 기본 구조를 짜놨다는점을 주목  
스프링 컨테이너는 기본적으로 안에 어떤 타입의 오브젝트를 만들때 한번만 생성함. (이게 스프링에서 자주나오는 싱글톤)  
getBean을 호출할때마다 새로 오브젝트를 생성해서 전달하는게 아니라 한 번만 생성해둔 다음 생성해둔 오브젝트를 getBean 호출때마다 전달함(재사용)
그래서 스프링 컨테이너를 싱글톤 레지스트리라고도 함

컨트롤러는 기본적으로 웹 컨트롤러이기 때문에 웹을 통해서 들어온 요청사항을 한번 검증하고  
비즈니스 로직을 제공해주는 다른 오브젝트한테 요청을 보내서 결과를 받은 다음 웹 클라이언트에게 어떤 형식으로 돌려줄 것인가를 결정하는 역할만 하면 됨

비즈니스 로직을 처리할 SimpleHelloService를 추가
```java
public class SimpleHelloService {

    public String sayHello(String name) {
        return "Hello, " + name + "!";
    }
}
```

SimpleHelloService를 추가하며 HelloController는 SimpleHelloService에 요청을 보내서 처리하도록 수정
```java
public class HelloController {

    public String hello(String name) {
        SimpleHelloService service = new SimpleHelloService();
        
        return service.sayHello(name);
    }
}
```

Controller는 웹 요청을 검증하는 역할을 하기 때문에 name이 잘못들어왔는지 검증하는 로직을 추가
```java
return service.sayHello(Objects.requireNonNull(name));
```
- `Objects.requireNonNull()`는 전달받은 파라미터가 null일 경우 NPE를 throw

서버를 실행하고 응답이 기대한 대로 들어왔는지 확인
```bash
$ http -v GET ":8080/hello?name=Spring"

HTTP/1.1 200 
Connection: keep-alive
Content-Length: 15
Content-Type: text/plain;charset=ISO-8859-1
Date: Thu, 20 Jun 2024 11:24:44 GMT
Keep-Alive: timeout=60

Hello, Spring!
```

## Dependency Injection

스프링 컨테이너를 얘기할 때 Spring IoC/DI Container 혹은 스프링 DI 컨테이너라고도 얘기함

앞서 만든 HelloController와 SimpleHelloService는 의존관계가 성립되어 있음  
SimpleHelloService의 변경이 HelloController에 영향을 줌 (HelloController가 SimpleHelloService에 의존하고 있음)

이게 어떤 문제가 되는지?  
ComplexHelloService를 SimpleHelloService 대신 사용하고 싶을 경우, HelloController의 코드를 수정해야 함  
변경이 발생할 때마다 코드를 수정하고, 배포를 다시 해야함

이런 문제를 어떻게 해결하는지?  
Java에서 많이 쓰이는 소프트웨어 원칙이 있음  
HelloController는 특정 메소드를 가지고 있는 HelloService라는 인터페이스에만 의존하도록 만드는 것  
그리고 인터페이스를 구현한 SimpleHelloService, ComplexHelloService를 생성  
이렇게 진행한다면, HelloController가 특정 클래스에 의존하지 않으므로 인터페이스를 구현한 클래스를 아무리 많이 만들어도 HelloController의 코드를 수정하지 않을 수 있음

이게 끝은 아님  
소스코드 레벨에서는 의존하고 있지 않더라도 실제 런타임 레벨에서는 어떤 오브젝트를 사용할지 결정되어있어야 함  
호출할 때 어느 클래스로 만든 오브젝트의 메소드를 이용하는 것인지 알수가 없음. 그래서 연관관계를 만들어줘야 함  
이게 바로 Dependency Injection

DI에는 어셈블러 Assembler라는 제 3의 존재가 필요함  
어떤 클래스의 오브젝트를 사용할지 결정했다면, 누군가가 이게 가능하게 만들어줘야 함  
오브젝트를 new 키워드로 만드는 대신에 외부에서 오브젝트를 만들어서 HelloController가 사용할 수 있도록 주입해주는게 어셈블러  
이 어셈블러를 우리는 스프링 컨테이너라고 부름

주입해주는 방식은 여러 개가 있음  
1. HelloController를 생성할 때, 생성자로 특정 클래스로 주입해주는 방식
2. Factory Method로 Bean을 만들도록 하면서 파라미터로 넘기는 방식
3. HelloController에 Property를 정의해서 Setter Method를 통해 사용해야될 클래스를 주입해주는 방식

이게 스프링 컨테이너를 사용해야 하는 가장 중요한 이유

## 의존 오브젝트 DI 적용

기존 코드는 HelloController가 SimpleHelloService의 오브젝트를 직접 생성해서 사용하는 방식  
이를 스프링 Bean으로 등록하고 스프링 컨테이너가 어셈블러로서 DI하는 작업까지 수행

인터페이스를 생성하고 인터페이스를 구현한 클래스에서 메소드를 정의하는 방식으로 수정  
이는 리팩토링 기능을 이용하면 편리 (우클릭 > Refactor > Extract Interface)

```java
public interface HelloService {
    String sayHello(String name);
}

public class SimpleHelloService implements HelloService {
    @Override
    public String sayHello(String name) {
        return "Hello, " + name + "!";
    }
}
```

HelloController 수정

```java
public class HelloController {
    private final HelloService service;

    public HelloController(HelloService service) {
        this.service = service;
    }

    public String hello(String name) {

        return service.sayHello(Objects.requireNonNull(name));
    }
}
```
- 오브젝트 생성 제거
  - DI 주입받을 변수 선언 및 생성자로 주입

Application.java 수정

```java
GenericApplicationContext context = new GenericApplicationContext();
context.registerBean(HelloController.class);
context.registerBean(SimpleHelloService.class);
context.refresh();
```
- SimpleHelloService Bean 등록
  - 스프링이 구성정보를 만들때는 정확이 어떤 클래스로 구성정보를 만들지 지정해줘야 하기 때문에 인터페이스(HelloService)는 불가능

HelloController의 생성자에서 HelloService 인터페이스가 필요하므로,  
DI가 진행될 때는 스프링 컨테이너가 Bean으로 등록된 클래스들 중 해당 인터페이스를 구현할 클래스를 찾아서 주입해줌

서버를 실행시키고 요청을 보내서 응답 확인

```bash
$ http -v GET ":8080/hello?name=Spring"

HTTP/1.1 200 
Connection: keep-alive
Content-Length: 15
Content-Type: text/plain;charset=ISO-8859-1
Date: Sun, 23 Jun 2024 09:06:29 GMT
Keep-Alive: timeout=60

Hello, Spring!
```

## DispatcherServlet으로 전환

## Annotation 매핑 정보 사용

## 스프링 컨테이너로 통합

## 자바코드 구성 정보 사용

## @Component 스캔

## Bean의 생명주기 메소드

## SpringBootApplication