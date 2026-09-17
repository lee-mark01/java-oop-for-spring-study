# 3장 — 자바는 객체지향을 어떻게 구현하는가

《객체지향의 사실과 오해》에서는 객체지향의 핵심을 객체들이 **역할과 책임을 가지고 협력하는 것**이라고 설명했다. 그렇다면 이런 객체를 자바 코드에서는 어떻게 표현할 수 있을까?

예를 들어 빵 주문 서비스를 만든다고 해보자. 고객이 빵을 주문하려면 상품은 자신의 재고를 확인하고, 결제는 잔액 차감 처리하며, 주문은 주문 결과를 기억해야 한다. 이렇게 각자의 책임을 가진 객체들이 협력해 하나의 기능을 완성하는 것이 객체지향의 기본적인 모습이다.

## 현실의 대상을 코드로 옮기기

먼저 주문에 필요한 상품을 코드로 표현해야 한다. 하지만 현실의 빵은 이름, 가격, 냄새, 모양, 재료, 온도 등 수많은 특징을 가지고 있다. 이 모든 특징이 주문 시스템에 필요한 것은 아니다.

주문 시스템에서 중요한 정보가 상품명, 가격, 재고라면 다음과 같이 표현할 수 있다.

```java
public class Product {
    private String name;
    private int price;
    private int stock;
}
```

현실의 수많은 특징 중 현재 문제에 필요한 특징만 선택한 것이다. 이것이 **추상화**다.

추상화의 결과로 만들어진 `Product`는 상품이라는 대상을 분류한 **클래스**다. 실제 판매할 상품을 만들려면 이 클래스를 이용해 객체를 생성해야 한다.

```java
Product bread = new Product();
Product cake = new Product();
```

여기서 `Product`는 클래스이고, `new Product()`로 만들어진 각각의 상품은 객체다. `bread`와 `cake`는 객체 자체가 아니라 생성된 객체를 가리키는 참조변수다.

즉, 클래스와 객체의 관계는 다음과 같다.

> 클래스는 객체들의 공통 상태와 행동을 정의한 타입이고, 객체는 실행 중 실제 상태를 가지고 책임을 수행하는 개별 존재다.

객체를 만들었다면 단순히 데이터를 보관하는 데서 끝나서는 안 된다. 상품이 가격과 재고를 알고 있다면, 그 정보를 사용하는 행동도 상품이 맡는 것이 자연스럽다.

```java
public class Product {
    private String name;
    private int price;
    private int stock;

    public int calculatePrice(int quantity) {
        return price * quantity;
    }

    public void decreaseStock(int quantity) {
        if (stock < quantity) {
            throw new IllegalStateException("재고 부족");
        }

        stock -= quantity;
    }
}
```

가격과 재고 정보를 가진 Product 객체가 금액 계산과 재고 차감을 담당하도록 설계할 수 있다.

이제 주문 서비스는 상품의 재고를 꺼내 직접 계산하지 않고 상품에게 메시지를 보낼 수 있다.

```java
product.decreaseStock(quantity);
```

`Product`는 요청받은 일을 자신의 정보와 규칙을 이용해 처리한다. 이것이 객체가 자신의 책임을 수행하는 모습이다.

## 공통점을 발견하면 상속으로 확장할 수 있다

상품을 모델링한 것처럼 동물도 공통된 상태와 행동을 가진 클래스로 표현할 수 있다.

```java
public class Animal {
    private String name;

    public void showName() {
        System.out.println("동물의 이름");
    }
}
```

그런데 동물 중에는 펭귄처럼 더 구체적인 종류도 있다. 펭귄은 동물의 공통 특징을 가지면서 헤엄치는 행동을 추가로 가진다.

```java
public class Penguin extends Animal {

    public void swim() {
        System.out.println("헤엄친다.");
    }
}
```

이처럼 상위 타입의 공통 상태와 행동을 바탕으로 더 구체적인 하위 타입을 만드는 것이 **상속**이다.

상속 관계는 다음 문장이 자연스러워야 한다.

> 펭귄은 동물의 한 종류다.
> `Penguin is a kind of Animal.`


## 상속 관계가 있으면 다형성을 사용할 수 있다

`Penguin`은 `Animal`의 하위 타입이므로 펭귄 객체를 `Animal` 타입의 변수에 담을 수 있다.

```java
Animal animal = new Penguin();
```

이 문장은 `Penguin` 객체를 `Animal`의 한 종류로 다루겠다는 뜻이다.

`Penguin`이 `Animal`의 `showName()`을 아래와 같이 오버라이드 해서 재정의할 수 있습니다. 

```java
public class Penguin extends Animal {

    @Override
    public void showName() {
        System.out.println("펭귄의 이름");
    }
}
```

그러면 다음 코드의 실행 결과는 실제 객체에 따라 달라진다.

```java
Penguin pororo = new Penguin();
Animal loopy = new Penguin();
Animal crong = new Animal();

pororo.showName(); // Penguin.showName()
loopy.showName();  // Penguin.showName()
crong.showName();  // Animal.showName()
```

`loopy`의 참조 타입은 `Animal`이지만 실제로 생성된 객체는 `Penguin`이다. 따라서 `Penguin`에서 재정의한 메서드가 실행된다.

자바는 컴파일할 때 참조 타입을 기준으로 호출 가능한 메서드인지 확인하고, 실행할 때 실제 객체를 기준으로 오버라이딩된 메서드를 선택한다. 이를 **동적 바인딩**이라고 한다.

같은 `showName()`을 호출했지만 객체마다 다르게 응답할 수 있다. 이것이 **다형성**이다.

다만 참조 타입이 알지 못하는 메서드는 바로 호출할 수 없다.

```java
Animal animal = new Penguin();

animal.showName(); // 가능
animal.swim();     // 컴파일 오류
```

실제 객체가 `Penguin`이어도 `Animal` 타입에는 `swim()`이 선언되어 있지 않기 때문이다.

### 오버라이딩과 오버로딩

다형성을 이해할 때 오버라이딩과 오버로딩을 구분해야 한다.

* **오버라이딩**: 상속받은 메서드를 같은 형식으로 재정의
* **오버로딩**: 같은 이름의 메서드를 매개변수를 다르게 하여 여러 개 정의

```java
void order(long productId) {
}

void order(long productId, int quantity) {
}
```

오버라이딩은 실제 객체에 따라 실행할 메서드가 결정되고, 오버로딩은 전달한 인자의 형태에 따라 컴파일할 때 결정된다.

## 인터페이스를 사용하면 역할 중심으로 협력할 수 있다

상속을 이용하면 하위 객체를 상위 타입으로 다룰 수 있지만, 객체의 역할만 표현하고 싶을 때는 인터페이스를 사용할 수 있다.

주문 서비스에는 결제가 필요하지만, 결제 방식이 반드시 카드일 필요는 없다.

```java
public interface PaymentProcessor {
    void pay(int amount);
}
```

`PaymentProcessor`는 결제 방법을 구현하지 않고 “결제할 수 있어야 한다”라는 역할만 정의한다.

```java
public class CardPayment implements PaymentProcessor {

    @Override
    public void pay(int amount) {
        System.out.println("카드 결제");
    }
}
```

```java
public class KakaoPayment implements PaymentProcessor {

    @Override
    public void pay(int amount) {
        System.out.println("카카오페이 결제");
    }
}
```

두 클래스는 구체적인 처리 방법은 다르지만 동일한 결제 역할을 수행한다.

```java
PaymentProcessor payment = new CardPayment();
payment.pay(10_000);
```

주문 서비스는 카드사의 처리 방법을 알 필요 없이 `PaymentProcessor`에게 `pay()`를 요청하면 된다. 실제 객체를 `KakaoPayment`로 바꾸더라도 요청 방법은 동일하다.

이것이 《객체지향의 사실과 오해》에서 말한 **같은 역할을 서로 다른 객체가 수행할 수 있다는 것**과 연결된다.

## 객체가 책임을 지려면 내부 상태를 보호해야 한다

다형성을 통해 객체를 교체할 수 있더라도 객체의 내부 상태를 외부에서 마음대로 바꿀 수 있다면 객체가 자신의 책임을 보장하기 어렵다.

예를 들어 주문 서비스가 상품의 재고를 직접 꺼내 수정한다고 해보자.

```java
int stock = product.getStock();

if (stock < quantity) {
    throw new IllegalStateException("재고 부족");
}

product.setStock(stock - quantity);
```

이 경우 주문 서비스가 상품의 재고 구조와 재고 부족 규칙을 모두 알아야 한다. 다른 서비스에서도 서로 다른 방법으로 재고를 수정할 수 있다.

따라서 상품의 상태는 숨기고, 상품이 허용하는 행동만 외부에 제공한다.

```java
public class Product {
    private int stock;

    public void decreaseStock(int quantity) {
        if (stock < quantity) {
            throw new IllegalStateException("재고 부족");
        }

        stock -= quantity;
    }
}
```

외부에서는 내부 상태를 직접 조작하지 않고 행동만 요청한다.

```java
product.decreaseStock(quantity);
```

이처럼 상태와 행동을 하나의 객체에 묶고 내부 구현을 숨기는 것이 **캡슐화**다. 캡슐화는 단순히 필드에 `private`을 붙이는 것이 아니라, 객체가 자신의 상태와 규칙을 직접 관리하도록 만드는 것이다.

접근제어자는 캡슐화의 경계를 지정한다.

| 접근제어자       | 접근 범위                  |
| ----------- | ---------------------- |
| `private`   | 같은 클래스                 |
| `default`   | 같은 패키지                 |
| `protected` | 같은 패키지와 다른 패키지의 자식 클래스 |
| `public`    | 모든 곳                   |

일반적으로 필드는 `private`으로 보호하고 외부에 필요한 행동만 `public` 메서드로 공개한다.

## 객체는 참조변수를 통해 사용한다

객체의 상태를 보호했다면 마지막으로 객체가 변수에 어떻게 저장되는지 이해해야 한다.

기본형 변수를 복사하면 값 자체가 복사된다.

```java
int a = 10;
int b = a;

b = 20;

System.out.println(a); // 10
```

`a`와 `b`는 서로 독립적인 값을 가진다.

반면 참조변수를 복사하면 객체가 하나 더 생성되는 것이 아니다.

```java
Animal first = new Animal();
Animal second = first;
```

`new`는 한 번만 실행됐기 때문에 객체는 하나다. `first`와 `second`에는 같은 객체를 가리키는 참조값이 들어 있다.

따라서 `second`를 이용해 객체의 상태를 바꾸면 `first`에서도 변경된 상태가 보인다.

```java
second = new Animal();
```

이 코드는 기존 객체를 변경한 것이 아니다. `second`가 새로운 객체를 가리키도록 연결을 변경한 것이다.

자바는 기본형과 참조형 모두 **값에 의한 전달**을 사용한다. 객체를 메서드에 전달할 때는 객체 자체가 전달되는 것이 아니라 참조값의 복사본이 전달된다.
