# Section 01. Spring Boot 살펴보기

## Spring Boot 소개

**Spring을 기반으로** 실무 환경에 사용 가능한 수준의 **독립 실행형 애플리케이션**을   
복잡한 고민 없이 빠르게 작성할 수 있게 도와주는 여러가지 도구의 모음

Spring 자체를 확장하고 있는 프레임워크, 혹은 라이브러리

> Spring != Spring Boot

* Spring Boot의 핵심 목표
  * 매우 빠르고 광범위한 영역의 스프링 개발 경험을 제공
  * 강한 주장을 가지고 즉시 적용 가능한 기술 조합을 제공하면서,  
    필요에 따라 원하는 방식으로 손쉽게 변형 가능
  * 프로젝트에서 필요로 하는 다양한 비기능적인 기술  
    (내장형 서버, 보안, 메트릭, 상태 체크, 외부 설정 방식 등)
  * 코드 생성이나 XML 설정을 필요로 하지 않음

## Spring Boot의 역사

Spring Web Application은 컨테이너 안에 배포하고 동작하는 방식  
문제는 이걸 위해 알아야할 기본지식이 너무 많음

[대두된 Github Issue](https://github.com/spring-projects/spring-framework/issues/14521)  
2012년 Containerless한 Spring을 원하는 사용자

2013년 Spring 를 수정하지 않고 Spring Boot 라는 새로운 프로젝트를 개발

이후 계속되는 버전업

## Containerless

Serverless와 유사  
서버에 대한 설치, 관리 등을 개발자가 신경쓰지 않고 애플리케이션을 배포하고 운영하는 것

Servlet Container가 동작하지만, Spring Boot를 통해서 설정을 신경쓰지 않고 개발을 시작할 수 있음  
Servlet Container를 띄우는 작업조차도 알아서 해줌

### Container

Spring은 IoC 컨테이너

* Web Component
  * Dynamic Content를 제공하기 위한 Component
  * Java에서는 **Servlet**
* Web Client
  * Web Component와 Request, Response를 주고받음
* Web Container
  * 여러 개의 Web Component의 라이프 사이클 등을 관리하는 개체
  * Servlet을 담은 Container는 **Servlet Container**
  * 독립적인 환경이여서 Tomcat 등을 설치하고 배포하는 과정이 필요함
    * 신경쓸게 정말 많음!!
    * Tomcat이 아닌 다른 환경을 사용한다면 그에 맞는 설정을 또 학습해야함
* Spring Container
  * Servlet Container의 뒤에 위치
  * 여러 개의 Bean을 가지고 있음


## Opinionated

## Spring Boot의 이해
