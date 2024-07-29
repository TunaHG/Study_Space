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

Environment에서 프로퍼티를 직접 읽어오는 방법도 있지만 Spring에서는 변수에 프로퍼티값을 주입해주는 방법이 있음

```java
@MyAutoConfiguration
@ConditionalMyOnClass("org.apache.catalina.startup.Tomcat")
public class TomcatWebServerConfig {
    
    @Value("${contextPath}")
    String contextPath;
    
    @Bean("tomcatWebServerFactory")
    @ConditionalOnMissingBean
    public ServletWebServerFactory servletWebServerFactory() {
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();

        // 확인용
        System.out.println("contextPath: " + this.contextPath);
        
        factory.setContextPath(this.contextPath);
        return factory;
    }
}
```
- `@Value` 파라미터로는 치환자(placeholder)를 넣음

해당 필드가 선언된 Class의 Bean이 생성되며 프로퍼티가 주입됨

이대로 실행해보면 에러가 발생함  
그래서 어떤 값이 전달되고 있는지를 확인해보기 위해 찍어보면 `@Value`에 치환자로 넣은 값이 그대로 출력중
```shell
2024-07-29T20:12:58.549+09:00  INFO 88268 --- [toby] [           main] c.s.t.s.helloboot.TobyApplication        : ...
contextPath: ${contextPath}
2024-07-29T20:12:58.804+09:00  WARN 88268 --- [toby] [           main] ConfigServletWebServerApplicationContext : ...
```

`@Value`에 붙은 치환자를 교체해주는 것은 스프링 컨테이너의 기본 동작 방식은 아님  
프로퍼티로 교체해주는 후처리 작업을 하는 기능을 스프링 컨테이너에 추가해야 함  
AutoConfig로 동작하게 진행하기위해 새로운 클래스 하나 생성

```java
@MyAutoConfiguration
public class PropertyPlaceholderConfig {
    @Bean
    PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }
}
```
- PropertySourcesPlaceholderConfigurer는 소스코드를 타고 들어가보면 BeanFactoryPostProcessor를 구현한 구현체임

AutoConfig 클래스를 새로 만들었으면, .imports 파일에 추가적으로 명시해줘야 함
```imports
com.study.toby.section08.config.autoconfig.PropertyPlaceholderConfig
com.study.toby.section08.config.autoconfig.DispatcherServletConfig
com.study.toby.section08.config.autoconfig.TomcatWebServerConfig
com.study.toby.section08.config.autoconfig.JettyWebServerConfig
```

이후 다시 실행해보면 아까 찍어놓은 print 문으로 정상적으로 프로퍼티 값을 가져온 것을 확인할 수 있음
```shell
2024-07-29T20:15:09.154+09:00  INFO 88377 --- [toby] [           main] o.s.c.a.ConfigurationClassEnhancer       : ...
contextPath: /toby
2024-07-29T20:15:09.219+09:00  INFO 88377 --- [toby] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port 8080 (http)
```

## 프로퍼티 클래스의 분리

실제로는 Tomcat web server를 위해 설정할 프로퍼티가 정말 많음  
그리고 다른 클래스에서 동일한 프로퍼티를 사용해야하면 중복된 변수를 선언해줘야함  
그래서 프로퍼티 클래스로 따로 분리할 수 있음

분리하기 전에 Tomcat 에서 사용할 만한 port를 추가적으로 지정해봄
```java
@MyAutoConfiguration
@ConditionalMyOnClass("org.apache.catalina.startup.Tomcat")
public class TomcatWebServerConfig {

    @Value("${contextPath}")
    String contextPath;
    
    @Value("${port}")
    int port;

    @Bean("tomcatWebServerFactory")
    @ConditionalOnMissingBean
    public ServletWebServerFactory servletWebServerFactory() {
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
        factory.setContextPath(this.contextPath);
        factory.setPort(this.port);
        return factory;
    }
}
```

위와 같이 설정 후 실행하면 에러가 발생함  
왜 발생하는지를 살펴보면 port 프로퍼티를 `@Value`를 통해 가져오려고 했으나, port로 선언된 프로퍼티가 존재하지 않아서 에러가 발생
하지만 매번 모든 프로퍼티를 지정할 수는 없으니 대신 default 값을 지정해 줄 수 있음
```java
@Value("${port:9090}")
    int port;
```
- placeholder에 `:`로 추가적으로 명시하면 default 값을 명시할 수 있음

이제 프로퍼티 클래스를 생성해봄
```java
public class ServerProperties {
    private String contextPath;

    private int port;

    public String getContextPath() {
        return contextPath;
    }

    public void setContextPath(String contextPath) {
        this.contextPath = contextPath;
    }

    public int getPort() {
        return port;
    }

    public void setPort(int port) {
        this.port = port;
    }
}
```

프로퍼티 클래스를 Bean으로 만들기 때문에, 해당 클래스를 파라미터로 주입받아서 사용할 수 있음  
TomcatWebServerConfig에서 파라미터로 받아와서 사용
```java
@MyAutoConfiguration
@ConditionalMyOnClass("org.apache.catalina.startup.Tomcat")
public class TomcatWebServerConfig {
    @Bean("tomcatWebServerFactory")
    @ConditionalOnMissingBean
    public ServletWebServerFactory servletWebServerFactory(ServerProperties properties) {
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();

        factory.setContextPath(properties.getContextPath());
        factory.setPort(properties.getPort());

        return factory;
    }

    @Bean
    public ServerProperties serverProperties(Environment environment) {
        ServerProperties serverProperties = new ServerProperties();

        serverProperties.setContextPath(environment.getProperty("contextPath"));
        serverProperties.setPort(Integer.parseInt(environment.getProperty("port")));

        return serverProperties;
    }
}
```
- 현재는 프로퍼티 클래스의 Bean 팩토리 메소드에서 오브젝트를 생성할 때 Environment에서 가져옴

하지만 프로퍼티 클래스는 Tomcat에서만 사용되는 것이 아니므로, 자동 구성에 사용할 Config 클래스를 하나 더 생성
```java
// ServerPropertiesConfig.java
@MyAutoConfiguration
public class ServerPropertiesConfig {
    @Bean
    public ServerProperties serverProperties(Environment environment) {
        ServerProperties serverProperties = new ServerProperties();

        serverProperties.setContextPath(environment.getProperty("contextPath"));
        serverProperties.setPort(Integer.parseInt(environment.getProperty("port")));

        return serverProperties;
    }
}
```

Auto Config 클래스를 하나 더 추가했으므로, .imports 파일에도 추가해줘야 함
```imports
com.study.toby.section08.config.autoconfig.ServerPropertiesConfig
com.study.toby.section08.config.autoconfig.PropertyPlaceholderConfig
com.study.toby.section08.config.autoconfig.DispatcherServletConfig
com.study.toby.section08.config.autoconfig.TomcatWebServerConfig
com.study.toby.section08.config.autoconfig.JettyWebServerConfig
```

이대로 실행하면 에러가 발생함. 아까는 port의 default 값을 정해줬지만, 이번에는 정해주지 않았기 때문  
그래서 application.properties에 접근해서 port값을 명시해줌
```properties
contextPath=/toby
port=9090
```

앞으로 진행할 강의에서도 9090 port를 계속 사용할 예정이므로 테스트 코드도 모두 9090 port로 테스트하도록 수정

기존 ServerPropertiesConfig 코드를 살펴보면 Environment를 주입받아서 값을 일일이 추가했음  
이걸 더 편하게 하기 위해 Spring Boot가 제공하는 Binder를 사용
```java
@MyAutoConfiguration
public class ServerPropertiesConfig {
    @Bean
    public ServerProperties serverProperties(Environment environment) {
        return Binder.get(environment).bind("", ServerProperties.class).get();
    }
}
```
- 일일이 값을 꺼내오는 로직을 작성하지 않고 getter/setter로 되어있는 프로퍼티 이름과 일치하는 것을 찾아 자동으로 Binding해줌


## 프로퍼티 빈의 후처리기 도입
