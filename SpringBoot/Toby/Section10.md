# Section 10. Spring Boot 자세히 살펴보기

## 스프링 부트의 자동 구성과 테스트로 전환

그동안 만들었던 자동구성과 관련된 config 패키지, resources에 생성해둔 .imports 파일까지 모두 제거  
TobyApplication에 선언했던 어노테이션 `@MySpringBootApplication`에서 My를 제거하고 SpringBoot에서 제공해주는 어노테이션 사용  
`@SpringBootApplication`을 살펴보면 그동안 진행했던 것처럼 `@EnableAutoConfiguration`, `@ComponentScan` 등이 메타 어노테이션으로 선언된 것을 확인할 수 있음
```java
// TobyApplication.java
// @MySpringBootApplication -- 제거
@SpringBootApplication
public class TobyApplication {
    // ...
}
```

`@SpringBootApplication`으로도 기존의 테스트들이 동작하는지 확인해야 함  
다만, 일부를 수정해야함  
우선 application.properties 부터 수정
```properties
# 기존
spring.application.name=toby
my.name=ApplicationProperties
server.contextPath=/toby
server.port=9090
data.driver-class-name=org.h2.Driver
data.url=jdbc:h2:mem:
data.username=sa
data.password=

# 수정
server.servlet.context-path=/toby
server.port=9090
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:
spring.datasource.username=sa
spring.datasource.password=
```
- Spring의 자동구성에서 사용하는 property 이름으로 수정
  - 기존에 선언한 property는 간단하게 만들기 위해 임의로 지정한 이름이기 때문

위와 같이 .properties를 수정하고 서버를 실행시킨 다음 HelloApiTest를 실행해봄
```shell
# ...
> Task :test
# ...
BUILD SUCCESSFUL in 580ms
# ...
```
- 문제없이 성공하는 것을 확인할 수 있음

`@HelloBooTest`는 제거  
실습을 위해 구현한 어노테이션이고 SpringBoot에 있는 기능을 이용해서 테스트를 만들어야 하기 때문  

DataSourceTest도 문제없이 실행되는지 확인  
다만, 앞서 HelloBootTest를 제거했으므로 `@JdbcTest`로 수정해서 테스트 진행
```java
@JdbcTest
public class DataSourceTest {
    // ...
}
```
- `@JdbcTest`는 Jdbc 기능을 테스트하기 위해 SpringBoot에서 지원하는 어노테이션

문제없이 성공하는 것을 확인
```shell
# ...
2025-01-16T23:11:28.402+09:00  INFO 22598 --- [    Test worker] c.s.t.s.helloboot.DataSourceTest         : Starting DataSourceTest using Java 17.0.13 with PID 22598 (started by tuna in /Users/tuna/IdeaProjects/toby)
2025-01-16T23:11:28.403+09:00  INFO 22598 --- [    Test worker] c.s.t.s.helloboot.DataSourceTest         : No active profile set, falling back to 1 default profile: "default"
2025-01-16T23:11:28.494+09:00  INFO 22598 --- [    Test worker] beddedDataSourceBeanFactoryPostProcessor : Replacing 'dataSource' DataSource bean with embedded version *(1)
2025-01-16T23:11:28.545+09:00  INFO 22598 --- [    Test worker] o.s.j.d.e.EmbeddedDatabaseFactory        : Starting embedded database: url='jdbc:h2:mem:28fa097e-edde-4a8b-986d-e262979b4fd9;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=false', username='sa' *(2)
2025-01-16T23:11:28.636+09:00  INFO 22598 --- [    Test worker] c.s.t.s.helloboot.DataSourceTest         : Started DataSourceTest in 0.315 seconds (process running for 0.648)
```
- (1) 자동구성에 의해 기본적으로 만들어지는 DataSource를 embedded version으로 교체했다는 로깅
  - embedded DB를 활용해서 테스트하면 운영용 DB를 사용하는 것보다 테스트가 빠르게 동작함.
  - `@JdbcTest`가 embedded DB로 교체해서 사용해주는 것
- (2) DB 설정이 어떻게 되었는지 로깅

이외에 HelloBootTest를 선언해두었던 다른 테스트 클래스도 수정  
기본적으로 모두 `@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.NONE)`으로 대체  
JdbcTemplateTest의 경우에는 JDBC에 대한 테스트 이므로 DataSourceTest처럼 `@JdbcTest`로 수정

MemberRepositoryTest는 `@JdbcTest`를 사용하지 않고 `@SpringBootTest`를 사용했는데  
TobyApplication에 선언한 DB를 초기화하는 코드를 동작시켜서 컨테이너를 돌리고 테스트를 실행하는 방식으로 테스트하기 위함
```shell
# ...
2025-01-16T23:16:35.251+09:00  INFO 22715 --- [    Test worker] c.s.t.s.helloboot.MemberRepositoryTest   : Started MemberRepositoryTest in 0.395 seconds (process running for 0.743)
# ...
> Task :test
# ...
BUILD SUCCESSFUL in 1s
# ...
```
- 문제없이 모두 성공하는 것을 확인
- increaseCount가 먼저 테스트되고 findMemberFailed이 나중에 테스트되면서 findMemberFailed 테스트가 실패한다면  
  test class에 `@Transactional`을 선언하고 테스트하면 성공 (관련 내용은 아래 설명)

HelloServiceCountTest 역시 `@SpringBootTest`를 사용  
MemberRepositoryTest와 같은 방식으로 테스트를 동작하면 Service Bean을 가져와서 Repository까지 이용해서 embedded DB에 데이터가 쓰여지는 것까지 확인
```shell
# ...
2025-01-16T23:17:54.399+09:00  INFO 22732 --- [    Test worker] c.s.t.s.helloboot.HelloServiceCountTest  : Started HelloServiceCountTest in 0.381 seconds (process running for 0.722)
# ...
> Task :test
# ...
BUILD SUCCESSFUL in 1s
# ...
```
- 문제없이 모두 성공하는 것을 확인

JdbcTemplateTest는 `@JdbcTest`를 사용  
```shell
# ...
2025-01-16T23:18:24.619+09:00  INFO 22756 --- [    Test worker] c.s.t.s.helloboot.JdbcTemplateTest       : Started JdbcTemplateTest in 0.306 seconds (process running for 0.643)
# ...
> Task :test
# ...
BUILD SUCCESSFUL in 1s
# ...
```
- 문제없이 모두 성공하는 것을 확인

이 상태에서 전체 테스트를 수행해보면 HelloServiceCountTest가 실패함 (혹은 다른 테스트가 실패할 수 있음. 실행순서에 따라 다르기 때문)  
- assertThat으로 검증한 테스트가 실패함. 데이터 개수가 다르거나 존재하지 않아야 할 데이터가 존재함
  - 다른 테스트에서 embedded DB에 수정한 데이터가 그대로 조회되기 때문
  - SpringBootTest가 기본적으로 `@Transactional`이 선언되어 있지 않기 때문에 데이터가 rollback되지 않기 때문
    ```java
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.NONE)
    @Transactional // 추가 필요
    public class HelloServiceCountTest {
        // ...
    }
    ```
  - 데이터를 수정하는 모든 테스트에 `@Transactional`을 선언후 다시 실행하면 모두 성공하는 것을 확인

앞서 테스트했던 HelloApiTest를 살펴보면 테스트를 실행시킬때마다 TobyApplication을 실행해야 했음  
그래서 테스트를 수행할 때 TobyApplication을 자동으로 실행되게 할 수 있음  
코드 수정 없이 TobyApplication이 실행되어 있다면 종료하고 HelloApiTest를 실행해봄
```shell
# ...
I/O error on GET request for "http://localhost:9090/toby/hello": Connection refused
# 
```
- 애플리케이션 서버가 떠있지 않으므로 실패하는 것을 확인

이제 HelloApiTest에 다음의 코드를 추가
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT)
public class HelloApiTest {
    // ...
}
```
- `@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT)`
  - application.properties에 선언한 port를 이용해서 실제 tomcat 서버를 띄우고 테스트를 수행해달라고 요청하는 의미

위 코드를 추가하고 다시 HelloApiTest를 실행
```shell
# ...
2025-01-16T23:22:37.002+09:00  INFO 22907 --- [    Test worker] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 9090 (http) with context path '/toby' *(1)
2025-01-16T23:22:37.005+09:00  INFO 22907 --- [    Test worker] c.s.t.section10.helloboot.HelloApiTest   : Started HelloApiTest in 0.605 seconds (process running for 0.948)
# ...
> Task :test
# ...
BUILD SUCCESSFUL in 1s
# ...
```
- 테스트가 성공하는 것을 확인
- (1) application.properties로 선언해둔 port 및 contextPath로 서버가 잘 뜬걸 확인

이처럼 앞서 작성했던 코드들이 모두 SpringBoot에서 제공하는 기능들이고,  
SpringBoot에서 제공하는 기능들을 사용하도록 수정했을 때 테스트 내부 로직의 수정없이 테스트가 모두 성공하는 것을 확인할 수 있음

## 스프링 부트 자세히 살펴보기

SpringBoot가 내장하고 있는 자동 구성에 의해 알아서 동작하는 과정
- SpringBoot에서 사용할 기술을 선택하기
- Spring Initializr를 사용해서 프로젝트 생성
- SpringBoot Starter와 Dependencies를 활용한 클래스/라이브러리 추가
- @AutoConfiguration을 활용한 자동 구성 후보 로딩
- @Conditional을 활용한 매칭 조건 판별
- Default 자동 구성 인프라 Bean 준비
- application.properties를 활용한 외부설정 프로퍼티 설정

위와 같이 진행되서 자동 구성 인프라 Bean들이 최종적으로 생성

직접 작성한 코드가 등록되는 과정
- @ComponentScan
- @Component와 같은 애플리케이션 로직 Bean 로딩
- @Configuration과 같은 커스텀 인프라 Bean 로딩
- 이외의 추가 인프라 Bean 로딩

위와 같이 진행되서 유저 구성 애플리케이션 Bean이 최종적으로 생성  

정작 개발할 때 필요한 것들은 다음과 같은 것들을 알아야 함
- 어떤 기술을 사용할 것인가?
- 해당 기술에 대해 SpringBoot가 제공해주는 자동 구성 Bean은 어떤 것이 있는가?
- 해당 Bean의 속성은 어떤 것이 있는가? 또 Default 값은 무엇인가?
- 외부 프로퍼티 설정은 어떤 식으로 가능한가?
- 어떤 경우에는 해당 자동 구성 Bean이 선택되고 어떻게 조건이 달라지면 다른 종류의 자동 구성 Bean이 사용되는가?

## 자동 구성 분석 방법

SpringBoot가 제공하는 자동 구성이 어떤 게 적용되고 있는지, 어떻게 이용할 수 있는지 확인

**자동구성 클래스 Conditional 결과 로그**  
서버 실행시 자동구성과 관련된 정보들을 로그로 출력해서 확인 가능
- `-Ddebug`: JVM의 argument로 디버그 로깅을 설정하는 방법
- `--deubg`: 프로그램 전체에 대한 argument로 디버그 로깅을 설정하는 방법

SpringBoot가 imports 파일로부터 읽어온 자동 구성 후보들 중에서 Conditional Test를 수행  
Conditional 체크 후 통과해서 구동시 필요한 Bean을 등록  
선택되지 않은 것들도 이유가 출력됨. (클래스가 없거나 프로퍼티 설정이 없거나 등)  
다만, 너무 많은 내용이 출력되다보니 확인하기 힘들 수 있음 (자동 구성 클래스의 후보가 144가지인데 중첩된 클래스도 있고 그 안에 Bean 메소드에 조건이 또 있다보니)

**자동구성 클래스 Conditional 결과 Bean**  
- `ConditionEvaluationReport`: 자동으로 등록되는 Bean, Condition 체크 후의 결과를 정보로 담고 있는 Bean을 제공

**등록된 Bean 확인**  
- `ListableBeanFactory`: ApplicationContext안에 들어있는, SpringContainer 안에 생성된 Bean의 목록을 확인할 수 있게 해주는 메소드를 제공하는 인터페이스

**문서에서 관련 기술, 자동구성, 프로퍼티 확인**  
- `SpringBoot Reference`

AutoConfiguration 앞을 보면 주제가 나옴. 해당 주제를 가지고 문서를 확인  
소스코드 내에서 클래스나 인터페이스 위에 작성된 JavaDoc을 확인하면 정말 많은 정보가 작성되어 있음

**자동 구성 클래스와 조건, Bean 확인**  
- @AutoConfiguration
- @Conditional
- Condition
- @Bean

앞선 ConditionEvaluationReport에서 매칭되었다고 나온 정보를 찾으면 AutoConfiguration이 붙어있음  
어떤 조건을 만족해서 매칭이 되서 Bean이 됬는지 확인, 더 궁금하면 Condition도 확인  
어떤 타입의 Bean들이 생성되는지 확인

**프로퍼티 클래스와 바인딩**  
- Properties
- Bind
- Customizer
- Configurer

프로퍼티를 작성할 때는 그냥 텍스트로 작성하지만, 실제로는 어떤 타입으로 변경이 되는구나를 확인 가능  
또 중첩이 되어 있는 경우가 있어서, 중첩된 오브젝트 클래스는 어떤 것들이 설정되는가 확인 가능  
Binding을 해서 직접 프로퍼티를 가져오는 경우도 있음  

AutoConfiguration 자동구성 클래스를 보면, Customizer, Configurer가 있음  
Customizer는 어떤 오브젝트의 기본 설정을 변경하는 책임만 담당하는 오브젝트를 만들어서 주입받는 경우도 있음  
Configurer는 인터페이스 타입으로 Spring F/W와 관련된 주요환 기술을 설정하는 것들을 오브젝트 안에 모아놓음


추가적으로  
기본 구성 정보에서는 선택이 안되는데 특정 라이브러리를 추가하면 다른 옵션으로 바꿀수 있는 것들이 있음  
예를 들면, SpringBoot에서 기본적으로 데이터 소스는 Hikari를 사용하는데 다른 라이브러리를 선언하고 교체할 수 있음  
이런 것들은 SpringBoot의 Reference를 확인하면 어떤 것들이 가능한지 작성되어 있음

## 자동 구성 조건 결과 확인

새로운 프로젝트 SpringBootAutoConfiguration을 하나 생성하여 학습  
아무런 Dependency도 선택하지 않고 생성

프로젝트 생성후 main 메소드를 실행해보면 Spring Container가 뜬 이후에 종료됨  
Tomcat같은 서버를 연결하지 않았으니 바로 종료되는 것

Run Configuration으로 접근해서 Build and run의 우측 Modifiy options에서 add VM options로 VM 옵션을 지정 가능  
여기서는 앞서 살펴본 내용중 디버깅 로그를 확인할 수 있는 `-Ddebug` 라고 작성  
이후 main 메소드 다시 실행해보면 많은 정보들이 출력됨.  
쭉 위로 올려보면 `CONDITIONS EVALUATION REPORT` 라고 나오는 것을 확인할 수 있음  
Condition 조건을 통과한 Positive matches를 확인할 수 있음  
Condition 조건을 통과하지 못해서 매칭되지 못한 Negative matches도 확인할 수 있음

많은 로그들이 나와서 필요한 정보를 찾기 불편함  
Condition evaluation report가 컨테이너에 Bean으로 등록되기 때문에 거기서 필요한 정보들을 추출하거나 조작해서 살펴볼 수 있음  
기존에 작성했던 VM option은 제거하고 main 메소드가 있는 Application class를 일부 수정함
```java
@SpringBootApplicaiton
public class SpringBootApplication {
    @Bean
    ApplicationRunner run(ConditionEvaluationReport report) {
        return args -> {
            System.out.println(
                report.getConditionAndOutcomesBySource().entrySet().stream()
                    .filter(co -> co.getValue().isFullMatch())
                    .filter(co -> co.getKey().indexOf("Jmx") < 0)
                    .map(co -> {
                        System.out.println(co.getKey());
                        co.getValue().forEach(c -> {
                            System.out.println("\t" + c.getOutcome());
                        })
                        System.out.println();
                        return co;
                    }).count()
            );
        }
    }

    public static void main(String[] args) {
        SpringApplication.run(SpringBootApplication.class, args);
    }
}
```
- ConditionEvaluationReport Bean을 파라미터로 주입받음
- getConditionAndOutcomesBySource()는 모든 Condition 체크한 기록들이 맵 형태로 반환됨
- 필터링은 모든 value중에 Bean 메소드가 만드는 Bean을 등록할 것인가 체크하는 조건을 통과한 것만 필터링하는 것
- getOutcome()은 어떤 Condition을 통과했는지 확인 가능
- JMX를 제외한 자동구성 Bean이 몇개가 등록되는지 확인해보면 13개가 등록됨

## Core 자동 구성 살펴보기

가장 먼저 AopAutoConfiguration을 살펴보면,  
위에 달려있는 Condition이 @ConditionalOnProperty로 확인 가능  
spring.aop.auto 프로퍼티가 true값이면 매칭되는 조건인데, 선언한 적 없는데도 매칭되었음. 이는 마지막의 matchIfMissting 조건 때문임  
프로퍼티의 default값을 havingValue로 지정한 것과 동일하게 설정해주는 느낌

이후 캐시와 관련된 자동 구성이 3가지 나옴 (Generic, NoOp, Simple)  
관련해서는 [Reference](https://docs.spring.io/spring-boot/reference/io/caching.html)를 찾아봄  
SpringBoot에서 사용하는 방식, 주의할점 등등 자세한 설명이 나와있음

이후 Lifecycle과 관련된 Bean이 등록되어있는데, 직접 다룰 일은 거의 없음  
Spring이 제공하는 컨테이너 라이프사이클과 관련된 Bean을 만들어주는 자동 구성

이후 PropertyPlaceholder는 앞서 학습한 내용중 하나

core 쪽에서 관심을 가질만한 건 TaskExecution 자동 구성  
직접 접근해서 살펴보면 ThreadPoolTaskExecutor를 조건으로 잡고 있는데 해당 class는 Spring에 존재  
그리고 앞서 학습한 @EnableConfigurationProperties 가 선언되어 있음  
이제 어떤 Bean을 만드는지 확인해보면, Builder Bean이 따로 존재하는 것을 확인할 수 있음  
Builder Bean에서 주입받은 TaskExecutionProperties를 확인해보면 어떤 프로퍼티로 설정할 수 있는지 확인 가능  
역시 관련해서도 [Reference](https://docs.spring.io/spring-boot/reference/features/task-execution-and-scheduling.html)를 살펴보면 추가적으로 확인 가능  
ThreadPoolTaskExectuor를 직접 선언해서 사용하면 corePoolSize가 1로 지정되어 있음

## Web 자동 구성 살펴보기

Web 모듈을 로딩하려면 의존성으로 spring-boot-starter-web을 넣어주면 됨  
```groovy
// build.gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
}
```
- starter-web을 추가하면 기존에 추가되어있던 starter는 web에 포함되어있기 때문에 따로 선언하지 않아도 됨
  - IDEA의 우측에서 gradle을 확인할 수 있는데 해당 탭에서 Dependencies를 확인가능

다음으로 자동 구성이 어떤게 나오는지 살펴봄  
앞서 살펴본 내용은 JMX 관련된 항목을 제외하면 13개의 자동 구성 클래스와 Bean들이 조건을 통과하는 걸로 나왔음  
의존성을 추가한 후 앞서 작성한 run()을 실행해보면 62개가 출력되는 것을 확인할 수 있음

**HttpMessageConvertersAutoConfiguration**  
- JSON 형태로 인코딩된 RequestBody 파싱 혹은 ResponseBody 생성하는 등의 작업을 담당
- JSON 뿐만 아니라 여러 메시지 컨버터들이 Spring Web에 포함되어있는데, 그런 것들을 자동 구성 해주는 클래스
- 3개의 Configuration 파일을 import함
  - JacksonHttpMessageConvertersConfiguration
  - GsonHttpMessageConvertersConfiguration
  - JsonbHttpMessageConvertersConfiguration
- 기본적으로 메시지 컨버터를 등록해주는 Bean도 선언되어 있음

JSON을 처리하는 Jackson과 관련된 내용이 자동구성 클래스에 많이 포함되어 있음  
그 중 기억할만한 내용은 Jackson2ObjectMapperConfiguration  

**Jackson2ObjectMapperConfiguration**
- ObjectMapper를 자주 사용하는데, SpringBoot에서 기본적으로 만들어주는 Bean이 있음
  - 직접 Bean을 선언하거나 사용하고 싶은 경우 Jackson2ObjectMapperBuilder.class를 참고하면 유용함
  - CustomizerConfiguration 및 타고 들어가면 보이는 JacksonProperties까지 참고하면 유용함

**RestTemplateAutoConfiguration**
- RestTemplate Bean을 직접 생성하지 않음. 대신 RestTemplateBuilder Bean을 제공함

**EmbeddedWebServerFactoryCustomizerAutoConfiguration**
- 이전 실습에서 공부했던 ServerProperties를 확인할 수 있음
  - Servlet 컨테이너의 기본 동작을 변경할 수 있는 프로퍼티들을 확인할 수 있음
- ServerProperties를 받아서 바로 톰캣 웹서버를 만드는게 아니라 여기서 또 Customizer를 생성함
  - Customizer에 ServerProperties를 전달해서 설정한 프로퍼티로 웹서버 설정을 변경함

**ServletWebServerFactoryAutoConfiguration**
- 여기서 Tomcat, Jetty, UnderTow에 대한 설정들을 확인할 수 있음
- Customzier를 받아와서 각 서버 설정들을 변경하는 것을 확인할 수 있음

추가적으로 참고하면 좋을 Configuration
- DispatcherServletAutoConfiguration
- HttpEncodingAutoConfiguration
- MultipartAutoConfiguration
- ErrorMvcAutoConfiguration

## Jdbc 자동 구성 살펴보기

Jdbc 모듈을 로딩하려면 의존성으로 spring-boot-starter-jdbc을 넣어주면 됨  
```groovy
// build.gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-jdbc'
}
```
- 앞서 살펴본것과 동일하게 gradle 탭에서 어떤 의존성을 로딩하는지 확인해보면 좋음

어떤 자동 구성 클래스가 등록되는지 살펴보기 위해 run()을 실행하면 에러가 발생함  
`Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured`  
DataSource 정보를 추가해주거나, 임베디드 DB를 사용하는 방식으로 선언해주면 된다는점도 아래에 명시되어있음.

임베디드 DB로 hsql DB를 넣어서 테스트해봄
```groovy
// build.gradle
dependencies {
    implementation 'org.hsqldb:hsqldb:2.7.1'
}
```

이제 다시 run() 실행해보면 자동 구성 클래스가 등록되는 것을 확인할 수 있음  
32개의 자동 구성 클래스가 등록됨

살펴보면 좋을 클래스들
- PersistenceExceptionTranslationAutoConfiguration
- DataSourceAutoConfiguration
  - DataSourceProperties
- DataSourceTransactionManagerAutoConfiguration
  - JdbcTransactionManager
- JdbcTemplateAutoConfiguration
  - JdbcProperties
- DataSourceInitializationConfiguration
- SqlInitializationAutoConfiguration
- TransactionAutoConfiguration
