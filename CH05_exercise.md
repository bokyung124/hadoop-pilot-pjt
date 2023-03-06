# 1. HBase 설치
- CM 홈 - [서비스 추가] - [HBase] 선택 - [계속]
![](img/CH05/hbase%20%EC%84%A4%EC%B9%98.png)

<br>

- HBase의 Master와 RegionServer 설치 위치 지정
- Server01, Server02만 있는 상태이므로, 모든 설치 위치를 Server02로 지정
- 추기로 Thrift Server는 Server01에 설치하며, REST 서버는 사용하지 않음
![](img/CH05/hbase%20%EC%82%AC%EC%9A%A9%EC%9E%90%20%EC%A7%80%EC%A0%95%20-%20server3%EA%B0%80%20%EC%97%86%EC%96%B4%EC%84%9C%20%EC%A0%84%EB%B6%80%20server2%EB%A1%9C.png)

<br>

- HBase Service의 기본 설정값 변경
- HBase의 테이블 복제 및 인덱스는 사용하지 않음
- 기본값을 유지하고 [계속] 클릭

<br>

- HBase의 Thrift Http 서버 활성화
- CM 홈 - [HBase] - [구성]
- "HBase Thrift Http 서버 설정" 검색
- **HBase(서비스 전체)** 항목 체크 후 [변경 내용 저장]
![](img/CH05/hbase%20%EA%B5%AC%EC%84%B1%20%EB%B3%80%EA%B2%BD.png)

<br>

- HBase 서비스 시작 또는 재시작
- 그럼 HBase의 Master 서버와 RegionServer 기동 화면이 활성화될 것
- HBase처럼 규모가 있는 컴포넌트가 설치되면 이미 설치된 소프트웨어(주키퍼, HDFS, YARN 등)의 클라이언트 설정값에 변동 발생 -> 클라이언트 재배포를 통해 변경사항 반영

<br>

- HBase가 정상적으로 설치되었는지 확인하기 위해 HBase 쉘에서 테스트용 테이블을 만들고, 해당 테이블에 Put / Get 명령 실행
```bash
hbase shell
> create 'smartcar_test_table', 'cf'
> put 'smartcar_test_table', 'row-key1', 'cf:model', 'Z0001'
> put 'smartcar_test_table', 'rew-key1', 'cf:no', '12345'
> get 'smartcar_test_table', 'row-key1'
```
![](img/CH05/hbase%20shell%20put%2C%20get%20%ED%85%8C%EC%8A%A4%ED%8A%B8.png)

```bash
# table 삭제
> disable 'smartcar_test_table'
> drop 'smartcar_test_table'
> exit
```
![](img/CH05/hbase%20shell%20disable.png)
![](img/CH05/hbase%20shell%20drop.png)

<br>

- HBase 웹 관리자 화면에 접속하면 HBase의 다양한 상태를 모니터링할 수 있음   
http://server02.hadoop.com:16010
![](img/CH05/hbase%20%EC%9B%B9%EA%B4%80%EB%A6%AC%EC%9E%90%20%ED%99%94%EB%A9%B4.png)

<br>

- HBase는 자원 소모가 높은 서버이므로 사용하지 않을 때는 일시 정지

<br>

# 2. 레디스 설치
- CM의 소프트웨어 컴포넌트로 포함되어 있지 않기 때문에 레디스 설치 패키지로 Server02에 직접 설치

<br>

- Server02에 root 계정으로 로그인
- **gcc**와 **tcl** 설치

```bash
yum install -y gcc*
yum install -y tcl
```

> yum 명령 중 "removing mirrorlist with no valid mirrors:.." 에러가 발생하면 아래 명령 모두 실행
```bash
echo "http: //vault.centos.org/6.10/os/×86_64/" > /var/cache/yum/x86_64/6/base/mirrorlist.txt
echo "http://vault.centos.org/6.10/extras/x86_64/" > /var/cache/yum/×86_64/6/extras/mirrorlist.txt
echo "http://vault.centos.org/6. 10/updates/x86_64/" > /var/cache/yum/x86_64/6/updates/mirrorlist.txt
echo "http://vault.centos.org/6.10/sclo/x86_64/rh" › /var/cache/yum/×86_64/6/centos-sclo-rh/mirrorlist.txt 
echo "http://vault.centos.org/6.10/sclo/x86_64/sclo" >/var/cache/yum/×86_64/6/centos-sclo-sclo/mirrorlist.txt
```

<br>

- 레디스 5.0.7 다운받아 빌드 및 설치 진행
```bash
cd /home/pilot-pjt
wget http://download.redis.io/releases/redis-5.0.7.tar.gz
tar -xvf redis-5.0.7.tar.gz
cd /home/pilot-pjt/redis-5.0.7
make
make install
cd /home/pilot-pjt/redis-5.0.7/utils
chmod 75 install_server.sh
```
![](img/CH05/redis%20%EC%84%A4%EC%B9%981.png)
![](img/CH05/redis%20%EC%84%A4%EC%B9%982-%20make.png)
![](img/CH05/redis%20%EC%84%A4%EC%B9%983.png)

<br>

- 인스톨 스크립트 실행
- 여러 확인 메시지들 모두 엔터키 입력
```bash
./install_server.sh
```
![](img/CH05/redis%20%EC%84%A4%EC%B9%984.png)

<br>

- 레디스 인스턴스의 포트, 로그, 데이터 파일 등의 설정값 및 위치 정보를 물어보면 기본값을 그대로 유지하고 엔터키 입력
- 아래 `vi` 명령으로 레디스 서버 기동 여부 확인
```bash
vi /var/log/redis_6379.log
```
![](img/CH05/redis%20%EC%84%9C%EB%B2%84%20%EA%B8%B0%EB%8F%99%20%EC%97%AC%EB%B6%80%20%ED%99%95%EC%9D%B8.png)

<br>

- 레디스가 성공적으로 설치됐는지 점검하기 위해 아래 명령 실행 후 `Redis is running` 메시지 확인
```bash
service redis_6379 status
```
> redis 서버 포트 기본값: 6379

- 레디스 서비스 시작 `service redis_6379 start`
- 레디스 서비스 종료 `service redis_6379 stop`

<br>

- 레디스 서버에 원격 접근을 하기 위해 설정 파일을 열어 아래와 같이 수정
- 설정 파일은 `/etc/redis/6379.conf`에 위치
```bash
vi /etc/redis/6379.conf
```

- 바인딩 IP 제한 해제
    - `bind 127.0.0.1` 부분 주석 처리
- 패스워드 입력 해제
    - `protected-mode yes` 부분을 찾아 **yes** -> **no** 로 변경
![](img/CH05/redis%20%EC%84%A4%EC%A0%95%ED%8C%8C%EC%9D%BC%20%EC%88%98%EC%A0%95.png)

- 레디스 서버 재시작
```bash
service redis_6379 restart
```

<br>

- **레디스 CLI를 통해 간단히 레디스 서버에 데이터 저장(Set) / 조회(Get)**   
    - `key:1`이라는 키로 "Hello!BigData" 값을 레디스 서버에 저장하고 다시 조회
    - 마지막으로 `key:1`의 데이터 삭제하고 레디스 CLI 종료
```bash
redis-cli
> set key:1 Hello!BigData
> get key:1
> del key:1
> quit
```
![](img/CH05/redis%20%EC%9E%AC%EC%8B%9C%EC%9E%91%20%ED%9B%84%20cli%20%ED%85%8C%EC%8A%A4%ED%8A%B8.png)

<br>

# 3. 스톰 설치
- Server02에 root 계정으로 접속해서 스톰 설치를 위한 tar 파일 다운로드
```bash
cd /home/pilot-pjt
wget http://archive.apache.org/dist/storm/apache-storm-1.2.3/apache-storm-1.2.3.tar.gz
tar -xvf apache-storm-1.2.3.tar.gz
ln -s apache-storm-1.2.3 storm
```

<br>

- **스톰 환경설정 파일 수정**
- `storm.yaml` 파일 열어서 아래 내용과 동일하게 입력
```bash
cd /home/pilot-pjt/storm/conf
vi storm.yaml
```
```
storm.zookeeper.servers:
 - "server02.hadoop.com"

storm.local.dir: "/home/pilot-pjt/storm/data"

nimbus.seeds: ["server02.hadoop.com"]

supervisor.slots.ports:
 - 6700

ui.port: 8088
```
![](img/CH05/storm.yaml%20%ED%8C%8C%EC%9D%BC%20%EC%88%98%EC%A0%95.png)

- 5개의 스톰 설정값
    - 주키퍼 정보
    - 스톰이 작동하는 데 필요한 데이터 저장소
    - 스톰의 Nimbus 정보
    - Worker의 포트로서, 포트의 개수만큼 멀티 Worker가 만들어짐
        - 파일럿 프로젝트에서는 6700번 포트의 단일 워커만 작동
    - 마지막으로 스톰 UI 접속 포트 설정

<br>

- **스톰의 로그 레벨 조정**
- 기본값인 "info"로 되어있는데, 스톰에서는 대규모 트랜잭션 데이터가 유입되면서 과도한 로그로 인해 성능 저하와 디스크 공간 부족이 발생할 수 있음
- 이 같은 문제르 방지하기 위해 두 개의 파일 수정
    - cluster.zml , worker.xml 파일
    - 75번째 줄과 91번째 줄 사이의  logger 설정에서 `level="info"` -> `level="error`로 모두 변경

```bash
cd /home/pilot-pjt/storm/long4j2
vi cluster.xml
vi worker.xml
```
![](img/CH05/storm%20%EB%A1%9C%EA%B7%B8%20%EB%A0%88%EB%B2%A8%20%EC%A1%B0%EC%A0%95.png)

<br>

- **root 계정의 프로파일의 스톰 path 설정**
```bash
vi /root/.bash_profile
```
- `/home/pilot-pjt/storm/bin` 경로 추가

![](img/CH05/strom%20path%20%EC%84%A4%EC%A0%95.png)

- **수정한 root 계정의 프로파일 정보 다시 읽어옴**
```bash
source /root/.bash_profile
```

<br>

- CH02에서 설정했던 자바 명령의 링크 정보가 변경되는 경우가 있음
- `java -version`을 실행한 후, '1.8.x'가 아니면 아래 명령을 통해 JDK 1.8 링크로 변경
```bash
rm /usr/bin/java
rm /usr/bin/javac
ln -s /usr/java/jdk1.8.0_181-cloudera/bin/javac /usr/bin/javac
ln -s /usr/java/jdk1.8.0_181-cloudera/bin/java /usr/bin/java
```

<br>

- 기본적인 스톰 설치 완료
- 리눅스가 재시작할 때도 스톰이 자동으로 실행되도록 설정
- `storm-nimbus`, `storm-supervisor`, `storm-ui` 3개의 스톰 서비스가 있고, 3개의 자동 실행 스크립트 필요
- 파일질라 이용해서 sever02에 3개의 스크립트 생성
![](img/CH05/%EC%8A%A4%ED%86%B0%20%EC%84%9C%EB%B9%84%EC%8A%A4%20%EC%9E%90%EB%8F%99%20%EC%8B%A4%ED%96%89%20%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%20%EC%97%85%EB%A1%9C%EB%93%9C.png)

<br>

- **업로드한 세 파일의 권한 변경**
```bash
chmod 755 /etc/rc.d/init.d/storm-nimbus
chmod 755 /etc/rc.d/init.d/storm-supervisor
chmod 755 /etc/rc.d/init.d/storm-ui
```
![](img/CH05/storm%20%EC%9E%90%EB%8F%99%EC%8B%A4%ED%96%89%20%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%20%ED%8C%8C%EC%9D%BC%20%EA%B6%8C%ED%95%9C%20%EB%B3%80%EA%B2%BD.png)

<br>

- **서비스 등록 스크립트에 대한 Log 및 Pid 디렉터리 생성**
```bash
mkdir /var/log/storm
mkdir /var/run/storm
```
![](img/CH05/%EC%84%9C%EB%B9%84%EC%8A%A4%20%EB%93%B1%EB%A1%9D%20%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EC%97%90%20%EB%8C%80%ED%95%9C%20%EB%94%94%EB%A0%89%ED%84%B0%EB%A6%AC.png)

<br>

- **세 파일에 대해 아래의 service/chkconfig 등록 명령 각각 실행
```bash
  
```

<br>

- **아래 명령으로 스톰이 정상적으로 구동됐는지 확인**
```bash
service storm-nimbus status
service storm-supervisor status
service storm-ui status
```
- 모두 `...is running` 상태인지 확인

<br>

- 스톰 UI에 접속해서 스톰의 중요 상태들을 모니터링할 수 있음   
http://server02.hadoop.com:8088
![](img/CH05/storm%20ui.png)

<br>

# 4. 실시간 적재 기능 구현
