# 오브젝트 챕더13 서브클래싱과 서브타이핑

### 상속에 대한 해묵은 불신과 오해를 풀기 위해서는 상속이 두 가지 용도로 사용된다는 사실을 이해해야 한다.

1. **타입 계층을 구현**
    
    부모 클래스는 자식 클래스의 일반화이고 자식 클래스는 부모 클래스의 특수화이다.
    
2. **코드 재사용**
    
    점진적으로 어플리케이션의 기능을 확장할 수 있지만, 이 경우 부모와 자식이 강하게 결합되어 수정하기 어려운 코드를 만들 가능성이 높다.
    

우리는 상속을 사용하는 일차적인 목표로 코드 재사용이 아닌 타입 계층을 구현하도록 해야 한다.

그렇다면 타입 계층은 무엇일까?

## 타입

### 개념 관점의 타입과 프로그래밍 언어 관점의 타입

개념 관점으로 바라보면 우리가 인지하는 세상의 사물의 종류를 타입이라고 한다.

즉, 우리가 인식하는 객체들에 적용하는 개념이나 아이디어를 가리켜 타입이라고 부른다.

어떤 대상이 타입으로 분류될 때 그 대상을 타입의 **인스턴스**라고 부르며 일반적으로 타입의 인스턴스를 **객체**라고 부른다.

프로그래밍 언어 관점으로 타입을 보면 연속적인 비트에 의미와 제약을 부여하기 위해서 사용된다.

즉 프로그래밍 언어 관점에서 타입은 비트 묶음에 의미를 부여하기 위해 정의된 제약과 규칙을 가리키는 것이다.

### 객체지향 패러다임 관점의 타입

이제 위에서 보았던 두 정의를 객체지향 패러다임 관점에서 조합해보면, 프로그래밍 언어의 관점에서 타입은 호출 가능한 오퍼레이션의 집합을 정의하고, 객체지향 프로그래밍에서 오퍼레이션은 객체가 수신할 수 있는 메시지를 의미한다. 따라서 객체의 타입이란 객체가 수신할 수 있는 메시지의 종류를 정의하는 것이다.

즉, 이것은 퍼블릭 인터페이스이다. (객체가 수신할 수 있는 메시지의 집합)

따라서 객체지향 프로그래밍에서 타입을 정의하는 것은 객체의 퍼블릭 인터페이스를 정의하는 것과 동일하다.

> 정의를 해보자면, 객체지향 프로그래밍 관점에서 타입은 객체의 퍼블릭 인터페이스가 객체의 타입을 결정하며 동일한 퍼블릭 인터페이스를 제공하는 객체들은 동일한 타입으로 구분된다.
> 

## 타입 계층

### 타입 사이의 포함관계

타입은 객체들의 집합이기 때문에 다른 타입을 포함하는 것이 가능하다.

타입 안에 포함된 객체들을 좀 더 상세한 기준으로 묶어서 새로운 타입을 정의하면 이 새로운 타입은 자연스럽게 기존 타입의 부분집합이 되는 것이다.

즉, 객체지향 언어 타입에는 클래스 기반 언어 타입과 프로토타입 기반 언어 타입이 포함되며, 객체지향 언어 타입은 프로그래밍 언어 타입에 포함되는 것이다.

또한 타입이 다른 타입에 포함될 수 있기 때문에 동일한 인스턴스가 하나 이상의 타입으로 분류되는 것도 가능하다.

자바를 생각해보면, 프로그래밍 언어 인 동시에 객체지향 언어이고 동시에 더 세부적으로는 클래스 기반 언어 타입에 속하게 된다.

이렇게 타입은 다른 타입에 속하기도 하고 다른 타입을 포함하기도 하는데, 이때 다른 타입을 포함하는 타입을 일반적이라고 하며, 포함되는 타입을 더 특수하다고 한다.

이때 일반적인 타입을 **슈퍼타입** 이라고 부르고 특수한 타입을 **서브타입** 이라고 부른다.

### 이제 일반화와 특수화를 정의해보자.

내연의 관점에서 일반화와 특수화를 우선 생각해보면,

- 일반화 : 어떤 타입의 정의를 더 보편적이고 추상적으로 만드는 과정
- 특수화 : 일반적인 타입의 정의를 더 구체화 한 것 (설명을 덧붙여 구체화)

외연의 관점에서 일반화와 특수화를 생각해보면

- 일반화 : 특수한 타입의 인스턴스 집합을 포함하는 슈퍼셋이다.
- 특수화 : 일반적인 타입의 인스턴스 집합에 포함된 서브셋이다.

여기서 슈퍼셋과 서브셋이 슈퍼타입과 서브타입의 유래이다.

### 객체지향 프로그래밍과 타입 계층

객체의 타입을 결정하는 것은 퍼블릭 인터페이스이다.

이때 퍼블릭 인터페이스 관점에서 슈퍼타입과 서브타입을 정의할 수 있다.

- 슈퍼타입 : 서브타입이 정의한 퍼블릭 인터페이스를 일반화 시켜 상대적으로 범용적이고 넓은 의미로 정의한 것
- 서브타입 : 슈퍼타입이 정의한 퍼블릭 인터페이스를 특수화 시켜 상대적으로 구체적이고 좁은 의미로 정의한 것

**중요한 것은 서브타입의 인스턴스는 슈퍼타입의 인스턴스로 간주될 수 있다는 것으로 이것이 상속과 다형성의 관계를 이해하기 위한 출발점이다.**

## 서브클래싱과 서브타이핑

상속을 이용해 타입 계층을 구현하는 것은 부모 클래스가 슈퍼타입의 역할을 자식 클래스가 서브타입의 역할을 수행하도록 클래스 사이의 관계를 정의하는 것이다.

그렇다면 타입 계층을 구현할 때 지켜야 하는 제약사항이 무엇이 있을지 클래스와 상속의 관점에서 살펴보자.

### 언제 상속을 사용해야 하는가

상속의 올바른 용도는 위에서 설명한 바와 같이 코드의 재사용이 아닌 타입 계층을 구현하는 것이다.

그렇다면 어떤 조건을 지켜야 올바른 상속을 사용한 것일까? 아래의 질문에 대답을 해보고 모두 YES인지 확인하자.

1. 상속 관계가 is-a 관계를 모델링 하는가?
    - [자식 클래스]는 [부모 클래스]다. 라고 말해도 이상하지 않아야 한다.
2. 클라이언트 입장에서 부모 클래스의 타입으로 자식 클래스를 사용해도 무방한가?
    - 상속 계층을 사용하는 클라이언트의 입장에서 부모 클래스와 자식 클래스의 차이를 몰라야 한다.
        
        이를 자식과 부모 사이의 행동 호환성이라고 한다.
        

설계 관점에서 상속을 적용할지 결정하기 위해서는 2번에 집중하는 것이 중요하다고 한다.

2번이 만족되면 1번 또한 만족되기 때문이다.

1번에 집중하다 보면 잘못될 수 있는데, 단적인 예시를 살펴보자.

### 펭귄과 새가 있다면, 펭귄은 새인가? 라는 질문을 할 수 있다.

이것은 잘못되었는가? 잘못되지 않았다.

하지만 기대되는 행동으로 바라보면 잘못되었다. 왜냐하면 새는 날 수 있지만 펭귄은 날 수 없기 때문이다.

이는 is-a라는 말을 너무 단편적으로 받아들일 경우에 어떤 혼란이 발생할 수 있는지 잘 보여준다.

**이제 위의 질문을 행동 호환성으로 살펴보자.**

펭귄과 새는 서로 다른 행동 방식을 보이기 때문에 이 둘을 동일한 타입 계층으로 묶어서는 안된다.

행동 호환성으로 따지기 위해서는 클라이언트 관점에서 두 타입이 동일하게 행동할 것이라고 기대하고 바라볼 수 있어야 한다는 것이다.

새는 날지만 펭귄은 날 수 없기 때문에 이 둘의 행동 호환성이 맞지 않을 수 있다는 것이다.

### 클라이언트의 기대에 따라 계층 분리하기

펭귄과 새 사이에 행동 호환성이 맞지 않지만, 몇가지 코드(try-catch, 예외 등등) 를 추가해서 억지로 상속 관계를 유지할 수 있을 것이다.

하지만 또 다른 새가 상속 계층에 추가된다면 코드가 추가되는 등 변경이 되게 될 것이다.

이는 개방-폐쇄 원칙을 위배한다.

이렇듯 행동 호환성을 만족시키지 않는 상속 계층을 유지하는 것은 쉽지 않다.

이러한 문제를 해결할 수 있는 방법은 클라이언트의 기대에 맞게 상속 계층을 분리하는 것이다.

이처럼 인터페이스를 클라이언트의 기대에 따라 분리하는 것을 통해 변경에 의해 영향을 제어하는 설계원칙인 ISP(인터페이스 분리 원칙) 이라고 한다.

### 서브클래싱과 서브타이핑

상속을 언제 사용하는가에 대해서 첨에 재사용을 위해서와 타입 계층을 구성하기 위해서라고 다뤘었다.

사람들은 상속을 사용하는 두가지 목적에 대해서 이름을 붙였는데, 서브클래싱과 서브타이핑이 그것이다.

각각 첫번째와 두번째 목적을 가리킨다.

**이때 서브클래싱은 구현 상속 혹은 클래스 상속이라고 하며 자식 클래스의 인스턴스가 부모 클래스의 인스턴스를 대체할 수 없다.**

**서브타이핑은 인터페이스 상속이라고 하며 자식 클래스 인스턴스가 부모 클래스 인스턴스를 대체할 수 있다.**

슈퍼타입 인스턴스를 요구하는 곳에서 서브타입 인스턴스를 사용하기 위해서는 최소한 서브타입의 퍼블릭 인터페이스가 슈퍼타입의 퍼블릭 인터페이스와 동일하거나 더 많은 오퍼레이션을 포함해야 한다. 즉, 행동 호환성이 만족되어야 한다.

이는 곧 대체 가능성을 포함하는 것이다.

## 리스코프 치환 원칙

올바른 상속 관계의 특징을 정의하기 위해 리스코프 치환 원칙(LSP)가 발표되었는데, 이에 따르면 상속 관계로 연결한 클래스가 서브타이핑 관계를 만족시키기 위해서는 다음의 조건을 만족해야 한다.

> 서브타입은 그것의 기반 타입에 대해 대체 가능해야 한다는 것으로 클라이언트가 차이점을 인식하지 못한 채 기반 클래스의 인터페이스를 통해 서브클래스를 사용할 수 있어야 한다.
> 

즉, 리스코프 치환 원칙은 행동 호환성을 원칙으로 정리한 것이다.

### 클라이언트와 대체 가능성

리스코프 치환 원칙은 자식 클래스가 부모 클래스를 대체하기 위해서는 부모 클래스에 대한 클라이언트의 가정을 준수해야 한다는 것을 강조한다.

이는 Stack과 Vector가 서브타이핑이 아닌 서브클래싱 관계인 이유도 마찬가지이다.

상속으로 인해 Stack에 포함돼서는 안되는 Vector의 퍼블릭 인터페이스가 Stack의 퍼블릭 인터페이스에 포함되었기 때문이다.

예를 들면 Vector를 사용하는 클라이언트의 관점에서 Stack의 행동은 Vector의 행동과 호환되지 않는다.

즉, Vector를 사용하는 클라이언트와 Stack을 사용하는 클라이언트가 전송할 수 있는 메시지와 기대하는 행동이 다르다는 것이다. 즉, 서로 다른 크라이언트와 협력해야 한다는 것을 의미한다.

따라서 리스코프 치환 원칙은 “클라이언트와 격리한 채로 본 모델을 의미있게 검증하는 것은 불가능하다” 는 중요한 결론을 이끈다.

중요한 것은 대체 가능성을 결정하는 것은 클라이언트라는 것이다.

### is-a 관계 다시 살펴보기

그렇다면 다시 is-a 관계를 살펴보자.

클라이언트 관점에서 자식 클래스의 행동이 부모 클래스의 행동과 호환되지 않고 그로 인해 대체가 불가능하다면 어휘적으로 is-a 라고 말할 수 있더라도 그 관계를 is-a 관계라고 말할 수 없다.

즉, is-a 관계로 표현된 것은 앞에 “클라이언트 입장에서”라는 말이 내포되어 있다고 생각하면 된다.

### 리스코프 치환 원칙은 유연한 설계의 기반이다

리스코프 치환 원칙은 클라이언트가 어떤 자식 클래스와도 안정적으로 협력할 수 있는 상속 구조를 구현할 수 있는 가이드라인을 제공한다.

새로운 자식 클래스를 추가하더라도 클라이언트 입장에서 동일하게 행동하기만 한다면 클라이언트를 수정하지 않고도 계속해서 확장할 수 있다. 이는 퍼블릭 인터페이스의 행동 방식이 변경되지 않는다면 클라이언트의 코드를 변경하지 않고도 새로운 자식 클래스와 협력할 수 있게 된다는 것이다.

## 계약에 의한 설계와 서브타이핑

클라이언트와 서버 사이의 협력을 의무와 이익으로 구성된 계약의 관점에서 표현하는 것을 계약에 의한 설계(Design By Contract, DBC) 라고 부른다.

이는 클라이언트가 정상적으로 메소드를 실행하기 위해 만족시켜야 하는 **사전조건과** 메소드가 실행된 후에 서버가 클라이언트에게 보장해야 하는 **사후조건**, 메소드 실행 전과 실행 후에 인스턴스가 만족해야 하는 **클래스 불변식**의 세가지 요소로 구성된다.

<aside>
💡 서브타입이 리스코프 치환 원칙을 만족시키기 위해서는 클라이언트와 슈퍼타입 간에 체결된 ‘계약’을 준수해야 한다.

</aside>

### 서브타입과 계약

계약의 관점에서 상속이 초래하는 가장 큰 문제점은 자식 클래스가 부모 클래스의 메소드를 오버라이딩 할 수 있다는 것이다.

이때 몇가지 규칙이 있는데, 다음과 같다.

- 서브타입에 더 강력한 사전조건을 정의할 수 없다.
    - 더 강력한 서전조건을 정의하게 되면 클라이언트가 이전의 사전조건만을 알고 있기 때문에 협력에 문제가 생길 수 있다.
- 서브타입에 더 약화된 사전조건은 정의할 수 있다.
    - 이는 기존 협력에 어떤 영향도 끼치지 않기 때문에 가능하다.
- 서브타입에 더 약화된 사후조건을 정의할 수 없다.
    - 잘못된 값이 나올 수 있기 때문에 협력에 문제가 생길 수 있다.
- 서브타입에 더 강력한 사후조건은 정의할 수 있다.
    - 이는 기존 협력에 영향을 끼치지 않기 때문에 가능하다.

계약에 의한 설계는 클라이언트 관점에서의 대체 가능성을 계약으로 설명할 수 있다는 사실을 잘 보여준다.

따라서 서브타이핑을 위해 상속을 사용하고 있다면 부모 클래스가 클라이언트와 맺고 있는 계약에 관해 깊이 있게 고민하기 바란다.
