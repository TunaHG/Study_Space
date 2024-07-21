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

Bean으로 등록할 Configuration의 후보를 지정해놨으므로, 조건을 달아서 그 중 어떤 구성 정보를 활용할지 결정을 내리는 매커니즘을 만들어봄  
Tomcat, Jetty 중 어떤 것을 선택하는지는 나중에 확인.

우선 Import Selector가 로딩하는 모든 Configuration 후보 중에서 어떤 것을 Bean으로 등록할지 말지를 선택하게 만드는 방법 추가  
```java
@MyAutoConfiguration
@Conditional(JettyWebServerConfig.JettyCondition.class)
public class JettyWebServerConfig {
    @Bean("jettyWebServerFactory")
    public ServletWebServerFactory servletWebServerFactory() {
        return new JettyServletWebServerFactory();
    }

    static class JettyCondition implements Condition {
        @Override
        public boolean matches(ConditionContext context,
                               AnnotatedTypeMetadata metadata) {
            return true;
        }
    }
}
```
- Class 레벨에 @Conditional 선언
  - @Conditional은 element를 꼭 하나 등록해야 하는데, 그 타입은 Condition이라는 인터페이스를 구현한 클래스여야 함
- Condition이라는 인터페이스를 구현한 JettyCondition이라는 Class를 inner class로 선언
  - 스프링 컨테이너에게 조건에 해당하는지를 알려주는 matches() 오버라이딩
    - ConditionContext는 현재 스프링 컨테이너와 애플리케이션이 돌아가고 있는 환경에 대한 정보를 얻을 수 있는 오브젝트
    - AnnotatedTypeMetaData는 @Conditional이 선언된 클래스와 그 클래스가 사용하는 다른 어노테이션들의 정보를 얻을 수 있는 오브젝트

JettyWebServerConfig를 수정했으니, TomcatWebServerConfig도 수정
```java
@MyAutoConfiguration
@Conditional(TomcatWebServerConfig.TomcatCondition.class)
public class TomcatWebServerConfig {
    @Bean("tomcatWebServerFactory")
    public ServletWebServerFactory servletWebServerFactory() {
        return new TomcatServletWebServerFactory();
    }

    static class TomcatCondition implements Condition {
        @Override
        public boolean matches(ConditionContext context,
                               AnnotatedTypeMetadata metadata) {
            return false;
        }
    }
}
```

우선 위처럼 JettyCondition.matches()의 return 값을 true, TomcatCondition.matches()의 return 값을 false로 설정하고 서버 실행

```shell
...
2024-07-18T17:56:27.086+09:00  INFO 96894 --- [toby] [           main] o.s.b.web.embedded.jetty.JettyWebServer  : Jetty started on port 8080 (http/1.1) with context path '/'
2024-07-18T17:56:27.087+09:00  INFO 96894 --- [toby] [           main] c.s.t.s.helloboot.TobyApplication        : Started TobyApplication in 0.429 seconds (process running for 0.624)
```
- 정상적으로 서버가 실행됬음을 확인
- 로그를 보면 Tomcat이 아닌 Jetty 서버가 실행되었음을 확인할 수 있음
  - 반대로 Jetty를 false, Tomcat을 true로 수정하고 Tomcat 서버가 동작하는지 확인

@Conditional은 Class 레벨과 Method 레벨에 모두 선언할 수 있음  
하지만 @Conditional을 선언한 Class 내의 Method에 @Conditional을 또 선언하면,  
Class 레벨의 @Conditional을 먼저 체크하고 그 다음 Method 레벨의 @Condtional을 체크함  
(Class 레벨의 @Conditional이 false라면 Method 레벨은 확인하지도 않음)

이제 조건을 부여하는 방법을 학습하기 전에, 위와 같은 새로운 기술에 대한 학습용 테스트를 먼저 만들어봄

## @Conditional 학습테스트

ConfigurationTest를 진행했던 config 디렉토리에 ConditionalTest 새로 생성

```java
public class ConditionalTest {
    // ... 테스트 메소드

    @Configuration
    @Conditional(TrueCondition.class)
    static class Config1 {
        @Bean
        MyBean myBean() {
            return new MyBean();
        }
    }

    @Configuration
    @Conditional(FalseCondition.class)
    static class Config2 {
        @Bean
        MyBean myBean() {
            return new MyBean();
        }
    }

    static class MyBean {

    }

    static class TrueCondition implements Condition {
        @Override
        public boolean matches(ConditionContext context,
                                AnnotatedTypeMetadata metadata) {
            return true;
        }
    }

    static class FalseCondition implements Condition {
        @Override
        public boolean matches(ConditionContext context,
                                AnnotatedTypeMetadata metadata) {
            return false;
        }
    }
}
```
- 등록에 사용할 MyBean static class 생성
- 등록이 되는지 확인할 Config1 static class 생성
  - static class는 중첩되어 있는 클래스긴 하지만, 외부 클래스와 직접적인 관련은 없는 그냥 평범한 클래스와 같다고 생각
  - 등록이 되는지만 확인할 예정이므로 Bean 팩토리 메소드 하나 생성
- 등록이 안되는지 확인할 Config2 static class 생성
- 앞서 생성한 Config1, Config2에 @Conditional로 조건 추가
  - @Conditional에 들어갈 클래스로 TrueCondition static class 추가
  - TrueCondition은 항상 true만 return 하므로 false를 return할 FalseCondition static class도 추가 생성
  - Config1은 TrueCondition으로 @Conditional element 선언, Config2는 반대로 FalseCondition 선언

Config Class들을 생성했으니 테스트 메소드 생성
```java
public class ConditionalTest {
    @Test
    void conditional() {
        // true
        ApplicationContextRunner runner = new ApplicationContextRunner();
        runner.withUserConfiguration(Config1.class)
              .run(context -> {
                  assertThat(context).hasSingleBean(MyBean.class);
                  assertThat(context).hasSingleBean(Config1.class);
              });

        // false
        new ApplicationContextRunner().withUserConfiguration(Config2.class)
            .run(context -> {
                assertThat(context).doesNotHaveBean(MyBean.class);
                assertThat(context).doesNotHaveBean(Config2.class);
            });
    }

    // ... static class 들
}
```
- ApplicationContextRunner: Assert 라이브러리에서 만들어진 테스트 전용 애플리케이션 컨텍스트
  - .withConfiguration(): Configuration 클래스 설정
  - .run(): 람다 표현식 내에서 검증 메소드 실행 가능
- false인 경우는 ApplcationContextRunner를 재사용하지 않을 경우, 변수 선언이 필요없으니 바로 사용
- Config1을 사용하면 조건이 true이므로 MyBean, Config1이 모두 Bean으로 생성
- Config2를 사용하면 조건이 false이므로 MyBean, Config2이 모두 Bean으로 생성되지 않음

@Conditional을 메타 어노테이션으로 가지는 @TrueConditional, @FalseConditional 생성
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Conditional(TrueCondition.class)
@interface TrueConditional {

}

@Configuration
// @Conditional(TrueCondition.class) -- 제거
@TrueConditional
static class Config1 {
    @Bean
    MyBean myBean() {
        return new MyBean();
    }
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Conditional(FalseCondition.class)
@interface FalseConditional {

}

@Configuration
// @Conditional(FalseCondition.class) -- 제거
@FalseConditional
static class Config2 {
    @Bean
    MyBean myBean() {
        return new MyBean();
    }
}
```

true, false를 다 적용할 수 있고, 그 조건을 어노테이션의 element로 지정할 수 있는 방식으로 @BooleanConditional 생성
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Conditional(BooleanCondition.class)
@interface BooleanConditional {
    boolean value();
}

static class BooleanCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context,
                            AnnotatedTypeMetadata metadata) {
        Map<String, Object> attributes = metadata.getAnnotationAttributes(BooleanConditional.class.getName());
        return (Boolean) attributes.get("value");
    }
}
```
- element가 필요하니 BooleanConditional 내에 value() 하나 선언
  - element가 하나만 존재한다면, 어노테이션의 파라미터에 이름을 생략할 수 있음
- @BooleanConditional의 조건 파악에 사용될 BooleanCondition static class도 생성
  - 어노테이션의 element 값을 가져오기 위해 AnnotatedTypeMetadata 사용
  - getAnnotationAttributes(): 메타데이터에서 읽어올 어노테이션 내의 attributes(= elements). 어노테이션 이름을 활용해서 읽어옴

@BooleanConditional을 활용하도록 Config1, Config2를 수정
```java
@Configuration
// @TrueConditional -- 제거
@BooleanConditional(true)
static class Config1 {
    @Bean
    MyBean myBean() {
        return new MyBean();
    }
}

@Configuration
// @FalseConditional -- 제거
@BooleanConditional(false)
static class Config2 {
    @Bean
    MyBean myBean() {
        return new MyBean();
    }
}
```

## 커스텀 @Conditional

조건에 따라 Tomcat과 Jetty 중에서 하나를 사용하도록 설정할 예정  
스프링 부트가 자동 구성에서 가장 대표적으로 사용하고 있는 방법은 어떤 라이브러리가 이 프로젝트에 포함되어 있는가를 체크해보는 방법  
TomcatServletWebServerFactory나 JettyServletWebServerFactory는 스프링 자체에 들어가 있는 객체고 Tomcat이나 Jetty 자체의 클래스는 아님  
Tomcat은 org.apache.catalina.startup 패키지에 Tomcat이라는 클래스가 있고  
Jetty는 org.eclipse.jetty.server 패키지에 Server라는 클래스가 있음

Tomcat, Server 클래스가 존재하는지를 기준으로 각 Server를 띄우는 조건을 추가해봄
```java
// TomcatWebServerConfig.java
static class TomcatCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context,
                            AnnotatedTypeMetadata metadata) {
        return ClassUtils.isPresent(
            "org.apache.catalina.startup.Tomcat",
            context.getClassLoader()
        );
    }
}

// JettyWebServerConfig.java
static class JettyCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context,
                            AnnotatedTypeMetadata metadata) {
        return ClassUtils.isPresent(
            "org.eclipse.jetty.server.Server",
            context.getClassLoader()
        );
    }
}
```
- 스프링에 내장된 유틸리티 클래스로 ClassUtils 활용
  - ClassUtils.isPresent()를 통해 클래스가 현재 프로젝트에 존재하는지 확인
  - 클래스명만 작성하는게 아니라 패키지가 포함된 전체 경로를 String으로 작성해야 함
  - ClassLoader를 전달해야 하는데 ConditionContext에서 가져올 수 있음

이와 같이 TomcatWebServerConfig와 JettyWebServerConfig를 수정하면 현재 Tomcat, Jetty가 모두 라이브러리로 선언되어 있기 때문에  
어떤 WebServer를 띄울지 선택하지 못해 서버 실행이 실패함
```shell
org.springframework.context.ApplicationContextException: Unable to start web server
...
Caused by: org.springframework.context.ApplicationContextException: Unable to start ServletWebServerApplicationContext due to multiple ServletWebServerFactory beans : tomcatWebServerFactory,jettyWebServerFactory
...
```

그래서 build.gradle에서 Jetty 의존성을 제거하거나, Tomcat을 제거해야함
```groovy
// Jetty 의존성을 제거할 경우
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    // implementation 'org.springframework.boot:spring-boot-starter-jetty'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

// Tomcat 의존성을 제거할 경우
dependencies {
    implementation('org.springframework.boot:spring-boot-starter-web') {
        exclude group: 'org.springframework.boot', module: 'spring-boot-starter-tomcat'
    }
    implementation 'org.springframework.boot:spring-boot-starter-jetty'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```
- Jetty 의존성은 새로 설정해주었으므로 그냥 삭제하면 문제없음
- Tomcat은 spring-boot-starter-web에 내장되어 있기 때문에 해당 의존성을 가져올때 Tomcat만 제외해야함
  - exclude를 사용해서 tomcat만 제외하고 가져오도록 설정

TomcatWebServerConfig와 JettyWebServerConfig를 살펴보면, Tomcat과 Server 클래스를 판단하는 경로를 작성해주는 부분이외에는 모두 동일함  
위 테스트 코드에서 @BooleanConditional을 생성했던 것처럼 @Conditional을 메타 어노테이션으로 가지는 새로운 어노테이션 생성
```java
// ConditionalMyOnClass.java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Conditional(MyOnClassCondition.class)
public @interface ConditionalMyOnClass {
    String value();
}

// MyOnClassCondition.java
public class MyOnClassCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context,
                           AnnotatedTypeMetadata metadata) {
        Map<String, Object> attrs = metadata.getAnnotationAttributes(ConditionalMyOnClass.class.getName());
        String value = (String) attrs.get("value");
        return ClassUtils.isPresent(value, context.getClassLoader());
    }
}
```
- 어노테이션에서는 String value()로 클래스가 존재하는지 체크할 클래스의 이름을 전달받을 변수 선언
- 조건을 검사할 MyOnClassCondition도 추가 생성
  - 메타데이터에서 어노테이션의 Attribute를 가져와서 클래스의 이름 가져옴

위에서 생성한 어노테이션을 활용해서 TomcatWebServerConfig와 JettyWebServerConfig를 수정
```java
// TomcatWebServerConfig.java
@MyAutoConfiguration
@ConditionalMyOnClass("org.apache.catalina.startup.Tomcat")
public class TomcatWebServerConfig {
    // ...
}

// JettyWebServerConfig.java
@MyAutoConfiguration
@ConditionalMyOnClass("org.eclipse.jetty.server.Server")
public class JettyWebServerConfig {
    // ...
}
```
- 각 Config 클래스에 inner class로 선언되어 있던 TomcatCondition과 JettyCondition은 이제 필요없으니 제거

## 자동 구성 정보 대체하기

## 스프링 부트의 @Conditional
