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

## 빈 오브젝트의 역할과 구분

## 인프라 빈 구성 정보의 분리

## 동적인 자동 구성 정보 등록

## 자동 구성 정보 파일 분리

## 자동 구성 어노테이션 적용

## @Configuration과 proxyBeanMethods
