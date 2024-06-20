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

## Dependency Injection

## 의존 오브젝트 DI 적용

## DispatcherServlet으로 전환

## Annotation 매핑 정보 사용

## 스프링 컨테이너로 통합

## 자바코드 구성 정보 사용

## @Component 스캔

## Bean의 생명주기 메소드

## SpringBootApplication