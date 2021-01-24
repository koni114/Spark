## 스파크 간단히 살펴보기
- DataFrame과 SQL을 사용해 클러스터, Spark 어플리케이션, 구조적 API를 살펴보는 챕터

### 2.1 Spark의 기본 아키텍처
- 데이터를 처리할 때 집에있는 한대의 컴퓨터를 가지고 처리하기란 어려운 일
- <b>컴퓨터 클러스터</b>는 여러 컴퓨터의 자원을 모아 하나의 컴퓨터처럼 사용할 수 있게 해줌  
  그러나 컴퓨터 클러스터를 구성하는 것만으로는 부족하며,   
  클러스터에서 작업을 조율할 수 있는 프레임워크가 필요
- Spark는 클러스터의 처리 작업을 관리, 조율함
- Spark가 연산에 사용할 클러스터는 <b>Spark stand-alone 클러스터 매니저, 하둡 YARN, 메소스 </b> 같은 클러스터 매니저에서 관리
- 사용자는 클러스터 매니저에 Spark 애플리케이션을 제출(submit)함  
  이를 받은 클러스터 매니저는 애플리케이션에 필요한 자원을 할당  
  우리는 할당받은 자원으로 작업을 처리

### 2.1.1 Spark 애플리케이션
~~~
  드라이버 프로세스         익스 큐터
--------------------      --
|  SparkSession    |     |  |
|     ^    v       |      --        --
|   사용자 코드    |               |  |
--------------------                --
              ^ v         ^ v       ^ v
-------------------------------------------------             
|            클러스터 매니저                    |
-------------------------------------------------
~~~
- 클러스터 매니저가 물리적 머신을 관리하고 스파크 애플리케이션에 자원을 할당하는 방법을 나타냄
- 클러스터 매니저는 Spark Stand-alone 클러스터 매니저, 하둡 YARN, 메소스 중 하나 선택 가능
- 하나의 클러스터에서 여러 개의 Spark 애플리케이션 실행 가능
- 사용자는 클러스터 노드에 할당할 익스큐터 수를 지정 가능 


#### Spark 애플리케이션을 이해하기 위한 핵심 사항
  - Spark는 사용 가능한 자원을 파악하기 위해 클러스터 매니저 사용
  - 드라이버 프로세스는 주어진 작업을 완료하기 위해 드라이버 프로그램의 명령을 익스큐터에서  
    실행할 책임이 있음

- Spark 애플리케이션은 드라이버 프로세스와 다수의 익스큐터(executor) 프로세스로 구성

#### 드라이버 프로세스
-  Spark 애플리케이션에서의 심장과 같은 존재
- 주요 역할
  - 드라이버 프로세스는 클러스터 노드 중 하나에서 실행되며 main() 함수를 실행
  - 전반적인 익스큐터 프로세스의 작업과 관련된 분석 수행
  - 스파크 애플리케이션 정보의 유지 관리
  - 사용자 프로그램이나 입력에 대한 응답
  - 배포, 스케줄링 역할 수행
- 스파크의 언어 API를 통해 다양한 언어로 실행 가능

#### 익스큐터
- 드라이버 프로세스가 할당한 작업 수행
- 드라이버가 할당한 코드를 실행하고, 진행 상황을 다시 드라이버 노드에 보고

### 2.2 스파크의 다양한 언어 API
- 스파크의 언어 API를 이용하면 다양한 프로그래밍 언어로 스파크 코드를 실행 할 수 있음
- Spark는 모든 언어에 맞는 '핵심 개념' 을 제공
- 이러한 핵심 개념은 클러스터 머신에서 실행되는 스파크 코드로 변환됨
- 다음 목록은 언어별 요약 정보임  
  - Scala
    - Spark은 Scala로 개발되어 있으므로, Scala가 Spark의 기본 언어  
  - Java
    - Spark가 Scala로 개발되어 있지만, Spark 창시자들은 Java를 이용해 Spark 코드를 작성 할 수 있도록 심혈을 기울임
  - Python
    - 스칼라가 지원하는 거의 모든 구조를 지원
  - SQL
    - Spark은 ANSI SQL:2003 표준 중 일부를 지원
    - 분석가나 비프로그래머도 SQL을 이용해 spark의 강력한 빅데이터 처리 기능 사용
  - R
    - Spark에서는 일반적으로 사용하는 2개의 라이브러리 존재
      - SparkR : Spark Core에 포함
      - sparklyr 

- SparkSession과스파크 언어 API 간의 관계
~~~
                 JVM  
           ---------------
<------   |               | <------> 파이선 프로세스
<------   | SparkSession  | <------> R 프로세스
executor로 --------------- 
전달
~~~
- 각 언어 API는 앞서 말한 핵심 개념을 유지하고 있음
- 사용자는 Spark 코드를 실행하기 위해 SparkSession을 진입점으로 사용할 수 있음
- python이나 R로 스파크를 사용할 때는 JVM 코드를 명시적으로 작성하지 않음  
- Spark은 사용자를 대신해 Python이나 R로 작성한 코드를 executor의 JVM에서 실행할 수 있는 코드로 변환

### 2.3 Spark API
- 다양한 언어로 Spark를 사용할 수 있는 이유는 Spark가 기본적으로 2가지 API를 제공하기 때문
  - 저수준의 비구조적 API
  - 고수준의 구조적 API

### 2.4 Spark 시작하기
- 실제 Spark 애플리케이션을 개발하려면 사용자 명령과 데이터를 Spark Application에 전송하는 방법을 알아야 함
- 해당 예제를 실행하려면 Spark 로컬 모드로 실행해야 함
- 대화형 모드로 Spark를 시작하면 Spark 애플리케이션을 관리하는 SparkSession이 자동으로 생성됨
- Stand-alone 애플리케이션(spark-submit)으로 Spark를 시작하면 사용자 애플리케이션 코드에서 SparkSession 객체를 직접 생성해야 함

### 2.5 SparkSession
- Spark 애플리케이션은 SparkSession이라고 불리우는 드라이버 프로세스로 제어
- SparkSession 인스턴스는 사용자가 정의한 처리 명령을 클러스터에서 실행
- 하나의 SparkSession은 하나의 스파크 애플리케이션에 대응  
- Scala와 python 콘솔을 시작하면 spark 변수로 SparkSession을 사용 할 수 있음
~~~scala
spark
# 다음과 같은 결과 출력
res0: org.apache.spark.sql.SparkSession = org.apache.spark.sql.SparkSession@7ec13984
~~~
- 일정 범위의 숫자를 만드는 간단한 작업을 수행
~~~scala
val myRange = spark.range(1000).toDF("number")
~~~
- 위의 생성한 DataFrame은 한 개의 컬럼(column)과 1000개의 로우(row)로 구성되며  
  각 로우에는 0 ~ 999의 값이 생성
- 이 숫자들은 <b>분산 컬렉션</b>을 나타냄  
  클러스터 모드에서 코드 예제를 실행하면 숫자 범위의 각 부분이 서로 다른 익스큐터에 할당  
- 이것이 Spark의 DataFrame!

### 2.6 DataFrame
- DataFrame은 가장 대표적인 구조적 API
- DataFrame은 테이블의 데이터를 로우와 컬럼으로 단순하게 표현
- 컬럼과 컬럼의 타입을 정의한 목록을 <b>스키마(schema)</b> 라고 부름
- DataFrame을 스프레드시트와 비교할 수 있는데, 스프레드시트는 한 대의 컴퓨터에 있지만,  
  Spark DataFrame은 수천 대의 컴퓨터에 분산되어 있음  
- 여러 컴퓨터에 분산하는 이유는 데이터가 너무 크거나, 계산에 오랜 시간이 걸릴 수 있기 때문
- python과 R도 마찬가지로 DataFrame의 개념이 있지만, 단일 컴퓨터에 존재(일반적)
- 이런 상황에서는 DataFrame으로 수행할 수 있는 작업이 해당 머신이 가진 자원에 따라 제한될 수 밖에 없음
- Spark은 pandas의 DataFrame과 R의 DataFrame을 spark DataFrame으로 쉽게 변환 가능
- Spark은 Dataset, DataFrame, SQL 테이블, RDD라는 몇 가지 핵심 추상화 개념을 가지고 있음  
  이 개념 모두 분산 데이터 모음을 표현  

### 2.6.1 파티션
- Spark는 모든 익스큐터가 병렬로 작업을 수행할 수 있도록 파티션이라고 불리는 청크 단위로 데이터를 분할함
- 파티션은 클러스터의 물리적 머신에 존재하는 로우의 집합을 의미
- DataFrame의 파티션은 실행 중에 데이터가 컴퓨터 클러스터에서 물리적으로 분산되는 방식을 나타냄  
- 만약 파티션이 하나고, 수천개의 익스큐터가 있더라도 병렬성은 1
- 수백개의 파티션이 있고 익스큐터가 하나밖에 없다면 병렬성은 1
- DataFrame을 사용하면 파티션을 수동 또는 개별적으로 처리할 필요가 없음  
  <b>물리적 파티션에 데이터 변환용 함수</b>를 지정하면 스파크가 실제 처리 방법을 결정  

### 2.6.2 트렌스포메이션
- Spark의 핵심 데이터 구조는 불변성(immutable)을 가짐  
  즉 한번 생성하면 변경할 수 없음
- DataFrame을 변경하려면 원하는 변경 방법을 Spark에 알려줘야 하는데,  
  이때 사용하는 명령을 <b>트렌스포메이션</b> 이라고 함
- 다음 코드는 DataFrame에서 짝수를 찾는 예제
~~~scala
val myRange  = spark.range(1000).toDF("number")
val divisBy2 = myRange.where("number % 2 = 0")
~~~
- 위 코드를 실행해도 결과는 출력되지 않음  
  (응답값은 출력되지만 데이터를 처리한 결과는 아님)
- 추상적인 트렌스포메이션만 지정한 상태이기 때문에 액션(action)을 호출하지 않으면  
  Spark는 실제 트렌스포메이션을 수행하지 않음
- 트렌스포메이션은 Spark에서 비즈니스 로직을 표현하는 핵심 개념  
- 트렌스포메이션에는 <b>좁은 의존성(narrow dependency)</b> 과  <b>넓은 의존성(wide dependency)</b> 두 가지 유형이 있음

#### 좁은 의존성(narrow dependency)
- 좁은 의존성을 가진 트렌스포메이션은 각 입력 파티션이 하나의 출력 파티션에만 영향을 미침  
  (입력 파티션과 출력 파티션이 1:1 대응)
- 이전 코드에서 where 구문은 좁은 의존성을 가짐
- Spark에서 <b>파이프라이닝(pipelining)</b>을 자동으로 수행함  
  즉, DataFrame에서 여러 필터를 지정하는 경우 모든 작업이 메모리에서 일어남
~~~
      좁은 의존성 트렌스포메이션

 입력 파티션1 -------> 출력 파티션1

 입력 파티션2 -------> 출력 파티션2

 입력 파티션3 -------> 출력 파티션3
~~~


#### 넓은 의존성(wide dependency)
- 하나의 입력 파티션이 여러 출력 파티션에 영향을 미침
~~~
      넓은 의존성 프렌스포메이션

  입력 파티션1 ---------> 출력 파티션1
               \
                \_ _ _ _> 출력 파티션2 
                ...
~~~

### 2.7.1 지연 연산(lazy evaluation)
- 지연 연산(lazy evaluation)은 스파크가 연산 그래프를 처리하기 직전까지 기다리는 동작 방식
- Spark은 특정 연산 명령이 내려진 즉시 데이터를 수정하지 않고, 원시 데이터에 적용할  
  트랜스포메이션의 <b>실행 계획</b>을 생성
- Spark은 코드를 실행하는 마지막 순간까지 대기하다가 원형 DataFrame 트렌스포메이션을 간결한  
 물리적 실행 계획으로 컴파일함
- Spark은 이 과정을 거치며 전체 데이터 흐름을 최적화 하는 엄청난 강점을 가지고 있음

#### ex) 조건절 푸시다운(predicate pushdown)  
- 아주 복잡한 Spark job이 하나의 로우만 가져오는 필터를 가지고 있다면,  
  필요한 레코드 하나만 읽는 것이 가장 효율적  
- Spark은 이 필터를 데이터소스로 위임하는 최적화 작업을 자동으로 수행  
  (만약 데이터 저장소가 DB라면 WHERE절의 처리를 위임함으로써 처리에 필요 자원을 최소화 할 수 있음)  

### 2.8 액션(action)
- 사용자는 트렌스포메이션을 사용해 논리적 실행 계획을 세울 수 있는데, 실제 연산을 수행하려면  
 액션 명령을 내려야 함
- 가장 단순한 액션인 count 메소드는 DataFrame의 전체 레코드 수를 반환
~~~scala
divisBy2.count()
~~~
- count 외에도 3가지 액션이 존재
  - 콘솔에서 데이터를 보는 액션
  - 각 언어로 된 네이티브 객체에 데이터를 모으는 액션
  - 출력 데이터소스에 저장하는 액션
- 액션을 지정하면 Spark job이 시작 됨
- Spark job은 필터를 수행 한 후 파티션별로 레코드 수를 카운트 함
- 그리고 각 언어에 적합한 네이티브 객체에 결과를 모음  
- 이때 Spark가 제공하는 스파크 UI로 클러스터에서 실행 중인 Spark job을 모니터링 할 수 있음  

### 2.9 Spark UI
- Spark Job의 진행 상황을 모니터링할 때 사용
- 드라이버 노드의 4040 포트로 접속할 수 있음
- 로컬 실행시 스파크 UI의 주소는 http://localhost:4040 임
- Spark Job의 상태, 환경 설정, 클러스터 상태등의 정보를 확인할 수 있음
- Spark Job을 튜닝하고 디버깅할 때 매우 유용
- <b>Spark Job은 개별 액션에 의해 트리거되는 다수의 프렌스포메이션으로 이루어져 있으며 Spark UI로 Job을 모니터링 할 수 있음</b>만 기억하자

### 2.10 종합 예제
- 미국 교통통계국의 항공 운항 데이터 중 일부를 Spark으로 분석
- CSV file 사용
- tip : spark-shell에서 multi-line 입력 방법
  - :paste 사용!

#### 1. csv file 읽어오기
~~~scala
val flightData2015 = spark
.read
.option("inferSchema", "true")      #- 2
.option("header", "true")           #- 3
.csv("C:/Spark-The-Definitive-Guide-master/data/flight-data/csv/2015-summary.csv") #- 4
~~~
- Spark 다양한 데이터소스를 지원  
  데이터는  SparkSession의 DataFrameReader 클래스를 사용해서 읽음
- #2- 예제에서는 Spark DataFrame의 스키마 정보를 알아내는 <b>스키마 추론</b> 기능을 사용
- #3- 첫 로우를 header로 지정
- #4- data loading
- Spark는 스키마 정보를 얻기 위해 데이터를 조금 읽고, 해당 로우의 데이터 타입을 스키마 데이터 타입에 맞게 분석
- 운영 환경에서는 데이터를 읽는 시점에 스키마를 엄격하게 지정하는 옵션 사용 

#### 2. take action으로 head와 같은 결과 확인
~~~
flightData2015.take(3)
~~~
- 스칼라와 파이썬에서 사용하는 DataFrame은 불특정 다수의 로우와 컬럼을 가짐
- 로우의 수를 알 수 없는 이유는 데이터를 읽는 과정이 지연 연산 형태의 트랜스포메이션이기 때문

#### 3. sort, explain 메소드 함수 사용
- sort 메소드는 DataFrame을 변경하지 않고, 순서만 변환해 새로운 DataFrame으로 반환
- sort는 단지 트렌스포메이션이기 때문에 호출시 아무 변화도 일어나지 않는데  
  Spark는 실행 계획을 만들고 검토하여 클러스터에서 처리할 방법을 알아냄
- DataFrame 객체에 explain 메소드를 호출하면 DataFrame의 계보나 Spark의 쿼리 실행 계획을 확인 할 수 있음
~~~scala
flightData2015.sort("count").explain()

// 결과
== Physical Plan ==
*(2) Sort [count#12 ASC NULLS FIRST], true, 0
+- Exchange rangepartitioning(count#12 ASC NULLS FIRST, 200)
   +- *(1) FileScan csv [DEST_COUNTRY_NAME#10,ORIGIN_COUNTRY_NAME#11,count#12] Batched: false, Format: CSV, Location: InMemoryFileIndex[file:/C:/Spark-The-Definitive-Guide-master/data/flight-data/csv/2015-summary.csv], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<DEST_COUNTRY_NAME:string,ORIGIN_COUNTRY_NAME:string,count:int>
~~~
- 결과는 실행 계획을 확인할 수 있음
- 위에서 아래 방향으로 읽으면서 최종 결과는 가장 위에, 데이터소스는 가장 아래 있음

#### 4. 트랜스포메이션 실행 계획을 시작하기 위한 액션 호출
- 액션 실행시 몇가지 설정 필요
- Spark는 셔플 수행 시 기본적으로 200개의 셔플 파티션을 생성  
  이 값을 5로 설정의 셔플의 출력 파티션 수를 줄여보자

~~~scala
spark.conf.set("spark.sql.shuffle.partitions", "5")
fileData2015.sort("count").take(2)
~~~
- Spark는 계보를 통해 입력 데이터에 수행한 연산을 전체 파티션에서 어떻게 재연산하는지 알 수 있음
- 이 기능은 스파크의 프로그래밍 모델인 함수형 프로그래밍의 핵심
- 함수형 프로그래밍은 데이터 변환 규칙이 일정한 경우 같은 입력에 대해 항상 같은 출력을 생성
- 사용자는 물리적 데이터를 직접 다루진 않지만 <b>셔플 파티션 파라미터</b>와 같은  
  속성으로 물리적 실행 특성을 제어함
- 출력 파티션 수를 변경하면 런타임이 크게 달라질 수 있음

### 2.10.1 종합예제 : DataFrame과 SQL
- DataFrame과 SQL을 사용하는 복잡한 작업을 수행해보자
- Spark는 언어와 상관없이 같은 방식으로 트렌스포메이션 수행 가능
- 사용자가 SQL이나 python, R, Scala, Java의 DataFrame으로 비즈니스 로직을 표현하면  
  Spark 에서 실제 코드를 실행하기 전에 로직을 기본 실행 계획으로 컴파일함  
- Spark SQL을 사용하면 모든 DataFrame을 Table이나 View로 등록 후 SQL 쿼리로 사용 가능
- Spark는 SQL 쿼리를 DataFrame 코드와 같은 실행 계획으로 컴파일하므로  
  둘 사이의 성능 차이는 없음

#### 1. DataFrame --> Table로 변환
- `createOrReplaceTempView` 메서드 호출로 Table 생성
- 해당 명령어 수행 후 SQL로 데이터 조회 가능

~~~scala
flightData2015.createOrReplaceTempView("flight_data_2015")
~~~

#### 2. SQL 쿼리 실행
- spark.sql 메서드로 SQL 쿼리 실행
- spark은 SparkSession의 변수
- DataFrame에 쿼리를 수행하면 새로운 DataFrame을 반환
- 로직을 작성할 때 반복적인 느낌이 들지만, 매우 효율적
- 다음 두 가지 실행 방법은 동일한 실행 계획으로 컴파일 됨!!

~~~scala
// sql query 를 통한 방식
val sqlway = spark.sql("""
SELECT DEST_COUNTRY_NAME, count(1)
FROM flight_data_2015
GROUP_BY DEST_COUNTRY_NAME
""")

// Spark DataFrame을 통한 방식
val dataFrameWay = flightData2015
.groupBy("DEST_COUNTRY_NAME")
.count()

sqlway.explain()
dataFrameWay.explain()
~~~

#### 3. 특정 위치를 왕래하는 최대 비행 횟수를 구하기
- max 함수는 필터링을 수행해 단일 로우를 결과로 반환하는 트렌스포메이션임을 기억

~~~scala
// sql
spark.sql("SELECT max(count) from filght_data_2015").take(1)

// spark dataFrame
import org.apache.spark.sql.functions.max
flightData2015.select(max("count")).take(1)
~~~

#### 4. 상위 5개의 도착 국가를 찾아내기
~~~scala
// sql
val maxSql = spark.sql("""
SELECT DEST_COUNTRY_NAME, sum(count) as destination_total
FROM flight_data_2015
GROUP_BY DEST_COUNTRY_NAME
ORDER_BY sum(count) DESC
LIMIT 5
""")

// scala
import org.apache.spark.sql.functions.desc

flightData2015
.groupBy("DEST_COUNTRY_NAME")
.sum("count")
.withColumnRenamed("sum(count)", "destination_total")
.sort(desc("destination_total"))
.limit(5)
.show()
~~~
- DataFrame의 `explain()`을 통해 확인해보면 총 7가지 단계가 있음
- 실행 계획은 트랜스포메이션의 지향성 비순환 그래프(directed acyclic graph, DAG)이며,  
  액션이 호출되면 결과 생성
- DAG의 각 단계는 불변성을 가진 신규 DataFrame을 생성함



### 용어 정리
- ANSI SQL
  - DBMS(Oracle, My-SQL, DB2 등등)들에서 각기 다른 SQL를 사용하므로,  
  미국 표준 협회(American National Standards Institute)에서 이를 표준화하여  
  표준 SQL문을 정립 시켜 놓은 것  
- 네이티브 객체
  - 브라우저 혹은 구동 엔진에 내장되어 있는 객체를 말한다
- 데이터소스(DataSource)
  - DB와 Connection을 맺을 때, ConnectionPool에는 여러 Connection 객체가 존재  
    이때 각각 Application에서 직접적으로 이용하면 체계적인 관리가 어렵게 되므로, DataSource라는 개념을 도입하여 사용하고 있음
    DataSource라는 객체는 Connection Pool을 관리하는 목적으로 사용되는 객체  
    서버로부터 데이터베이스에 대해 연결을 구축하기 위해 사용되는 이름
- 프로세스(process)
  - 실행중에 있는 프로그램
  - 스케줄링의 대상이 되는 작업(task)과 같은 의미로 쓰임
  - 프로세스 내부에는 최소 하나의 스레드로 구성. 실제로는 스레드 단위로 스케줄링을 함
  - 하드디스크에 있는 프로그램을 실행하면 실행을 위해 메모리 할당이 이루어지고 할당된 메모리 공간으로  
    바이너리 코드가 올라가게 됨. 이 순간부터 프로세스라 불림
- 세션(Session)
  - 일정 시간동안 같은 사용자로부터 들어오는 일련의 요구를 하나의 상태로 보고  
   그 상태를 일정하게 유지시키는 기술
  - 여기서 말하는 일정 시간은 사용자가 웹 브라우저를 통해 웹 서버에 접속한 시점으로부터 웹 브라우저를 종료함으로써 연결을 끝내는 시점 
- 술어(predicate)
  - true / false를 판단할 수 있는 식이나 boolean 값을 리턴하는 함수를 술어(predicate)라고 함