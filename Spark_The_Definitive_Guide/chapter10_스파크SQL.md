# 스파크 SQL
- Spark SQL은 스파크에서 가장 중요하고 강력한 기능 중 하나
- ANSI-SQL 규격에 맞추어 예제를 다시 작성하거나, 모든 종류의 SQL 표현식을 열거하지는 않음
- Spark SQL을 이용해 DB에 생성된 View나 테이블에 SQL 쿼리를 실행할 수 있음
- 또한 시스템 함수를 사용하거나 UDF를 정의할 수 있음
- 그리고 워크로드를 최적화하기 위해 쿼리 실행 계획을 분석할 수도 있음
- Spark SQL은 DataFrame과 Dataset API로 통합되어 있음
- 따라서 데이터 변환 시 SQL과 DataFrame의 기능을 모두 사용할 수 있으며 두 방식 모두 동일한 실행 코드로 컴파일됨


## 10.1 SQL이란
- SQL 또는 구조적 질의 언어(Structured Query Language)는 데이터에 대한 관계형 연산을 표현하기 위한 도메인 특화 언어
- 모든 관계형 DB에서 사용되며, 많은 'NoSQL' DB에서도 쉽게 사용할 수 있는 변형된 자체 SQL을 제공함

## 10.2 빅데이터와 SQL: Apache Hive
- Spark가 등장하기 전에는 Hive가 빅데이터 SQL 접근 계층에서 사실상의 표준이였음
- 페이스북에서 최초로 개발한 Hive는 SQL 처리가 필요한 빅데이터 업계에서 믿을 수 없을 정도로 인기 있는 도구가 되었음
- 분석가들이 하이브로 SQL 쿼리를 실행할 수 있게 되면서 하둡을 다양한 산업군으로 진출시키는 데 다방면으로 도움을 줌
- Spark는 RDD를 이용하는 범용 처리 엔진으로 시작했지만 이제는 많은 사용자가 Spark SQL을 사용하고 있음

## 10.3 빅데이터와 SQL: 스파크 SQL
- Spark 2.0 버전에서는 하이브를 지원할 수 있는 상위 호환 기능으로 ANSI-SQL과 HiveQL을 모두 지원하는 자체 개발된 SQL 파서가 포함되어 있음
- Spark SQL은 DataFrame과의 뛰어난 호환성 덕분에 다양한 기업에서 강력한 기능으로 자리매김 할 것임
- Spark SQL의 강력함은 몇 가지 핵심 요인에서 비롯됨
- SQL 분석가들은 쓰리프트 서버(Thrift Server)나 SQL 인터페이스에 접속해 Spark의 연산 능력을 활용할 수 있음
- 데이터 엔지니어와 과학자는 전체 데이터 처리 파이프라인에 Spark SQL을 사용할 수 있음
- 이 통합형 API는 SQL로 데이터를 조회하고 DataFrame으로 변환한 다음 Spark의 MLlib이 제공하는 대규모 머신러닝 알고리즘 중 하나를 사용해 수행한 결과를 다른 데이터소스에 저장하는 전체 과정을 가능하게 만듬
- Spark SQL은 온라인 트랜잭션 처리(online transaction processing, OLTP) 데이터베이스가 아닌 온라인 분석용(online analytic processing, OLAP) DB로 동작함
- <b>즉 매우 낮은 지연 시간이 필요한 쿼리를 수행하기 위한 용도로는 사용할 수 없음</b>
- 언젠가는 '인플레이스 수정(in-place modification)'방식을 지원하겠지만 현재는 사용할 수 없음

### 10.3.1 Spark와 Hive의 관계
- Spark SQL은 Hive MetaStore를 사용하므로 하이브와 잘 연동할 수 있음
- Hive MetaStore는 여러 세션에서 사용할 테이블 정보를 보관하고 있음
- Spark SQL은 하이브 메타스토어에 접속 한 뒤 조회할 파일 수를 최소화하기 위해 메타데이터를 참조함
- 이 기능은 기존 하둡 환경의 모든 워크로드를 Spark로 이관하려는 사용자들에게 인기를 얻고 있음

#### Hive MetaStore
- 하이브 메타스토어에 접속하려면 몇 가지 속성이 필요함
- 먼저 접근하려는 하이브 메타스토어에 적합한 버전을 `spark.sql.hive.metastore.version`에 설정해야함
- 기본값은 1.2.1임. 또한 HiveMetastoreClient가 초기화하는 방식을 변경하려면 spark.sql.hive.metastore.jars를 설정해야함
- Spark는 기본 버전을 사용하지만 메이븐 저장소나 자바 가상 머신의 표준 포맷에 맞게 classPath에 정의할 수도 있음
- 하이브 메타스토어가 저장된 다른 DB에 접속하려면 적합한 클래스 접두사(MySQL를 사용하려면 com.mysql.jdbc로 명시)를 정의해야 함
- 또한 Spark와 Hive에서 공유할 수 있도록 클래스 접두사를 spark.sql.hive.metastore.sharedPrefixes 속성에 설정함

## 10.4 Spark SQL 쿼리 실행 방법
- Spark는 SQL 쿼리를 실행할 수 있는 몇 가지 인터페이스를 제공

###  10.4.1 Spark SQL CLI
- Spark SQL CLI(명령행 인터페이스)는 로컬 환경의 명령행에서 기본 스파크 SQL 쿼리를 실행할 수 있는 편리한 도구
- Spark SQL CLI는 쓰리프트 JDBC 서버와 통신할 수 없음
- Spark SQL CLI를 사용하려면 Spark 디렉터리에서 다음 명령을 실행
~~~
./bin/spark-sql
~~~
- Spark가 설치된 경로의 conf 디렉터리에 `hive-site.xml`, `core-site.xml`, `hdfs-site.xml` 파일을 배치해 하이브를 사용할 수 있는 환경을 구성할 수 있음
- 사용 가능한 전체 옵션을 보려면 `./bin/spark-sql --help` 확인

### 10.4.2 Spark 프로그래밍 SQL 인터페이스
- 서버를 설정해 SQL을 사용할 수도 있지만, 스파크에서 지원하는 언어 API로 비정형 SQL을 실행할 수도 있음
- 이를 위해 `SparkSession` 객체의 `sql` 메서드를 사용
- 처리된 결과는 DataFrame을 반환
- 예를 들어 파이썬이나 스칼라에서는 다음과 같은 코드를 실행할 수 있음
~~~scala
spark.sql("SELECT 1 + 1").show()
~~~
- `spark.sql("SELECT 1 + 1").show()` 명령은 프로그래밍 방식으로 평가할 수 있는 DataFrame을 반환
- 다른 트랜스포메이션과 마찬가지로 즉시 처리되지 않고 지연 처리됨
- 또한 DataFrame을 사용하는 것보다 SQL 코드로 표현하기 훨씬 쉬운 트랜스포메이션이기 때문에 엄청나게 강력한 인터페이스
- 함수에 여러 줄로 구성된 문자열을 전달할 수 있으므로 여러 줄로 구성된 쿼리를 아주 간단히 표현할 수 있음
- 예를 들어 다음과 같은 코드 실행이 가능
~~~scala
spark.sql("""SELECT user_id, department, first_name FROM professors WHERE department IN 
(SELECT name FROM department WHERE create_date >= '0216-01-01')""")
~~~
- 심지어 SQL과 DataFrame은 완벽하게 연동될 수 있으므로 더 강력함
- 예를 들어 DataFrame을 생성하고 SQL을 사용해 처리할 수 있으며 그 결과를 다시 DataFrame으로 돌려받게 됨
- 이러한 방식은 다양한 처리에 자주 사용하게 될 매우 효과적인 패턴 중 하나
~~~scala
// DataFrame -> SQL에서 사용할 수 있도록 처리
spark.read.format("json")
.load("/data/flight-data/json/2015-summary.json")
.createOrReplaceTempView("some_sql_view")

spark.sql("""
SELECT DEST_COUNTRY_NAME, sum(count)
FROM some_sql_view GROUP BY DEST_COUNTRY_NAME
""")
.where("DEST_COUNTRY_NAME like 'S%'").where("`sum(count)` > 10")
.count() // SQL의 결과를 DataFrame으로 반환
~~~

### 10.4.3 Spark SQL 쓰리프트 JDBC/ODBC 서버
- Spark는 JDBC 인터페이스를 제공
- 사용자나 원격 프로그램은 Spark SQL을 실행하기 위해 이 인터페이스로 Spark Driver에 접속함
- 비즈니스 분석가가 태플로같은 비즈니스 인텔리전스 소프트웨어를 이용해 Spark에 접속하는 형태가 가장 대표적인 활용 사례
- 쓰리프트 JDBC/ODBC 서버는 하이브 1.2.1 버전의 HiveServer2에 맞추어 구현되어 있음
- Spark나 Hive 1.2.1 버전에 있는 beeline 스크립트를 이용해 JDBC 서버를 테스트해볼 수 있음
- JDBC/ODBC 서버를 시작하려면 Spark 디렉터리에서 다음 명령 실행
~~~scala
./sbin/start-thriftserver.sh
~~~
- 이 스크립트는 bin/spark-submit 스크립트에서 사용할 수 있는 모든 명령행 옵션을 제공
- 쓰리프트 서버의 전체 설정 옵션을 확인하려면 `./sbin/start-thriftserver.sh --help`명령 실행
- 쓰리프트 서버는 기본적으로 localhost:10000 주소를 사용
- 환경변수나 시스템 속성을 지정해 쓰리프트 서버의 주소를 변경할 수 있음
- 환경변수는 다음과 같이 설정
~~~scala
export HIVE_SERVER2_THRIFT_PORT=<listening-port>
export HIVE_SERVER2_THRIFT_BIND_HOST=<listening-host>
./sbin/start-thriftserver.sh \
--master <master-uri> \
~~~
- 시스템 속성은 다음과 같이 설정
~~~scala
./sbin/start-thriftserver.sh \
--hiveconf hive.server2.thrift.port=<listening-port> \
--hiveconf hive.server2.thrift.bind.host=<listening-host> \
--master<master-uri>
...
~~~
- 서버가 시작되면 다음 명령을 사용해 접속 테스트를 함
~~~scala
./bin/beeline
beeline> !connect jdbc:hive2://localhost:10000
~~~
- beeline은 사용자 이름과 비밀번호를 요구함
- 비보안 모드의 경우에는 단순히 로컬 사용자 이름을 입력하며 비밀번호는 입력하지 않아도 됨
- 보안 모드의 경우에는 beeline 문서에서 제시하는 방법을 따라야 함

## 10.5 카탈로그
- Spark SQL에서 가장 높은 추상화 단계는 카탈로그임
- 카탈로그는 테이블에 저장된 데이터에 대한 메타데이터뿐만 아니라 데이터베이스, 함수, 그리고 뷰에 대한 정보를 추상화함
- 카탈로그는 `org.apache.spark.sql.catalog.Catalog` 패키지로 사용할 수 있음
- 카탈로그는 테이블, 데이터베이스 그리고 함수를 조회하는 등 여러 가지 유용한 함수를 제공함
- 이와 관련된 내용은 잠시 후에 알아보자
- 매우 명확한 내용이기 때문에 코드 예제는 생략했지만 Spark SQL을 사용하는 또 다른 방식의 프로그래밍 인터페이스
- 이장에서는 SQL 실행만 다루겠음. 따라서 프로그래밍 방식의 인터페이스를 사용하는 경우라면 `spark.sql` 함수를 사용해 관련 코드를 실행할 수 있다는 점을 기억하자

## 10.6 테이블
- Spark SQL을 사용해 유용한 작업을 수행하려면 먼저 테이블을 정의해야 함
- Table은 명령을 실행할 데이터의 구조라는 점에서 DataFrame과 논리적으로 동일함
- 테이블을 사용해 9장에서 알아본 조인, 필터링, 집계 등 여러 데이터 변환 작업을 수행할 수 있음
- DataFrame은 프로그래밍 언어로 정의하지만 테이블은 DB에서 정의함
- <b>Spark에서 테이블을 생성하면 default 데이터베이스에 등록됨</b>
- 스파크 2.x 버전에서 테이블은 항상 데이터를 가지고 있다는 점에서 반드시 기억해야함
- 임시 테이블의 개념이 없으며 데이터를 가지지 않는 뷰만 존재
- 테이블을 제거하면 모든 데이터가 삭제되므로 조심해야함

### 10.6.1 Spark 관리형 테이블
- <b>관리형 테이블</b>과 <b>외부 테이블</b>의 개념은 반드시 기억해두어야 함
- 테이블은 두 가지 중요한 정보를 저장
- 테이블의 데이터와 테이블에 대한 데이터, 즉 <b>메타데이터</b>임
- Spark는 데이터뿐만 아니라 파일에 대한 메타데이터를 관리할 수 있음
- 디스크에 저장된 파일을 이용해 테이블을 정의하면 외부 테이블을 정의하는 것
- DataFrame의 `saveAsTable` 메서드는 Spark과 관련된 모든 정보를 추적할 수 있는 관리형 테이블을 만들 수 있음 
- `saveAsTable` 메서드는 테이블을 읽고 데이터를 Spark format으로  변환한 후 새로운 경로에 저장
- 새로운 실행 계획에 이러한 동작이 반영되어 있음을 알 수 있으며, 하이브의 기본 웨어하우스 경로에 데이터를 저장하는 것을 확인할 수 있음
- 데이터 저장 경로를 변경하려면 SparkSession을 생성할 때 `spark.sql.warehouse.dir` 속성에 원하는 디렉터리 경로를 설정함
- 기본 저장 경로는 `/user/hive/warehouse` 임
- 저장 경로 하위에서 DB 목록을 확인할 수 있음. Spark는 DB 개념도 존재함
-`show tables IN databaseName` 쿼리를 사용해 특정 데이터베이스의 테이블을 확인할 수도 있음
- 쿼리에서 databaseName 부분은 테이블을 조회할 데이터베이스 이름을 나타냄
- 신규 클러스터나 로컬 모드에서 실행하면 빈 테이블 목록을 나타냄

### 10.6.2 테이블 생성하기
- 다양한 데이터소스를 사용해 테이블을 생성할 수 있음
- Spark는 SQL에서 전체 데이터소스 API를 재사용할 수 있는 독특한 기능을 지원
- 즉, 테이블을 정의한 다음 테이블에 데이터를 적재할 필요가 없음
- Spark는 실행 즉시 테이블을 생성
- 파일에서 데이터를 읽을 때 모든 종류의 정교한 옵션을 지정할 수도 있음
- 예를 들어 9장에서 사용한 항공운항 데이터를 읽는 방법은 다음과 같음
~~~scala
CREATE TABLE flights (
    DEST_COUNTRY_NAME STRING, ORIGIN_COUNTRY_NAME STRING, count LONG)
USING JSON OPTIONS(path '/data/flight-data/json/2015-summary.json')
~~~

#### USING과 STORED AS 구문
- 위 예제에서 USING 구문은 매우 중요. 포맷을 지정하지 않으면 Spark는 기본적으로 하이브 SerDe 설정을 사용
- 하이브 SerDe는 Spark의 자체 직렬화보다 훨씬 느리므로 테이블을 사용하는 Reader와 Writer 성능에 영향을 미침
- 하이브 사용자는 STORED AS 구문으로 하이브 테이블을 생성할 수 있음
- 테이블의 특정 컬럼에 코멘트를 추가해 다른 개발자의 이해를 도울 수 있음
~~~scala
CREATE TABLE flights_csv (
    DEST_COUNTRY_NAME STRING,
    ORIGIN_COUNTRY_NAME STRING COMMENT "remember, the US will be most prevalent",
    count LONG)
USING csv OPTIONS (header true, path '/data/flight-data/csv/2015-summary.csv')
~~~
- 또한 SELECT 쿼리의 결과를 이용해 테이블을 생성할 수도 있음
~~~scala
// CTAS 패턴
CREATE TABLE flights_from_select USING parquet AS SELECT * FROM flights
~~~
- 또한 테이블이 없는 경우에만 생성하도록 지정할 수도 있음
- 앞 예제에서는 USING 구문을 명시적으로 지정하지 않았으므로 하이브 호환 테이블을 만들게 됨
- 또한 다음과 같은 쿼리를 사용할 수도 있음
~~~scala 
CREATE TABLE IF NOT EXISTS flights_from_select
AS SELECT * FROM flights
~~~
- 마지막으로 9장에서 알아본 것처럼 파티셔닝된 데이터셋을 저장해 데이터 레이아웃을 제어할 수 있음
~~~scala
CREATE TABLE partitioned_flights USING parquet PARTITIONED BY (DEST_COUNTRY_NAME) AS SELECT DEST_COUNTRY_NAME, ORIGIN_COUNTRY
~~~
- Spark에 접속한 세션에서도 생성된 테이블을 사용할 수 있음
- 임시 테이블 개념은 현재 스파크에 존재하지 않음
- 사용자는 임시 뷰를 만들어 이 기능을 사용할 수 있음

### 10.6.3 외부 테이블 생성하기
- 이 장 서두에서 언급했듯이 하이브는 초기 빅데이터 SQL 시스템 중 하나였음
- Spark SQL은 완벽하게 하이브 SQL과 호환됨
- 기존 하이브 쿼리문을 Spark SQL로 변환해야 하는 상황을 만날 수도 있음
- 다행히도 대부분의 하이브 쿼리문은 Spark SQL에서 바로 사용 가능
- 다음은 <b>외부 테이블</b>을 생성하는 예제
- Spark는 외부 테이블의 메타데이터를 관리함. 하지만 데이터 파일은 Spark에서 관리하지 않음. CREATE EXTERNAL TABLE 구문을 사용해 외부 테이블을 생성할 수 있음
~~~scala
CREATE EXTERNAL TABLE hive_flights (
    DEST_COUNTRY_NAME STRING, ORIGIN_COUNTRY_NAME STRING, count LONG)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LOCATION 'data/flight-data-hive'
~~~
- 또한 SELECT 쿼리의 결과를 이용해 외부 테이블을 생성할 수도 있음
~~~scala
CREATE EXTERNAL TABLE hive_flights_2
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
LOCATION 'data/flight-data-hive' AS SELECT * FROM flights
~~~

### 10.6.4 테이블에 데이터 삽입하기
- 데이터 삽입은 표준 SQL 문법을 따름 
~~~scala
INSERT INTO flights_from_select
SELECT DEST_COUNTRY_NAME, ORIGIN_COUNTRY_NAME, count FROM  flights LIMIT 20
~~~
- 특정 파티션에만 저장하고 싶은 경우 파티션 명세를 추가할 수도 있음
- 쓰기 연산은 파티셔닝 스키마에 맞게 데이터를 저장함(위 예제의 쿼리가 매우 느리게 동작할 수 있음)
- 하지만 마지막 파티션에만 데이터 파일이 추가됨
~~~scala
INSERT INTO partitioned_flights
PARTITION (DEST_COUNTRY_NAME= " UNITED STATES" ) 
SELECT count, ORIGIN_COUNTRY_NAME FROM flights
WHERE DEST_COUNTRY_NAME= 'UNITED STATES' LIMIT 12
~~~

### 10.6.5 테이블 메타데이터 확인하기
- 테이블 생성 시 코멘트를 추가할 수 있음. 추가된 코멘트를 확인하려면 DESCRIBE 구문을 사용
- DESCRIBE 구문은 테이블의 메타데이터 정보를 반환
~~~scala
DESCRIBE TABLE flights_csv
~~~
- 다음 명령어를 사용해 파티셔닝 스키마 정보도 확인할 수 있음
- 이 명령은 파티션된 테이블에서만 동작
~~~scala
SHOW PARTITIONS partitioned_flights
~~~



## 용어 정리
- Thrift Server
  - 페이스북이 개발한 가변적인 이종 언어 서비스 개발을 위한 소프트웨어 프레임워크
  - Thrift를 사용하는 이유는 다양한 언어를 사용하여 개발한 소프트웨어를 쉽게 결합하기 위함
  - Apache에서 제공하는 Thrift는 다양한 플랫폼간의 매우 편리하게 사용할 수 있는 통합 RPC 환경을 제공