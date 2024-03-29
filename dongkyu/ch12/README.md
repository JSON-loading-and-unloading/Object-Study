# 12장 다형성

- 코드 재사용을 목적으로 한 상속은 유연하지 못한 설계를 낳는다.
  - 코드 재사용 목적으로 상속을 사용하지 말아야 한다.
- 상속은 다형성을 구현하는 가장 일반적인 방법이지만 상속 외에도 다형성을 구현할 수 있다.

## 01 다형성

- 컴퓨터 과학에서 다형성이란
  - 하나의 추상 인터페이스에 대해 서로 다른 구현을 연결할 수 있는 능력
- 다형성 (Polymorphism) 분류
  - 유니버셜 (Universal)
    - 매개변수 (Parametric)
    - 포함 (Inclusion)
  - 임시 (Ad Hoc)
    - 오버로딩 (Overloading)
    - 강제 (Coercion)
- 오버로딩 다형성
  - 동일한 이름 메서드가 서로 다른 타입 파라미터를 받는 경우
- 강제 다형성
  - 자동적인 타입 변환이나 직접 구현한 타입 변환을 이용해 동일한 연산자를 다양한 타입에 사용할 수 있는 방식
  - ex) `+`는 피연산자가 모두 정수일 땐 정수 덧셈 연산자, 문자열일 땐 연결 연산자로 동작한다.
- 매개변수 다형성
  - 제네릭 프로그래밍과 관련성이 높다.
  - 클래스 인스턴스 변수나 메서드 매개변수 타입을 임의로 지정하고 사용 시점에 구체 타입으로 지정
  - ex) `List<T>`
- 포함 다형성
  - 서브타입 다형성이라 부르며 동일한 메시지가 수신 객체 타입에 따라 다르게 수행되는 능력

## 02 상속의 양면성

- 객체지향 패러다임의 근간은 데이터와 행동을 객체라는 하나의 실행 단위 안으로 통합하는 것
- 상속의 목적은 코드 재사용이 아닌 다형성을 가능하게 하는 타입 계층을 구축하기 위한 것
  - 상속은 단순히 데이터와 행동의 관점에서 코드를 재사용하는 메커니즘이 아니다.
- 상속 메커니즘을 이해하기 위해 아래 개념을 이해해야 한다.
  - 업캐스팅
  - 동적 메서드 탐색
  - 동적 바인딩
  - self 참조
  - super 참조

### 상속을 사용한 강의 평가

- 아래처럼 수강생들 성적을 계산하는 프로그램을 구현해본다.
  - `Pass:3 Fail:2, A:1 B:1 C:1 D:0 F:2`
- 그런데 이미 아래와 같은 결과를 출력하는 `Lecture` 클래스가 존재하는 상황
  - `Pass:3 Fail 2`

```java
public class Lecture {
    private int pass; // 이수 여부 판단할 기준 점수
    private String title; // 강의 제목
    private List<Integer> scores = new ArrayList<>(); // 학생들의 성적

    public Lecture(String title, int pass, List<Integer> scores) {
        this.title = title;
        this.pass = pass;
        this.scores = scores;
    }

    public double average() {
        return scores.stream().mapToInt(Integer::intValue).average().orElse(0);
    }

    public String evaluate() {
        return String.format("Pass:%d Fail:%d", passCount(), failCount());
    }

    // ...
}
```

- 상속을 이용하면 새로운 기능을 빠르고 쉽게 추가할 수 있을 것이다.
  - `Lecture`를 상속한 `GradeLecture`
  - `Lecture`의 `evaluate`를 오버라이딩하여 추가된 기능을 구현할 수 있다.

```java
public class GradeLecture extends Lecture {
    private List<Grade> grades;

    public GradeLecture(String name, int pass, List<Grade> grades, List<Integer> scores) {
        super(name, pass, scores);
        this.grades = grades;
    }

    @Override
    public String evaluate() {
        return super.evaluate() + ", " + gradesStatistics();
    }

    private String gradesStatistics() {
        return grades.stream().map(grade -> format(grade)).collect(joining(" "));
    }

    private String format(Grade grade) {
        return String.format("%s:%d", grade.getName(), gradeCount(grade));
    }

    private long gradeCount(Grade grade) {
        return getScores().stream().filter(grade::include).count();
    }

    public double average(String gradeName) {
        return grades.stream()
                .filter(each -> each.isName(gradeName))
                .findFirst()
                .map(this::gradeAverage)
                .orElse(0d);
    }

    private double gradeAverage(Grade grade) {
        return getScores().stream()
                .filter(grade::include)
                .mapToInt(Integer::intValue)
                .average()
                .orElse(0);
    }
}
```

### 데이터 관점의 상속

- 자식 클래스 인스턴스 안에 부모 클래스 인스턴스 변수를 포함하는 것으로 볼 수 있다.
- 자식 클래스에선 부모 클래스 인스턴스 변수를 자동으로 포함한다.

### 행동 관점의 상속

- 부모 클래스가 정의한 일부 메서드를 자식 클래스의 메서드로 포함시키는 것
  - 부모 클래스의 모든 퍼블릭 메서드는 자식 클래스의 퍼블릭 인터페이스에 포함된다.
- 행동 관점에서 상속과 다형성을 이해하려면 상속으로 연결된 클래스 사이 메서드 탐색 과정을 이해해야 한다.
  - ex) 런타임에 자식 클래스에 정의되지 않은 메서드가 있는 경우 부모에서 탐색
- 객체와 메서드의 메모리 할당
  - 객체의 경우 서로 다른 상태를 저장하도록 인스턴스 별로 독립적인 메모리를 할당 받는다.
  - 메서드의 경우 인스턴스끼리 공유가 가능하기에 한 번만 메모리에 로드하고 포인터를 갖게 하는 것이 경제적이다.
- ex) 두 개의 `Lecture` 인스턴스 생성 후의 메모리 상태
  - 각 인스턴스는 별도의 메모리를 할당
  - `Lecture` 클래스는 하나만 할당되고 인스턴트들이 class라는 이름의 포인터로 이에 접근
  - `Lecture` 클래스는 parent라는 이름의 포인터로 `Object`에 접근
- 구체적인 구현 방법이나 메모리 구조는 언어나 플랫폼에 따라 상이하다.

## 03 업캐스팅과 동적 바인딩

### 같은 메시지, 다른 메서드

- 교수별 강의에 대한 성적 통계를 계산하는 기능을 추가해보자.
- `Professor`의 `compileStatistics` 메서드로 통계 정보를 생성한다.
  - `Lecture`의 `evaluate`와 `average` 메서드를 호출

```java
public class Professor {
    private String name;
    private Lecture lecture;

    public Professor(String name, Lecture lecture) {
        this.name = name;
        this.lecture = lecture;
    }

    public String compileStatistics() {
        return String.format("[%s] %s - Avg: %.1f", name,
                lecture.evaluate(), lecture.average());
    }
}
```

- `Professor`가 `Lecture`를 생성자로 받기에 `GradeLecture`를 전달 받아도 기능 수행에는 문제가 없다.

```java
Professor professor = new Professor("다익스트라", 
                        new Lecture("알고리즘",
                            70,
                            List.of(81, 95, 75, 50, 45)));

String statistics = professor.compileStatistics();
// [다익스트라] Pass:3 Fail:2 - AVG: 69.2

Professor professor = new Professor("다익스트라", 
                        new GradeLecture("알고리즘",
                            70,
                            List.of(new Grade("A", 100, 95), ...),
                            List.of(81, 95, 75, 50, 45)));

String statistics = professor.compileStatistics();
// [다익스트라] Pass:3 Fail:2, A:1 B:1 C:1 D:1 F:1 - AVG: 69.2
```

### 업캐스팅

- 상속을 이용하면 부모 클래스 인스턴스에 전송할 메시지를 자식 인스턴스에게 전송할 수 있다.
- 모든 객체지향 언어는 명시적 타입 변환 없이 부모 클래스 타입 참조변수에 자식 클래스 인스턴스 대입을 허용한다.
- ex) `Lecture lecture = new GradeLecture(…)`
- 다운 캐스팅
  - 반대로 부모 클래스 인스턴스를 자식 클래스 인스턴스 타입으로 변환하려면 명시적 다운캐스팅이 필요하다.
  - ex) `GradeLecture gradeLecture = (GradeLecture)lecture;`

### 동적 바인딩

- 정적 바인딩 (static binding)
  - 전통적 언어에선 호출될 함수가 컴파일타임에 결정된다.
  - 초기 바인딩 (early binding), 컴파일타임 바인딩(compile-time binding)이라고도 부른다.
- 동적 바인딩 (dynamic binding)
  - 실행될 메서드를 런타임에 결정하는 방식
  - 선언된 변수 타입이 아닌 수신 객체 타입에 따라 실행되는 메서드가 결정되는 것
  - 컴파일타임이 아닌 런타임에 메시지를 처리할 메서드를 결정하기 때문에 가능

## 04 동적 메서드 탐색과 다형성

- 객체지향 시스템에서 실행할 메서드 선택 규칙
  - 메서드 수신 객체는 자신을 생성한 클래스에 적절한 메서드가 있는지 검사
  - 메서드를 못찾으면 적합한 메서드를 찾을 때까지 상속 계층을 탐색
  - 상속 계층의 최상위 클래스에서도 메서드를 못찾으면 예외와 함께 탐색을 중단
- self 참조 (self reference)
  - 객체가 메시지를 수신하면
    - 컴파일러는 self 참조라는 임시 변수를 자동으로 생성 후
    - 메시지를 수신한 객체를 가르키도록 한다.
  - 동적 메서드 탐색은 self가 가르키는 객체를 시작으로 상속 계층 역방향으로 이루어진다.
  - 탐색이 끝나면 self는 자동으로 소멸된다.
  - 정적 타입 언어에서 self 참조는 this라 한다.
    - ex) C++, 자바, C#
- 동적 메서드 탐색의 두 가지 원리
  - 자동적인 메시지 위임
    - 자신이 이해할 수 없는 메시지를 전송받으면 부모에게 처리를 위임
  - 동적인 문맥
    - 메시지를 수신하여 어떤 메서드로 실행할 것인지는 런타임에 결정
    - 탐색 경로는 self 참조를 이용해 결정한다.

### 자동적인 메시지 위임

- 상속을 이용하면 메시지는 상속 계층을 따라 부모에게 자동으로 위임된다.
  - 상속 계층 정의란 메서드 탐색 경로를 정의하는 것과 동일
- 메서드 오버라이딩
  - 자식 클래스의 메서드가 부모 클래스 메서드보다 먼저 탐색되기 때문에 벌어지는 현상
- 메서드 오버로딩
  - 메서드 시그니처가 완전히 동일하지 않다면 상속 계층에 걸쳐 부모 메서드와 자식 메서드가 공존할 수도 있다.
  - 상속 계층 사이의 같은 이름의 다른 시그니처의 메서드를 정의하는 것도 엄연히 메서드 오버로딩이다.
  - C++에선 상속 계층 사이의 오버로딩을 지원하지 않기는하다.

### 동적인 문맥

- 메시지 수신 객체에 따라 실행될 메서드가 동적으로 바뀐다.
  - 이러한 동적 문맥을 결정하는 것은 메시지 수신 객체를 가리키는 self 참조
  - self 참조가 문맥을 결정하기에 어떤 메서드가 실행될지 예상하기 어렵게 한다.

```java
public class Lecture {
    public String stats() {
        return String.format("ㅅTitle: %s, Evaluation Method: %s", title, getEvaluationMethod());
    }

    public String getEvaluationMethod() {
        return "Pass or Fail";
    }
}
```

- 위 코드에서 `stats()` 메서드는 자신의 `getEvaluationMethod()`를 호출하고 있다.
  - 더 정확히는 현재 객체에게 `getEvaluationMethod` 메시지를 전송하는 것
  - 현재 객체란 self가 가리키는 객체

```java
public class GradeLecture extends Lecture {
    @Override
    public String getEvaluationMethod() {
        return "Grade";
    }
}
```

- `GradeLecture`가 `getEvaluationMethod`를 오버라이드하면 self 참조에 따라 `GradeLecture`로 생성한 인스턴스는 바로 위 메서드가 실행되게 된다.
  - 메서드 탐색이 시작되는 self 참조가 `GradeLecture`부터이기 때문

### 이해할 수 없는 메시지

- 메서드 탐색을 최상위 부모 클래스까지 해도 처리할 수 없는 메시지라면 어떻게 할까
- 정적 타입 언어와 이해할 수 없는 메시지
  - 컴파일타임에 자신이 이해할 수 없는 메시지인지 파악해서 컴파일 에러를 발생시킨다.
  - 유연성이 부족하지만 안정적이다.
- 동적 타입 언어와 이해할 수 없는 메시지
  - 실제로 코드를 실행해보기 전에는 메시지 처리 가능 여부를 판단할 수 없다.
  - 실행 시점에 최상위 상속 계층에서 예외가 던져진다.
  - 동적 타입 언어에선 이해할 수 없는 메시지에 응답할 수 있는 메서드를 구현하는 방법이 존재한다.
  - 이러한 유연성은 코드를 이해하고 수정하기 어렵게하고 디버깅을 복잡하게 하기도 한다.

### self 대 super

- 자식 클래스에서 부모 클래스의 구현을 재사용해야 하는 경우 super 참조를 사용할 수 있다.
- 엄밀히 말해 super는 부모 클래스의 메서드를 호출하는 것이 아니다.
- super 참조는 이 클래스의 부모 클래스에서부터 메서드 탐색을 시작하라는 의미이다.
  - 즉 상속 계층 최상위까지도 올라갈 수 있다.
  - 이를 super 전송(super send)라 부른다.
- self 전송은 어떤 클래스에서 메시지 탐색이 시작될지 알지 못하는 반면 super는 항상 해당 클래스 부모부터 탐색을 시작한다.
  - self 전송은 실행될 메서드가 동적으로 결정되지만 super 전송의 경우 컴파일 시점에 결정해 놓을 수 있다.

## 05 상속 대 위임

### 위임과 self 참조

- 상속 계층 객체들 사이에서 객체들은 self 참조를 공유한다.
  - 메서드 탐색 중 자식, 부모 클래스 인스턴스가 동일한 self를 바라본다.
- 자신이 수신한 메시지를 다른 객체에게 동일하게 전달해서 처리하는 것을 위임(delegation)이라 부른다.
  - 위임은 항상 현재 실행 문맥을 가리키는 self 참조를 인자로 전달한다.
- self 참조의 전달은 결과적으로 자식 클래스 인스턴스와 부모 클래스 인스턴스 사이에 동일한 실행 문맥을 공유할 수 있게 해준다.

### 프로토타입 기반의 객체지향 언어

- 클래스가 존재하지 않고 객체만 존재하는 프로토타입 기반 객체지향 언어에서 객체 간 위임을 통해 상속을 구현할 수 있다.
- 자바 스크립트에서는 prototype 체인으로 연결된 객체 사이에 메시지를 위임함으로써 상속을 구현할 수 있다.
- 객체지향 패러다임에서 클래스는 필수 요소가 아닌 것이다.
  - 상속 이외에도 다형성을 구현할 수 있기에
- 중요한 것은 클래스 기반 상속과 객체 기반 위임 사이에 기본 개념과 메커니즘을 공유한다는 점이다.