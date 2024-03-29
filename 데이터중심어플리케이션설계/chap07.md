![image](https://user-images.githubusercontent.com/27190617/200301877-a8d8db96-42c9-4365-b74b-4185a9fd2a02.png)

> 6장과 비교했을 때 굉장히 내용이 많다.   
> 다양한 정의 및 내용이 나오다보니 개인적으로 헷갈리는 부분이 많았다.  

## **트랜잭션 (Transaction)** 

-   애플리케이션에서 다수의 읽기와 쓰기 작업을 하나의 논리적 단위로 묶는 방법  
    (개념적으로 하나의 트랜잭션 내에서는 모든 읽기 / 쓰기는 하나의 연산으로 실행) 
-   트랜잭션은 전체가 성공(commit) or 실패(abort, rollback) 된다. 

#### **트랜잭션이 필요한 이유** 

> 1\. DB, SW, HW, 모두 언제라도 연산 및 작업이 실패할 수 있다.   
> 2\. 네트워크 오류로 인해 노드 사이의 통신 실패   
> 3\. Client 사이의 경쟁 조건으로 인해 예상치 못한 버그 유발 가능 

### **ACID** 

-   원자성(**A**tomicity), 일관성(**C**onsistency), 격리성(**I**solation), 지속성(**D**urability) 
-   DB마다 ACID 구현은 제각각. (ex\_ 격리성의 의미 주변에는 모호함이 존재) 
-   이와 반대로 ACID 표준을 따르지 않는 시스템은 **BASE** 로 부름
    -   가용성(**B**asically **A**vailable), 유연성(**S**oft state), 최종적 일관성(**E**ventual consistency) 

#### **원자성(Atomicity)**

-   시스템은 연산을 실행하기 전, 후의 상태만 존재, 중간 상태란 없다. 
-   일부만 처리된 후결함이 발생시, commit할 수 없다면 abort되고 DB는 해당 write을 abort 하거나 rollback
-   원자성은 동시성과 관련이 없다. 
    -   여러 프로세스가 동시에 같은 data 접근시, 무슨 일이 생기는 지는 격리성에서 다룬다. 

####  **일관성**(**Consistency)**

이미 지난 앞선 장에서도 등장하였으며, 이후의 장에서도 여러 의미를 가지고 있다.

-   **replica** consistency와 비동기식으로 복제되는 시스템에서 발생하는 **최종적 일관성**
-   **일관성 해싱**은 어떤 시스템들에서 재균형화를 위해 사용하는 파티셔닝 
-   CAP 정리(9장)에서 **일관성 = 선형성(linearizability)** 
-   **ACID 맥락 : 일관성 = DB가 좋은 상태** 
    -   데이터(의 상태)는 항상 진실이어야하며, 데이터에 관한 어떤 선언(불변식(invariant))이 존재한다.   
        불변식이 유효한 DB에서 트랜잭션이 시작한다면, 해당 트랜잭션에서 실행된 모든 쓰기의 유효성이 보존된다면 불변식은 항상 만족된다. 
-   하지만 트랜잭션의 일관성은 어플리케이션에 의존적 
    -   DB - 외래 키 제약 조건, 유일성 제약 조건 
    -   이외 불변식을 위반하는 데이터 write 작업은 DB에서 막을 수 없음 (실제로 일관성은 어플리케이션의 속성)

#### **격리성(Isolation)**

-   동시에 실행되는 트랜잭션은 서로 격리된다 - 트랜잭션은 다른 트랜잭션의 작업을 방해할 수 없다. 
-   **직렬성 격리**와 **스냅숏 격리**로 구현 
    -   **직렬성 격리 : 각 트랜잭션이 전체 DB에서 실행되는 유일한 트랜잭션처럼 동작.** **동시에 실행되었더라도 커밋의 결과는 순차적으로 실행했을 때 결과와 동일하도록 보장** 
    -   **스냅숏 격리 : MVCC(다중 버전 동시성 제어)**로 구현 

#### **지속성(Durability)** 

-   트랜잭션 커밋이 성공한다면 HW 결함, DB가 죽어도 데이터는 손실되지 않는다. 
-   쓰기전 로그(write-ahead log), 복제, 백업 등을 통해 구현 

> 1장에서 나온 것처럼 모든 HDD, BackUp이 동시에 파괴되어 버리면 DB에선 지속성을 유지할 수 있는 방법이 X 

---

### **이상 현상 정리** 

**dirty read (더티 읽기)** 

**: 하나의 트랜잭션에서 작업이 완료되지 않았는데도 다른 트랜잭션에서 볼 수 있게 되는 현상** 

**dirty write (더티 쓰기)**  

**: 아직 커밋되지 않은 트랜잭션에서 쓴 데이터를 새로운 트랜잭션이 덮어쓰는 현상**

![image](https://user-images.githubusercontent.com/27190617/200302058-9ee504f8-3440-4a59-8012-b09fb6b7c6e7.png)
-   해결 방법은 밑에서 나오겠지만, 먼저 쓴 트랜잭션이 커밋이나 어보트될 때까지 두 번째 쓰기를 지연시킴
-   지연을 위해서는 lock을 획득, 대부분의 트랜잭션 구현을 통해 방지 가능 

**갱신 손실**

**: 동시 쓰기 작업 시 발생하는 쓰기 충돌 중 하나. read-modify-write 주기로 작업할 때 발생** 

![image](https://user-images.githubusercontent.com/27190617/200302085-72b5ea2e-e412-44ba-8292-43fd69c368f3.png)
****두 트랜잭션이 read-modify-write 작업을 하면 두 번째 쓰기 작업이 첫 번째 쓰기 작업을 포함하지 않으므로  
하나는 손실되어 더티 쓰기가 아닌 정상적인 프로세스로서 실행되어도 결과값이 기대값이 다르게 된다.****

#### ****Non-Reapeatable Read ( Read Skew ) 이상 현상**** 

![image](https://user-images.githubusercontent.com/27190617/200302104-69337f0a-db9b-4a27-9f19-d5f5387caf89.png)
 더티 읽기가 아닌 정상적인 프로세스 처리에도 불구하고, 시간 차이로 인해 발생하는 현상이다. 

물론 새로고침을 통해 재확인을 한다면 문제는 없지만, 백업 진행 혹은 분석, 무결성 확인에 대해서는 중요할 수 있다. 

이는 뒤에서 나올 스냅숏 격리(Snapshot Isolation)을 통해 문제를 막을 수 있다.

#### **Write Skew 현상과 팬텀(Phantom)**  

**두 트랜잭션이 두 개의 다른 객체를 읽어 그 중 일부를 갱신할 때 나타나는 "정책적인 위반 사항"**

-   이는 서로 다른 객체를 갱신하기 때문에 더티 쓰기도 갱신 손실도 아님
-   하지만, 정책적으로 두 객체가 경쟁 조건

![image](https://user-images.githubusercontent.com/27190617/200302134-6dda28cb-8831-4af5-9c58-9c09411509ba.png)

> 예시 :   
> \- 의사들의  호출 대기 상황   
> 조건 :  
> \- 호출 시점시, 최소 한 명의 의사는 호출이 가능한 상황이여야함(호출 대기)  
> 의사들은 다른 동료가 한명이라도 동일한 교대 순번에 대해 호출 대기를 하고 있다면 본인은 호출 대기를 그만 둘 수 있다.   
> 상황 :   
> \- 엘리스와 밥이 동일한 날에 함께 호출 대기중, 거의 동시에 둘다 쉬려고 호출 대기 상태를 끄는 상황 

#### **팬텀(Phantom)**

-   **어떠한 트랜잭션에서 실행한 쓰기가 다른 트랜잭션의 검색 질의 결과를 바꾸는 효과** 
-   뒤에서 나올 스냅샷 격리는 읽기 전용 질의에서 팬텀을 회피하지만, **읽기 쓰기 트랜잭션에서는 팬텀이 write skew를 유발할 수 있음** 
-   **스냅숏 격리로는 안되고 직렬성 격리가 필요함** 
-   **ex) 빈 회의실을 찾아 중복 없이 예약하려는 경우** 

## **격리 수준** 

****동시성 이슈(경쟁 조건)을 막기 위해 오로지 직렬성 격리를 한다면, 성능 비용면에서 손해가 많기 때문에, 현실에서는 완화된 트랜잭션 격리를 사용한다. 

완화된 트랜잭션의 각각의 격리 수준과 해당 격리에서 발생할 수 있는 이상 현상(문제)들에 대한 소개가 나온다. 

#### **READ  UNCOMMITTED**

-   더티 읽기 있음 / 더티 쓰기 없음 

#### **READ COMMITTED ( 커밋 후 읽기 )**

가장 기본적인 수준의 트랜잭션 격리. 해당 수준에서는 두가지를 보장해준다. 

-   DB에서는 읽을 때 커밋된 데이터만 본다. (dirty read X) 

![image](https://user-images.githubusercontent.com/27190617/200302203-19f78983-34d1-47cf-9bd1-a50b16727785.png)

-   DB에서 쓸 때 커밋된 데이터만 덮어쓴다. (dirty write X)
    -   로우 수준, 객체의 lock을 획득하여 보통 먼저 쓴 트랜잭션이 커밋되거나 어보트될 때까지 두 번째 write을 지연시킨다.

**read committed에서 발생할 수 있는 동시성 문제** 

-   위에서 보인 그림 7-1에 나온 두번의 카운터 증가 사이의 발생하는 경쟁 조건은 막지 못한다. 
    -   해당 경우는 첫 번째 트랜잭션이 커밋된 후에 두 번째 쓰기가 일어났으므로 더티 쓰기가 아니다. 
-   또한 앞서 설명한 read skew 현상 또한 막을 수 없다. 

![image](https://user-images.githubusercontent.com/27190617/200302218-8914cbdd-e3e3-49c3-ba10-cc64d1bde952.png)

#### **Snapshot Isolation 과 Repeatable read**

스냅숏 격리 구현은 read committed처럼 전형적으로 더티 쓰기 방지를 위해 write lock을 사용한다. 

하지만 읽을 때에는 아무런 lock도 사용하지 않는다. 

-   스냅샷 격리는 널리 쓰이는 기능으로 PostgreSQL, InnoDB 엔진을 사용하는 Oracle, MySQL, SQL\_Server에서 지원
    -   오라클에서는 **직렬성,** PostgreSQL, Mysql 에서는 **Repeatable read**라 칭함 
-   **읽는 쪽에서 쓰는 쪽을 결코 차단하지 않고, 쓰는 쪽에서도 읽는 쪽을 결코 차단하지 않는다.** 
    -   **읽기 전용 트랜잭션에 유용**
-   **DB는 객체의 여러 버전을 유지할 수 있어야 한다. ( MVCC - 다중 버전 동시성 제어 )**  
    -   **∵ 진행중인 각각의 트랜잭션은 서로 다른 시점의 스냅숏을 보기 때문에** 
-   **해당 격리 수준에서는 read-modify-write 도중에 갱신 손실 탐지시에 트랜잭션을 abort 시킨다.** 
    -   **예외 Mysql - InnoDB : 갱신 손실 감지 X** 

> 스냅숏 격리를 지원하는 저장소 엔진 : read-committed 격리를 위해서도 MVCC를 사용한다. 

![image](https://user-images.githubusercontent.com/27190617/200302250-b90a43b5-9d40-41fc-a25d-ec19c0e8fdd3.png)

> **일관된 스냅숏을 보는 가시성 규칙 (스냅숏의 가시성 규칙 동작 방식)**   
> 1\. DB는 각 트랜잭션 시작시, 그 시점에 진행중인 모든 트랜잭션의 목록을 생성( 해당 트랜잭션들이 쓴 데이터는 모두 무시)   
> 2\. abort 된 트랜잭션이 쓴 데이터 또한 무시   
> 3\. txId가 더 큰 트랜잭션이 쓴 데이터는 해당 트랜잭션의 커밋 여부와 관계 없이 모두 무시   
> 4\. 그 외의 모든 데이터는 query를 통해 확인 가능 

> **스냅숏 격리에서의 색인**   
>   
> MVCC를 지원하는 DB에서의 색인은 보통 객체의 모든 버전을 가리키게하고, index query가 현재 트랜잭션에서 볼 수 없는 버전들을 걸러내도록 한다. 물론 현실에서는 세부사항에 따라 MVCC의 성능이 바뀌게 된다.
