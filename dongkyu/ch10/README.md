# 10 상속과 코드 재사용

## 01 상속과 중복 코드

### DRY 원칙

- 중복 코드는 변경을 방해한다.
  - 중복 코드를 수정하려면 어떤 코드가 중복인지 찾아야 한다.
  - 중복 코드 묶음을 찾았다면 모든 코드를 일관되게 수정해야 하나다.
- 중복 여부 판단 기준은 변경이다.
  - 중복 코드를 결정 짓는 기준은 코드의 모양이 아니다.
  - 요구사항 변경 시 두 코드를 함께 수정해야 한다면 중복이다.
  - 모양은 단지 중복의 징후
- DRY 원칙
  - Don’t Repeat Yourself
  - 모든 지식은 시스템 내에서 단일하고 애매하지 않고 정말 믿을 만한 표현 양식을 가져야 한다.
  - DRY 원칙은 한 번, 단 한번(Once and Only Once) 원칙 또는 단일 지점 제어(Single-Point Control) 원칙이라고도 부른다.

### 중복과 변경

- 한 달에 한 번씩 가입자별로 전화 요금을 계산하는 간단한 애플리케이션 예제로 중복 코드의 문제를 이해해보자.
  - 전화 요금 계산 규칙은 통화 시간을 단위 시간당 요금으로 나눠주면 된다.

```java
public class Call { // 개별 통화 기간을 저장
    private LocalDateTime from;
    private LocalDateTime to;

    public Call(LocalDateTime from, LocalDateTime to) {
        this.from = from;
        this.to = to;
    }

    public Duration getDuration() {
        return Duration.between(from, to);
    }

    public LocalDateTime getFrom() {
        return from;
    }
}

public class Phone { // Call의 목록을 관리하며 통화 요금을 계산
    private Money amount; // 단위 요금
    private Duration seconds; // 단위 시간
    private List<Call> calls = new ArrayList<>();

    public Phone(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }

    public void call(Call call) {
        calls.add(call);
    }

    public List<Call> getCalls() {
        return calls;
    }

    public Money getAmount() {
        return amount;
    }

    public Duration getSeconds() {
        return seconds;
    }

    // seconds 당 amount씩 부과되는 요금제에서 통화 요금을 계산
    // 통화 시간 / 단위 시간 * 단위 요금
    public Money calculateFee() { 
        Money result = Money.ZERO;
        for(Call call : calls) {
            result = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
        }
        return result;
    }
}
```

- ‘심야 할인 요금제’라는 새로운 방식을 추가해야 하는 상황
- 요구사항을 구현하는 가장 빠른 방법은 코드를 복사해서 `NightlyDiscountPhone`이라는 새 클래스를 만드는 것이다.

```java
public class NightlyDiscountPhone {
    private static final int LATE_NIGHT_HOUR = 22;

    private Money nightlyAmount; // 10시 이후 적용할 통화 요금
    private Money regularAmount; // 10시 이전 적용할 통화 요금
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();

    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds) {
        this.nightlyAmount = nightlyAmount;
        this.regularAmount = regularAmount;
        this.seconds = seconds;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;
        for(Call call : calls) {
            if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                result = result.plus(nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            } else {
                result = result.plus(regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            }
        }
        return result;
    }
}
```

### 중복 코드 수정하기

- 새로운 요구사항으로 통화 요금에 부과할 세금을 계산해야 한다.
  - 세금을 추가하려면 `Phone`과 `NightlyDiscountPhone` 양쪽 모두 수정해야 한다.

```java
public class Phone {
    // ...
    private double taxRate;

    public Phone(Money amount, Duration seconds, double taxRate) {
        // ...
        this.taxRate = taxRate;
    }

    // ...

    public Money calculateFee() {
        Money result = Money.ZERO;
        for(Call call : calls) {
            result = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
        }
        return result.plus(result.times(taxRate));
    }
}

public class NightlyDiscountPhone {
    // ...
    private double taxRate;

    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds, double taxRate) {
        // ...
        this.taxRate = taxRate;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;
        for(Call call : calls) {
            if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                result = result.plus(nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            } else {
                result = result.plus(regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            }
        }
        return result.minus(result.times(taxRate));
    }
}
```

- 중복 코드의 단점
  - 많은 코드 더미에서 중복 코드 파악이 쉽지 않다.
  - 중복은 항상 함께 수정해야 하기에 실수가 버그로 이어질 가능성이 높다.
  - 중복 코드를 서로 다르게 수정하기가 쉽다.
    - `Phone`에서는 `plus` 메서드로 세금을 더했지만 `NightlyDiscountPhone`에선 `minus`를 호출하고 있다.

### 타입 코드 사용하기

- 두 클래스의 중복을 제거하는 한 방법은 클래스를 하나로 합치는 것이다.
- 요금제를 구분하는 타입 코드를 추가하고 타입에 따라 로직을 분기한다.
- 하지만 타입 코드를 사용하는 클래스는 낮은 응집도와 높은 결합도라는 문제에 시달리게 된다.

```java
public class Phone {
    private static final int LATE_NIGHT_HOUR = 22;
    enum PhoneType { REGULAR, NIGHTLY }

    private PhoneType type;
    
    // ...

    public Money calculateFee() {
        Money result = Money.ZERO;

        for(Call call : calls) {
            if (type == PhoneType.REGULAR) {
                result = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            } else {
                if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                    result = result.plus(nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
                } else {
                    result = result.plus(regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
                }
            }
        }
        return result;
    }
}
```

### 상속을 이용해서 중복 코드 제거하기

```java
public class NightlyDiscountPhone extends Phone {
    private static final int LATE_NIGHT_HOUR = 22;

    private Money nightlyAmount;

    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds) {
        super(regularAmount, seconds);
        this.nightlyAmount = nightlyAmount;
    }

    @Override
    public Money calculateFee() {
        Money result = super.calculateFee();
        Money nightlyFee = Money.ZERO;
        for(Call call : getCalls()) {
            if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                nightlyFee = nightlyFee.plus(
                        getAmount().minus(nightlyAmount).times(
                                call.getDuration().getSeconds() / getSeconds().getSeconds()));
            }
        }
        return result.minus(nightlyFee);
    }
}
```

- `NightlyDiscountPhone`이 `Phone`을 상속하면 중복을 제거할 수 있다.
- `calculateFee` 메서드
  - 10시 이전 요금은 부모 클래스의 `caculateFee`로 계산하고 10시 이후 통화 요금을 빼주고 있다.
  - 개발자가 10시 이전 요금을 `Phone`에서 처리하기로 결정했기 때문에 10시 이전 요금 계산 시 필요한 `regularAmount`와 `seconds`를 `Phone`에게 전달한 것
  - 상속을 염두하지 않고 설계된 클래스를 상속으로 개발하면 개발자의 가정을 이해하기 전에는 코드를 이해하기 어렵다.
- 상속으로 코드를 재사용하려면 부모 클래스 개발자가 세운 가정이나 추론 과정을 명확히 이해해야 한다.
  - 높은 결합도
  - 구현에 대한 정확한 지식을 가져야 함

### 강하게 결합된 Phone과 NightlyDiscountPhone

- 부모, 자식 클래스 상이의 결합이 문제인 이유는 너무 강하게 결합되어 있어 나타난다.
- 세금 계산 요구사항이 추가된 경우를 살펴보자.

```java
public class Phone {
    // ...
    public Money calculateFee() {
        // ...
        return result.plus(result.times(taxRate));
    }
}

public class NightlyDiscountPhone extends Phone {
    // ...
    @Override
    public Money calculateFee() {
        // ...
        return result.minus(nightlyFee.plus(nightlyFee.times(getTaxRate())));
    }
}
```

- 상속을 사용했더라도 세금을 계산하는 유사한 로직이 또 중복으로 나타난다.
- 이는 `NightlyDiscountPhone`이 `Phone`의 구현에 너무 강하게 결합되어 있기 때문에 발생하는 문제다.

> 상속을 위한 경고 1 - 자식 클래스 메서드 안에서 super 참조를 이용해 부모를 직접 호출할 경우 두 클래스는 강하게 결합된다. super 호출을 제거할 수 있는 방법을 찾아 결합도를 제거하라.
>

## 02 취약한 기반 클래스 문제

- 취약한 기반 클래스 문제
  - Fragile Base Class Problem, Brittle Base Class Problem
  - 상속 관계로 연결된 자식 클래스가 부모 클래스 변경에 취약해지는 현상
  - 캡슐화를 약화시키고 결합도를 높인다.
- 상속 관계를 추가할 수록 전체 시스템 결합도가 높아진다.
  - 자식 클래스의 기능 확장엔 용이하지만 부모 클래스를 점진적으로 개선하는 것은 어렵게 만든다.
  - 최악의 경우 모든 자식 클래스를 수정하고 테스트해야할 수도 있다.

### 불필요한 인터페이스 상속 문제

- 자바 초기 버전에서 상속을 잘못 사용한 대표적인 사례
  - `java.util.Properties`
  - `java.util.Stack`
- `Stack`은 스택 자료 구조를 구현한 클래스
  - 임의의 위치에서 요소를 추출하고 삽입할 수 있는 `Vector` 리스트 자료 구조를 상속
  - `Vector`는 임의의 위치에 요수를 삽입, 추출할 수 있기에 퍼블릭 인터페이스로 `add`와 `remove`가 존재한다.
  - `Vector`를 상속한 `Stack`에도 당연히 `add`와 `remove`가 노출되어 있어 스택의 규칙을 위반할 수 있게 된다.
  - ex) `stack.add(0, “4th”)`
- 인터페이스의 설계는 제대로 쓰기엔 쉽게, 엉터리로 쓰기엔 어렵게 만들어야 한다.

> 상속을 위한 경고2 - 상속받은 부모 클래스 메서드가 자식 클래스 내부 구조에 대한 규칙을 깨트릴 수 있다.
>

### 메서드 오버라이딩 오작용 문제

- 이펙티브 자바에서 `HashSet` 구현에 강하게 결합된 `InstrumentHashSet` 클래스를 소개한다.
  - [아이템18. 상속보다는 컴포지션을 사용하라](https://ldk980130.github.io/TIL/java/effective-java/item-18.html)

> 상속을 위한 경고 3 - 자식 클래스가 부모 클래스의 메서드를 오버라이딩할 경우 부모 클래스가 자신의 메서드를 사용하는 방법에 자식 클래스가 결합될 수 있다.
>

- 상속은 코드 재사용을 위해 캡슐화를 희생한다.

### 부모 클래스와 자식 클래스의 동시 수정 문제

- 부모 클래스를 수정할 때 자식 클래스도 함께 수정해야하는 경우가 있다.
  - 부모 클래스 메서드를 오버라이딩하거나 불필요한 인터페이스를 상속받지 않았음에도
- 결합도란 다른 대상에 대해 알고 있는 지식의 양이다.
  - 코드의 재사용을 위한 상속은 부모 자식을 강하게 결합시켜 함께 수정시켜야 하는 상황도 빈번하다.

> 상속을 위한 경고 4 - 클래스를 상속하면 결합도로 인해 자식 클래스와 부모 클래스 구현을 영원히 변경하지 않거나 자식 클래스와 부모 클래스를 동시에 변경하거나 둘 중 하나를 선택할 수밖에 없다.
>

## 03 Phone 다시 살펴보기

### 추상화에 의존하자

- 자식 클래스가 부모 클래스의 구현이 아닌 추상화에 의존하도록 만들어야 한다.
- 코드 중복 제거를 위해 상속을 도입할 때 두 가지 원칙
  - 두 메서드가 유사하게 보인다면 차이점을 메서드로 추출하라.
    - 메서드 추출을 통해 두 메서드를 동일한 형태로 보이도록 만들 수 있다.
  - 부모 클래스 코드를 하위로 내리지 말고 자식 클래스 코드를 상위로 올려라
    - 재사용성과 응집도 측면에서 띄어나다.

### 차이를 메서드로 추출하라

- 변하는 것으로부터 변하지 않는 것을 분리하라
- 변하는 부분을 찾고 이를 캡슐화하라
- `Phone`과 `NightlyDiscountPhone`의 경우
  - 통화 요금을 계산하는 로직이 다르다.
  - `Call`을 순회하며 결과를 집계하는 로직은 같다.

```java
public class Phone {
    // ...
    public Money calculateFee() {
        Money result = Money.ZERO;
        for(Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }
        return result;
    }

    private Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}

public class NightlyDiscountPhone {
    // ...
    public Money calculateFee() {
        Money result = Money.ZERO;
        for(Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }
        return result;
    }

    private Money calculateCallFee(Call call) {
        if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
            return result.plus(nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
        } else {
            return result.plus(regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
        }
    }
}
```

### 중복 코드를 부모 클래스로 올려라

- 추상 클래스를 추가하여 공통 부분을 부모 클래스로 이동시킨다.
- 공통 코드를 옮길 때 메서드를 먼저 이동시켜야 그에 필요한 인스턴스 변수를 컴파일 에러를 통해 알 수 있다.

```java
public abstract class AbstractPhone {
    private List<Call> calls = new ArrayList<>();

    public Money calculateFee() {
        Money result = Money.ZERO;
        for(Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }
        return result;
    }

    abstract protected Money calculateCallFee(Call call);
}

public class Phone extends AbstractPhone {
    private Money amount;
    private Duration seconds;

    // ...

    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}

public class NightlyDiscountPhone extends AbstractPhone {
    private static final int LATE_NIGHT_HOUR = 22;

    private Money nightlyAmount;
    private Money regularAmount;
    private Duration seconds;

    // ...

    @Override
    protected Money calculateCallFee(Call call) {
        if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
            return nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        } else {
            return regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        }
    }
}
```

- 위 설계는 자식 클래스들이 추상화에 의존하게 된다.

### 추상화가 핵심이다

- 추상화에 의존하는 설계에서 각 클래스들은 서로 다른 변경 이유를 가진다.
  - `AbstractPhone`: 전체 통화 목록을 계산하는 방법이 바뀔 경우
  - `Phone`: 일반 요금제의 통화 한 건을 계산하는 방식이 바뀔 경우
  - `NightlyDiscountPhone`: 심야 할인 요금제의 통화 한 건을 계산하는 방식이 바뀔 경우
  - 하나의 변경 이유만을 가지기에 응집도가 높다.
- 자식 클래스들은 부모 클래스의 구체적 구현이 아닌 추상화에 의존한다.
  - 정확히는 추상 메서드인 `calculateCallFee`
  - 메서드 시그니처가 변경되지 않는 한 부모 클래스 내부 구현이 바뀌어도 자식 클래스는 영향 받지 않는다.
- 의존성 역전 원칙을 준수한다.
  - 추상화에 의존하기 때문
- OCP를 준수한다.
  - 새 자식 클래스를 만들고 추상 메서드만 오버라이드하면 된다.

### 의도를 드러내는 이름 선택하기

- `Phone`은 일반 요금제와 관련된 내용이라는 것을 명시적으로 전달하지 못한다.
  - `NightlyDiscountPhone`은 심야 요금제라는 것이 명확하다.
  - `Phone`을 `RegularPhone`으로 변경하는 것이 적절하다.
  - `AbstactPhone`이 `Phone`이 되면 명확해진다.

### 세금 추가하기

- 수정된 코드가 더 변경에 용이할 지는 해보기 전까진 장담할 수 없다.
- 세금은 모든 요금에 공통으로 적용돼야 하기에 추상 클래스 `Phone`을 수정하면 자식 클래스가 공유할 수 있을 것이다.

```java
public abstract class Phone {
    private double taxRate;
    private List<Call> calls = new ArrayList<>();

    public Phone(double taxRate) {
        this.taxRate = taxRate;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;
        for(Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }
        return result.plus(result.times(taxRate));
    }

    protected abstract Money calculateCallFee(Call call);
}
```

- `Phone`에 `taxRate` 새로운 변수를 추가했기에 자식 클래스 생성자 역시 수정해야 한다.
  - 각 자식 클래스 생성자에서 `taxRate`를 추가적으로 받고 `super`를 호출해야 한다.
- 클래스 사이의 상속은 행동뿐만 아니라 인스턴스 변수에 대해서도 결합되게 만든다.
  - 행동만 변경된다면 상속 계층의 각 클래스들을 독립적으로 진화시킬 수 있다.
  - 하지만 변수가 추가된다면 상속 계층 전반에 걸친 변경을 유발한다.
- 그래도 인스턴스 초기화 로직을 변경하는 것이 동일한 코드를 중복시키는 것보다는 현명하다.
- 상속으로 인한 클래스 간 결합을 피할 수 있는 방법은 없지만 추상화를 통해 상속 계층 전체에 걸쳐 부작용이 퍼지지 않게 막을 수 있다.

## 04 차이에 의한 프로그래밍

- 차이에 의한 프로그래밍 (programming by difference)
  - 기존 코드와 다른 부분만 추가함으로써 애플리케이션 기능을 확장하는 방법
- 차이에의한 프로그래밍의 목표는 중복 코드를 제거하고 재사용하는 것
- 재사용 가능한 코드란 심각한 버그가 존재하지 않는 코드
- 객체지향에서 중복 코드 제거 및 재사용을 위한 가장 유명한 방법은 상속
- 상속이 코드 재사용 측면에서 매우 강력한 도구이지만 오용과 남용은 확장하기 어렵게 만든다.
- 상속의 단점을 피하면서 코드를 재사용하는 방법이 바로 합성이다.