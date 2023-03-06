[출처: 실무로 배우는 빅데이터 기술, 김강원 저]

# 1. 요구사항 파악

#### 1) 차량의 다양한 장치로부터 발생하는 로그 파일을 수집해서 기능별 상태를 점검한다.
#### 2) 운전자의 운행 정보가 담긴 로그를 실시간으로 수집해서 주행 패턴을 분석한다.

<br>

# 2. 데이터셋 살펴보기

## 1) 스마트카 상태 정보 데이터
- 스마트카의 각종 센서로부터 발생하는 차량의 상태 정보 데이터셋
- 요구사항 1과 관련, 로그 시뮬레이터를 통해 생성됨

## 2) 스마트카 운전자 운행 데이터
- 스마트카 운전자의 운전 패턴 / 운행 정보가 담긴 데이터셋
- 요구사항 2와 관련, 로그 시뮬레이터를 통해 생성됨

## 3) 스마트카 마스터 데이터
- 스마트카 운전자의 프로파일 정보가 담긴 데이터셋
- 요구사항 1, 2와 관련된 분석 데이터셋을 만들 때 활용, 이미 만들어진 샘플 파일 이용

## 4) 스마트카 물품 구매 이력 데이터
- 스마트카 운전자가 차량 내의 스마트 스크린을 통해 쇼핑몰에서 구입한 차량 물품 구매 목록 데이터셋
- 요구사항 1, 2와 관련된 분석 데이터셋을 만들 때 활용, 이미 만들어진 샘플 파일 이용

<br>

# 3. 파일럿 프로젝트 소프트웨어 아키텍처
![sw](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbyLmtt%2FbtrUIZmQfNM%2Fbzo2admbnvciKPiCJTOeck%2Fimg.png)

- 하둡을 중심으로 앞쪽을 수집/적재 (전처리) 영역, 뒤쪽을 탐색/분석 (후처리) 영역

<br>

## 1) 수집 레이어
- `Flume`: 차량의 로그 수집
- `Storm:` 실시간 로그 이벤트 처리
- `Kafka`: 플럼과 스톰 사이에서 데이터의 안정적인 수집을 위해 버퍼링, 트랜잭션 처리
![collect](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcWmWKE%2FbtrUM57qDGA%2FrkDf9rPGJunJGJxaip96s0%2Fimg.png)

<br>

## 2) 적재 레이어
- `Hadoop`, `HBase`, `Redis`
- 대용량 로그파일: 플럼 -> 하둡
- 실시간 데이터: 플럼 -> 카프카 -> 스톰 -> HBase/Redis
- 스톰을 통해 실시간 이벤트 분석 -> 결과에 따라 HBase와 레디스로 나누어 적재
![load](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FFZraV%2FbtrUKmB4dbg%2FIk87w7E4vULbeq0B5XRJx1%2Fimg.png)

<br>

## 3) 처리/탐색 레이어
- `하이브`: 하둡에 적재된 데이터 정제/변형/통합/분리/탐색 등의 작업 수행, 데이터를 정형화된 구조로 정규화해 데이터마트 생성
- `스쿱`: 가공/분석된 데이터 외부로 제공 + 분석/응용 단계에서도 사용
- `우지`: 길고 복잡한 처리/탐색 프로세스를 우지의 워크플로로 구성해 복잡도 낮추고 자동화
![process](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbt92DO%2FbtrUP0K6TGp%2FppmunqNB41RoKZXQlkfMwk%2Fimg.png)

<br>

## 4) 분석/응용 레이어
- `임팔라`, `제플린`: 스마트카 상태 점검과 운전자 운행 패턴 빠르게 분석
- `머하웃`, `스파크ML`: 스마트카 데이터 분석을 위한 군집, 분류/예측, 추천 등
- `R`: 통계 분석
- `텐서플로`: 딥러닝 모델 생성
- `플라스크`: 서비스 API 제공
![analysis](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbLt9Z5%2FbtrUNU5JckW%2FRzRkc1cyi7ytgxY9WTpU0K%2Fimg.png)

<br>

# 4. 하드웨어 아키텍처
![hw](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbqUmw2%2FbtrUOQPvzpr%2FSSz0Knsc6ALZxmUGpErP1K%2Fimg.png)

<br>

# 5. Cloudera Manager (CM)

- 빅데이터 자동화 관리 툴
- 하둡을 포함한 에코시스템 17개 편리하게 설치 및 관리
![cm](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdgrhZY%2FbtrUKYOQmH2%2Fkn9pfO6wlfXFhPPAYhI7U1%2Fimg.png)