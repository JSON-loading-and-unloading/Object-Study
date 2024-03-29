![KakaoTalk_20240223_120506817_03](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/33773af6-553c-4cf6-aba1-3ebf2138106d)<h1>디자인 패턴과 프레임 워크</h1>

<h2>디자인 패턴과 설계 재사용</h2>

<h3>패턴 분류</h3>

1. 아키텍처 패턴 : 디자인 패턴 상위, 미리 정의된 서브 시스템들을 제공하고, 각 서브시스템들의 책임을 정의하며, 서브시스템들 사이의 관계를 조직화하는 규칙과 가이드라인을 포함
2. 분석 패턴 : 도메인 내의 개념적인 문제를 해결하는데 초점을 맞춘다. 업무 모델링 시에 발견되는 공통적인 구조를 표현하는 개념들의 집합
3. 디자인 패턴 : 특정한 설계 문제를 해결하는 목적으로 하며, 프로그래밍 언어나 프로그래밍 패러다임에 독립적이다.
4. 이디엄 : 주어진 언어의 기능을 사용해 컴포넌트, 혹은 컴포넌트 간의 특정 측면을 구현하는 방법을 서술



<h3>패턴과 책임-주도 설계</h3>


![KakaoTalk_20240223_120506817_03](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/96cd96f7-81d2-4bc5-93cb-6102dd96ac22)

Composite 패턴</br>
 패턴의 구성 요소는 클래스가 아니라 '역할'이다.</br>
 패턴 구성요소 : Component, Composite, Leaf</br></br>

 OverlappedDiscountPolicy는 Composite역할 수행</br>
 AmountDiscountPolicy, PercentDiscountPolicy는 Leaf역할 수행</br></br>

※폴더(디렉토리) 안에는 파일이 들어 있을수도 있고 파일을 담은 또 다른 폴더도 들어있을 수 있다. 이를 복합적으로 담을수 있다 해서 Composite 객체라고 불리운다. </br>
※반면 파일은 단일 객체 이기 때문에 이를 Leaf 객체라고 불리운다. 즉 Leaf는 자식이 없다.</br>


<h3>캡슐화와 디자인 패턴</h3>

Strategy 패턴</br>

![KakaoTalk_20240223_120506817_02](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/9ee1650b-58a1-42e4-9ba8-71b3f9627254)


Strategy 목적 : 알고리즘의 변경을 캡슐화하는 것이고 이를 구현하기 위해 객체 합성을 이용</br></br>

영화에 적용될 할인 정책의 종류는 Movie가 참조하는 DiscountPolicy의 서브클래스가 무엇이냐에 따라 결정된다.</br>
Strategy패턴을 이용하면 Movie와 DiscountPolicy 사이의 결합도를 낮게 유지할 수 있기 때문에 런타임에 알고리즘을 변경할 수 있다.</br>


Template Method 패턴</br>


![KakaoTalk_20240223_120506817_01](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/ae6c93ce-47bf-4cb4-a5df-99366f3250a3)


Template Method 패턴 : 알고리즘을 캡슐화하기 위해 합성 관계가 아닌 상속 관계를 사용하는 것 (변경하지 않는 부분은 부모 클래스로, 변하는 부분은 자식 클래스로 분리함으로써 변경을 캡슐화 한다.)</br></br>

※추상 메소드를 이용해 변경을 캡슐화 한다.</br>
※합성보다는 결합도가 높은 상속을 사용</br>
calculateDiscountAmount 메서드가 바로 서브 클래스의 변경을 캡슐화하기 위해 사용되는 추상 메서드다. 부모 클래스의 calculateFee 메서드 안에서 추상 메서드인 calculateDiscountAmount를 호출하고 자식 클래스들이 이 메서드를 오버라이딩해서
변하는 부분을 구현한다는 것이 중요하다.</br>


Decorator 패턴</br>


![KakaoTalk_20240223_120506817](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/6e785807-211c-4c06-9b7b-bbe848d062f7)


 - Decorator 패턴은 객체의 행동을 동적으로 추가할 수 있게 해주는 패턴으로서 기본적으로 객체의 행동을 결합하기 위해 객체 합성을 사용한다.
 - Decorator 패턴은 선택적인 행동의 개수와 순서에 대한 변경을 캡슐화할 수 있다.


<h2>프레임워크와 코드 재사용</h2>

<h3>코드 재사용 대 설계 재사용</h3>

가장 이상적인 형태의 재사용 방법은 설계 재사용과 코드 재사용을 적절한 수준으로 조합하는 것이다.</br></br>

설계를 재사용하면서도 유사한 코드를 반복적으로 구현하는 문제를 피할 수 있는 방법은 없을까?</br>
프레임워크 사용 : 추상 클래스나 인터페이스를 정의하고 인스턴스 사이의 상호작용을 통해 시스템 전체 혹은 일부를 구현해 놓은 재사용 가능한 설계, 또는 애플리케이션 개발자가 현재의 요구사항에 맞게 커스터마이징할 수 있는 애플리케이션의 골격이다.</br></br>

프레임워크는 코드를 재사용함으로써 설계 아이디어를 재사용한다.</br>
또한, 애플리케이션의 아키텍처를 제공하며 문제해결에 필요한 설계 결정과 이에 필요한 기반 코드를 함께 포함한다.</br></br>

<h3>상위 정책과 하위 정책으로 패키지 분리하기</h3>

협력을 유연하게 만들기 위해서는 추상화를 이용해 변경을 캡슐화해야한다.</br>
또한, 협력을 구현하는 코드 안의 의존성은 가급적이면 추상 클래스나 인터페이스와 같은 추상화를 향하도록 작성해야 한다.</br></br>

상위정책이 세부 사항보다 더 다양한 상황에서 재사용될 수 있어야 한다.</br>
하지만 상위 정책이 세부 사항에 의존하게 되면 상위 정책이 필요한 모든 경우에 세부 사항도 항상 함께 존재해야 하기 때문에 상위 정책의 재사용성이 낮아진다.</br>
이를 해결할 수 있는 좋은 방법은 의존성 역전 원칙에 맞게 상위 정책과 세부 사항 모두 추상화에 의존하게 만드는 것이다.</br></br>

동일한 역할을 수행하는 객체들 사이의 협력 구조를 다양한 애플리케이션 안에서 재사용하는 것이 핵심이다.</br></br>

이를 위해서는 변하는 것과 변하지 않는 것을 서로 분리한다.</br></br>

변하지 않는 것은 상위 정책엥 속하는 역할들의 협력 구조이고, 변하는 것은 구체적인 세부 사항이다.</br>


![KakaoTalk_20240225_180612073](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/b25b10bd-20e3-4e46-985d-b27373c2b487)

<h3>제어 역전 관리</h3>

의존성을 역전시킨 객체지향 구조에서는 프레임워크가 애플리케이션에 속하는 서브 클래스의 메서드를 호출한다.</br>
따라서 프레임워크를 사용할 경우 개별 애플리케이션에서 프레임워크로 제어 흐름의 주체가 이동한다.</br>
즉, 의존성을 역전시키면 제어 흐름의 주체 역시 역전된다.</br>
이를 제어 역전원리라고 한다.</br>


![KakaoTalk_20240225_180612073_01](https://github.com/JSON-loading-and-unloading/Object-Study/assets/106163272/a03c0026-c43e-465c-8159-28d253ecc179)

위 그림에서 전체적인 협력 흐름은 프레임워크에 정의돼 있다.</br>
특정한 기본 정책을 구현하는 개발자는 Feecondition을 대체할 서브 타입만 개발하면 프레임워크에 정의된 플로우에 따라 요금이 계산된다.</br></br>

프레임워크에서는 일반적인 해결책만 제공하고 애플리케이션에 따라 달라질 수 있는 특정한 동작은 비워둔다.</br>
객체지향 시대애는 프레임워크가 호출하는 코드를 작성해야만 한다.</br>
제어가 우리에게서 프레임워크로 넘어가 버린 것이다. 제어의 역전이 됐다.</br>










