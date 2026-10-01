# 06_스프링이 사랑한 디자인 패턴
- 디자인 패턴: 개발자들이 정제 표준 설계 패턴
  
### 01_어댑터 패턴(Adapter Pattern)
**호출 당하는 쪽의 메서드를 호출하는 쪽의 코드에 대응하도록 중간에 변환기를 통해 호출하는 패턴**

```java
sa.runServiceA();
sb.runServiceB();
```
A와 B의 메서드 이름이 다르지만 어댑터를 끼우면 둘 다 runService()로 호출할 수 있다.

```java
adapterA.runService();  // 안에서 sa.runServiceA() 호출
adapterB.runService();  // 안에서 sb.runServiceB() 호출
```

- 언제 쓸 수 있을까?

예를 들어 만든 프로그램이 runService()라는 이름으로 서비스를 사용하도록 짜여 있는데 새로 가져온 서비스에는 runServiceB()만 있다. 

그 서비스 코드를 고치는 대신 B용 어댑터가 runService() 요청을 받아 runServiceB()를 호출하게 만들 수 있다.

외부 라이브러리처럼 직접 수정하기 어렵거나 여러 서비스의 호출 방식이 다를 때 유용하다.

### 02_프록시 패턴(Poroxy pattern)
프록시: 대리자, 대변인
- 실제 객체 대신 요청을 받고 필요한 처리를 한 뒤 실제 객체를 호출하는 패턴
  아래의 프록시는 서비스 실행 전에 로그를 출력한다.

```java
// 실제 객체와 프록시가 사용하는 공통 규격
interface Service {
    void run(); //1
}

// 실제 일을 하는 객체
class RealService implements Service {
    public void run() {
        System.out.println("서비스 실행");
    }
}

// 실제 객체 대신 요청을 받는 프록시
class ProxyService implements Service {
    private Service real = new RealService(); //2

    public void run() {
        System.out.println("호출 기록 남기기"); //4
        real.run(); //3
    }
}

public class Main {
    public static void main(String[] args) {
        Service service = new ProxyService();
        service.run();
    }
}
```
출력 결과

``호출 기록 남기기
서비스 실행``

- Main이 run()을 호출하면 프록시가 먼저 로그를 남기고 실제 객체의 run()을 호출해서 실제 서비스를 실행한다.

앞의 어댑터는 호출 방식을 맞춰 주는거라면 프록시는 같은 호출 방식으로 실제 객체를 대신해 요청을 받는 것이다.

```java
// 실제 서비스
class RealService {
    void deleteUser() {
        System.out.println("회원 삭제 완료");
    }
}

// 권한 확인하는 프록
class ProxyService {
    RealService real = new RealService();

    void deleteUser(boolean isAdmin) {
        if (isAdmin) {
            real.deleteUser(); // 관리자면 실제 서비스 호출
        } else {
            System.out.println("삭제 권한이 없습니다");
        }
    }
}

```

관리자만 회원 정보를 삭제할 수 있는 프로그램을 만드는 경우
누구나 바로 호출하면 회원을 삭제할 수 있다. 그래서 앞에 권한을 확인하는 프록시를 둔다.
실제 서비스에서는 관리자 여부를 로그인 정보를 통해 확인한다.


- 프록시 패턴의 중요 포인트
  1. 대리자는 실제 서비스와 같은 이름의 메서드를 구현한다. 이때 인터페이스를 사용한다.
  2. 대리자는 실제 서비스에 대한 참조 변수를 갖는다.
  3. 대리자는 실제 서비스의 같은 이름을 가진 메서드를 호출하고 그 값을 클라이언트에게 돌려준다.
  4. 대리자는 실제 서비스의 메서드를 호출 전후에도 별도의 로직을 수행할 수 있다.

### 03_데코레이터 패턴(Decorator Patter)

|패턴 | 차이점 |
| ---| ---|
| 프록시 패턴      |  제어의 흐름을 변경하거나 별도의 로직 처리를 목적으로 하며 반환값을 변경하지 않는다.|
| 데코레이터 패턴 |  반환값에 장식을 더한다.     |   

```java
interface Service {
    String run();
}

// 원래 서비스
class RealService implements Service {
    public String run() {
        return "안녕하세요";
    }
}

// 반환값에 장식을 추가하는 데코레이터
class Decorator implements Service {
    private Service service;

    Decorator(Service service) {
        this.service = service;
    }

    public String run() {
        return "😊 " + service.run() + "!"; // 원래 결과에 장식을 추가
    }
}

public class Main {
    public static void main(String[] args) {
        Service original = new RealService();
        Service decorated = new Decorator(original);

        System.out.println(original.run());  //실제 서비스를 바로 호출
        System.out.println(decorated.run()); // 데코레이터를 거친 실제 서비스를 호출
    }
}
```
출력 결과 

``안녕하세요
😊 안녕하세요!``

### 04_싱글턴 패턴(Singleton Pattern)
- 인스턴스를 하나만 만들어서 재사용한다.
- 다음의 세 가지가 반드시 필요함.
  1. new를 실행할 수 없도록 생성자에 private 접근 제어자를 지정한다.
  2. 유일한 단일 객체를 반환할 수 있는 정적 메서드가 필요하다.
  3. 유일한 단일 객체를 참조할 수 있는 정적 변수가 필요하다. 

```java
class Settings {
    // 객체를 하나만 생성해서 보관
    private static final Settings instance = new Settings();

    // 외부에서 new Settings()로 생성하지 못하게 막음
    private Settings() {}

    // 이미 만든 객체를 반환
    public static Settings getInstance() {
        return instance;
    }

    public void show() {
        System.out.println("설정 확인");
    }
}

public class Main {
    public static void main(String[] args) {
        Settings a = Settings.getInstance();
        Settings b = Settings.getInstance();

        a.show();
        b.show();

        System.out.println(a == b); // 같은 객체인지 확인
    }
}
```

출력 결과

``설정 확인
설정 확인
true``

- getInstance()를 두 번 호출해도 새 객체를 만들지 않고 같은 instance를 반환한다.
- 
  그래서 a와 b는 같은 객체를 가리킨다.
- 앱 전체에서 같은 정보를 공유해야 하는 객체에 적용한다.

### 05_템플릿 메서드 패턴(Template Method Pattern)
**상위 클래스의 견본 메서드에서 하위 클래스가 오버라이딩한 메서드를 호출하는 패턴**

**Animal.java**

```java
package templateMethodPattern;

public abstract class Animal {
    // 템플릿 메서드
    public void playWithOwner() {
        System.out.println("귀염둥이 이리온...");
        play();
        runSomething();
        System.out.println("잘했어");
    }

    // 추상 메서드
    abstract void play();

    // Hook(갈고리) 메서드
    void runSomething() {
        System.out.println("꼬리 살랑 살랑~");
    }
}
```
**Dog.java**
```java
package templateMethodPattern;

public class Dog extends Animal {
    @Override
    // 추상 메서드 오버라이딩
    void play() {
        System.out.println("멍! 멍!");
    }

    @Override
    // Hook(갈고리) 메서드 오버라이딩
    void runSomething() {
        System.out.println("멍! 멍!~ 꼬리 살랑 살랑~");
    }
}
```
부모의 introduce()가 소개 → 울음소리 → 마무리 순서를 정한다.

makeSound()만 자식에 따라 달라진다.

<**탬플릿 메서드 패턴**>
| 템플릿 메서드의 구성 요소 | 상위 클래스 (Animal) | 하위 클래스 (Dog/Cat) |
| --- | ---| ---|
|템플릿 메서드, 공통 로직을 수행 | playWithOwner() |     |
|템플릿 메서드에서 호출하는 추상 메서드 | play()| 오버라이딩 필수|
|템플릿 메서드에서 호출하는 훅(Hook) | runSomething() | 오버라이딩 선택|

### 06_팩터리 메서드 패턴(Factory Method Pattern)

팩터리 메서드: 객체를 생성 반환하는 메서드
팩터리 메서드 패턴: 하위 클래스에서 팩터리 메서드를 오버라이딩해서 객체를 반환하는 것

**팩터리 메서드 Animal.java**
```java
public abstract class Animal {
  // 추상 팩터리 메서드
  abstract AnimalToy getToy();
}
```
**Dog.java**
```java
public class Dog extends Animal{
  //추상 팩터리 메서드 오버라이딩
  @override
  AnimalToy getToy(){
      return new DogToy();
    }
}
```
< 탬플릿 메서드와 팩터리 메서드 구분 >
| 구분 | 템플릿 메서드 | 팩터리 메서드 |
|---|---|---|
| 목적 | 공통 흐름에서 일부 동작 변경 | 생성할 객체의 종류 변경 |
| 하위 클래스가 결정하는 것 | 어떻게 실행할까? | 어떤 객체를 만들까? |
| 예시 | 동물마다 다르게 울기 | 동물마다 다른 장난감 만들기 |

### 07_전략 패턴(Strategy Pattern)
**전략 패턴을 구성하는 세 가지 요소**
1. 전략 메서드를 가진 객체
2. 전략 객체를 사용하는 컨텍스트(전략 객체의 사용자/소비자)
3. 전략 객체를 생성해 컨텍스트에 주입하는 클라이언트 (전략 객체의 공급자)

**전략 인터페이스를 나타냄 Strategy.java**
```java
package strategyPattern;

public interface Strategy {
    public abstract void runStrategy();
}
```

**전략 인터페이스를 구현함 StrategyGun.java**
```java
package strategyPattern;

public class StrategyGun implements Strategy {
    @Override
    public void runStrategy() {
        System.out.println("탕, 타당, 타다당");
    }
}
```
**전략 객체를 사용함 Soldier.java**
```java
package strategyPattern;

public class Soldier {
    void runContext(Strategy strategy) {
        System.out.println("전투 시작");
        strategy.runStrategy();
        System.out.println("전투 종료");
    }
}
```
**객체 생성, 주입하는 클라이언트 Client.java**
```java
package strategyPattern;

public class Client {
    public static void main(String[] args) {
        // 컨텍스트(전략을 사용하는 군인) 생성
        Soldier rambo = new Soldier();

        // 총 전략을 생성하고 전달 → 총으로 전투
        rambo.runContext(new StrategyGun());
    }
}
```
### 08_템플릿 콜백 패턴(Template Callback Pattern) 

**전략 패턴과 모든 것이 동일한데 전략을 익명 내부 클래스로 정의해서 사용한다는 특징이 있음**
```java
// 콜백의 공통 규격
interface Strategy {
    void runStrategy();
}

// 공통 실행 흐름
class Soldier {
    void runContext(Strategy strategy) {
        System.out.println("전투 시작");
        strategy.runStrategy(); // 전달받은 콜백 실행
        System.out.println("전투 종료");
    }
}

public class Client {
    public static void main(String[] args) {
        Soldier rambo = new Soldier();

        // 익명 클래스로 콜백을 만들어 전달
        rambo.runContext(new Strategy() {
            @Override
            public void runStrategy() {
                System.out.println("총으로 공격");
            }
        });

        // 다른 콜백 전달
        rambo.runContext(new Strategy() {
            @Override
            public void runStrategy() {
                System.out.println("검으로 공격");
            }
        });
    }
}
```
앞의 전략 패턴 예시에서는 따로 만들었던 무기 클래스들은 여기서는 익명 클래스로 필요한 동작을 그 자리에서 정의해서 전달한다.

한 곳에서만 쓰는 짧은 동작은 익명 클래스, 여러 곳에서 반복해서 쓰고 관리해야 할 것이 많으면 이름이 있는 별도 클래스로 쓰는게 좋다.

< 총정리>

| 패턴 | 핵심 역할 | 실무에서 언제 쓰면 좋을까? |
|---|---|---|
| **어댑터** | 서로 다른 인터페이스를 맞춰 줌 | 외부 결제 API를 우리 서비스의 결제 규격에 맞춰 연결할 때 |
| **프록시** | 실제 객체 앞에서 접근을 제어함 | 서비스 실행 전 권한을 검사하거나, 필요한 시점까지 객체 생성을 미룰 때 |
| **데코레이터** | 객체를 감싸 기능을 추가함 | 데이터 입출력에 압축·암호화 같은 기능을 조합해서 붙일 때 |
| **싱글턴** | 한 인스턴스를 공유하도록 함 | 여러 곳에서 같은 설정 관리 객체를 사용해야 할 때 |
| **템플릿 메서드** | 공통 흐름을 상위 클래스에 두고, 일부 단계를 하위 클래스가 구현함 | 파일 읽기 → 데이터 처리 → 저장 흐름에서 파일 형식별 처리를 다르게 할 때 |
| **팩터리 메서드** | 생성할 객체를 하위 클래스가 결정함 | 알림 발송의 공통 흐름에서 이메일·문자 발송 객체를 각각 생성할 때 |
| **전략** | 사용할 동작을 객체로 분리해서 교체함 | 결제 방식이나 할인 정책을 선택·변경할 때 |
| **템플릿 콜백** | 공통 흐름 중 필요한 동작을 콜백으로 전달받음 | DB 연결·정리 처리는 공통으로 하고, 실행할 SQL 처리를 전달할 때 |
