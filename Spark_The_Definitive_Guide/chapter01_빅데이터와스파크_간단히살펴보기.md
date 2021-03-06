## chapter01 아파치 스파크란
### Spark의 장점
- Spark는 하둡의 가장 큰 특징 중 하나인 MapReduce의 느린 연산 처리 속도를 개선하기 위해  
  in-memory 방식의 처리를 기반으로 연산 속도를 개선  
- Apache Hive의 장점인 HQL을 사용할 수 있고, SQL에 친숙한 사용자들을 위하여  
  Spark SQL 제공
- Apache Storm의 스트리밍 처리 기술을 지원하는 Spark Streaming을 제공
- ML 처리를 위한 MLlib 라이브러리 제공  
- Spark GraphX를 제공
- Spark을 사용해 빅데이터 처리 및 분석과 관련된 모든 과정을 수행 가능  
- 단일 프로그래밍 인터페이스를 통해 구현할 수 있는 능력을 제공  
- MapReduce의 비효율성(중간 결과의 Disk 저장)을 제거하여 특정 상황에서 기존 Framework보다  
  100배 이상 성능을 발휘함
- Apache Spark은 Spark 2.0을 출시하면서 구조적 API에 큰 공을 들였는데,  
  과거 RDD(탄력적 분산 데이터셋)을 통해 함수형 프로그래밍 방식으로 데이터를 조작해야 했음  
  최근 소개된 구조적 API는 복잡함을 제거하여 처리과정에 집중할 수 있게 함

### 아파치 스파크란
- 통합 컴퓨팅 엔진이며, 클러스터 환경에서 데이터를 병렬로 처리하는 라이브러리 집합
- python, Java, Scala, R 언어 지원하며, SQL 뿐만 아니라 스트리밍, ML까지 포함한 다양한  
  범위의 라이브러리 제공
- 단일 노트북 환경에서 부터 수천대의 서버로 구성된 클러스터까지 다양한 환경에서 실행 가능  

~~~
# 스파크 기능 구성
     -------------------------------------------------------------
    | 구조적 스트리밍   |  고급 분석 | 라이브러리 및 에코 시스템  |
     -------------------------------------------------------------
    |             구조적 API(Dataset, DataFrame, SQL)             |
     -------------------------------------------------------------
    |                저수준 API(RDD, 분산형변수)                  |
     -------------------------------------------------------------
~~~

### 1.1 아파치 스파크의 철학
Apache Spark를 설명하는 <b>빅데이터를 위한 통합 컴퓨팅 엔진과   
라이브러리 집합</b> 이라는 문장 기준하에 spark의 핵심요소를 파악해보자

#### 통합
- Spark은 <b>빅데이터 애플리케이션 개발에 필요한 통합 플랫폼을 제공하자</b>라는 핵심 목표를 가지고 있음
- 여기서의 통합은 다양한 데이터 분석 작업(읽기,SQL, ML등..)을 다양한 처리유형과 라이브러리를 결합해 수행한다는 것을 의미함
- 예를들어, SQL 쿼리로 데이터를 읽고 ML 라이브러리로 ML 모델을 평가 할 경우, Spark 엔진은  
  이 두단계를 하나로 병합하고 데이터를 한 번만 조회할 수 있게 해줌
- 스파크가 발표되기 전에는 통합 엔진을 제공하는 병렬 데이터 처리용 오픈소스가 없었음    
  따라서 사용자는 다양한 API와 시스템을 직접 조합하여 Application을 작성해야 했음 

#### 컴퓨팅 엔진
- Spark은 통합이라는 관점을 중시하면서 기능의 범위를 컴퓨팅 엔진으로 제한  
  (보통 컴퓨팅이라함은, 계산을 의미함)
  그 결과 <b> Spark은 저장소 시스템의 연산하는 역할만 수행, 영구 저장소 역할은 수행하지 않음</b>  
- 대신 cloud 기반 에저 스토리지, 아마존 S3, Apache Hadoop,  
  key/value형식인 Apache Cassandra, 메세지 전달 서비스인 Apache kafka 등의 저장소를 지원 
- Spark은 내부에 데이터를 오랜 시간 저장하지 않으며, 특정 저장 시스템을 선호하지도 않음  
- Spark 연산 기능에 초점을 맞추면서 Apache Hadoop 같은 기존 BigData Plaform과 차별화 하고 있음
- Hadoop은 범용 서버 클러스터 환경에서 저비용 저장 장치를 사용하도록 설계된 HDFS과 컴퓨팅 시스템(MapReduce)를 가지고 있으며, 두 시스템은 밀접하게 연관되어 있어 하나의 시스템만 단독으로 사용이 어려움  
- Hadoop은 특히 다른 저장소의 데이터에 접근하는 Application과 개발하기 어려움  
- Spark은 하둡 저장소와 잘 호환되며, 연산 노드와 저장소를 별도로 구매할 수 있는 공개형 cloud환경이나 스트리밍 애플리케이션이 필요한 환경 등 하둡 아키텍처를 사용할 수 없는 환경에서도 많이 사용됨  

#### 라이브러리
- Spark 컴포넌트는 데이터 분석 작업에 필요한 통합 API를 제공하는 통합 엔진 기반의 자체 Library임
- Spark 엔진에서 제공하는 표준 라이브러리와 오픈소스 커뮤니티에서 서드파티 패키지 형태로  
  제공하는 다양한 외부 라이브러리 제공  
- Spark SQL, MLlib, Spark Streaming, 구조적 Streaming, GraphX library 제공
- 외부 라이브러리 목록은 spark-package.org에서 확인 가능
 
### 1.2 스파크의 등장 배경
- 데이터 분석에 새로운 처리 엔진과 프로그래밍 모델이 필요한 근본적인 이유는  
  컴퓨터 어플리케이션과 H/W에 바탕을 이루는 경제적 요인의 변화 때문  
- 역사적으로 컴퓨터는 프로세서 성능의 향상에 힘입어 매년 빨라졌는데, 2005년경에 H/W의 성능 향상이 멈춤  
  (물리적인 방열 한계로 인해 H/W 개발자들은 단일 프로세서를 향상 시키는 방법 대신 모든 코어가 같은 속도로 동작하는 병렬 CPU 코어를 더 많이 추가하는 방향으로 선회)
- 이러한 현상은 애플리케이션의 성능 향상을 위해 병렬 처리가 필요하며,  
  Spark와 같은 새로운 프로그래밍 모델의 세상이 도래할 것임을 암시  
- 이러한 새로운 기술 덕분에 데이터 저장과 수집 기술은 눈에 띄게 느려지지 않음 
- 결과적으로 데이터 수집 기술은 극히 저렴해 졌지만, 데이터는 클러스터에서 처리해야 할 만큼 거대해짐  
- 지난 50년간 개발된 소프트웨어는 더는 자동으로 성능이 향상되지 않았고, 데이터 처리 어플리케이션에 적용한 전통적인 프로그래밍 모델도 더는 힘을 발휘하지 못함  
- 이러한 문제를 해결하기 위해 Apache Spark가 탄생!  

### 1.4 스파크의 현재와 미래
- 새로운 고수준 스트리밍 엔진인 구조적 스트리밍을 2016년에 소개  
  Netfilx, Uber 등 대량 데이터 기반으로 프로젝트를 수행하는 곳에서 활용도가 점점 높아지고 있음  

### 1.5 스파크 실행하기
- 사용자는 python, java, scala, R, SQL 언어에서 스파크 사용 가능  
- Spark은 Scala로 구현되어 JVM 기반으로 동작함  
  따라서 노트북이나 클러스터 환경에서 Spark을 실행하려면 Java를 설치해야 함  
  파이썬을 사용하려면 python intepreter 2.7 version 이상, R 사용시 R 설치해야 함

#### Scala 언어 기반 Spark 사용하기 위한 설치 요약
- jdk 1.8 설치
- scala 2.11 설치
- spark 내려받기
  - 해당 서적의 code를 실행시키려면 2.3.2 or 2.2.2 version 설치
  - https://archive.apache.org/dist/spark/spark-2.3.2/
  -  spark-2.3.2-bin-hadoop2.7.tgz 설치 후 원하는 디렉토리에 압축 해제
- window 10에서 설치시, winUtils download 필요  
  하단 블로그 참조
- spark 실행하기
  - 로컬 환경에서 spark 직접 실행하기
    - bin directory 이동
    - cmd file은 window용, 확장자가 없는 파일은 Mac OS용
    - spark 실행에 필요한 각각의 파일은 다음과 같은 역할을 함
      - Linux
        - spark-shell : 기본
        - pyspark : PySpark
        - sparkR : SparkR
        - spark-sql : SparkSQL
        - spark-submit : 스파크 어플리케이션
      - Window
        - spark-shell.cmd/spark-shell2.cmd : 기본
        - pyspark.cmd/pyspark2.cmd : PySpark
        - sparkR.cmd/sparkR2.cmd : RSpark
        - spark-submit.cmd spark-submit2.cmd : 스파크 어플리케이션
  - 클라우드 환경에서 spark 실행하기
  - docker 기반 환경에서 spark 실행하기
- window 10에서 spark 설치 참고 blog  
  https://jjangjjong.tistory.com/24

- 스파크 대화형 콘솔 실행하기
  - 스칼라 콘솔 실행하기 : spark-shell 실행
  - 파이썬 콘솔 실행하기 : pyspark 실행
  - 해당 명령을 수행한 다음 'spark'을 실행하고 엔터키를 누르면 파이썬 환경과 동일하게
    SparkSession 객체가 출력됨  
  - SQL 콘솔 수행하기 : spark-sql

- 클라우드에서 스파크 실행하기
  - 스파크 학습용으로 대화형 notebook을 사용하고 싶은 경우, 데이터브릭스 커뮤니티 판을 사용하면 됨

#### 데이터브릭스 이용하기
- 데이터브릭스는 회사명이자, spark 런타임 솔루션 명
- 데이터브릭스는 클라우드 환경을 기반으로 스파크 실행 환경 제공  
- 데이터브릭스는 관리형 클라우드 이므로, 다음과 같은 기능 제공
  - 관리형 spark 클러스터 환경
  - 대화형 데이터 탐색 및 시각화 기능
  - 운영용 파이프라인 스케줄러
  - 선호하는 스파크 기반 애플리케이션을 위한 플랫폼
- FREE TRIALS 을 다운받아 사용해 보면 됨



### 용어 정리
- HQL(Hypernate Query Language)
  - SQL과 비슷하며, 추가적인 컨밴션을 정의할 수 있음  
  - 완전히 객체지향적이며, 상속, 다형성, 캡슐화 등이 구현 가능  

- 대화형 데이터 분석
  - 사용자가 입력한 query에 바로 반응하여 결과를 반환하여 주는 분석 방법