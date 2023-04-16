---
layout: single
title: "[카테캠 BE] week1 - 자바 객체지향 프로그래밍"
date: 2023-04-16 2:47:00 +0900
categories:
    - kakao tech campus
---

4월 10일부터 첫주차 교육이 시작됐어요. 

지금은 1단계 웹 개발 기초 교육 과정으로 총 10주간 자바와 `Spring`, `MySQL`을 학습한다고 하더라고요.

* 1주차: Java (객체 지향 입문)
* 2주차: Java (객체 지향 핵심)
* 3주차: Java 기능 (함수, 스트림 등)
* 4~6주차: Spring (개요, 원리 실습 등)
* 7~8주차: MySQL (개요, 최적화 등)
* 9주차: 보충학습기간

도중에 중간고사 기간 때문에 한주 쉬어서 총 10주라고 해요.

2학년때 자바 수업 이후로는 자바를 크게 사용한 적이 없어서 걱정했는데 생각보다 쉬웠어요! 학과 알고리즘 수업은 `C++`을 사용했고 프로젝트의 백엔드 기술로는 `Node.js`의 `Express`를 사용했어서 자바와 안 친했거든요. 한국은 자바공화국이라고 하니 이참에 자바와 친해져봐야죠.🥹

1단계 트랙에서는 매주 노션으로 학습 내용 정리를 해야하는데 블로그에도 올릴려고요! 그럼 배운 내용 정리해볼께요~

---

## 객체지향 프로그래밍

![oop.png](/assets/images/2023-04-17/oop.png)

- 기존 절차 지향 프로그래밍과는 다른 패러다임.
- 프로그램을 객체 간의 소통, 협력을 통하여 구현한다.

> **절차 지향 프로그래밍이란?**  
> 사건의 흐름에 따른 프로그래밍. 일반적인 프로그래밍에서 위에서부터 한 줄 씩 실행되는 프로그램이다.  
>  
> 객체 지향은 절차 지향 위에서 객체들을 정의하고 생성하여 객체들을 상호작용 시켜 프로그램을 구성한다.


## 클래스

객체의 청사진(blueprint)

```java
public class Student {
	// 멤버 변수들
	int studentId;
	String studentName;
	int majorCode;
	int grade;
}
```

- 클래스는 대문자로 시작하는 것이 좋다.
- public 클래스는 .java 파일과 이름이 동일하며 하나만 존재 가능.  
    public 외에는 여러개 같이 존재 가능
- 객체를 위한 클래스 설계 시 다음을 고려하여 만든다.
    - 속성: 멤버변수
    - 기능: 메서드(method)

## 함수와 메서드

- 자바에서 함수는 static 키워드 사용
- 메서드는 static 키워드 없는 멤버 함수들.
- 함수는 static 키워드를 통해 객체의 생성 유무와 상관없이, 프로그램이 빌드 될때 생성됨
- 함수가 호출되면 지역 변수들은 스택 메모리에 계속 저장된다.
    - 따라서 재귀적으로 호출되었을 때, 스택오버플로우 발생 주의

## 객체의 생성(인스턴스)

```java
Student studentLee = new Student();
```

자바에서 만들어진 객체 하나하나를 인스턴스라고 부른다.

`new` 키워드를 통해 생성한 인스턴스는 참조변수에 저장되고, 이는 인스턴스의 주소를 저장하고 있다. 인스턴스는 `Heap memory` 에 저장된다.

## 생성자

```java
public class Student {
	public Student() {
		System.out.println("생성자입니다!");
	}
}
```

생성자는 명시적으로 생성하지 않아도 default 생성자가 존재한다.

```java
// default 생성자
public UserInfo(){}
```

생성자는 `new` 키워드를 통해 인스턴스가 생성되며 자동 호출된다.

### 생성자 오버로딩

```java
public class UserInfo {
	public UserInfo(){
		System.out.println("생성자입니다!");
	}
	
	public UserInfo(String userId) {
		System.out.println("2번째 생성자입니다!");
		System.out.println(userId + "가 파라미터로 들어왔습니다!");
	}
}
```

생성자를 오버로딩하여 다양하게 생성할 수 있다.

## 접근 제어 지시자(access modifier)

클래스 외부에서 클래스의 멤버 변수, 메서드, 생성자를 사용할 수 있는지 여부를 지정하는 키워드

- private : 같은 클래스 내부에서만 접근 가능(외부 클래스, 상속 관계의 클래스에서도 접근 불가)
- 아무것도 없음(default) : 같은 패키지 내부에서만 접근 가능(상속 관계라도 패키지가 다르면 접근 불가)
- protected : 같은 패키지나 상속관계의 클래스에서 접근 가능하고 그 외 외부에서는 접근 할 수 없음
- public : 클래스의 외부 어디서나 접근 할 수 있음

## 캡슐화(encapsualtion)

필요한 정보와 기능만 외부에 오픈하여 일관된 인터페이스를 제공한다.(정보은닉)

### 정보 은닉

멤버 변수에 직접 접근시키지 않고 getter와 setter를 통해 접근 방법을 정해준다.

### get()/set() 메서드

- getter/setter라고 부른다.
- private로 선언된 멤버 변수에 대해 접근, 수정 할 수 있는 메서드를 public으로 제공

## this 키워드

인스턴스 자신의 메모리를 가리킨다.

![this1.png](/assets/images/2023-04-17/this1.png)

```java
public void setYear(int year)
{
    this.year = year;
}
```

### 다른 생성자를 호출하는 this

```java
public class Person {

	String name;
	int age;
	
	public Person() {
		this("이름없음", 1);
	}
	
	public Person(String name, int age) {
		this.name = name;
		this.age = age;
	}
}
```

## static 변수

여러 인스턴스가 공유하는(공통으로 사용하는) 변수

![static.png](/assets/images/2023-04-17/static.png)

- 인스턴스가 생성될 때 만들어지는 변수가 아닌, 처음 프로그램이 메모리에 로딩될 때 메모리를 할당
- 클래스 변수, 정적변수라고도 함(vs. 인스턴스 변수)
- 인스턴스 생성과 상관 없이 사용 가능하므로 클래스 이름으로 직접 참조

```java
static int serialNum;
```

![mem.png](/assets/images/2023-04-17/mem.png)

- static 변수: 데이터 영역
- 참조 변수: 스택 메모리
- 인스턴스: 힙 메모리

## static 메서드

- 인스턴스 생성과 상관 없이 사용 가능(클래스 이름으로 직접 참조)
- static 메서드에서는 인스턴스 변수(일반 멤버 변수)를 사용할 수 없다.  
    인스턴스 생성 전에도 생성되어있는 static 변수만 사용가능

```java
private static int serialNum = 1000;

public static int getSerialNum() {
	return serialNum;
}
```

📋**참고**
![variable.png](/assets/images/2023-04-17/variable.png)

## 싱글톤 패턴
<div style="display:flex;">
    <p>
        <img src="/assets/images/2023-04-17/singleton.png">
    </p>
    <ul>
        <li>인스턴스가 단 한 개만 생성되어야 하는 경우 사용하는 디자인 패턴</li>
        <li>static 변수, 메서드를 활용하여 구현 할 수 있음</li>
    </ul>
</div>


```java
public class Singleton {
    private static Singleton instance = new Singleton();

    private Singleton() {

    }
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

---
## ✏️여담
다른 언어들에서 객체지향 개념은 끝도 없이 배웠어서 그런지 단순 자바 문법만 익숙해지면 될거 같네요. 알고리즘 문제라도 자바로 좀 풀어봐야겠어요.

일단 다음 주는 중간고사 기간이니까 잠시 **🔥시험🔥**에 집중해야돼요. 할일이 태산..🥲🥲

