## chapter06 다양한 데이터 타입 다루기
- 이 장에서는 Spark의 구조적 연산에서 가장 중요한 내용인 표현식을 만드는 방법을 알아보자
- 또한 다양한 데이터 타입을 다루는 방법도 알아보자
- 타입 종류
  - 불리언 타입 
  - 수치 타입
  - 문자열 타입
  - date와 timestampe 타입
  - null 값 다루기 
  - 복합 데이터 타입
  - 사용자 정의 함수

### 6.1 API는 어디서 찾을까
- spark는 여전히 활발하게 성장하고 있는 프로젝트이므로 , 데이터 변환용 함수를 어떻게 찾는지 알아야 함
- 데이터 변환용 함수를 찾기 위해 핵심적으로 보아야 할 부분은 다음과 같음
  - DataFrame(Dataset) 메서드  
    - DataFrameStatFunction 와 DataFrameNaFunctions 등 Dataset의 하위 모듈은 다양한 메서드를 제공
    - DataFrameStatFunctions는 다양한 통계적 함수 제공
    - DataFrameNaFunctions는 null 데이터를 다루는 데 필요한 함수 제공
  - Column 메서드
    - Column API는 Spark 문서를 참조(http://bit.ly/2FloFbr)  
    - org.apache.spark.sql.functions 패키지는 데이터 타입과 관련된 다양한 함수 제공

- 다음은 분석에 사용할 DataFrame을 생성하는 예제
~~~scala
val df = spark.read.format("csv")
.option("header", "true")
.option("inferSchema", "true")
.load("C:/Spark-The-Definitive-Guide-master/data/retail-data/by-day/2010-12-01.csv")
df.printSchema()
df.createOrReplaceTempView("dfTable")
~~~

### 6.2 Spark 데이터 타입으로 변환하기
- 프로그래밍 언어의 고유 데이터 타입을 Spark 데이터 타입으로 변환해보자
- <b>Spark 데이터 타입 변환은 반드시 알아야 함</b>
- `lit` 함수를 이용
- `lit` 함수는 다른 언어의 데이터 타입을 Spark 데이터 타입에 맞게 변환
- Scala와 python의 다양한 데이터 타입을 Spark 데이터 타입으로 변환해보자
~~~scala
import org.apache.spark.sql.functions.lit
df.select(lit(5), lit("five"), lit(5.0))
~~~
- SQL에서는 Spark 데이터 타입으로 변환할 필요가 없으므로 직접 입력
~~~SQL
SELECT 5, "five", 5.0
~~~

### 6.3 불리언 데이터 타입 다루기
- 불리언은 모든 필터링 작업의 기반이므로 데이터 분석에 필수적임
- 불리언 구문은 and, or, true, false로 구성
- 불리언 구문을 이용해 true 또는 false로 평가되는 논리 문법을 만듬
- 다음은 소매 데이터셋을 사용해 불리언을 다루는 예제
~~~scala
import org.apache.spark.sql.functions.col
df.where(col("InvoiceNo").equalTo(536365))
.select("InvoiceNo", "Description")
.show(5, false)
~~~
- Spark에서 동등 여부를 판별해 필터링하려면 <b>===</b> 또는 <b>=!=</b>를 사용해야 함  
  또는 not 함수나 equalTo 메서드를 사용할 수 있음
- 가장 명확한 방법은 문자열 표현식에 조건절 명시  
  이 방법은 python과 scala 모두 사용가능하며, 다음과 같이 일치하지 않음을 표현할 수 있음
~~~scala
df.where("InvoiceNo = 536365")
.show(5, false)
df.where("InvoiceNo <> 536365")
.show(5, false)
~~~
- and 메서드나 or 메서드를 사용해서 불리언 표현식을 여러 부분에 지정 가능
- 불리언 표현식을 사용하는 경우 항상 모든 표현식을 and 메서드로 묶어 차례대로 필터를 적용해야 함  
그 이유는 불리언 문을 차례대로 표현하더라도 Spark는 내부적으로 and 구문을 필터 사이에 추가해 모든 필터를 하나의 문장으로 변환  
그런 다음 모든 필터를 처리
- 원한다면 and 구문으로 조건문을 만들 수도 있지만 차례로 조건을 나열하면 이해하기 쉽고 읽기도 편함
- or 구문 사용시 반드시 동일한 구문에 조건을 정의해야 함
~~~scala
val priceFilter = col("UnitPrice") > 600
val descripFilter = col("Description").contains("POSTAGE")
df.where(col("StockCode").isin("DOT")).where(priceFilter.or(descripFilter)).show()
~~~
~~~SQL
// SQL
SELECT * FROM dfTable WHERE StockCode in ("DOT") AND (UnitPrice > 600 OR instr(Description, "POSTAGE") >= 1)
~~~
- 불리언 컬럼을 사용해 DataFrame을 filtering 할 수 있음  
- 하단 예제는 filtering 조건들을 조합하여 isExpensive 불리언 컬럼을 추가하고, 해당 불리언으로 filtering하는 예제
~~~scala
val DOTCodeFilter = col("StockCode") === "DOT"
val priceFilter = col("UnitPrice") > 600
val descripFilter = col("Description").contains("POSTAGE") 

df.withColumn("isExpensive", DOTCodeFilter.and(priceFilter.or(descripFilter)))
.where("isExpensive") // alias
.select("UnitPrice", "isExpensive").show(5)
~~~
~~~SQL
// SQL
SELECT UnitPrice, (StockCode = 'DOT' and (UnitPrice > 600 OR instr(Description, "POSTAGE") >= 1)) as isExpensive
FROM dfTable
WHERE StockCode = 'DOT' and (UnitPrice > 600 OR instr(Description, "POSTAGE") >= 1)
~~~
- SQL이 익숙하다면 지금까지 설명한 모든 구문이 익숙할 것임
- 이 모든 구문은 where절로 표현할 수 있음
- 필터를 표현해야 한다면 DataFrame 인터페이스 방식보다 SQL이 훨씬 쉬움  
  Spark SQL을 사용한다고 성능 저하가 발생하는 것은 아님!
- 다음 두 문장은 동일하게 처리됨
~~~scala
import org.apache.spark.sql.functions.{expr, not, col}
// dataFrame IF
df.withColumn("isExpensive", not(col("UnitPrice").leq(250)))
.filter("isExpensive")
.select("Description", "UnitPrice").show(5)

// SQL IF
df.withColumn("isExpensive", expr("NOT UnitPrice <= 250"))
.filter("isExpensive")
.select("Description", "UnitPrice").show(5)
~~~
- 불리언 표현식을 만들 때 <b>null 값 데이터</b>는 조금 다르게 처리해야 하는데,  
  예를 들어 null 값에 안전한(null-safe) 동치 테스트 수행할 수 있음
~~~scala
df.where(col("Description").eqNullSafe("hello")).show()
~~~

### 6.4 수치형 데이터 다루기
- `count`는 빅데이터 처리에서 필터링 다음으로 많이 수행하는 작업
- 상황을 가정해 가상의 예제를 만들어보자  
  소매 데이터셋의 수량을 잘못 기록했고, 실제 수량은 (현재 수량 * 단위 가격)^2 + 5 공식으로 구할 수 있다는 사실을 알게됨
- `pow` 함수를 사용해보자
~~~scala
import org.apache.spark.sql.functions.{expr, pow}
val fabricatedQuantity = pow(col("Quantity") * col("UnitPrice"), 2) + 5
df.select(expr("CustomerId"), fabricatedQuantity.alias("realQuantity")).show(2)
~~~
~~~scala
df.selectExpr(
    "CustomerId",
    "(POWER((Qunatity * UnitPrice), 2.0) + 5) as realQuantity").show(2)
~~~
- 반올림, 소수점 자리를 버리기 위해 Integer 변환 등 사용 가능
  - `round` : 5 이상 반올림
  - `bround` : 버림
~~~scala 
import org.apache.spark.sql.functions.{round, bround}
df.select(round(col("UnitPrice"), 1).alias("rounded"), col("UnitPrice")).show(5)
// 결과
+-------+---------+
|rounded|UnitPrice|
+-------+---------+
|    2.6|     2.55|
|    3.4|     3.39|
|    2.8|     2.75|
|    3.4|     3.39|
|    3.4|     3.39|
+-------+---------+
~~~
~~~scala
import org.apache.spark.sql.functions.lit
df.select(round(lit("2.5")), bround(lit("2.5"))).show(2)
// 결과
+-------------+--------------+
|round(2.5, 0)|bround(2.5, 0)|
+-------------+--------------+
|          3.0|           2.0|
|          3.0|           2.0|
+-------------+--------------+
~~~
- 다음과 같이 피어슨 상관계수를 구할 수도 있음
~~~scala
import org.apache.spark.sql.functions.{corr}
df.stat.corr("Quantity", "UnitPrice")
df.select(corr("Quantity", "UnitPrice")).show() 
~~~
~~~SQL
SELECT corr(Quantity, UnitPrice) FROM dfTable
~~~
- 하나 이상의 컬럼에 대한 요약 통계를 계산하는 작업 역시 자주 수행됨
- 요약 통계는 `describe` 메서드를 사용해 얻을 수 있음
- `describe` 메서드는 관련 컬럼에 대한 집계(count), 평균(mean), 표준편차(std), 최솟값(min) , 최댓값(max) 값을 계산
- 통계 스키마는 변경될 수 있으므로 `descibe` 메소드는 콘솔 확인용으로만 사용해야 함
~~~scala
df.describe().show()
~~~
- 정확한 수치가 필요하다면 함수를 import 해서 컬럼에 적용하는 방식으로 직접 수행 가능
~~~scala
import org.apache.spark.sql.functions.{count, mean, stddev_pop, min, max}
~~~
- `StatFunctions` 패키지는 다양한 통계 함수 제공  
  stat 속성을 사용해 접근할 수 있으며 다양한 통곗값을 계산할 때 사용하는 DataFrame 메서드임
- 예를 들어 `approxQuantile` 메서드를 사용해 데이터의 백분위수를 정확하게 계산하거나 근사치 계산 가능
~~~scala
val colName = "UnitPrice"
val quantileProbs = Array(0.5)
val relError = 0.05
df.stat.approxQuantile("UnitPrice", quantileProbs, relError) // 2.51
~~~
- StatFunctions 패키지는 교차표(cross-tabulation)나 자주 사용하는 항목 쌍을 확인하는 용도의 메서드 제공  
  연산결과가 너무 크면 화면에 모두 보이지 않을 수 있음
~~~scala
df.stat.crosstab("StockCode", "Quantity").show()
~~~
- `StatFunctions` 패키지의 `monotonically_increasing_id` 사용하면 모든 로우에 고유 Id 부여
- 0부터 시작
~~~scala
import org.apache.spark.sql.functions.monotonically_increasing_id
df.select(monotonically_increasing_id()).show(2)
~~~

### 6.5 문자열 데이터 타입 다루기
- 로그 파일에 정규표현식을 사용해 데이터 추출, 데이터 치환, 문자열 존재 여부, 대/소문자 변환 처리등의 작업을 할 수 있음
- 대/소문자 변환 작업부터 시작해보자. `initcap` 함수는 주어진 문자열에서 공백으로 나뉘는 모든 단어의 첫 글자를 대문자로 변경함
~~~scala
import org.apache.spark.sql.functions.{initcap}
df.select(initcap(col("Description"))).show(2, false) // false --> 글자 생략 x
~~~
- `lower` 함수나 `upper` 함수를 이용해 문자열 전체를 대문자로 변경할 수 있음
~~~scala
import org.apache.spark.sql.functions.{lower, upper}
df.select(col("Description"),
lower(col("Description")),
upper(lower(col("Description")))
).show(2)
~~~
- `lpad`, `rpad` : 문자열 공백 추가, `lstrip`, `rstrip`, `strip` : 문자열 공백 제거

#### 6.5.1 정규 표현식
- 정규표현식을 사용해 문자열에서 값을 추출하거나 다른 값으로 치환하는데 필요한 규칙 모음을 정의할 수 있음
- Spark는 자바 정규 표현식이 가진 강력한 능력을 활용  
  자바 정규 표현식 문법은 보통 사용하는 언어의 문법과 약간 다르므로, 운영 환경에서 정규 표현식을 사용하기 전에 다시 한번 검토해야 함
- Spark는 정규 표현식을 위해 `regexp_extract` 함수와 `regexp_replace` 함수를 제공  
  이 함수들은 값을 추출하고 치환하는 역할 수행
- `regexp_replace`를 이용해 'description' 컬럼의 값을 'COLOR'로 치환해보자
~~~scala
import org.apache.spark.sql.functions.regexp_replace
val simpleColors = Seq("black", "white", "red", "green", "blue")
val regexString = simpleColors.map(_.toUpperCase).mkString("|") // map 함수를 이용하여 simpleColors 의 값들을 | 연산자로 연결
df.select(
    regexp_replace(col("Description"), regexString, "COLOR").alias("color_clean"),
    col("Description")).show(2)
~~~
- 주어진 문자를 다른 문자로 치환해야 할 때도 있음
- `translate` 함수를 이용해 문자 치환
- 이 연산은 문자 단위로 이루어 짐. 교체 문자열에서 색인된 문자에 해당되는 모든 문자 치환함
~~~scala
import org.apache.spark.sql.functions.translate
// L = 1, E = 3, T = 7로 치환
df.select(translate(col("Description"), "LEET", "1337")).col("Description")
~~~
- 처음 나타난 색상 이름을 추출하는 것과 같은 작업 수행 가능
~~~scala
 import org.apache.spark.sql.functions.regexp_extract
 val simpleColors = Seq("black", "white", "red", "green", "blue")
 val regexString  = simpleColors.map(_.toUpperCase).mkString("(", "|", ")")
 df.select(
     regexp_extract(col("Description"), regexSring, 1).alias("color_clean"),
     col("Description")).show(2)
 )
~~~
- `contains` 메소드를 사용하여 값 추출 없이 단순히 값의 존재 여부를 확인 할 수 있음  
  해당 메서드는 불리언 타입으로 반환
~~~scala
val containsBlack = col("Description").contains("BLACK")
val containsWhite = col("DESCRIPTION").contains("WHITE")
df.withColumn("hasSimpleColor", containsBlack.or(containsWhite))
.where("hasSimpleColor")
.select("Description").show(3, false)
~~~
- 값의 개수가 늘어나면 복잡해짐
- 동적으로 인수가 변하는 상황을 Spark는 어덯게 처리하는지 알아보자
- 값 목록을 인수로 변환해 함수에 전달할 때는 varargs라 불리는 스칼라 고유 기능을 사용  
  이 기능을 사용해 임의 길이의 배열을 효율적으로 다룰 수 있음
- `select` 메서드와 `varargs`를 함께 사용해 원하는 만큼 동적으로 컬럼 생성할 수 있음

~~~scala
val simpleColors = Seq("black", "white", "red", "green", "blue")
val selectedColumns = simpleColors.map(color => {
    col("Description").contains(color.toUpperCase).alias(s"is_$color")
}):+expr("*")

df.select(selectedColumns:_*).where(col("is_white").or(col("is_red"))).select("Description").show(3, false)
~~~
- `locate` 함수를 사용하면 문자열의 위치를 정수로 반환