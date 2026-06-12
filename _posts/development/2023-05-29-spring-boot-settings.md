---
layout: single
title: "Spring boot 개발환경 세팅"
date: 2023-05-29 17:52:00 +0900
last_modified_at: 2024-01-18 12:14:00 +0900
categories:
  - development
header:
  teaser: /assets/images/2023/05/29/intellij-version.png
---

[이전 게시물](/development/spring-settings/)에서 스프링을 다뤘는데 이제 스프링 부트를 세팅해볼게요.

스프링 부트는 스프링보다 더 자동화되어 있고 편리한 프레임워크입니다. 스프링에서는 별도의 톰캣을 설치해 프로그램을 톰캣에 올려야했지만, 스프링 부트에서는 내장 톰캣이 존재해 바로 실행이 가능해요.

또한, 스프링 부트에서는 서블릿과 jsp 같은 옛날 기술말고 `Thymeleaf` 타임리프를 사용합니다.

## IntelliJ 시작하기

[홈페이지](https://www.jetbrains.com/ko-kr/idea/download/#section=windows)에서 Ultimate 버전을 다운 받아주세요. Community 버전은 웹 개발과 관련된 기능이 덜 지원 되기 때문에 왠만하면 Ultimate 버전을 쓰는게 좋아요. Community 버전으로도 방법이 다 있지만 좋은 기능 쓰는게 좋잖아요?

![intellij-version](/assets/images/2023/05/29/intellij-version.png)

Ultimate는 한달 무료인데 학생 인증 받으면 1년치 라이센스를 줍니다! 다시 인증하면 또 1년 더 주니까 학생 메일 있으시면 Ultimate로 다운받으세요.

### 프로젝트 생성

**FILE - NEW - Project -> Spring Initializr**로 Spring 프로젝트를 생성합니다.

![create-project](/assets/images/2023/05/29/create-project.png)

- Name: Project 이름을 작성합니다.
- Language: 프로그램은 Java로 작성할 거에요.
- Type: Gradle이 Maven 보다 빠르고 코드 가독성도 좋아서 많이 쓰인답니다.
  - Groovy는 java 거의 비슷한 스크립트 언어로 java 프로젝트니까 Groovy로 선택합니다.
  - kotlin으로 하시고 싶으신 분은 취향껏 하시면 돼요.
  - Maven은 pom.xml로 dependency를 관리합니다.
- Group: 만드는 프로그램의 도메인을 거꾸로 적어주세요.
  - naver의 서버라면 com.naver
  - 부산대 서버라면 kr.ac.pusan
  - 프로젝트 폴더 안에 src/main/java 내부로 com/naver로 패키지가 생성됩니다.
  - 나중에 이 패키지 안에서 프로그램을 작성하게 됩니다.
- JDK는

> [start.spring.io](https://start.spring.io/)에서 프로젝트 생성하는 것과 동일하게 되어 있습니다.  
> Community 버전은 여기서 GENERATE하여 압축풀어서 시작하시면 됩니다.

### 프로젝트 실행

- Application.java

  ```java
  @RestController
  @SpringBootApplication
  public class Ch1Application {

      public static void main(String[] args) {
          SpringApplication.run(Ch1Application.class, args);
      }

      @GetMapping("/")
      public String hello() {
          return "Hello, Spring Boot";
      }
  }
  ```

RestController를 붙이고 루트 url에 Get으로 Hello, Spring Boot를 전달합니다.

spring boot는 톰캣이 내장되어 있기 때문에 실행만 하시면 돼요!

실행 후 localhost:8080에 접속하면 출력이 될거에요.

![hello, spring boot](/assets/images/2023/05/29/hello.png)

### jar 파일로 배포

intelliJ의 우측 바에서 gradle(코끼리)를 누르고 Tasks - build -bootJar을 실행합니다.

이제 build 폴더에서 libs에 project명-0.0.1-SNAPSHOT.jar이 생성되었을 겁니다. console에서 jar 파일을 실행하면 서버가 실행되요.

```sh
$ java -jar ch1-0.0.1-SNAPSHOT.jar
```

혹은, 80포트(그냥 http://localhost로 접속 가능)로 하고 싶다면 옵션을 줄 수도 있어요.

```sh
$ java -jar ch1-0.0.1-SNAPSHOT.jar --serer.port=80
```

## 👀참고

[Spring Boot 시작하기 \| 남궁성의 Spring Boot 강좌](https://github.com/castello/springboot_basic)
