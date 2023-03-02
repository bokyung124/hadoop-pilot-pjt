# 1. 하둡 설치
- 이미 CH02에서 CM을 통해 설치 완료
- 하둡은 수집, 적재, 처리, 분석의 전 영역에 걸쳐 모든 컴포넌트와 연결되어 설치 구성됨

<br>

- 하둡 웹 관리 화면   
http://server01.hadoop.com:9870    
![](img/CH04/%ED%95%98%EB%91%A1%20%EC%83%81%ED%83%9C%20%EC%A0%95%EB%B3%B4%20%ED%99%95%EC%9D%B8.png)

<br>

- 하둡 웹 관리 화면은 CM 홈에서 [HDFS] 선택 후, 상단의 [NameNode 웹 UI] 메뉴로 접근할 수 있음
 - 하둡 클러스터의 오버뷰와 각종 설정들의 요약 정보가 메인 화면에서 제공됨
    - 전체 용량, 사용률, 활성/비활성 노드, 네임노드 상태, 저널노드 상태 등)
- 상단의 데이터노드 메뉴등을 통해 다양한 하둡 클러스터 정보 확인 가능

<br>

- 추가로 하둡은 주요 리소스를 관리 및 모니터링할 수 있는 웹 UI 제공
- 파일럿 환경의 잡 히스토리 서버는 VM의 비정상 종료 및 리소스 부족 현상 등으로 셧다운이 자주 발생함
- 잡 모니터링 관련 기능에 문제가 발생할 경우 CM 홈의 [YARN (MR2 Include)] -> [인스턴스]에서 JobHistory Server 상태가 **시작됨** 상태인지 확인하고, 정지 상태이면 재시작

<br>

[리소스 매니저](http://server01.hadoop.com:8088/cluster)
[Job History](http://server01.hadoop.com:19888/jobhistory)

<br>

- 파일럿 환경에서 SW 설치가 진행될 때마다 CM의 모니터링이 각 서버들의 인스턴스에서 자원 부족 및 클록 오프셋 경고 메시지 등을 표시
- 파일럿 환경에서는 크게 문제될 상황은 아니지만, 일단 클록 오프셋을 조정해 경고 메시지를 줄일 수 있음
- 클록 오프셋 조정
    - CM 홈 상단 [호스트] -> [모든 호스트] -> 우측 상단의 [구성]
    - 검색창에 '클록 오프셋' 입력
    - 경고/심각 항목에 모두 `안함`으로 섧정한 후 저장
- 클록 오프셋 외에도 다양한 경고 메시지가 표시될 때, 파일럿 환경에 맞춰 임계값을 수정해 경고 메시지를 없앨 수 있음

<br>

# 2. 적재 기능 구현
## 1) SmartCar 에이전트 수정
- CM 홈 - [Flume] - [구성]
- CH03에서 만든 SmartCar_Agent의 구성 파일에서 Logger Sink의 구성 요소들을 HDFS Sink로 교체

![](img/CH04/flume%20%EA%B5%AC%EC%84%B1%EC%9A%94%EC%86%8C%20%EA%B5%90%EC%B2%B4.png)

<br>

- 수정할 기존 Logger Sink
```conf
vSmartCar_Agent.sources  = SmartCarInfo_SpoolSource DriverCarInfo_TailSource
SmartCar_Agent.channels = SmartCarInfo_Channel DriverCarInfo_Channel
SmartCar_Agent.sinks    = SmartCarInfo_HdfsSink DriverCarInfo_KafkaSink  #1


SmartCar_Agent.sources.SmartCarInfo_SpoolSource.type = spooldir
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.spoolDir = /home/pilot-pjt/working/car-batch-log
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.deletePolicy = immediate
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.batchSize = 1000

#2
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors = timeInterceptor typeInterceptor collectDayInterceptor filterInterceptor

#3
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.timeInterceptor.type = timestamp
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.timeInterceptor.preserveExisting = true

#4
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.typeInterceptor.type = static
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.typeInterceptor.key = logType
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.typeInterceptor.value = car-batch-log

#5
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.collectDayInterceptor.type = com.wikibook.bigdata.smartcar.flume.CollectDayInterceptor$Builder

SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.filterInterceptor.type = regex_filter
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.filterInterceptor.regex = ^\\d{14}
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.filterInterceptor.excludeEvents = false

SmartCar_Agent.channels.SmartCarInfo_Channel.type = memory
SmartCar_Agent.channels.SmartCarInfo_Channel.capacity  = 100000
SmartCar_Agent.channels.SmartCarInfo_Channel.transactionCapacity  = 10000

#6
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.type = hdfs
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.path = /pilot-pjt/collect/%{logType}/wrk_date=%Y%m%d
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.filePrefix = %{logType}
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.fileSuffix = .log
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.fileType = DataStream
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.writeFormat = Text
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.batchSize = 10000
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.rollInterval = 0
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.rollCount = 0
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.idleTimeout = 100
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.callTimeout = 600000
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.rollSize = 67108864
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.threadsPoolSize = 10

#7
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.channels = SmartCarInfo_Channel
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.channel = SmartCarInfo_Channel


SmartCar_Agent.sources.DriverCarInfo_TailSource.type = exec
SmartCar_Agent.sources.DriverCarInfo_TailSource.command = tail -F /home/pilot-pjt/working/driver-realtime-log/SmartCarDriverInfo.log
SmartCar_Agent.sources.DriverCarInfo_TailSource.restart = true
SmartCar_Agent.sources.DriverCarInfo_TailSource.batchSize = 1000

SmartCar_Agent.sources.DriverCarInfo_TailSource.interceptors = filterInterceptor2
SmartCar_Agent.sources.DriverCarInfo_TailSource.interceptors.filterInterceptor2.type = regex_filter
SmartCar_Agent.sources.DriverCarInfo_TailSource.interceptors.filterInterceptor2.regex = ^\\d{14}
SmartCar_Agent.sources.DriverCarInfo_TailSource.interceptors.filterInterceptor2.excludeEvents = false

SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.type = org.apache.flume.sink.kafka.KafkaSink
SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.topic = SmartCar-Topic
SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.brokerList = server02.hadoop.com:9092
SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.requiredAcks = 1
SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.batchSize = 1000


SmartCar_Agent.channels.DriverCarInfo_Channel.type = memory
SmartCar_Agent.channels.DriverCarInfo_Channel.capacity= 100000
SmartCar_Agent.channels.DriverCarInfo_Channel.transactionCapacity = 10000


SmartCar_Agent.sources.DriverCarInfo_TailSource.channels = DriverCarInfo_Channel
SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.channel = DriverCarInfo_Channel
```

- #1
    - HDFS Sink 정보를 설정하기 위한 SmartCarInfo_HdfsSink 리소스 선언
- #2
    - 3개의 Interceptor 추가
    - 타임스탬프를 활용하기 위한 `timeInterceptor`
    - 로그 유형에 해당하는 상수값을 정의하기 위한 `typeInterceptor`
    - 수집 일자를 추가하기 위한 `collectDayInterceptor`
- #3
    - timeInterceptor의 설정
    - 타임스탬프 Interceptor를 추가하면, 플럼의 이벤트 헤더에 현재 타임 스탬프가 설정되어 필요시 헤더로부터 타임스탬프 값을 가져와 활용할 수 있음
- #4
    - typeInterceptor의 설정
    - 플럼의 해당 이벤트 내에서 사용할 상수를 선언하고 값을 설정
    - `logType` 이라는 상수를 선언했고, 값은 "car-batch-log"로 설정했음
- #5
    - collectDayInterceptor의 설정
    - 플럼 이벤트 바디에 수집된 당일의 작업 날짜(YYYYMMDD)를 추가하기 위한 Interceptor
    - 플럼에서 기본으로 제공하는 Interceptor가 아닌, 이번 파일럿 프로젝트를 위해 추가로 개발한 사용자 정의 Interceptor

- collectDayInterceptor
```java
package com.wikibook.bigdata.smartcar.flume;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;


public class CollectDayInterceptor implements Interceptor {


	public CollectDayInterceptor(){
	}

	@Override
	public void initialize() {

	}

	@Override
	public Event intercept(Event event) {

		String eventBody = new String(event.getBody()) + "," + getToDate();
		event.setBody(eventBody.getBytes());
		return event;

	}


	@Override
	public void close() {
	}

	
	@Override
	public List<Event> intercept(List<Event> events)
	{
		for (Event event:events) {
			intercept(event);
		}
		return events;
	}
	

	public static class Builder implements Interceptor.Builder
	{
		@Override
		public void configure(Context context) {
		}

		@Override
		public Interceptor build() {
			return new CollectDayInterceptor();
		}
	}

	public  String getToDate() {

		long todaytime;
		SimpleDateFormat day;
		String toDay;

		todaytime = System.currentTimeMillis(); 
		day = new SimpleDateFormat("yyyyMMdd");

		toDay =  day.format(new Date(todaytime));

		return toDay;

	}
}
```

- #6
    - HDFS Sink의 상세 설정 값
    - 앞서 등록한 Interceptor의 값을 활용해 HDFS의 경로를 동적으로 파티션하는 "path" 설정과, 적재 시 HDFS에 생성되는 파일명의 규칙, 파일의 크기(64MB) 등 정의

- #7
    - HDFS Sink인 `SmartCarInfo_HdfsSink`를 Memory Channel인 `SmartCarInfo_Channel`과 연결

# 3. 적재 기능 테스트
## 1) 플럼의 사용자 정의 Interceptor 추가
- 사용자 정의 Interceptor인 `collectDayInterceptor`를 플럼의 **Library** 디렉터리에 추가
- '파일질라' 사용

## 2) 플럼의 Conf 파일 수정
- 플럼의 Conf 파일을 HDFS에 적재하는 Sink 구조로 변경

- CM 홈 - [Flume] - [구성] - '구성 파일'
```conf
SmartCar_Agent.sources  = SmartCarInfo_SpoolSource DriverCarInfo_TailSource
SmartCar_Agent.channels = SmartCarInfo_Channel DriverCarInfo_Channel
SmartCar_Agent.sinks    = SmartCarInfo_HdfsSink DriverCarInfo_KafkaSink


SmartCar_Agent.sources.SmartCarInfo_SpoolSource.type = spooldir
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.spoolDir = /home/pilot-pjt/working/car-batch-log
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.deletePolicy = immediate
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.batchSize = 1000


SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors = timeInterceptor typeInterceptor collectDayInterceptor filterInterceptor

SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.timeInterceptor.type = timestamp
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.timeInterceptor.preserveExisting = true

SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.typeInterceptor.type = static
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.typeInterceptor.key = logType
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.typeInterceptor.value = car-batch-log

SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.collectDayInterceptor.type = com.wikibook.bigdata.smartcar.flume.CollectDayInterceptor$Builder

SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.filterInterceptor.type = regex_filter
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.filterInterceptor.regex = ^\\d{14}
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.filterInterceptor.excludeEvents = false

SmartCar_Agent.channels.SmartCarInfo_Channel.type = memory
SmartCar_Agent.channels.SmartCarInfo_Channel.capacity  = 100000
SmartCar_Agent.channels.SmartCarInfo_Channel.transactionCapacity  = 10000


SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.type = hdfs
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.path = /pilot-pjt/collect/%{logType}/wrk_date=%Y%m%d
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.filePrefix = %{logType}
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.fileSuffix = .log
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.fileType = DataStream
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.writeFormat = Text
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.batchSize = 10000
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.rollInterval = 0
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.rollCount = 0
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.idleTimeout = 100
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.callTimeout = 600000
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.rollSize = 67108864
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.hdfs.threadsPoolSize = 10


SmartCar_Agent.sources.SmartCarInfo_SpoolSource.channels = SmartCarInfo_Channel
SmartCar_Agent.sinks.SmartCarInfo_HdfsSink.channel = SmartCarInfo_Channel


SmartCar_Agent.sources.DriverCarInfo_TailSource.type = exec
SmartCar_Agent.sources.DriverCarInfo_TailSource.command = tail -F /home/pilot-pjt/working/driver-realtime-log/SmartCarDriverInfo.log
SmartCar_Agent.sources.DriverCarInfo_TailSource.restart = true
SmartCar_Agent.sources.DriverCarInfo_TailSource.batchSize = 1000

SmartCar_Agent.sources.DriverCarInfo_TailSource.interceptors = filterInterceptor2
SmartCar_Agent.sources.DriverCarInfo_TailSource.interceptors.filterInterceptor2.type = regex_filter
SmartCar_Agent.sources.DriverCarInfo_TailSource.interceptors.filterInterceptor2.regex = ^\\d{14}
SmartCar_Agent.sources.DriverCarInfo_TailSource.interceptors.filterInterceptor2.excludeEvents = false

SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.type = org.apache.flume.sink.kafka.KafkaSink
SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.topic = SmartCar-Topic
SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.brokerList = server02.hadoop.com:9092
SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.requiredAcks = 1
SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.batchSize = 1000


SmartCar_Agent.channels.DriverCarInfo_Channel.type = memory
SmartCar_Agent.channels.DriverCarInfo_Channel.capacity= 100000
SmartCar_Agent.channels.DriverCarInfo_Channel.transactionCapacity = 10000


SmartCar_Agent.sources.DriverCarInfo_TailSource.channels = DriverCarInfo_Channel
SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.channel = DriverCarInfo_Channel
```

- 변경 내용 저장 후 플럼 재시작

<br>

## 3) SmartCar 로그 시뮬레이터 작동
- CH04에서는 스마트카의 상태 정보 로그 파일을 하둡에 적재
- 2개의 시뮬레이터 중 **CarLogMain.java**만 작동시켜 2016년 1월 1일 날짜의 스마트카 상태 정보 로그 파일 생성

<br>

1) Server02에 SSH로 접속하고, bigdata.smartcar.loggen-1.0.jar가 위치한 경로인 /home/pilot-pjt/working 으로 이동
```bash
cd /home/pilot-pjt/working
```

<br>

2) 스마트카 로그 시뮬레이터를 아래의 자바 명령으로 백그라운드 방식으로 실행
- 2016-01-01의 100대의 스마트카 상태 정보 로그 생성
```
java -cp bigdata.smartcar.loggen-1.0.jar com.wikibook.bigdata.smartcar.loggen.CarLogMain 20160101 100 &
```

<br>

3) /home/pilot-pjt/working/SmartCar 경로에 SmartCarStatusInfo_20160101.txt 파일 생성됐는지 확인
- 2016년 1월 1일 날짜로 100대의 스마트카 상태 정보가 기록된 것을 확인할 수 있음
- 최종 로그 파일의 크기는 100MB이고, 생성되기까지 1~2분 정도 걸림

```bash
cd /home/pilot-pjt/working/SmartCar
tail -f SmartCarStatusInfo_20160101.txt
```

![](img/CH04/hdfs%20%EC%8A%A4%EB%A7%88%ED%8A%B8%EC%B9%B4%20%EC%83%81%ED%83%9C%EC%A0%95%EB%B3%B4%ED%8C%8C%EC%9D%BC%20%ED%99%95%EC%9D%B81.png)

<br>

## 4) 플럼 이벤트 작동
- 플럼의 SmartCar 에이전트가 정상적으로 작동하고 있다면, SpoolDir이 참조하고 있는 /home/pilot-pjt/working/car-batch-log 경로에 파일이 생성됨과 동시에 플럼의 파일 수집 이벤트가 작동

<br>

1) /home/pilot-pjt/working/SmartCar 경로에 만들어진 SmartCarStatusInfo_20160101.txt 파일을 플럼의 SmartCarInfo의 SpoolDir 경로인 /home/pilot-pjt/working/car-batch-log로 옮겨서 플럼의 File 이벤트가 작동되도록 함
```bash
mv /home/pilot-pjt/working/SmartCar/SmartCarStatusInfo_20160101.txt /home/pilot-pjt/working/car-batch-log/
```

<br>

2) 플럼의 실행 로그를 통해 SmartCarInfo_Agent가 정상적으로 작동하는지 확인
```bash
cd /var/log/flume-ng/
tail -f /var/log/flume-ng/flume-cmf-flume-AGENT-server02.hadoop.com.log
```

![](img/CH04/)