---
layout: single
title: "[카테캠 BE] week2 - 자바 기초 복습"
date: 2023-04-25 00:11:00 +0900
categories:
    - kakao tech campus
---

2주차는 시험 기간과 겹쳐 필수 강의가 없었어요. 대신 선택강의를 자유롭게 수강하면 된다고 해서, 저는 기초부터 한번 훑고 넘어 갈려고 기초 강의를 들었어요.

## 자바 프로그래밍

자바는 대표적인 객체 지향 언어로써, OOP를 통해 코드 재사용성, 유지보수 등에 장점이 있다.

자바는 컴파일러를 통해 바이트 코드가 생성되고, 이것이 각 환경의 자바 가상 머신(JVM, Java Virtual Machine)에서 구동된다.

![vm.png](/assets/images/2023-04-25/vm.png)

> 자바는 **이식성**(portablilty)가 뛰어나다!

## 자료형(data type)

- 기본 자료형(primitive data type)
    
    ![intdatatype.png](/assets/images/2023-04-25/intdatatype.png)
    

## Numerical notation

```java
// 10진법
int num = 10;
// 2진법
int bNum = 0B1010;
// 8진법
int oNum = 012;
// 16진법
int xNum = 0XA;
```

- long의 경우 `long lnum = 12345678900L;` 숫자 뒤에 `L` 또는 `l`을 써서 long 형 표시해야한다.

---

### 실수형 - Floating point(부동 소수점 방식)

![realnum.png](/assets/images/2023-04-25/realnum.png)

sign bit(1bit) + exponent(8bit) + mantissa(23bit)로 이루어져있다.

![float.png](/assets/images/2023-04-25/float.png)

> 자바에서 실수의 기본 타입은 `double`을 사용

```java
double dnum = 3.14;
// float의 경우 수 뒤에 F나 f를 붙여 float임을 표
float fnum = 3.14F;
```

### 부동소수점 표기의 문제점

10진법의 소수를 2진법으로 변환한 후 가수부가 제한되기 때문에 오차가 발생할 수 있다.

- 0.1의 2진법 표현  
    $$ 0.1_{(10)} = 0.0001100110011..._{(2)} $$  
    무한 소수로 표현 되기 때문에 제한된 가수부에서 값이 완전히 표현되지 않는다

> `float`를 사용하여 연산하지 말고 **가능한 정수**로 연산을 하자!

---

### 문자형

- 자바는 문자를 나타내기 위해 전세계 표준인 UNICODE를 사용
- utf-16 인코딩을 사용 (모든 문자를 2바이트로 표시)

> `C언어`에서는 1byte(ASKII)를 사용하는 반면, `JAVA` 는 2byte(UNICODE)를 사용한다.

---

### 논리형

- `true`/`false`


## Local variable type inference(Java 10 이상)

```java
var i = 10;
var j = 10.0;
var str = "hello";
```

- 추론 가능한 변수에 대한 자료형을 선언하지 않음
- 지역 변수만 사용 가능
- 한번 선언하여 추론 된 변수는 다른 타입의 값을 대입 할 수 없음
    - 타 스크립트 언어들은 대입 가능
    - 단지, `var` 로 코드 작성할 때 타입 적지 않을 뿐, 변수에 타입이 부여되어 고정된다.

## 상수(constant)

- final 예약어를 사용하여 선언
- 한번 초기화된 이후에는 수정할 수 없다(Named Constant)

```java
final int MAX_NUM = 100;
final int MIN_NUM;

MIN_NUM = 0;
```

## 리터럴(literal)

- 프로그램에서 쓰이는 모든 값들(숫자, 문자, 논리 값)
- 자바에서는 리터럴이 상수 풀 내에 저장된다.

## Type conversion

- Implicit(묵시적, 자동 형 변환)
    - wide한 자료형(바이트 크기가 큰, 더 정밀하게 표현되는)으로 변환은 자동으로 이루어짐
- Explicit(명시적, 강제 형 변환)
    - narrow한 자료형(바이트 크기가 작은, 값이 온전히 표현되지 않는)으로 변환은 명시적으로 표기 해야함

```java
double dNum = 3.14;
int iNum2 = (int)dNum;
```

## 증감연산자 prefix & postfix

- prefix(++i) : 증감 연산 후 statement 수행
- postifx(i++): statement 수행 후 증감 연산

```java
int i = 0;
System.out.println(++i); // 출력: 1
System.out.println(i);   // 출력: 1

int j = 0;
System.out.println(j++); // 출력: 0
System.out.println(j);   // 출력: 1
```

## switch - case 문

```java
switch(medal) {
	case "Gold":
		System.out.println("금메달 입니다.");
		break;
	case "Silver":
		System.out.println("은메달 입니다.");
		break;
	case "Bronze":
		System.out.println("동메달 입니다.");
		break;
	default:
		System.out.println("메달이 없습니다.");
		break;
}
```

## 새로운 Switch Expression(Java 13 이상)

```java
int result = switch (day) {
    case "Monday", "Mon" -> 1;
    case "Tuesday", "Tue" -> 2;
    case "Wednesday", "Wed" -> 3;
    case "Thursday", "Thu" -> 4;
    case "Friday", "Fri" -> 5;
    case "Saturday", "Sat" -> 6;
    case "Sunday", "Sun" -> 7;
    default -> {
        System.out.println("Invalid day of the week: " + day);
        yield -1; // 값 반환
    }
};
```

- 쉼표(,)로 조건 구분
- 기본적으로 값 리턴
    - 조건 내에서 코드 실행을 하고 싶다면 `yield` 키워드로 값 반환이 필수

## STD 입출력

```java
import java.util.Scanner;

public class IOTest {

	public static void main(String[] args) {

		Scanner scanner = new Scanner(System.in);
		input = scanner.nextInt();

		System.out.println(input);
	}
```

## 배열(array)

- 동일한 자료형의 순차적 자료 구조
- 인덱스 연산자[]를 이용하여 빠른 참조가 가능
- 물리적 위치와 논리적 위치가 동일
- 배열의 순서는 0부터 시작

> 자바에서는 객체 배열을 구현한 ArrayList를 많이 활용함

### 배열의 선언

```java
int[] arr1 = new int[10];
int arr2[] = new int[10];

//개수 생략해야 함
int[] numbers = new int[] {10, 20, 30};

// new int[]  생략 가능 
int[] numbers = {10, 20, 30};

// 선언후 배열을 생성하는 경우는 new int[] 생략할 수 없음
int[] ids; 
ids = new int[] {10, 20, 30};
```

### 배열의 길이

```java
int[] arr = new int[10];

System.out.println(arr.length);

// 출력: 10
```

### 배열의 복사

- **Shallow copy(얕은 복사)**
    - 그냥 대입 시 얕은 복사가 이루어짐
- **Deep copy(깊은 복사)**
    - 반복문을 통해 객체 하나씩 전부 복사
    - clonable 인터페이스 구현 등으로 수행

## ArrayList (java.util 패키지)

- 배열의 요소 추가/삭제 등이 용이

![float.png](/assets/images/2023-04-25/method.png)

```java
ArrayList<Book> library = new ArrayList<Book>();
		
library.add(new Book("태백산맥1", "조정래"));
library.add(new Book("태백산맥2", "조정래"));
library.add(new Book("태백산맥3", "조정래"));
library.add(new Book("태백산맥4", "조정래"));
library.add(new Book("태백산맥5", "조정래"));

for(int i =0; i<library.size(); i++) {
	library.get(i).showBookInfo();
}
```

---
## ✏️여담
중간고사 기간이라 쉬운거 공부만 했어요. 다음주 부터 다시 열심히 달려야죠🏃🏻‍♂️

