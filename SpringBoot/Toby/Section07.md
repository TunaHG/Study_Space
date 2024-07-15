# Section 07. 조건부 자동 구성

## 스타터와 Jetty 서버 구성 추가

SpringBoot에 내장된 @AutoConfiguration  
spring-boot-autoconfigure 라이브러리를 살펴보면 해당 클래스 및 파일들이 존재함  
해당 라이브러리 파일을 살펴보면 META-INF/spring 경로에 .imports 파일이 있고, SpringBoot에서 설정한 자동구성정보가 있음  
스타터로 생성된 144개의 AutoConfiguration이 설정되어 있음

144개의 클래스들 및 스프링에서 기본적으로 설정되어있는 Bean들이 모두 서버가 실행될 때마다 등록되지는 않음  
팩토리 메소드에 어떤 조건을 걸어서 적용할까 말까를 결정하는 프로세스를 거치게 되어 있음

자바에는 톰캣이 표준이 아님.  
톰캣은 자바의 서블릿 컨테이너 기술을 구현한 라이브러리 중에 하나. 톰캣말고 다양한 서블릿 컨테이너 기술이 있음

이번 section엔 조건을 걸어서, 조건에 따라 톰캣 혹은 다른 서블릿 컨테이너(여기서는 Jetty) 중 어떤 것을 사용할지 결정하도록 구현

이전에 선언한 TomcatServletWebServerFactory 클래스는 tomcat-embedded-core 라이브러리에 존재   
해당 라이브러리는 build.gradle에 선언한 dependencies의 spring-boot-starter-web으로부터 다운로드받음
해당 dependency로 부터 어떤 라이브러리들을 다운로드받았는지는 IDEA에서 확인할 수도 있지만 터미널로도 확인 가능  

```shell
$ ./gradlew dependencies --configuration compilClasspath
```

spring-boot-starter-jetty dependency를 추가
```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-jetty'
}
```
dependency가 로딩되었는지 확인하는 방법은 터미널에 나가서 확인해보거나, IDEA의 gradle 설정에서 의존 라이브러리들을 확인해볼 수도 있음

기존에 만들었던 TomcatWebServerConfig를 복사하여 JettyWebServerConfig를 생성해봄  
생성하는 목적은 위에서 말했다싶이 어떤 조건을 부여해서 해당 조건을 만족하면 Jetty 서버를 띄우고, 다른 조건을 만족하면 Tomcat 서버를 띄우도록 설정하기 위함
```java
// JettyWebServerConfig.java
@MyAutoConfiguration
public class JettyWebServerConfig {
    @Bean("jettyWebServerFactory")
    public ServletWebServerFactory servletWebServerFactory() {
        return new JettyServletWebServerFactory();
    }
}
```
- TomcatWebServerConfig와 동일하나, Bean 팩토리 메소드의 응답값을 Tomcat이 아닌 Jetty로 변경
- Bean의 이름은 기본적으로 팩토리 메소드의 이름으로 정해지나, @Bean에 선언해줄 수 있음
  - TomcatWebServerConfig의 Bean 팩토리 메소드도 이름을 지정 (`tomcatWebServerFactory`)

JettyWebServerConfig를 생성했으니 기존에 생성한 .imports 파일에 해당 Config 파일의 경로를 추가
```
# /resources/META-INF/spring/com.study.toby.config.MyAutoConfiguration.imports
com.study.toby.config.autoconfig.DispatcherServletConfig
com.study.toby.config.autoconfig.TomcatWebServerConfig
com.study.toby.config.autoconfig.JettyWebServerConfig
```

현재 상태로 서버를 실행시켜 보면, ServletWebServerFactory Bean이 Tomcat, Jetty로 두개가 선언되어 있으니  
둘 중 어떤 Bean을 사용할지 스프링부트가 판단하지 못하기 때문에 에러가 발생하고 서버 실행이 실패함

```shell
2024-07-15T16:31:12.452+09:00 ERROR 74543 --- [toby] [           main] o.s.boot.SpringApplication               : Application run failed

org.springframework.context.ApplicationContextException: Unable to start web server
	...
Caused by: org.springframework.context.ApplicationContextException: Unable to start ServletWebServerApplicationContext due to multiple ServletWebServerFactory beans : tomcatWebServerFactory,jettyWebServerFactory
	...
```

## @Conditional과 Condition



## @Conditional 학습테스트

## 커스텀 @Conditional

## 자동 구성 정보 대체하기

## 스프링 부트의 @Conditional
