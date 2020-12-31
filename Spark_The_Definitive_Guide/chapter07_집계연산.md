## 집계 연산
- 집계를 수행하려면 키나 그룹을 지정하고 하나 이상의 컬럼을 변환하는 방법을 지정하는  
  집계 함수를 사용
- Spark의 집계 능력은 다양한 활용 사례와 가능성으로 비추어 보았을 때 매우 정교하며 충분히 발달해 있음
- Spark는 모든 데이터 타입을 다루는 것 외에도 다음과 같은 <b>그룹화 데이터 타입</b>을 생성할 수 있음
  - 가장 간단한 형태의 그룹화는 select 구문에서 집계를 수행해 DataFrame의 전체 데이터를 요약하는 것
  - `group by`는 하나 이상의 키를 지정할 수 있으며 값을 가진 컬럼을 변환하기 위해 다른 집계 함수 사용 가능
  - `window`는 하나 이상의 키를 지정할 수 있으며 값을 가진 컬럼을 변환하기 위해 다른 집계 함수 사용 가능  
    함수의 입력으로 사용할 로우는 현재 로우와 어느 정도 연관성이 있어야 함
  - `grouping set`은 서로 다른 레벨의 값을 집계할 때 사용. SQL, DataFrame의 롤업 그리고 큐브를 사용할 수 있음
  - `rollup`은 하나 이상의 키를 지정할 수 있음. 그리고 컬럼을 변환하는데 다른 집계 함수를 사용하여  
    계층적으로 요약된 값을 구할 수 있음
  - `cube`는 하나 이상의 키를 지정할 수 있으며 값을 가진 컬럼을 변환하기 위해 다른 집계 함수를 사용 할 수 있음. 큐브는 모든 컬럼 조합에 대한 요약 값 계산
- 지정된 집계 함수에 따라 그룹화된 결과는 `RelationalGroupedDataset`을 반환
- BigData를 사용해 연산을 수행하는 경우, 질문에 대한 정확한 답을 얻기 위해서는 연산, 네트워크, 저장소 등 상당한 비용이 들 수밖에 없음
- 그러므로 수용 가능한 정도의 정확도에 맞춰 근사치를 계산하는 것이 비용을 고려했을 때 더 효율적
- 근사치 계산용 함수를 사용해 스파크 잡의 실행과 속도를 개선할 수 있음  
  특히 대화형 셸을 이용해 비정형 분석을 수행하는 경우에 유용
- 구매 이력 데이터를 사용해 파티션을 훨씬 적은 수로 분할할 수 있도록 리파티셔닝하고 빠르게 접근할 수 있도록 캐싱하겠음
- 파티션 수를 줄이는 이유는 적은 양의 데이터를 가진 수많은 파일이 존재하기 때문
~~~scala
val df = spark.read.format("csv")
.option("header", "true")
.option("inferschema", "true")
.load("C:/Spark-The-Definitive-Guide-master/data/retail-data/all/*.csv")
.coalesce(5)

df.cache()
df.createOrReplaceTempView("dfTable")
~~~
- 다음은 `count` 메서드를 사용한 간단한 예제
~~~scala
df.count() == 541909
~~~
- `count` 메서드가 프랜스포메이션이 아닌 액션임을 다시 상기하자  
  그러므로 결과를 즉시 반환
- 위 예제처럼 `count` 메서드는 데이터셋의 전체 크기를 알아보는 용도로 사용되지만  
  메모리에 DataFrame 캐싱 작업을 수행하는 용도로 사용되기도 함
- 지금은 `count` 메서드가 조금 이질적으로 보일 수 있는데, 그 이유는  
  함수가 아니라 메서드 형태로 존재하고 트랜스포메이션처럼 지연 연산 방식이 아닌  
  즉시 연산을 수행하기 때문
- 다음 절에서는 지연 연산 방식으로 `count` 메서드를 사용하는 방법을 알아보자

### 7.1 집계 함수
- 모든 집계는 6장에서 사용한 DataFrame의 .stat 속성을 이용하는 특별한 경우를 제외한다면 함수를 사용
- 집계 함수는 org.apache.spark.sql.functions 패키지에서 찾아볼 수 있음

#### 7.1.1 count
- 다음 예제에서 `count` 함수는 엑션이 아닌 트랜스포메이션으로 동작함
- `count` 함수는 두 가지 방식으로 사용 가능
  - `count` 함수에 특정 컬럼을 지정하는 방식(트랜스포메이션)
  - `count` 함수에 *나 1를 사용하는 방식
- 다음 예제와 같이 `count` 함수를 이용해 전체 row 수를 카운트 할 수 있음
~~~
import org.apache.spark.sql.functions.count
df.select(count("StockCode")).show()
~~~
#### 주의!
- count(*) 를 사용하면 null값을 포함하여 카운트하고, count("columnName") 형식으로 사용하면 null 값을 카운트 하지 않음

#### 7.1.2 countDistinct
- 전체 레코드 수가 아닌 개별 레코드 수를 카운트할 때 사용
- 개별 컬럼을 처리하는데 적합
~~~
import org.apache.spark.sql.functions.countDistinct
df.select(countDistinct("StockCode")).show()
~~~

#### 7.1.3 approx_count_distinct
- 대규모 데이터셋을 다루다 보면 정확한 고유 개수가 무의미 할 때도 있음
- 어느 정도 수준의 근사치만으로도 유의미하다면 `approx_count_distinct` 함수를 사용해 근사치 계산 가능
~~~scala
import org.apache.spark.sql.functions.approx_count_distinct
df.select(approx_count_distinct("StockCode", 0.1)).show()
~~~
- `approx_count_distinct` 함수는 최대 추정 오류율 이라는 한가지 파라미터를 사용
- 예제에서는 큰 오류율을 설정했기 때문에 기대치에서 크게 벗어나는 결과를 얻게 되지만,  
  `countDistinct` 함수보다 더 빠르게 결과 반환
- 이 함수의 성능은 대규모 데이터셋을 사용할 때 훨씬 더 좋아짐

#### 7.1.4 first와 last
- 함수명에서도 알 수 있듯이 `first`와 `last` 함수는 DataFrame의 첫 번째 값이나  
  마지막 값을 얻을 때 사용
- 이들 함수는 DataFrame의 값이 아닌 로우를 기반으로 동작
~~~scala
import org.apache.spark.sql.functions.{first, last}
df.select(first("StockCode"), last("StockCode")).show()
~~~

#### 7.1.5 min과 max
- DataFrame에서 최솟값과 최댓값을 추출하려면 `min`과 `max` 함수 사용
~~~scala
import org.apache.spark.sql.functions.{min, max}
df.select(min("Quality"), max("Quailty")).show()
~~~

#### 7.1.6 sum
- DataFrame 에서 특정 컬럼의 모든 값을 합산 하려면 sum 함수 사용
~~~scala
import org.apache.spark.sql.functions.sum
df.select(sum("Quantity")).show()
~~~

#### 7.1.7 sumDistinct
- 고윳값 합산
~~~scala
import org.apache.spark.sql.functions.sumDistinct
df.select(sumDistinct("Quantity")).show()
~~~

#### 7.1.8 avg
- Spark의 avg 함수나 mean 함수를 사용하면 평균값을 더 쉽게 구할 수 있음
- 다음 예제는 집계된 컬럼을 재활용하기 위해 `alias` 메서드를 사용함

~~~scala
import org.apache.spark.sql.functions.{sum, count, avg, expr}
df.select(
    count("Quantity").alias("total_transactions"),
    sum("Quantity").alias("total_purchases"),
    avg("Quantity").alias("avg_purchases"),
    expr("mean(Quantity)").alias("mean_purchases")
).selectExpr(
    "total_purchases/total_transactions", 
    "avg_purchases",
    "mean_purchases").show()
~~~ 

#### 7.1.9 분산과 표준편차
- Spark에서는 함수를 사용해 분산과 표준편차를 계산할 수 있음
- Spark는 표본표준편차(sample standard deviation)뿐만 아니라 모표준편차(population) 방식도 지원하기 때문에 주의가 필요함
- 이 둘은 완전히 다른 통계 방식이기 때문에 구분해서 사용해야 함
- `variance` 함수나 `stddev` 함수를 사용한다면 기본적으로 표본표준분산과 표본표준편차 공식을 이용함  
- 모표준분산이나 모표준편차 방식을 사용하려면 다음 예제와 같이 `var_pop` 함수나 `stddev_pop` 함수를 사용
- 모표준편차와 표본표준편차의 차이는 분모에 N-1로 나누느냐, N으로 나누느냐의 차이 
~~~scala
import org.apache.spark.sql.functions.{var_pop, stddev_pop}
import org.apache.spark.sql.functions.{var_samp, stddev_samp}

df.select(var_pop("Quantity"), var_samp("Quantity"),
          stddev_pop("Quantity"), stddev_samp("Quantity")).show()
~~~

#### 7.1.10 비대칭도와 첨도
- 비대칭도(skewness)와 첨도(kurtosis) 모두 데이터의 변곡점을 측정하는 방법
- 비대칭도는 데이터 평균의 비대칭 정도를 측정하고, 첨도는 데이터 끝 부분을 측정함
- 비대칭도와 첨도는 특히 확률 변수와 확률분포로 데이터를 모델링 할 때 특히 중요함
~~~
import org.apache.spark.sql.functions.{skewness, kurtosis}
df.select(skewness("Quantity"), kurtosis("Quantity")).show()
~~~   

#### 7.1.11 공분산과 상관관계 
- `cov` 함수와 `corr` 함수를 사용해 공분산(covariance)과 상관관계(correlation)을 계산 할 수 있음
- 공분산은 데이터 입력값에 따라 다른 범위를 가짐
- 상관관계는 피어슨 상관계수를 측정하며 -1과 1사이의 값을 가짐
- `var` 함수처럼 표본공분산(sample covariance) 방식이나 모공분산(population covariance) 방식으로 공분산을 계산할 수도 있음  
그러므로 사용하고자 하는 방식을 명확하게 지정하는 것이 좋음
- 상관관계는 모집단이나 표본집단에 대한 개념이 없음
~~~scala
import org.apache.spark.sql.functions.{corr, covar_pop, covar_samp}
df.select(corr("Quality"), covar_pop("Quality"), covar_samp("Quality")).show()
~~~

#### 7.1.12 복합 데이터 타입의 집계
- Spark는 복합 데이터 타입을 사용해 집계를 수행할 수 있음
- 예를 들어 특정 컬럼의 값을 리스트로 수집하거나, 셋 데이터 타입으로 고윳값만 수집할 수 있음
- 수집된 데이터는 처리 파이프라인에서 다양한 프로그래밍 방식으로 다루거나 사용자 정의 함수를 사용해 전체 데이터에 접근 할 수 있음
~~~scala
import org.apache.spark.sql.functions.{collect_set, collect_list}
df.agg(collect_set("Country"), collect_list("Country")).show()
~~~

### 7.2 그룹화
- 지금까지는 DataFrame 수준의 집계만 다뤘는데, 데이터 그룹 기반의 집계를 수행하는 경우가 더 많음
- 예를들어 고유한 송장번호(InvoiceNo)를 기준으로 그룹을 만들고 그룹별 물품 수 카운트를 하면 또 다른 DataFrame을 반환하며 지연 처리방식으로 수행됨
- 그룹화 작업은 하나 이상의 컬럼을 그룹화하고 집계 연산을 수행하는 두 단계로 이루어짐
- 첫 번째 단계에서는 `RelationalGroupedDataset`이 반환되고, 두 번째 단계에서는  
  DataFrame이 반환됨
- 그룹의 기준이 되는 컬럼은 여러 개 지정 가능
~~~scala
df.groupby("InvoiceNo", "CustomerId").count().show()
~~~

#### 7.2.1 표현식을 이용한 그룹화
- 카운팅은 메서드로 사용할 수 있으므로 특별함. 하지만 메서드 대신 `count` 함수를 사용할 것을 추천
- 또한 `count` 함수를 select 구문에 표현식으로 지정하는 것 보다 `agg` 메서드를 사용하는 것이 좋음
- `agg` 메서드는 여러 집계 처리를 한 번에 지정할 수 있으며, 집계에 표현식을 사용할 수 있음
- 또한 트랜스포메이션이 완료된 컬럼에 `alias` 메서드를 적용할 수 있음
~~~scala
import org.apache.spark.sql.functions.count
df.groupBy("InvoiceNo").agg(
  count("Quantity").alias("quan"),
  expr("count(Quantity)")).show()
~~~

#### 7.2.2 맵을 이용한 그룹화
- 컬럼을 키로, 수행할 집계 함수의 문자열을 값으로 하는 맵(Map) 타입을 사용해  
  트랜스포메이션을 정의할 수 있음
- 수행할 집계 함수를 한 줄로 작성하면 여러 컬럼명을 재사용할 수 있음
~~~scala
df.groupBy("InvoiceNo").agg("Quantity" -> "avg", "Quantity" -> "stddev_pop").show()
~~~

### 7.3 윈도우 함수
- 윈도우 함수를 집계에 사용할 수도 있음
- 윈도우 함수는 데이터의 특정 윈도우를 대상으로 고유의 집계 연산을 수행
- 데이터의 윈도우는 현재 데이터에 대한 참조를 사용해 정의함
- 윈도우 명세(window specification)은 함수에 전달될 로우를 결정함
- 표준 group-by 함수와 유사해 보일수도 있으므로 이 둘의 차이점을 알아보자

#### 표준 group-by vs window
- group-by 함수를 사용하면 모든 로우 레코드가 단일 그룹으로만 이동
- 윈도우 함수는 프레임에 입력되는 모든 로우에 대해 결괏값 계산 프레임은 로우 그룹 기반의 테이블을 의미
- 윈도우 함수는 프레임에 입력되는 모든 로우에 대해 결괏값을 계산함
- 프레임은 로우 그룹 기반의 테이블을 의미함
- 각 로우는 하나 이상의 프레임에 할당될 수 있음
- 가장 흔하게 사용되는 방법 중 하나는 하루를 나타내는 값의 롤링 평균(rolling average)를 구하는 것
- 이 작업을 수행하려면 개별 로우가 7개의 다른 프레임으로 구성되어 있어야 함
- Spark는 3가지 종류의 윈도우 함수를 지원함
  - 랭크 함수(rank function)
  - 분석 함수(analytic function)
  - 집계 함수(aggregate function)
- 예제를 위해 주문 일자(InvoiceDate) 컬럼을 변환해 'date' 컬럼을 만듬. 이 컬럼은 시간 정보를 제외한 날짜 정보만 가짐
~~~scala
import org.apache.spark.sql.functions.{col, to_date}
val dfWithDate = df.withColumn("date", to_date(col("InvoiceDate"), "MM/d/yyyy H:mm"))
dfWithDate.createOrReplaceTempView("dfWithDate")
~~~
- 윈도우 함수를 정의하기 위해 첫 번째 단계로 윈도우 명세(window specification)를 만듬
- 여기서 사용하는 `partitionBy` 메서드는 지금까지 사용해온 파티셔닝 스키마의 개념과는 관련이 없으며 그룹을 어떻게 나눌지 결정하는 것과 유사한 개념
- `orderBy` 메서드는 파티션의 정렬 방식을 정의함
- 프레임 명세(rowBeteen 구문)는 입력된 로우의 참조를 기반으로 프레임에 로우가 포함될 수 있는지 결정함
- 다음 예제는 첫 로우부터 현재 로우까지 확인함
~~~scala
import org.apache.spark.sql.expressions.Window
import org.apache.spark.sql.functions.col

val windowSpec = Window
.partitionBy("CustomerId", "date")
.orderBy(col("Quantity").desc)
.rowsBetween(Window.unboundedPreceding, Window.currentRow)
~~~
- 이제 집계 함수를 사용해 고객을 좀 더 자세히 알아보자  
여기서는 시간대별 최대 구매 개수를 구하는 예를 들어보자
- 위 예제에서 사용한 집계 함수에 컬럼명이나 표현식을 전달해야 함
- 이 함수를 적용할 데이터 프레임이 정의된 윈도우 명세도 함께 사용
~~~scala
import org.apache.spark.sql.functions.max
val maxPurchaseQuantity = max(col("Quantity")).over(windowSpec)
~~~ 
- 앞 예제는 컬럼이나 표현식을 반환하므로 DataFrame의 select 구문에서 사용 가능
- 먼저 구매량 순위를 만들어보자  
  `dense_rank` 함수를 사용해 모든 고객에 대해 최대 구매 수량을 가진 날짜가 언제인지 알아보자
- 동일한 값이 나오거나 중복 로우가 발생해 순위가 비어 있을 수 있으므로,  
  `rank` 함수 대신 `dense_rank` 함수를 사용
~~~scala
import org.apache.spark.sql.functions.{dense_rank, rank}

val purchaseDenseRank = dense_rank().over(windowSpec)
val purchaseRank = rank().over(windowSpec)
~~~
- 이 예제 또한 select 구문에서 사용할 수 있는 컬럼 반환  
  (하단 예제를 보면 이해가 됨)
- 이제 `select` 메서드를 이용해 계산된 윈도우 값을 확인해보자
~~~scala
import org.apache.spark.sql.functions.col
dfWithDate.where("CustomerId IS NOT NULL").orderBy("CustomerId")
.select(
  col("CustomerId"),
  col("date"),
  col("Quantity"),
  purchaseRank.alias("quantityRank"),
  purchaseDenseRank.alias("purchaseDenseRank"),
  maxPurchaseQuantity.alias("maxPurchaseQuantity")).show()
~~~
~~~scala
// 결과
+----------+----------+--------+------------+-----------------+-------------------+
|CustomerId|      date|Quantity|quantityRank|purchaseDenseRank|maxPurchaseQuantity|
+----------+----------+--------+------------+-----------------+-------------------+
|     12346|2011-01-18|   74215|           1|                1|              74215|
|     12346|2011-01-18|  -74215|           2|                2|              74215|
|     12347|2010-12-07|      36|           1|                1|                 36|
|     12347|2010-12-07|      30|           2|                2|                 36|
|     12347|2010-12-07|      24|           3|                3|                 36|
|     12347|2010-12-07|      12|           4|                4|                 36|
|     12347|2010-12-07|      12|           4|                4|                 36|
|     12347|2010-12-07|      12|           4|                4|                 36|
|     12347|2010-12-07|      12|           4|                4|                 36|
|     12347|2010-12-07|      12|           4|                4|                 36|
|     12347|2010-12-07|      12|           4|                4|                 36|
|     12347|2010-12-07|      12|           4|                4|                 36|
|     12347|2010-12-07|      12|           4|                4|                 36|
|     12347|2010-12-07|      12|           4|                4|                 36|
|     12347|2010-12-07|      12|           4|                4|                 36|
|     12347|2010-12-07|      12|           4|                4|                 36|
|     12347|2010-12-07|      12|           4|                4|                 36|
|     12347|2010-12-07|      12|           4|                4|                 36|
|     12347|2010-12-07|       6|          17|                5|                 36|
|     12347|2010-12-07|       6|          17|                5|                 36|
+----------+----------+--------+------------+-----------------+-------------------+
~~~

### 7.4 그룹화 셋
- 여러 그룹에 걸쳐 집계할 수 있는 무언가가 필요할 수 있음  
  <b>그룹화 셋</b>이 그 역할을 수행할 수 있음
- 그룹화 셋은 여러 집계를 수행하는 저수준 기능
- 그룹화 셋을 이용하면 group-by 구문에서 원하는 형태로 집계를 생성할 수 있음
- 다음 예제서 재고코드(stockCode)와 고객(CustomerId)별 총 수량을 얻기 위해  
  다음과 같은 SQL 표현식을 사용 함
~~~scala
val dfNoNull = dfWithDate.drop()
dfNoNull.createOrReplaceTempView("dfNoNull")

// SQL
SELECT CustomerId, stockCode, sum(Quantity) FROM dfNoNull
GROUP BY CustomerId, stockCode
ORDER BY CustomerId DESC, stockCode DESC
~~~
- 그룹화 셋을 사용해 동일한 작업 수행 가능
~~~scala
SELECT CustomerId, stockCode, sum(Quantity) FROM dfNoNull
GROUP BY CustomerId, stockCode GROUPING SETS((CustomerId, stockCode))
ORDER BY CustomerId DESC, stockCode DESC
~~~
- 그룹화 셋은 null 값에 따라 집계 수준이 달라짐. null 값을 제거하지 않았다면  
  부정확한 결괏값을 얻게 됨. 이 규칙은 큐브, 롤업 그리고 그룹화 셋에 적용됨
- 고객이나 재고 코드에 상관없이 총 수량의 합산 결과를 추가하려하면  
  `group-by` 구문을 사용해 처리하는 것을 불가능한데, 그룹화 셋을 이용하면 가능함
- `GROUPING SETS` 구문에 집계 방식을 지정하면 됨. 이 과정은 여러 개의 그룹을 하나로 묶으면 됨
~~~scala
SELECT CustomerId, stockCode, sum(Quantity) FROM dfNoNull
GROUP BY CustomerId, stockCode GROUPING SETS((CustomerId, stockCode), ())
ORDER BY sum(Quantity) DESC, CustomerId DESC, stockCode DESC
~~~
- `GROUPING SETS` 구문은 SQL에서만 사용 가능. DataFrame에서 동일한 연산을 수행하려면  
  `rollup` 메서드와 `cube` 메서드를 사용

#### 7.4.1 롤업
- 지금까지는 명시적 그룹화에 대해서 알아봤는데, 다양한 컬럼을 그룹화 키로 설정하면  
  그룹화 키로 설정된 조합뿐만 아니라 데이터셋에서 볼 수 있는 실제 조합을 모두 살펴 볼 수 있음  
- 롤업은 group-by 스타일의 다양한 연산을 수행할 수 있는 다차원 집계 기능
- 다음 예제에서는 시간(신규 Date 컬럼)과 공간(Country 컬럼)을 축으로 하는 롤업을 생성함
- 롤업의 결과로 생성된 DataFrame은 모든 날짜의 총합, 날짜별 총합, 날짜별 국가별 총합을 포함
~~~scala
val rolledUpDF = dfNoNull.rollup("Date", "Country").agg(sum("Quantity"))
.selectExpr("Date", "Country", "`sum(Quantity)` as total_Quantity")
.orderBy("Date")
rolledUpDF.show()
~~~
- null 값을 가진 로우에서 전체 날짜의 총 합계를 확인 할수 있음
- 롤업된 두 개의 컬럼값이 모두 null인 로우는 두 컬럼에 속한 레코드의 전체 합계를 나타냄
~~~scala
rolledUpDF.where("Country IS NULL").show()
~~~

#### 7.4.2 큐브
- 큐브는 롤업을 고차원적으로 사용할 수 있게 해줌
- 큐브는 요소들을 계층적으로 다루는 대신 모든 차원에 대해 동일한 작업을 수행
- 즉, 전체 기간에 대해 날짜와 국가별 결과를 얻을 수 있음
- 기존 기능만으로 다음과 같은 정보를 가진 테이블을 만들 수 있을까? 
  - 전체 날짜와 모든 국가에 대한 합계
  - 모든 국가의 날짜별 합계
  - 전체 날짜의 국가별 합계
  - 날짜별 국가별 합계
- 쉽게 말하면, group_by("Date", "Country") + group_by("Date") + group_by("Country") 기능을 모두 결합한 내용임!
- 메소드 호출 방식은 롤업과 매우 유사하며 `rollup` 메서드 대신 `cube` 메서드 호출
~~~scala
dfNoNull.cube("Date", "Country").agg(sum(col("Quantity")))
.select("Date", "Country", "sum(Quantity)").orderBy("Date").show()
~~~

#### 7.4.3 그룹화 메타데이터
- 큐브와 롤업을 사용하다 보면 집계 수준에 따라 쉽게 필터링하기 위해 집계 수준을 조회하는 경우가 발생. 이때 `grouping_id`를 사용
- `grouping_id`는 결과 데이터셋의 집계 수준을 명시하는 컬럼 제공
- 예제의 쿼리는 다음과 같은 네 개의 개별 그룹화 ID값을 반환

#### 그룹화 ID의 의미
- 그룹화ID --> 3
  - 가장 높은 계층의 집계 결과에서 나타남  
    `customerId`나 `stockCode`에 관계없이 총 수량을 제공
- 그룹화ID --> 2
  - 개별 재고 코드의 모든 집계 결과에서 나타남  
  - customerId에 관계없이 재고 코드별 총 수량을 제공함
- 그룹화ID --> 1
  - 구매한 물품에 관계없이 customerId를 기반으로 총 수량을 제공
- 그룹화ID --> 0
  - customerId와 stockCode별 조합에 따라 총 수량 제공
~~~scala
import org.apache.spark.sql.functions.{grouping_id, sum, expr}
dfNoNull.cube("customerId", "stockCode").agg(grouping_id(), sum("Quantity"))
.orderBy(col("grouping_id()").desc)
.show()
~~~

#### 7.4.4 피벗
- 피벗을 사용해 로우를 컬럼으로 변환할 수 있음
- 현재 데이터셋에는 Country 컬럼이 있는데, 피벗을 사용해 국가별로 집계 함수를 적용할 수 있으며 쿼리를 사용해 쉽게 결과를 확인할 수 있음
~~~scala
val pivoted = dfWithDate.groupBy("date").pivot("Country").sum()
~~~
- DataFrame은 국가명, 수치형 변수 그리고 날짜를 나타내는 컬럼을 조합한 컬럼을 가짐
- USA와 관련된 컬럼을 살펴보면 `USA_sum(Quantity)`, `USA_sum(UnitPrice)` 그리고 `USA_sum(CustomerID)`가 있음
- 집계를 수행했기 때문에 수치형 컬럼으로 나타남
~~~scala
pivoted.where("date > '2011-12-05'").select("date", "`USA_sum(Quantity)`").show()

// 결과
+----------+-----------------+
|      date|USA_sum(Quantity)|
+----------+-----------------+
|2011-12-06|             null|
|2011-12-09|             null|
|2011-12-08|             -196|
|2011-12-07|             null|
+----------+-----------------+
~~~
- 이제 컬럼의 모든 값을 단일 그룹화해서 계산할 수 있음. 하지만 데이터를 탐색하는 방식에 따라 피벗을 수행한 결괏값이 감소할 수도 있음
- 특정 컬럼의 카디널리티가 낮다면 스키마와 쿼리 대상을 확인할 수 있도록 피벗을 사용해 다수의 컬럼으로 변환하는 것이 좋음

### 7.5 사용자 정의 집계 함수
- 사용자 정의 집계 함수(user-defined aggregation function, UDAF)는 직접 제작한 함수나 비즈니스 규칙에 기반을 둔 자체 집계 함수를 정의하는 방법
- UDAF를 사용해서 입력 데이터 그룹에 직접 개발한 연산을 수행할 수 있음
- Spark는 입력 데이터의 모든 그룹의 중간 결과를 단일 `AggregationBuffer`에 저장해 관리  
 UDAF를 생성하려면 기본 클래스인 `UserDefinedAggregateFunction`을 상속 받고 다음과 같은 메서드를 정의해야 함
  - inputSchema : UDAF 입력 파라미터의 스키마를 StructType으로 정의
  - bufferSchema : UDAF 중간 결과의 스키마를 StructType으로 정의
  - dataType : 반환될 값의 DataType을 정의
  - deterministic : UDAF가 동일한 입력값에 대해 항상 동일한 결과를 반환한는지 불리언값으로 확인
  - initialize : 집계용 버퍼의 값을 초기화하는 로직을 정의
  - update : 입력받은 로우를 기반으로 내부 버퍼를 업데이트하는 로직을 정의
  - merge : 두 개의 집계용 버퍼를 병합하는 로직을 정의
  - evaluate : 집계의 최종 결과를 생성하는 로직을 정의

- 다음 예제는 입력된 모든 로우의 컬럼이 true인지 아닌지를 판단하는 BoolAnd 클래스를 구현함
- 만약 하나의 컬럼이라도 true가 아니라면 false 반환
~~~scala
import org.apache.spark.sql.expressions.MutableAggregationBuffer
import org.apache.spark.sql.expressions.UserDefinedAggregateFunction
import org.apache.spark.sql.Row
import org.apache.spark.sql.types._

class BoolAnd extends UserDefinedAggregateFunction {
  def inputSchema: org.apache.spark.sql.types.StructType = 
    StructType(StructField("value", BooleanType) :: Nil)
  
  def bufferSchema: StructType = StructType(
    StructField("result", BooleanType) :: Nil)
  
  def dataType:DataType = BooleanType
  
  def deterministic: Boolean = true
  
  def initialize(buffer: MutableAggregationBuffer): Unit = {
    buffer(0) = true
  }
  
  def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
    buffer1(0) = buffer1.getAs[Boolean](0) && input.getAs[Boolean](0)
  }
  
  def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
    buffer1(0) = buffer1.getAs[Boolean], buffer2.getAs[Boolean](0)
  }
  
  def evaluate(buffer: Row): Any = {
    buffer(0)
  }
  }
~~~
- 이제 간단히 클래스를 초기화하고 함수로 등록함
~~~scala
// scala 코드
val ba = new BoolAnd
spark.udf.register("booland", ba)
import org.apache.spark.sql.functions._

spark.range(1)
.selectExpr("explode(array(TRUE, TRUE, TRUE)) as t")
.selectExpr("explode(array(TRUE, FALSE, TRUE)) as f", "t")
// 결과
+-----+----+
|    f|   t|
+-----+----+
| true|true|
|false|true|
| true|true|
| true|true|
|false|true|
| true|true|
| true|true|
|false|true|
| true|true|
+-----+----+


spark.range(1)
.selectExpr("explode(array(TRUE, TRUE, TRUE)) as t")
.selectExpr("explode(array(TRUE, FALSE, TRUE)) as f", "t")
.select(ba(col("t")), expr("booland(f)"))
.show()
~~~
- 2.3 버전 기준 UDAF는 현재 스칼라와 자바로만 사용할 수 있고 HIVE UDF에서 알아본 것처럼 함수로 등록할 수 있음
