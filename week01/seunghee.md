# Week 02 - 자바와 절차적 / 구조적 프로그래밍

## 1. Java 프로그램 실행 과정

Java 프로그램의 실행 구조를 이해하려면 먼저 JDK, JRE, JVM의 역할을 구분할 필요가 있다.

### JDK

JDK는 `Java Development Kit`의 약자로, Java 프로그램을 **개발하기 위한 도구 모음**이다.

대표적으로 Java 소스 코드를 바이트코드로 변환하는 `javac` 컴파일러가 포함된다.

```text
Main.java
   ↓ javac
Main.class
```

---

### JRE

JRE는 `Java Runtime Environment`의 약자로, Java 프로그램을 **실행하기 위한 환경**이다.

개념적으로는 다음과 같이 이해할 수 있다.

```text
JRE
= JVM
+ Java 실행에 필요한 라이브러리와 환경
```

프로그램을 개발하는 것이 아니라 이미 만들어진 Java 프로그램을 실행하기만 한다면 개발 도구인 JDK까지 필요하지 않을 수 있기 때문에 구분이 된다.

---

### JVM

JVM은 `Java Virtual Machine`의 약자로, Java 바이트코드를 실제로 실행하는 가상 머신이다.

```text
Java Source Code
     ↓
   javac
     ↓
Bytecode (.class)
     ↓
    JVM
     ↓
  OS / CPU
```

이 구조 덕분에 Java는 같은 바이트코드를 여러 운영체제에서 실행할 수 있다.

운영체제마다 JVM 구현은 다르지만 JVM이 동일한 바이트코드를 각 환경에 맞게 실행해주기 때문이다.

---

## 2. 구조적 프로그래밍

프로그램의 규모가 커질수록 모든 코드를 하나의 긴 흐름으로 작성하면 이해하고 관리하기 어려워진다.

구조적 프로그래밍에서는 프로그램을 의미 있는 기능 단위로 나누어 복잡성을 줄인다.

대표적인 방법이 메서드를 사용하는 것이다.

예를 들어 같은 계산이 반복된다면,

```java
int result1 = 10 * 2 + 3;
int result2 = 20 * 2 + 3;
int result3 = 30 * 2 + 3;
```

이를 하나의 메서드로 분리할 수 있다.

```java
static int calculate(int value) {
    return value * 2 + 3;
}
```

이렇게 하면 다음과 같은 장점이 있다.

- 중복 코드를 줄일 수 있다.
- 하나의 기능을 의미 있는 단위로 분리할 수 있다.
- 코드를 읽는 사람이 기능을 더 쉽게 파악할 수 있다.
- 지역변수를 사용해 변수의 영향 범위를 제한할 수 있다.

그렇다면 메서드를 호출했을 때 Java 메모리에서는 실제로 어떤 일이 일어날까?

이를 설명하기 위해 책에서는 T 메모리 구조를 사용한다.

---

## 3. T 메모리 구조

책에서는 Java 프로그램의 메모리 구조를 이해하기 쉽게 다음과 같은 형태로 표현한다.

```text
┌───────────────────────┐
│        Static         │
├───────────┬───────────┤
│   Stack   │   Heap    │
└───────────┴───────────┘
```

크게 보면 각 영역은 다음 역할을 가진다.

```text
Static
→ 클래스 수준의 정보와 공유되는 데이터 저장

Stack
→ 메서드 호출과 지역변수 저장

Heap
→ 객체 저장
```


2장에서는 특히 메서드 호출과 함께 Stack 영역이 중요하게 다뤄진다.

---

## 4. Stack과 메서드 실행

메서드가 호출되면 해당 메서드를 실행하기 위한 **Stack Frame**이 생성된다.

예를 들어 다음 코드가 있다고 하자.

```java
public static void main(String[] args) {
    int a = 10;
    foo(a);
}

static void foo(int x) {
    int b = 20;
}
```

`foo()`가 실행 중이라면 Stack은 개념적으로 다음과 같이 구성될 수 있다.

```text
Stack

┌─────────────────┐
│ foo Stack Frame │
│ x = 10          │
│ b = 20          │
├─────────────────┤
│ main Stack Frame│
│ a = 10          │
└─────────────────┘
```

`main()`이 먼저 호출되었기 때문에 `main`의 Stack Frame이 만들어지고, 그 안에 지역변수 `a`가 존재한다.

이후 `foo()`를 호출하면 새로운 Stack Frame이 그 위에 생성된다.

`foo()`의 Stack Frame에는 매개변수 `x`와 지역변수 `b`가 존재한다.

`foo()`가 종료되면 해당 Stack Frame은 제거된다.

```text
foo() 종료

┌─────────────────┐
│ main Stack Frame│
│ a = 10          │
└─────────────────┘
```

따라서 `foo()` 안에 있던 `x`와 `b`도 함께 생명주기가 끝난다.

이후 `main()`까지 종료되면 `main`의 Stack Frame도 제거된다.

---

## 5. 지역변수의 특징

### Scope

지역변수는 선언된 범위 안에서만 사용할 수 있다.

```java
static void test() {
    int a = 10;

    if (a > 5) {
        int b = 20;
        System.out.println(b);
    }
}
```

여기서 `b`는 `if` 블록 안에서는 사용할 수 있지만 블록 밖에서는 사용할 수 없다.

```java
System.out.println(b); // 사용 불가
```

이처럼 변수를 사용할 수 있는 범위를 `Scope`라고 한다.

변수의 Scope를 좁게 유지하면 해당 값이 어디에서 사용되고 변경되는지 파악하기 쉬워진다.

---

### 지역변수의 초기화

Java에서는 지역변수를 선언한 뒤 값을 넣지 않은 상태로 사용할 수 없다.

```java
int i;

System.out.println(i);
```

이 경우 다음과 같은 컴파일 오류가 발생할 수 있다.

```text
The local variable i may not have been initialized
```

따라서 지역변수는 사용하기 전에 명확하게 초기화해야 한다.

---

## 6. 메서드 간 데이터 전달

각 메서드는 자신의 Stack Frame을 가지므로 다른 메서드의 지역변수를 직접 사용할 수 없다.

```java
static void mainMethod() {
    int a = 10;
    foo();
}

static void foo() {
    // a를 직접 사용할 수 없음
}
```

`foo()`에서 `a`의 값이 필요하다면 매개변수로 전달해야 한다.

```java
static void mainMethod() {
    int a = 10;
    foo(a);
}

static void foo(int x) {
    System.out.println(x);
}
```

즉, 각 메서드는 자신에게 필요한 데이터를 매개변수로 전달받고, 필요한 경우 반환값을 통해 다시 데이터를 돌려준다.

---

### Call by Value

Java에서 메서드에 인자를 전달할 때는 값을 복사해서 전달한다.

이를 `Call by Value`라고 한다.

```java
static void change(int x) {
    x = 100;
}

public static void main(String[] args) {
    int a = 10;

    change(a);

    System.out.println(a);
}
```

결과는 다음과 같다.

```text
10
```

`change(a)`를 호출했을 때 `a`라는 변수 자체가 전달되는 것이 아니라 `a`가 가지고 있던 값 `10`이 복사되어 `x`에 전달된다.

```text
main Stack Frame

a = 10
   │
   │ 값 복사
   ▼

change Stack Frame

x = 10
```

따라서 `change()` 안에서 `x`를 변경해도 `main()`의 `a`는 바뀌지 않는다.

```text
main
a = 10

change
x = 100
```

---

## 7. 데이터의 주인과 생명주기

지금까지는 Stack과 지역변수를 중심으로 살펴봤다.

하지만 Stack 외부에도 다양한 변수들이 존재합니다.

변수를 이해할 때는 **이 데이터의 주인이 누구인가**를 기준으로 생각하면 이해하기 쉽다.

```text
Local Variable
→ 메서드의 데이터

Instance Variable
→ 객체의 데이터

Static Variable
→ 클래스가 공유하는 데이터
```

---

### Local Variable

지역변수는 메서드가 실행되는 동안 필요한 임시 데이터이다.

```java
void study() {
    int hours = 3;
}
```

메서드가 종료되면 해당 Stack Frame과 함께 생명주기가 끝난다.

---

### Instance Variable

Instance 변수는 특정 객체 하나가 가지고 있는 상태이다.

```java
class Student {
    String name;
}
```

객체를 여러 개 생성하면 각각 다른 `name` 값을 가질 수 있다.

```java
Student a = new Student();
a.name = "철수";

Student b = new Student();
b.name = "영희";
```

Heap에는 서로 다른 객체가 존재한다.

```text
Heap

Student 객체 A
name = "철수"

Student 객체 B
name = "영희"
```

즉, Instance 변수는 객체와 함께 존재한다.

---

### Static Variable

Static 변수는 특정 객체 하나가 아니라 클래스 전체에서 공유하는 데이터이다.

```java
class Student {
    static int totalStudents;
}
```

---

## 8. Heap과 Garbage Collector

객체는 Heap 영역에 생성된다.

```java
Student student = new Student();
```

`new Student()`를 실행하면 `Student` 객체가 Heap에 생성된다.

지역변수 `student`는 객체 자체를 저장하는 것이 아니라 Heap에 존재하는 객체를 가리키는 reference 값을 가진다.

```text
Stack

student
   │
   │ reference
   ▼

Heap

Student 객체
```

객체는 Stack Frame처럼 메서드가 종료된다고 바로 사라지는 것이 아니다.

객체가 더 이상 프로그램에서 접근할 수 없는 상태가 되면 Garbage Collector의 메모리 회수 대상이 될 수 있다.

정리하면, 데이터마다 생명주기가 다르다.

```text
Local Variable
→ 메서드 실행과 함께 존재

Instance Variable
→ 객체와 함께 존재

Static Variable
→ 클래스 수준에서 오래 유지
```

---

## 9. 공유 상태는 왜 조심해야 하는가

여러 메서드가 하나의 공유 변수를 자유롭게 읽고 수정하면 값의 변화 과정을 추적하기 어려워질 수 있다.

```java
static int count;
```

여러 메서드에서 `count`를 수정한다고 생각해보자.

```java
methodA();
methodB();
methodC();
```

어느 순간 `count`가 예상하지 못한 값을 가지고 있다면 어떤 메서드가 값을 변경했는지 확인해야 한다.

반대로 지역변수라면 영향을 미치는 범위가 제한된다.

```java
void methodA() {
    int count = 0;
}
```

이 `count`에 문제가 발생했다면 우선 `methodA()` 내부를 살펴보면 된다.

따라서 필요한 범위보다 넓게 상태를 공유하기보다는 변수의 Scope를 가능한 한 좁게 유지하는 것이 코드를 이해하고 유지보수하는 데 유리하다.

이 문제는 실행 흐름이 여러 개가 되는 멀티스레드 환경에서 더 중요해진다.

---

## 10. 멀티스레드

하나의 프로세스 안에서 여러 Thread가 실행될 수 있다.

각 Thread는 Stack 내부에서 자신만의 Stack을 가진다.
따라서 서로 다른 Thread의 지역변수는 기본적으로 독립적이다.

반면 Heap과 Static 영역는 여러 Thread가 공유한다.
그래서 여러 Thread가 같은 공유 상태를 동시에 읽고 수정하면 문제가 발생할 수 있다.

```java
static int count = 0;
```

Thread 1과 Thread 2가 동시에 `count`를 읽고 수정한다면 실행 순서에 따라 결과가 달라질 수 있다.

이처럼 여러 실행 흐름이 동일한 공유 상태에 동시에 접근하면서 실행 순서에 따라 결과가 달라질 수 있는 상황을 **Race Condition**이라고 한다.

공유 자원에 대한 접근을 제어하기 위해 Lock과 같은 동기화 방법을 사용할 수 있다.

다만 Lock을 사용하는 구간에서는 여러 Thread의 동시 접근이 제한된다.

```text
Lock 사용
≠ 멀티스레드의 장점을 모두 포기

Lock 사용
= 공유 자원에 접근하는 필요한 구간의 동시 실행을 제한
```

따라서 공유 데이터의 안전성과 동시성을 함께 고려해야 한다.

---

## 멀티 프로세스
멀티 스레드와는 달리, 멀티 프로세스는 각 Process가 독립적인 메모리 공간을 가진다.

```text
Process A
└─ Memory A

Process B
└─ Memory B
```

서로의 메모리 공간이 분리되어 있기 때문에 격리성이 높다.

대신 프로세스마다 독립적인 메모리 공간을 사용하기 때문에 상대적으로 더 많은 메모리가 필요할 수 있다.
