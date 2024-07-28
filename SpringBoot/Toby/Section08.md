# Section 08. 외부 설정을 이용한 자동 구성

## Environment 추상화와 프로퍼티

외부설정은 자동 구성중에 어떤 타이밍에 어떤 방식으로 적용되는가를 살펴봄

그 전에 7강에서 학습한 @Conditional을 활용해서 자동 구성 정보가 어떻게 등록되는지 살펴봄  
1. .imports 파일로부터 자동 구성 후보를 로딩함
2. Starter, Dependecy, Classpath Class로부터 가져온 Class들 중 @Conditional으로 자동 구성 정보를 등록할 클래스를 판단
3. @Conditional으로 자동 구성 정보를 등록할 @Bean의 조건을 체크
   - 개발자가 동일한 타입의 Bean을 유저 구성 정보로 만들어 놓은게 있는지 확인
      - 만들어 놓은게 있다면 해당 Bean을 사용하며 자동 구성 정보에 들어있는 Bean은 무시됨
      - 만들어 놓은게 없다면 미리 준비 되어있는 AutoConfiguration의 Bean을 사용

여기서 하나가 더 추가되는데, 외부 설정 정보를 이용해서 생성된 Bean의 프로퍼티 값을 수정하는 것  
자동구성에 의해서 만들어지는 Bean은 대부분 다 default 값이 들어가 있음 (Tomcat의 port는 8080이 기본인 것처럼)  
필요한 경우 그것을 바꾸고 싶은데, 커스텀 Bean으로 해결하기엔 번거로운 일임  
그래서 Spring Boot는 자동 구성 Configuration 클래스에 다양한 프로퍼티를 변경할수 있는 방법을 제공함  

Spring 프레임워크가 제공하는 Environment Abstraction  
Spring 프레임워크가 제공하는 것으로는  
StanardEnvironment에 해당하는 System Properties, System Environment Variables  
StanardServletEnvironment에 해당하는 ServletConfig Parameters, ServletContext Parameters, JNDI  
코드 내에서 지정하는 @PropertySource  
Spring Boot에서 제공하는 application.properties, xml, yml 등이 있음

위와 같은 방법들로 프로퍼티값을 등록하고 가져올 수 있음

## 자동 구성에 Envirionment 프로퍼티 적용

Spring Initialzr를 사용하면 `resources/application.properties`가 생성됨  

스프링 부트 애플리케이션이 시작되면(스프링 컨테이너가 초기화되고 Bean들이 다 만들어지면) 특정 기능이 자동으로 수행되게 만들어봄  
스프링 컨테이너에 만들어진 Bean을 사용할 수 있어야 함  
우리가 확인하려는 Environment도 우리가 직접 만들어가 자동 구성에 의해 만들어 지는건 아니고 스프링 컨테이너가 직접 생성해서 Bean으로 등록하는 것

ApplicationRunner라는 인터페이스를 구현한 Bean을 등록하면 스프링 부트 초기화 작업이 끝난 이후 해당 오브젝트를 run 메소드를 통해 실행  
이 때, 애플리케이션을 실행할 때 인자값으로 받은 정보도 함께 받을 수 있음  
ApplicationRunner 구현체를 Bean으로 등록하기 때문에 그 자체로 DI를 받을 수 있는 것

우선 지난 7강에 만든 custom tomcat webserver config 파일을 제거 (helloboot 패키지 내 WebServerConfiguration.java로 만듬)  
그리고 `@EnableMyAutoConfiguration`은 TobyApplication에 붙어있었는데, MySpringBootApplication에 붙이도록 수정했음

TobyApplication에 Bean 메소드 생성
```java
@MySpringBootApplication
public class TobyApplication {

    @Bean
    ApplicationRunner applicationRunner(Environment env) {
        return args -> {
            String name = env.getProperty("my.name");
            System.out.println("my.name: " + name);
        };
    }

    public static void main(String[] args) {
        SpringApplication.run(TobyApplication.class, args);
    }

}
```
- ApplicationRunner 구현체를 return하도록 설정하고 익명클래스로 구현
- Environment를 파라미터로 지정해서 Bean을 생성할 때 DI 받도록 지정

위와 같이 설정해놓고 서버를 실행하면 현재는 해당 프로퍼티를 설정해 놓지 않았으니 null이 출력됨
```shell
2024-07-28T15:11:39.583+09:00  INFO 69291 --- [toby] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/'
2024-07-28T15:11:39.584+09:00  INFO 69291 --- [toby] [           main] c.s.t.s.helloboot.TobyApplication        : Started TobyApplication in 0.352 seconds (process running for 0.564)
my.name: null
```

앞서 설명한 프로퍼티 등록 방법중 application.properties를 통해 등록하고 가져오도록 설정해봄
```properties
my.name=ApplicationProperties
```

위와 같이 .properties를 추가하고 다시 서버를 실행해보면 입력해놓은 값이 정상적으로 출력되는 것을 확인할 수 있음
```shell
2024-07-28T15:12:07.972+09:00  INFO 69305 --- [toby] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/'
2024-07-28T15:12:07.973+09:00  INFO 69305 --- [toby] [           main] c.s.t.s.helloboot.TobyApplication        : Started TobyApplication in 0.398 seconds (process running for 0.585)
my.name: ApplicationProperties
```

프로퍼티 등록 방법중 우선순위가 있는데, `application.properties`보다 우선하는 것은 환경변수임  
IntelliJ에서는 서버를 실행하는 부분에 edit configurations 팝업에서 Environment variables로 설정 가능  
보통 환경변수명은 관례적으로 대문자를 사용하며 .보다는 _를 사용함  
그래서 `MY_NAME=EnvironmentVariable` 이라는 값을 Environment variables 칸에 입력  
> 혹시 팝업에서 Environment variables 입력칸이 보이지 않는다면, Build and run 구분 문구 우측 Modify options를 눌러서 해당 칸을 활성화해야 함

.properties와 환경변수를 설정해놓고 어떤 값이 출력되는지 직접 확인해봄  
```shell
2024-07-28T15:13:39.498+09:00  INFO 69333 --- [toby] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/'
2024-07-28T15:13:39.499+09:00  INFO 69333 --- [toby] [           main] c.s.t.s.helloboot.TobyApplication        : Started TobyApplication in 0.324 seconds (process running for 0.511)
my.name: EnvironmentVariable
```

이 환경변수보다 더 높은 우선순위를 가지는 것은 System Property인데,  
System Property 중에서 우리가 명시적으로 선언할 수 있는 것은 Java 명령으로 프로그램을 실행할 때 -D 옵션을 주고 프로퍼티 이름과 밸류를 주는 방법
해당 값을 주는 방법은 앞서 edit configurations 팝업에서 modify options에서 add VM options을 활성화하고 `-Dmy.name=SystemProperty` 입력

이제 서버를 또 다시 실행해서 SystemProperty가 출력되는지 확인
```shell
2024-07-28T15:28:04.722+09:00  INFO 69511 --- [toby] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/'
2024-07-28T15:28:04.723+09:00  INFO 69511 --- [toby] [           main] c.s.t.s.helloboot.TobyApplication        : Started TobyApplication in 0.331 seconds (process running for 0.56)
my.name: SystemProperty
```

기본적으로는 `application.properties`에 입력해놓고 오버라이딩하고싶으면 환경변수를 사용하는게 일반적인 사용방법

다음으로, 자동 구성의 기본 설정 값을 변경해봄
TomcatWebServerConfig를 살펴보면, Factory의 오브젝트를 굳이 만들었다는 것은 실제 Tomcat 서버를 만들어서 서블릿 컨테이너를 띄우기 전에 다양한 설정을 손쉽게 할 수 있는 방법을 제공하기 위해서임  
오브젝트를 변수로 받아서 살펴보면, 여러 setter 메소드를 가지고 있어 Tomcat을 띄우는데 필요한 여러 작업들을 지정할 수 있음  
여기서는 ContextPath를 지정해봄
```java
@MyAutoConfiguration
@ConditionalMyOnClass("org.apache.catalina.startup.Tomcat")
public class TomcatWebServerConfig {
    @Bean("tomcatWebServerFactory")
    @ConditionalOnMissingBean
    public ServletWebServerFactory servletWebServerFactory() {
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
        factory.setContextPath("/app");
        return factory;
    }
}
```

위 코드로 실행해보면 아래와 같은 로그로 `/app` Context Path로 서버가 뜬것을 확인할 수 있음
```shell
2024-07-28T15:29:26.785+09:00  INFO 69574 --- [toby] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/app'
2024-07-28T15:29:26.786+09:00  INFO 69574 --- [toby] [           main] c.s.t.s.helloboot.TobyApplication        : Started TobyApplication in 0.317 seconds (process running for 0.512)
```

이렇게 띄운 API 서버로 요청을 직접 보내서 확인해봄
```shell
$ http -v GET ":8080/app/hello?name=Spring"

GET /app/hello?name=Spring HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Connection: keep-alive
Host: localhost:8080
User-Agent: HTTPie/3.2.2



HTTP/1.1 200 
Connection: keep-alive
Content-Length: 16
Content-Type: text/plain;charset=ISO-8859-1
Date: Sun, 28 Jul 2024 06:29:54 GMT
Keep-Alive: timeout=60

*Hello, Spring!*
```

이제 /app이 아니라 ContextPath를 Environment를 통해서 프로퍼티값을 읽어서 지정할 수 있게 사용해봄
```java
@MyAutoConfiguration
@ConditionalMyOnClass("org.apache.catalina.startup.Tomcat")
public class TomcatWebServerConfig {
    @Bean("tomcatWebServerFactory")
    @ConditionalOnMissingBean
    public ServletWebServerFactory servletWebServerFactory(Environment env) {
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
      //   factory.setContextPath("/app");
        factory.setContextPath(env.getProperty("contextPath"));
        return factory;
    }
}
```

프로퍼티값을 지정하기 위해 `application.properties`를 지정해봄
```properties
contextPath=/toby
```

서버를 실행해서 `/toby` Context Path로 실행되는지 확인
```shell
2024-07-28T15:31:17.026+09:00  INFO 69724 --- [toby] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/toby'
2024-07-28T15:31:17.027+09:00  INFO 69724 --- [toby] [           main] c.s.t.s.helloboot.TobyApplication        : Started TobyApplication in 0.338 seconds (process running for 0.531)
```

이렇게 진행하면 테스트들이 실패할 것이므로 테스트 코드들에도 경로를 추가해줘야함

간단한 값 하나를 설정하는 것은 학습한 내용처럼 진행해보면 됨  
하지만 실제로 자동 구성에 의해 만들어지는 Bean들은 속성값이 굉장히 많음  
그래서 이것을 조금 더 효율적으로 다룰 수 있는 방식들을 Spring Boot가 제공하고 있어 이후에 그것을 학습해봄

## @Value와 PropertySourcesPlaceholderConfigurer

## 프로퍼티 클래스의 분리

## 프로퍼티 빈의 후처리기 도입
