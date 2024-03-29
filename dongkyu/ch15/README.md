# 15 디자인패턴과 프레임워크

- 디자인 패턴
  - 소프트웨어 설계에서 반복적으로 발생하는 문제에 대해 반복적으로 적용할 수 있는 해결 방법
- 프레임워크
  - 설계 뿐만 아니라 설계와 코드를 재사용하기 위한 것
  - 애플리케이션 아키텍처를 구현 코드 형태로 제공
  - 각 애플리케이션 요구에 따라 적절히 커스터마이징할 수 있는 확장 포인트를 제공
- 디자인 패턴과 프레임워크 모두 협력을 일관성 있게 만들기 위한 방법

## 01 디자인 패턴과 설계 재사용

### 소프트웨어 패턴

- ‘GoF의 디자앤피턴’에 의해 패턴이 대중화된 후 수많은 패턴이 등장했다.
- 패턴이라는 거대한 숲 속에서 길을 잃지 않기 위해선 패턴의 정의보다 패턴이라는 용어 자체가 풍기는 뉘앙스를 이해해야 한다.
- 패턴의 핵심적인 특징
  - 반복적으로 발생하는 문제와 해법의 쌍으로 정의된다.
  - 패턴을 사용함으로써 이미 알려진 문제와 해법을 문서로 정리할 수 있고 다른 사람과 의사소통할 수 있다.
  - 패턴은 추상적인 원칙과 실제 코드 작성 사이의 간극을 메워주며 실질적 코드 작성을 돕는다.
  - 패턴의 요점은 패턴이 실무에서 탄생했다는 점

### 패턴 분류

- 패턴은 범위나 적용 단계에 따라 4가지로 나눌 수 있다.
  - 아키텍처 패턴
  - 분석 패턴
  - 디자인 패턴
  - 이디엄
- 아키텍처 패턴
  - 디자인 패턴의 상위 개념으로 소프트웨어 전체 구조를 결정하기 위해 사용할 수 있는 패턴
- 이디엄 패턴
  - 디자인 패턴의 하위 개념으로 특정 프로그래밍 언어에만 국한된 패턴
  - ex) C++의 COINT POINTER는 참조 카운트를 세는 기법인데 JVM의 GC에선 유용하지 않다.
- 분석 패턴
  - 위 패턴들이 기술적 문제를 해결하는데 초점을 둔다면 분석 패턴은 도메인 내의 개념적인 문제를 해결하는 데 초점을 맞춘다.
  - 하나의 도메인에 대해서만 적절할 수도 있고 여러 도메인에 걸쳐 적용할 수도 있다.

### 패턴과 책임-주도 설계

- 객체지향 설계에서 중요한 것
  - 올바른 책임을 올바른 객체에게 할당
  - 객체 간 유연한 협력 관계를 구축
- 패턴은 공통으로 사용할 수 있는 역할, 책임, 협력의 탬플릿
  - ex) STRATEGY 패턴: 다양한 알고리즘을 동적으로 교체할 수 있는 역할과 책임의 집합을 제공
- 패턴의 구성 요소는 클래스가 아닌 ‘역할’
  - 즉 패턴 탬플릿을 구성할 수 있는 다양한 방법이 존재한다느 사실을 암시
  - 하나의 객체가 복수의 역할을 수행하거나 다수의 클래스가 동일한 역할을 구현할 수도 있다.

### 캡슐화와 디자인 패턴

- 대부분의 디자인 패턴은 협력을 일관성 있고 유연하게 만드는 것을 목적으로 한다.
- 영화 예매 시스템에서 `Movie`가 `DiscountPolicy` 상속 계층을 합성 관계로 구성 → STRATEGY 패턴
  - 결합도는 낮지만 복잡도가 높다.
- 영화 예매 시스템에서 `Movie`를 상속 계층으로 구성하여 `AmountDiscountMovie`와 `PercentDiscountMovie`를 도출 → TEMPLATE METHOD 패턴
  - 결합도는 높아지지만 복잡도는 낮아진다.
- 영화 예매 시스템에서 `OverlappedDiscountPolicy`를 통해 개별 객체와 복합 객체라는 객체 수와 관련된 변경을 캡슐화 → COMPOSITE 패턴
- 핸드폰 과금 시스템 설계에서 객체의 행동을 동적으로 추가할 수 있게 합성을 사용한 것 → DECORATOR 패턴
  - 선택적인 행동 개수와 순서에 대한 변경을 캡슐화

### 패턴은 출발점이다

- 패턴이 설계의 목표가 돼서는 안 된다.
- 패턴을 사용하면서 부딪히는 대부분의 문제는 패턴의 맹목적 사용 시에 발생한다.
  - ‘망치를 들면 모든 것이 못으로 보인다’
- 타당한 이유 없이 패턴을 적용하면 설계 의도를 이해하지 못하게 된다.
- 패턴을 가장 효과적으로 적용하는 방법은 패턴을 지향하거나 패턴 목표로 리팩터링하는 것이다.
  - 패턴이 적용된 최종 결과를 이해하는 것 보다 패턴을 목표로 리팩터링하는 것이 가치있다.

## 02 프레임워크와 코드 재사용

### 코드 재사용 대 설계 재사용

- 디자인 패턴을 적용하기 이해선 프로그래밍 언어 특성에 맞춰 가공하고 코드를 재작성해야 한다는 단점이 있다.
  - 디자인 패턴은 프로그래밍 언어에 독립적인 설계 아이디어를 제공하기 때문
- 컴포넌트 기반의 재사용
  - 부품을 조립해서 제품을 만들듯이 기존 컴포넌트를 조립해서 애플리케이션을 구축
  - 아이디어 자체는 이상적이지만 현실적이진 않다.
  - 애플리케이션과 도메인의 다양성으로 인해 문제가 아주 비슷한 경우는 거의 없기 때문
- 가장 이상적인 혀애는 설계 재사용과 코드 재사용을 적절하게 조합하는 것
- 프레임워크
  - 추상 클래스나 인터페이스를 정의하고 인스턴스 사이의 상호작용을 통해 전체 혹은 일부를 구현해 놓은 재사용 가능한 설계
  - 개발자가 현재 요구사항에 맞게 커스터마이징할 수 있는 애플리케이션의 골격
  - 프레임워크는 코드를 재사용함으로써 설계 아이디어를 재사용한다.

### 상위 정책과 하위 정책으로 패키지 분리하기

- 프레임워크의 핵심은 추상화이다.
  - 추상 클래스와 인터페이스는 일관성 있는 협력을 만드는 핵심 재료다.
  - 협력을 구현하는 코드 안의 의존성은 가급적이면 추상화를 향하도록 해야 한다.
- 객체지향 이전에는 상위 레벨 모듈이 하위 모듈에, 상위 정책이 세부적인 사항에 의존했다.
  - 상위 정책은 상대적으로 안정적이지만 세부 사항은 자주 변경된다.
  - 상위 정책은 세부 사항에 비해 재사용될 가능성이 높다.
  - 상위 정책이 세부 사항에 의존하게 되면 상위 정책의 재사용성이 낮아진다.
- 의존성 역전 원칙으로 상위 정책과 세부 사항 모두 추상화에 의존하게 만들어야 한다.
  - 의존성 역전 원칙 관점에서 세부 사항은 ‘변경’을 의미
- 변하는 부분과 변하지 않는 부분을 별도 패키지로 분리해야 한다.
  - 변하는 것과 변하지 않는 것들을 서로 다른 주기로 배포할 수 있도록 ‘배포 단위로 분리해야 한다.
  - 세부 사항(변경)이 상위 정책(추상화)에 의존하도록 패키지 간 의존성을 구성해야 한다.

### 제어 역전 원리

- 의존성 역전 원리는 프레임워크의 가장 기본적인 설계 메커니즘이다.
  - 프레임워크가 애플리케이션에 속하는 서브 클래스를 호출한다.
- 제어 역전 원리 (Inversion of Control)
  - 프레임워크를 사용하면 개별 애플리케이션에서 프레임워크로 제어 흐름 주체가 이동한다.
  - 프레임워크에선 일반적 해결책만 제공하고 애플리케이션에 따라 달라질 수 있는 동작은 비워둔다.
  - 협력을 제어하는 주체가 바로 프레임워크
