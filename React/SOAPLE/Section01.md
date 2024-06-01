# Section 01. 리액트 소개

## 리액트는 무엇인가?

A JavaScript library for building user interfaces

* library
  * 자주 사용되는 기능들을 정리해 모아 놓은 것
* user interface, UI
  * 사용자와 컴퓨터 프로그램간의 상호작용을 위해 중간에서 입력과 출력을 제어해주는 것

* 웹 사이트를 구축할때 사용하는 대표적인 JS 라이브러리
  * Angular JS
  * React JS
  * Vue JS

**프레임워크 vs 라이브러리**  
프레임워크는 프로그램의 흐름에 대한 제어 권한이 개발자가 아닌 프레임워크 자신에게 있음  
라이브러리는 제어 권한이 개발자에게 있음

## 리액트의 장점과 단점

**장점**  
* 빠른 업데이트 & 렌더링 속도
  * Virtual DOM (Document Object Model)
    * 하나의 웹사이트에 대한 정보를 모두 담아두고 있는 큰 그릇
  * 업데이트해야 할 최소한의 부분만을 탐색해서 업데이트
* Component-Based
  * Component - 구성요소
  * 리액트는 여러 Component로 모여있고, 각 Component는 또 다른 Component들로 조합될 수 있음
* 재사용성 Reusability
  * 해당 소프트웨어 또는 모듈이 다른 곳에서도 쉽게 사용될 수 있음
  * 재사용성이 높아지면 개발 기간이 단축되고, 유지 보수가 용이함
* Meta에서 꾸준히 버전 업데이트를 진행중
* 활발한 지식 공유 및 커뮤니티
  * 스택 오버플로우

**단점**  
* 방대한 학습량
* 높은 상태관리 복잡도
  * 외부 상태 관리 라이브러리를 사용하는 경우가 많음
