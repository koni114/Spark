## chapter06 다양한 데이터 타입 다루기
- 이 장에서는 Spark의 구조적 연산에서 가장 중요한 내용인 표현식을 만드는 방법을 알아보자
- 또한 다양한 데이터 타입을 다루는 방법도 알아보자
- 타입 종류
  - 불리언 타입 
  - 수치 타입
  - 문자열 타입
  - date와 timestamp 타입
  - null 값 다루기 
  - 복합 데이터 타입
  - 사용자 정의 함수

### 6.1 API는 어디서 찾을까
- spark는 여전히 활발하게 성장하고 있는 프로젝트이므로 , 데이터 변환용 함수를 어떻게 찾는지 알아야 함
- 데이터 변환용 함수를 찾기 위해 핵심적으로 보아야 할 부분은 다음과 같음
  - DataFrame(Dataset) 메서드  
    - DataFrameStatFunctions 와 DataFrameNaFunctions 등 Dataset의 하위 모듈은 다양한 메서드를 제공
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
df.withColumn("isExpensive", not(col("UnitPrice").leq(250))) // Less than or equal to
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
- 동적으로 인수가 변하는 상황을 Spark는 어떻게 처리하는지 알아보자
- 값 목록을 인수로 변환해 함수에 전달할 때는 varargs라 불리는 스칼라 고유 기능을 사용  
  이 기능을 사용해 임의 길이의 배열을 효율적으로 다룰 수 있음
- <b>`select` 메서드와 `varargs`를 함께 사용해 원하는 만큼 동적으로 컬럼 생성할 수 있음</b>

~~~scala
// 꼭 알아두기!
val simpleColors = Seq("black", "white", "red", "green", "blue")
val selectedColumns = simpleColors.map(color => {
    col("Description").contains(color.toUpperCase).alias(s"is_$color")
}):+expr("*")

df.select(selectedColumns:_*).where(col("is_white").or(col("is_red"))).select("Description").show(3, false)
~~~
- `locate` 함수를 사용하면 문자열의 위치를 정수로 반환

### 6.6 날짜와 timestamp 데이터 타입 다루기
- Spark는 날짜형의 복잡함을 피하고자 두 가지 종류의 시간 관련 정보만 집중적으로 관리함
- 하나는 <b>달력 형태의 날짜(date)</b>이고, 다른 하나는 날짜와 시간 정보를 모두 가지는 <b>timestamp</b>임
- Spark는 inferschema 옵션이 활성화된 경우 날짜와 타임스탬프를 포함해  
  컬럼의 데이터 타입을 최대한 정확하게 식별하려 시도함
- <b>Spark는 특정 날짜 포멧을 명시하지 않아도 자체적으로 식별해 데이터를 읽을 수 있음</b>
- 날짜와 timestamp를 다루는 작업은 문자열을 다루는 작업과 관련이 많음  
  날짜나 시간을 문자열로 저장하고 런타임에 날짜 타입으로 변환하는 경우가 많기 때문
- 데이터베이스나 구조적 데이터를 다룰 때는 이러한 작업이 드물지만,  
  텍스트나 CSV 파일을 다룰 때는 많이 발생함
- 만약 특이한 포맷의 날짜와 시간 데이터를 어쩔 수 없이 읽어야 한다면  
  각 단계별로 어떤 데이터 타입과 포맷을 유지하는지 정확히 알고 트랜스포메이션을 적용해야 함
- `TimestampType` 클래스는 초 단위 정밀도까지 지원함  
  만약 밀리세컨드나 마이크로세컨드 단위를 다룬다면 Long DataType으로 데이터를 변환해  
  처리하는 우회 정책을 사용해야 함
- 그 이상의 정밀도는 `TimestampType`으로 변환될 때 제거
- Spark는 특정 시점에 데이터 포멧이 약간 특이하게 변할 수 있음  
  이런 문제를 피하려면 파싱이나 변환 작업을 해야함
- 다음은 오늘 날짜와 타임스탬프 값을 구하는 예제 `current_date`, `current_timestamp` 함수 사용
~~~scala
import org.apache.spark.sql.functions.{current_date, current_timestamp}
val dateDF = spark.range(10)
.withColumn("today", current_date())
.withColumn("now", current_timestamp())

dateDF.createOrReplaceTempView("dateTable")
dateDF.printSchema()
~~~
- 위 예제로 만들어진 DataFrame을 사용해 오늘을 기준으로 5일 전후의 날짜를 구해보자
- `date_sub` 함수와 `date_add` 함수는 컬럼과 더하거나 뺄 날짜 수를 인수로 전달 해야함
~~~scala
import org.apache.spark.sql.functions.{date_add, date_sub}
dateDF.select(date_sub(col("today"), 5), date_add(col("today"), 5)).show(1)

// 결과
+------------------+------------------+
|date_sub(today, 5)|date_add(today, 5)|
+------------------+------------------+
|        2020-12-24|        2021-01-03|
+------------------+------------------+
only showing top 1 row
~~~
- 두 날짜의 차이를 구하는 작업도 자주 발생함  
  두 날짜 사이의 일 수를 반환하는 `datediff` 함수를 사용해 이러한 작업 수행 가능
- 두 날짜 사이의 개월 수를 반환하는 `months_between` 함수도 있음
~~~scala
import org.apache.spark.sql.functions.{datediff, months_between, to_date}
dateDF.withColumn("week_ago", date_sub(col("today"), 7))
.select(datediff(col("week_ago"), col("today"))).show(1)

dateDF.select(
  to_date(lit("2016-01-01")).alias("start"),
  to_date(lit("2017-05-22")).alias("end"))
.select(months_between(col("start"), col("end"))).show(1)
~~~
- `to_date` 함수는 문자열을 날짜로 변환할 수 있으며 필요에 따라 날짜 포멧도 함께 지정 가능
- 함수의 날짜 포멧은 반드시 <b>Java의 `SimpleDateFormat` 클래스가 지원하는 포멧을 사용해야 함</b>
- Spark는 날짜를 파싱할 수 없다면 null 값을 반환함  
  다단계 처리 파이프라인에서는 조금 까다로울 수 있음. why? 날짜 데이터 포멧의 데이터가 추가로 나타날 수 있기 때문
- 다음은 년-일-월 형태의 날짜 포멧을 사용해보자 
  Spark 는 날짜를 파싱할 수 없으므로, null값 반환  
~~~scala
dateDF.select(to_date(lit("2016-20-12")), to_date(lit("2017-12-11"))).show(1)
// 결과
+---------------------+---------------------+
|to_date('2016-20-12')|to_date('2017-12-11')|
+---------------------+---------------------+
|                 null|           2017-12-11|
+---------------------+---------------------+
only showing top 1 row
~~~ 
- 원래 의도한 날짜인(2017-11-12)가 2017-12-11로 잘못 표기되어 나타나고 있음  
  즉 디버깅하기 매우 까다로움
- 위의 문제를 해결하기 위하여 견고한 방법을 찾아보자
- 먼저 자바의 `SimpleDateFormat`에 맞춰 날짜 포맷을 지정
- 문제를 해결하기 위하여 `to_date`함수와 `to_timestamp` 함수를 사용  
  `to_date` 함수는 필요에 따라 날짜 포맷을 지정할 수 있지만, `to_timestamp` 함수는 반드시 날짜 포멧을 지정해야 함
~~~scala
import org.apache.spark.sql.functions.{to_date}

val dateFormat = "yyyy-dd-MM"
val cleanDateDF = spark.range(1).select(
  to_date(lit("2017-12-11"), dateFormat).alias("date"),
  to_date(lit("2017-20-12"), dateFormat).alias("date2"))

cleanDateDF.createOrReplaceTempView("dateTable2")
~~~
~~~SQL
SELECT to_date(date, 'yyyy-dd-MM'), to_date(date2, 'yyyy-dd-MM'), to_date(date)
FROM dateTable2
~~~
- 항상 날짜 포멧을 지정해야 하는 `to_timestamp` 함수의 예제를 살펴보자
~~~scala
import org.apache.spark.sql.functions.{to_timestamp}
cleanDateDF.select(to_timestamp(col("date"), dateFormat)).show()
// 결과
+----------------------------------+
|to_timestamp(`date`, 'yyyy-dd-MM')|
+----------------------------------+
|               2017-11-12 00:00:00|
+----------------------------------+
~~~
- 올바른 포맷과 타입의 날짜나 타임스탬프를 사용한다면 쉽게 비교 가능
- 날짜를 비교할 때는 날짜나 타임스탬프 타입을 사용하거나 yyyy-MM-dd 포맷에 맞는 문자열을 지정함
~~~scala
cleanDateDF.filter(col("date2") > lit("2017-12-12")).show()
~~~
- Spark가 리터럴로 인식하는 문자열을 지정해 날짜를 비교할 수도 있음
~~~scala
cleanDateDF.filter(col("date2") > "'2017-12-12'").show()
~~~
- 암시적 형변환은 매우 위험함  
  특히 null 값 또는 다른 시간대나 포맷을 가진 날짜 데이터를 다룰 때는 더 위험함
- 암시적 형변환에 의지하지 말고 명시적으로 데이터 타입을 변환해 사용하자  
  --> date format을 반드시 줘서 사용하자는 의미! 

### 6.7 null 값 다루기
- Spark에서는 빈 문자열이나 대체 값 대신 null값을 사용해야 최적화를 수행할 수 있음
- DataFrame의 하위 패키지인 <b>.na</b>를 사용하는 것이 DataFrame에서 null 값을 다루는 기본 방식
- 또한 연산을 수행하면서 스파크가 null값을 제어하는 방법을 명시적으로 지정하는 몇 가지 함수도 있음
- null 값을 허용하지 않는 컬럼을 선언해도 <b>강제성</b>은 없음  
  즉, null값을 허용하지 않는 컬럼에 null 값을 컬럼에 넣을 수 있다는 얘기
- 만약 null 값이 없어야 하는 컬럼에 null 값이 존재한다면, 
  부정확한 결과를 초래하거나 디버깅하기 어려운 오류를 만날 수 있음
- null 값을 다루는 방법은 두가지가 있음
  - 명시적으로 null 값을 제거
  - 전역 또는 컬럼 단위로 null 값을 특정 값으로 채워 넣기

#### 6.7.1 coalesce
- Spark의 `coalesce` 함수는 값이 있으면 해당 값을 반환하고, 값이 null이면 여러 인수 중
  값을 가지는 첫 번째 인수 값을 반환
- 아래 예시는 Description 컬럼의 값이 null인지 확인하여 null이면 CustomerId 값을 반환하고, 
null이 아니면 description 컬럼 값을 반환하는 예제
~~~scala
import org.apache.spark.sql.functions.coalesce
df.select(coalesce(col("Description"), col("CustomerId"))).show()
~~~

#### 6.7.2 ifnull, nullIf, nvl, nvl2
- `ifnull` : 첫 번째 값이 null이면 두 번째 값 반환
- `nullif` : 두 값이 같으면 null 반환, 두 값이 다르면 첫 번째 값 반환
- `nvl` : 첫 번째 값이 null이면 두 번째 값을 반환. 첫 번째 값이 null이 아니면 첫 번째 값 반환
- `nvl2` : 첫 번째 값이 null이 아니면 두 번째 값 반환. 첫 번째 값이 null이면 세 번째 인수로 저장된 값 반환
~~~SQL
SELECT
  ifnull(null, 'return_value'),
  nullIf('value', 'value'),
  nvl(null, 'return_value'),
  nvl2('not_null', 'return_value', 'else_value')
FROM dfTable LIMIT 1
~~~

#### 6.7.3 drop 
- null 값을 가진 로우를 제거하는 가장 간단한 함수
- null 값을 가지는 모든 row를 제거
~~~scala
df.na.drop()
df.na.drop("any")
~~~
- any를 지정한 경우는 로우의 컬럼값 중 하나라도 null 값을 가지면 해당 로우 제거
- all을 지정한 경우 모든 컬럼 값이 null/NaN인 경우에만 해당 로우 제거
~~~scala
df.na.drop("all")
~~~ 
- drop 메서드에 배열 형태의 컬럼을 인수로 전달해 적용할 수도 있음
~~~scala 
df.na.drop("all", Seq("StockCode", "InvoiceNo"))
~~~

#### 6.7.4 fill
- `fill` 함수를 사용해 하나 이상의 컬럼을 특정 값으로 채울 수 있음
- 채워 넣을 값, 컬럼 집합으로 구성된 Map을 인수로 사용
- 예를 들어 String, Integer 데이터 타입의 컬럼에 존재하는 null 값을 다른 값으로 채워 넣는 방법은 다음과 같음
~~~scala
df.na.fill("All Null values become this String") // String type
df.na.fill(5, Seq("StockCode", "InvoiceNo")) // Integer type
~~~
- <b>Scala Map 타입을 사용해 다수의 컬럼에 fill 메서드를 적용할 수도 있음</b>  
  여기서 Key는 컬럼명이며, value는 null값을 채우는데 사용할 값
~~~scala 
val fillColValues = Map("StockCode" -> 5, "Description" -> "No value")
df.na.fill(fillColValues)
~~~

#### 6.7.5 replace
- 조건에 따라 다른 값으로 대체하는 방법
- `replace` 메서드를 사용하려면 변경하고자 하는 값과 원래 값의 데이터 타입이 같아야 함
~~~Scala
df.na.replace("Description", Map("" -> "UNKNOWN"))
~~~

### 6.8 정렬하기
- 5장에서 설명한 것처럼 `asc_nulls_first`, `desc_nulls_first`, `asc_nulls_last`, `desc_nulls_last` 함수를 사용해 DataFrame을 정렬할 때 null 값이 표시되는 기준을 지정할 수 있음

### 6.9 복합 데이터 타입 다루기
- 복합 데이터 타입을 사용하면 해결하려는 문제에 더욱 적합한 방식으로 데이터를 구성하고 구조화할 수 있음
- 복합 데이터 타입에는 구조체(struct), 배열(array), 맵(map)이 있음

#### 6.9.1 구조체
- 구조체는 DataFrame 내부의 DataFrame으로 생각할 수 있음
- 쿼리문에서 다수의 컬럼을 괄호로 묶어 구조체를 만들 수 있음
~~~scala
df.selectExpr("(Description, InvoiceNo) as complex", "\*")
df.selectExpr("struct(Description, InvoiceNo) as complex", "\*")

import org.apache.spark.sql.functions.struct
val complexDF = df.select(struct("Description", "InvoiceNo").alias("complex"))
complexDF.createOrReplaceTempView("complexDF")
~~~
- 복합 데이터 타입을 가진 DataFrame을 만들어 보았음
- 이를 다른 DataFrame을 조회하는 것과 동일하게 사용할 수 있는데,  
  차이점은 문법에 (.)을 사용하거나 getField 메서드를 사용한다는 점
~~~scala
complexDF.select("complex.Description")
complexDF.select(col("complex").getField("Description"))
~~~
- (*) 문자를 사용해 모든 값을 조회할 수 있으며, 모든 컬럼을 DataFrame의 최상위 수준으로 끌어올릴 수 있음
~~~scala
complexDF.select("complex.*")
~~~

#### 6.9.2 배열
- 데이터에서 Description 컬럼의 모든 단어를 하나의 로우로 변환해보자
- Description 컬럼을 복합 데이터 타입인 배열로 변환

#### split
- 배열로 변환하려면 `split` 함수 사용. split 함수에 구분자를 인수로 전달해 배열로 변환
~~~scala
import org.apache.spark.sql.functions.split
df.select(split(col("Description"), " ")).show(2)
~~~
- `split` 함수는 Spark에서 복합 데이터 타입을 마치 또 다른 컬럼처럼 다룰 수 있는 강력한 기능 중 하나
~~~scala
df.select(split(col("Description"), " ")).alias("array_col")
.selectExpr("array_col[0]").show(2)
~~~

#### 배열의 길이
- `size` 함수를 이용해 배열의 길이를 알 수 있음
~~~scala
import org.apache.spark.sql.functions.sizes
df.select(size(split(col("Description"), " "))).show(2) // 5와 3 출력
~~~

#### array_contains
- `array_contains` 함수를 사용해 배열에 특정 값이 존재하는지 확인 가능
~~~scala
import org.apache.spark.sql.functions.array_contains
df.select(array_contains(split(col("Description"), " "),  "WHITE")).show(2)
// 실행 결과
+--------------------------------------------+
|array_contains(split(Description,  ), WHITE)|
+--------------------------------------------+
|                                        true|
|                                        true|
+--------------------------------------------+
~~~
- 복합 데이터 타입의 배열에 존재하는 모든 값을 로우로 변환하려면 `explode` 함수 사용

#### explode
- `explode` 함수는 배열 타입의 컬럼을 입력 받음. 그리고 입력된 컬럼의 배열값에  
  포함된 모든 값을 로우로 변환
- 나머지 컬럼값은 중복되어 표시
~~~
"Hello World", "other col" (split 함수 적용) --> ["Hello", "World"], "other col"
  (explode 함수 적용) --> "Hello", "other col"
                          "World", "other col"
~~~
~~~scala
import org.apache.spark.sql.functions.{split, explode}
df.withColumn("splitted", split(col("Description"), " "))
  .withColumn("exploded", explode(col("splitted")))
  .select("Description", "InvoiceNo", "exploded").show(2)

// 실행 결과
+--------------------+---------+--------+
|         Description|InvoiceNo|exploded|
+--------------------+---------+--------+
|WHITE HANGING HEA...|   536365|   WHITE|
|WHITE HANGING HEA...|   536365| HANGING|
+--------------------+---------+--------+
~~~

#### 6.9.3 맵
- 맵은 map 함수와 컬럼의 키-값 쌍을 이용해 생성
- 배열과 동일한 방법으로 값을 선택 가능
~~~scala
import org.apache.spark.sql.functions.map
df.select(map(col("Description"), col("InvoiceNo")).alias("complex_map")).show(2, false)
// 실행 결과
+----------------------------------------------+
|complex_map                                   |
+----------------------------------------------+
|[WHITE HANGING HEART T-LIGHT HOLDER -> 536365]|
|[WHITE METAL LANTERN -> 536365]               |
+----------------------------------------------+
~~~
- 적합한 key를 사용해 데이터를 조회할 수 있으며, 해당 키가 없다면 null 값 반환
~~~scala
df.select(map(col("Description"), col("InvoiceNo")).alias("complex_map"))
.selectExpr("complex_map['WHITE METAL LANTERN']").show(2)
// 결과
+--------------------------------+
|complex_map[WHITE METAL LANTERN]|
+--------------------------------+
|                            null|
|                          536365|
+--------------------------------+
~~~
- map 타입은 분해하여 컬럼으로 변환 가능
~~~scala
df.select(map(col("Description"), col("InvoiceNo")).alias("complex_map"))
.selectExpr("explode(complex_map)").show(2)
// 결과 
+--------------------+------+
|                 key| value|
+--------------------+------+
|WHITE HANGING HEA...|536365|
| WHITE METAL LANTERN|536365|
+--------------------+------+
~~~

### 6.10 JSON 다루기
- Spark는 JSON을 다루기 위한 몇 가지 고유 기능을 지원
- Spark에서는 문자열 형태의 JSON을 직접 조작할 수 있으며, JSON을 파싱하거나  
  객체로 만들 수 있음
- 다음은 JSON 컬럼을 생성하는 예제
~~~
val jsonDF = spark.range(1).selectExpr("""
'{"myJSONKey" : {"myJSONValue" : [1, 2, 3]}}' as jsonString""")
~~~
- `get_json_object` 함수로 JSON 객체(딕셔너리나 배열)를 인라인 쿼리로 조회할 수 있음  
  중첩이 없는 단일 수준의 JSON 객체라면 `json_tuple`을 사용할 수도 있음
~~~scala
import org.apache.spark.sql.functions.{get_json_object, json_tuple}
jsonDF.select(
  get_json_object(col("jsonString"), "$.myJSONKey.myJSONValue[1]") as "column",
  json_tuple(col("jsonString"), "myJSONKey")).show(2)
~~~
- `to_json` 함수를 사용해 StrucType을 JSON 문자열로 변경할 수 있음
~~~scala
import org.apache.spark.sql.functions.to_json
df.selectExpr("(InvoiceNo, Description) as myStruct")
.select(to_json("myStruct"))
~~~
- `to_json` 함수에 JSON 데이터소스와 동일한 형태의 딕셔너리(맵)을 파라미터로 사용할 수 있음
- `from_json` 함수를 사용해 JSON 문자열을 다시 객체로 변환 할 수 있음
- `from_json` 함수는 파라미터로 반드시 스키마를 지정해야 함. 필요에 따라 맵 데이터 타입의 옵션을 인수로 지정할 수 있음
~~~scala
import org.apache.spark.sql.functions.{to_json, from_json}
val parseSchema = StructType(Array(
  new StructField("InvoiceNo", StringType, true),
  new StructField("Description", StringType, true)))

df.selectExpr("(InvoiceNo, Description) as myStruct")
.select(to_json(col("myStruct")).alias("newJSON"))
.select(from_json(col("newJSON"), parseSchema), col("newJSON")).show(2)
~~~

### 6.11 사용자 정의 함수(User Define Function)
- Spark의 가장 강력한 기능 중 하나는 UDF를 사용할 수 있다는 점
- UDF는 파이썬이나 스칼라 그리고 외부 라이브러리를 사용해 사용자가 원하는 형태로 트랜스포메이션을 만들 수 있게 함
- UDF는 하나 이상의 컬럼을 입력으로 받고, 반환할 수 있음
- Spark UDF는 여러 가지 프로그래밍 언어로 개발할 수 있으므로 엄청나게 강력함
- UDF는 레코드별로 데이터를 처리하는 함수이기 때문에 독특한 포멧이나 도메인에 특화된 언어를 사용하지 않음
- UDF는 기본적으로 특정 sparkSession이나 Context에서 사용할 수 있도록 임시 함수 형태로 등록됨
- Scala, python, java로 UDF를 개발할 수 있음  
  하지만 언어별로 성능에 영향을 미칠 수 있으므로 주의해야 함
- 성능 차이를 설명하기 위하여 UDF를 생성해서 Spark에 등록하고, 생성된 UDF를 사용해  
  코드를 실행하는 과정에서 정확히 무슨 일이 발생하는지 알아보겠음
- 첫 번째로 실제 함수가 필요함  
  예제로 사용할 UDF를 만들어 보겠음. 숫자를 입력받아 세제곱 연산을 하는 power3 함수를 개발해보자
~~~scala
val udfExampleDF = spark.range(5).toDF("num")
def power3(number:Double):Double = number * number * number
power3(2.0)
~~~
- 이제 함수를 만들었으므로 모든 워커 노드에서 생성된 함수를 사용할 수 있도록 Spark에 등록할 차례임
- <b>Spark는 드라이버에서 함수를 직렬화하고 네트워크를 통해 모든 익스큐터 프로세스로 전달</b>  
이 과정은 언어와 관계없이 발생함
- 함수를 개발한 언어에 따라서 근본적으로 동작하는 방식이 달라짐

#### Scala, Java UDF
- Scala나 Java로 함수를 작성했다면 JVM 환경에서만 사용할 수 있음  
  따라서 Spark 내장 함수가 제공하는 코드 생성 기능의 장점을 활용할 수 없어 약간의 성능 저하가 발생함

 #### 파이썬 UDF 
- 파이썬으로 함수를 작성했다면 매우 다르게 작동함  
  Spark는 워커 노드에 파이썬 프로세스를 실행하고 파이썬이 이해할 수 있는 포맷으로  
  모든 데이터를 직렬화함
- 그리고 파이썬 프로세스에 있는 데이터의 로우마다 함수를 실행하고  
  마지막으로 JVM과 Spark에 처리 결과를 반환
~~~
// 파이썬 UDF 처리 과정
                                      -------> 익스큐터 프로세스 
                                                     |  ^
                                    |                v  |
                                    |         워커 파이썬 프로세스
드라이버                            | 
--------------------------          |                           익스큐터 프로세스 
| 스칼라 UDF |  spark    |--(함수 직렬화 후 워커에 전달)-->         |    ^
| 파이썬 UDF |  session  |                                          v    |
--------------------------                                     워커 파이썬 프로세스
~~~
#### 파이썬 UDF 실행시 문제점
- 파이썬 프로세스를 시작하는 부하도 크지만, 진짜 부하는 파이썬으로  
  데이터를 전달하기 위해 직렬화 하는 과정에서 발생
- 이 특성은 두 가지 문제점을 발생
  - 직렬화에 큰 부하가 발생
  - 데이터가 파이썬으로 전달되면 스파크에서 워커 메모리를 관리할 수 없음
- 그러므로 JVM과 파이썬이 동일한 머신에서 메모리 경합을 하면 자원에 제약이 생겨  
  워커가 비정상적으로 종료될 가능성이 있음
- 결과적으로 Java나 Scala로 함수를 개발하는 것이 좋음
- UDF를 실행해보자. 먼저 DataFrame에서 사용할 수 있도록 함수를 등록
~~~scala
import org.apache.spark.sql.functions.udf
val power3udf = udf(power3(_:Double):Double)
~~~
- 이제 power3 함수를 DataFrame의 다른 함수와 동일한 방법으로 사용할 수 있음
~~~scala
udfExampleDF.select(power3df(col("num"))).show()
~~~
- 아직까지는 UDF를 DataFrame에서만 사용할 수 있고 문자열 표현식(expr)에서는 사용할 수 없음
- UDF를 spark SQL 함수로 등록하면 모든 프로그래밍 언어와 SQL에서 UDF를 사용할 수 있음
~~~scala
spark.udf.register("power3", power3(_:Double):Double)
udfExampleDF.selectExpr("power3(num)").show(2)
~~~
- 스칼라로 제작한 UDF를 Spark SQL 함수로 등록했기 때문에  
  python에서 SQL 표현식을 이용해 해당 함수를 사용 가능
- 파이썬 함수를 SQL 함수로 등록할 수 있음. 즉 다른 언어에서도 동일한 방법으로 사용할 수 있음
- Spark는 파이썬의 데이터 타입과는 다른 자체 데이터 타입을 사용하기 때문에  
  함수를 정의할 때 반환 타입을 지정하는 것이 좋음
- 함수에서 반환될 실제 데이터 타입과 일치하지 않는 데이터 타입을 지정하면  
  <b>Spark는 오류가 아닌 null값을 발생시킴</b>
- 다음 예제와 같이 함수의 반환 데이터 타입을 DoubleType으로 변경하면 이러한 현상 확인 가능
~~~python
// 파이썬 코드
from pyspark.sql.types import IntegerType, DoubleType
spark.udf.register("power3py", power3, DoubleType())
udfExampleDF.selectExpr("power3py(num)").show(2)
~~~
- null값은 반환하는 이유는 range 메서드가 Integer 데이터 타입의 데이터를 만들기 때문
- 이 문제를 해결하려면 파이썬 함수가 Integer 데이터 타입 대신 Float 데이터 타입을 반환하도록 수정해야 함

### 6.12 Hive UDF
- 하이브 문법을 사용해서 만든 UDF/UDAF도 사용할 수 있음
- 이렇게 하려면 SparkSession을 생성할 때 `SparkSession.builder().enableHiveSupport()`를 명시해 하이브 지원 기능을 활성화해야 함
- 하이브 지원이 활성화되면 SQL로 UDF로 등록 가능
- 사전에 컴파일된 스칼라와 자바 패키지에서만 지원되므로 라이브러리 의존성을 명시해야 함


### 용어 정리
- 인라인 쿼리
