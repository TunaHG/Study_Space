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

애플리케이션의 로직과 긴말한 연관이 있는게 서블릿 코드 안에 등장함  
- 매핑: 웹 요청을 가지고 처리해줄 컨트롤러를 찾는 작업
  - uri, method로 어떤 컨트롤러에게 전달할지 판단하는 부분
- 바인딩: DTO에 여러 파라미터를 넣어주는 작업

이러한 부분은 스프링을 이용해서 다른 전략으로 변경 -> DispatcherServlet

```java
// GenericApplicationContenxt context = new GenericApplicationContext();
GenericWebApplicationContext context = new GenericWebApplicationContext();

WebServer webServer = serverFactory.getWebServer(servletContext -> {
    servletContext.addServlet("dispatcherServlet", new DispatcherServlet(context))
        .addMapping("/*");
});
```
- applicationContext를 DispatcherServlet 생성자에 전달
  - 스프링 컨테이너와 커뮤니케이션하는 방법 (작업을 위임할 오브젝트를 찾아야 하는데 그 때 사용할 컨테이너 전달)
- 다만, 웹 요청을 처리하기 위해서는 GenericWebApplicationContext 타입을 사용해야 함

이제 다시 서버를 실행시키고 요청을 보내서 응답 확인

```bash
$ http -v GET ":8080/hello?name=Spring"

HTTP/1.1 404 
Connection: keep-alive
Content-Language: en
Content-Length: 732
Content-Type: text/html;charset=utf-8
Date: Mon, 24 Jun 2024 12:49:38 GMT
Keep-Alive: timeout=60
```

DispatcherServlet한테 어떤 웹 요청을 어떤 오브젝트에게 전달해야할지 정보를 전달하지 않았기에 404 발생  
정보를 전달하는 방법으로 여러 방법이 있으나, 가장 각광받은 방법은 매핑 정보를 요청을 처리할 컨트롤러 클래스 안에다가 집어넣는 방법

## Annotation 매핑 정보 사용

컨트롤러 클래스에 매핑 정보를 집어넣는 방법 (어노테이션 활용)

```java
@GetMapping("/hello")
public String hello(String name) {
    return service.sayHello(Objects.requireNonNull(name));
}
```
- HttpMethod GET으로 들어오는 것 중에서 URL이 /hello로 된 것을 해당 컨트롤러 메소드가 처리
  - 과거에는 `@RequestMapping(method = RequestMethod.GET, value = "/hello")`와 같이 사용
- `@RestController`는 DispatcherServlet 하고는 직접 관련이 없어서 현재는 사용하지 않아도 됨

앞서 애플리케이션 컨텍스트를 생성자로 받은 디스패처 서블릿(서블릿 컨테이너)는 Bean을 모두 탐색함  
그 중 매핑 정보를 가진 클래스를 찾아서 그 안의 요청 정보를 추출함  
그 후 매핑에 사용할 매핑 테이블을 만들고, 웹 요청이 들어오면 그걸 참고해서 담당할 Bean 오브젝트와 메소드를 확인

사실 이렇게만 작성하면 디스패처 서블릿이 메소드 레벨까지는 다 찾지 못함  
그래서 클래스 단에 `@RequestMapping`을 추가해야 함

```java
@RequestMapping("/toby")
public class HelloController {
    // ...
}
```

클래스 단에서 사용하는 `@RequestMapping`에도 URI을 추가할 수 있음. 여기 선언된 URI와 메소드단의 URI가 합쳐진게 최종 URI가 됨  
위 경우에는 `/toby/hello`가 최종 URI

여기까지 진행 후 다시 서버를 재가동하고 요청 보내서 응답 확인

```bash
$ http -v GET ":8080/toby/hello?name=Spring"

HTTP/1.1 404 
Connection: keep-alive
Content-Language: en
Content-Length: 732
Content-Type: text/html;charset=utf-8
Date: Mon, 24 Jun 2024 13:02:36 GMT
Keep-Alive: timeout=60

```

응답이 제대로 나오지 않음  
이건 현재 Controller에서 String으로 응답하고 있는데, 여기서 스프링의 기본동작과 연관 있음  
디스패처 서블릿은 String을 return하면 가장 기본이 되는 동작이 View라고 불리는 HTML 템플릿을 찾아서 응답하는 것  
그래서 해당 반환값 String이라는 이름의 View가 있는지 체크하는데 없으니 404 발생

String을 웹 응답의 Body에 넣어서 전달하게 하는 방식은 어노테이션을 하나 추가해야 함

```java
@GetMapping("/hello")
@ResponseBody
public String hello(String name) {
    return service.sayHello(Objects.requireNonNull(name));
}
```

추가한 다음 서버를 재가동하고 요청을 보내서 응답 확인

```bash
$ http -v GET ":8080/toby/hello?name=Spring"

HTTP/1.1 200 
Connection: keep-alive
Content-Length: 14
Content-Type: text/plain;charset=ISO-8859-1
Date: Mon, 24 Jun 2024 13:15:08 GMT
Keep-Alive: timeout=60

Hello, Spring!
```

초기에는 `@ResponseBody`를 사용하지 않아도 제대로 응답이 나왔었는데,  
그 이유는 클래스단에 `@RestController`이 선언되면 해당 클래스 내의 메소드들은 `@ResponseBody`가 없어도 붙어있다고 가정함

다만 여기서 발생하는 스프링 부트 버전 문제가 발생  
스프링 부트 2.7 버전에서는 위 코드대로 해도 문제가 없었으나, 3 버전으로 업데이트 되며 @Controller가 클래스단에 추가적으로 존재해야 정상 동작

```java
@Controller
@RequestMapping("/toby")
public class HelloController {
    // ...
}
```

## 스프링 컨테이너로 통합

서블릿 컨테이너를 만들고 서블릿을 초기화하는 등의 작업을 스프링 컨테이너가 초기화되는 과정중에 일어나도록 변경  
스프링 부트가 그렇게 동작하고 있음

스프링 컨테이너의 초기화 작업은 refresh()에서 진행  
해당 메소드 내부 구현 코드를 살펴보면 템플릿 메소드로 만들어져 있다는 것을 알 수 있음

템플릿 메소드 패턴을 사용하면 그 안에 여러 개의 Hook 메소드를 주입해 넣기도 함  
템플릿 메소드 안에서 일정한 순서에 의해서 작업들이 호출되는데 그 중 서브 클래스에서 확장하는 방법을 통해  
특정 시점에 어떤 작업을 수행하게 해서 기능을 유연하게 확장하도록 만드는 기법  
그 Hook 메소드 이름이 onRefresh()

템플릿 메소드 패턴은 상속을 통해서 기능을 확장하도록 만드니까 GenericWebApplicationContext를 상속하는 클래스를 생성  
다만, 클래스를 따로 정의하기는 번거로우니 간단하게 익명 클래스를 사용

```java
GenericWebApplicationContext context = new GenericWebApplicationContext() {
    @Override
    protected void onRefresh() {
        super.onRefresh();

        TomcatServletWebServerFactory serverFactory = new TomcatServletWebServerFactory();
        WebServer webServer = serverFactory.getWebServer(servletContext -> {
            servletContext.addServlet("dispatcherServlet", new DispatcherServlet(this))
                .addMapping("/*");
        });
        webServer.start();
    }
};
```
- 부모 클래스의 `onRefresh()`를 호출하는 것을 빼먹으면 안됨
  - 부모 클래스에서도 해당 메소드를 확장해서 추가작업을 진행하기 때문
- `onRefresh()` 안으로 서블릿 초기화하는 작업 코드 이동
  - 다만 context 자체가 변수가 아니라 메소드를 선언하는 해당 클래스이므로 `this` 사용

변경한 다음 서버를 재가동하고 요청을 보내서 응답 확인

```bash
$ http -v GET ":8080/toby/hello?name=Spring"

HTTP/1.1 200 
Connection: keep-alive
Content-Length: 14
Content-Type: text/plain;charset=ISO-8859-1
Date: Mon, 24 Jun 2024 13:29:07 GMT
Keep-Alive: timeout=60

Hello, Spring!
```

## 자바코드 구성 정보 사용

스프링 컨테이너가 사용하는 구성 정보를 어떻게 오브젝트로 만들어서 컨테이너 내에 컴포넌트로 등록해두고  
스프링 컨테이너 안에 Bean 오브젝트가 다른 오브젝트에 의존하고 있다면 관계를 어떻게 맺어줄 것인가, 어느 시점에 오브젝트를 주입할 것인가 등등  
이런 정보들을 스프링 컨테이너에 구성 정보로 제공해줘야 함

과거에는 외부 설정파일을 사용했었음. 이번에는 Factory Method 이용

일반적으로는 이런 방식을 사용하지 않음. 필요할 때만 사용  
Bean 오브젝트를 만들고 초기화하는 작업이 복잡할 때 복잡한 설정 정보로 나열하는 대신에 자바 코드로 만들면 간결하고 이해하기 쉬움

```java
@Configuration
public class TobyApplication {
    @Bean
    public HelloController helloController(HelloService service) {
        return new HelloController(service);
    }
    
    @Bean
    public HelloService helloService() {
        return new SimpleHelloService();
    }

    public static void main(String[] args) {
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext() {
            @Override
            protected void onRefresh() {
                super.onRefresh();

                TomcatServletWebServerFactory serverFactory = new TomcatServletWebServerFactory();
                WebServer webServer = serverFactory.getWebServer(servletContext -> {
                    servletContext.addServlet("dispatcherServlet", new DispatcherServlet(this))
                        .addMapping("/*");
                });
                webServer.start();
            }
        };
        context.register(TobyApplication.class);
        context.refresh();
    }
}
```
- HelloController를 생성할 때는 HelloService가 필요함
  - 스프링 컨테이너한테 의존 오브젝트를 파라미터로 넘겨달라고 선언
- HelloService 타입으로 return
  - SimpleHelloService 타입을 return 하지 않고 Bean을 주입받을때 어떤 타입을 기대하는가를 생각하고 결정
- @Bean
  - 스프링 컨테이너가 Bean 오브젝트로 사용되는 그 오브젝트를 생성하는 Factory method임을 전달하는 어노테이션
- @Configuration
  - 스프링 컨테이너가 제대로 인지할 수 있도록 클래스 단에 선언하는 어노테이션
- AnnotationConfigWebApplicationContext 로 스프링 컨테이너 변경
  - GenericWebApplicationContext는 자바 코드로 만든 Configuration 정보를 읽을 수 없음
  - registerBean()을 더 이상 지원하지 않음
  - register()로 자바 코드로 된 구성 정보를 가지고 있는 클래스를 등록해줘야함

변경후 정상적으로 동작하는지 서버를 재가동하고 요청을 보내서 응답을 확인

```bash
$ http -v GET ":8080/toby/hello?name=Spring"

HTTP/1.1 200 
Connection: keep-alive
Content-Length: 14
Content-Type: text/plain;charset=ISO-8859-1
Date: Mon, 24 Jun 2024 13:42:14 GMT
Keep-Alive: timeout=60

Hello, Spring!
```

## @Component 스캔

## Bean의 생명주기 메소드

## SpringBootApplication