# Spark
- 인메모리 기반
- 대용량 고속 처리 엔진
- 분산 클러스터 컴퓨팅 프레임워크

## Spark 특징
- Speed : In-memory 기반으로 속도가 빠름 
- Easy to Use : 다양한 언어 지원
- Generality : 다양한 컴포넌트 지원(ex) MLib, graphX, streaming 등)
- Run EveryWhere  
  - YARN, Mesos, Kubernetes 등 다양한 클러스터에서 동작 가능
  - HDFS, Casandra, HBase 등 다양한 포멧 지원
- MapReduce 같은 경우 작업 중간 결과를 디스크에다가 저장하기 때문에 IO로 인하여 작업 속도에 제약이 생김

## 컴포넌트 구성
- Spark Core
  - 메인 컴포넌트로 작업 스케줄링, 메모리 관리, 장애 복구와 같은 기본적인 기능을 제공
  - RDD, DataFrame, Dataset의 연산을 처리
- Spark Lib
 - Spark SQL
   - SQL을 이용하여  RDD, DataFrame, Dataset 작업을 생성, 처리
   - Hive metaStore와 연결하여 SQL 작업 처리 가능
 - Spark Streaming
   - 실시간 데이터 스트림을 처리하는 컴포넌트
   - 스트림 데이터를 작은 단위로 쪼개서 RDD 처럼 처리
 - MLib
   - 스파크 기반의 머신러닝 기능을 제공하는 컴포넌트
 - GraphX
    - 다양한 시각화 PLOT 제공
 - cluster Manager 
   - Spark에서 제공하는 stand-alone 관리자를 이용할 수도 있고,   
     Mesos, YARN, kubernetes 등의 관리자를 지원

## Spark 구조
### Spark
- 작업 관리를 하는 드라이버
- 실행되는 노드 관리를 하는 클러스터 매니저

### Spark application 구조

#### Spark application 이란?
- Spark 실행 프로그램
- driver와 executor process로 실행되는 프로그램
- clutser manager가 Spark application의 리소스를 효율적으로 배분

#### Spark application 구성 요소
- Master-Slave 구조
- 작업을 관장하는 driver
- 작업을 수행하는 executor 

#### driver
- main 함수를 실행
- spark-context 객체를 생성
- cluster-manager와 통신하면서 자원 관리 지원
- application life cycle 관리
- 드라이버는 실행 시점에 디플로이 모드를 클라이언트 모드와 클러스터 모드로 설정
  - 클라이언트 모드 : 클러스터 외부에서 드라이버를 실행
  - 클러스터 모드 : 클러스터 내부에서 드라이버를 실행

#### executor
- 실제 작업을 진행하는 프로세스
- executor는 task 단위로 작업을 실행하고 결과를 driver에게 전달
- executor는 동작 중 오류가 발생하면 재수행

#### task
- executor에서 실행되는 실제 작업
- executor의 cash를 공유하여 작업의 속도를 높일 수 있음

#### Spark application 작업
- Job  
  Spark application으로 제출된 작업
- Stage  
  job을 작업의 단위에 따라 구분한 것
- Task  
  executor 실행 단위로 나눈 것

### Clutser manager
- YARN : Hadoop의 clutser manager
- Mesos : apache의 cluster manager
- StandAlone : Spark에서 자체적으로 제공하는 클러스터 매니저

### Spark Monitoring
- 웹UI를 이용하는 방법
- REST API를 이용하는 방법

### Spark Monitoring을 통해 확인 가능한 정보
- stage와 task 목록
- RDD 크기와 메모리 사용량
- 환경변수 정보
- executor의 정보

### Spark application 실행 방법
- 실행은 Scala, Java, python, R로 구현 가능
- jar 파일로 묶어서 실행
- REPL도 가능

### spark-submit
- scala, java로 작성한 Spark application을 jar 파일로 실행할 경우  
  spark-submit을 통해 실행 가능

### log4j 로그 출력
- spark shell이나 다른 작업에서 로그를 출력할 때  
  log4j를 이용하여 로그를 출력할 수 있음
- {SPARK_HOME}/conf/ 아래 log4j.properties 파일을 생성

## RDD, DataFrame, Dataset
- RDD  
  - 인메모리 처리를 통해 속도를 높일 수 있었지만  
    테이블 조인과 같은 처리를 사용자가 직접해야만 했음  
    이에 따라 고수준 API인 DataFrame, Dataset이 나오게 됨

- DataFrame
  - data를 schema 형태로 추상화 함
  - 카탈리스트 옵티마이저가 쿼리를 최적화하여 처리

- Dataset
  - 데이터의 타입 체크
  - 데이터 직렬화를 위한 인코더
  - 카탈리스트 옵티마이저 지원

## 용어 정리
- 스키마
  - 데이터베이스의 구조와 제약 조건에 관한 전반적인 명세를 기술한 메타데이터의 집합
