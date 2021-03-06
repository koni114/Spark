## chapter05 구조적 API 기본 연산 
- 실질적인 DataFrame과 DataFrame의 데이터를 다루는 기능 소개
- DataFrame은 Row 타입의 <b>레코드</b>와 각 레코드에 수행할 연산 표현식을 나타내는 여러 <b>컬럼</b>으로 구성
- <b>스키마</b>는 각 컬럼명와 데이터 타입을 정의
- DataFrame의 <b>파티셔닝</b>은 DataFrame이나 Dataset이 클러스터에서 물리적으로 배치되는 형태 정의
- <b>파티셔닝 스키마</b>는 파티션을 배치하는 방법을 정의
- 파티셔닝의 분할 기준은 특정 컬럼이나 비결정론적(매번 변하는 정도로 이해하면 됨) 값을 기반으로 설정 

#### DataFrame 생성
~~~scala
val df = spark.read.format("json")
    .load("C:/Spark-The-Definitive-Guide-master/data/flight-data/json/2015-summary.json")
~~~
  
#### DataFrame의 스키마 살펴보기
~~~scala
df.printSchema()

root
 |-- DEST_COUNTRY_NAME: string(nullable = true)
 |-- ORIGIN_COUNTRY_NAME: string(nullable = true)
 |-- count: long(nullable = true)
~~~

### 5.1 스키마
- 스키마는 DataFrame의 컬럼명과 컬럼 타입을 정의함
- 데이터소스에서 스키마를 얻거나 직접 정의할 수 있음
- 데이터를 읽기 전에 스키마를 정의해야 하는지의 여부는 상황에 따라 달라짐
- 비정형 분석에서는 스키마-온-리드가 대부분 잘 작동(CSV JSON 같은 일반 텍스트 파일을 사용하면 다소 느릴 수 있음)
- 하지만 Long Type을 Integer Type으로 잘못 인식 하는 등 정밀도의 문제가 발생할 수 있음
- 운영 환경에서 ETL 작업을 수행한다면, 직접 스키마를 정의해야 함  
  ETL 작업 중에 데이터 타입을 알기 힘든 CSV나 JSON 등의 데이터 소스를 사용하는 경우,  
  스키마 추론 과정에서 읽어 들인 샘플 데이터의 타입에 따라 스키마를 결정할 수 있음
- 이번 예제에서는 4장에서 사용한 데이터 파일 사용  
  반정형 JSON Data
~~~scala
spark.read.format("json").load("C:/Spark-The-Definitive-Guide-master/data/flight-data/json/2015-summary.json").schema

//- 실행 결과
org.apache.spark.sql.types.StructType = StructType(StructField(DEST_COUNTRY_NAME,StringType,true),    
           StructField(ORIGIN_COUNTRY_NAME,StringType,true), 
           StructField(count,LongType,true))
~~~
- 여러 개의 StructField 타입 필드로 구성된 StructType 객체
- StructField은 (이름, 데이터 타입, 컬럼값 유무(null)) 로 구성
- 필요한 경우 컬럼과 관련된 <b>메타데이터</b>를 지정 할 수 있는데, 메타데이터는 해당 컬럼과 관련된 정보이며, Spark의 MLlib에서 사용
- 스키마는 복합 데이터 타입인 StructType을 가질 수 있음
- Spark는 런타임에 데이터 타입이 스키마의 데이터 타입과 일치하지 않으면 오류 발생

#### DataFrame에 직접 스키마를 만들고 적용하는 예제
~~~scala
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
~~~scala
import org.apache.spark.sql.functions.{col, column}

col("someColumnsName")
column("someColumnsName")
~~~
- Scala의 고유 기능을 사용해 더 간단한 방법으로 컬럼을 참조할 수 있음
~~~scala
$"myColumn" // $ 마크 이용
'myColumn   // 틱마크 ' 이용
~~~
- 컬럼이 DataFrame에 있을지 없을지는 알 수 없음
- 컬럼명을 <b>카탈로그</b>에 저장된 정보와 비교하기 전까지 <b>미확인</b> 상태로 남음
- <b>분석기</b>가 동작하는 단계에서 컬럼과 테이블 분석

#### 명시적 컬럼 참조
- DataFrame의 컬럼은 col 함수를 통해 참조
- col 메서드는 조인 시 유용
- <b>col 메서드를 사용해 명시적으로 컬럼을 정의하면 Spark는 분석기 실행 단계에서 컬럼 확인 절차 생략</b>
~~~scala
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
~~~scala
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
~~~scala
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
~~~scala
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
- Row 객체는 내부적으로 바이트 배열(byte[])을 가짐
- 이 바이트 배열 인터페이스는 오직 컬럼 표현식으로만 다룰 수 있으므로 사용자에게 절대 노출되지 않음
- DataFrame을 사용해 드라이버에게 개별 로우를 반환하는 명령은 항상 하나 이상의 Row 타입을 반환

#### DataFrame의 first 메서드로 로우 확인
~~~scala
df.first()

// 결과
res0: org.apache.spark.sql.Row = [United States,Romania,15]
~~~

#### 5.3.1 로우 생성하기
- 각 컬럼에 해당하는 값을 사용해 Row 객체를 직접 생성할 수 있음
- Row 객체는 스키마 정보를 가지고 있지 않음
- DataFrame만 유일하게 스키마를 가짐
- 따라서 Row 객체를 직접 생성하려면 DataFrame의 스키마 순서와 동일하게 해야 함
~~~scala
import org.apache.spark.sql.Row
val myRow = Row("Hello", null, 1, false)
~~~
- 로우 데이터에 접근하는 방법은 원하는 위치를 지정하면 됨  
  ex) myRow(0)
- Scala나 Java에서는 헬퍼 메서드를 사용하거나, 명시적으로 데이터 타입을 지정해야 함
- python이나 R은 자동으로 데이터 타입이 변환됨
~~~scala
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
  - 로우, 컬럼 변환 및 그 반대로 변환
  - 컬럼값을 기준으로 로우 순서 변경 

#### 5.4.1 DataFrame 생성하기
- 원시 데이터소스에서 DataFrame을 생성할 수 있음
- DataFrame을 생성하고 후반부에 SQL 쿼리를 실행하고 SQL의 기본 트랜스메이션을 확인하기 위해 임시 뷰로 등록하자
~~~scala
val df = spark.read.format("json")
.load("C:/Spark-The-Definitive-Guide-master/data/flight-data/json/2015-summary.json")
df.createOrReplaceTempView("dfTable")
~~~
- Row 객체를 가진 Seq 타입을 직접 변환해 DataFrame으로 생성 가능
~~~scala
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
~~~SQL
// SQL
SELECT * FROM dataFrameTable
SELECT columnName FROM dataFrameTable
SELECT columnName * 10, otherColumn, someOtherCol as c FROM dataFrameTable
~~~
- 즉, DataFrame의 컬럼을 다룰 때 SQL 사용 가능
- select 메서드를 사용하는 것이 가장 쉬움
~~~scala
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
~~~scala
df.select("DEST_COUNTRY_NAME", "ORIGIN_COUNTRY_NAME").show(2)
~~~
- 추가적으로 다양한 컬럼 참조 방법을 이용해 사용 가능함을 기억하자
~~~scala
df.select(
    df.col("DEST_COUNTRY_NAME"),
    col("DEST_COUNTRY_NAME"),
    column("DEST_COUNTRY_NAME"),
    'DEST_COUNTRY_NAME,
    $"DEST_COUNTRY_NAME",
    expr("DEST_COUNTRY_NAME"))
.show(2)
~~~

- Column 객체에 문자열을 섞어 쓰는 실수를 많이 하는데, 조심하자  
  다음 예제는 컴파일 오류를 발생시킴

~~~scala
df.select(col("DEST_COUNTRY_NAME"), "DEST_COUNTRY_NAME")
~~~

- expr 함수는 가장 유연한 참조 방법
- expr 함수는 단순 컬럼 참조나 문자열을 이용해 컬럼 참조 가능
- AS 키워드로 컬럼명 변경 후, alias 메서드로 원래 컬럼명으로 되돌려보자
- 보통 select 함수에 expr 메소드를 자주 사용함
~~~scala
df select(expr("DEST_COUNTRY_NAME AS destination")).alias("DEST_COUNTRY_NAME").show(2)
~~~

- selectExpr 메서드를 사용하여 간단하고 효율적으로 할 수 있음
~~~scala
df.selectExpr("DEST_COUNTRY_NAME as newColumnName", "DEST_COUNTRY_NAME").show(2)
~~~

- DataFrame에 출발지와 도착지가 같은지 나타내는 withinCountry 컬럼 추가 예제
~~~scala
df.selectExpr(
  "*", // 모든 원본 컬럼 포함
  "(DEST_COUNTRY_NAME = ORIGIN_COUNTRY_NAME) as withinCountry" 
).show(2)
~~~
- select 표현식에는 DataFrame의 컬럼에 대한 집계 함수를 지정할 수 있음
~~~scala
df.selectExpr("avg(count)", "count(distinct(DEST_COUNTRY_NAME))").show(2)
~~~

#### 5.4.3 스파크 데이터 타입으로 변환하기
- 때로는 새로운 컬럼이 아닌 명시적인 값을 spark에 전달해야함  
  (컬럼에 특정 값을 지정하고 싶을 때)
- 명시적인 값은 상숫값일 수 있고, 추후 비교에 사용할 무언가가 될 수도 있음
- 이때 <b>리터럴</b>을 사용하는데, 프로그래밍 언어의 리터럴 값을  
  스파크가 이해할 수 있는 값으로 변환
- `lit` 메소드 사용
~~~scala
import org.apache.spark.sql.functions.lit
df.select(expr("*"), lit(1).as("One")).show(2)

//- 결과
+-----------------+-------------------+-----+---+
|DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|One|
+-----------------+-------------------+-----+---+
|    United States|            Romania|   15|  1|
|    United States|            Croatia|    1|  1|
+-----------------+-------------------+-----+---+
~~~
- 어떤 상수나 프로그래밍으로 생성된 변숫값이 특정 컬럼의 값보다 큰지 확인할 때 리터럴 사용

#### 5.4.4 컬럼 추가하기
- DataFrame에 신규 컬럼을 추가하는 공식적인 방법은 DataFrame의 withColumn 사용
- 숫자 1을 값으로 가지는 컬럼 추가 예제
~~~scala
df.withColumn("numberOne", lit(1)).show(2)
~~~
- 다음은 출발지와 도착지가 같은지 여부를 boolean 타입으로 표현하는 예제
~~~scala
df.withColumn("withinCountry", expr("ORIGIN_COUNTRY_NAME == DEST_COUNTRY_NAME")).show(2)
~~~
- withColumn 메소드는 두 개의 인수를 사용  
  하나는 컬럼명, 다른 하나는 값을 생성할 표현식
- withColumn 메서드로 컬럼명을 변경할 수도 있음
~~~scala
df.withColumn("Destination", expr("DEST_COUNTRY_NAME")).columns
~~~

#### 5.4.5 컬럼명 변경하기
- withColumnRenamed 메서드로 컬럼명 변경 가능
- 첫 번째 인수로 전달된 컬럼명을 두 번째 인수의 문자열로 변경함
~~~scala
df.withColumnRenamed("DEST_COUNTRY_NAME", "dest").columns
~~~

#### 5.4.6 예약 문자와 키워드
- 공백이나 '-'는 컬럼명에 사용 불가능
- 예약 문자를 컬럼명에 사용하려면 백틱(`) 문자를 이용해 이스케이핑 해야함
- withColumn 메서드를 사용해 예약 문자가 포함된 컬럼 생성 예제
~~~scala
import org.apache.spark.sql.functions.expr
val dfWithLongColName = df.withColumn(
  "This Long Column-Name",
  expr("ORIGIN_COUNTRY_NAME"))
~~~
- 위 예제에서는 withColumn 메서드의 첫 번째 인수로 새로운 컬럼명을 나타내는 문자열을 지정했기 때문에 이스케이프 문자가 필요 없음
- <b>표현식</b>에서는 컬럼을 참조하므로, 백틱(`) 문자 사용
~~~scala
dfWithLongColName.selectExpr(
  "`This Long Column-Name`",
  "`This Long Column-Name` as `new col`"
).show(2)
~~~
- 표현식 대신 문자열을 사용해 명시적으로 컬럼을 참조하면 리터럴로 해석되기 때문에  
  예약 문자가 포함된 컬럼 참조 가능
- 예약 문자나 키워드를 사용하는 표현식에만 이스케이프 처리 필요
- 다음 두 예제는 모두 같은 DataFrame을 만들어냄
~~~scala
dfWithLongColName.select(col("This Long Column-Name")).columns
dfWithLongColName.select(expr("`This Long Column-Name`")).columns
~~~

#### 5.4.7 대소문자 구분
- Spark는 기본적으로 대소문자를 가리지 않음
- 다음과 같은 설정을 사용해 Spark에서 대소문자 구분하게 만들 수 있음
~~~scala
set spark.sql.caseSensitive true
~~~

#### 5.4.8 컬럼 제거하기
- `drop` 메소드 사용하여 제거가능
~~~scala
df.drop("ORIGIN_COUNTRY_NAME").columns
df.drop("ORIGIN_COUNTRY_NAME", "DEST_COUNTRY_NAME")
~~~

#### 5.4.9 컬럼의 데이터 타입 변경하기
- `cast` 메서드를 통해 변경 가능
- 다음은 count 컬럼을 Integer 데이터 타입에서 String 데이터 타입으로 형변환하는 예제
~~~scala
df.withColumn("count2", col("count").cast("string"))
~~~

#### 5.4.10 로우 필터링하기
- 로우를 필터링하려면 참과 거짓을 판별하는 표현식이 필요
- DataFrame의 가장 일반적인 표현식이나 컬럼을 다루는 기능을 이용해 표현식을 만드는 것
- `where`, `filter` 메서드로 필터링 가능
- 두 메서드 모두 같은 연산을 수행하며 같은 파라미터 타입 사용
- 이 중 SQL과 유사한 `where` 메서드를 앞으로 계속 사용하겠지만, 
  `filter`도 사용할 수 있다는 점 기억하자
~~~scala
df.filter(col("count") < 2).show(2)
df.where("count < 2").show(2)
~~~
- 여러 필터 적용시 차례대로 필터를 적용하고 판단은 Spark에게 맡겨야 함  
~~~scala
df.where(col("count") < 2).where(col("ORIGIN_COUNTRY_NAME") != "Croatia").show(2)
~~~

#### 5.4.11 고유한 로우 얻기
- distinct 메서드를 사용해 고윳값을 찾을 수 있음
~~~scala 
df.select("ORIGIN_COUNTRY_NAME", "DEST_COUNTRY_NAME").distinct().count()
~~~

#### 5.4.12 무작위 샘플 만들기
- DataFrame의 sample 메서드 사용
- DataFrame에서 표본 데이터 추출 비율을 지정할 수 있으며, 복원 추출이나 비복원 추출의 사용여부 지정 가능
~~~scala
val seed = 5
val withReplacement = false
val fraction = 0.5
df.sample(withReplacement, fraction, seed).count()
~~~

#### 5.4.13 임의 분할하기
- 분할 가중치를 함수의 파라미터로 설정해 원본 DataFrame을 서로 다룬 데이터를 가진 두 개의  
  DataFrame으로 나누는 예제
~~~scala
val dataFrames = df.randomSplit(Array(0.25, 0.75), seed)
dataFrames(0).count() > dataFrames(1).count()
~~~

#### 5.4.14 로우 합치기와 추가하기
- DataFrame은 불변하므로, 레코드를 합치려면 기존의 DataFrame을 수정하기엔 불가능하며 원본 DataFrame을 새로운 DataFrame과 통합해야함  
- 통합하려면 두 개의 DataFrame은 반드시 동일한 스키마와 컬럼 수를 가져야 함
- <b>union 메소드는 컬럼명(스키마) 기반 병합이 아닌, 컬럼 위치를 기반으로 동작</b>
~~~scala
import org.apache.spark.sql.Row
val schema = df.schema
val newRows = Seq(
  Row("New Country", "Other Country", 5L),
  Row("New Country 2", "Other Country 3", 1L)
)

val parallelizedRows = spark.sparkContext.parallelized(newRows)
val newDF = spark.createDataFrame(parallelizedRows, schema)

df.union(newDF)
.where("count = 1")
.where($"ORIGIN_COUNTRY_NAME" =!= "United States")
.show() // 전체 데이터 조회시, 신규 데이터 확인 가능
~~~
- 스칼라에서는 반드시 `=!=` 연산자 사용해야 함  
  컬럼 표현식과 문자열을 비교했을 때, `=!=` 연산자 사용시 컬럼 표현식($"ORIGIN_COUNTRY_NAME")  
  이 아닌 컬럼의 실제값을 비교 대상 문자열(United States)과 비교
- DataFrame을 View로 만들거나, 테이블로 등록하면 DataFrame의 변경작업과 관계없이  
  동적으로 참조 가능

#### 5.4.15 로우 정렬하기
- `sort`와 `orderBy` 메서드를 이용해 DataFrame 정렬 가능
- 두 메서드는 완전히 같은 방식으로 동작함  
  (spark code를 살펴보면 orderBy 메소드에서 sort 메소드 호출)
- 두 메서드 모두 컬럼 표현식과 문자열을 사용할 수 있음. 또한 다수의 컬럼 지정 가능
- 기본 동작 방식은 오름차순 정렬
~~~scala
// Scala
import org.apache.spark.sql.functions.{desc, asc}
df.orderBy(expr("count desc")).show(2)
df.orderBy(desc("count"), asc("DESC_COUNTRY_NAME")).show(2)
~~~
~~~SQL
// SQL
SELECT * FROM dfTable ORDER BY count DESC, DEST_COUNTRY_NAME ASC LIMIT 2
~~~
- `asc_nulls_first`, `desc_nulls_first`, `asc_nulls_last`, `desc_nulls_last` 메서드를 이용하여 null 값을 가지는 레코드의 위치를 지정할 수 있음  
- 예를들어 `asc_nulls_first` 함수는 null값은 제일 상단에 위치함
- 트랜스포메이션을 처리하기 전에 성능 최적화를 위해 파티션별 정렬을 수행하기도 함  
  `sortWithinPartitions` 메서드로 가능
~~~scala
spark.read.format("json").load("/data/flight-data/json/*-summary.json")
.sortWithinPartitions("count")
~~~

#### 5.4.16 로우 수 제한하기
- DataFrame에서 추출할 로우 수를 제한해야 할 때가 있음  
  예를들어 DataFrame에서 상위 10개의 결과만을 보고자 할 때 등
- `limit` 메서드를 이용해서 추출할 로우 수 제한
~~~scala
df.limit(5).show()
df.orderBy(expr("count desc")).limit(6).show()
~~~

#### 5.4.17 repartition과 coalesce
- 또 다른 최적화 기법으로 자주 filtering하는 컬럼을 기준으로 데이터를 분할 하는 것
- 이를 통해 파티셔닝 스키마와 파티션 수를 포함해 클러스터 전반의 물리적 데이터 구성을 제어할 수 있음
- `repartition` 메서드를 호출하면 무조건 전체 데이터를 셔플함  
  향후에 사용할 파티션 수가 현재 파티션 수보다 많거나 컬럼을 기준으로 파티션을 만드는 경우에만 사용해야 함
~~~scala
df.rdd.getNumPartitions // 1
df.repartition(5)       //- partition 수 재지정
df.repartition(col("DSET_COUNTRY_NAME")) //- 자주 필터링되는 컬럼을 기준으로 파티션 재분배
~~~
- 선택적으로 파티션 수를 지정할 수도 있음
~~~scala
df.repartition(5, col("DEST_COUNTRY_NAME"))
~~~
- `coalesce` 메서드는 전체 데이터를 셔플하지 않고 파티션을 병합하려는 경우에 사용  
(파티션 수를 줄이려면 셔플이 일어나는 `repartition` 대신 `coalesce` 사용해야 함)  
- 다음은 목적지를 기준으로 셔플을 수행해 5개의 파티션으로 나누고, 전체 데이터를 셔플 없이 평합하는 예제
~~~scala
df.repartition(5, col("DEST_COUNTRY_NAME")).coalesce(2)
~~~

#### 5.4.18 드라이버로 로우 데이터 수집하기
- Spark는 드라이버에서 클러스터 상태 정보를 유지함 
- 로컬 환경에서 데이터를 다루려면 드라이버로 데이터를 수집해야함
- 아직 드라이버로 데이터를 수집하는 연산은 정확하게 설명하지 않았지만  
  몇 가지 메서드는 사용해 보았음
- `collect` 메서드는 전체 DataFrame의 모든 데이터를 수집  
  (데이터셋을 드라이버에 다시 가져오기 위해 액션을 수행)
  `take` 메서드는 상위 N개의 로우를 반환  
  `show` 메서드는 여러 로우를 보기 좋게 출력
~~~scala
val collectDF = df.limit(10)
collectDF.take(5)
collectDF.show()
collectDF.show(5, false)
collectDF.collect()
~~~

- 전체 데이터셋에 대한 반복(iterate) 처리를 위해 드라이버로 로우를 모우는 또 다른 방법이 있음  
- `toLocalIterator` 메서드는 이터레이터(iterator) 로 모든 파티션의 데이터를 드라이버로 전달  
  `toLocalIterator` 메서드를 사용해 데이터셋의 파티션을 차례로 반복 처리 가능
~~~scala
collectDF.toLocalIterator()
~~~
- 드라이버로 모든 데이터 컬랙션을 수집하는 작업은 매우 큰 비용(CPU, 메모리, 네트워크)이 발생
- 대규모 데이터셋에 `collect` 명령을 수행하면 드라이버가 비정상적으로 종료될 수 있음
- `toLocalIterator` 메서드도 마찬가지임. 또한 연산을 병렬로 수행하지 않고 차례로 처리하기 때문에 시간이 오래걸림

### 용어 정리
- 함수와 메소드의 차이
  - 함수는 독립적이고, 메소드는 class에 종속적임
- 스키마-온-리드
  - 데이터의 스키마 파악을 데이터를 read하는 시점에 한다는 의미