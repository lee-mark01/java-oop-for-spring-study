# 5주차 — 스프링 삼각형과 설정 정보

## 범위

7장. 스프링 삼각형과 설정 정보

## 1. 스프링 삼각형

스프링의 핵심 프로그래밍 모델은 **IoC / DI, AOP, PSA**이다.

| 개념 | 역할 |
|---|---|
| IoC / DI | 객체를 생성하고 필요한 객체를 연결한다. |
| AOP | 여러 기능에 반복되는 공통 처리를 분리한다. |
| PSA | 서로 다른 기술을 일관된 방식으로 사용하게 한다. |

스프링은 이를 통해 객체가 자신의 핵심 역할에 집중하도록 돕는다.

POJO(Plain Old Java Object)는 특정 프레임워크의 상속 구조 등에 얽매이지 않는 일반적인 자바 객체를 뜻한다.

## 2. IoC / DI

### 2-1. 의존성과 직접 생성의 문제

의존성은 **어떤 객체가 자신의 일을 수행하기 위해 다른 객체를 사용하는 관계**이다.

```java
class StudyService {
    private EmailNotifier notifier = new EmailNotifier();
}
```

`StudyService`는 알림 객체를 사용하면서 생성까지 직접 한다.

문자 알림으로 변경하려면 서비스 내부의 `new EmailNotifier()`도 수정해야 한다.

`new`는 의존성을 보여주는 대표적인 코드지만 외부에서 객체를 전달받아 사용하는 경우에도 의존성은 존재한다.

| 관계 | 특징 | 예시 |
|---|---|---|
| 집합 관계 | 부분이 전체와 별도로 존재할 수 있다. | 스터디 모임과 회원 |
| 구성 관계 | 부분의 생명주기가 전체에 종속된다. | 문서와 문서 내부의 단락 |

### 2-2. DI — 의존성 주입

DI는 **필요한 객체를 직접 만들지 않고 외부에서 전달받는 방식**이다.

먼저 알림 기능의 공통 규약을 정의한다.

```java
interface Notifier {
    void send(String message);
}

class EmailNotifier implements Notifier {
    @Override
    public void send(String message) {
        System.out.println("이메일: " + message);
    }
}

class SmsNotifier implements Notifier {
    @Override
    public void send(String message) {
        System.out.println("문자: " + message);
    }
}
```

서비스는 구체적인 알림 방식 대신 `Notifier`를 사용한다.

### 2-3. 생성자 주입

객체를 생성할 때 생성자로 의존성을 전달한다.

```java
class StudyService {
    private final Notifier notifier;

    public StudyService(Notifier notifier) {
        this.notifier = notifier;
    }

    public void register(String title) {
        System.out.println("스터디 등록: " + title);
        notifier.send("새 일정이 등록되었습니다.");
    }
}
```

```java
StudyService service = new StudyService(new EmailNotifier());
service.register("스프링 공부");
```

문자 알림으로 바꾸려면 전달하는 객체만 변경한다.

```java
StudyService service = new StudyService(new SmsNotifier());
```

서비스는 알림 객체를 사용하고 외부 코드는 어떤 알림 객체를 전달할지 결정한다.

**장점**

- 필요한 의존성이 생성자에 드러난다.
- `final`로 의존성 참조의 재할당을 막을 수 있다.
- 테스트용 객체를 쉽게 전달할 수 있다.
- 구현을 교체해도 서비스 코드를 유지하기 쉽다.

| 설계 원칙 | 연결되는 부분 |
|---|---|
| DIP | 구체 클래스 대신 `Notifier`라는 추상화에 의존한다. |
| OCP | 새로운 알림 구현을 추가해도 기존 서비스의 수정을 줄인다. |

### 2-4. 수정자 주입

객체를 먼저 만든 뒤 `setter`로 의존성을 전달한다.

아래 코드는 앞의 생성자 주입 예제와 별개의 예제이다.

```java
class StudyService {
    private Notifier notifier;

    public void setNotifier(Notifier notifier) {
        this.notifier = notifier;
    }

    public void register(String title) {
        System.out.println("스터디 등록: " + title);

        if (notifier != null) {
            notifier.send("새 일정이 등록되었습니다.");
        }
    }
}
```

```java
StudyService service = new StudyService();
service.setNotifier(new EmailNotifier());
service.register("스프링 공부");
```

| 구분 | 생성자 주입 | 수정자 주입 |
|---|---|---|
| 주입 시점 | 객체 생성 시 | 객체 생성 후 |
| 주요 용도 | 필수 의존성 | 선택적 의존성 |
| 참조 교체 | `final`로 제한 가능 | 수정자로 가능 |
| 주입 전 상태 | 생성 시 전달하도록 요구 | 의존성이 없는 상태가 가능 |

필수 의존성에는 생성자 주입을 우선 고려한다.

A가 B를 필요로 하고 B도 A를 필요로 하는 **순환 의존성**은 생성자 주입으로 구성할 수 없다. 일부 환경에서는 수정자 주입으로 연결할 수 있지만 먼저 책임 분리를 검토하는 것이 좋다.

### 2-5. IoC — 제어의 역전

직접 작성하던 객체 생성과 연결을 스프링 컨테이너에 맡길 수 있다.

**객체 관리의 제어권이 외부로 넘어가는 것이 IoC이다.**

- IoC: 제어권을 외부에 맡기는 원리
- DI: 필요한 객체를 외부에서 전달하는 방식

DI는 IoC를 실현하는 방법 중 하나이며 스프링 없이도 사용할 수 있다.

| 용어 | 의미 |
|---|---|
| 스프링 컨테이너 | 객체의 생성, 연결, 생명주기를 관리한다. |
| 빈(Bean) | 컨테이너가 관리하는 객체이다. |
| 설정 정보 | 등록할 객체와 연결 관계를 알려준다. |
| `ApplicationContext` | 일반적으로 사용하는 컨테이너 인터페이스이다. |

## 3. 설정 정보와 자동 주입

### 3-1. XML 설정

XML의 `<beans>` 내부에서 빈과 의존 관계를 지정한다.

```xml
<bean id="emailNotifier" class="study.EmailNotifier"/>

<bean id="studyService" class="study.StudyService">
    <constructor-arg ref="emailNotifier"/>
</bean>
```

- `id`: 빈의 이름
- `class`: 생성할 클래스
- `constructor-arg`: 생성자에 전달할 값 또는 객체
- `ref`: 전달할 빈의 이름

수정자 주입은 다음처럼 표현한다.

```xml
<property name="notifier" ref="emailNotifier"/>
```

`setNotifier()`를 통해 `emailNotifier` 빈을 전달한다는 의미이다.

컨테이너에서 빈을 가져올 때는 `getBean()`을 사용한다.

```java
try (var context =
        new ClassPathXmlApplicationContext("study.xml")) {
    StudyService service =
        context.getBean("studyService", StudyService.class);
}
```

### 3-2. 자바 설정

```java
@Configuration
class StudyConfig {

    @Bean
    public Notifier notifier() {
        return new EmailNotifier();
    }

    @Bean
    public StudyService studyService(Notifier notifier) {
        return new StudyService(notifier);
    }
}
```

- `@Configuration`: 설정 클래스 표시
- `@Bean`: 반환하는 객체를 빈으로 등록

객체 생성 코드가 사라지는 것이 아니라 업무 객체에서 설정 코드로 이동한다.

### 3-3. 컴포넌트 스캔

클래스에 애너테이션을 붙이고 스캔 범위를 지정하면 빈으로 등록할 수 있다.

```java
@Configuration
@ComponentScan("study")
class StudyConfig {
}
```

| 애너테이션 | 주요 역할 |
|---|---|
| `@Component` | 일반 컴포넌트 |
| `@Service` | 업무 로직 |
| `@Repository` | 데이터 접근 |
| `@Controller` | 웹 요청 처리 |

빈 등록은 관리할 객체를 정하는 것이고, 의존성 주입은 필요한 객체를 연결하는 것이다.

### 3-4. `@Autowired`와 `@Resource`

`@Autowired`는 타입을 기준으로 주입 후보를 찾으며, 생성자·수정자·필드에 사용할 수 있다.

```java
@Autowired
public StudyService(Notifier notifier) {
    this.notifier = notifier;
}
```

생성자가 하나라면 `@Autowired`를 생략할 수 있다.

같은 타입의 빈이 여러 개라면 선택 기준이 필요하다.

```java
public StudyService(
        @Qualifier("smsNotifier") Notifier notifier) {
    this.notifier = notifier;
}
```

- `@Qualifier`: 타입에 맞는 후보를 좁힌다.
- `@Primary`: 기본으로 선택할 후보를 지정한다.

필드 주입도 가능하지만, 의존성이 생성자에 드러나지 않고 테스트용 객체를 전달하기 불편하다.

```java
@Autowired
private Notifier notifier;
```

`@Resource`는 주로 이름으로 빈을 찾는다.

```java
@Resource(name = "smsNotifier")
private Notifier notifier;
```

| 구분 | `@Autowired` | `@Resource` |
|---|---|---|
| 주된 기준 | 타입 | 이름 |
| 대상 지정 | `@Qualifier` | `name` |
| 생성자 사용 | 가능 | 불가능 |

`@Resource`에서 이름을 생략하면 필드명이나 속성명을 사용하며, 기본 이름의 빈이 없으면 타입 조회로 대체할 수 있다.

이름을 직접 지정하면 해당 이름의 빈을 찾는다. 최신 스프링 계열에서는 `jakarta.annotation.Resource`를 사용한다.

XML에 등록된 빈의 주입 애너테이션을 처리하려면 `<context:annotation-config/>` 같은 설정이 필요하다.

## 4. AOP — 공통 처리 분리

### 4-1. 핵심 관심사와 횡단 관심사

| 구분 | 의미 | 예시 |
|---|---|---|
| 핵심 관심사 | 기능의 본래 업무 | 스터디 등록, 가입 신청 |
| 횡단 관심사 | 여러 기능의 공통 처리 | 로깅, 시간 측정, 트랜잭션 |

AOP는 **공통 처리를 별도로 정의하고 적용 대상과 시점을 지정하는 방식**이다.

공통 함수와 달리 업무 코드에서 해당 처리를 직접 호출하지 않아도 적용할 수 있다.

### 4-2. 주요 용어

| 용어 | 의미 |
|---|---|
| Target | 부가 기능을 적용할 객체 |
| Join Point | 부가 기능을 적용할 수 있는 실행 지점 |
| Pointcut | 적용할 지점을 고르는 조건 |
| Advice | 실행 시점과 부가 작업 |
| Aspect | 관련 Pointcut과 Advice를 모은 모듈 |
| Advisor | Advice와 적용 조건을 묶는 스프링 구성 요소 |

스프링 AOP의 Join Point는 메서드 실행이다.

**Pointcut으로 선택한 실행 지점에 Advice를 적용한다.**

### 4-3. Advice 종류

| 애너테이션 | 실행 시점 |
|---|---|
| `@Before` | 실행 전 |
| `@AfterReturning` | 정상 반환 후 |
| `@AfterThrowing` | 예외를 던지며 종료한 후 |
| `@After` | 정상·예외 종료 모두 |
| `@Around` | 호출 전체를 감싸서 처리 |

`@After`는 `finally`와 비슷하다.

`@Around`는 실행 전후 처리와 대상 호출 여부 등을 제어할 수 있다.

### 4-4. 시간 측정 예제

```java
@Aspect
@Component
class TimingAspect {

    @Around("execution(* study.service..*(..))")
    public Object measure(ProceedingJoinPoint joinPoint)
            throws Throwable {
        long start = System.nanoTime();

        try {
            return joinPoint.proceed();
        } finally {
            System.out.println(
                "실행 시간: " + (System.nanoTime() - start) + "ns"
            );
        }
    }
}
```

- `@Aspect`: AOP 공통 처리 클래스
- `execution(...)`: 대상 메서드 선택 조건
- `proceed()`: 다음 Advice 또는 실제 대상 메서드로 실행 진행

위 Pointcut은 `study.service`와 하위 패키지의 메서드를 선택한다.

`JoinPoint`에서는 메서드와 인자 등의 정보를 확인하고, `ProceedingJoinPoint`는 `proceed()`도 제공한다.

일반 자바 설정에서는 `@EnableAspectJAutoProxy`, XML에서는 `<aop:aspectj-autoproxy/>`로 프록시 처리를 활성화할 수 있다.

### 4-5. 프록시를 통한 동작

프록시는 실제 객체 앞에서 요청을 먼저 받는 대리 객체이다.

시간 측정 예제의 순서는 다음과 같다.

1. 프록시가 호출을 받는다.
2. 시간 측정을 시작한다.
3. 실제 서비스 메서드를 실행한다.
4. 실행 시간을 출력하고 결과를 전달한다.

실제 메서드 본문을 직접 수정하기보다 호출을 감싸서 처리한다.

| 방식 | 특징 |
|---|---|
| JDK 동적 프록시 | 인터페이스 기반 |
| CGLIB 기반 프록시 | 대상 클래스를 상속 |

따라서 인터페이스가 반드시 필요한 것은 아니다.

**주의점**

- 같은 객체 내부의 직접 호출은 프록시를 우회한다.
- 직접 `new`로 만든 일반 객체에는 자동 적용되지 않는다.
- private 메서드는 일반적인 프록시 AOP로 가로챌 수 없다.
- 클래스 기반 프록시에서는 final 메서드에 제약이 있다.

## 5. 트랜잭션과 PSA

### 5-1. `@Transactional`과 AOP

스터디 정보와 개설자의 참여 정보를 함께 저장한다고 가정한다.

두 저장 작업을 하나의 처리 단위로 묶기 위해 트랜잭션을 사용한다.

```java
@Transactional
public void createStudy(String title, Long ownerId) {
    Long studyId = studyRepository.save(title);
    participantRepository.save(studyId, ownerId);
}
```

업무 메서드에는 저장 작업만 남고 프록시가 트랜잭션 처리를 담당한다.

새 트랜잭션을 시작하는 경우 실제 메서드 실행 후 정상 종료하면 커밋하고 롤백 규칙에 해당하는 예외가 발생하면 롤백한다.

기본 롤백 규칙은 다음과 같다.

| 예외 | 기본 처리 |
|---|---|
| `RuntimeException`, `Error` | 롤백 |
| 체크 예외 | 자동 롤백 대상이 아님 |

체크 예외도 롤백하려면 `rollbackFor = Exception.class` 등을 지정할 수 있다.

트랜잭션 관리 설정과 적절한 트랜잭션 매니저가 필요하며 같은 객체 내부의 직접 호출에는 기본 프록시 방식의 처리가 적용되지 않는다.

### 5-2. PSA — 일관성 있는 서비스 추상화

PSA는 **서로 다른 기술을 공통된 방식으로 사용하도록 추상화하는 것**이다.

JDBC와 JPA는 트랜잭션 API가 다르지만 스프링에서는 공통 추상화를 사용할 수 있다.

일반적인 명령형 트랜잭션의 공통 인터페이스는 `PlatformTransactionManager`이다.

| 구현체 | 담당 기술 |
|---|---|
| `DataSourceTransactionManager` | JDBC |
| `JpaTransactionManager` | JPA |
| `JtaTransactionManager` | JTA |

서비스는 `@Transactional`로 경계를 표현하고 실제 기술에 맞는 매니저를 설정으로 연결한다.

기술을 교체하면 저장소 구현과 설정 등은 바뀔 수 있다. PSA는 모든 수정을 없애는 것이 아니라 수정 범위를 줄인다.

### 5-3. 다른 추상화 사례

| 사례 | 공통 사용 방식 |
|---|---|
| JDBC | `Connection`, `Statement`, `ResultSet` |
| 스프링 OXM | `Marshaller`, `Unmarshaller` |
| 스프링 캐시 | 캐시 추상화와 관련 애너테이션 |

JDBC는 각 드라이버가 공통 규약을 구현하는 자바 표준 API이다. 서로 다른 기술을 공통 방식으로 연결한다는 점에서 어댑터 패턴과 연결해 이해할 수 있다.

OXM은 객체와 XML 사이의 변환이다.

- `Marshaller`: 객체 → XML
- `Unmarshaller`: XML → 객체

## 6. 세 개념의 연결

| 개념 | 스터디 서비스에서의 역할 |
|---|---|
| IoC / DI | 저장소와 알림 객체를 연결한다. |
| AOP | 시간 측정과 트랜잭션을 적용한다. |
| PSA | 기술별 트랜잭션 처리를 공통 방식으로 사용한다. |

`@Transactional`은 두 관점에서 이해할 수 있다.

- **AOP:** 업무 메서드 호출을 감싸 트랜잭션 처리를 적용한다.
- **PSA:** 기술마다 다른 트랜잭션 API를 공통 추상화로 사용한다.
