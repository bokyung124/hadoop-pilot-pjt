# 1. Flume ��ġ
- CM Ȩ���� [���� �߰�] - [Flume] ����
![](/img/CH03/flume%20%EC%84%A4%EC%B9%98.png)   
- ���� ȣ��Ʈ `server02.hadoop.com` ����

<br>

- __Flume Heap Memory ����__
    - 50Mib -> __100MiB__
    ![](img/CH03/flume%20heap%20memory%20%EB%B3%80%EA%B2%BD.png)   

<br>

# 2. Kafka ��ġ
- CM Ȩ���� [���� �߰�] - [Kafka] ����
![](img/CH03/kafka%20%EC%84%A4%EC%B9%98.png)   
- ���� ȣ��Ʈ `server02.hadoop.com` ����

<br>

- __ī��ī�� ����� �޽��� ���� �Ⱓ ª�� ����__
    - `Data Retention Time` 7�� -> __10��__
    ![](img/CH03/kafka%20data%20retention%20time%20%EB%B3%80%EA%B2%BD.png)

<br>

# 3. �÷� ���� ��� ����
- �÷����� 2���� ������Ʈ ����
    - ����Ʈī �������� �����ϴ� `SmartCarInfo Agent`
    - �������� �������� �����ϴ� `DriverCarInfo Agent`

- ������Ʈ ���� 
  - �÷��� �ν��� �� �ִ� Ư�� ���͸��� `{Agent ���� �̸�}.conf` ���� ����
  - ���Ϸ� ������Ʈ������ CM���� �����ϴ� �÷� ���� ���� ������ ���� ������Ʈ ���ϰ� ���� ����

## 1) SmartCar ������Ʈ ����
- CM Ȩ [Flume] - [����]
- `���� ����` �׸� ����

<br>

- Agent �̸�: `SmartCar_Agent`
- ���� ����   
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

- #1
  - �÷��� ������Ʈ���� ����� Source, Channle, Sink�� �� ���ҽ� ���� ����
- #2
	- ������Ʈ�� Source ����
	- #1���� Source�� �����ߴ� `SmartCarInfo_SpoolSource` ������ type�� `spooldir`�� ����
	- `spooldir` : ������ Ư�� ���͸��� ����͸��ϰ� �ִٰ� ���ο� ������ �����Ǹ� �̺�Ʈ�� �����ؼ� `batchSize` ��������ŭ �о #3�� Channel�� ������ ����
- #3 
	- ������Ʈ�� Channel�μ� `SmartCarInfo_Channel`�� type�� `memory`�� ����
	- ä���� ����) memory / file
	- Memory Channel�� Source�κ��� ���� �����͸� �޸� �� �߰� ���� -> ���� ������, ������ ����
	- File Channel�� Source���� ������ �����͸� �޾� ���� ���Ͻý��� ����� `dataDirs`�� �ӽ÷� �����ߴٰ� Sink���� ������ ���� -> ���� ������, ������ ����
- #4
	- ������Ʈ�� ���� ������
	- SmartCarInfo_LoggerSink�� type�� `logger`�� ����
	- Logger Sink�� ������ �����͸� �׽�Ʈ �� ����� �������� �÷��� ǥ�� ��� �α� ������ `/var/log/flume-ng/flume-cmf-flume-AGENT-server02.hadoop.com.log`�� ���
- #5
	- Source�� Channel Sink ����
	- �ռ� ������ SmartCarInfo_SpoolSource�� ä�ΰ��� `SmartCarInfo_Channel`�� ����
	- SmartCarInfo_LoggerSink�� ä�ΰ��� `SmartCarInfo_Channel`�� ����
	- File -> Channel -> Sink�� �̾����� ������Ʈ ���ҽ��� �ϳ��� ����

<br>

## 2) SmartCar ������Ʈ�� Interceptor �߰�
- `Interceptor` : Source�� Channel�� �߰����� �����͸� �����ϴ� ����
- �÷��� Source���� ���ԵǴ� ������ �� �Ϻ� �����͸� �����ϰų� �ʿ��� �����͸� ���͸��ϴ� �� �߰��� �����͸� �߰�/����/�����ϴ� �� ����
- �÷����� ������ ���� ���� `Event` - Header�� ���� Body�� ������
- Interceptor�� Event�� Header�� Ư������ �߰��ϰų� Body�� �����͸� �����ϴ� ������� Ȱ���

<br>

- ���Ϸ� ������Ʈ������ SmartCarInfo �α� ������ �����ϴµ� �� 4���� Interceptor�� �߰��� ��
- �̹� �忡���� `Filter Interceptor` �ϳ��� �߰�
- �ռ� �ۼ��� SmartCarInfo ������Ʈ �����ؼ� Filter Interceptor ���

<br>

- CM Ȩ [Flume] - [����] - [���� ����]
- Source�� Channel ���̿� Interceptor �߰�
```conf
SmartCar_Agent.sources  = SmartCarInfo_SpoolSource
SmartCar_Agent.channels = SmartCarInfo_Channel
SmartCar_Agent.sinks    = SmartCarInfo_LoggerSink 

SmartCar_Agent.sources.SmartCarInfo_SpoolSource.type = spooldir
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.spoolDir = /home/pilot-pjt/working/car-batch-log
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.deletePolicy = immediate
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.batchSize = 1000

#1
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors = filterInterceptor

#2
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.filterInterceptor.type = regex_filter
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.filterInterceptor.regex = ^\\d{14}
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.filterInterceptor.excludeEvents = false


SmartCar_Agent.channels.SmartCarInfo_Channel.type = memory
SmartCar_Agent.channels.SmartCarInfo_Channel.capacity  = 100000
SmartCar_Agent.channels.SmartCarInfo_Channel.transactionCapacity  = 10000


SmartCar_Agent.sinks.SmartCarInfo_LoggerSink.type = logger

SmartCar_Agent.sources.SmartCarInfo_SpoolSource.channels = SmartCarInfo_Channel
SmartCar_Agent.sinks.SmartCarInfo_LoggerSink.channel = SmartCarInfo_Channel
```

- #1
	- ���� �����͸� ���͸��ϱ� ���� `filterInterceptor` ������ �����ؼ� SmartCarInfo_SpoolSource�� �Ҵ�
- #2
	- filterInterceptor�� type�� `regex_filter`�� ����
	- ���� ǥ����(Regular Expression)�� �̿��� ���� �����͸� ���͸��� �� �����ϰ� ���
	- �Ʒ� �̹������� ����Ʈī �α� �����Ⱑ ���� �α� ���� ������ ����, �߰��� �α� ������ ������ �˸��� ��Ÿ ������ ���ԵǾ� ����
	- �� ��Ÿ ������ ���� �����Ϳ��� �����ϰ�, �ʿ��� �����͸� �����ؾ� ��
	- �̶� ������ ���� ǥ������ �̿��ؼ� �ذ� ����
		- ����Ʈī �α��� ���) �������� �α� �����Ͱ� �߻����� �� 14�ڸ��� ��¥ ������ ����
		- �� 14�ڸ� ��¥ �������� �����ϴ� �����Ϳ� ���� ���� ǥ���� `^\\d{14}`�� "regex" �Ӽ��� ����
		- "excludeEvents" �Ӽ��� __false__ �� �Ǿ��ִµ�, __true__ �� �ϸ� �ݴ�� ���� ��� �����ϰ� ��

<br>

## 3) DriverCarInfo ������Ʈ ����
- �ռ� �ۼ��� SmartCar ������Ʈ�� DriverCarInfo ������Ʈ�� ���� Source, Channel, Sink�� �߰��Ͽ� ����
- �� ���� �÷� ������Ʈ ���Ͽ� ���� ���� ������Ʈ�� ����� ���

<br>

- __DriverCarInfo ���ҽ� ���� �߰�__
```conf
#1
SmartCar_Agent.sources  = SmartCarInfo_SpoolSource DriverCarInfo_TailSource
SmartCar_Agent.channels = SmartCarInfo_Channel DriverCarInfo_Channel
SmartCar_Agent.sinks    = SmartCarInfo_LoggerSink DriverCarInfo_KafkaSink

SmartCar_Agent.sources.SmartCarInfo_SpoolSource.type = spooldir
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.spoolDir = /home/pilot-pjt/working /car-batch-log
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.deletePolicy = immediate
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.batchSize = 1000

SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors = filterInterceptor

SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.filterInterceptor.type = regex_filter
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.filterInterceptor.regex = ^\\d{14}
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.filterInterceptor.excludeEvents = false


SmartCar_Agent.channels.SmartCarInfo_Channel.type = memory
SmartCar_Agent.channels.SmartCarInfo_Channel.capacity  = 100000
SmartCar_Agent.channels.SmartCarInfo_Channel.transactionCapacity  = 10000


SmartCar_Agent.sinks.SmartCarInfo_LoggerSink.type = logger

SmartCar_Agent.sources.SmartCarInfo_SpoolSource.channels = SmartCarInfo_Channel
SmartCar_Agent.sinks.SmartCarInfo_LoggerSink.channel = SmartCarInfo_Channel
```
- #1
	- ������Ʈ�� Source, Channel, Sink�� DriverCarInfo ���ҽ� ���� �߰�

<br>

- __DriverCarInfo ���� �߰�__

```conf
SmartCar_Agent.sources  = SmartCarInfo_SpoolSource DriverCarInfo_TailSource
SmartCar_Agent.channels = SmartCarInfo_Channel DriverCarInfo_Channel
SmartCar_Agent.sinks    = SmartCarInfo_LoggerSink DriverCarInfo_KafkaSink

SmartCar_Agent.sources.SmartCarInfo_SpoolSource.type = spooldir
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.spoolDir = /home/pilot-pjt/working/car-batch-log
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.deletePolicy = immediate
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.batchSize = 1000

SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors = filterInterceptor

SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.filterInterceptor.type = regex_filter
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.filterInterceptor.regex = ^\\d{14}
SmartCar_Agent.sources.SmartCarInfo_SpoolSource.interceptors.filterInterceptor.excludeEvents = false


SmartCar_Agent.channels.SmartCarInfo_Channel.type = memory
SmartCar_Agent.channels.SmartCarInfo_Channel.capacity  = 100000
SmartCar_Agent.channels.SmartCarInfo_Channel.transactionCapacity  = 10000


SmartCar_Agent.sinks.SmartCarInfo_LoggerSink.type = logger

SmartCar_Agent.sources.SmartCarInfo_SpoolSource.channels = SmartCarInfo_Channel
SmartCar_Agent.sinks.SmartCarInfo_LoggerSink.channel = SmartCarInfo_Channel

#1
SmartCar_Agent.sources.DriverCarInfo_TailSource.type = exec
SmartCar_Agent.sources.DriverCarInfo_TailSource.command = tail -F /home/pilot-pjt/working/driver-realtime-log/SmartCarDriverInfo.log
SmartCar_Agent.sources.DriverCarInfo_TailSource.restart = true
SmartCar_Agent.sources.DriverCarInfo_TailSource.batchSize = 1000

SmartCar_Agent.sources.DriverCarInfo_TailSource.interceptors = filterInterceptor2

#2
SmartCar_Agent.sources.DriverCarInfo_TailSource.interceptors.filterInterceptor2.type = regex_filter
SmartCar_Agent.sources.DriverCarInfo_TailSource.interceptors.filterInterceptor2.regex = ^\\d{14}
SmartCar_Agent.sources.DriverCarInfo_TailSource.interceptors.filterInterceptor2.excludeEvents = false

#3
SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.type = org.apache.flume.sink.kafka.KafkaSink
SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.topic = SmartCar-Topic
SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.brokerList = server02.hadoop.com:9092
SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.requiredAcks = 1
SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.batchSize = 1000

#4
SmartCar_Agent.channels.DriverCarInfo_Channel.type = memory
SmartCar_Agent.channels.DriverCarInfo_Channel.capacity= 100000
SmartCar_Agent.channels.DriverCarInfo_Channel.transactionCapacity = 10000

#5
SmartCar_Agent.sources.DriverCarInfo_TailSource.channels = DriverCarInfo_Channel
SmartCar_Agent.sinks.DriverCarInfo_KafkaSink.channel = DriverCarInfo_Channel
```

- �ռ� ������ SmartCarInfo ������Ʈ�� �����ϰ� Source, Interceptor, Channel, Sink ������� ����
- �Ϻ� Source�� Sink ������ �޶���

<br>

- #1 
	- Source�� type�� `exec`
	- `exec`�� �÷� �ܺο��� ������ ����� ����� �÷��� Event�� ������ ������ �� �ִ� ��� ����
	- ����Ʈī �������� ���� ������ �α� �ùķ����͸� ���� `/home/pilot-pjt/working/driver-realtime-log/SmartCarDriverInfo.log`�� ����
	-> �������� `tail` ����� �÷��� `exec`�� �����ؼ� �������� �ǽð� ���� ���� ����

- #2
	- Interceptor ���� 
	- ������ ���͸��� ���� `regex_filter`�� �߰�

- #3
	- ����Ʈī �������� �ǽð� ���� ������ �÷����� ������ ���ÿ� ī��ī�� ����
	- �÷��� `KafkaSink`�� ������ ����, ī��ī ���Ŀ ������ �������� server02.hadoop.com:9092�� �����ؼ� SmartCar-Topic�� �����͸� 100���� ��ġ ũ��� ����

- #4
	- DriverCarInfo�� Channel�� `Memory Channel`�� ����

- #5
	- DriverCarInfo�� Source�� Sink�� Channel�� �ռ� ������ DriverCarInfo_Channel�� �����ؼ� Source-Channel-Sink ���� �ϼ�

<br>

# 4. ī��ī ��� ����
- ī��ī�� ���� �������� ��� ������ ���� ����
- �̹� �÷��� `DriverCarInfo_KafkaSink`�� ���� ������ �ǽð� �����͸� ī��ī�� �����ϴ� ��� ������ ������ ����
- ī��ī ��ɾ �̿��� ī��ī�� `���Ŀ` �ȿ� ������ ����ϰ� �� ���� ���� & ī��ī�� `Producer` ����� ���� ���ȿ� ������ ����
- ���ȿ� �� �����͸� �ٽ� ī��ī�� `Consumer` ��ɾ�� ����

<br>

## 1) ī��ī Topic ����
- ī��ī�� ��ġ�Ǿ� �ִ� Server02���� ī��ī�� `CLI` ��ɾ �̿��� �پ��� ī��ī ��� ���
- PuTTY�� Server02�� SSH ���� - root �������� �α���
- �Ʒ� ī��ī ���� ��ɾ�� SmartCar-Topic ����
```bash
kafka-topics --create --zookeeper server02.hadoop.2181 --replication-factor 1 --partitions 1 --topic SmartCar-Topic
```

<br>

- �� ��ɾ� �����ϰ� `Created Topic SmartCar-Topic` �޽����� ������ ������ ���������� ������ ��
- `--zookeeper` �ɼ�
	- ������ ��Ÿ �������� Zookeeper�� Z��忡 ��������� ������
	- replication-factor �ɼ��� ī��ī�� ���� Broker�� �����, ������ �����͸� replication-factor ������ŭ �����ϰ� ��. ���Ϸ� ������ ���� ī��ī ���Ŀ -> ���� ��� 1
	- partitions �ɼ��� �ش� Topic�� �����͵��� partitions ������ŭ �и������. ���� Broker�ּ� ����/�б� ���� ����� ���� ���
- `--topic` �ɼ�
	- ���Ϸ� ȯ�濡�� ����� ���ȸ� ����
	- `SmartCar-Topic` �̶�� �̸����� ���� ����
	- �÷��� DriverCarInfo_KafkaSink���� ������ ���� �̸��� ���ƾ� ��

<br>

## 2) ī��ī Producer ���
- Server02 SSH ���� �� �Ʒ� ��� ����
```bash
kafka-console-producer --broker-list server02.hadoop.com:9092 -topic SmartCar-Topic
```
