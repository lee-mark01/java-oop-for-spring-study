# 3주차

## 범위

4장. 자바가 확장한 객체 지향 ~ 5장. 객체 지향 설계 5원칙 - SOLID

## 학습 내용

### 4장

#### 1. 추상 메서드와 추상 클래스

- 키워드 abstract 를 이용하여 선언.
  ```java
  public abstract class Vehicle {
    String model;
    
    Vehicle(String model) {
      this.model = model;
    }
    
    abstract void move();
    
    void stop() {
      System.out.printf("%s 정지합니다.", model);
    }
  }
  ```

- 추상 메서드: 선언되어 있지만, 구현되어 있지는 않은 메서드. 
- 추상 메서드를 하나라도 갖고 있는 클래스는 반드시 추상 클래스로 선언해야 한다.
  단, 추상 메서드가 없어도 추상 클래스로 선언 가능.

- 추상 클래스는 인스턴스를 만들 수 없고, 이를 상속받는 경우 하위 클래스는 메서드를 반드시 구현해야 한다.

##### 사용하는 이유

> 공통 기능은 물려주고, 서로 다른 핵심 동작은 반드시 직접 구현하도록 강제

1. 코드 재사용, 중복 제거

   - 모든 하위 클래스가 공통으로 사용하는 필드와 메서드를 상위 클래스에 모아둠.

2. 불완전한 객체 생성 방지

   > 동물 객체는 어떻게 울어야 하지? / 누가 실수로 동물 객체를 만들면 어떡하지? (p.149)

   - 개념적으로만 존재하는 상위 클래스가 단독으로 인스턴스화되는 것을 컴파일 단계에서 차단.

3. 하위 클래스에 규격 강제

   - 협업을 하거나, 프로젝트 규모가 커지는 경우, 같은 기능을 각각 다른 이름으로 만들어 불편함을 야기함.
   - 추상 메서드를 미리 정의해두어 추상 클래스를 상속받아 사용하는 사람은 반드시 추상 메서드를 구현해야 함.

   

#### 2. 생성자

- 객체 생성 메서드. 클래스의 인스턴스를 만들 때 사용.
- 키워드 new 를 사용함.
- 생성자가 없는 경우, 자바 컴파일러는 기본 생성자를 자동으로 생성.
  인자가 있는 생성자를 하나라도 만들 경우, 기본 생성자를 자동으로 생성하지 않음.
- 오버로딩 가능. 즉, 같은 이름에 인자가 다른 생성자를 여러 개 만들 수 있음.



#### 3. final

- final 클래스: 해당 클래스를 상속할 수 없음.
  - 무분별한 상속을 방지.
  - `String`, `Integer` 등의 래퍼 클래스는 모두 final 클래스. 이를 상속받아 시스템 핵심 동작을 악의적으로 오버라이딩하는 것을 시스템 차원에서 방지.
- final 필드(변수): 상수. 한번 초기화하면 값을 변경할 수 없음.
  - 의도치 않은 곳에서 값이 변경되어 발생하는 오류를 차단. 외부에서 setter 등으로 객체 내부 상태를 훼손하는 것을 구조적으로 방지.
  - 스레드 안전성. 여러 스레드가 동시 접근해도 상태가 바뀌지 않기 때문에, 복잡한 동기화 로직 없이도 안전하게 공유할 수 있음.
- final 메서드: 해당 메서드를 오버라이딩 할 수 없음.
  - 상위 클래스에서 정의한 절대 바뀌면 안되는 핵심 기능을 하위 클래스에서 변경하는 것을 방지.



#### 4. 인터페이스

- 클래스가 반드시 구현해야 하는 동작의 규격을 정의한 것.
- 추상 메서드들로 구성되며, 인스턴스 생성 불가.
- 인터페이스 내부 모든 변수는 자동으로 `public static final` 이 적용되어 상수로 동작.
- "~ 할 수 있는(is able to)"

##### 사용하는 이유

1. 다중 상속(구현) 지원
   - 자바는 클래스의 단일 상속만을 허용(다이아몬드 문제)
   - 인터페이스는 여러 개를 동시에 구현할 수 있어 **유연한 확장** 가능.
2. 느슨한 결합
   - 구체적인 구현 클래스가 아닌 인터페이스에 의존하게 함.
   - 구현체의 내부 코드나 구현 방식이 변경되더라도, 호출자에게 그 영향이 전파되지 않음.



#### 5. this와 super

- this: 객체가 자기 자신을 지칭.
- super: 상위 클래스 인스턴스를 지칭.
- 지역 변수와 속성(객체 변수, 정적 변수) 이름이 같은 경우 지역 변수가 우선하기 때문에, 이 경우 객체 변수를 사용하려면 this 를 사용.



### 5장

#### SOLID

| 원칙                      | 핵심 요약                                                    |
| ------------------------- | ------------------------------------------------------------ |
| **SRP** (단일 책임)       | 하나의 클래스는 **단 하나의 변경 이유**만 가져야 한다.       |
| **OCP** (개방 폐쇄)       | 기존 코드는 **변경하지 않고**, 새로운 기능은 클래스 추가로 **확장**한다. |
| **LSP** (리스코프 치환)   | 하위 타입은 상위 타입의 **동작 규약을 깨뜨리지 않고 완벽히 대체** 가능해야 한다. |
| **ISP** (인터페이스 분리) | 클라이언트가 **쓰지도 않는 메서드에 의존하지 않도록** 작게 쪼갠다. |
| **DIP** (의존관계 역전)   | 구체 클래스(구현)에 의존하지 말고, **인터페이스(추상화)에 의존**한다. |

**🙏 SOLID 원칙 위반 사례와 리팩토링 코드는 AI를 사용하여 만들었음.**

#### 1. 단일 책임 원칙 (Single Responsibility Principle)

- 하나의 클래스는 하나의 책임만 져야 한다.
- 특정 하위 개념에만 유효한 속성을 상위 클래스에 정의할 경우, 해당 속성을 필요로 하지 않는 대상까지 불필요한 책임을 떠안게 됨.(p. 180)
  이 경우, 특정 속성을 필요로 하는 대상만을 위한 별도 클래스로 분리하여 고유 책임을 부여하고,
  공통점이 많은 경우 상위 클래스에 공통 요소를 두고, 하위 클래스가 이를 상속받아 차이점만 구현하게 함.
  공통점이 없는 경우 상위 클래스를 제거하고, 독립된 클래스로 각각 분리 처리.



#### 2. 개방 폐쇄 원칙 (Open-Closed Principle)

- 소프트웨어 엔티티는 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 한다.

- **확장에 열려 있다**: 요구사항이 바뀌거나 새로운 기능이 추가될 때, 해당 기능을 확장(새 코드를 추가)하여 시스템 동작을 변경할 수 있어야 한다.

- **변경에 닫혀 있다**: 새로운 기능을 확장하거나 요구사항을 반영할 때, 기존의 잘 동작하던 코드는 수정하지 않아야 한다.

- 핵심 메커니즘: 추상화와 다형성.

- OCP 위반 사례:

  ```java
  public class PaymentService {
      public void processPayment(String paymentType, int amount) {
          if ("CREDIT_CARD".equals(paymentType)) {
              // 신용카드 결제
          } else if ("NAVER_PAY".equals(paymentType)) {
              // 네이버페이 결제
          } else if ("KAKAO_PAY".equals(paymentType)) {
              // 카카오페이 결제
          } ...
      }
  }
  ```

  - 새 결제 방식이 추가될 때마다, 위 코드에 분기 처리를 추가해주어야 함.

  👇
  OCP 준수 리팩토링

  ```java
  // 1. 변하지 않는 계약(인터페이스) 정의
  public interface PaymentMethod {
      void pay(int amount);
  }
  
  // 2. 확장에 열린 개별 구현체들
  public class CreditCardPayment implements PaymentMethod {
      @Override
      public void pay(int amount) {
          // 신용카드 결제 처리 로직
      }
  }
  
  public class KakaoPayPayment implements PaymentMethod {
      @Override
      public void pay(int amount) {
          // 카카오페이 결제 처리 로직
      }
  }
  
  // 3. 변경에 닫힌 서비스 클래스
  public class PaymentService {
      public void processPayment(PaymentMethod paymentMethod, int amount) {
          // 구체적인 결제 방식이 무엇인지 몰라도 인터페이스 호출만으로 결제 수행
          paymentMethod.pay(amount);
      }
  }
  ```

  - **개방(Open):** 새로운 결제 수단(예: `ApplePayPayment`)이 추가되더라도 `PaymentMethod`를 구현하는 새 클래스를 만들기만 하면 됨.
  - **폐쇄(Closed):** 새로운 결제 수단이 추가되어도 기존의 `PaymentService`나 다른 결제 구현체 코드는 단 한 줄도 손댈 필요가 없습니다.



#### 3. 리스코프 치환 원칙 (Liskov Substitution Principle)

- 하위 타입은 언제나 자신의 상위 타입으로 교체할 수 있어야 한다.
  즉, 프로그램에서 상위 클래스 타입의 객체 자리에 하위 클래스의 인스턴스를 갈아 끼워 넣어도 프로그램의 논리적 동작과 규약이 깨지지 않아야 함.

- LSP 위반 사례:
  ```java
  // 상위 클래스: 직사각형
  public class Rectangle {
      protected int width;
      protected int height;
  
      public void setWidth(int width) {
          this.width = width;
      }
  
      public void setHeight(int height) {
          this.height = height;
      }
  
      public int getArea() {
          return width * height;
      }
  }
  
  // 하위 클래스: 정사각형 (네 변의 길이가 같아야 하므로 둘 다 변경)
  public class Square extends Rectangle {
      @Override
      public void setWidth(int width) {
          this.width = width;
          this.height = width; // 정사각형 규칙 유지를 위해 세로도 변경
      }
  
      @Override
      public void setHeight(int height) {
          this.width = height; // 정사각형 규칙 유지를 위해 가로도 변경
          this.height = height;
      }
  }
  
  public class GeometryService {
      // 직사각형의 가로를 5, 세로를 10으로 설정해 넓이가 50이 되기를 기대하는 메서드
      public void resizeRectangle(Rectangle r) {
          r.setWidth(5);
          r.setHeight(10);
  
          // Rectangle 규약상 넓이는 50이어야 함
          if (r.getArea() != 50) {
              throw new IllegalStateException("넓이가 50이 아닙니다!");
          }
      }
  }
  ```

  - `Square`는 `Rectangle`의 자리에 대체되어 들어갔지만, 상위 클래스가 보장하던 행위 규약("가로와 세로는 독립적으로 변경된다")을 깨뜨렸음.



#### 4. 인터페이스 분리 원칙 (Interface Segregation Principle)

- 클라이언트는 자신이 사용하지 않는 메서드에 의존하지 않아야 한다.
  클라이언트가 실제로 필요로 하는 역할과 행위 단위로 인터페이스를 작게 쪼개라는 것.

- ISP 위반 사례:
  ```java
  // ISP 위반: 모든 기능을 한 인터페이스에 몰아넣음
  public interface MultiFunctionPrinter {
      void print();
      void scan();
      void fax();
  }
  
  // 최신 복합기: 모든 기능을 지원하므로 문제없음
  public class SmartOfficePrinter implements MultiFunctionPrinter {
      public void print() { System.out.println("출력"); }
      public void scan()  { System.out.println("스캔"); }
      public void fax()   { System.out.println("팩스 전송"); }
  }
  
  // 단순 프린터: scan과 fax 기능이 물리적으로 없음
  public class BasicPrinter implements MultiFunctionPrinter {
      public void print() {
          System.out.println("출력");
      }
  
      @Override
      public void scan() {
          // 쓰지도 않는 메서드를 억지로 구현하거나 예외를 던져야 함
          throw new UnsupportedOperationException("스캔 기능을 지원하지 않습니다.");
      }
  
      @Override
      public void fax() {
          throw new UnsupportedOperationException("팩스 기능을 지원하지 않습니다.");
      }
  }
  ```

  👇
  ISP 준수 리팩토링:

  ```java
  // 1. 단일 책임을 갖는 작고 구체적인 인터페이스들
  public interface Printable {
      void print();
  }
  
  public interface Scannable {
      void scan();
  }
  
  public interface Faxable {
      void fax();
  }
  
  // 단순 프린터: Printable만 구현
  public class BasicPrinter implements Printable {
      @Override
      public void print() {
          System.out.println("출력");
      }
  }
  
  // 복합기: 필요한 인터페이스들을 다중 구현
  public class SmartOfficePrinter implements Printable, Scannable, Faxable {
      @Override
      public void print() { System.out.println("출력"); }
  
      @Override
      public void scan()  { System.out.println("스캔"); }
  
      @Override
      public void fax()   { System.out.println("팩스 전송"); }
  }
  ```



#### 4. 의존 역전 원칙 (Dependency Inversion Principle)

- 고수준 모듈은 저수준 모듈의 구현에 의존해서는 안되며, 둘 모두 추상화에 의존해야 한다.
  자주 바뀌는 구체적인 부품(세부 구현)에 매달리지 말고, 변하지 않는 인터페이스(약속)를 바라보고 개발할 것.

- DIP 위반 사례:
  ```java
  public class Car {
      // 특정 회사 타이어에 강하게 묶여 있음
      private HankookTire tire = new HankookTire();
  
      public void run() {
          tire.rollHankook();
      }
  }
  ```

  👇
  DIP 준수 리팩토링:

  ```java
  // 1. 공통 규격 (인터페이스)
  public interface Tire {
      void roll();
  }
  
  // 2. 자동차는 어떤 타이어가 올지 몰라도 규격(Tire)만 바라봄
  public class Car {
      private Tire tire;
  
      // 외부에서 규격에 맞는 타이어를 끼워줌
      public Car(Tire tire) {
          this.tire = tire;
      }
  
      public void run() {
          tire.roll();
      }
  }
  
  // 3. 타이어 제조사들은 그 규약에 맞춰 만들기만 함
  public class HankookTire implements Tire {
      public void roll() { System.out.println("한국타이어 회전"); }
  }
  
  public class MichelinSnowTire implements Tire {
      public void roll() { System.out.println("미쉐린 스노우타이어 회전"); }
  }
  ```

  
