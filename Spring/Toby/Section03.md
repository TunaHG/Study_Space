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

## 프론트 컨트롤러

## 프론트 컨트롤러로 전환

## Hello 컨트롤러 매핑과 바인딩