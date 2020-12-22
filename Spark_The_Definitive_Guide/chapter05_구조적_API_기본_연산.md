## chapter05 구조적 API 기본 연산 
- 실질적인 DataFrame과 DataFrame의 데이터를 다루는 기능 소개
- DataFrame은 Row 타입의 <b>레코드</b>와 각 레코드에 수행할 연산 표현식을 나타내는 여러 <b>컬럼</b>으로 구성
- <b>스키마</b>는 각 컬럼명와 데이터 타입을 정의
- DataFrame의 <b>파티셔닝</b>은 DataFrame이나 Dataset이 클러스터에서 물리적으로 배치되는 형태 정의
- <b>파티셔닝 스키마</b>는 파티션을 배치하는 방법을 정의
- 파티셔닝의 분할 기준은 특정 컬럼이나 비결정론적(매번 변하는 정도로 이해하면 됨) 값을 기반으로 설정 

#### DataFrame 생성
~~~
val df = spark.read.format("json")
    .load("C:/Spark-The-Definitive-Guide-master/data/flight-data/json/2015-summary.json")
~~~

#### DataFrame의 스키마 살펴보기
~~~
df.printSchema()
~~~

### 5.1 스키마
- 스키마는 DataFrame의 컬럼명과 컬럼 타입을 정의함
- 데이터소스에서 스카마를 얻거나 직접 정의할 수 있음
- 데이터를 읽기 전에 스키마를 정의해야 하는지의 여부는 상황에 따라 달라짐
- 비정형 분석에서는 스키마-온-리드가 대부분 잘 작동(CSV JSON 같은 일반 텍스트 파일을 사용하면 다소 느릴 수 있음)
- 하지만 Long Type을 Integer Type으로 잘못 인식 하는 등 정밀도의 문제가 발생할 수 있음
- 운영 환경에서 ETL 작업을 수행한다면, 직접 스키마를 정의해야 함  
  ETL 작업 중에 데이터 타입을 알기 힘든 CSV나 JSON 등의 데이터 소스를 사용하는 경우,  
  스키마 추론 과정에서 읽어 들인 샘플 데이터의 타입에 따라 스키마를 결정할 수 있음
- 이번 예제에서는 4장에서 사용한 데이터 파일 사용  
  반정형 JSON Data
~~~
spark.read.format("json").load("C:/Spark-The-Definitive-Guide-master/data/flight-data/json/2015-summary.json").schema
//- 실행 결과
org.apache.spark.sql.types.StructType = StructType(StructField(DEST_COUNTRY_NAME,StringType,true), StructField(ORIGIN_COUNTRY_NAME,StringType,true), StructField(count,LongType,true))
~~~
- 여러 개의 StructField 타입 필드로 구성된 StructType 객체
- StructField은 (이름, 데이터 타입, 컬럼값 유무(hull)) 로 구성
- 필요한 경우 컬럼과 관련된 <b>메타데이터</b>를 지정 할 수 있는데, 메타데이터는 해당 컬럼과 관련된 정보이며, Spark의 MLlib에서 사용
- 스키마는 복합 데이터 타입인 StructType을 가질 수 있음
- Spark는 런타임에 데이터 타입이 스키마의 데이터 타입과 일치하지 않으면 오류 발생

#### DataFrame에 직접 스키마를 만들고 적용하는 예제
~~~
import org.apache.spark.sql.types.{StructField, StructType, StringType, LongType}
import org.apache.spark.sql.types.Metadata

val myManualSchema = StructType(Array(
    StructField("DEST_COUNTRY_NAME", StringType, true),
    StructField("ORIGIN_COUNTRY_NAME", StringType, true),
    StructField("count",LongType, false,
      Metadata.fromJson("{\"hello\":\"world\"}"))
))

val df = spark.read.format("json").schema(myManualSchema)
    .load("C:/Spark-The-Definitive-Guide-master/data/flight-data/json/2015-summary.json")
~~~
- Spark는 자체 데이터 타입 정보를 사용하므로, 프로그래밍 언어의 데이터 타입을 Spark의 데이터 타입으로 설정할 수 없음

### 5.2 컬럼과 표현식
- 사용자는 표현식으로 DataFrame의 컬럼을 선택, 조작, 제거할 수 있음
- Spark의 컬럼은 표현식을 사용해 레코드 단위로 계산한 값을 단순하게 나타내는 논리적인 구조
- 컬럼의 실제값을 얻으려면 로우가 필요하고, 로우를 얻으려면 DataFrame이 필요함
- DataFrame을 통하지 않으면 외부에서 컬럼에 접근할 수 없음
- 컬럼 내용을 수정하려면 반드시 DataFrame의 Spark 트랜스포메이션을 이용해야함

#### 5.2.1 컬럼
- 컬럼을 생성하고 참조할 수 있는 여러 방법이 있지만, col 함수와 column 함수를 사용하는 것이 
  가장 간단함
- 이들 함수는 컬럼명을 인수로 받음
~~~
import org.apache.spark.sql.functions.{col, column}

col("someColumnsName")
column("someColumnsName")
~~~
- Scala의 고유 기능을 사용해 더 간단한 방법으로 컬럼을 참조할 수 있음
~~~
$"myColumn" // $ 마크 이용
'myColumn   //- 틱마크 ' 이용
~~~
- 컬럼이 DataFrame에 있을지 없을지는 알 수 없음
- 컬럼은 컬럼명을 <b>카탈로그</b>에 저장된 정보와 비교하기 전까지 <b>미확인</b> 상태로 남음
- <b>분석기</b>가 동작하는 단계에서 컬럼과 테이블 분석

#### 명시적 컬럼 참조
- DataFrame의 컬럼은 col 함수를 통해 참조
- col 메서드는 조인 시 유용
- col 메서드를 사용해 명시적으로 컬럼을 정의하면 Spark는 분석기 실행 단계에서 컬럼 확인 절차 생략
~~~
df.col("count")
~~~

#### 5.2.2 표현식
- 앞서 DataFrame을 정의할 때 컬럼은 표현식이라고 했음
- 여기서 말하는 <b>표현식</b>이란, DataFrame 레코드의 여러 값에 대한 트랜스포메이션 집합을 의미
- 여러 컬럼명을 입력받아 식별하고, '단일 값'을 만들기 위해 다양한 표현식을 각 레코드에 적용하는 함수라고 생각할 수 있음
- 여기서 단일 값은 Map이나 Array처럼 복합 데이터 타입일 수 있음
- 표현식은 <b>expr</b> 함수로 가장 간단히 사용될 수 있음. 이 함수를 이용해 DataFrame의 컬럼을 참조 할 수 있음
- 예를 들어 expr("someCol")은 col("someCol") 구문과 동일하게 동작

#### 표현식으로 컬럼 표현
- 컬럼은 표현식의 일부 기능을 제공  
  --> 컬럼 단위 별 사칙연산이 가능하다는 의미
- 이 때 col 함수를 호출해 컬럼에 트랜스포메이션을 수행하려면, 반드시 컬럼 참조를 사용 해야 함(col 함수로 사칙연산 수행 하려면, 컬럼 참조( ==> col('colName')) 를 사용해야 함)
- 다음 예제를 잘 기억하자
~~~
expr("someCol - 5")
col("someCol") - 5
expr("someCol") - 5
~~~
- 모두 같은 트랜스포메이션 과정을 거침
- 그 이유는 <b>Spark가 연산 순서를 지정하는 논리적 트리로 컴파일 하기 때문</b>
- 다음의 두가지는 꼭 기억하자
  - <b>컬럼은 단지 표현식일 뿐이다</b>
  - <b>컬럼과 컬럼의 트랜스포메이션은 파싱된 표현식과 동일한 논리적 실행 계획으로 컴파일됨</b>
- 예제와 함께 자세히 알아보자(** 중요)
~~~
((col("someCol" + 5) * 200) - 6) < col("otherCol")
~~~
- 다음은 위의 예제의 논리적 트리 개요
~~~
                        <
                       / \ 
                     -    otherCol
                   /   \
                  *     6
                /  \     
               +   200
             /   \
        someCol   5
~~~
- 위의 트리구조는 지향성 비순환 그래프(DAG)임
- 위의 트리 구조는 다음 코드로 동일하게 표현 가능
~~~
import org.apache.spark.sql.functions.expr
expr("(((someCol + 5) * 200) - 6) < otherCol")
~~~
- SQL의 SELECT 구문에 이전 표현식을 사용해도 잘 동작하고 동일한 결과를 생성함
- <b>그 이유는 SQL 표현식과 위 예제의 DataFrame</b> 코드는 실행 시점에 동일한 논리 트리로 컴파일 되기 때문
- 따라서 DataFrame 코드나 SQL로 표현식 작성이 가능하며, 동일한 성능을 발휘

#### DataFrame 컬럼에 접근하기
- <b>printSchema</b> 메서드로 DF의 전체 컬럼 정보 확인 가능
- 프로그래밍 방식으로 접근할 때는 DataFrame의 columns 속성 사용
~~~
spark.read.format("json").load("C:/Spark-The-Definitive-Guide-master/data/flight-data/json/2015-summary.json").columns
~~~

### 레코드와 로우
- Spark에서 DataFrame의 각 로우는 하나의 레코드임
- Spark는 레코드를 <b>Row 객체</b>로 표현함
- Spark는 값을 생성하기 위해 컬럼 표현식으로 Row 객체를 다룸
- Row 객체는 내부적으로 바이트 배열을 가짐
- 이 바이트 배열 인터페이스는 오직 컬럼 표현식으로만 다룰 수 있으므로 사용자에게 절대 노출되지 않음
- DataFrame을 사용해 드라이버에게 개별 로우를 반환하는 명령은 항상 하나 이상의 Row 타입을 반환

#### DataFrame의 first 메서드로 로우 확인
~~~
df.first()

// 결과
res0: org.apache.spark.sql.Row = [United States,Romania,15]
~~~

#### 5.3.1 로우 생성하기
- 각 컬럼에 해당하는 값을 사용해 Row 객체를 직접 생성할 수 있음
- Row 객체는 스키마 정보를 가지고 있지 않음
- DataFrame만 유일하게 스키마를 가짐
- 따라서 Row 객체를 직접 생성하려면 DataFrame의 스키마 순서와 동일하게 해야 함
~~~
import org.apache.spark.sql.Row
val myRow = Row("Hello", null, 1, false)
~~~
- 로우 데이터에 접근하는 방법은 원하는 위치를 지정하면 됨  
  ex) myRow(0)
- Scala나 Java에서는 헬퍼 메서드를 사용하거나, 명시적으로 데이터 타입을 지정해야 함
- python이나 R은 자동으로 데이터 타입이 변환됨
~~~
// Scala
myRow(0) // Any 타입. 결과 : res1: Any = Hello
myRow(0).asInstanceOf[String] // String 타입 결과 : res3: String = Hello
myRow.getString(0) // String 타입
myRow.getInt(2)    // Int 타입
~~~
- Dataset을 이용하면 JVM 객체를 가진 Dataset을 얻을 수 있음

### 5.4 DataFrame의 트랜스포메이션
- DataFrame을 다루는 방법 주요 몇가지를 알아보자
  - 로우, 컬럼 추가
  - 로우, 컬럼 제거
  - 로쿠, 컬럼 변환 및 그 반대로 변환
  - 컬럼값을 기준으로 로우 순서 변경 

#### 5.4.1 DataFrame 생성하기
- 원시 데이터소스에서 DataFrame을 생성할 수 있음
- DataFrame을 생성하고 후반부에 SQL 쿼리를 실행하고 SQL의 기본 트랜스메이션을 확인하기 위해 임시 뷰로 등록하자
~~~
val df = spark.read.format("json")
.load("C:/Spark-The-Definitive-Guide-master/data/flight-data/json/2015-summary.json")
df.createOrReplaceTempView("dfTable")
~~~
- Row 객체를 가진 Seq 타입을 직접 변환해 DataFrame으로 생성 가능
~~~
import org.apache.spark.sql.Row
import org.apache.spark.sql.types.{StructField, StructType, StringType, LongType}

//- 1 방법 1
val myManualSchema = new StructType(Array(
    new StructField("some", StringType, true),
    new StructField("col", StringType, true),
    new StructField("names", LongType, false)))

val myRows = Seq(Row("Hello", null, 1L))
val myRDD = spark.sparkContext.parallelize(myRows)
val myDf  = spark.createDataFrame(myRDD, myManualSchema)

myDf.show()

// 결과
+-----+----+-----+
| some| col|names|
+-----+----+-----+
|Hello|null|    1|
+-----+----+-----+


// -2 방법 2
val myDF = Seq(("HELLO", 2, 1L)).toDF("col1", "col2", "col3")
myDF.show()

// 결과
+-----+----+----+
| col1|col2|col3|
+-----+----+----+
|HELLO|   2|   1|
+-----+----+----+
~~~
- 다음은 가장 유용하게 사용할 수 있는 메소드를 알아보자
  - 컬럼이나 표현식을 사용하는 select 메소드
  - 문자열 표현식을 사용하는 selectExpr 메소드
  - 메서드로 사용할 수 없는 org.apache.spark.sql.functions 패키지에 포함된 다양한 함수

- 이 세가지 유형의 메서드로 DataFrame을 다룰 때 필요한 대부분의 트랜스포메이션 작업을 해결할 수 있음

#### 5.4.2 select 와 selectExpr
- 해당 메서드를 사용하면 데이터 테이블에 SQL을 실행하는 것처럼 DataFrame에서도
  SQL 사용 가능
~~~
// SQL
SELECT * FROM dataFrameTable
SELECT columnName FROM dataFrameTable
SELECT columnName * 10, otherColumn, someOtherCol as c FROM dataFrameTable
~~~
- 즉, DataFrame의 컬럼을 다룰 때 SQL 사용 가능
- select 메서드를 사용하는 것이 가장 쉬움
~~~
// scala
df.select("DEST_COUNTRY_NAME").show(2)

// SQL
SELECT "DEST_COUNTRY_NAME
FROM dfTable 
limit 2

// 실행 결과
+-----------------+
|DEST_COUNTRY_NAME|
+-----------------+
|    United States|
|    United States|
+-----------------+
~~~

- select 메소드를 이용한 여러 컬럼 선택 예제
~~~
df.select("DEST_COUNTRY_NAME", "ORIGIN_COUNTRY_NAME").show(2)
~~~
- 추가적으로 다양한 컬럼 참조 방법을 이용해 사용 가능함을 기억하자
~~~
df.select(
    df.col("DEST_COUNTRY_NAME"),
    col("DEST_COUNTRY_NAME"),
    column("DEST_COUNTRY_NAME"),
    'DEST_COUNTRY_NAME,
    $"DEST_COUNTRY_NAME",
    expr("DEST_COUNTRY_NAME"))
.show(2)
~~~


### 용어 정리
- 함수와 메소드의 차이
  - 함수는 독립적이고, 메소드는 class에 종속적임

