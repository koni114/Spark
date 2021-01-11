# RDD, DataFrame, Dataset
- Spark 어플리케이션을 구현하는 방법은 Spark v1에서 발표한 RDD를 사용하는 방법과  
  RDD의 단점으로 개선하여 발표한 Dataset과 DataFrame을 이용하는 방법 두 가지가 있음

## Dataset, DataFrame을 이용한 애플리케이션 처리 방법

### Spark-session 초기화  
- dataset, dataframe은 spark-session을 이용하여 처리
- spark-session을 초기화하는 방법은 다음과 같음
~~~scala
import org,apache.spark.sql.SparkSession

val spark = SparkSession
 .builder()
 .appName("Spark SQL basic example")
 .config("spark.some.config.option", "some-value")
 .getOrCreate()
~~~
- `spark-shell`을 이용할 경우, REPL shell이 spark-session 객체를 초기화함

#### 하이브 메타스토어 연결
- Spark-session은 단독으로 사용할 수도 있지만, 하이브 메타스토어와 연결하여 사용할 수 있음
- Spark-session 생성시에 `hive.metastore.uris` 값을 설정하면 메타스토어와 연결됨
~~~scala
// hive.metastore.uris 옵션에 하이브 메타스토어 접속 주소를 입력한다. 
val spark = SparkSession.builder().appName("sample").config("hive.metastore.uris", "thrift://hive_metastore_ip:hive_metastore_port").enableHiveSupport().getOrCreate()

// 데이터베이스 조회 
scala> spark.sql("show databases").show()
+-------------+
| databaseName|
+-------------+
|      test_db1|
|      test_db2|
~~~

### DataFrame 초기화
#### DataFrame 초기화
- spark-session의 `read` 메소드로 생성 할 수 있음
- `read`는  json, parquet, orc, text 등 다양한 형식의 데이터를 읽을 수 있음
- 다음 예제는 스파크 세션을 이용하여 데이터 프레임을 초기화 하는 방법
- . people.json 파일을 읽어서 데이터 프레임을 생성함
~~~scala
// json 형식의 데이터 입력 
$ cat people.json
{"name":"Michael"}
{"name":"Andy", "age":30}
{"name":"Justin", "age":19}

val df = spark.read.json("/user/people.json")
scala> df.show()
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+
~~~
#### RDD를 이용한 데이터 프레임 초기화
- RDD를 이용한 데이터 초기화는 여러가지 방법이 있는데,  
  스키마구조를 지정할 수도 있고, 지정하지 않으면 스파크에서 임시 칼럼명을 지정
- 배열 RDD 데이터를 데이터프레임으로 초기화하는 예제
~~~scala
val wordsRDD = sc.parallelize(Array("a", "b", "c", "d", "a", "a", "b", "b", "c", "d", "d", "d", "d"))
val wordsDF = wordsRDD.toDF()


// 데이터 프레임 확인 
scala> wordsDF.show()
+-----+
|value|
+-----+
|    a|
|    b|
|    c|

// 칼럼명을 지정했을 때 
val wordsDF = wordsRDD.toDF("word")
scala> wordsDF.show()
+----+
|word|
+----+
|   a|
|   b|
|   c|
~~~

#### 복합구조의 RDD를 데이터 프레임으로 초기화
- 칼럼이 여러개인 데이터를 이용하여 데이터 프레임을 초기화는 다음과 같음
~~~scala
val peopleRDD = sc.parallelize(
  Seq( ("David", 150),
       ("White", 200),
       ("Paul",  170) )
)

val peopleDF = peopleRDD.toDF("name", "salary")
scala> peopleDF.show()
+-----+------+
| name|salary|
+-----+------+
|David|   150|
|White|   200|
| Paul|   170|
+-----+------+
~~~

#### 스키마를 생성하여 데이터 프레임 초기화
-`StrucType`, `StrucField` 를 이용하여 스키마를 생성하고, 이를 이용하여 DataFrame을 초기화함
~~~scala
import org.apache.spark.sql._
import org.apache.spark.sql.types._

// RDD를 Row 로 초기화 
val peopleRDD = sc.parallelize(
  Seq(
       Row("David", 150),
       Row("White", 200),
       Row("Paul",  170)
  )
)

// RDD를 데이터프레임으로 변형하기 위한 스키마 생성
val peopleSchema = new StructType().add(StructField("name",   StringType, true)).add(StructField("salary", IntegerType, true))

// 데이터 프레임 생성 
val peopleDF = spark.createDataFrame(peopleRDD, peopleSchema)

scala> peopleDF.show()
+-----+------+
| name|salary|
+-----+------+
|David|   150|
|White|   200|
| Paul|   170|
+-----+------+
~~~

#### 외부 데이터를 읽어서 데이터 프레임 초기화
- 외부 데이터를 읽어서 데이터 프레임을 초기화 할 수도 있음
- json 형태의 파일은 구조를 가지고 있기 때문에 자동으로 스키마를 생성
- txt 형태의 파일은 구조가 없기 때문에 스키마를 생성하여 초기화
- TXT 파일을 이용한 데이터 프레임 초기화 예제
~~~scala
val peopleRDD = sc.textFile("/user/people.txt")
val peopleSchema = new StructType().add(StructField("name",   StringType, true)).add(StructField("age", IntegerType, true))
val sepPeopleRdd = peopleRDD.map(line => line.split(",")).map(x => Row(x(0), x(1).trim.toInt))
val peopleDF = spark.createDataFrame(sepPeopleRdd, peopleSchema)

scala> peopleDS.show()
+----+---+
|name|age|
+----+---+
|   A| 29|
|   B| 30|
|   C| 19|
|   D| 15|
|   F| 20|
+----+---+
~~~

#### JSON 파일을 이용한 데이터 프레임 초기화
- JSON 형태의 파일은 데이터가 구조화 되어 있기 때문에 자동으로 초기화 됨
~~~scala
val peopleDF = spark.read.json("/user/shs/people.json")

scala> peopleDF.show()
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+
~~~

### 데이터프레임 연산
- 스키마 구조를 가지고 있기 때문에 쿼리를 날리는 것처럼 작업할 수 있음
- 데이터 프레임 연산 방법은 크게 2가지 
  - 구조화된 형태의 데이터를 명령어 체인을 이용 연산 
  - SQL을 이용하여 연산할 수도 있음

#### 스키마 확인
- `printSchema`를 이용. 현재 데이터프레임 구조를 출력함
~~~scala
val df = spark.read.json("/user/people.json")

// 스키마 출력 
scala> df.printSchema()
root
 |-- age: long (nullable = true)
 |-- name: string (nullable = true)

scala> df.show()
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+
~~~

#### 조회
- 데이터 조회는 `select`를 이용합니다. 칼럼 데이터에 대한 연산을 하고 싶을때는 `$` 기호를 이용하여 처리
~~~scala
// name 칼럼만 조회 
scala> df.select("name").show()
+-------+
|   name|
+-------+
|Michael|
|   Andy|
| Justin|
+-------+

// name, age 순으로 age에 값을 1더하여 조회 
scala> df.select($"name", $"age" + 1).show()
+-------+---------+
|   name|(age + 1)|
+-------+---------+
|Michael|     null|
|   Andy|       31|
| Justin|       20|
+-------+---------+
~~~

#### show() 함수 설정
- 조회 결과를 확인하는 show() 함수를 이용할 때 기본적으로 보여주는 데이터의 길이와 칼럼의 사이즈를 제한하여 출력
- show() 함수는 아래와 같이 출력하는 라인의 개수와 칼럼 사이즈 조절 여부를 설정할 수 있음
~~~scala
// show 함수 선언 
def show(numRows: Int, truncate: Boolean): Unit = println(showString(numRows, truncate))

// 사용방법 
scala> show(10, false)
scala> show(100, true)
~~~

#### 필터링
- 필터링은 filter를 이용하여 처리
~~~scala
// 필터링 처리 
scala> df.filter($"age" > 21).show()
+---+----+
|age|name|
+---+----+
| 30|Andy|
+---+----+

// select에 filter 조건 추가 
scala> df.select($"name", $"age").filter($"age" > 20).show()
scala> df.select($"name", $"age").filter("age > 20").show()
+----+---+
|name|age|
+----+---+
|Andy| 30|
+----+---+
~~~

#### 그룹핑
-그룹핑은 groupBy를 이용하여 처리
~~~scala
// 그룹핑 처리 
scala> df.groupBy("age").count().show()
+----+-----+
| age|count|
+----+-----+
|  19|    1|
|null|    1|
|  30|    1|
+----+-----+
~~~

#### 칼럼 추가
- 새로운 칼럼을 추가할 때는 withColumn을 이용
~~~scala
// age가 NULL일 때는 KKK, 값이 있을 때는 TTT를 출력 
scala> df.withColumn("xx", when($"age".isNull, "KKK").otherwise("TTT")).show()
+----+-------+---+
| age|   name| xx|
+----+-------+---+
|null|Michael|KKK|
|  30|   Andy|TTT|
|  19| Justin|TTT|
+----+-------+---+
~~~

### 데이터프레임의 SQL을 이용한 데이터 조회
#### 뷰 생성
- 데이터프레임은 SQL 쿼리를 이용하여 데이터를 조회
- 데이터프레임을 이용하여 뷰를 생성하고, SQL 쿼리를 실행하면 됨
~~~scala
val df = spark.read.json("/user/people.json")

// DataFrame으로 뷰를 생성 
df.createOrReplaceTempView("people")

// 스파크세션을 이용하여 SQL 쿼리 작성 
scala> spark.sql("SELECT * FROM people").show()
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+
~~~

#### SQL 사용
- 생성한 뷰를 이용하여 데이터베이스에 문의하듯이 SQL을 호출하면 됨
~~~scala
// 조회 조건 추가 
scala> spark.sql("SELECT * FROM people WHERE age > 20").show()
+---+----+
|age|name|
+---+----+
| 30|Andy|
+---+----+

// 그룹핑 추가 
scala> spark.sql("SELECT age, count(1) FROM people GROUP BY age").show()
+----+--------+
| age|count(1)|
+----+--------+
|  19|       1|
|null|       1|
|  30|       1|
+----+--------+
~~~

## 데이터셋 초기화
- 데이터셋 연산에 대해서 알아보자
- 데이타셋은 RDD와 유사하지만 객체를 직렬화 할때 자바의 기본 시리얼라이제이션이나 kyro를 사용하지 않고,  
  스파크의 인코더(Encoder) 를 이용하여 RDD 보다 속도가 빠름
- 데이터셋 초기화는 내부 데이터를 이용하는 방법과 외부 데이터를 이용하는 방법이 있음

### 내부 데이터 이용한 초기화
- 내부 데이터를 이용한 데이터셋 초기화는 다음과 같음
~~~scala
val seq = Seq(
       ("David", 150),
       ("White", 200),
       ("Paul",  170)
  )
val peopleDS = seq.toDS()

scala> peopleDS.show()
+-----+---+
|   _1| _2|
+-----+---+
|David|150|
|White|200|
| Paul|170|
+-----+---+

scala> peopleDS.select("_1").show()
+-----+
|   _1|
+-----+
|David|
|White|
| Paul|
+-----+
~~~

### 케이스 클래스를 이용한 초기화
- 케이스 클래스를 이용한 내부 데이터 초기화는 다음과 같습니다. 케이스 클래스를 이용하면 데이터를 조회할 때 칼럼명을 이용할 수 있음
~~~scala
case class People(name: String, salary: Int)
val peopleSeq = Seq(
       People("David", 150),
       People("White", 200),
       People("Paul",  170)
  )
val peopleDS = peopleSeq.toDS()

scala> peopleDS.show()
+-----+------+
| name|salary|
+-----+------+
|David|   150|
|White|   200|
| Paul|   170|
+-----+------+


scala> peopleDS.select("salary").show()
+------+
|salary|
+------+
|   150|
|   200|
|   170|
+------+
~~~

### RDD를 데이터셋으로 초기화
- RDD를 데이터셋으로 변환하기 위해서는 데이터 프레임으로 변경하고 데이터셋으로 변환하면 됨
~~~scala
import org.apache.spark.sql._
import org.apache.spark.sql.types._

val peopleRDD = sc.textFile("/user/people.txt")
val peopleSchema = new StructType().add(StructField("name",   StringType, true)).add(StructField("age", IntegerType, true))
val sepPeopleRdd = peopleRDD.map(line => line.split(",")).map(x => Row(x(0), x(1).trim.toInt))
val peopleDF = spark.createDataFrame(sepPeopleRdd, peopleSchema)
peopleDF.show()

case class People(name: String, age: Long)
val peopleDS = peopleDF.as[People]
peopleDS.show()
~~~

### 데이타프레임을 데이터셋으로 초기화
- 데이터프레임을 데이터셋으로 변환하는 것은 데이터프레임에 정적데이터로 변경을 위해서 `as`를 이용해서 클래스를 지정해 주면 변환됨
- 스키마와 클래스 이름이 같으면 자동으로 바인딩 됨
~~~scala
case class People(name: String, age: Long)
val peopleDF = spark.read.json("/user/people.json")
val peopleDS = peopleDF.as[People]

scala> peopleDS.show()
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+
~~~