# Section 02. Spring Boot 시작하기

## 개발환경 준비

SpringBoot 2.7.6 기준

JDK 8, 11, 17 설치 (실습은 많이 사용하는 11로 설치)  
하나의 하드웨어에서 여러 JDK 버전을 사용해야 한다면 [SDKman], [jabba] 와 같은 툴 사용 추천  
실습은 SDKman을 활용하여 실습  
```bash
# SDKman 다운로드
$ curl -s "https://get.sdkman.io" | bash

# 다운로드 가능한 모든 자바 버전 및 Distribution 확인
$ sdk list java

# amazon-coretto 11.0.17 다운로드
$ sdk install java 11.0.17-amzn

# Java 버전 확인
$ java --version

# 현재 디렉토리에서 지정한 자바 버전 사용
$ sdk use java 17.0.5-amzn

# 지정한 자바 버전으로 변경되었는지 확인
$ java --version
```

SpringBoot CLI를 통해 Spring도 설치 가능  
SpringBoot 역시 SDKman으로 설치 가능
```bash
# SpringBoot 2.7.6 설치
$ sdk install springboot 2.7.6

# 설치된 spring boot 실행
$ spring
```

## 프로젝트 생성

[Spring Initializr] 활용하거나 IntelliJ IDEA 활용하여 생성  
2024/06/10 현재는 2.7.6 버전은 위 방법을 통해서 생성할 수 없음  

아래 명령어들은 SDKman을 활용하여 Spring을 설치하는 방법이나, 위와 동일하게 현재 2.7.6은 생성할 수 없음
```bash
# spring 설치 도움
$ spring shell

# init 명령어 이후 tab으로 option들 확인후 사용
$ init -b 2.7.6 -d web -g toby -j 11 -n helloboot -x helloboot

# spring shell exit
$ exit

# 생성된 springboot 프로젝트로 이동
$ cd helloboot
```

> 강의에서는 2.7.6을 사용했지만 현재 해당버전으로 생성이 불가능하여 java, springboot 버전 모두를 변경하여 진행  
> java 17.0.11 coretto, spring boot 3.3.0  
> 진행하면서 버전이 달라 오류가 발생하는 부분은 찾아서 수정하고 진행

결론적으로, 프로젝트 생성은 spring boot 3.3.0, java 17을 사용하고  
gralde-groovy, dependencies는 spring-web 하나만 추가하여 생성

## Hello 컨트롤러

```java
// HelloController.java 생성 후 작성
@RestController
public class HelloController {
    
    @GetMapping("/hello")
    public String hello(String name) {
        return "Hello " + name;
    }
}
```
- `@RestController` - HTML을 통째로 return하는 대신에 API 요청에 대한 응답을 특정한 type으로 인코딩해서 보낼때 사용하는 어노테이션
- `@GetMapping` - web의 요청에 HTTP 메소드가 GET으로 되어있는 것만 받도록 설정하는 어노테이션
- `(String name) - paramater` - URL의 `?` 뒤에 쿼리스트링으로 날아오는 것

spring boot run 이후 브라우저에서 `localhost:8080/hello?name=Spring` 으로 접근하여 확인

## Hello API 테스트

HTTP API 요청을 만들고 응답을 확인하는데 사용되는 여러 도구들  
- 크롬 브라우저 개발자 모드
- Terminal Command line에서 curl 명령 활용
- [Httpie]
- http request in IntelliJ IDEA
- Postman API Platform
- JUnit Test

이번 실습에서는 Httpie를 활용하여 진행

```bash
# httpie 설치
$ brew install httpie

# http 요청
$ http -v ":8080/hello?name=Spring"
```

http의 -v 옵션은 요청과 응답의 상세내용을 보여주고, localhost는 생략가능

## HTTP 요청과 응답

Section01에서 진행한 웹 애플리케이션이 동작하는 구조 설명 요약
- 웹 클라이언트가 웹 요청을 웹 컨테이너에 전달
- 웹 컨테이너는 전달받은 요청을 처리할 웹 컴포넌트를 찾아서 요청을 위임
- 웹 컴포넌트는 요청을 분석하고 필요한 작업을 수행
- 수행 이후 만들어낸 결과를 다시 웹 응답으로 웹 클라이언트에게 응답

중요한 부분은 웹 클라이언트와 웹 컨테이너 사이에 요청과 응답이 항상 쌍을 맺어서 존재한다는 것

### HTTP

프로토콜, 요청과 응답에 대해서 어떤식으로 주고받을지 정의되어 있음

Request
- Request Line: Method, Path, HTTP Version
- Headers
- Message Body

```bash
# 위 http command의 Request 상세내용
GET /hello?name=Spring HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Connection: keep-alive
Host: localhost:8080
User-Agent: HTTPie/3.2.2
```

Response
- Status Line: HTTP Version, Status Code, Status Text
- Headers
- Message Body

```bash
# 위 http command의 Response 상세내용
HTTP/1.1 200 
Connection: keep-alive
Content-Length: 12
Content-Type: text/plain;charset=UTF-8
Date: Mon, 10 Jun 2024 08:58:10 GMT
Keep-Alive: timeout=60

Hello Spring
```

<!-- 링크 -->
[SDKman]: https://sdkman.io/
[jabba]: https://github.com/shyiko/jabba
[Spring Initializr]: https://start.spring.io/
[Httpie]: https://httpie.io/