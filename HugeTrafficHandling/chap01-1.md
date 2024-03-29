![image](https://user-images.githubusercontent.com/27190617/216613119-49d80bfa-68d0-41f6-b729-7817b07aaed2.png)

### 단일 서버
웹 앱, 데이터베이스, 캐시 등이 전부 서버 한 대에서 실행된다.
 

### 사용자 요청 처리 흐름
1. 사용자는 도메인 이름(api.mysite.com)을 이용해서 웹사이트에 접속한다. 이 접속을 위해서는 도메인 이름을 DNS에 질의하여 IP 주소로 변환하는 과정이 필요하다.
2. DNS 조회 결과로 IP 주소가 반환된다. (웹 서버의 주소)
3. 해당 IP 주소로 HTTP 요청이 웹 서버에 전달된다.
4. 요청을 받은 웹 서버는 HTML 페이지나 JSON 형태의 응답을 반환한다.

![image](https://user-images.githubusercontent.com/27190617/216613203-fde00786-6c17-4c5f-ba12-21bb521b39c7.png)

두 가지 종류의 단말으로부터 요청

웹 애플리케이션
- 서버 구현용 언어(자바, 파이썬) : 비즈니스 로직, 데이터 저장 처리
- 클라이언트 구현용 언어(HTML, JS) : 프레젠테이션용
모바일 앱
- HTTP 프로토콜 이용 : 모바일 앱과 웹 서버 간 통신 
- JSON 데이터 포맷 사용

### 데이터베이스

- 사용자가 증가할 경우, 하나의 서버로는 부족해진다. 여러 서버를 두어야 한다.
- 아래 그림은 웹/모바일 트래픽 처리(웹 계층)와 데이터베이스(데이터 계층) 용을 따로 분리했다.

![image](https://user-images.githubusercontent.com/27190617/216613336-55df48ff-9d28-41b5-8dc5-567b3c9141d0.png)

데이터베이스의 종류

- 관계형 데이터베이스(Relational Database)
  - MySQL, Oracle, PostgreSQL
  - 테이블, 열, 칼럼
  - join 지원
- 비-관계형 데이터베이스(NoSQL)
  - CouchDB, Neo4j, Cassandra, HBase, Amazon DynamoDB
  - NoSQL의 4가지 분류
    - 키-값 저장소(key-value store) 
    - 그래프 저장소(graph store)
    - 칼럼 저장소(column store)
    - 문서 저장소(document store)
  - join 지원 X

NoSQL 활용 예시
- 아주 낮은 응답 지연시간(latency) 요구되는 경우
- 다루는 데이터가 비정형(unstructured)이라 관계형 데이터가 아니다.
- 데이터(JSON, YAML, XML 등)를 직렬화하거나(serialize) 역직렬화(deserial-ize)할 수 있기만 하면 된다.
- 아주 많은 양의 데이터를 저장해야 하는 경우

### 수직적 규모 확장 vs 수평적 규모 확장
수직적 확장 : 서버로 유입되는 트래픽 양이 적을 때
- 스케일 업(scale up)
  - 수직적 규모 확장 프로세스
  - 서버에 고사양 자원(더 좋은 CPU, 더 많은 RAM)을 추가하는 행위
  - 단점
- 스케일 아웃(scale out)
  - 수평적 규모 확장 프로세스
  - 더 많은 서버를 추가하여 성능 개선하는 행위
  - 수직적 규모 확장 프로세스의 단점으로 대규모 애플리케이션을 지원하는 데 수평적 규모 확장 프로세스를 사용한다.
 
 
 ### 로드밸런서(load balancer)
 - 부하 분산 집합에 속한 웹 서버들에게 트래픽 부하를 고르게 분산하는 역할
 - 너무 많은 사용자가 접속하거나 웹 서버가 한게 상황에 도달하면 응답 속도가 느려지거나 서버 접속이 불가능해질 수 있다. -> 로드밸런서 도입
 - 사용자는 로드밸런서의 공개 IP 주소로 접속해 웹 서버는 클라이언트의 접속을 직접 처리하지 않는다.
 - 보안 향상을 위해 서버 간 통신에는 사설 IP 주소를 이용함으로싸, 인터넷을 통해서 접속하지 못하게 한다. 로드밸런서는 웹 서버와 통신하기 위해 이 사설 주소를 사용한다.
 - 로드 밸런서는 자동적으로 트래픽을 분산해주기 때문에 서버1이 다운되면, 모든 트래픽은 서버2로 전송한다.

![image](https://user-images.githubusercontent.com/27190617/216613815-878b91eb-0c05-4cb3-bb51-a68c1d539f77.png)

### 데이터베이스 다중화
- 데이터 계층의 문제 해결 방법
- 서버 사이에 주(master)-부(slave) 관계를 설정하고 데이터 원본은 주 서버에, 사본은 부 서버에 저장하는 방식
- 주 서버 : 쓰기 연산, insert, delete, update
- 부 서버 : 주 서버의 사본을 전달받고, 읽기 연산만 지원 
- 대부분의 애플리케이션은 쓰기 연산 < 읽기 연산 
- 주 서버보다 부 서버의 수가 더 많다.
 ![image](https://user-images.githubusercontent.com/27190617/216613911-c290492f-05d1-49ca-bae3-d180afee79d9.png)

- 다중화의 장점 
  - 더 나은 성능 :
 master-slave 다중화 모델에서 쓰기 연산은 주 데이터베이스 서버로, 읽기 연산은 부 데이터베이스 서버로 분산된다.
 병렬로 처리될 수 있는 질의(query) 수가 늘어나 성능이 향상된다.
  - 안정성 : 데이터베이스 서버 가운데 일부가 파괴되도 데이터의 보존이 가능하다. 
  - 가용성 : 여러 지역에 데이터를 복사해서, 하나의 데이터가 장애가 발생해도 다른 서버에서 복구 가능
  - 주 서버가 다운될 경우?
    - 부 서버가 일시적으로 주 서버가 되고, 부 서버에 보관되었던 데이터가 최신 상태가 아닐 수 있을 경우에는 복구 스크립트(recovery script)를 돌려서 부족한 데이터를 추가해준다.
![image](https://user-images.githubusercontent.com/27190617/216614179-a2834e97-0371-4c5c-b6d5-7983c6a3710d.png)

- 사용자는 DNS에서 로드밸런서의 public IP를 발급
- 사용자는 해당 IP 주소로 로드밸런서에 접속
- HTTP 요청은 서버 1 or 서버 2로 전달
- 웹 서버는 읽기 연산을 부 데이터베이스 서버로
- 웹 서버는 쓰기 연산을 주 데이터 베이스로 전달
