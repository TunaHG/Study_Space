# Section 06. 자동 구성 기반 애플리케이션

## 메타 어노테이션과 합성 어노테이션

메타 어노테이션,  
앞서 @Component를 설명하면서 @Controller, @Service가 @Component를 자신의 어노테이션으로 부여해서 동일한 효과를 만들어 내는 것을 확인했음  
어노테이션의 이름으로 역할을 구분하는 등의 추가적인 정보를 얻을수 있고, 추가적인 메타 어노테이션으로 부가적인 효과를 기대할 수 있음

다만, 메타 어노테이션은 상속과는 다름  
모든 어노테이션이 메타 어노테이션이 될 수는 없음. 어노테이션 선언중 Target에 어노테이션 타입이 선언되어 있어야 메타 어노테이션으로 사용 가능

Spring 말고 다른 라이브러리에서도 메타 어노테이션을 잘 활용하고 있음  
JUnit5를 예를 들어 살펴봄

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@UnitTest
@interface FastUnitTest {
    
}

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE, ElementType.METHOD})
@Test
@interface UnitTest {
    
}

public class HelloServiceTest {
    @UnitTest
    void simpleHelloService() {
        SimpleHelloService helloService = new SimpleHelloService();

        String res = helloService.sayHello("Spring");

        assertThat(res).isEqualTo("Hello, Spring!");
    }

    @FastUnitTest
    void helloDecorator() {
        HelloDecorator decorator = new HelloDecorator(name -> name);

        String res = decorator.sayHello("Spring");

        assertThat(res).isEqualTo("*Spring*");
    }
}
```
- @Test가 선언되어 있으면 테스트 메소드로 인식하고 실행
- 단위테스트임을 명시하기 위해 @UnitTest를 새로 생성
  - @Test를 메타 어노테이션으로 가지도록 생성
- 빨리 돌아가는 단위 테스트임을 명시하기 위해 @FastUnitTest를 새로 생성
  - @UnitTest를 메타 어노테이션을 생성
  - 다만, @UnitTest의 @Target에 ANNOTATION_TYPE이 선언되어 있어야 함

메타 어노테이션 이외에도 합성 어노테이션(Composed Annotation)이라 불리는 어노테이션 활용 방법이 있음  
메타 어노테이션을 하나 이상 적용해서 만드는 경우에 이렇게 부름  
어노테이션이 4~5개 선언되고 반복적으로 사용되면 합성 어노테이션 형태로 만들어서 간략하게 사용할 수 있음

## 합성 어노테이션의 적용

TobyApplication.java의 변형 형태를 비교하며 살펴봄

```java
// 초기 버전
@SpringBootApplication
public class TobyApplication {
    public static void main(String[] args) {
        SpringApplication.run(TobyApplication.class, args);
    }
}

// 수정 버전
@Configuration
@ComponentScan
public class TobyApplication {

    @Bean
    public ServletWebServerFactory servletWebServerFactory() {
        return new TomcatServletWebServerFactory();
    }

    @Bean
    public DispatcherServlet dispatcherServlet() {
        return new DispatcherServlet();
    }

    public static void main(String[] args) {
        SpringApplication.run(TobyApplication.class, args);
    }

}
```
- 수정 버전에서 초기 버전으로 어떻게 변화하는지 단계적으로 살펴봄

우선 클래스단에 선언되어 있는 어노테이션 두개를 합성 어노테이션으로 하나의 어노테이션을 쓰도록 수정

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Configuration
@ComponentScan
public @interface MySpringBootApplication {
}
```
- 새로운 어노테이션 @MySpringBootApplication 생성
- Retention RUNTIME 선언
  - 원래 어노테이션의 Retention default 값은 CLASS임. 
    이는 어노테이션 정보가 컴파일된 클래스 파일까지는 살아있지만 런타임에 메모리로 로딩할때는 정보가 사라진다는 의미
  - 그래서 런타임까지 어노테이션 정보가 유지되도록 RUNTIME으로 선언
- Target TYPE 선언
  - TYPE - Class, Interface, Enum이 TYPE에 해당됨
- 수정 버전에서 사용하던 @Configuration, @ComponentScan을 메타 어노테이션으로 선언

그 다음으로는 Bean을 생성하는 두 개의 팩토리 메소드를 수정

```java
@Configuration
public class Config {

    @Bean
    public ServletWebServerFactory servletWebServerFactory() {
        return new TomcatServletWebServerFactory();
    }

    @Bean
    public DispatcherServlet dispatcherServlet() {
        return new DispatcherServlet();
    }
}
```
- Bean을 등록하는 구성 정보 클래스를 별도로 생성
- 해당 클래스가 스프링에서 구성 정보로 활용되려면 @Configuration이 클래스단에 선언되어야 함
  - 그냥 @Component로 선언해도 @MySpringBootApplication에 @ComponentScan이 있어서 동작하지만, 
    @Configuration내에 @Component가 메타 어노테이션으로 선언되어 있으니 메타 어노테이션을 활용

최종적으로 TobyApplication은 다음과 같은 초기와 비슷한 형태가 됨

```java
// 수정 버전
@MySpringBootApplication
public class TobyApplication {
    public static void main(String[] args) {
        SpringApplication.run(TobyApplication.class, args);
    }
}

// 초기 버전
@SpringBootApplication
public class TobyApplication {
    public static void main(String[] args) {
        SpringApplication.run(TobyApplication.class, args);
    }
}
```

## Bean 오브젝트의 역할과 구분

지금까지 만들었던 Bean들을 살펴봄  
HelloController, HelloDecorator, SimpleHelloService와 같은 직접 코드로 기능을 만들었던 클래스들  
TomcatServletWebServerFactory, DispatcherServlet과 같은 내장형 서블릿 컨테이너를 이용하는 방식으로 동작하기 위해 요구된 클래스들

어떤 식으로 위 Bean들을 구분해서 구성 정보를 어떤 전략으로 작성할 것인지 고민

우선 스프링 컨테이너에 올라가는 Bean들을 구분하는 방법을 살펴봄  
크게 스프링 컨테이너가 생성하고 관리하는 Bean들은 컨테이너 인프라스트럭쳐 Bean과 애플리케이션 Bean으로 구분  

**애플리케이션 Bean은 개발자가 어떤 Bean을 사용하겠다고 명시적으로 구성 정보를 제공한 것**  
예를 들면, HelloController, SimpleHelloService, DataSource, JpaEntityManagerFactory, 등등  

**컨테이너 인프라스트럭쳐 Bean은 스프링 컨테이너 자신이거나 기능을 확장하면서 추가해온 것들**  
개발을 할때 Bean으로 등록해달라고 요청하지 않지만 컨테이너가 스스로 Bean으로 등록해서 동작시키는 방식으로 이용  
예를들면, ApplicationContext, Environment, BeanPostProcessor, 등등

컨테이너 인프라스트럭쳐 Bean은 일반적으로 우리의 관심사가 아님  
Bean이기 때문에 DI 받아서 사용할 수는 있음  

애플리케이션 Bean도 세부적으로 두 개로 구분할 수 있음  
애플리케이션 로직 Bean과 애플리케이션 인프라스트럭쳐 Bean  
**애플리케이션 로직 Bean은 애플리케이션의 기능, 비즈니스 로직, 도메인 로직 등을 담고 있는 Bean**  
**애플리케이션 인프라스트럭쳐 Bean은 기술과 관련된 것으로, 직접 작성하지 않고 이미 만들어진 것을 Bean 구성 정보를 작성해서 사용하는 Bean**

앞서 살펴본 지금까지 직접 만들었던 Bean으로 다시 살펴보면,  
HelloController, HelloDecorator, SimpleHelloService는 애플리케이션 로직 Bean  
TomcatServletWebServerFactory, DispatcherServlet은 애플리케이션 인프라스트럭쳐 Bean  
컨테이너가 뜰 때 항상 필요해서 컨테이너 인프라스트럭쳐 Bean이라고 생각할 수 있지만 명시적으로 Bean을 선언해야 등록이 되고 사용되기 때문에 애플리케이션 인프라스트럭쳐 Bean으로 생각

Spring Boot의 개발자들이 분류하는 방식으로 살펴보면,  
HelloController, HelloDecorator, SimpleHelloService는 **사용자 구성정보**를 담고 있는 Bean  
이건 ComponentScan으로 자동으로 자바 코드와 어노테이션에서 구성 정보를 읽어오는 방식 활용  
TomcatServletWebServerFactory, DispatcherServlet, DataSource 등등은 **자동 구성정보**로 구성정보가 만들어지는 Bean  
자동 구성 정보라는 매커니즘을 통해서 등록이 되는 방식 활용

좌측 ComponentScan으로 등록하는 방법은 기존에 살펴봤으니, 자동 구성 정보를 등록하는 방법 살펴봄  
필요한 자동 구성 정보 Bean들이 담긴 Configuration 클래스를 따로 구성해둠  
그리고 Spring Boot가 애플리케이션의 필요에 따라 Configuration들을 골라서 필요한 방식으로 구성해서 자동으로 적용  
자동 구성 정보를 이용한 AutoConfiguration이란 방식을 적용해볼 예정

## 인프라 빈 구성 정보의 분리

사용자 구성정보는 주로 ComponentScan에 의해서 등록이 됨

```java
// TobyApplication.java
@MySpringBootApplication
public class TobyApplication {
    // ...
}

// MySpringBootApplication.java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Configuration
@ComponentScan
public @interface MySpringBootApplication {
}
```
- ComponentScan은 TobyApplication에 붙인 우리가 만든 합성 어노테이션 `@MySpringBootApplication` 
  - 그 안에 존재하는 `@ComponentScan`에 의해서 패키지 밑에 있는 Bean들이 자동으로 등록됨
- Configuration이 선언되어 있어서 해당 어노테이션이 선언된 TobyApplication이 Bean으로 등록됨

자동 구성 정보라고 판단되는 TomcatServletWebServerFactory, DispatcherServlet는 이전에 Config 클래스에 생성했었음  
Spring Boot 스타일로 애플리케이션을 만들때는 Config를 지금처럼 직접 Bean으로 등록하는 일을 하지 않아도 자동으로 등록이 되게 만들어야 함  
그래서 ComponentScan 대상에서 제외되도록 먼저 설정

ComponentScan은 basePackage를 선언하지 않으면 해당 파일이 선언되어진 패키지를 기준으로 인식  
그래서 패키지를 옮겨버리면 컴포넌트 스캔 대상에서 제외됨. config 패키지를 하나 생성해서 해당 패키지로 Config 클래스를 이동  
그 후 실행하면 TomcatServletWebServerFactory, DispatcherServlet가 없기 때문에 동작이 실패함

컴포넌트 스캔 대상으로 만들지 않더라도 구성 정보에 포함은 시켜야 함  
TobyApplication에 선언된 `@MySpringBootApplication`을 기준으로 Config을 등록해야 함

```java
// MySpringBootApplication.java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Configuration
@ComponentScan
@Import(Config.class)
public @interface MySpringBootApplication {
}
```
- `@Configuration`이 붙은 클래스들을 @Import를 이용해서 뒤에 클래스 이름을 지정하면 구성 정보에 직접 추가 가능

ComponentScan과 Import가 뭐가 다른지?  
일단 Config 내에 존재하던 두 Bean을 각각의 Configuration 클래스로 분리  
AutoConfiguration은 미리 준비해둔 Configuration을 스프링 부트가 어떤게 필요한지 판단하고 자동으로 사용하게 해주는 것  

```java
// TomcatWebServerConfig.java
@Configuration
public class TomcatWebServerConfig {
    @Bean
    public ServletWebServerFactory servletWebServerFactory() {
        return new TomcatServletWebServerFactory();
    }
}

// DispatcherServletConfig.java
@Configuration
public class DispatcherServletConfig {
    @Bean
    public DispatcherServlet dispatcherServlet() {
        return new DispatcherServlet();
    }
}

// MySpringBootApplication.java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Configuration
@ComponentScan
@Import({TomcatWebServerConfig.class, DispatcherServletConfig.class})
public @interface MySpringBootApplication {
}
```
- 두 Configuration 클래스 생성 후 `@MySpringBootApplication`의 `@Import`도 두 클래스를 모두 선언
  - 두 AutoConfiguration 클래스는 `config/autoconfig` 패키지에 생성

AutoConfiguration 대상 클래스가 늘어날 때마다 Import문이 점점 늘어나야 하는데,  
최상위 레벨의 어노테이션에 이런 정보들이 다 나열되는 것은 피하는게 좋음  
그래서 어노테이션을 하나 더 부여

```java
// EnableMyAutoConfiguration.java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Import({TomcatWebServerConfig.class, DispatcherServletConfig.class})
public @interface EnableMyAutoConfiguration {
}

// MySpringBootApplication.java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Configuration
@ComponentScan
public @interface MySpringBootApplication {
}

// TobyApplication.java
@MySpringBootApplication
@EnableMyAutoConfiguration
public class TobyApplication {
    // ...
}
```
- `@EnableMyAutoConfiguration`에 `@MySpringBootApplication`에 선언했던 Import를 모두 가져옴
- `@MySpringBootApplication`에 `@Import`를 제거하고 TobyApplication에 `@EnableMyAutoConfiguration` 선언

순수하게 애플리케이션 로직을 담고 있는 클래스 외에 프레임워크, 라이브러리 처럼 재사용이 가능하도록 만들어지는 부분들은 애플리케이션 패키지에서 다 제거  
MySpringBootApplication.java도 `config` 패키지로 이동

> `@ComponentScan`과 `@Import`의 추가 비교  
> `@ComponentScan`은 일반적으로 개발하는 중 Bean 클래스를 계속 추가하는 애플리케이션 코드 내의 Bean을 등록하는데 사용하기 적합  
> `@Import`는 자동완성과 같이 어느 정도 등록할 Bean의 후보가 명확하게 사전 정의된 경우에 사용하는 것이 적합

## 동적인 자동 구성 정보 등록

지금은 @Import 하는 클래스들의 이름이 하드 코딩되어 있음. 모든 SpringBoot가 해당 클래스들을 사용하면 괜찮지만, 그렇지 않음  
그래서 어떤 Configuration들을 동적으로 가져올 수 있는 매커니즘을 도입해야 함  
동적이라는 것은 @EnableMyAutoConfiguration을 수정하지 않아도 Configuration을 추가할 수 있다는 것

Import Selector라는 것을 활용해야 함. (Spring 3.1부터 도입)  
ImportSelector.java 파일을 살펴봄

`String[] selectImports(AnnotaionMetadata importingClassMetadata);`  
import 할 Configuration 클래스의 이름을 String으로 만들어주면, String에 해당하는 Configuration 클래스들을 컨테이너가 구성정보로 사용함  
이번엔 해당 메소드를 구현하는게 아니라 이 메소드를 한번 더 확장한 DefaultImportSelector 라는걸 사용  
Configuration 클래스에 구성 정보 생성 작업이 모두 끝난 다음에 ImportSelector가 동작하도록 순서를 지연할 수 있게 만들어주는 것

MyAutoConfigImportSelector 클래스를 새로 생성
```java
public class MyAutoConfigImportSelector implements DeferredImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[] {
            "com.study.toby.config.autoconfig.DispatcherServletConfig",
            "com.study.toby.config.autoconfig.TomcatWebServerConfig"
        };
    }
}
```
- DeferredImportSelector는 ImportSelector의 서브 인터페이스
  - 구현해야될 메소드는 동일
- selectImports() 메소드 반환값으로 기존 Config 클래스 두 개를 디렉토리 포함해서 String으로 전달
- ImportSelector는 @Configuration을 붙일 필요가 없음

이제 Import로 MyAutoConfigImportSelector를 가져오도록 수정
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Import(MyAutoConfigImportSelector.class)
public @interface EnableMyAutoConfiguration {
}
```

서버를 동작해보면 정상 실행!  
MyAutoConfigImportSelector.selectImport()의 String 값들을 주석처리하고 다시 실행해보면 에러 발생

> 여기서 쭉 따라왔으면 실행시켰을 때, 에러가 발생하지 않고 정상적으로 동작할 수도 있음  
> 그 이유는 config 패키지와 @ComponentScan이 메타 어노테이션으로 추가되어 있는 @MySpringBootApplication을 선언한 TobyApplication이 하나의 패키지 내에 있기 때문  
> TobyApplication에 붙인 어노테이션의 @ComponentScan의 basePackages를 설정하지 않았기 때문에  
> 선언된 TobyApplication이 존재하는 디렉토리 하위의 모든 디렉토리를 Scan하고 있기 때문

## 자동 구성 정보 파일 분리

소스 코드에 등록해 놨던 정보를 외부 설정 파일로 빼내는 작업 진행  
읽어오는 Configuration은 SpringBoot의 자동 구성 정보 생성에 사용할 것들이기 때문에 Annotation을 또 하나 생성  
```java
// MyAutoConfiguration.java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Configuration
public @interface MyAutoConfiguration {
}
```
- @Configuration을 메타 어노테이션으로 선언
- 자동 구성 방식에 사용할 Configuration 클래스의 목록을 추가

MyAutoConfigImportSelector를 수정
```java
public class MyAutoConfigImportSelector implements DeferredImportSelector {
    private final ClassLoader classLoader;

    public MyAutoConfigImportSelector(ClassLoader classLoader) {
        this.classLoader = classLoader;
    }

    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        ImportCandidates candidates = ImportCandidates.load(MyAutoConfiguration.class, classLoader);

        return candidates.getCandidates().toArray(new String[0]);
    }
}
```
- ClassLoader
  - 어떤 애플리케이션의 classpath에서 리소스를 읽어올 때는 클래스 로더를 사용
    - 클래스 로더는 스프링 컨테이너가 Bean을 생성하기 위해 클래스를 로딩할 때 사용
  - BeanClassLoaderAware라는 인터페이스를 구현하면 스프링 컨테이너가 Bean 클래스 로더를 주입해줌
    - 다른 방법은, 구현하지 않고 생성자를 통해 주입이 되도록 만드는 방법. 이 방법을 사용
- load()로 가져온 ImportCandidates에는 Configuration 클래스들의 목록이 들어있음
  - 그 목록을 selectImports()의 return 값으로 전달

ImportCandidates.load()를 들어가서 살펴보면,  
클래스 패스의 META-INF/spring/full-qualified-annotation-name.imports 라는 파일을 읽어온다고 나와있음  
그래서 해당 파일을 생성해봄  

```
# src/mainresoureces/META-INF/spring 디렉토리에
# com.study.toby.config.MyAutoConfiguration.imports 파일 생성

com.study.toby.config.autoconfig.DispatcherServletConfig
com.study.toby.config.autoconfig.TomcatWebServerConfig
```

서버를 동작해보면 정상 실행!  
...MyAutoConfiguration.imports 파일의 내용을 지우고 다시 실행해보면 에러 발생

## 자동 구성 어노테이션 적용

.imports 파일에 작성한 두 Configuration 클래스에는 기존 @Configuration을 제거하고 직접 생성한 @MyAutoConfiguration을 붙임  
이렇게 해야 동작하는 것은 아닌데, 일종의 관례

그리고 MyAutoCongiruation에 붙은 메타 어노테이션 @Configuration을 수정  
```java
// TomcatWebServerConfig.java
@MyAutoConfiguration
public class TomcatWebServerConfig {
    // ...
}

// DispatcherServletConfig.java
@MyAutoConfiguration
public class DispatcherServletConfig {
    // ...
}
```
- proxyBeanMethod 값을 false로 수정

## @Configuration과 proxyBeanMethods

@Configuration의 동작 방식에 대해 이해하기 위해 테스트를 작성  
기존 테스트와 별도의 패키지로 분리해서 ConfigurationTest 생성
```java
// package com.study.toby.config
public class ConfigurationTest {
    @Test
    void configuration() {
        
    }
}
```

Common이라는 Bean을 의존하는 Bean1, Bean2가 있다고 가정  
Bean은 싱글톤으로 생성되기 때문에 Bean1, 2에 주입되는 Common이 같은 Bean이어야 함  
먼저 해당 Bean Class들을 생성
```java
```
- MyConfig를 살펴보면, Bean1과 Bean2의 factory 메소드를 생성할 때, common() 메소드를 호출하고 있는데  
  이는 각각 Common을 생성하는 코드기 때문에 결국 Bean1과 Bean2가 가지는 Common이 다르게 됨
- 이를 테스트 하는 코드가 테스트 메소드에 존재함
  - 두 객체의 값이 같은지를 비교하는 isSameAs() 활용
  - MyConfig 오브젝트를 만들고, Bean1, Bean2를 가져온 다음 두 오브젝트의 common이 같은지 비교

다만, MyConfig를 스프링 컨테이너의 구성 정보로 사용하게 되면 동작하는 방식이 달라짐
```java
```
- MyConfig를 직접 생성하지 않고, 애플리케이션 컨텍스트를 만들어서 MyConfig Bean을 등록해주고 초기화를 진행함
- 이후 애플리케이션 컨텍스트로부터 Bean1, Bean2 오브젝트를 가져옴
- Bean1, Bean2에서 가지고 있는 Common을 비교

기본적으로 @Configuration proxyBeanMethod가 default 값인 true로 설정되어 있는 경우  
Configuration 클래스가 Bean으로 등록될 때 직접 Bean으로 등록되는 것이 아니라  
Proxy Object를 앞에 하나 두고 그게 Bean으로 등록이 됨

어떤 식으로 Proxy가 만들어지는지 테스트를 통해 살펴봄  
```java
```
- MyConfigProxy 클래스를 추가
  - common()을 오버라이딩하되, super.common()으로부터 받는 반환값 Common을 변수로 설정
- 이를 테스트하는 proxyCommonMethod() 테스트 메소드 생성
  - 이전에 진행해본 테스트와 동일하게 MyConfigProxy 생성후 Bean1, 2를 가져오고 각각의 Common이 같은지 비교
- 이전에 실패했던 테스트와 다르게 성공하는 것을 확인할 수 있음

팩토리 메소드를 사용해서 오브젝트를 생성하는 코드를 여러 번 호출하더라도 한 개의 오브젝트만 사용됨  
Spring의 기본적인 동작 방식인데 이걸 자바 코드로만 가능하게 하려다 보니 약간 다르게 동작하는 위험성이 생김  
하나의 Bean을 두 개 이상의 다른 Bean에서 의존하고 있다면 팩토리 메소드를 호출할 때마다 새로운 Bean이 만들어지는 위험성  
스프링이 그것을 해결하기 위해 @Configuration이 붙은 클래스는 기본적으로 Proxy를 만들어서 기능을 확장

앞서 @MyAutoConfiguration에서는 proxyBeanMethod를 false로 수정했음  
Bean 팩토리 메소드를 통해서 Bean 오브젝트를 만들 때 또 다른 Bean 팩토리 메소드를 호출해서 의존 오브젝트를 가져오는 식으로  
코드를 작성하지 않았다면 굳이 매번 시간이 걸리고 비용이 드는 Proxy를 만드는 방식으로 이걸 사용할 필요가 없기 때문

스프링에 적용된 것을 찾아봄  
@EnableScheduling를 살펴보면 @Import로 SchedulingConfiguration이라는 다른 Configuration 클래스를 로딩함  
SchedulingConfiguration로 들어가서 살펴보면 클래스 Bean을 생성하는데, Bean을 생성하는 동안 다른 오브젝트를 의존하지 않고 있음  
이런 경우라면 굳이 매번 Proxy를 만들어서 적용할 필요가 없으니 proxyBeanMethod 값이 false로 선언되어 있음
