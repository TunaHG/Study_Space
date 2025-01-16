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

## 자동 구성 분석 방법

## 자동 구성 조건 결과 확인

## Core 자동 구성 살펴보기

## Web 자동 구성 살펴보기

## Jdbc 자동 구성 살펴보기

## 정리
