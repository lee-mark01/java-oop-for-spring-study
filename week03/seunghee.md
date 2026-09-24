# 4장 - 자바가 확장한 객체지향

4장은 객체지향의 새로운 원리를 배우기보다는,  
3장에서 배운 객체지향 개념을 **Java 문법으로 어떻게 표현하는지** 살펴보는 장이라고 생각한다.

각 키워드는 단순히 문법으로 외우기보다,

> **이 문법이 어떤 설계 의도를 표현하기 위해 존재하는가?**

를 중심으로 정리한다.

---

# 01. abstract 키워드 - 추상 클래스와 추상 메서드

## 책에서 배운 내용

추상 클래스는 `abstract` 키워드가 붙은 클래스이다.

```java
abstract class Animal {
}
```

추상 클래스는 직접 인스턴스를 생성할 수 없다.

```java
new Animal(); // 불가능
```

추상 메서드는 구현부 없이 선언만 존재하는 메서드이다.

```java
abstract void sound();
```

추상 메서드를 하나라도 포함하고 있는 클래스는 반드시 추상 클래스여야 한다.

```java
abstract class Animal {
    abstract void sound();
}
```

추상 클래스를 상속받은 하위 클래스는 추상 메서드를 반드시 구현해야 한다.

```java
class Dog extends Animal {

    @Override
    void sound() {
        System.out.println("멍멍");
    }
}
```

만약 구현하지 않는다면 해당 하위 클래스 역시 추상 클래스가 되어야 한다.

---

## 그래서 왜 사용하는가

추상 클래스는 **공통적인 구조와 구현은 부모가 제공하고, 구현체마다 달라지는 부분만 자식에게 맡기고 싶을 때** 사용할 수 있다.

예를 들어 결제 수단마다 금액 검증은 동일하지만 실제 결제 방식은 다르다고 하자.

```java
abstract class Payment {

    protected int amount;

    public Payment(int amount) {
        this.amount = amount;
    }

    public void validate() {
        if (amount <= 0) {
            throw new IllegalArgumentException();
        }
    }

    public abstract void pay();
}
```

카드 결제는 자신의 방식으로 `pay()`를 구현한다.

```java
class CardPayment extends Payment {

    public CardPayment(int amount) {
        super(amount);
    }

    @Override
    public void pay() {
        System.out.println("카드 결제");
    }
}
```

즉,

```text
공통 상태 / 공통 구현
→ 부모 클래스

구현체마다 달라지는 동작
→ 추상 메서드

실제 구현
→ 자식 클래스
```

로 역할을 나눌 수 있다.

---

## 실무에서는

다음과 같은 경우 추상 클래스를 고려할 수 있다.

- 여러 구현체가 공통 상태를 공유할 때
- 공통 로직을 상당 부분 공유할 때
- 전체 처리 흐름은 같고 일부 단계만 다를 때
- 특정 동작은 반드시 구현하도록 강제하고 싶을 때


예를 들어 파일 처리 흐름이 항상 동일하고 파싱 방식만 다르다고 하자.

```java
abstract class FileProcessor {

    public final void process() {
        open();
        parse();
        save();
        close();
    }

    private void open() {
        System.out.println("파일 열기");
    }

    protected abstract void parse();

    private void save() {
        System.out.println("결과 저장");
    }

    private void close() {
        System.out.println("파일 닫기");
    }
}
```

이 구조처럼 **전체 처리 순서는 부모가 정하고 일부 단계만 자식이 구현하는 방식**은 Template Method Pattern과 연결된다.

---

## Spring에서는

Java 프레임워크에서는 공통 동작을 프레임워크가 제공하고 개발자가 필요한 부분만 구현하도록 추상 클래스를 제공하는 경우가 있다.

대표적으로 `OncePerRequestFilter`를 상속해서 JWT 필터를 구현하는 경우가 있다.

```java
public class JwtFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(...) {
        // JWT 검증 로직
    }
}
```

Filter의 전체 실행 구조는 프레임워크가 관리하고 개발자는 필요한 부분만 구현한다.

---


# 02. 생성자 - 객체가 어떤 상태로 태어날 것인가

## 책에서 배운 내용

생성자는 객체가 생성될 때 호출된다.

```java
class User {

    User() {
    }
}
```

개발자가 생성자를 하나도 만들지 않으면 Java가 기본 생성자를 만들어준다.

```java
class User {
}
```

사실상 다음 생성자가 존재하는 것처럼 사용할 수 있다.

하지만 개발자가 인자가 있는 생성자를 하나라도 직접 선언하면 Java는 기본 생성자를 자동으로 만들어주지 않는다.

```java
class User {

    User(String name) {
    }
}
```

따라서 다음은 불가능하다.

```java
new User();
```

---

## 그래서 왜 사용하는가

생성자는 객체가 생성될 때 반드시 필요한 값을 강제하는 곳이라고 이해하면 된다.

예를 들어 이름이 반드시 필요한 사용자라면,

```java
class User {

    private final String name;

    public User(String name) {
        this.name = name;
    }
}
```

이제 이름 없는 객체를 만들 수 없다.

```java
new User(); // 불가능
```

반드시:

```java
new User("승희");
```

처럼 만들어야 한다.

생성자에서 검증도 가능하다.

```java
public User(String name) {
    if (name == null || name.isBlank()) {
        throw new IllegalArgumentException("이름은 필수입니다.");
    }

    this.name = name;
}
```

즉 객체가 만들어지는 순간부터 정상적인 상태를 유지하도록 할 수 있다.

---

## 실무에서는

생성자는 보통 다음 용도로 중요하다.

```text
필수 값 강제
의존성 전달
초기 상태 설정
잘못된 객체 생성 방지
```

## Spring에서는

Spring에서는 생성자 주입을 매우 자주 사용한다.

```java
@Service
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }
}
```

이 코드에는 다음 의미가 있다.

> `OrderService`는 `OrderRepository` 없이 존재할 수 없다.

따라서 필요한 의존성을 객체 생성 시점에 강제할 수 있다.

Lombok의 `@RequiredArgsConstructor`를 이용해 생성자 코드를 줄이기도 한다.

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
}
```

JPA Entity에서는 프레임워크 요구 때문에 기본 생성자를 만들어두는 경우도 많다.

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member {
}
```


# 03. 인스턴스 초기화 블록과 static 초기화 블록

## 책에서 배운 내용

클래스 내부에는 일반 초기화 블록을 작성할 수 있다.

```java
class User {

    {
        System.out.println("객체 생성");
    }
}
```

이 블록은 객체를 생성할 때마다 실행된다.

```java
new User();
new User();
```

두 번 생성하면 두 번 실행된다.

생성자보다 먼저 실행된다.

```java
class User {

    {
        System.out.println("초기화 블록");
    }

    User() {
        System.out.println("생성자");
    }
}
```

실행 순서는:

```text
초기화 블록
→ 생성자
```

이다.

---

### static 블록

`static` 블록은 클래스가 초기화될 때 한 번 실행된다.

```java
class Config {

    static {
        System.out.println("클래스 초기화");
    }
}
```

객체를 여러 개 생성해도 일반적으로 static 초기화는 클래스 초기화 과정에서 한 번 수행된다.

```text
static block
→ instance block
→ constructor
```

첫 객체 이후에는:

```text
instance block
→ constructor
```

---

## 그래서 왜 사용하는가

인스턴스 초기화 블록은 여러 생성자에서 공통적으로 실행해야 하는 초기화 코드를 한곳에 모을 수 있다.

`static` 블록은 클래스 전체에서 한 번만 준비해야 하는 데이터를 초기화할 때 사용할 수 있다.


## 실무에서는

인스턴스 초기화 블록은 상대적으로 잘 사용하지 않는다

---

## Spring에서는

Spring에서는 복잡한 초기화가 필요하다면 Java의 `static` 블록보다 Bean 생명주기를 이용하는 경우가 많다.

예를 들어 Bean이 생성된 이후 초기 작업이 필요하면 `@PostConstruct` 등을 사용할 수 있다.

```java
@Component
public class CacheLoader {

    @PostConstruct
    public void init() {
        // Bean 생성 후 초기화
    }
}
```

---

# 04. final 키워드 - 더 이상 변경하지 X

## 책에서 배운 내용

`final`은 붙는 위치에 따라 의미가 달라진다.

### final class

```java
final class User {
}
```

상속할 수 없다.

```java
class AdminUser extends User { } // 불가능
```

### final method

```java
class Parent {

    final void execute() {
    }
}
```

자식 클래스에서 오버라이딩할 수 없다.

### final variable

```java
final int count = 10;

count = 20; // 불가능
```

한 번 할당한 값을 다시 할당할 수 없다.

---

## 그래서 왜 사용하는가

`final`의 핵심은:

> **이 부분은 더 이상 변경되면 안 된다는 설계 의도를 코드로 강제하는 것**

이다.

```text
final class
→ 더 이상 상속하지 마

final method
→ 이 동작을 바꾸지 마

final variable
→ 다시 할당하지 마
```


## 실무에서는

의존성이나 객체 생성 시 결정된 값을 이후 변경하지 않도록 할 때 많이 사용한다.

```java
private final OrderRepository orderRepository;
```

상수는 보통:

```java
public static final int MAX_RETRY_COUNT = 3;
```

처럼 `static final`을 함께 사용한다.

---

## Spring에서는

생성자 주입과 함께 가장 자주 보는 형태 중 하나다.

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
}
```

의존성이 생성될 때 결정되고 이후 다른 객체로 바뀌지 않는다는 의도를 표현한다.

---

# 05. instanceof 연산자 - 실제 객체 타입 확인하기

## 책에서 배운 내용

`instanceof`는 객체가 특정 클래스의 인스턴스인지 확인하는 연산자이다.

```java
Animal animal = new Dog();

animal instanceof Dog
```

결과는 `true`이다.

상속 관계도 포함된다.

```java
animal instanceof Animal
```

역시 `true`이다.

인터페이스를 구현한 경우도 확인할 수 있다.

```java
dog instanceof Runnable
```

---

## 그래서 왜 사용하는가

부모 타입이나 인터페이스 타입으로 객체를 다루고 있을 때 실제 객체의 타입을 확인하고싶은 경우 사용할 수 있다.

```java
Animal animal = new Dog();

if (animal instanceof Dog) {
    Dog dog = (Dog) animal;
    dog.bark();
}
```

---

## 실무에서는

안전한 다운캐스팅이나 타입별 처리가 필요한 경우 사용할 수 있다.

하지만 이런 코드가 많아지면 주의할 필요가 있다.

```java
if (payment instanceof CardPayment) {
    ...
} else if (payment instanceof KakaoPayment) {
    ...
} else if (payment instanceof NaverPayment) {
    ...
}
```

새로운 결제 방법이 생길 때마다 조건문을 수정해야 한다.

이 경우 다형성을 이용하면 더 좋은 구조가 될 수도 있다.

```java
interface Payment {
    void pay();
}
```

각 구현체가 자신의 동작을 구현한다.

```java
class CardPayment implements Payment {

    @Override
    public void pay() {
        // 카드 결제
    }
}
```

사용하는 쪽에서는 타입을 확인할 필요가 없다.

```java
payment.pay();
```

---

## Spring에서는

Spring에서도 특정 Bean의 실제 타입을 확인하는 상황이 존재할 수 있지만, 일반적인 비즈니스 코드에서는 특정 구현체 타입을 계속 검사하기보다 인터페이스와 다형성을 이용하는 설계가 더 자연스러운 경우가 많다.

---

# 06. package 키워드 - 클래스가 속할 공간 만들기

## 책에서 배운 내용

`package`는 클래스의 namespace를 만든다.

```java
package com.example.member;
```

같은 이름의 클래스가 존재하더라도 패키지가 다르면 서로 다른 타입이다.

```text
com.example.member.User

com.example.admin.User
```

---

## 그래서 왜 사용하는가

대규모 프로그램에서는 같은 이름의 클래스가 등장할 수 있다.

패키지를 이용하면 이를 구분할 수 있다.

하지만 실무에서 package의 역할은 이름 충돌 방지만 있는 것은 아니다.

> **관련된 코드를 하나의 책임과 기능 단위로 묶는 역할**

도 한다.

---

## 실무에서는

예를 들어 쇼핑몰 프로젝트라면 다음과 같이 나눌 수 있다.

```text
com.example.shop

├─ member
│  ├─ controller
│  ├─ service
│  ├─ repository
│  └─ domain
│
├─ order
│  ├─ controller
│  ├─ service
│  ├─ repository
│  └─ domain
│
└─ payment
```

패키지 구조를 보면 코드의 책임을 어느 정도 파악할 수 있다.

---

### package-private

접근 제어자를 붙이지 않은 클래스나 멤버는 기본적으로 package 단위 접근 범위를 가진다.

즉 패키지는 단순 폴더 구조뿐만 아니라 접근 범위와도 관련이 있다.

---

# 07. interface와 implements - 역할과 계약

## 책에서 배운 내용

인터페이스는 클래스가 구현해야 하는 기능의 규격을 정의한다.

```java
interface Payment {
    void pay();
}
```

구현 클래스는 `implements`를 사용한다.

```java
class CardPayment implements Payment {

    @Override
    public void pay() {
        System.out.println("카드 결제");
    }
}
```

전통적인 인터페이스의 추상 메서드는 다음과 같이 작성해도:

```java
void pay();
```

사실상:

```java
public abstract void pay();
```

와 같은 의미를 가진다.

인터페이스 필드는:

```java
int MAX_COUNT = 10;
```

사실상:

```java
public static final int MAX_COUNT = 10;
```

이다.

---

## 그래서 왜 사용하는가

인터페이스는 구현 방법보다 **역할을 먼저 정의하기 위해 사용한다.**

```java
interface Payment {
    void pay();
}
```

이 인터페이스의 의미는:

> Payment 역할을 하는 객체라면 `pay()` 기능을 제공해야 한다.

이다.

이를 구현하는 객체는 서로 완전히 다른 방식으로 동작할 수 있다.

```text
Payment

├─ CardPayment
├─ KakaoPayment
└─ TossPayment
```

사용하는 쪽에서는 구체적인 구현체보다 `Payment`라는 역할만 알면 된다.

---

## abstract class와 interface 차이

간단하게 보면:

```text
interface
→ 역할 / 계약 중심

abstract class
→ 공통 기반 / 공통 구현 중심
```

인터페이스를 고려할 때:

- 같은 역할을 여러 구현체가 수행해야 할 때
- 구현체를 쉽게 교체하고 싶을 때
- 구체 구현보다 추상화에 의존하고 싶을 때

추상 클래스를 고려할 때:

- 공통 상태가 필요할 때
- 공유할 공통 구현이 많을 때
- 부모 클래스와 자식 클래스의 관계가 자연스러울 때
- 전체 흐름 일부를 자식에게 구현시키고 싶을 때

일반적으로 역할만 필요하다면 먼저 interface를 고려하고, 공통 상태나 구현을 상속할 명확한 이유가 있다면 abstract class를 고려할 수 있다.

---

## Java 8 이후 interface

Java 8 이후 인터페이스는 추상 메서드만 가질 수 있는 것은 아니다.

### default method

```java
interface Payment {

    default void cancel() {
        System.out.println("기본 취소");
    }
}
```

기본 구현을 제공할 수 있다.

기존 인터페이스에 새로운 기능을 추가하면서 기존 구현체를 모두 깨뜨리지 않도록 하는 데 도움이 된다.

### static method

```java
interface Payment {

    static void printInfo() {
        System.out.println("Payment");
    }
}
```

인터페이스 자체에 속하는 정적 메서드도 정의할 수 있다.

Java 9부터는 인터페이스 내부에서 사용하는 `private` 메서드도 가능하다.

---

## 함수형 인터페이스와 Lambda

람다와 인터페이스도 연결된다.

추상 메서드가 하나인 인터페이스를 함수형 인터페이스라고 한다.

```java
@FunctionalInterface
interface Calculator {

    int calculate(int a, int b);
}
```

기존 방식이라면:

```java
Calculator add = new Calculator() {

    @Override
    public int calculate(int a, int b) {
        return a + b;
    }
};
```

람다를 사용하면:

```java
Calculator add = (a, b) -> a + b;
```

처럼 표현할 수 있다.

즉 람다는 함수형 인터페이스의 구현을 간단하게 표현하는 문법이다.

이를 이용하면 로직 자체를 값처럼 전달할 수 있다.

```java
list.stream()
    .filter(user -> user.isActive())
    .map(user -> user.getName())
    .toList();
```

---

## Spring에서는

Spring에서 인터페이스는 매우 자주 사용된다.

예를 들어:

```java
interface PaymentGateway {
    void pay();
}
```

구현체:

```java
@Component
class TossPaymentGateway implements PaymentGateway {

    @Override
    public void pay() {
        // Toss 결제
    }
}
```

서비스에서는 구체 구현체가 아니라 인터페이스에 의존할 수 있다.

```java
@Service
public class OrderService {

    private final PaymentGateway paymentGateway;

    public OrderService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }
}
```

이 구조는 이후 배우게 될 DIP와 Spring DI로 연결된다.

---

