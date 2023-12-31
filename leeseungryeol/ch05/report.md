<h1>책임 할당하기</h1>

데이터 중심 설계 -> 책임 중심 설계</br>
- 데이터보다 행동을 먼저 결정하라
- 협력이라는 문맥 안에서 책임을 결정하라

<h5>데이터보다 행동을 먼저 결절하라</h5>
책임 중심의 설계에서는 '이 객체가 수행해야 하는 책임은 무엇인가'를 결정한 후에 '이 책임을 수행하는 데 필요한 데이터는 무엇인가'를 결정한다.

<h5>협력이라는 문맥 안에서 책임을 결정하라</h5>

협력에 적합한 책임이란 메시지 수신자가 아니라 메시지 전송자에게 적합한 책임을 의미한다.</br>
=> 협력에 적합한 책임을 수확하기 위해서는 객체를 결정한 후에 메시지를 선택하는 것이 아니라 메시지를 결정한 후에 객체를 선택해야한다.</br>
ex) 이 클래스가 필요하다는 점은 알겠는데 이 클래스는 무엇을 해야하지? 라고 질문하지 않고 메시지를 전송해야하는데 누구에게 전송해야하지?라고 질문하는 것.</br>
설계의 핵심 질문을 이렇게 바꾸는 것이 메시지 기반 설계로 향하는 첫걸음이다.</br>

<h5>책임 주도 설계</h5>
- 시스템이 사용자에게 제공해야 하는 기능인 시스템 책임을 파악한다.</br>
- 시스템 책임을 더 작은 책임으로 분할한다.</br>
- 분할된 책임을 수행할 수 있는 적절한 객체 또는 역할을 찾아 책임을 할당한다.</br>
- 객체가 책임을 수행하는 도중 다른 객체의 도움이 필요한 경우 이를 책임질 적절한 객체 또는 역할을 찾는다.</br>
- 해당 객체 또는 역할에게 책임을 할당함으로써 두 객체가 협력하게 된다.</br>

<h2>책임 할당을 위한 GRASP 패턴</h2>

<h4>도메인 개념에서 출발하기</h4>
설계를 시작하기 전에 도메인에 대한 개략적인 모습을 그려 보는 것이 유용하다.
<h4>정보 전문가에게 책임을 할당하라</h4>
애플리케이션이 제공해야 하는 기능을 애플리케이션의 책임으로 생각하는 것이다.
 1. 예매하라
 2. 예매할 객체를 지정(필요한 정보를 가장 많이 알고 있는 객체 선정) => Screening
 3. 가격을 계산하라
 4. 가격을 계산할 객체 지정 => Movie
 5. 할인 여부 판단
 6. 할인 여부 판단을 할 객체를 지정 => Discounst Condition
 7. 예약할 객체를 지정(최종 결과물은 Reservation 인스턴스를 생성)
 8. Reservation을 위한 정보를 많이 알고 있는 객체 => Screening

 5,6에서 Discount Condition에 대한 결합을 Movie가 아닌 Screening에 할 수 있다.
 하지만, Screening에 할 경우 Screening에는 할인을 계산하는 정보를 떠안아야 한다.
 => 예매 요금을 계산하는 방식이 변경될 경우 Screening도 함께 변경해야 한다. 


 <h2>구현을 통한 검증</h2>

 2장 내용과 코드가 같음.

 <h5>클래스 응집도 판단하기 </h5>
  - 클래스가 하나 이상의 이유로 변경돼야 한다면 응집도가 낮은 것이다. 변경의 이유를 기준으로 클래스를 분리한다.
  - 클래스의 인스턴스를 초기화하는 시점에 경우에 따라 서로 다른 속성들을 초기화하고 있다면 응집도가 낮은 것이다. 초기화되는 속성의 그룹을 기준으로 클래스르 분리한다.
  - 메서드 그룹이 속성 그룹을 사용하는지 여부로 나뉜다면 응집도가 낮은 것이다. 이들 그룹을 기준으로 클래스를 분리한다.

 <h5>도메인의 구조가 코드의 구조를 이끈다</h5>

구현을 가이드할 수 있는 도메인 모델을 선택하라. 객체지향은 도메인의 개념과 구조를 반영한 코드를 가능하게 만들기 때문에 도메인의 구조가 코드의 구조를 이끌어 내는 것은 자연스러울뿐만 아니라 바람직한 것이다.

 <h5>코드의 구조가 도메인의 구조에 대한 새로운 통찰력을 제공한다</h5>
도메인 모델은 단순히 도메인의 개념과 관계를 모아 놓은 것이 아니다. 도메인 모델은 구현과 밀접한 관계를 맺어야 한다.
도메인 모델은 코드에 대한 가이드를 제공할 수 있어야 하며 코드의 변화에 발맞춰 함께 변화해야 한다.
    

<h2>책임 주도 설계의 대안</h2>

<h5>메서드 응집도</h5>

긴 메서드는 부정적인 영향을 미침
1. 로직의 일부만 재사용 불가능
2. 일부 로직만 수정하더라도 메서드의 나머지 부분에서 버그가 발생할 확률이 높다.
3. 변경이 필요할 때 수정해야 할 부분을 찾기 어렵다.
4. 한눈에 파악하기 어렵기 때문에 코드를 전체적으로 이해하는데 많은 시간이 걸린다.


=> 메서드를 작게 분해해서 각 메서드 응집도를 높여라

1. 재사용성 높아짐
2. 가독성 높아짐
3. 오버라이딩도 쉽음

