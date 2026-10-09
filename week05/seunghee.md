# 7장 스프링 삼각형과 설정 정보

## IoC (제어의 역전) / DI (의존성 주입) 
객체를 누가 만들고, 누가 연결하는가? 관점으로 보려고 한다.

책은 자동차-타이어 예시로 설명하고 있다.
```java
class Car {
  private Tire tire = new KoreaTire();
}
```
이 경우 `Car` 객체가 `KoreaTire` 객체를 생성하고 있다.
이를 우리는 아래와 같이 말한다.
> `Car`가 `KoreaTire`를 직접 선택한다. 직접 생성한다. `KoreaTire`에 의존한다.
> `Tire`를 바꾸려면 `Car` 코드를 고쳐야 해.

### 생성자를 통한 의존성 주입
반면 생성자를 통한 DI는 아래 코드와 같다.
```java
class Car {
  private Tire tire;

  Car(Tire tire){
    this.tire = tire;
  }
}
```

### 속성을 통한 의존성 주입
생성자가 아니라 setter 같은 속성 접근자 메서드를 통해서도 의존성을 주입할 수 있다.

```java
class Car {
  private Tire tire;

  void setTire(tire){
    this.tire = tire;
  }
}
```

```java
Car car = new Car();
car.setTire(new KoreaTire());
```
이것도 Car가 직접 KoreaTire를 생성하지 않고, 다른 객체에서 생성한 후 전달받으니 DI이다.
