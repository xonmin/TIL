# 3장 프로그래밍 패러다임

### 구조적 프로그래밍
- 무분별한 `goto` 를 사용하지않고, 분기(`if/then/else`)와 반복(`do/while/until`)과 같은 제어구문을 사용하도록 하는 방식
- 구조적 프로그래밍은 **제어흐름의 직접적인 전환에 대해 규칙을 부과**

### 객체 지향 프로그래밍
- 함수 호출 스택 프레임(stack frame)을 heap으로 옮기면, 함수 호출이 반환된 후에도 함수에서 선언된 지역 변수가 오랫동안 유지될 수 있음을 발견
- 즉, heap에서 호출한 함수는 클래스의 생성자가, 오랫동안 유지된 지역 변수 -> 인스턴스 변수, 중첩 함수 -> 메서드
- 객체 지향 프로그래밍은 **제어흐름의 간접적인 전환에 대해 규칙을 부과**
> 함수 포인터를 특정 규칙에 따라 사용한다.

### 함수형 프로그래밍
- 수학적 문제를 해결하는 과정에서 `lambda` 계산법 발견
- lambda 의 기초 개념 불변성 : 심볼의 값이 변경되지 X
- 함수형 언어에는 할당문이 전혀 없다는 뜻
- 함수형 프로그래밍은 `할당문에 대해 규칙을 부과한다`
> 할당문을 쓰지 못하도록 한다.

### 결론
- 각 세가지 프로그래밍 페러다임은 일종의 규칙을 부과함으로써, 프로그래머에게 특정 권한 박탈
- 패러다임은 **무엇을 해서는 안되는지를 말한다.**
