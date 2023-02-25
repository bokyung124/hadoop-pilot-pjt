# 1. Flume 설치
- CM 홈에서 [서비스 추가] - [Flume] 선택
![](/img/CH03/flume%20%EC%84%A4%EC%B9%98.png)   
- 서버 호스트 `server02.hadoop.com` 선택

<br>

- __Flume Heap Memory 수정__
    - 50Mib -> __100MiB__
    ![](img/CH03/flume%20heap%20memory%20%EB%B3%80%EA%B2%BD.png)   

<br>

# 2. Kafka 설치
- CM 홈에서 [서비스 추가] - [Kafka] 선택
![](img/CH03/kafka%20%EC%84%A4%EC%B9%98.png)   
- 서버 호스트 `server02.hadoop.com` 선택

<br>

- __카프카에 저장될 메시지 보관 기간 짧게 조정__
    - `Data Retention Time` 7일 -> __10분__
    ![](img/CH03/kafka%20data%20retention%20time%20%EB%B3%80%EA%B2%BD.png)

<br>

# 3. 플럼 수집 기능 구현
- 플럼에서 2개의 에이전트 구현
    - 스마트카 