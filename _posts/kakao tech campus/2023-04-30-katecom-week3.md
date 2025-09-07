---
layout: single
title: "[카테캠 BE] week3 - 자바 객체 지향 핵심(상속, 다형성, 인터페이스)"
date: 2023-04-30 22:10:00 +0900
categories:
  - kakao tech campus
---

3주차의 내용은 상속, 다형성, 인터페이스 등의 핵심적인 객체 지향 내용과 Object 클래스, Class 클래스, String 클래스 등을 배웠습니다.

## 상속(Inheritance)

![inheritance.png](/assets/images/2023/04/30/inheritance.png)

- 화살표 방향 유의

```java
class B extends A{
}
```

> extends 키워드 뒤에는 **단 하나의 클래스**  
> 자바는 **단일 상속(single inheritance)**만 지원

### protected 접근 제어자

외부 클래스는 접근 할 수 없지만, **하위 클래스는 접근 가능**

**(주의)** 상위 클래스에 선언된 private 멤버 변수는 하위 클래스에서 접근 불가
{: .notice--danger}

### 형 변환(업캐스팅)

상속관계에서 하위 클래스는 상위 클래스로 묵시적 변환이 가능하다(역은 성립 X)

- `Customer customerLee = new VIPCustomer();`
- 자바에서는 항상 인스턴스의 메서드가 호출 (가상메서드의 원리)
- 자바의 모든 메서드는 가상 메서드(virtual method)

### 가상 메서드

![virtual.png](/assets/images/2023/04/30/virtual.png)

- 가상 메서드 테이블 기반으로 메서드 주소를 찾아간다

### 오버라이딩(overriding)

상위 클래스에 정의된 메서드를 하위 클래스에서 동일한 이름으로 재정의하는 것

### @overriding 애노테이션(annotation)

- annotation은 주석이라는 의미
- 컴파일러에게 특별한 정보를 제공하는 역할

![annotation.png](/assets/images/2023/04/30/annotation.png)

## 메서드와 인스턴스의 저장 위치

![mem.png](/assets/images/2023/04/30/mem.png)

- 인스턴스는 스택 메모리에 생성
- 인스턴스의 멤버 변수는 힙 메모리에 생성
- 메서드는 프로그램이 로드되며 코드영역에 생성

## 다형성(polymorphism)

하나의 코드가 여러 자료형으로 구현되어 실행되는 것

> 정보은닉, 상속과 더불어 객체지향 프로그래밍의 가장 큰 특징 중 하나

```java
class Animal{

    public void move() {
        System.out.println("동물이 움직입니다.");
    }

    public void eating() {

    }
}

class Human extends Animal{
    public void move() {
        System.out.println("사람이 두발로 걷습니다.");
    }

    public void readBooks() {
        System.out.println("사람이 책을 읽습니다.");
    }
}

class Tiger extends Animal{

    public void move() {
        System.out.println("호랑이가 네 발로 뜁니다.");
    }

    public void hunting() {
        System.out.println("호랑이가 사냥을 합니다.");
    }
}

class Eagle extends Animal{
    public void move() {
        System.out.println("독수리가 하늘을 날아갑니다.");
    }

    public void flying() {
        System.out.println("독수리가 날개를 쭉 펴고 멀리 날아갑니다");
    }
}

public class AnimalTest {

    public static void main(String[] args) {

        Animal hAnimal = new Human();
        Animal tAnimal = new Tiger();
        Animal eAnimal = new Eagle();

        AnimalTest test = new AnimalTest();
        test.moveAnimal(hAnimal);
        test.moveAnimal(tAnimal);
        test.moveAnimal(eAnimal);

        ArrayList<Animal> animalList = new ArrayList<Animal>();
        animalList.add(hAnimal);
        animalList.add(tAnimal);
        animalList.add(eAnimal);

        for(Animal animal : animalList) {
            animal.move();
        }
    }

    public void moveAnimal(Animal animal) {
        animal.move();

    }
}
```

- 각 객체의 특징을 하위 클래스로 정의
- 상위 클래스의 리스트로 한번에 관리가 가능하다.
- 각각의 특징을 if-else구문으로 하나씩 관리하지 않고, 각 기능에 맞게 하위 클래스에서 메서드 재정의를 한다.

## 객체 간의 관계

### IS-A 관계 (is a reationship: inheritance)

일반적인(general) **개념**과 구체적인(specific) **개념** 간의 관계

- 상위 클래스: 일반적인 개념(Animal)
- 하위 클래스: 구체적인 개념(Human, Tiger, Eagle)

> 상위 클래스 수정 시 하위 클래스에 영향을 미칠 수 있다.

### HAS-A 관계(composition)

클래스가 다른 클래스를 **포함**하는 관계

> 단순히 멤버 변수로 다른 클래스를 포함 시킨다.

## 다운 캐스팅(downcasting)

업캐스팅된 클래스를 다시 원래의 타입으로 형 변환(명시적 필요)

```java
Customer vc = new VIPCustomer();              //묵시적
VIPCustomer vCustomer = (VIPCustomer)vc;      //명시적
```

### instanceof

원래 인스턴스의 형이 맞는지 여부를 체크하여 에러를 예방할 수 있다.

```java
if ( animal instanceof Human) {
    Human human = (Human)animal;
    human.readBooks();
}
```

## 추상 클래스(abstract class)

구현 코드 없이 메서드의 선언만 있는 추상 메서드(abstract method)를 포함한 클래스

- `abstract` 예약어를 사용
- 추상 클래스는 `new` 불가(인스턴스화 불가)
- 추상 클래스의 추상 메서드는 하위 클래스가 상속 하여 구현(필수)
  ![notebook.png](/assets/images/2023/04/30/notebook.png)

> `abstract` 표현은 `Italic` 체를 사용하여 표현한다.

### 템플릿 메서드(Template Method) - design pattern

추상 클래스로 선언된 상위 클래스에서 전체적인 흐름을 정의해둔다.

- 상속받은 하위 클래스에서 추상 메서드를 재정의하여 프로그램 제작
- 수정하면 안되는 부분은 final로 선언하여 하위 클래스에서 재정의 할 수 없게 함

> 프레임워크에서 많이 사용되는 설계 패턴

### final 예약어

- final 변수 : 값이 변경될 수 없는 상수
- final 메서드 : 하위 클래스에서 재정의 할 수 없는 메서드
- final 클래스 : 상속할 수 없는 클래스

## 인터페이스(Interface)

모든 메서드가 추상 메서드, 모든 변수가 상수로 선언된 클래스

- `public abstract`/`public static`을 메서드/변수에 붙이지 않아도 자동으로 추상 메서드와 상수로 간주된다.  


![calc.png](/assets/images/2023/04/30/calc.png)

> Interface의 구현은 **점선**으로 표현된다.

### 인터페이스의 의미

클래스나 프로그램이 제공하는 기능을 명시적으로 선언해 놓은 일종의 명세(specification)

- 클라이언트 프로그램은 인터페이스에 선언된 메서드 명세만 보고 이를 구현한 클래스를 사용
- 어떤 객체가 인터페이스 타입 → 그 인터페이스가 제공하는 모든 메서드를 구현했음을 의미
- **예) JDBC 인터페이스**
  - Java와의 커넥션(Interface)이 필요함
  - ORACLE/MS-SQL/MySQL 등에서 인터페이스를 구현한 JAR 파일 제공
  - 사용자(클라이언트)는 단순히 인터페이스 명세를 보고 사용

### 인터페이스와 다형성

- 하나의 인터페이스를 여러 객체가 구현
- 클라이언트 프로그램은 인터페이스의 메서드를 활용하여 여러 객체의 구현을 사용 가능(다형성)
- **예) 인터페이스를 활용한 DAO(Data Access Object)**
  db.properties에 있는 db 정보를 읽어서 interface를 구현한 각 클래스로 생성하여 사용.
  패키지 구조를 유의해서 보자!
  - Structure
  ![hierarchy.png](/assets/images/2023/04/30/hierarchy.png)
  - UserInfoDao.java (interface)
    ```java
    package ch13.domain.userInfo.dao;

    import ch13.domain.userInfo.UserInfo;

    public interface UserInfoDao {
        void insertUserInfo(UserInfo userInfo);
        void updateUserInfo(UserInfo userInfo);
        void deleteUserInfo(UserInfo userInfo);
    }
    ```
  - UserInfoMySqlDao.java
    ```java
    package ch13.domain.userInfo.dao.mysql;

    import ch13.domain.userInfo.UserInfo;
    import ch13.domain.userInfo.dao.UserInfoDao;

    public class UserInfoMySqlDao implements UserInfoDao {
        @Override
        public void insertUserInfo(UserInfo userInfo) {
            System.out.println("Insert into MySQL DB userID = " + userInfo.getUserId());
        }

        @Override
        public void updateUserInfo(UserInfo userInfo) {
            System.out.println("Update into MySQL DB userID = " + userInfo.getUserId());

        }

        @Override
        public void deleteUserInfo(UserInfo userInfo) {
            System.out.println("Delete into MySQL DB userID = " + userInfo.getUserId());
        }
    }
    ```
  - UserInfoOracleDao.java
    ```java
    package ch13.domain.userInfo.dao.oracle;

    import ch13.domain.userInfo.UserInfo;
    import ch13.domain.userInfo.dao.UserInfoDao;

    public class UserInfoOracleDao implements UserInfoDao {
        @Override
        public void insertUserInfo(UserInfo userInfo) {
            System.out.println("Insert into Oracle DB userID = " + userInfo.getUserId());
        }

        @Override
        public void updateUserInfo(UserInfo userInfo) {
            System.out.println("Update into Oracle DB userID = " + userInfo.getUserId());
        }

        @Override
        public void deleteUserInfo(UserInfo userInfo) {
            System.out.println("Delete into Oracle DB userID = " + userInfo.getUserId());
        }
    }
    ```
  - UserInfoClient.java
    ```java
    package ch13.web.userInfo;

    import ch13.domain.userInfo.UserInfo;
    import ch13.domain.userInfo.dao.UserInfoDao;
    import ch13.domain.userInfo.dao.mysql.UserInfoMySqlDao;
    import ch13.domain.userInfo.dao.oracle.UserInfoOracleDao;

    import java.io.FileInputStream;
    import java.io.IOException;
    import java.util.Properties;

    public class UserInfoClient {
        public static void main(String[] args) throws IOException {
            FileInputStream fis = new FileInputStream("db.properties");

            Properties prop = new Properties();
            prop.load(fis);

            String dbType = prop.getProperty("DBTYPE");

            UserInfo userInfo = new UserInfo();
            userInfo.setUserId("12345");
            UserInfoDao userInfoDao = null;

            if (dbType.equals("ORACLE"))
                userInfoDao = new UserInfoOracleDao();
            else if (dbType.equals("MYSQL"))
                userInfoDao = new UserInfoMySqlDao();
            else {
                System.out.println("db error");
                return;
            }
            userInfoDao.insertUserInfo(userInfo);
            userInfoDao.updateUserInfo(userInfo);
            userInfoDao.deleteUserInfo(userInfo);
        }
    }
    ```
  - db.properties
    ```java
    DBTYPE=MYSQL
    ```

### 인터페이스의 추가적인 메서드

기본적으로 전부 추상 메서드이지만 추가적으로 사용가능한 메서드들이 있다

- **디폴트 메서드(자바 8이후)**
  - `default` 키워드를 사용하여 생성
  - 구현을 가지는 메서드
  - 인터페이스를 구현하는 클래스들에서 공통으로 사용할 수 있는 기본 메서드
- **정적 메서드(자바 8이후)**
  - `static` 키워드 사용하여 생성
  - 인스턴스 생성과 상관 없이 인터페이스 타입으로 사용할 수 있는 메서드
- **private 메서드 (자바 9이후)**
  - `private` 키워드 사용하여 생성
  - 인터페이스 내부에서만 사용하기 위해 구현하는 메서드
  - `default` 메서드나 `static` 메서드에서 사용함

### 인터페이스의 다중 상속

인터페이스는 구현 코드가 없으므로 하나의 클래스가 여러 인터페이스는 구현 할 수 있음

```java
public class Customer implements Buy, Sell {
	// class 구현
}
```

- 디폴트 메서드가 중복 되는 경우 반드시 디폴트 메서드를 재정의 필요

### 인터페이스 사이의 상속

- `extends` 키워드로 상속
- 다중 상속 또한 가능하다.

### 클래스 상속과 인터페이스 구현 같이 사용

```java
public class BookShelf extends Shelf implements Queue{
	// class 구현
}
```

## Objcet 클래스 - 모든 클래스의 최상위 클래스

- `java.lang` 패키지 안에 존재
  - 자동 import 됨
  - 많이 사용하는 기본 클래스들이 속한 패키지
  - `String`, `Integer`, `System` 등

### `toString()` 메서드

- 객체의 정보를 `String`으로 변환하기 위해 재정의해서 사용
- 원래는 `클래스명@주소` 로 출력되지만, `toString()` 을 재정의하면 원하는 문자열로 출력가능

### `equals()` 메서드

- 원래는 두 인스턴스의 주소 값을 비교하여 true/false 반환
- 재정의하여 논리적 동일함 여부를 구현 가능

### `hashCode()` 메서드

- 논리적으로 동일함을 위해 `equals()`를 재정의 하였다면 `hashCode()`도 재정의 하여 동일한 `hashCode` 값이 반환되도록 한다.

```java
public boolean equals(Object obj) {
	if( obj instanceof Student) {
		Student std = (Student)obj;
		if(this.studentId == std.studentId )
			return true;
		else return false;
	}
	return false;

}

@Override
public int hashCode() {
	return studentId;
}
```

### clone() 메서드

- 객체의 원본을 복제하는데 사용하는 메서드
- 해당 클래스의 clone() 메서드의 사용을 허용한다는 의미로 `Cloneable` 인터페이스를 명시해 줌

> `clone()`을 사용하면 객체의 정보(멤버 변수 값 등...)가 **동일한 또 다른 인스턴스가 생성**되는 것이므로, **객체 지향 프로그램**에서의 **정보 은닉**, **객체 보호**의 관점에서 **위배**될 수 있다

```java
public class Student implements Cloneable{

    .......

	@Override
	public Student clone() throws CloneNotSupportedException {
		// TODO Auto-generated method stub
		return super.clone();
	}
}
```

## String

- `String str1 = new String("abc");`
  - 힙 메모리에 인스턴스로 생성
  - 생성자로 같은 값으로 생성 시, 생성 될 때마다 다른 주소를 가짐
    - 인스턴스 비교 시 다르다(false) 나옴
- `String str2 = "abc";`
  - “abc”가 constant pool(상수 풀)에 존재
  - “abc”로 생성한 객체는 같은 주소를 가짐
    - 인스턴스 비교 시 같다(true) 나옴
- 한번 생성된 String은 불변(immutable) - final 키워드로 문자열이 보관
  - String.concat()시 두 문자열이 이어지는 것이 아닌 이어진 새로운 문자열이 생성
  - 대신 `StringBuilder`, `StringBuffer` 사용하면 메모리 낭비를 줄일 수 있다.

### StringBuilder / StringBuffer

문자열을 여러번 연결하거나 변경할 때 사용

```java
StringBuilder buffer = new StringBuilder("java");
buffer.append(" android");
System.out.println(buffer.toString());
```

- 내부적으로 가변적인 `char[]`를 멤버 변수로 가짐
  - 새로운 인스턴스를 생성하지 않고 `char[]`를 변경
- `StringBuffer`는 멀티 쓰레드 프로그래밍에서 동기화(`synchronization`)을 보장
  - 단일 쓰레드 프로그램에서는 `StringBuilder` 사용을 권장

### text block(java 13이상)

- 문자열을 """ """ 사이에 이어서 생성
- html, json 문자열을 만드는데 유용하게 사용

```java
"""
<html>

    <body>
        <span>example text</span>
    </body>
</html>"""
```

- 첫 줄 indent 기준으로 작성

## Class 클래스

컴파일된 class 파일을 로드하여 객체를 동적 로드하고, 정보를 가져오는 메소드를 제공

> **(참고)**자바의 모든 클래스와 인터페이스는 컴파일 후 class 파일로 생성된다.

- 동적 로딩
  - 클래스 명(문자열)으로 가져오기
  ```java
  Class c = Class.forName("java.lang.String");
  ```
  - 클래스 이름으로 직접 가져오기
  ```java
  Class c = String.class;
  ```
  - 생성된 인스턴스에서 Class 클래스 가져오기
  ```java
  String s = new String();
  Class c = s.getClass();  //Object 메서드
  ```
- reflection 프로그래밍
  - Class 클래스를 사용하여 클래스의 정보(생성자, 변수, 메서드) 등을 파악하여 인스턴스를 생성하고, 메서드를 호출하는 방식의 프로그래밍
  - 로컬 메모리에 객체 없는 경우, 원격 프로그래밍, 객체의 타입을 알 수 없는 경우에 사용
  - java.lang.reflect 패키지에 있는 클래스를 활용하여 프로그래밍
  - 일반적으로 자료형을 알고 있는 경우엔 사용하지 않음
  - `newInstance()`로 인스턴스 생성
    ```java
    Class c = Class.forName("java.lang.String");
    String str = (String)c.newInstance();

    Class[] parameterTypes = {String.class};
    Constructor cons = c.getConstructor(parameterTypes);

    Object[] initargs = {"hello world!"};
    String str2 = (String)cons.newInstance(initargs);

    System.out.println(str2);
    ```

---

## ✏️여담

자바 기억이 새록새록 나면서 문법에 완전 친숙해졌어요. 옛날에 했다고 다 까먹었나 했더니 금방 다 생각나네요. 빨리 스프링도 배우고 싶어요!🚗
