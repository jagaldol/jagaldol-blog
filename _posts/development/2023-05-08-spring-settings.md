---
layout: single
title: "Spring 개발환경 세팅"
date: 2023-05-08 16:48:00 +0900
last_modified_at: 2023-05-29 14:36:00 +0900
categories:
  - development
header:
  teaser: /assets/images/2023/05/08/tomcat-struct.png
---

스프링을 시작하기에 앞서 개발에 필요한 환경 세팅을 해보도록 하겠습니다~ 스프링 부트가 아닌 스프링인 점을 유의해주세요.

## Spring 개발환경

- Java: **[Java11](https://jdk.java.net/archive/)**
- IDE: **[STS 3](https://github.com/spring-attic/toolsuite-distribution/wiki/Spring-Tool-Suite-3), [IntelliJ](https://www.jetbrains.com/ko-kr/idea/download/#section=windows)**
- 웹 서버: **[Tomcat 9](https://tomcat.apache.org/download-90.cgi)**
- 웹 브라우저: **chrome**
- 데이터 베이스: **MySQL**
- 기타: **VS code, Git, AWS, Maven**

**Spring Boot**가 아닌 **Spring**을 사용합니다!

STS 4에서는 Spring Boot만 지원이 되서 STS 3버전으로 설치해야해요. JAVA 11 jdk 설치 후 환경변수 등록하는 건 간단하니까 넘어갈께요.

### VS Code 유용한 확장팩

- **prettier** : 자동 코드 포맷팅
- **indent-rainbow :** 들여쓰기(indent)에 색이 부여되어 구별이 잘 된다!
- **Auto Rename Tag html :** 태그 앞에 꺼 바꾸면, 뒤에 꺼도 자동으로 바뀐다

### Tomcat 기본 사용법

1. [Tomcat 다운로드 페이지](https://tomcat.apache.org/download-10.cgi)에서 zip 파일을 다운 받고 C 드라이브에 압축을 푼다.
2. Tomcat 저장위치(C드라이브에 바로 풀었을 때 - C:\apache-tomcat-10.1.8)에서 bin 폴더로 이동하여 cmd 실행
3. **startup 명령어**로 브라우저에서 localhost:8080에 톰캣의 디폴트 페이지가 나오는지 확인한다.

   ```bash
   C:\apache-tomcat-10.1.8\bin> startup
   ```

4. **shutdown 명령어**로 톰캣을 종료한다.

   ```bash
   C:\apache-tomcat-10.1.8\bin> shutdown
   ```

## Tomcat과 Apache의 차이점

Apache는 웹서버(Web Server)이고 Tomcat은 웹 어플리케이션 서버(WAS; Web Application Server)입니다.

웹서버는 지정된 포트에 접속 시 `html`, `css`, `javascript` 파일을 클라이언트에게 보내주는 역할을 수행합니다.

Tomcat은 Apache의 역할에 더해, JSP와 Servlet을 구동하는 역할도 수행합니다.

![tomcat-struct](/assets/images/2023/05/08/tomcat-struct.png)

tomcat WAS의 구조는 Apache(Web server)를 포함하며 그 위에서 동적인 자료도 효율적으로 처리할 수 있다고 합니다. 자세한건 자세히 공부해봐야 알겠네요 ㅎ

## STS 3(Spring Tool Suite 3) 시작하기

압축해제 후 sts-bundle의 RELEASE 폴더로 가주세요. **STS.exe** 를 실행하시면 IDE가 열립니다!

### 프로젝트 생성

**FILE - NEW - Spring Legacy Project**로 Spring 프로젝트를 생성합니다.

**Spring Legacy Project**가 아닌 **Spring Starter Project**는 Spring Boot를 위한 항목입니다
{: .notice--info}

![create-project](/assets/images/2023/05/08/create-project.png)

**Project name**을 작성하고 Template으로 Spring MVC Project를 골라서 Next를 누릅니다.

> ⚠️Spring MVC Project가 없을 때⚠️  
> **Configure templates**를 눌러서 spring-data-gemfire와 spring-integration을 삭제하고 spring-defaults만 남겨서 `Aplly and Close`합니다.

이제 **Enter a topLevelPackage.**가 나오는데 웹 주소를 거꾸로 치면 됩니다.

- (e.g.) app.mytest.com 이면 **com.mytest.app**을 입력

### tomcat 연결

![navigator](/assets/images/2023/05/08/navigator.png)

왼쪽 아래를 보면 Server가 있는데 여기서 Tomcat를 연결해줍니다. 저는 이미 연결해서 나오지만 아무 연결 없으면 New server 만들기 버튼이 나올거에요!

![new-server](/assets/images/2023/05/08/new-server.png)

설치한 tomcat9를 검색합니다.

![tomcat-setting](/assets/images/2023/05/08/tomcat-setting.png)

- tomcat9 설치된 폴더를 연결해줍니다!

### 프로젝트 실행

프로젝트 명 우클릭 - Run As - Run on Server 실행

tomcat으로 자동으로 실행되어 STS 상에서 http://localhost:8080를 열게 됩니다.

<div class="notice--info" markdown="1">
**STS 상이 아닌 크롬에서 열고 싶을 때**

1. 우측 위 돋보기(검색) 버튼 클릭 - web browser 검색 or  
   Window - Preferences - General - Web Browser 진입
2. Use external web browser 선택
3. External web brwosers - Chrome으로 변경(기본 브라우저가 Chrome으로 되있으면 변경 필요 X)
</div>

### 결과

![result](/assets/images/2023/05/08/result.png)

기본 입력되어 있는 hello world가 출력되었습니다. 컴퓨터 언어가 한글이라 한글이 깨지네요💧

그래도 잘 되는 것까지 확인했으니까 좋았어요!

## 🔗스프링 개념 정리 보러가기

카카오 테크 캠퍼스에서 배운 스프링 개념 정리한 게시글입니다. 스프링에 대해 공부하시는 분들은 한번씩 구경하고 가세요!

- [week5 - 스프링 기초, HTTP 통신](/kakao%20tech%20campus/week5/)
- [week6 - 스프링 MVC, Servlet 등](/kakao%20tech%20campus/week6/)
- [week7 - 스프링(세션, 예외 처리, WEBDataBinder 등)](/kakao%20tech%20campus/week7/)

## 👀참고

[아파치, 톰캣의 차이 \| 유혁의 개발 스토리](https://yoo-hyeok.tistory.com/137)

[Spring 시작하기 \| 남궁성의 Spring framework 강좌](https://github.com/castello/spring_basic)
