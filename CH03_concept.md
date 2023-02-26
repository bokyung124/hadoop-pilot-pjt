[출처: 실무로 배우는 빅데이터 기술, 김강원 저]

# 1. 플럼 Flume

## 1) 플럼
- 빅데이터를 __수집__할 때 다양한 수집 요구사항들을 해결하기 위한 기능으로 구성된 소프트웨어
- 통신 프로토콜, 메시지 포맷, 발생 주기, 데이터 크기 등 데이터를 수집할 때 고려해야 할 것들을 쉽게 해결할 수 있는 기능과 아키텍처 제공

<br>

## 2) 주요 구성요소
|구성요소|설명|
|---|---|
|Source| 다양한 원천 시스템의 데이터를 수집하기 위해 Avro, Thrift, JMS, Spool Dir, Kafka 등 컴포넌트 제공 <br> 수집한 데이터 Channel로 전달|
| Channel | Source와 Sink 연결 <br> 데이터를 버퍼링하는 컴포넌트로 메모리, 파일, 데이터베이스를 채널의 저장소로 활용|
|Sink|수집한 데이터를 Channel로부터 전달받아 최종 목적지에 저장하기 위한 기능 <br> HDFS, Hive, Logger, Avro, ElasticSearch, Thrift 등 제공|
|Interceptor |Source와 Channel 사이에서 데이터 필터링 및 가공하는 컴포넌트 <br> Timestamp, Host, Regex Filtering 등 기본 제공 <br> + 필요 시 사용자 정의 Interceptor 추가|
|Agent|Source → (Interceptor) → Channel → Sink 컴포넌트 순으로 구성된 작업 단위 <br> 독립된 인스턴스로 생성|

</br>

## 3) 플럼 아키텍처
- 플럼 메커니즘 : Source, Channel, Sink 만을 활용하는 매우 단순하고 직관적인 구조
- `Source`에서 데이터 로드, `Channel`에서 데이터 임시 저장, `Sink`를 통해 목적지에 최종 적재

</br>

### 3-1) 유형 1    
![flume1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcFIlJB%2FbtrWPK83bP5%2FhRnLmBlAMQNtXg9zQyb6Pk%2Fimg.png)

- 가장 단순한 플럼 에이전트 구성
- 원천 데이터를 특별한 처리 없이 단순 수집/적재   
<br>

### 3-2) 유형 2
![flume2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F4z94Z%2FbtrWQtS8Zed%2FT0fVeS4HqQiSjfCbMoOqSk%2Fimg.png)

- `Interceptor`를 추가해 데이터 가공
- 데이터의 특성에 따라 Channel에서 다수의 Sink 컴포넌트로 라우팅이 필요할 때 구성   
> 데이터 통신에서의 라우팅: 네트워크상에서 주소를 이용하여 목적지까지 메시지를 전달하는 방법을 체계적으로 결정하는 경로선택 과정
- 한 개의 플럼 에이전트 안에서 두 개 이상의 S-C-S 컴포넌트 구성 및 관리도 가능

<br>

### 3-3) 유형 3
![flume3](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbO9D2Z%2FbtrWQLstjKN%2FcuVXRU2bSWA7hpvReeoUmK%2Fimg.png)
- 플럼 에이전트에서 수집한 데이터를 에이전트 2, 3에 전송할 때 로드밸런싱, 복제, 페일오버 등의 기능을 선택적으로 수행 가능   
- 수집해야 할 원천 시스템은 한 곳이지만, 높은 성능과 안정성이 필요할 때 주로 사용   
<br>

> __로드밸런싱(부하분산)__: 서버가 처리해야 할 업무 혹은 요청을 여러 대의 서버로 나누어 처리하는 것. 한 대의 서버로 부하가 집중되지 않도록 트래픽을 관리해 각각의 서버가 최적의 퍼포먼스를 보일 수 있도록 하는 것이 목적   

> __페일오버(장애 극복 기능)__: 컴퓨터 서버, 시스템, 네트워크 등에서 이상이 생겼을 때 예비 시스템으로 자동전환되는 기능. 시스템 설계에서 높은 가용성과 신뢰성이 요구되는 경우 페일오버 기능을 탑재하는 것이 일반적 

<br>

### 3-4) 유형 4
![flume4](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbtuTWx%2FbtrWQq952UX%2F4ZT1Aa7j0By0TU1F4SRZkK%2Fimg.png)

- 수집해야 할 원천 시스템이 다양하고, 대규모의 데이터가 유입될 때
- 에이전트 1, 2, 3, 4에서 수집한 데이터를 에이전트 5에서 집계하고, 이때 에이전트 6으로 이중화해서 성능과 안정성 보장

<br>

## 4) 활용 방안
- 스마트카에서 발생하는 로그 직접 수집하는 역할. 로그 유형에 따라 두 가지 에이전트 구성할 것

### 4-1) 100대의 스마트카 상태 정보 로그파일
- 로그 시뮬레이터를 통해 매일 생성됨
- 생성된 상태 정보 파일을 플럼 에이전트가 일 단위로 수집해서 하둡에 적재, 이후 대규모 배치 분석에 활용
![flume5](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcMRe8i%2FbtrWSu35Iq5%2Fzk4ItlgvgxxrfuybwCO8X1%2Fimg.png)

<br>

### 4-2) 스마트카 운전자 100명의 운행 정보 실시간 기록
- 로그 시뮬레이터에 의해 운행 정보 실시간 로그 파일 생성됨
- 로그 발생과 동시에 플럼 에이전트가 수집해서 kafka에 전송
![flume6](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FemDzEF%2FbtrWPOp1t4r%2F3rf9dPHwJh78bvtrC2VzZk%2Fimg.png)

<br>

# 2. 카프카 Kafka

## 1) 카프카
- Message Oriented Middleware (MOM) 소프트웨어 중 하나
- 대규모로 발생하는 메시지성 데이터를 비동기 방식으로 중계
- 원천 시스템으로부터 대규모 트랜잭션 데이터가 발생했을 때, 중간에 데이터를 버퍼링하면서 타깃 시스템에 안정적으로 전송해주는 중간 시스템

<br>

## 2) 주요 구성요소
|구성요소|설명|
|---|---|
|Broker| 카프카의 서비스 인스턴스 <br> 다수의 Broker를 클러스터로 구성하고 Topic이 생성되는 물리적 서버|
|Topic| Broker에서 데이터의 발행/소비 처리를 위한 저장소|
|Provider|Broker의 특정 Topic에 데이터를 전송(발행)하는 역할 <br> 카프카 라이브러리를 통해 구현|
|Consumer|Broker의 특정 Topic에서 데이터를 수신(소비)하는 역할 <br> 카프카 라이브러리를 통해 구현|

<br>

## 3) 카프카 아키텍처
- 클러스터 방식에 따라 세가지 아키텍처 구성 가능, 반드시 주키퍼 이용

### 3-1) 유형 1 - 싱글 브로커 / 싱글 노드
![kafka1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbHQAQw%2FbtrWQsfLOc6%2FeMIWZZ0jZO4Mvtu4UTmZdK%2Fimg.png)
- 1대의 카프카 서버와 1개의 Broker만 구성한 아키텍처
- 대량의 발행 / 소비 요건이 없고, 업무 도메인이 단순할 때 이용

<br>

### 3-2) 유형 2 - 멀티 브로커 / 싱글 노드
![kafka2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcdOoeq%2FbtrWPOp1CkC%2FDp36P6KAWKOdHjQk42QWO0%2Fimg.png)
- 1대의 카프카 서버에 2개의 Broker를 구성한 아키텍처
- 물리적인 카프카 서버가 1대이므로 대량의 발행 / 소비 요건에는 사용 어려움
- 하지만, 업무 도메인이 복잡해서 메시지 처리를 분리 관리해야 할 때 이용

<br>

### 3-3) 유형 3 - 멀티 브로커 / 멀티 노드
![kafka3](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FefW4SY%2FbtrWQJO6l5T%2FXz4FyS48o1EvPlczR6qmbK%2Fimg.png)
- 2대 이상의 카프카 서버로 멀티 브로커 구성
- 대규모 발행 / 소비 데이터 처리에 적합
- 물리적으로 나눠진 브로커 간의 데이터 복제가 가능해 안정성이 높음
- 업무 도메인별 메시지 그룹을 분류할 수 있어 복잡한 메시지 송/수신에 적합

<br>

## 4) 활용 방안
- 플럼이 실시간 데이터를 수집해 카프카 Topic에 전달하면, 카프카는 받은 데이터를 Topic에 임시로 저장하고 있다가 Consumer 프로그램이 작동해 Topic에서 데이터 가져감   
![kafka4](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcdo8X3%2FbtrWRCIjAQr%2Fj1jY1R9kpa2kKZR1S2BOs1%2Fimg.png)

<br>

- 카프카 활용 목적 : 플럼이 아주 빠르게 발생하는 데이터를 실시간으로 수집하게 되면, 이를 최종 목적지에 전달하기 전 중간에서 안정적인 버퍼링 처리가 필요
- 카프카를 거치지 않고 바로 타깃 저장소인 HBase에 전송 → HBase에 장애가 발생하면 플럼의 Channel에 전송하지 못한 데이터들이 빠르게 쌓여 곧바로 플럼의 장애로도 이어짐 → 데이터 유실 발생

<br>

- HBase에 장에가 발생해도 카프카에서 데이터를 저장해 놓았다가 HBase가 복구되면 곧바로 재처리 가능   
&#43; 플럼이 수집한 데이터를 카프카의 토픽에 비동기로 전송함으로써 수집 속도가 빨라짐      
> __비동기 방식__: 동시에 일어나지 않을 수 있음 (요청을 보냈을 때 응답 상태와 상관없이 다음 동작을 수행 할 수 있음)    
  → 자원을 효율적으로 이용할 수 있음

![kafka5](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdLfb9Y%2FbtrWS1Oiqj3%2Fe3vIsmeuGDCkAVAVsom3PK%2Fimg.png)

<br>

# 3. 수집 파일럿 실행 1단계 - 수집 아키텍처

## 1) 수집 요구사항
- 차량의 다양한 장치로부터 발생하는 로그 파일 수집해서 기능별 상태 점검
- 운전자의 운행 정보가 담긴 로그를 실시간으로 수집해서 주행 패턴 분석

<br>

|수집 요구사항 구체화 |	분석 및 해결 방안|
|---|---|
|스마트카로부터 로그 파일 주기적으로 발생 | 플럼 → 대용량 배치 파일 및 실시간 로그 파일 수집|
|스마트카의 배치 로그 파일 이벤트 감지|플럼의 Source 컴포넌트 중 SpoolDir → 주기적인 로그 파일 발생 이벤트 감지|
|스마트카의 실시간 로그 발생 이벤트 감지|플럼의 Source 컴포넌트 중 Exec-Tail → 특정 로그 파일에서 로그 생성 이벤트 감지|
|스마트카가 만들어내는 로그 데이터에 가비지 데이터가 있을 수 있음|플럼의 Interceptor → 정상 패턴의 데이터만 필터링|
|수집 도중 장애가 발생해도 데이터를 안전하게 보관, 재처리해야 함|플럼의 메모리 Channel, 카프카 Broker → 로컬 디스크의 파일 시스템에 수집 데이터 임시 저장|
|스마트카의 실시간 로그 파일은 비동기 처리로 빠른 수집 처리|플럼에서 수집한 데이터를 카프카 Sink 컴포넌트를 이용해 카프카 Topic에 비동기 전송|

<br>

## 2) 수집 아키텍처
![collect1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FoLkHN%2FbtrWRDgajq4%2FSekbPKLFBAkMf0QXIkpWkk%2Fimg.png)   
<br>

### 2-1) 로그 시뮬레이터
- 스마트카의 상태 정보와 운전자의 운행 정보 로그를 가상으로 만드는 자바 로그 발생기   
<br>

- __스마트카 상태 정보__ : 100대 스마트카 장치들의 상태 정보를 3초 간격으로 발생 시킴, 1일 100MB 로그 파일 생성
- __운전자 운행 정보__ : 100명의 스마트카 운전자들의 운행 정보 실시간으로 발생 시킴, 하나의 운행 정보 로그는 4KB 미만, 동시에 최대 400KB 용량으로 실시간 데이터 발생

<br>

### 2-2) 플럼 에이전트1
- 대용량 로그 파일을 주기적으로 수집해서 표준 입출력 로거로 보여주는 레이어
- 스마트카 상태 정보를 기록한 로그 파일을 일별로 수집하기 위한 배치성 플럼 에이전트   
<br>

- `SpoolDir Source` : 약속된 로그 발생 디렉터리를 모니터링하다가 정의된 로그 파일 발생 시 해당 파일의 내용을 읽어서 수집하는 기능 제공
- `Memory Channel` : SpoolDir Source로부터 수집된 데이터를 메모리 Channel에 중간 적재. 버퍼링 기능을 제공하며 , Sink와 연결되어 트랜잭션 처리 지원함
- `Logger Sink` : Channel로부터 읽어들인 데이터를 플럼의 표준 로그 파일로 출력

<br>

### 2-3) 플럼 에이전트2
- 실시간으로 발생하는 로그를 라인 단위로 수집해 카프카의 Topic에 전송하는 레이어
- 스마트카 운전자의 운행 정보 실시간으로 수집하기 위한 실시간성 플럼 에이전트

<br>

- `Exec-Tail Source` : 로그가 쌓이고 있는 파일에 Tail 파이프라인을 이용해 실시간으로 데이터를 수집하는 기능
- `Memory Channel` : Exec-Tail Source로부터 수집한 데이터를 메모리 Channel에 버퍼링 처리를 하면서 임시 적재
- `Kafka Sink` : Channel로부터 읽어들인 데이터를 카프카 Broker의 특정 토픽에 비동기 방식으로 전송하는 Provider 역할 수행

<br>

### 2-4) 기타
- 플럼이 수집한 로그 데이터 임시 출력 및 저장

<br>

- `Flume Stdout` : 플럼의 Logger-Sink를 통해 표준 출력 로그가 출력됨
- `Kafka Topic` : 플럼의 Kafka-Sink는 수집된 실시간 로그를 임시 적재함
