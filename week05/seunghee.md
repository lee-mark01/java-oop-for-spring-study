# 7장 스프링 삼각형과 설정 정보

## IoC / DI

### 의존성이란 무엇인가
책은 자동차-타이어 예시로 시작하고 있다.
```java
class Car {
  // 나는 Tire를 사용하기 위해 직접 생성할게.
  private Tire tire = new KoreaTire();
}
```
이 경우 `Car` 객체가 `KoreaTire` 객체를 생성하고 있다.

이를 우리는 아래와 같이 말한다.
> `Car`가 `KoreaTire`라는 구체 클래스를 직접 선택한다. 직접 생성한다. 직접 의존한다.
> 그래서 나중에 KoreaTire를 AmericaTire로 바꾸려면 `Car` 코드를 고쳐야 한다.

---

### 생성자를 통한 의존성 주입
반면 생성자를 통한 DI는 아래 코드와 같다.
```java
class Car {
  private Tire tire;

  // 나는 Tire를 사용해
  Car(Tire tire){
    this.tire = tire;
  }
}
```
그리고 다른 객체에서 `Tire`를 생성해서 `Car`에 넣어준다.
```java
Tire tire = new KoreaTire();
Car car = new Car(tire);
```

---

### 속성을 통한 의존성 주입
생성자가 아니라 setter 같은 속성 접근자 메서드를 통해서도 의존성을 주입할 수 있다.

```java
class Car {
  private Tire tire;

  // 나는 Tire를 사용해 
  void setTire(tire){
    this.tire = tire;
  }
}
```

그리고 다른 객체에서 `Tire`를 생성해서 `Car`에 넣어준다.
```java
Car car = new Car();
car.setTire(new KoreaTire());
```
이것도 Car가 직접 KoreaTire를 생성하지 않고, 다른 객체에서 생성한 후 전달받으니 DI이다.

이렇게 의존성을 주입하는 방식은 객체의 "사용"과 "생성"을 분리할 수 있다.

사용은 필요한 객체 내에서, 생성은 필요한 객체 밖에서 한다.

---

### 두 방식의 차이
둘 다 DI이지만, 현재 Spring에서는 생성자 주입을 기본으로 생각하는 게 좋다고 한다. 

왜냐하면 생성자 주입은 객체가 만들어지는 순간 필요한 의존성이 모두 준비되기 때문이다.

setter 주입 같은 경우에는 Car가 만들어졌는데 Tire가 아직 없는 상태가 가능해진다.

따라서 필수 의존성은 보통 생성자로 받는 게 좋다.

---

## 누가 new를 하고 누가 주입을 해주는가? Spring의 등장
그렇다면 어떤 객체가 결국 new로 객체를 생성하고, 주입으로 객체를 연결하고 조립해야한다. 누가 할 것인가?

Spring에게 "Tire, Car, Driver 필요. Car에는 Tire를 넣고, Driver에는 Car를 넣어" 라고 알려주면 Spring이 대신 처리해준다.

그럼 개발자는 Car는 어떤 역할을 하고, Driver는 어떤 역할을 하는지 좀 더 핵심적인 일에 집중할 수 있다.

---

### 제어의 역전, IoC
제어란 내 코드가 객체 생성과 연결을 직접 제어한다는 의미이다.

아래 코드는 객체 생성을 제어하고 있다.
```java
new KoreaTire();
new Car(...);
new Driver(...);
```
그런데 Spring을 쓰면 Spring Container가 객체를 생성하고, 보관하고, 연결한다.

즉, 객체 관리의 제어권이 내 코드에서 Spring으로 넘어간 것이다.

그래서 IoC, Inversion of Control, 제어의 역전이라고 한다.

그리고 이제 DI (의존성 주입)의 주체는 Spring이 되었다. 

---

### 책의 XML 방식
XML 설정 처음 본다. 오래 전의 방식이라고 한다.

```XML
<bean id="tire" class="KoreaTire"/>

<bean id="car" class="Car">
    <constructor-arg ref="tire"/>
</bean>
```

의미는 KoreaTire 객체를 만들고, Car 객체를 만들고, Car 생성자에 KoreaTire 넣어 라는 의미이다.

> 즉, 이 XML은 Spring에게 객체를 어떻게 생성하고 연결할지 알려주는 설정 정보이다.

---

### 현재 많이 쓰는 Java/Annotation 기반 설정
그러나 요즘은 XML 대신 애너테이션과 Java 설정을 많이 사용한다.

```java
@Component
class KoreaTire implements Tire {
}
```

```java
@Component
class Car {
  private final Tire tire;

  Car (Tire tire){
    this.tire = tire;
  }
}
```

Spring은 애플리케이션 시작 시 *컴포넌트 스캔*을 수행한다.

`@Component`,`@Service`,`@Repository`, `@Controller` 등이 붙은 클래스를 찾아 Spring Bean으로 등록한다.

그 후 `Car` 객체를 생성하려고 보면 생성자에 `Tire`가 필요한 것을 확인한다.

`Car(Tire tire)`

그럼 컨테이너에 등록된 Bean 중 `Tire` 타입에 해당하는 Bean을 찾는다.

---

### @Autowired
`@Autowired`는 마찬가지로 필요한 의존성을 Spring이 찾아 주입하도록 지정하는 애너테이션이다. 

그리고 *타입*을 기준으로 Bean을 찾는다.

```java
@Autowired
Car(Tire tire) {
    this.tire = tire;
}
```

그러나 생성자가 하나뿐이라면 최근 Spring에서는 생성자에 @Autowired를 생략해도 자동으로 주입된다.

### @Resource
`@Resource`도 의존성을 주입하는 데 사용한다.

그리고 *이름*을 기준으로 Bean을 찾는다.

```java
@Resource(name = "koreaTire")
private Tire tire;
```

