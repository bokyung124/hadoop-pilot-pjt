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

- 인스톨 스크립트 실행
- 여러 확인 메시지들 모두 엔터키 입력
```bash
./install_server.sh
```
![](img/CH05/redis%20%EC%84%A4%EC%B9%984.png)