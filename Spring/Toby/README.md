# 토비의 스프링

* [인프런 강의](https://www.inflearn.com/course/%ED%86%A0%EB%B9%84-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-%EC%9D%B4%ED%95%B4%EC%99%80%EC%9B%90%EB%A6%AC/dashboard)
* [교보문고 도서](https://product.kyobobook.co.kr/detail/S000000935360)
* [강의자료 Repository](https://github.com/tobyspringboot/helloboot)

## 목차

### [Section 01](Section01.md). 스프링 부트 살펴보기

* [스프링 부트 소개](Section01.md/#spring-boot-소개)
* [스프링 부트의 역사](Section01.md/#spring-boot의-역사)
* [Containerless](Section01.md/#containerless)
* [Opinionated](Section01.md/#opinionated)
* [스프링 부트의 이해](Section01.md/#spring-boot의-이해)

### [Section 02](Section02.md). 스프링 부트 시작하기

* [개발환경 준비](Section02.md/#개발환경-준비)
* [프로젝트 생성](Section02.md/#프로젝트-생성)
* [Hello 컨트롤러](Section02.md/#hello-컨트롤러)
* [Hello API 테스트](Section02.md/#hello-api-테스트)
* [HTTP 요청과 응답](Section02.md/#http-요청과-응답)

### [Section 03](Section03.md). 독립 실행형 서블릿 애플리케이션

* [Containerless 개발 준비](Section03.md/#containerless-개발-준비)
* [서블릿 컨테이너 띄우기](Section03.md/#서블릿-컨테이너-띄우기)
* [서블릿 등록](Section03.md/#서블릿-등록)
* [서블릿 요청 처리](Section03.md/#서블릿-요청-처리)
* [프론트 컨트롤러](Section03.md/#프론트-컨트롤러)
* [프론트 컨트롤러로 전환](Section03.md/#프론트-컨트롤러로-전환)
* [Hello 컨트롤러 매핑과 바인딩](Section03.md/#hello-컨트롤러-매핑과-바인딩)

### [Section 04](Section04.md). 독립 실행형 스프링 애플리케이션

* [스프링 컨테이너 사용](Section04.md/#스프링-컨테이너-사용)
* [의존 오브젝트 추가](Section04.md/#의존-오브젝트-추가)
* [Dependency Injection](Section04.md/#dependency-injection)
* [의존 오브젝트 DI 적용](Section04.md/#의존-오브젝트-di-적용)
* [DispatcherServlet으로 전환](Section04.md/#dispatcherservlet으로-전환)
* [Annotation 매핑 정보 사용](Section04.md/#annotation-매핑-정보-사용)
* [스프링 컨테이너로 통합](Section04.md/#스프링-컨테이너로-통합)
* [자바코드 구성 정보 사용](Section04.md/#자바코드-구성-정보-사용)
* [@Component 스캔](Section04.md/#component-스캔)
* [Bean의 생명주기 메소드](Section04.md/#bean의-생명주기-메소드)
* [SpringBootApplication](Section04.md/#springbootapplication)

### [Section 05](Section05.md). DI와 테스트, 디자인 패턴

* [테스트 코드를 이용한 테스트](Section05.md/#테스트-코드를-이용한-테스트)
* [DI와 단위 테스트](Section05.md/#di와-단위-테스트)
* [DI를 이용한 Decorator, Proxy 패턴](Section05.md/#di를-이용한-decorator-proxy-패턴)

### [Section 06](Section06.md). 자동 구성 기반 애플리케이션

* [메타 어노테이션과 합성 어노테이션](Section06.md/#메타-어노테이션과-합성-어노테이션)
* [합성 어노테이션의 적용](Section06.md/#합성-어노테이션의-적용)
* [빈 오브젝트의 역할과 구분](Section06.md/#빈-오브젝트의-역할과-구분)
* [인프라 빈 구성 정보의 분리](Section06.md/#인프라-빈-구성-정보의-분리)
* [동적인 자동 구성 정보 등록](Section06.md/#동적인-자동-구성-정보-등록)
* [자동 구성 정보 파일 분리](Section06.md/#자동-구성-정보-파일-분리)
* [자동 구성 어노테이션 적용](Section06.md/#자동-구성-어노테이션-적용)
* [@Configuration과 proxyBeanMethods](Section06.md/#configuration과-proxybeanmethods)

### [Section 07](Section07.md). 조건부 자동 구성

* [스타터와 Jetty 서버 구성 추가](Section07.md/#스타터와-jetty-서버-구성-추가)
* [@Conditional과 Condition](Section07.md/#conditional과-condition)
* [@Conditional 학습테스트](Section07.md/#conditional-학습테스트)
* [커스텀 @Conditional](Section07.md/#커스텀-conditional)
* [자동 구성 정보 대체하기](Section07.md/#자동-구성-정보-대체하기)
* [스프링 부트의 @Conditional](Section07.md/#스프링-부트의-conditional)

### [Section 08](Section08.md). 외부 설정을 이용한 자동 구성

* [Environment 추상화와 프로퍼티](Section08.md/#environment-추상화와-프로퍼티)
* [자동 구성에 Envirionment 프로퍼티 적용](Section08.md/#자동-구성에-envirionment-프로퍼티-적용)
* [@Value와 PropertySourcesPlaceholderConfigurer](Section08.md/#value와-propertysourcesplaceholderconfigurer)
* [프로퍼티 클래스의 분리](Section08.md/#프로퍼티-클래스의-분리)
* [프로퍼티 빈의 후처리기 도입](Section08.md/#프로퍼티-빈의-후처리기-도입)

### [Section 09](Section09.md). Spring JDBC 자동 구성 개발

* [자동 구성 클래스와 빈 설계](Section09.md/#자동-구성-클래스와-빈-설계)
* [DataSource 자동 구성 클래스](Section09.md/#datasource-자동-구성-클래스)
* [JdbcTemplate과 트랜잭션 매니저 구성](Section09.md/#jdbctemplate과-트랜잭션-매니저-구성)
* [Hello 레포지토리](Section09.md/#hello-레포지토리)
* [레포지토리를 사용하는 HelloService](Section09.md/#레포지토리를-사용하는-helloservice)


### [Section 10](Section10.md). Spring Boot 자세히 살펴보기

* [스프링 부트의 자동 구성과 테스트로 전환](Section10.md/#스프링-부트의-자동-구성과-테스트로-전환)
* [스프링 부트 자세히 살펴보기](Section10.md/#스프링-부트-자세히-살펴보기)
* [자동 구성 분석 방법](Section10.md/#자동-구성-분석-방법)
* [자동 구성 조건 결과 확인](Section10.md/#자동-구성-조건-결과-확인)
* [Core 자동 구성 살펴보기](Section10.md/#core-자동-구성-살펴보기)
* [Web 자동 구성 살펴보기](Section10.md/#web-자동-구성-살펴보기)
* [Jdbc 자동 구성 살펴보기](Section10.md/#jdbc-자동-구성-살펴보기)
* [정리](Section10.md/#정리)

### [Section 11](Section11.md). 업데이트

* [스프링 부트 3.0으로 예제 업그레이드](Section11.md/#스프링-부트-30으로-예제-업그레이드)