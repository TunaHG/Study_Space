# 토비의 스프링

* [인프런 강의]
* [교보문고 도서]
* [강의자료 Repository]

## 목차

### [Section 01]. 스프링 부트 살펴보기

* 스프링 부트 소개
* 스프링 부트의 역사
* Containerless
* Opinionated
* 스프링 부트의 이해

### [Section 02]. 스프링 부트 시작하기

* 개발환경 준비
* 프로젝트 생성
* Hello 컨트롤러
* Hello API 테스트
* HTTP 요청과 응답

### [Section 03]. 독립 실행형 서블릿 애플리케이션

* Containerless 개발 준비
* 서블릿 컨테이너 띄우기
* 서블릿 등록
* 서블릿 요청 처리
* 프론트 컨트롤러
* 프론트 컨트롤러로 전환
* Hello 컨트롤러 매핑과 바인딩

### [Section 04]. 독립 실행형 스프링 애플리케이션

* 스프링 컨테이너 사용
* 의존 오브젝트 추가
* Dependency Injection
* 의존 오브젝트 DI 적용
* DispatcherServlet으로 전환
* Annotation 매핑 정보 사용
* 스프링 컨테이너로 통합
* 자바코드 구성 정보 사용
* @Component 스캔
* Bean의 생명주기 메소드
* SpringBootApplication

### [Section 05]. DI와 테스트, 디자인 패턴

* 테스트 코드를 이용한 테스트
* DI와 단위 테스트
* DI를 이용한 Decorator, Proxy 패턴

### [Section 06]. 자동 구성 기반 애플리케이션

* 메타 어노테이션과 합성 어노테이션
* 합성 어노테이션의 적용
* 빈 오브젝트의 역할과 구분
* 인프라 빈 구성 정보의 분리
* 동적인 자동 구성 정보 등록
* 자동 구성 정보 파일 분리
* 자동 구성 어노테이션 적용
* @Configuration과 proxyBeanMethods

### [Section 07]. 조건부 자동 구성

* 스타터와 Jetty 서버 구성 추가
* @Conditional과 Condition
* @Conditional 학습테스트
* 커스텀 @Conditional
* 자동 구성 정보 대체하기
* 스프링 부트의 @Conditional

### [Section 08]. 외부 설정을 이용한 자동 구성

* Environment 추상화와 프로퍼티
* 자동 구성에 Envirionment 프로퍼티 적용
* @Value와 PropertySourcesPlaceholderConfigurer
* 프로퍼티 클래스의 분리
* 프로퍼티 빈의 후처리기 도입

### [Section 09]. Spring JDBC 자동 구성 개발

* 자동 구성 클래스와 빈 설계
* DataSource 자동 구성 클래스
* JdbcTemplate과 트랜잭션 매니저 구성
* Hello 레포지토리
* 레포지토리를 사용하는 HelloService


### [Section 10]. Spring Boot 자세히 살펴보기

* 스프링 부트의 자동 구성과 테스트로 전환
* 스프링 부트 자세히 살펴보기
* 자동 구성 분석 방법
* 자동 구성 조건 결과 확인
* Core 자동 구성 살펴보기
* Web 자동 구성 살펴보기
* Jdbc 자동 구성 살펴보기
* 정리

### [Section 11]. 업데이트

* 스프링 부트 3.0으로 예제 업그레이드

<!-- 링크 모음 -->
[인프런 강의]: https://www.inflearn.com/course/%ED%86%A0%EB%B9%84-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-%EC%9D%B4%ED%95%B4%EC%99%80%EC%9B%90%EB%A6%AC/dashboard
[교보문고 도서]: https://product.kyobobook.co.kr/detail/S000000935360
[강의자료 Repository]: https://github.com/tobyspringboot/helloboot
[Section 01]: ./Section01.md
[Section 02]: ./Section02.md
[Section 03]: ./Section03.md
[Section 04]: ./Section04.md
[Section 05]: ./Section05.md
[Section 06]: ./Section06.md
[Section 07]: ./Section07.md
[Section 08]: ./Section08.md
[Section 09]: ./Section09.md
[Section 10]: ./Section10.md
[Section 11]: ./Section11.md