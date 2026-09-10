# Week 01 - 자바와 절차적 / 구조적 프로그래밍

## 1. 구조적 프로그래밍은 왜 사용하는가

구조적 프로그래밍의 목적은 프로그램을 더 이해하기 쉽고 관리하기 쉽게 만드는 것이다.

특히 메서드를 사용하면 반복되는 코드를 하나의 기능 단위로 묶을 수 있다.

예를 들어 같은 계산이 여러 번 반복된다면,

```java
int result1 = 10 * 2 + 3;
int result2 = 20 * 2 + 3;
int result3 = 30 * 2 + 3;
```

다음과 같이 메서드로 분리할 수 있다.

```java
static int calculate(int value) {
    return value * 2 + 3;
}
```

이렇게 하면 코드의 중복을 줄일 수 있고, 코드가 어떤 기능을 수행하는지도 더 쉽게 파악할 수 있다.

또한 메서드 내부에서 지역변수를 사용하면 변수의 사용 범위를 제한할 수 있다.

지역변수는 필요한 범위 안에서만 존재하기 때문에 프로그램 전체에서 공유되는 변수보다 값의 변화 과정을 추적하기 쉽다.

---

## 2. T 메모리 구조

이 책에서는 Java 프로그램이 실행될 때 사용하는 메모리를 이해하기 쉽게 다음과 같은 **T 메모리 구조**로 설명한다.

```text
┌───────────────────────┐
│      Static 영역       │
├───────────┬───────────┤
│   Stack   │   Heap    │
│    영역    │    영역    │
└───────────┴───────────┘
```

크게 보면 각 영역은 다음과 같은 역할을 가진다.

```text
Static
→ 클래스 수준의 정보와 공유되는 데이터

Stack
→ 메서드 호출과 지역변수

Heap
→ 객체
```

각 영역에 존재하는 데이터는 역할과 생명주기가 서로 다르다.

---

## 3. Static 영역

Static 영역은 클래스 수준의 정보가 관리되는 영역이라고 볼 수 있다.

특히 `static`으로 선언된 필드는 특정 객체 하나에 속하는 것이 아니라 클래스 자체에 속한다.

```java
class Student {
    static int totalStudents;
}
```

`totalStudents`는 `Student` 객체마다 각각 존재하는 값이 아니라 클래스 전체에서 하나를 공유한다.

```java
Student.totalStudents = 10;
```

따라서 `static` 데이터는 특정 메서드 호출이 끝났다고 해서 사라지지 않고 비교적 긴 생명주기를 가진다.

---

## 4. Stack 영역과 Stack Frame

Stack 영역은 메서드가 호출될 때 필요한 정보가 저장되는 공간이다.

메서드가 호출되면 해당 메서드를 위한 **Stack Frame**이 생성된다.

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

`foo()`가 실행되고 있는 시점의 Stack은 개념적으로 다음과 같이 생각할 수 있다.

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

`foo()`가 종료되면 `foo`의 Stack Frame은 사라진다.

따라서 그 안에 존재하던 매개변수 `x`와 지역변수 `b`도 함께 사라진다.

```text
foo() 종료

┌─────────────────┐
│ main Stack Frame│
│ a = 10          │
└─────────────────┘
```

`main()`까지 종료되면 `main`의 Stack Frame도 사라진다.

즉, 지역변수는 기본적으로 **메서드 호출과 함께 존재하고 메서드가 종료되면 생명주기가 끝난다.**

---

## 5. 지역변수와 Scope

지역변수는 자신이 선언된 범위 안에서만 사용할 수 있다.

```java
static void test() {
    int a = 10;

    if (a > 5) {
        int b = 20;
        System.out.println(b);
    }
}
```

위 코드에서 `b`는 `if` 블록 내부에서만 사용할 수 있다.

```java
System.out.println(b); // 사용 불가
```

이렇게 변수를 사용할 수 있는 범위를 **Scope**라고 한다.

변수의 Scope를 좁게 유지하면 그 변수가 어디에서 사용되고 어디에서 변경되는지 파악하기 쉬워진다.

---

## 6. 메서드 Stack Frame은 서로 독립적이다

각 메서드는 자신의 Stack Frame을 가진다.

따라서 다른 메서드의 지역변수를 직접 사용할 수는 없다.

```java
static void mainMethod() {
    int a = 10;
    foo();
}

static void foo() {
    // a를 직접 사용할 수 없음
}
```

`foo()`에서 `a`의 값이 필요하다면 매개변수를 통해 전달해야 한다.

```java
static void mainMethod() {
    int a = 10;
    foo(a);
}

static void foo(int x) {
    System.out.println(x);
}
```

이처럼 각 메서드는 자신의 지역변수를 독립적으로 관리하고, 필요한 데이터는 매개변수와 반환값을 통해 주고받는다.

---

## 7. Call by Value

Java는 메서드에 값을 전달할 때 **값을 복사해서 전달한다.**

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

`change(a)`를 호출할 때 `a`라는 변수 자체가 전달되는 것이 아니라, `a`가 가지고 있던 값 `10`이 복사되어 `x`에 전달된다.

```text
main Stack Frame

a = 10
   │
   │ 값 복사
   ▼

change Stack Frame

x = 10
```

따라서 `change()` 안에서 `x`를 변경해도 `main()`의 `a`에는 영향을 주지 않는다.

```text
main
a = 10

change
x = 100
```

이 개념은 이후 객체를 메서드의 인자로 전달할 때도 중요하게 사용된다.

---

## 8. Heap 영역

Heap 영역은 객체가 생성되는 공간이다.

```java
Student student = new Student();
```

`new Student()`를 실행하면 `Student` 객체가 Heap에 생성된다.

지역변수 `student`는 객체 자체가 아니라 Heap에 존재하는 객체를 가리키는 reference 값을 가진다.

```text
Stack

student
   │
   │ reference
   ▼

Heap

Student 객체
```

객체가 더 이상 프로그램에서 접근할 수 없는 상태가 되면 Garbage Collector의 메모리 회수 대상이 될 수 있다.

---

## 9. 변수의 역할과 생명주기

Java에서 변수는 어디에 속하는지에 따라 역할과 생명주기가 달라진다.

### Static 변수

클래스 전체에서 공유하는 상태이다.

```java
class Student {
    static int totalStudents;
}
```

`totalStudents`는 특정 학생 한 명의 값이 아니라 `Student`라는 클래스 전체에서 공유하는 값이다.

---

### Instance 변수

객체 하나가 가지고 있는 상태이다.

```java
class Student {
    String name;
}
```

객체를 여러 개 만들면 각각 별도의 `name` 값을 가질 수 있다.

```java
Student a = new Student();
a.name = "철수";

Student b = new Student();
b.name = "영희";
```

개념적으로 Heap에는 각각 다른 객체가 존재한다.

```text
Heap

Student 객체 A
name = "철수"

Student 객체 B
name = "영희"
```

---

### Local 변수

메서드가 실행되는 동안 필요한 임시 데이터이다.

```java
void study() {
    int hours = 3;
}
```

`hours`는 학생 객체의 상태라기보다 `study()`라는 메서드가 실행될 때 잠시 사용하는 값이다.

세 종류를 정리하면 다음과 같다.

```text
static 변수
→ 클래스 전체가 공유하는 상태

instance 변수
→ 객체 하나가 가지는 상태

local 변수
→ 메서드 실행 중 필요한 임시 상태
```

결국 중요한 것은 **이 데이터의 주인이 누구인가**이다.

---

## 10. 지역변수는 초기화한 뒤 사용해야 한다

Java에서는 지역변수를 선언만 하고 값을 넣지 않은 상태로 사용하려고 하면 컴파일 오류가 발생한다.

```java
int i;

System.out.println(i);
```

다음과 같은 메시지를 볼 수 있다.

```text
The local variable i may not have been initialized
```

따라서 지역변수는 값을 사용하기 전에 명확하게 초기화해야 한다.

---

## 11. 공유 변수는 왜 조심해서 사용해야 하는가

여러 메서드가 같은 공유 변수를 자유롭게 읽고 수정하면 현재 값이 어떻게 만들어졌는지 추적하기 어려워질 수 있다.

```java
static int count;
```

여러 메서드에서 `count`를 수정한다고 생각해보자.

```java
methodA();
methodB();
methodC();
```

어느 순간 `count`가 예상하지 못한 값을 가지고 있다면 어떤 메서드가 값을 변경했는지 프로그램의 여러 부분을 확인해야 할 수 있다.

반대로 지역변수는 영향을 미치는 범위가 제한적이다.

```java
void methodA() {
    int count = 0;
}
```

이 `count`에 문제가 발생했다면 우선 `methodA()` 내부만 살펴보면 된다.

따라서 필요 이상으로 공유되는 상태를 만드는 것보다 변수의 Scope를 가능한 한 좁게 유지하는 것이 코드의 이해와 유지보수에 유리하다.

---

## 12. 멀티스레드와 공유 변수

하나의 프로세스 안에서 여러 개의 Thread가 실행될 수 있다.

여러 Thread는 Heap과 클래스 수준의 데이터 등을 공유하지만, 각 Thread는 자신만의 Stack을 가진다.

```text
Process

┌───────────────────────────┐
│       Shared Memory       │
│      Static / Heap        │
├─────────────┬─────────────┤
│  Thread 1   │  Thread 2   │
│  Stack 1    │  Stack 2    │
└─────────────┴─────────────┘
```

각 Thread가 자신의 Stack에 존재하는 지역변수만 사용한다면 서로 영향을 줄 가능성이 적다.

하지만 여러 Thread가 같은 공유 변수를 동시에 읽고 수정하면 문제가 발생할 수 있다.

```java
static int count = 0;
```

예를 들어 Thread 1과 Thread 2가 동시에 `count`를 읽고 수정한다면 실행 순서에 따라 결과가 달라질 수 있다.

이처럼 여러 실행 흐름이 동일한 공유 상태에 동시에 접근하면서 실행 순서에 따라 결과가 달라질 수 있는 상황을 **Race Condition**이라고 한다.

이러한 문제를 막기 위해 Lock과 같은 동기화 방법을 사용할 수 있다.

다만 Lock이 걸려 있는 구간에서는 여러 Thread가 동시에 실행되지 못하기 때문에 데이터의 안전성과 동시성을 함께 고려해야 한다.

```text
Lock 사용
≠ 멀티스레드 사용 포기

Lock 사용
= 필요한 구간의 동시 접근을 제한
```

---

## 13. 멀티프로세스와 멀티스레드

### 멀티프로세스

각 Process는 독립된 메모리 공간을 가진다.

```text
Process A
└─ Memory A

Process B
└─ Memory B
```

서로의 메모리 공간이 분리되어 있기 때문에 격리성이 높다.

대신 각각 독립적인 메모리를 사용하기 때문에 상대적으로 더 많은 메모리가 필요할 수 있다.

---

### 멀티스레드

여러 Thread가 하나의 Process 안에서 실행된다.

```text
Process

Shared
- Heap
- 클래스 관련 영역

Thread 1
└─ Stack 1

Thread 2
└─ Stack 2
```

일부 메모리를 공유하기 때문에 효율적으로 여러 실행 흐름을 만들 수 있다.

대신 공유 상태에 여러 Thread가 동시에 접근하는 경우 동시성 문제를 고려해야 한다.

---
