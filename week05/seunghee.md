# 7장 스프링 삼각형과 설정 정보

## 1. IoC / DI

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
--- 

## 2. AOP
AOP의 목적은 여러 핵심 로직에서 반복해서 끼어드는 공통 로직을 밖으로 빼고 핵심 관심사만 남기는 것

책의 예시로 DB 작업은 `커넥션 획득 -> 실제 DB 작업 -> 커넥션 반납` 에서 실제 DB 작업만 매번 달라지고 앞뒤는 반복된다.

---

### 관련 어노테이션들
1. `@Aspect` 어노테이션
> 이 클래스가 AOP용 공통 로직을 담는 클래스다 라고 표시하는 것

2. `@Before` 어노테이션
> 대상 메서드가 실행되기 전에 이 코드를 먼저 실행해라 라고 표시하는 것

3. `@After` 어노테이션
> 대상 메서드가 실행된 후에 이 코드를 실행해라 라고 표시하는 것

4. `@AfterReturning` 어노테이션
> 값을 정상적으로 반환한 후에 이 코드를 실행해라 라고 표시하는 것

5. `@Around` 어노테이션
> 실행 전후 전체를 감쌈. 실행 전에도 뭔가 할 수 있고, 실제 메서드를 호출한 뒤에도 뭔가 할 수 있다.

---

### 관련 용어들
1. Aspect: 공통 관심사를 모아놓은 전체 모듈
2. Advice: 실제로 끼워 넣을 코드
3. Pointcut: 어디에 끼워 넣을지 정하는 조건
4. Join Point: 실제로 끼어들 수 있는 지점
5. Advisor: Pointcut + Advice를 묶은 것

```java
@Before("execution(* com.example..*(..))") // -> Pointcut
public void log() { 
  System.out.println("로그"); // -> Advice
}
// 어떤 서비스 메서드 실행 지점 자체가 Join Point
// 이 전체를 담는 AOP 클래스가 Aspect
```
한문장으로 정리하면
> Aspect 안에 Advice가 있고, Pointcut으로 어떤 Join Point에 Advice를 적용할지 정한다.

---

### 대표적인 AOP: `@Transactional`
우리가 스프링 프로젝트에서 매~번 쓰는 트랜잭션 어노테이션이다.

아래의 순서대로 진행이 된다.


DB 커넥션을 획득 -> 트랜잭션 시작 -> 비즈니스 로직 수행 -> 성공하면 커밋/실패하면 롤백 -> 커넥션 종료

따라서 아래의 코드를 작성해야한다.

```java
public void order() {
    Connection con = null; 

    try {
        con = dataSource.getConnection(); // 커넥션 획득
        con.setAutoCommit(false); // 자동커밋 끄기 & 트랜잭션 시작 

        // 핵심 비즈니스 로직
        orderRepository.save(...); // 주문정보저장 (핵심 관심사)
        paymentRepository.save(...); // 결제정보저장 (핵심 관심사)

        con.commit(); // 성공하면 커밋

    } catch (Exception e) {
        if (con != null) {
            con.rollback(); // 실패하면 롤백
        }
        throw e;
    } finally {
        if (con != null) {
            con.close(); // 커넥션 정리
        }
    }
}
```

그런데 여기서 주문정보저장, 결제정보저장은 *핵심 관심사*이고, 나머지는 전부 *횡단관심사*이다.

DB 커밋과 롤백의 원자성이 필요한 모든 비즈니스 로직은 위 패턴이 반복된다.

여기서 AOP 용어를 연결하면,

1. Aspect: 트랜잭션이라는 횡단 관심사 전체
2. Advice: 트랜잭션 시작, commit, rollback처럼 실제로 실행되는 부가 로직
3. Pointcut: 어떤 메서드에 트랜잭션 기능을 적용할지 정하는 조건
4. Join Point: 실제 서비스 메서드가 실행되는 지점
5. Advisor: 어떤 Pointcut에 어떤 Advice를 적용할지 묶은 것

그리고 트랜잭션은 메서드 전후 전체를 감싸야함으로, 개념적으로 `Around` Advice와 잘 맞는다.

그래서 결국 `@Transactional` 을 붙이면, 아래와 같이 핵심 로직만 작성하면 된다. 그럼 결국 이 애너테이션은 "이 메서드 실행에 트랜잭션 기능을 적용해줘" 라고 표시하는 역할이 된다.
```java
@Transactional
public void order() {
  orderRepository.save(...);
  paymentRepository.save(...);
}
```

#### 위 AOP 과정에서 프록시의 역할
그럼 Spring은 실제로 메서드에 코드를 어떻게 주입할까?

order() 메서드 몸체에 공통 처리 코드를 주입할까? 그렇지 않다.

대리 객체, proxy를 둔다. 

6장에서 공부한 그 프록시 패턴 맞다. 

실제 객체 앞에 대리 객체를 둠으로 클라이언트가 원본 객체에 직접 접근하는 대신 프록시를 거쳐 *제어*하도록 만드는 구조 디자인 패

`Controller` 객체가 `Service` 객체를 호출하면, `Transaction Proxy`가 먼저 받는다.

```text
Controller
   ↓
Transaction Proxy
   ↓
실제 OrderService
```

그럼 프록시는 트랜잭션을 시작하고, 실제 OrderService.order()를 호출하고, 성공하면 커밋/실패하면 롤백을 수행한다.

개념적으로는 아래 코드처럼 프록시가 로직을 수행한다.

```java
class TransactionProxy {
  private final OrderService orderService;

  void order() {
    beginTransaction();

    try {
      realService.order();
      commit();
    } catch (Exception e) {
        rollback();
        throw e;
      }
    }
}
```

---

### 면접이나 실무에서 Spring AOP에 대해 추가로 알아야할 두가지
#### 첫째, Spring AOP는 주로 프록시 기반이어서 프록시를 거쳐야 적용이 된다.
#### 둘째, 그래서 같은 객체 내부에서 자기 메서드를 직접 호출하는 경우처럼 프록시를 우회하면 AOP가 적용되지 않을 수 있다.
