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
    - 스마트카 상태정보 수집하는 `SmartCarInfo Agent`
    - 운전자의 운행정보 수집하는 `DriverCarInfo Agent`

- 에이전트 생성 
  - 플럼이 인식할 수 있는 특정 디렉터리에 `{Agent 고유 이름}.conf` 파일 생성
  - 파일럿 프로젝트에서는 CM에서 제공하는 플럼 구성 정보 설정을 통해 에이전트 편리하게 생성 가능

## 1) SmartCar 에이전트 생성
- CM 홈 [Flume] - [구성]
- `구성 파일` 항목 수정

<br>

- Agent 이름: `SmartCar_Agent`
- 구성 파일   
```conf
#1
SmartCar_Agent.sources  = SmartCarInfo_SpoolSource
SmartCar_Agent.channels = SmartCarInfo_Channel
SmartCar_Agent.sinks    = SmartCarInfo_LoggerSink 

#2
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.type = spooldir
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.spoolDir = /home/pilot-pjt/working/car-batch-log
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.deletePolicy = immediate
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.batchSize = 1000

#3
SmartCar_Agent.channels.SmartCarInfo_Channel.type = memory
SmartCar_Agent.channels.SmartCarInfo_Channel.capacity  = 100000
SmartCar_Agent.channels.SmartCarInfo_Channel.transactionCapacity  = 10000

#4
SmartCar_Agent.sinks.SmartCarInfo_LoggerSink.type = logger

#5
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.channels = SmartCarInfo_Channel
SmartCar_Agent.sinks.SmartCarInfo_LoggerSink.channel = SmartCarInfo_Channel
```

- `1` 
  - 플럼의 에이전트에서 사용할
