# Week 04

## 범위
- 6장. 스프링이 사랑한 디자인 패턴

## 학습 내용


- 객체 지향 4대 특성: 도구
- 객체 지향 설계 5원칙: 도구를 올바르게 사용하는 방법
- **디자인 패턴**: 표준화된 레시피 -> 표준 설계 패턴
- **스프링 프레임워크**: 자바 엔터프라이즈 개발을 편하게 해주는 오픈소스 경량급 애플리케이션 프레임워크: **OOP 프레임워크**

디자인 패턴은 객체 지향의 특성 중 **상속, 인터페이스, 합성**(객체를 속성으로 사용)을 이용


### 4-1. 어댑터 패턴

"호출당하는 쪽의 메서드를 호출하는 쪽의 코드에 대응하도록 중간에 변환기를 통해 호출하는 패턴"

- **변환기**: 서로 다른 두 인터페이스 사이에 통신이 가능하게 하는 것
  - Ex) 충전기(핸드폰과 전원 콘센트 사이에서 둘을 연결)
- **ODBC/JDBC**: 어댑터 패턴을 이용해 다양한 데이터베이스 시스템을 단일한 인터페이스로 조작할 수 있게 해줌
- **JRE**: 자바로 만들어진 프로그램을 컴퓨터에서 실행하기 위해 필요한 자바 실행 환경, 어댑터 패턴
- JDBC, JRE → **개방 폐쇄 원칙(OCP)** 예시
- **합성**: 객체를 속성으로 만들어서 참조


```java
// 기존 서비스: 메서드 이름이 서로 다름
public class ServiceA {
    void runServiceA() { System.out.println("ServiceA"); }
}
public class ServiceB {
    void runServiceB() { System.out.println("ServiceB"); }
}

// 어댑터: 같은 이름(runService)으로 변환
public class AdapterServicA {
    ServiceA sa1 = new ServiceA();   // 합성
    void runService() { sa1.runServiceA(); }
}
public class AdapterServicB {
    ServiceB sb1 = new ServiceB();   // 합성
    void runService() { sb1.runServiceB(); }
}
```
```java
// 어댑터 없을 때 (ClientWithNoAdapter)
ServiceA sa1 = new ServiceA();
ServiceB sb1 = new ServiceB();
sa1.runServiceA();
sb1.runServiceB();

// 어댑터 있을 때 (ClientWithAdapter)
AdapterServicA asa1 = new AdapterServicA();
AdapterServicB asb1 = new AdapterServicB();
asa1.runService();
asb1.runService();
```

- **문제**: ServiceA와 ServiceB는 메서드 이름이 달라서(`runServiceA()`, `runServiceB()`) 클라이언트가 서비스마다 다른 호출 방식을 알아야 한다.
- **해결**: 기존 클래스는 수정하지 않고, 어댑터 클래스(`AdapterServicA`, `AdapterServicB`)가 `runService()`라는 같은 이름의 메서드를 제공한다. 어댑터 내부에서는 원래 메서드를 대신 호출한다.
- **결과**: 클라이언트는 어떤 서비스든 `runService()`로 호출할 수 있다.
- **한계**: 이 예제에는 공통 인터페이스가 없어서 클라이언트가 여전히 구체적인 어댑터 클래스 두 개를 알아야 한다. 실무에서는 Service 인터페이스를 두고 어댑터들이 이를 구현하게 해서 클라이언트가 인터페이스에만 의존하도록 만든다(예: JDBC).

### 4-2. 프록시 패턴
"제어 흐름을 조정하기 위한 목적으로 중간에 대리자를 두는 패턴"

- **대리자, 대변인**: 다른 누군가를 대신해 그 역할을 수행하는 존재
- 서비스 객체가 들어갈 자리에 대리자 객체를 대신 투입 → 클라이언트 쪽에서는 대리자를 통해 호출하는지 전혀 모르게 처리할 수도 있다.


```java
public interface IService {
    String runSomething();
}

// 실제 서비스
public class Service implements IService {
    public String runSomething() { return "서비스 짱!!!"; }
}

// 대리자(프록시)
public class Proxy implements IService {
    IService service1;

    public String runSomething() {
        System.out.println("호출에 대한 흐름 제어가 주목적, 반환 결과를 그대로 전달");
        service1 = new Service();
        return service1.runSomething();   // 실제 서비스에 위임
    }
}
```
```java
// 프록시 없을 때
Service service = new Service();
System.out.println(service.runSomething());

// 프록시 있을 때
IService proxy = new Proxy();
System.out.println(proxy.runSomething());
```
```
[프록시 없음]  Client ──runSomething()──▶ Service

[프록시 있음]  Client ──runSomething()──▶ Proxy ──runSomething()──▶ Service
                                        (호출 전후에 추가 작업 가능)
```

- 대리자는 실제 서비스와 같은 이름의 메서드를 구현한다. 이때 **인터페이스**를 사용한다.
- 대리자는 실제 서비스에 대한 참조 변수를 갖는다(**합성**).
- 대리자는 실제 서비스의 같은 이름을 가진 메서드를 호출하고 그 값을 클라이언트에게 **그대로** 돌려준다.
- 대리자는 실제 서비스의 메서드 호출 전후에 별도의 로직을 수행할 수도 있다. → **흐름 제어** (Proxy의 `println`)

- **의존 역전 원칙**: 스노우타이어, 일반타이어, 광폭타이어를 서로 교체해 주어도 자동차는 영향 받지 않음
- **개방 폐쇄 원칙**

### 4-3. 데코레이터 패턴
"메서드 호출의 반환값에 변화를 주기 위해 중간에 장식자를 두는 패턴"
- **장식자**: 원본에 장식을 더하는 패턴
- **프록시 패턴과 구현 방법이 같다.** 다만 프록시 패턴은 클라이언트가 최종적으로 돌려 받는 반환값을 조작하지 않고 그대로 전달하는 반면 데코레이터 패턴은 클라이언트가 받는 반환값에 장식을 덧입힌다.


```java
public class Decoreator implements IService {
    IService service;

    public String runSomething() {
        System.out.println("호출에 대한 장식 주목적, 클라이언트에게 반환 결과에 장식을 더하여 전달");
        service = new Service();
        return "정말" + service.runSomething();   // 프록시와 다른 부분: 반환값에 장식
    }
}
```
```java
IService decoreator = new Decoreator();
System.out.println(decoreator.runSomething());   // 정말서비스 짱!!!
```

- 장식자는 실제 서비스와 같은 이름의 메서드를 구현한다. 이때 **인터페이스**를 사용한다. → `Decoreator implements IService`, `runSomething()`
- 장식자는 실제 서비스에 대한 참조 변수를 갖는다(**합성**). → 필드 `IService service;`
- 장식자는 실제 서비스의 같은 이름을 가진 메서드를 호출하고, 그 반환값에 **장식을 더해** 클라이언트에게 돌려준다. → `"정말" + service.runSomething()`
- 장식자는 실제 서비스의 메서드 호출 전후에 별도의 로직을 수행할 수도 있다. → 호출 전 `System.out.println(...)`

- **개방 폐쇄 원칙**: Service를 수정하지 않고 장식 기능을 추가함
- **의존 역전 원칙**: 클라이언트가 IService 타입에 의존함

**생성자 주입으로 바꾸면** (원본의 `new Service()` 삭제)

```java
public class Decoreator implements IService {
    IService service;

    public Decoreator(IService service) {   // 생성자 주입
        this.service = service;
    }

    public String runSomething() {
        System.out.println("호출에 대한 장식 주목적, 클라이언트에게 반환 결과에 장식을 더하여 전달");
        return "정말" + service.runSomething();
    }
}
```
```java
IService s = new Decoreator(new Decoreator(new Service()));
System.out.println(s.runSomething());   // 정말정말서비스 짱!!!

// 안쪽부터 생성
// ① new Service()        → 원본
// ② new Decoreator(①)    → ①을 감싼 장식자 (안쪽)
// ③ new Decoreator(②)    → ②를 감싼 장식자 (바깥쪽) = s
```
- 생성자는 `IService` 타입이면 뭐든 받음 + `Decoreator` 자신도 `IService` 구현 → **장식자를 겹겹이 감쌀 수 있음**

### 4-4. 싱글턴 패턴

- 인스턴스를 하나만 만들어 사용하기 위한 패턴
- 커넥션 풀, 스레드 풀, 디바이스 설정 객체 등과 같은 경우 인스턴스를 여러 개 만들게 되면 불필요한 자원을 사용하게 되고, 또 프로그램이 예상치 못한 결과를 낳을 수 있다. 싱글턴 패턴은 오직 인스턴스를 하나만 만들고 그것을 계속해서 재사용한다.
- 의미상 두 개의 객체가 존재할 수 없다. 이를 구현하려면 객체 생성을 위한 new에 제약을 걸어야 하고, 만들어진 단일 객체를 반환할 수 있는 메서드가 필요하다.
- 반드시 필요한 세 가지
  - new를 실행할 수 없도록 생성자에 **private** 접근 제어자를 지정한다.
  - 유일한 단일 객체를 반환할 수 있는 **정적 메서드**가 필요하다.
  - 유일한 단일 객체를 참조할 **정적 참조 변수**가 필요하다.


```java
public class Singleton {
    static Singleton singletonObject;   // 정적 참조 변수

    private Singleton() {}              // private 생성자

    // 객체 반환 정적 메서드
    public static Singleton getInstance() {
        if (singletonObject == null) {
            singletonObject = new Singleton();
        }
        return singletonObject;
    }
}
```
```java
// Singleton s = new Singleton();   // private 생성자이므로 new 불가

Singleton s1 = Singleton.getInstance();
Singleton s2 = Singleton.getInstance();
Singleton s3 = Singleton.getInstance();
// s1, s2, s3 모두 같은 객체를 참조
```

### 4-5. 템플릿 메서드 패턴
"상위 클래스의 견본 메서드에서 하위 클래스가 오버라이딩한 메서드를 호출하는 패턴"

- 상위 클래스에 공통 로직을 수행하는 **템플릿 메서드**와 하위 클래스에 오버라이딩을 강제하는 **추상 메서드** 또는 선택적으로 오버라이딩할 수 있는 **훅 메서드**를 두는 패턴


```java
public abstract class Animal {
    // 템플릿 메서드: 공통 로직
    public void playWithOwner() {
        System.out.println("귀염둥이 이리온...");
        play();
        runSomething();
        System.out.println("잘했어");
    }

    // 추상 메서드: 오버라이딩 강제
    abstract void play();

    // 훅 메서드: 선택적으로 오버라이딩
    void runSomething() {
        System.out.println("꼬리 살랑 살랑~");
    }
}

public class Dog extends Animal {
    @Override void play() { System.out.println("멍! 멍!"); }
    @Override void runSomething() { System.out.println("멍! 멍!~ 꼬리 살랑 살랑~"); }
}

public class Cat extends Animal {
    @Override void play() { System.out.println("야옹~ 야옹~"); }
    @Override void runSomething() { System.out.println("야옹~ 야옹~ 꼬리 살랑 살랑~"); }
}
```
```java
Animal bolt = new Dog();
Animal kitty = new Cat();

bolt.playWithOwner();    // 상위 클래스의 템플릿 메서드 → Dog가 오버라이딩한 메서드 호출
kitty.playWithOwner();
```

### 4-6. 팩터리 메서드 패턴
"오버라이드 된 메서드가 객체를 반환하는 패턴"

- **팩터리 메서드**: 객체를 생성 반환하는 메서드
- 하위 클래스에서 팩터리 메서드를 오버라이딩해서 객체를 반환


```java
public abstract class Animal {
    // 추상 팩터리 메서드
    abstract AnimalToy getToy();
}

// 팩터리 메서드가 생성할 객체의 상위 클래스
public abstract class AnimalToy {
    abstract void identify();
}

public class Dog extends Animal {
    @Override
    AnimalToy getToy() { return new DogToy(); }   // 추상 팩터리 메서드 오버라이딩
}
public class Cat extends Animal {
    @Override
    AnimalToy getToy() { return new CatToy(); }
}

// 팩터리 메서드가 생성할 객체
public class DogToy extends AnimalToy {
    public void identify() { System.out.println("나는 테니스공! 강아지의 친구!"); }
}
public class CatToy extends AnimalToy {
    public void identify() { System.out.println("나는 캣타워! 고양이의 친구!"); }
}
```
```java
// 팩터리 메서드를 보유한 객체들 생성
Animal bolt = new Dog();
Animal kitty = new Cat();

// 팩터리 메서드가 반환하는 객체들
AnimalToy boltBall = bolt.getToy();
AnimalToy kittyTower = kitty.getToy();

boltBall.identify();
kittyTower.identify();
```

### 4-7. 전략 패턴
"클라이언트가 전략을 생성해 전략을 실행할 컨텍스트에 주입하는 패턴"

- 전략 메서드를 가진 **전략 객체** → `StrategyGun`, `StrategySword`, `StrategyBow`
- 전략 객체를 사용하는 **컨텍스트**(전략 객체의 사용자/소비자) → `Soldier`
- 전략 객체를 생성해 컨텍스트에 주입하는 **클라이언트**(제3자, 전략 객체의 공급자) → `Client`


```java
// 전략 인터페이스
public interface Strategy {
    public abstract void runStrategy();
}

// 전략 객체
public class StrategyGun implements Strategy {
    public void runStrategy() { System.out.println("탕, 타당, 타다당"); }
}
public class StrategySword implements Strategy {
    public void runStrategy() { System.out.println("챙.. 채쟁챙 챙챙"); }
}
public class StrategyBow implements Strategy {
    public void runStrategy() { System.out.println("슝.. 쐐액.. 쇅, 최종 병기"); }
}

// 컨텍스트
public class Soldier {
    void runContext(Strategy strategy) {
        System.out.println("전투 시작");
        strategy.runStrategy();
        System.out.println("전투 종료");
    }
}
```
```java
// 클라이언트: 전략을 생성해 컨텍스트에 주입
Soldier rambo = new Soldier();

rambo.runContext(new StrategyGun());
rambo.runContext(new StrategySword());
rambo.runContext(new StrategyBow());
```

### 4-8. 템플릿 콜백 패턴

- **의존성 주입**


```java
// 전략 객체를 따로 클래스로 만들지 않고, 익명 내부 클래스로 바로 만들어 주입
Soldier rambo = new Soldier();

rambo.runContext(new Strategy() {
    @Override
    public void runStrategy() {
        System.out.println("총! 총초종총 총! 총!");
    }
});

rambo.runContext(new Strategy() {
    @Override
    public void runStrategy() {
        System.out.println("칼! 카가갈 칼! 칼!");
    }
});
```


```java
// 중복되는 익명 내부 클래스 생성을 컨텍스트 안으로 이동
public class Soldier {
    void runContext(String weaponSound) {
        System.out.println("전투 시작");
        executeWeapon(weaponSound).runStrategy();
        System.out.println("전투 종료");
    }

    private Strategy executeWeapon(final String weaponSound) {
        return new Strategy() {
            @Override
            public void runStrategy() {
                System.out.println(weaponSound);
            }
        };
    }
}
```
```java
Soldier rambo = new Soldier();
rambo.runContext("총! 총초종총 총! 총!");
rambo.runContext("칼! 카가갈 칼! 칼!");
rambo.runContext("도끼! 독독..도도독 독끼!");
```

### 4-9. 스프링이 사랑한 다른 패턴들

- **프론트 컨트롤러 패턴**
