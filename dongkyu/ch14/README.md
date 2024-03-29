# 14 일관성 있는 협력

- 객체지향 패러다임 장점이 설계를 재사용할 수 있다는 것이다.
- 재사용을 위해서는 객체들의 협력 방식을 일관성 있게 만들어야 한다.

## 01 핸드폰 과금 시스템 변경하기

### 기본 정책 확장

- 확장된 핸드폰 과금 시스템의 요금 정책
  - 고정 요금 방식 - a초당 b원
  - 시간대별 방식 - a시부터 b시까지 c초당 d원
  - 요일별 방식 - 평일에는 a초당 b원, 공휴일은 a초당 c원
  - 구간별 방식 - 초기 a분 동안 b초당 c원, a분 ~ d분까지 b초당 d원, d분 초과 시 b초당 e원

```
Phone -----> RatePolicy <-----------
                  ↑                |
              ---- -------         |
BasicRatePolicy           AdditionalRatePolicy
      ↑                                ↑
FixedFeePolicy                 RateDiscountablePolicy
TimeOfDayDiscountPolicy        TaxablePolicy
DayOfWeekDiscountPolicy
DurationDiscountPolicy
```

### 고정 요금 방식 구현하기

```java
public class FixedFeePolicy extends BasicRatePolicy {
    private Money amount;
    private Duration seconds;
    // ...
    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}
```

### 시간대별 방식 구현하기

- 시간대별 방식은 통화 시작 시간 및 종료 시간뿐 아니라 시작 일자와 종료 일자도 함께 고려해야 한다.
  - 몇일에 걸쳐 통화를 한 경우도 고려해야 하기에 날짜 정보도 필요하다.
  - ‘10시 ~ 15시’라는 정보만으로는 ‘3일 10시 ~ 5일 15시’ 동안의 요금을 계산할 수 없다.
- 기간을 편하게 구현하기 위해 `DateTimeInterval` 클래스 추가
  - `Call`의 원래 인스턴스였던 `from`과 `to`를 하나의 값객체로 묶을 수 있다.

```java
public class Call {
	private DateTimeInterval interval;

	public Call(LocalDateTime from, LocalDateTime to) {
		this.interval = DateTimeInterval.of(from, to);
	}
	// ...
}
```

- 시간대별 방식 로직을 다음 두 단계로 나눠 구현할 필요가 있다.
  - 통화 기간을 일자별로 분리
  - 일자별로 분리된 기간을 다시 시간대별로 분리 후 각 기간에 대해 요금 계산
- 각 단계를 정보 전문가에게 할당할 수 있다.
  - `Call` - 통화 기간 정보를 알고 있는 전문가
  - `DateTimeInterval` - 기간을 처리하는 방법에 대한 전문가
  - `TimeOfDayDiscountPolicy` - 시간대별 기준을 잘 알고 있는 시간대별 분할 작업의 전문가

```java
public class TimeOfDayDiscountPolicy extends BasicRatePolicy {
    private List<LocalTime> starts = new ArrayList<LocalTime>();
    private List<LocalTime> ends = new ArrayList<LocalTime>();
    private List<Duration> durations = new ArrayList<Duration>();
    private List<Money>  amounts = new ArrayList<Money>();

    @Override
    protected Money calculateCallFee(Call call) {
        Money result = Money.ZERO;
        for(DateTimeInterval interval : call.splitByDay()) { // call에게 일자별 통화 기간 분리를 요청
            for(int loop=0; loop < starts.size(); loop++) {
                LocalTime from = from(interval, starts.get(loop));
                LocalTime to = to(interval, ends.get(loop));

                long intervalSeconds = Duration.between(from, to).getSeconds();
                long criteriaSeconds = durations.get(loop).getSeconds();

                Money intervalFee = amounts.get(loop);
                result.plus(intervalFee.times(intervalSeconds / criteriaSeconds));
            }
        }
        return result;
    }

    private LocalTime from(DateTimeInterval interval, LocalTime from) {
        LocalTime intervalFrom = interval.getFrom().toLocalTime();
        return intervalFrom.isBefore(from) ? from : intervalFrom;
    }

    private LocalTime to(DateTimeInterval interval, LocalTime to) {
        LocalTime intervalTo = interval.getTo().toLocalTime();
        return intervalTo.isAfter(to) ? to : intervalTo;
```

### 요일별 방식 구현하기

- 요일별 규칙을 다르게 설정할 수 있다.
- 4개의 `List`로 규칙을 정의한 시간대별 방식과 다르게 `DayOfWeekDiscountRule` 클래스로 규칙을 관리

```java
public class DayOfWeekDiscountRule {
    private List<DayOfWeek> dayOfWeeks = new ArrayList<>();
    private Duration duration = Duration.ZERO;
    private Money amount = Money.ZERO;
    // 생성자
    public Money calculate(DateTimeInterval interval) {
        if (dayOfWeeks.contains(interval.getFrom().getDayOfWeek())) {
            return amount.times(interval.duration().getSeconds() / duration.getSeconds());
        }
        return Money.ZERO;
    }
}

public class DayOfWeekDiscountPolicy extends BasicRatePolicy {
    private List<DayOfWeekDiscountRule> rules = new ArrayList<>();
    // 생성자
    @Override
    protected Money calculateCallFee(Call call) {
        Money result = Money.ZERO;
        for(DateTimeInterval interval : call.getInterval().splitByDay()) {
            for(DayOfWeekDiscountRule rule: rules) { 
                result.plus(rule.calculate(interval));
            }
        }
        return result;
    }
}
```

### 구간별 방식 구현하기 전 기존 방식 문제점

- 지금까지의 세 클래스 구현의 큰 문제점은 유사한 문제를 해결하고 있음에도 불구하고 설계 일관성이 없다는 것이다.
- 설계의 비일관성은 두 가지 상황에서 발목 잡는다.
  - 새로운 구현을 추가해야 하는 상황
  - 기존의 구현을 이해해야 하는 상황
- 새로운 구현을 추가해야 하는 상황
  - 앞서 구현한 고정 요금, 시간대별, 요일별 방식은 각각 규칙을 관리하는 방법이 달랐다.
    - 하나의 규칙, `List`로 관리 or 별도 클래스로 관리
  - 구간별 방식을 새로 구현해야할 때 어떤 방식으로 구현해도 문제 없어 보이고 점점 일관성이 어긋나게 된다.
- 코드를 이해하기 어렵다.
  - 요일별 방식 구현을 이해한다고 시간대별 방식을 바로 이해할 수 없다.
  - 서로 다른 구현 방식은 코드 이해에 오히려 방해가 된다.
- 결론적으로 유사한 기능을 서로 다른 방식으로 구현해서는 안 된다.

## 02 설계에 일관성 부여하기

- 일관성 있는 설계를 위해서는
  - 다양한 설계 경험을 익혀야 한다.
  - 널리 알려진 디자인 패턴을 학습하고 변경이라는 문맥 안에서 적용해 봐야 한다.
- 협력을 일관성 있게 만들기 위해서 따르면 좋은 지침
  - 변하는 개념을 변하지 않는 개념으로부터 분리
  - 변하는 개념을 캡슐화

### 조건 로직 대 객체 탐색

- 영화 예매 시스템 설계까 일관된 설계의 좋은 예다.
  - `Movie`가 `DiscountPolicy`를 의존하여 `Screening`을 전달하여 요금 계산을 요청
  - `DiscountPolicy`는 `DiscountCondition` 목록을 의존하여 각 조건에 해당하는 `Screening`을 매칭
- `Movie`, `DiscountPolicy`, `DiscountCondition` 사이의 합력은 변경을 기준으로 클래스를 분리한 좋은 예
  - 이 협력 방식을 이해하면 새로운 할인 정책을 마주치더라도 쉽게 이해할 수 있다.

### 캡슐화 다시 살펴보기

- 캡슐화란 변하는 어떤 것이든 감추는 것이다.
  - 코드 수정으로 인한 파급효과를 제어할 수 있는 모든 기법
- 다양한 종류의 캡슐화
  - 데이터 캡슐화
    - 클래스 내부 데이터를 은닉
  - 메서드 캡슐화
    - 메서드 가시성을 `public`일 필요가 없는 것들은 공개하지 않음으로써 은닉할 수 있다.
  - 객체 캡슐화
    - 객체가 다른 객체를 의존할 때 접근자를 `private`으로 은닉함으로 두 객체 간 관계를 변경하더라도 외부의 영향을 미치지 않을 수 있다.
  - 서브타입 캡슐화
    - 객체가 한 객체를 의존할 때 의존되는 객체의 서브타입에 대해선 의존하는 객체는 알지 못한다.
- 변경을 캡슐화하기 위해 서브타입 캡슐화와 객체 캡슐화를 조합할 수 있다.
  - 변하는 부분을 분리해서 타입 계층을 구성
  - 변하지 않는 부분의 일부로 타입 계층을 합성

## 03 일관성 있는 기본 정책 구현하기

### 변경 분리하기

- 고정요금 방식 외에 시간대별, 요일별, 구간별 방식은 어느 정도 유사한 형태를 가진다.
  - 고정요금 방식 - [단위시간]당 [요금]원
  - 시간대별 방식 - [시작시간]~[종료시간]까지 [단위시간]당 [요금]원
  - 요일별 방식 - [요일]별 [단위시간]당 [요금]원
  - 구간별 방식 - [통화구간]동안 [단위시간]당 [요금]원
- 시간대별, 요일별, 구간별 방식 공통점
  - 기본 정책은 한 개 이상의 ‘규칙’으로 구성
  - 하나의 ‘규칙’은 ‘적용조건’과 ‘단위요금’의 조합
- 변하지 않는 ‘규칙’으로부터 변하는 ‘적용조건’을 분리해야 한다.
  - 모든 규칙에 ‘적용조건’이 포함된다는 사실은 고정
  - 실제 조건의 세부 내용은 다르다.

### 변경 캡슐화하기

- 변하지 않는 ‘규칙’에서 변하는 ‘적용조건’을 분리해서 추상화해야 한다.
  - 추상화 후 시간대별, 요일별, 구간별 방식을 서브타입으로 구성 (서브타입 캡슐화)
  - 규칙이 적용조건을 표현하는 추상화를 합성 관계로 연결 (객체 캡슐화)
- 하나의 기본 정책은 하나 이상의 ‘규칙’들로 구성된다.
  - `BasicRatePolicy` —의존→ `List<FeeRule>`
- ‘규칙’은 ‘적용조건’과 ‘단위요금’을 가진다.
  - `FeeRule`에 `feePerDuration` 인스턴스 변수 존재
  - `FeeRule` —의존→ `FeeCondition`
- ‘적용조건’에 시간대별, 요일별, 구간별 서브타입이 존재
  - `TimeOfDayFeeCondition`, `DayOfWeekFeeCondition`, `DurationFeeCondition`

### 협력 패턴 설계하기

1. `BasicRatePolicy`가 `calculateFee` 메시지 수신
1. `BasicRatePolicy`는 전체 `Call`에 대해 요금을 계산
2. 각 `Call` 별로 `FeeRule`의 `calculateFee` 메서드를 실행
1. `FeeCondition` - 전체 통화 시간을 각 ‘규칙’의 ‘적용조건’을 만족하는 구간들로 나누기
2. `FeeRule` - 분리된 통화 구간에 ‘단위요금’을 적용하여 요금 계산

### 추상화 수준에서 협력 패턴 구현하기

```java
// Call의 통화 기간 중 '적용조건'을 만족하는 기간의 목록을 반환
public interface FeeCondition {
    List<DateTimeInterval> findTimeIntervals(Call call);
}

public class FeeRule {
    private FeeCondition feeCondition;
    private FeePerDuration feePerDuration;
    // ...
    public Money calculateFee(Call call) {
        return feeCondition.findTimeIntervals(call)
                .stream()
                .map(each -> feePerDuration.calculate(each)) // 단위요금 계산
                .reduce(Money.ZERO, (first, second) -> first.plus(second));
    }
}

public final class BasicRatePolicy implements RatePolicy {
    private List<FeeRule> feeRules = new ArrayList<>();
    // ...
    @Override
    public Money calculateFee(Phone phone) {
        return phone.getCalls()
                .stream()
                .map(call -> calculate(call))
                .reduce(Money.ZERO, (first, second) -> first.plus(second));
    }

    private Money calculate(Call call) {
        return feeRules
                .stream()
                .map(rule -> rule.calculateFee(call))
                .reduce(Money.ZERO, (first, second) -> first.plus(second));
    }
}
```

### 구체적인 협력 구현하기

```java
// 시간대별 정책
public class TimeOfDayFeeCondition implements FeeCondition {
    private LocalTime from;
    private LocalTime to;
    // ...
    @Override
    public List<DateTimeInterval> findTimeIntervals(Call call) {
        return call.getInterval().splitByDay()
                .stream()
                .filter(each -> from(each).isBefore(to(each)))
                .map(each -> DateTimeInterval.of(
                                LocalDateTime.of(each.getFrom().toLocalDate(), from(each)),
                                LocalDateTime.of(each.getTo().toLocalDate(), to(each))))
                .collect(Collectors.toList());
    }
    // ...
}

// 요일별 정책
public class DayOfWeekFeeCondition implements FeeCondition {
    private List<DayOfWeek> dayOfWeeks = new ArrayList<>();
    // ...
    @Override
    public List<DateTimeInterval> findTimeIntervals(Call call) {
        return call.getInterval()
                .splitByDay()
                .stream()
                .filter(each ->
                        dayOfWeeks.contains(each.getFrom().getDayOfWeek()))
                .collect(Collectors.toList());
    }
}

// 구간별 정책
public class DurationFeeCondition implements FeeCondition {
    private Duration from;
    private Duration to;
    // ...
    @Override
    public List<DateTimeInterval> findTimeIntervals(Call call) {
        if (call.getInterval().duration().compareTo(from) < 0) {
            return Collections.emptyList();
        }
        return Arrays.asList(DateTimeInterval.of(
                call.getInterval().getFrom().plus(from),
                call.getInterval().duration().compareTo(to) > 0 ?
                        call.getInterval().getFrom().plus(to) :
                        call.getInterval().getTo()));
    }
}
```

- 변경을 캡슐화하면서 얻을 수 있는 장점
  - 변하지 않는 부분을 재사용할 수 있다.
  - 기능 추가 시 변하는 부분만 구현하면 되기에 구현 난이도가 감소한다.
  - 구조를 강제할 수 있기에 설계 일관성이 무너지지 않는다.
  - 설계가 일관적이게 되면서 코드 이해가 편해진다.
- 유사한 기능에 유사한 협력 패턴을 적용하는 것은 개념적 무결성을 유지할 수 있는 가장 효과적인 방법
  - 개념적 무결성 = 일관성

### 협력 패턴에 맞추기

- 고정요금 정책은 위 3개 정책과는 다르다.
  - ‘규칙’이라는 개념이 필요하지 않고 ‘단위요금’만 있으면 된다.
- 이런 경우 또 다른 협력 패턴을 만드는 것보다 조금은 어색하더라도 일관성을 유지하는 설계를 선택하는 것이 좋다.
  - 개념적 무결성을 무너뜨리는 것보다 약간의 부조화를 수용하는 편이 낫다.

```java
public class FixedFeeCondition implements FeeCondition {
    @Override
    public List<DateTimeInterval> findTimeIntervals(Call call) {
        return Arrays.asList(call.getInterval()); // 적용조건이 의미 없기에 call의 전체 기간을 반환
    }
}
```

- 협력은 고정된 것이 아니기에 지속적인 개선이 필요하다.
  - 새로운 요구사항이 추가되며 일관성에 금이 가는 경우가 많다.
  - 요구사항의 변경에 따라 협력 역시 지속적으로 개선해야 한다.
