# Section 03. 독립 실행형 서블릿 애플리케이션

## Containerless 개발 준비

SpringBoot 프로젝트를 처음 생성할 때 자동으로 생성해주는 것들만 있어도 서버가 실행됨

* `@SpringBootApplication`
* `SpringApplication.run(TobyApplication.class, args);`

위 두개를 지우고 직접 구현하며 진행

## 서블릿 컨테이너 띄우기

서블릿 컨테이너를 설치하지 않고 standalone 프로그램을 만들어서, 거기서 서블릿 컨테이너를 띄우는 작업을 진행할 예정

빈 서블릿 컨테이너를 코드로 띄우는 작업 먼저 진행  
서블릿 컨테이너의 대명사라고 많이 알고 사용하는 Tomcat으로 사용  
내장형으로 임베디드 톰캣을 사용할 예정이며, SpringBoot 프로젝트를 만들때 알아서 설치되어 있음

```java
ServletWebServerFactory serverFactory = new TomcatServletWebServerFactory();
WebServer webServer = serverFactory.getWebServer();
webServer.start();
```

SpringBoot가 Tomcat Servlet 컨테이너를 내장해서 프로그램에서 코드로 쉽게 시작해서 사용할 수 있도록 만든 도우미 클래스인  
TomcatServletWebServerFactory 활용

ServletWebServerFactory나 WebServer와 같은 interface는 Tomcat, Jetty 등의 여러 개중 원하는 툴을 선택해서 사용할 수 있도록 추상화한 인터페이스

WebServer.start()로 실제 서버 실행

서버 실행시키고 요청해보면 404 를 응답하여 서버가 요청을 받고있다는 사실을 확인할 수 있음

```bash
$ http -v :8080/hello

HTTP/1.1 404 
Connection: keep-alive
Content-Language: en
Content-Length: 683
Content-Type: text/html;charset=utf-8
Date: Fri, 14 Jun 2024 08:37:21 GMT
Keep-Alive: timeout=60
```

## 서블릿 등록

서블릿 컨테이너 안에 들어가는 웹 컴포넌트를 서블릿라고 함  
서블릿 컨테이너가 클라이언트로부터 웹 요청을 받으면 여러 개의 서블릿중에서 요청을 맡길 서블릿을 결정하는 것을 매핑이라고 함

서블릿을 서블릿 컨테이너에 추가해야함.

getWebServer()에서 ServletContextInitializer를 파라미터로 받음. 익명 클래스로 전달  
onStartUp() 메소드만 오버라이딩하면 되므로 람다표현식으로 전환

```java
WebServer webServer = serverFactory.getWebServer(new ServletContextInitializer() {
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        
    }
});

// 위 코드를 람다표현식으로 전환
WebServer webServer = serverFactory.getWebServer(servletContext -> {

});
```

Servlet이 필요한 부분은 공통적인 코드를 미리 구현해놓고 상속해서 사용할 수 있도록 만들어 놓은 어댑터 클래스인 HttpServlet 사용.  
필요한 부분만 사용하면 되므로 익명 클래스로 사용

```java
WebServer webServer = serverFactory.getWebServer(servletContext -> {
    servletContext.addServlet("hello", new HttpServlet() {
        @Override
        protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            super.service(req, resp);
        }
    });
});
```

`addServlet()`만 진행해서 끝나는게 아니라 매핑을 추가해줘야 함 (`addMapping()`)  

```java
WebServer webServer = serverFactory.getWebServer(servletContext -> {
    servletContext.addServlet("hello", new HttpServlet() {
        @Override
        protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            super.service(req, resp);
        }
    }).addMapping("/hello");
});
```

웹 응답은 HttpServlet 익명 클래스 내의 `service()`에서 작성

```java
WebServer webServer = serverFactory.getWebServer(servletContext -> {
    servletContext.addServlet("hello", new HttpServlet() {
        @Override
        protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            resp.setStatus(200);
            resp.setHeader("Content-Type", "text/plain");
            resp.getWriter().println("Hello Servlet");
        }
    }).addMapping("/hello");
});
```

서버 실행시키고 요청해보면 확인 가능

```bash
$ http -v :8080/hello

HTTP/1.1 200 
Connection: keep-alive
Content-Length: 14
Content-Type: text/plain;charset=ISO-8859-1
Date: Fri, 14 Jun 2024 08:42:31 GMT
Keep-Alive: timeout=60

Hello Servlet
```

## 서블릿 요청 처리

String을 하드코딩해서 넣는건 오타의 위험성이 있어서 지양  
`200`의 경우 `HttpStatus.OK.value()`로 사용  
`Content-Type`의 경우 `HttpHeaders.CONTENT_TYPE` 상수가 존재  
`text/plain`의 경우 `MediaType.TEXT_PLAIN_VALUE` 상수가 존재

요청 중 URL 부분은 어떤 서블릿을 매핑하는지의 `addMapping()`에서 사용되었음

파라미터로 넘어오는 것은 `HttpServletRequest.getParameter()`를 사용하면 받을 수 있음

```java
WebServer webServer = serverFactory.getWebServer(servletContext -> {
    servletContext.addServlet("hello", new HttpServlet() {
        @Override
        protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            String name = req.getParameter("name");

            resp.setStatus(HttpStatus.OK.value());
            resp.setHeader(HttpHeaders.CONTENT_TYPE, MediaType.TEXT_PLAIN_VALUE);
            resp.getWriter().println("Hello " + name);
        }
    }).addMapping("/hello");
});
```

서버를 실행시키고 요청을 보내서 확인

```bash
$ http -v ":8080/hello?name=Spring"

HTTP/1.1 200 
Connection: keep-alive
Content-Length: 13
Content-Type: text/plain;charset=ISO-8859-1
Date: Sun, 16 Jun 2024 02:21:45 GMT
Keep-Alive: timeout=60

Hello Spring
```

위와 같이 파라미터가 존재할땐 `""`로 묶어줘야 제대로 동작

## 프론트 컨트롤러

서블릿은 요청마다 직접 하나씩 매핑을 해야 했음

서블릿이 개수가 늘어나다보니 공통적인 작업이 서블릿 코드 안에서 중복 등장  
서블릿은 request, response를 직접 다뤄줘야 했음

기본적인 서블릿만 가지고 기능을 개발하기는 한계가 있었음

공통적인 부분을 제일 앞단과 제일 뒷단에서 처리하게 하는 프론트 컨트롤러의 등장  
인증이나 보안, 다국어처리 등의 작업을 프론트 컨트롤러가 처리하는 방식이 일반화됨

## 프론트 컨트롤러로 전환

프론트 컨트롤러로 전환하려면 매핑 부분을 변경  
중앙화된 처리를 위해 모든 요청을 다 받아야함 (`/*`)

공통 기능을 처리하고 나서 서블릿 컨테이너의 매핑기능을 프론트 컨트롤러가 담당해야함

제어문으로 getRequestURI()만 확인하는게 아니라, getMethod()도 확인해야 함

```java
WebServer webServer = serverFactory.getWebServer(servletContext -> {
    servletContext.addServlet("hello", new HttpServlet() {
        @Override
        protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            if (req.getRequestURI().equals("/hello") && req.getMethod().equals(HttpMethod.GET.name())) {
                String name = req.getParameter("name");

                resp.setStatus(HttpStatus.OK.value());
                resp.setHeader(HttpHeaders.CONTENT_TYPE, MediaType.TEXT_PLAIN_VALUE);
                resp.getWriter().println("Hello " + name);
            } else if (req.getRequestURI().equals("/user")) {
                //
            } else {
                resp.setStatus(HttpStatus.NOT_FOUND.value());
            }
        }
    }).addMapping("/*");
});
```

서버를 실행시키고 기존과 동일하게 요청을 보내서 응답을 잘 받아오는지 확인

```bash
$ http -v ":8080/hello?name=Spring"

HTTP/1.1 200 
Connection: keep-alive
Content-Length: 13
Content-Type: text/plain;charset=ISO-8859-1
Date: Sun, 16 Jun 2024 02:35:22 GMT
Keep-Alive: timeout=60

Hello Spring
```

http 요청에서 아무것도 주지않아서 GET 요청  
POST 옵션을 줘서 제어문으로 선언하지 않은 요청일 경우 404 응답하는 것 확인

```bash
$ http -v POST ":8080/hello?name=Spring"

HTTP/1.1 404 
Connection: keep-alive
Content-Length: 0
Date: Sun, 16 Jun 2024 02:35:27 GMT
Keep-Alive: timeout=60
```

## Hello 컨트롤러 매핑과 바인딩

현재는 로직이 프론트 컨트롤러 내에 들어가있음. 이 로직을 분리해야함  
이전 Section에서 사용했던 HelloController를 일부 수정해서 활용

```java
public class HelloController {
    public String hello(String name) {
        return "Hello " + name;
    }
}
```

HelloController가 String name을 받아야 하니 요청에서 파라미터를 꺼내는 부분은 프론트 컨트롤러가 담당  
HelloController의 결과값을 프론트 컨트롤러의 getWriter()에 전달하여 웹 응답을 생성

```java
HelloController helloController = new HelloController();

WebServer webServer = serverFactory.getWebServer(servletContext -> {
    servletContext.addServlet("hello", new HttpServlet() {
        @Override
        protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            if (req.getRequestURI().equals("/hello") && req.getMethod().equals(HttpMethod.GET.name())) {
                String name = req.getParameter("name");

                String ret = helloController.hello(name);

                resp.setStatus(HttpStatus.OK.value());
                resp.setHeader(HttpHeaders.CONTENT_TYPE, MediaType.TEXT_PLAIN_VALUE);
                resp.getWriter().println(ret);
            } else if (req.getRequestURI().equals("/user")) {
                //
            } else {
                resp.setStatus(HttpStatus.NOT_FOUND.value());
            }
        }
    }).addMapping("/*");
});
```

위처럼 수정하고 의도한바 대로 동작하는지 서버 실행하고 요청보내서 확인  
GET 요청은 생략할 수 있지만 명시적으로 선언해도 무방함

```bash
$ http -v GET ":8080/hello?name=Spring"

HTTP/1.1 200 
Connection: keep-alive
Content-Length: 13
Content-Type: text/plain;charset=ISO-8859-1
Date: Sun, 16 Jun 2024 02:47:42 GMT
Keep-Alive: timeout=60

Hello Spring
```

실제 요청을 받아서 처리하는 코드를 동작시키는 동안에 두 가지 중요한 작업이 수행되는데 **매핑과 바인딩**

매핑이란?  
웹 요청에 들어있는 정보를 활용해서 어떤 로직을 수행하는 코드를 호출할 것인가 결정하는 작업

바인딩이란?  
Controller를 만든다고 하더라도 직접적으로 웹 요청과 응답을 다루는 오브젝트를 일반적으로 사용하지 않음  
대신 평범한 Java 타입으로 웹 요청 정보를 변환해서 받기를 원함  
파라미터를 프론트 컨트롤러가 꺼내서 HelloController로 String name만 전달해준 작업을 바인딩이라고 함