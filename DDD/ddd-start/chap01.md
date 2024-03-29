# 1장 도메인 모델 시작 

## 도메인이란 무엇인가 
소프트웨어로 해결하고자 하는 문제 영역 - 도메인 

도메인은 하위 도메인으로 이루어지기도 함 
- ex) 주문 도메인 
  - 하위 도메인으로 혜택, 결제, 배송, 정산 등의 도메인 존재

그러나 모든 도메인이 하위 도메인이 존재하는 것은 아니기 때문에, 상황에 따라 하위 도메인 구성 판단 

## 도메인 모델 
특정 도메인을 개념적으로 표현한 것 
- 도메인을 이해하기 위해서는 제공하는 기능, 도메인의 주요 데이터 구성 파악 
- 이러한 면에서 기능과 데이터릃 ㅏㅁ께 보여주는 객체 모델이 도메인을 모델링하기에 적합


## 도메인 모델 패턴 
일반적으로 어플리케이션의 아키텍처는 네 개의 계층으로 구성 
- UI : 사용자의 요청 처리 / 사용자에게 정보를 보여줌 
  - 소프트웨어 사용자 뿐만 아닌 외부 시스템도 사용자가 될 수 있음 
- 응용 : 사용자가 요청한 기능 실행 / 도메인 계층을 조합해 기능 실행 
- 도메인 : 시스템이 제공할 도메인의 규칙 구현
  - ex) 주문 도메인의 경우 출고 전에 배송지 변경 가능 
  - ex) 주문 취소는 배송 전에만 가능 
- 인프라스트럭쳐 : DB, 메시징 시스템과 같은 외부 시스템과 연동 처리 

도메인 모델 패턴은 아키텍쳐 상의 도메인 계층을 객체 지향 기법으로 구현하는 패턴 
- 도메인 모델에 규칙을 두는 이유 : 코드가 도메인 모델에만 위치하여 규칙이 바뀌거나 확장하더라도 다른 코드에 영향을 덜 주고 변경 내역을 모델에 반영 가능 

## 도메인 모델 도출 
개발자들은 기획서, 유스케이스, 사용자 스토디 등을 통해 요구사항의 도메인을 이해 후, 이를 통해 도메인 모델 만들고 개발 시작 
- 도메인 모델링의 기본 작업 
  - 모델을 구성하는 핵심요소, 규칙, 기능 찾기 

## 엔티티와 밸류 
도메인 모델은 엔티티와 밸류 이 두가지로 구분할 수 있다. 

### Entity 
- 식별자를 가진다는 특징 
- 식별자는 바뀌지 않고 고유하다
  - 엔티티를 구현한 클래스는 식별자를 이용해 `equals(), hashCode()`메서드를 구현 가능 

### Entity 식별자 생성 
Entity 식별자를 생성하는 시점은 도멩니의 특징과 사용하는 기술에 따라 달라짐
- 특정규칙에 따라 생성
- UUID 사용 
- 값을 직접 입력하여 사용 
  - ex) 회원 아이디 / 이메일 등 중복 케이스 발생 가능성이 있을 때는 중복 입력을 막는 로직 요구 
- 일련번호 사용(시퀀스 Num, DB Auto Increment)

### Value Type 
Value Type은 개념적으로 완전한 하나를 표현할 떄 사용 

- 나쁜 예제 
```java
public class ShippingInfo{
    //받는 사람 
    private String receiverName; 
    private String receiverPhoneNumber; 
    
    //주소 
    private String shippingAddress1;
    private String shippingAddress2;
    private String shippingZipCode; 
    ...
}

```

- 좋은 예제로 변경 
```java
//별도로 받는 사람 , 주소 등을 도메인의 개념이기 때문에 Value Type 클래스 생성 
public class Receiver {
    private String name;
    private String phoneNumber;
    ...
}

public class Address{
    private String address1;
    private String address2;
    private String zipcode;
    ...


// 내부에서 변수로 해당 밸류 클래스들을 가짐 
public class ShippingInfo {
    private Receiver receiver;
    private Address address;

    // ... 생성자, get 메서드
}

```

Value Type의 장점
- 해당 데이터(값/변수)의 도메인(개념/의미)를 명확하게 명시 가능
- 해당 Value Type을 위한 기능(메서드) 추가 가능 

Value Type 사용시의 특징 
- 데이터 변경 시 해당 인스턴스의 값을 변경하는 것이 아닌 변경할 값을 가지는 새로운 인스턴스를 생성하는 방식 선호 
  - > 불변 객체로 사용하라 
  - 이룰 통해 의도치 않은 곳에서의 데이터 변경을 방지 가능 
- Value 객체 비교에서 모든 속성이 같은지ㅇ우 식별자를 비교해야한다. 


### Entity 식별자와 밸류 타입 
일반적으로 식별자는 `String` 타입이지만, 식별자가 도메인 특성을 가지는 경우 식별자를 위한 Value Type을 사용하여 의미가 잘 들어나도록 한다. 

### 도메인 모델에 set 메서드 넣지 않기 
- `get/set` 메서드로 인해 (특히 `set`) 값이 어디서든 바뀔 수 있기 때문에 도메인 핵심 개념이나 의도가 코드에서 사라질 수 있음 
- 기본 생성자에 `set` 메서드로 값을 전달하다보면 깨체 내부에 누락되는 데이터 존재 가능 
  - 도메인 객체는 생성 시점에 (constructor) 필요한 것을 전달 

### 도메인 용어 
- 코드를 작성할 때 도메인에 사용하는 용어 매우 중요 -> 이름 짓기가 중요하다 
- 도메인에서 사용하는 용어를 반영하지 않는다면 해당 코드는 개발자에게 코드의 의미를 해석하는 일을 준다 
```java
public OrderState {
 STEP1, STEP2, STEP3, STEP4, STEP5, STEP6
}
```