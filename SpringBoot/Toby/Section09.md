# Section 09. Spring JDBC 자동 구성 개발

그동안 익힌 Spring Boot의 자동 구성을 Spring JDBC를 사용하는 인프라스트럭처 Bean을 만드는데 활용할 예정

## 자동 구성 클래스와 빈 설계

자동 구성 클래스를 설계할 때는 어떤 조건을 가질때 사용될 것인가를 먼저 결정해야 함  
보통 특정 클래스가 라이브러리에 포함되어 있는가, 스타터 등에 dependency로 잡혀있는가를 판단하는 기준이 있음  
이 때는 Spring 프레임워크에 있는 JDBC Operations 인터페이스가 있는지 확인

DataSource 인터페이스를 구현한 Bean을 먼저 생성해야 함
Spring 안에 간단하게 만들어진 SimpleDriverDataSource가 있음  
해당 Bean이 Spring 안에 포함되어 있어 다른 라이브러리를 포함하지 않아도 기본적으로 동작은 하게 만들 수 있지만 매우 비효율적임  
요즘은 HikariDataSource를 많이 사용함. 가장 성능이 좋다고 알려져 있음  
HikariDataSource가 존재하면 사용하고, 아니면 SimpleDriverDataSource를 사용하도록 개발할 예정

DataSource는 DB 와의 연결을 담당하는데, 필요한 정보는 코드에 고정시켜 놓을 수 없음  
그래서 프로퍼티를 통해 지정할 수 있도록 만들어야 함. 그래서 DataSourceProperties도 생성할 예정

코드에서 SQL을 이용해서 CRUD를 만들 때 전통적인 Java의 JDBC API는 너무 불편함  
거기에 템플릿 콜백 패턴을 적용해서 편리하게 사용할 수 있도록 만들어진 JdbcTemplate을 사용할 예정  
JdbcTemplate은 DataSource 타입의 Bean을 의존하게 되어 있음. 그래서 JdbcTemplate도 자동 구성 Bean으로 등록되도록 개발할 예정

하나가 더 필요한데 JdbcTransactionManager가 필요함.  
DataSource를 사용해서 DB에 액세스한 코드가 하나의 트랜잭션 안에서 잘 수행이 되도록 트랜잭션 경계 설정 작업을 해줘야 함  
이 트랜잭션을 관리해주는 PlatformTransactionManager 타입의 Bean으로 JdbcTransactionManager를 사용할 예정  
트랜잭션 매니지먼트 기능을 Spring에 요청하면 `@Transactional`을 이용하는 방식도 지원이 됨

위 구조로 자동 구성이 만들어질 때 조건을 하나 더 걸건데, @ConditionalOnSingleCandidate  
JdbcTemplate이나 JdbcTransactionManager 같은 것은 DataSource를 하나를 주입 받아서 동작하는데,  
사실 Java 프로그램에서 여러 개의 DataSource를 가지는 것도 불가능하지 않음  
그럴 경우 어떤 DataSource를 가져다가 JdbcTemplate이나 JdbcTransactionManager를 만들지 결정하기 어려움  
그래서 자동 구성에서는 기본적인 전제 조건을 DataSource 타입의 Bean이 하나만 존재하는 경우에만 위 Bean들을 만드는 방법을 이용

DataSource만 있는게 아니라 실제 DB가 있어야 함  
하지만 OracleDB나 MySQL이든 설치하고 연결해서 실행하기 번거로움  
그래서 인메모리 DB라고 하는 애플리케이션이 뜰 때 같이 떠서 동작하고, 애플리케이션이 종료될 때 같이 종료되는 h2 DB를 사용  
h2 DB를 임베디드 방식을 사용할 때는 DB 커넥션에 필요한 Url, username, password를 신경쓸 필요가 없음  

## DataSource 자동 구성 클래스

이제 위에서 설계한 대로 코드를 작성해봄  
먼저 DataSourceConfig
```java
// config/autoconfig/DataSourceConfig.java
@MyAutoConfiguration
@ConditionalMyOnClass("org.springframework.jdbc.core.JdbcOperations")
@EnableMyConfigurationProperties(MyDataSourceProperties.class)
public class DataSourceConfig {
    @Bean
    DataSource dataSource(MyDataSourceProperties properties) throws ClassNotFoundException {
        SimpleDriverDataSource dataSource = new SimpleDriverDataSource();

        dataSource.setDriverClass((Class<? extends Driver>) Class.forName(properties.getDriverClassName()));
        dataSource.setUrl(properties.getUrl());
        dataSource.setUsername(properties.getUsername());
        dataSource.setPassword(properties.getPassword());

        return dataSource;
    }
}

// config/autoconfig/MyDataSourceProperties.java
@MyConfigurationProperties(prefix = "data")
public class MyDataSourceProperties {
    private String driverClassName;
    private String url;
    private String username;
    private String password;

    // ... getter/setter methods
}
```
- 자동 구성 Config 이므로 `@MyAutoConfiguration`을 선언해주고, `.imports` 파일에도 선언해줘야 함
  ```
  com.study.toby.section09.config.autoconfig.DataSourceConfig
  com.study.toby.section09.config.autoconfig.PropertyPostProcessorConfig
  ...
  ```
- Jdbc Operations를 조건으로 `@ConditionalMyOnClass` 추가, `build.gradle`에 JDBC 의존성 추가해줘야 함
  ```groovy
  dependencies {
    // ...
    implementation('org.springframework:spring-jdbc')
    // ...
  }
  ```
- DataSource Bean 팩토리 메소드 생성
  - DB 연결정보를 가져와야 하는데, 프로퍼티를 사용해서 가져올 것이므로 주입받을 Properties 선언 및 생성
  - DataSource의 driverClass는 Class 타입을 받기 때문에 Properties에 선언된 클래스 이름으로 클래스를 가져오도록 구현
- 자동 구성 클래스가 `@Conditional` 조건이 충족되서 사용될때 Property 파일이 Bean으로 등록이 되도록 `@EnableMyConfigurationProperties` 선언

이대로 서버를 실행해보면, DataSourceConfig에서 DriverClass 클래스를 찾지 못해서 에러가 발생  
DataSourceConfig 까지는 잘 실행되는 것을 확인할 수 있음
```shell
Caused by: java.lang.NullPointerException: null
	at java.base/java.lang.Class.forName0(Native Method) ~[na:na]
	at java.base/java.lang.Class.forName(Class.java:375) ~[na:na]
	at com.study.toby.section09.config.autoconfig.DataSourceConfig.dataSource(DataSourceConfig.java:21) ~[main/:na]
```
- DataSourceConfig의 line 21이 `setDriverClass()` 부분

인메모리 DB인 h2 DB 의존성을 추가
```groovy
dependencies {
    // ...
    implementation('org.springframework:spring-jdbc')
    runtimeOnly('com.h2database:h2')
    // ...
}
```
- 코드에서 해당 드라이버 클래스를 사용할 일은 없기 때문에 `implementation`이 아닌 `runtimeOnly`로 설정

data property를 추가해줌
```properties
data.driver-class-name=org.h2.Driver
data.url=jdbc:h2:mem:
data.username=sa
data.password=
```

위와 같이 선언하고 다시 서버를 실행해보면 서버가 잘 뜨는 것을 확인할 수 있음  
다만, 이렇게 서버가 떴다고 DB가 제대로 붙었는지를 확신할 수 없으니 테스트 코드를 만들어서 실제 DB에 연결해보는 작업을 진행해봄
```java
// DataSourceTest.java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = TobyApplication.class)
@TestPropertySource("classpath:/application.properties")
public class DataSourceTest {
    @Autowired
    DataSource dataSource;

    @Test
    void connect() throws SQLException {
        Connection connection = dataSource.getConnection();
        connection.close();
    }
}
```
- Spring Container를 띄우고 그 안에 Bean 구성 정보를 넣고 실제 그 Bean을 가져와서 테스트하는 방식을 이용
  - `@ExtendWith`에 SpringExtension을 넣으면 SpringContext를 이용하는 Spring Container 테스트가 가능함
  - `@ContextConfiguration`에 모든 Bean 구성 정보를 끌어오는 시작점이 되는 클래스 TobyApplication.class를 넣어줌
- 테스트에서 사용할 DataSource를 주입받아야 함
  - `@Autowired`를 사용해서 필드주입을 사용
- DataSource에서 connection을 가져오고, 에러가 발생하지 않으면 `close()`하는 간단한 테스트 진행
  - 하지만 여기까지만 추가하고 테스트해보면 프로퍼티를 세팅하지 않았을 때와 유사한 에러가 발생함
  - `application.properties` 파일을 등록해주는 것이 Spring의 기본 동작 방식이 아니라 Spring Boot의 초기화 과정에서 추가해주는 것이기 때문
  - 그래서 `@TestPropertySource`를 활용해 `application.properties`를 읽어오도록 선언해야함

이제 두 번째 DataSource로 HikariDataSource를 만들어 봄  
Bean 팩토리 메소드 생성 전에 의존성을 먼저 추가해줌
```groovy
// build.gradle
dependencies {
    // ...
    implementation('org.springframework:spring-jdbc')
    runtimeOnly('com.h2database:h2')
    implementation('com.zaxxer:HikariCP')
    // ...
}
```

이제 HikariDataSource Bean 팩토리 메소드를 추가해줌
```java
// DataSourceConfig.java
@Bean
DataSource hikariDataSource(MyDataSourceProperties properties) {
    HikariDataSource dataSource = new HikariDataSource();

    dataSource.setDriverClassName(properties.getDriverClassName());
    dataSource.setJdbcUrl(properties.getUrl());
    dataSource.setUsername(properties.getUsername());
    dataSource.setPassword(properties.getPassword());

    return dataSource;
}
```
- DataSourceProperties는 동일하게 파라미터로 주입받아서 사용
- HikariDataSource는 드라이버 클래스 네임을 문자열로 주면 알아서 클래스 파일을 찾아서 사용하는 방식

이제 두 가지 DataSource Bean 팩토리 메소드가 있으니, 
조건을 걸어서 Hikari 관련 클래스가 존재하면 HikariDataSource Bean이 만들어지고  
HikariDataSource Bean을 찾을 수 없어서 DataSource Bean을 못 만들면 그 때 SimpleDriverDataSource Bean을 사용하도록 구현
```java
// DataSourceConfig.java
@MyAutoConfiguration
@ConditionalMyOnClass("org.springframework.jdbc.core.JdbcOperations")
@EnableMyConfigurationProperties(MyDataSourceProperties.class)
public class DataSourceConfig {
    @Bean
    @ConditionalMyOnClass("com.zaxxer.hikari.HikariDataSource")
    @ConditionalOnMissingBean
    DataSource hikariDataSource(MyDataSourceProperties properties) {
        HikariDataSource dataSource = new HikariDataSource();

        dataSource.setDriverClassName(properties.getDriverClassName());
        dataSource.setJdbcUrl(properties.getUrl());
        dataSource.setUsername(properties.getUsername());
        dataSource.setPassword(properties.getPassword());

        return dataSource;
    }

    @Bean
    @ConditionalOnMissingBean
    DataSource dataSource(MyDataSourceProperties properties) throws ClassNotFoundException {
        SimpleDriverDataSource dataSource = new SimpleDriverDataSource();

        dataSource.setDriverClass((Class<? extends Driver>) Class.forName(properties.getDriverClassName()));
        dataSource.setUrl(properties.getUrl());
        dataSource.setUsername(properties.getUsername());
        dataSource.setPassword(properties.getPassword());

        return dataSource;
    }
}
```
- Bean 메소드가 순서대로 실행이 된다고 가정하면 위에서부터 조건 테스트를 할테니, HikariDataSource Bean 팩토리 메소드를 위로 올림
- HikariDataSourceBean에 해당 클래스가 존재할때 Bean을 생성하도록 `@ConditionalMyOnClass` 추가
  - 그리고 혹시라도 custom Bean을 등록할 수 있도록 `@ConditionalOnMissingBean`을 추가
- SimpleDriverDataSourceBean 팩토리 메소드에도 Hikari Bean이 생성되지 않으면 해당 Bean을 사용하도록 `@ConditionalOnMissingBean`을 추가

이렇게 작성해두고 앞서 작성해둔 DataSourceTest를 돌려보면, HikariPool이라고 표시된 로그를 확인할 수 있음
```shell
08:37:43.724 [Test worker] INFO com.zaxxer.hikari.HikariDataSource -- HikariPool-1 - Starting...
08:37:43.801 [Test worker] INFO com.zaxxer.hikari.pool.HikariPool -- HikariPool-1 - Added connection conn0: url=jdbc:h2:mem: user=SA
08:37:43.802 [Test worker] INFO com.zaxxer.hikari.HikariDataSource -- HikariPool-1 - Start completed.
```

## JdbcTemplate과 트랜잭션 매니저 구성

## Hello 레포지토리

## 레포지토리를 사용하는 HelloService
