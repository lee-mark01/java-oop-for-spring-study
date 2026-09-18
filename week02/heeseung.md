# 2주차

## 범위

3장. 자바와 객체 지향

## 학습 내용

### 1. 클래스와 객체

>  클래스와 객체의 관계는 분류와 사물의 관계이다.

#### 클래스

- **사전적 정의:** 분류, 집합. 같은 특성을 지닌 여러 객체를 총칭하는 집합의 개념.
- **객체지향에서의 실질적 정의:**
  - 객체의 설계도. 객체를 정의해놓은 것.
  - 데이터와 함수의 결합.(속성과 기능의 정의)
  - 사용자 정의 타입.

#### 객체

- ≒ 오브젝트, 인스턴스
- 속성과 기능의 집합.

```java
// 클래스
class Person {
  // 속성(멤버 변수)
	String name;
  int age;
  boolean isBald;
  
  // 기능(메서드)
  void say() {
    System.out.printf("야호!! 나는 %s입니다! 나는 %d살입니다!", name, age);
  }
}

class Sample {
  public static void main(String[] args) {
    Person koo = new Person(); // 인스턴스 생성
    
    // koo 인스턴스 속성값 초기화
    koo.name = "구희승";
    koo.age = 99;
    koo.isBald = false;
    
    koo.say();
  }
}
```



### 2. 객체지향의 4대 특성

#### 추상화

- 객체의 모델링. 그 결과는 클래스.
- 구체적인 것을 분해해서 관심 영역(애플리케이션 경계, 컨텍스트)에 있는 특성만 가지고 재조합하는 것.

 #### 상속

- 상위 클래스의 특성을 재사용/확장하는 것.
- 상속을 통해 클래스를 작성하면, 코드 중복을 줄일 수 있어 유지보수에 용이.
  클래스간 상속관계를 맺으면 하위 클래스의 공통적인 부분은 상위 클래스에서 관리하고, 하위 클래스는 자신에게 정의된 멤버들만 관리하면 됨.
- extends 를 사용.

```java
class Animal {
  String name;
  int leg;
  
  void speak() {
    System.out.printf("다리 개수가 %d개인 동물입니다. 이름: %s\n", leg, name);
  }
}

class Dog extends Animal {
  void bark() {
    System.out.println("왈왈! ");
  }
}

class Sample {
  public static void main(String[] args) {
    Dog doge = new Dog();

    doge.name = "doge";
    doge.leg = 4;
    doge.speak(); // 다리 개수가 4개인 동물입니다. 이름: doge
    doge.bark(); // 왈왈!
  }
}
```

#### 다형성

- 상위 클래스 타입의 참조변수로 하위 클래스의 인스턴스를 참조할 수 있다.
  이 경우, 하위 클래스만 갖고 있는 멤버는 사용 불가하다.

```java
class Sample {
  public static void main(String[] args) {
    Animal doge = new Dog();

    doge.name = "doge";
    doge.leg = 4;
    doge.speak(); // 다리 개수가 4개인 동물입니다. 이름: doge
    doge.bark(); // 오류 발생.
  }
}
```

- 사용 편의성을 위해 오버로딩과 오버라이딩 제공.

  > **오버로딩**: 같은 이름의 메서드를 다른 인자 목록으로 여러 개 중복 정의하는 것.
  >
  > **오버라이딩**: 상위 클래스의 메서드를 같은 이름, 같은 인자로 재정의하는 것.

#### 캡슐화

- 외부로부터의 데이터 보호를 위해 접근 제어자 사용.
- public - 모두 접근 가능
  protected - 상속 / 같은 패키지 내의 클래스에서 접근 가능
  (default) - 같은 패키지 내의 클래스에서 접근 가능
  private - 본인만 접근 가능



### 3. Call by Reference

- 참조에 의한 호출. Call by Address(주소에 의한 호출) 라고도 함.
- 참조 자료형 변수는 값을 주소(포인터)로 판단하기 때문에 참조된 객체의 값이 변경된다.

```java
class CallByRef {
  public static void main(String[] args) {
    Animal doge = new Dog();
    doge.name = "도지";
    doge.leg = 4;
    doge.speak(); // 다리 개수가 4개인 동물입니다. 이름: doge
    updateName(doge);
    doge.speak(); // 다리 개수가 5개인 동물입니다. 이름: 춘식이
  }
  
  private static void updateData(Animal animal) {
    animal.name = "춘식이";
    animal.leg = 5;
  }
}
```

