# Spark 관련 꼭 암기해야할 사항들
## theory
### chapter02
- Spark Application은 SparkSession이라는 driver process를 통해 제어
- 하나의 SparkSession은 하나의 Spark Application에 대응
- Scala와 python 콘솔을 실행하면 `spark` 변수로 sparkSession을 사용할 수 있음
- 클러스터 매니저는 물리적 머신을 관리하고, Spark application에 자원을 할당
- DataFrame에서 컬럼명과 컬럼타입을 정의한 것을 스키마(schema)라고 함
- spark는 익스큐터가 병렬로 데이터 처리를 할 수 있도록 파티션이라고 하는 청크 단위로 데이터를 분할함
- 파티션은 클러스터의 물리적 머신에 존재하는 로우의 집합
- 파티션은 데이터가 컴퓨터 클러스터에서 물리적으로 분산되는 방식을 나타냄
- DataFrame을 변경하려면 변경 방법을 Spark에게 알려주어야 하는데, 이때 변경하고자 하는 명령을 트랜스포메이션이라고 함
- 액션을 호출하지 않으면 트랜스포메이션을 수행되지 않음
- 트랜스포메이션은 좁은 의존성(narrow dependency)와 넓은 의존성(wide dependency)가 있음
  - 좁은 의존성(narrow dependency) : 입력파티션이 하나의 출력파티션에 영향을 미침
  - 넓은 의존성(wide dependency) : 하나의 입력파티션이 여러파티션에 영향을 미침
- 지연 연산(Lazy evaluation)은 연산 그래프를 처리하기 직전까지 기다리는 동작 방식
- Spark는 특정 연산 지시 즉시 연산을 수행하지 않고, 수행 계획을 작성
- Spark는 filter를 데이터 소스에 위임하는 최적화 기능을 자동으로 수행(predicate pushdown)
- 여러가지 액션이 존재
  - count function
  - 데이터를 한 곳으로 모으는 액션
  - 콘솔에서 데이터를 보는 액션
  - 출력 데이터소스를 저장하는 액션
- 고수준 API인 DataFrame은 스키마 추론 기능이 존재
- 출력 파티션 수를 조정하면 런타임이 크게 달라질 수 있음
- 사용자가 SQL, R, Python, Scala 언어로 비즈니스 로직을 표현하면 Spark는 이를 기본 실행 계획으로 컴파일함  
  --> 결과적으로 어떤 언어를 사용하던 큰 성능의 차이는 없음
- 실행계획은 트랜스포메이션의 DAG(Directed Acyclic Graph)임
- 데이터소스(DataSource)는 ConnectionPool을 관리하는 목적으로 사용되는 객체

### chapter03 
- Spark의 기본 요소는 저수준 API, 고수준 API, 라이브러리로 구성
- `spark-submit` 명령을 통해 spark application 코드를 클러스터에 전송해 실행시키는 역할을 함
- `spark-submit` 명령과 같이 application 실행에 필요한 자원, 실행 방식 등 다양한 옵션 지정 가능
- Dataset은 타입 안정성(type-safe)를 지원하는 고수준 API  
  R, python과 같은 동적 언어에서는 쓸 수 없음  
  초기화에 선언한 타입 외에는 사용할 수 없음, Java는 JavaBean class, Scala는 case class를 이용해 선언 가능 
- Dataset의 장점은 `collect`, `take` 메소드를 사용하면 DataFrame의 Row 객체가 반환되는 것이 아닌, 초기에 지정한 타입의 객체를 반환
- 데이터 멍잉(wrangling, munging)은 원본 데이터를 다른 형태로 변환하거나 매핑하는 과정을 의미
- 스트리밍 데이터는 수천개의 데이터소스에서 연속적으로 생성되는 데이터를 의미  
- 스트리밍 처리도 지연 연산 중 하나이므로, 액션 처리를 통해 실행 계획을 수행해 주어야 함
- Spark의 모든 ML 알고리즘은 수치형 벡터 타입을 input Data로 사용
- Spark에서의 모델 학습은 크게 1. 학습되지 않은 모델을 초기화, 2. 해당 모델 학습으로 구성  
  다음과 같은 명명 규칙을 따름 ex) 학습전 알고리즘 명칭 : Algorithm, 학습후 알고리즘 명칭 : AlgorithmModel
- Spark의 거의 모든 기능은 RDD 기반으로 만들어짐
- 구조적 API에는 다음과 같은 3가지 분산 컬렉션 API가 있음
  - DataFrame, Dataset, SQL Table과 View
- 여기서 말하는 컬렉션(collection)은 데이터의 집합, 그룹을 의미. 보통 자료구조라고 생각할 수 있음

### chapter04
- 데이터는 크게 정형, 반정형, 비정형 3가지로 나눌 수 있는데  
  이 때 반정형은 metaData + schema는 있지만 연산 자체가 불가능한 데이터를 말함
- DataFrame, Dataset의 특징 기억하기   
  - 잘 정의된 로우, 컬럼을 가지는 '분산' table 형태의 컬렉션
  - lazy evaluation + 불변성을 가짐
- schema는 DF, DS의 분산 컬렉션의 타입을 정의
- schema는 데이터소스에서 얻거나, 직접 정의함. 여기서 데이터소스에서 얻는 방법을 schema-on-read라고 함
- Spark는 자체 데이터 정보를 가지고 있는 '카탈리스트' 엔진을 통해 실행 계획 수립 및 처리를 진행
- 카탈리스트 엔진은 실행 계획 최적화 수행하고 다양한 언어 API와 타입을 매핑시켜주는 매핑 테이블을 가지고 있음  
  --> R, Python 언어로 코드를 작성하더라도 대부분 Spark 자체 데이터 타입으로 수행됨
- Spark 자체의 표현식도 카탈리스트 엔진을 통해 Spark 데이터 타입으로 변환 후 연산이 수행됨
- DataFrame은 schema와 타입 일치 여부를 런타임 시점에 확인 : 비타입형이라 부름  
  Dataset은 schema와 일치 여부를 컴파일 시점에 확인 : 타입형이라 부름
- DataFrame은 Row 타입을 가지는 Dataset으로 볼 수 있는데,  
  이때 Row 타입은 '연산에 최적화된 인메모리 포멧'의 Spark 내부 표현방식임
- Row 타입을 사용하면 garbage collection과 객체 초기화 부하가 있는 JVM 포멧을 사용안하고 Spark 자체 데이터 타입을  
  사용하기 때문에 효율적인 연산 가능
- DataFrame을 사용하면 Spark의 최적화된 내부 포멧을 사용할 수 있음
- 컬럼은 단순 데이터 타입, 복합 데이터 타입, null로 구성
- DF/DS/SQL인 구조적 API가 실제 클러스터에서 수행되는 방식은 논리적 실행 계획 -> 물리적 실행 계획으로 변환하여 최적화 수행  
  그리고 물리적 수행 계획을 기반으로 cluster에서 RDD 연산을 수행
- 논리적 실행 계획에서는 추상적 트랜스포메이션만 표현. 드라이버나 익스큐터는 고려하지 않음
- 물리적 실행 계획은 논리적 실행 계획을 실제 클러스터에서 실행하는 방법을 정의  
  비용 모델을 이용해서 최적의 실행 전략을 선택. 예를 들어 조인 연산의 비용을 파악하여 최적의 전략 선택
- 물리적 실행 계획에서 DF/DS/SQL Table은 일련의 RDD와 트랜스포메이션으로 컴파일됨  
  즉, 이 때문에 Spark를 컴파일러라고도 부름
- Spark는 런타임 시점에 전체 Task나 Stage를 제거할 수 있는 Java Byte code를 생성해 추가 최적화 수행
- parquet은 컬럼형 기반으로 데이터를 저장하고 압축률이 높고 I/O 사용률이 적음  
  컬럼별 적합한 인코딩이 가능

## chapter05
- 파티셔닝 스키마는 파티션을 배치하는 방법을 정의
- 파티셔닝의 분할 기준은 특정 컬럼이나 비결정론적인(매번 변하는) 방법으로 수행
- 운영환경에서 ETL을 수행하는 경우에는 직접 Schema를 지정해 주는 것이 좋음
- CSV, JSON과 같은 스키마 추론이 힘든 경우, 스키마 추론 과정에서 읽어들인 샘플 데이터의 타입에 따라 스키마를 결정함
- 스키마는 여러개의 StructField type으로 구성된 StructType 객체
- 필요한 경우 컬럼과 관련된 메타데이터를 지정할 수 있는데, 이는 MLlib에서 사용
- Spark는 자체 데이터 타입을 사용하므로 프로그래밍 언어의 데이터 타입을 Spark의 데이터 타입으로 설정할 수 없음
- 컬럼 내용을 수정하려면 반드시 트랜스포메이션을 이용해야함
- 컬럼 함수  
  - `col("colName")`
  - `column("colName")`
  - `df.col("count")`
- 컬럼명 존재 여부는 카탈리스트 분석기를 통해 확인후 알 수 있음
- `col` 메서드를 사용해 명시적으로 컬럼을 사용하면 Spark는 분석기 단계에서 확인 절차 생략
- DataFrame의 컬럼은 표현식이라고 정의할 수 있는데, 이는 다양한 트랜스포메이션의 집합이라고 생각할 수 있음
- 컬럼 표현식 함수
  - `expr("someCol -5")`
  - `col("someCol") - 5`
  - `expr("someCol") - 5`
- 위의 식은 모두 같은 트랜스포메이션을 수행하는데, 이유는 Spark는 연산 순서를 지정하는 논리적 트리로 컴파일하기 때문
- 컬럼관련 다음의 두가지는 꼭 기억하자
  - 컬럼은 표현식일 뿐이다
  - 컬럼과 컬럼의 트랜스포메이션은 파싱된 표현식과 동일한 논리적 실행 계획으로 컴파일됨
- SQL의 표현식은 마찬가지로 논리적 트리로 컴파일됨
- Spark의 DataFrame의 각 로우는 하나의 레코드이며, 이 레코드를 Row 객체라고 표현함
- Row 객체는 내부적으로 바이트 배열(byte [])을 가짐, 또한 Row 객체는 스키마를 가지고 있지 않음
- Row 객체의 각 index 위치의 값을 접근할 때 명시적으로 데이터 타입을 지정해야 함
- dataFrame의 트랜스포메이션은 `select`, `selectExpr`, `org.apache.spark.sql.functions` 3가지로 대부분 처리 가능
- Spark는 기본적으로 대소문자를 구분하지 않음
- 필터링하기 : `filter(col("count") < 2)`, `where("count < 2")`
- `union` 메소드는 컬럼명 기반 병합이 아닌, 컬럼 위치를 기반으로 동작함
- 컬럼명 <-> 리터럴값 비교시 Scala에서는 `=!=` 연산자 사용
- 최적화 기법으로 자주 filter하는 데이터 별로 파티셔닝을 수행하는것 : `repartition`, `coalesce`    
  - `repartition` : 무조건 전체 데이터 셔플, 향후 사용할 파티션 수가 많을 때 사용
  -  `coalesce` : 전체 데이터를 셔플하지 않고 데이터를 병합하는 경우 사용. 파티션 수를 줄일 때 사용
- 드라이버로 데이터를 모으려면 `collect`, `toLocalIterator`, `take` 등을 사용. 이때 너무 큰 데이터를 모으는 경우, 비정상적으로 종료될 수 있음

## chapter06
- `lit` 함수는 다양한 언어의 타입을 Spark 데이터 타입으로 변환해줌 : `df.select(lit(5.0), lit(true))`
- filtering function --> `===`, `equalTo`
- 만약 더 쉽다면 `expr` 를 이용한 spark SQL 표현식을 사용해도 성능 상의 이슈는 없음
- boolean 표현식을 만들 때 null 값은 다르게 처리 해야 함(ex) null-safe 동치 테스트 수행)
- 값에 null이 포함되어 있는 경우, `eqNullSafe` 함수 사용 가능
- 수치형 함수 : `pow`, `round`, `bound`, `describe`, `count`, `mean`, `stddev_pop`, `min`, `max`
- `StatFunctions` 메서드는 다양한 통계값을 제공, ex) `approxQuantile` 데이터의 백분위를 근사값을 제공
- `monotonically_increasing_id` : 모든 행에 고유 ID 부여
- 문자열 함수 : `initcap` : 공백으로 나뉘는 글자의 첫글자를 대문자로 변경
- `lower`, `upper`, `lpad`, `rpad`, `lstrip`, `rstrip`, `strip`  
- `regexp_extract`, `regex_replace`, `translate`: char 한개씩 replace `translate(col("Description"), "LEET", "1337")`
- `contains` : 해당 문자 포함 여부
- 해당 구문은 좀 기억해 두자 : `.map(tmp => {col("colName").contains(tmp).alias(s"is_$color")}):+expr("*")`
- 날짜형 함수 : 날짜형은 크게 2가지만 관리 -->  달력 형태의 날짜(date), 날짜와 시간 형태를 모두 가지는 timestamp
- Spark는 특정 날짜 포멧을 지정하지 않아도 자체적으로 판단해 날짜형을 지정 가능
- `current_date`, `current_timestamp` : 오늘 날짜의 date와 timestamp 함수 사용
- `date_sub`, `date_add` : 해당 날짜형 컬럼을 기준으로 +, - 하는 함수
- `date_diff` : 두 날짜의 차이를 구하는 함수, `months_between` : 두 날짜의 개월 수 차이를 비교
- Spark는 `to_date` 함수 사용시 변환할 수 없는 날짜이면 null 반환  
  각각 날짜의 format을 지정하고 싶으면 `to_date("2020-20-11", "yyyy-dd-MM")` 으로 지정
- `to_timestamp` : 항상 날짜 포맷을 지정해야함
- 날짜 비교시, 날짜 또는 timestamp, yyyy-MM-dd format의 문자열을 사용  
- 날짜 비교시, 항상 format을 지정해서 사용하자
- Spark에서는 빈 문자열이나 대체 값 대신 null value를 사용해야 최적화를 수행할 수 있음  
  DataFrame의 하위 패키지인 .na를 사용하는 것이 DataFrame에서 null 값을 다루는 기본 방식
- Spark에서는 null값을 허용하지 않는 컬럼을 선언할 수 있지만 강제성은 없음
- `coalesce` : 값이 있으면 해당 값을 반환하고, null이면 여러 인수 값 중 첫 번째 인수 값을 반환  
- `ifnull` : 첫 번째 인수 값이 null이면, 두 번째 인수 값 반환
- `nullif` : 두 값이 같으면 null 반환, 두 값이 다르면 첫번 째값 반환
- `nvl` : 첫 번째 값이 null이면 두 번째 값 반환, 첫 번째 값이 null이 아니면, 첫 번째 값 반환
- `nvl2` : 첫 번째 값이 null이 아니면 두 번째 값 반환, 첫 번째 값이 null이면 세 번째 인수로 저장된 값 반환
- `na.drop`: null값을 가진 로우를 제거하는 가장 간단한 함수    
  `na.drop("any")` : any 지정시 로우 값 중 하나라도 null값 포함시 row 제거
- `na.fill("채워 넣을 값", "컬럼")` : null값을 채워 넣기 위한 함수
- `replace("Description", Map("" - ""))` : 대체할 값과 대체될 값의 타입이 같아야 함   
- `asc_nulls_first` : 오름차순 정렬시 null 값을 first, `desc_nulls_first` : 내림차순 정렬시 null 값 first  
  `asc_nulls_last` : 오름차순 정렬시 null 값 last, `desc_nulls_first` : 내림차순 정렬시 null 값 last
- 복합 데이터 타입 : `struct, ()`로 복합 데이터 타입 생성, 복합 데이터 타입 내 스칼라 값 추출시 `complex`, `getField` 함수 사용
- `.*` 문자로 모든 값 조회 가능하며, 해당 컬럼 값을 최상위 수준으로 끌어올릴 수 있음 : `complexDF.select("complex.*")`
- 배열 : `split` : 문자열을 배열로 쪼갬, `size` : 배열의 길이, `array_contains` : 배열에 특정 값이 존재하는지 확인  
  `explode` : 배열 값을 입력 받아 입력된 컬럼의 모든 값을 row로 만들어 버림
~~~
"Hello World", "other col" (split 함수 적용) --> ["Hello", "World"], "other col"
  (explode 함수 적용) --> "Hello", "other col"
                          "World", "other col"
~~~
- `map` : 컬럼의 키-값 쌍을 이용해 생성. 적합한 key를 이용해 데이터를 조회할 수 있음(`complex_map["keyName"]`)  
  map의 타입을 `explode` 함수를 이용해서 컬럼으로 만들 수 있음
- JSON 타입 : `get_json_object` 함수로 JSON 객체를 인라인 쿼리로 조회 가능.   
  `to_json` : StructType 을 JSON으로 변경 가능  
  `from_json` : JSON 문자열을 StructType으로 변경 가능 
- Spark는 드라이버에서 UDF를 직렬화하고, 네트워크를 통해 모든 익스큐터 프로세스를 전달
- python UDF 사용시 Spark는 워커 노드에 파이썬 프로세스를 실행하고, 파이썬이 이해할 수 있는 포맷으로 데이터를 직렬화함  
  파이썬 프로세스에 있는 데이터의 로우마다 함수를 실행하고 JVM과 Spark에 처리 결과를 반환 
- 파이썬 실행시 문제점
  - 데이터를 직렬화하는 데 큰 부하가 발생
  - 데이터가 파이썬으로 전달되면 Spark에서 워커 메모리를 관리 할 수 없음  
    따라서 파이썬과 Spark가 동일한 머신에서 실행되면 메모리 경합을 해 자원에 제약이 생겨 비정상적으로 종료될 수 있음  
    따라서 java나 Scala로 UDF를 만드는 것이 좋음
- `udf` 를 사용하면 dataFrame의 함수로 사용 가능  
  `spark.udf.register("power3", power3(_:Double):Double)` 를 사용하면 `expr` 내에서 사용 가능
- Spark는 자체 데이터 타입을 사용하기 때문에 UDF 정의시 반환 타입을 지정하는 것이 좋음   
  반환되는 타입과 실제 데이터 타입과 매칭되지 않으면 Spark는 null값을 발생시킴  
  Spark 함수가 전부 다 되는 것은 아님 

## chapter07
- Spark는 수행 가능한 정확도에 맞춰 근사치를 계산하여 성능을 높일 수 있음  
- `count()` 메소드는 데이터의 수를 세는 용도 뿐만 아니라 dataFrame을 캐싱하는데 사용
- 집계 함수는 `org.apache.spark.sql.functions` 에서 찾아볼 수 있음
- `count`
  - count 함수는 트랜스포메이션과 액션 두 가지 방식으로 사용 가능
  - `count("colName")`은 트랜스포메이션 방식, 컬럼명이 들어가면 null을 제거하고 count,  
    `*`가 들어가면 null 포함 count
- `countDistinct` 
  - distinct된 data count
- `approx_count_distinct` 
  - countdistinct의 근사치 계산. 최대 추정 오류율이라는 parameter 있음
- `first`, `last` 
  - 첫 번째와 마지막 값을 얻을 때 사용
- `min`, `max`, `sum`, `sumDistinct`, `avg`
- `variance`, `stddev`
  - 표본표준편차, 표본분산값 계산
- `var_pop`, `stddev_pop`
  - 모표준편차, 모분산값 계산
- 모표준편차와 표본표준편차의 차이는 분모에 N으로 나누느냐, N-1로 나누느냐의 차이
- `skewness`, `kurtosis`
 - 비대칭도(skewness), 첨도(kurtosis)를 계산
- `corr`, `covar_pop`, `covar_samp`
  - 상관계수, 모공분산, 모표본분산
- Spark은 특정 컬럼을 복합 데이터 타입으로 처리 할 수 있음  
  예를 들어 리스트나 Set Data Collection Type으로 처리 가능
- 다음과 같은 구문을 사용할 수 있음을 기억하자
~~~scala
// agg 함수를 사용하여 집계 가능
df.groupby("colName").agg(
  count("colName").alias("name"),
  expr("count(Colname)")).show()

// Map을 이용한 그룹화
df.groupby("colName").agg("colName1" -> "avg", "colName2" -> "stddev_pop")
~~~
- Spark는 3가지 윈도우 집계 함수 지원 --> rank, analytic, aggregate
- 다음의 예제를 눈에 익게 해두자
~~~scala
import org.apache.spark.sql.expressions.Window
import org.apache.spark.sql.functions.{col, to_date}
import org.apache.spark.sql.functions.max

val dfWithDate = df.withColumn("date", to_date(col("InvoiceDate"), "MM/d/yyyy H:mm"))
dfWithDate.createOrReplaceTempView("dfWithDate")

val windowSpec = Window
.partitionBy("CustomerId", "date") // 파티셔닝 스키마 X, 그룹을 어떻게 나눌지 결정하는 것
.orderBy(col("Quantity").desc)     // 파티션의 정렬 방식을 정의함
.rowsBetween(Window.unboundedPreceding, Window.currentRow)

val maxPurchaseQuantity = max(col("Quantity")).over(windowSpec)
val purchaseDenseRank = dense_rank().over(windowSpec)
val purchaseRank = rank().over(windowSpec)

dfWithDate.where("CustomerId IS NOT NULL").orderBy("CustomerId")
.select(
  col("CustomerId"),
  col("date"),
  col("Quantity"),
  purchaseRank.alias("quantityRank"),
  purchaseDenseRank.alias("purchaseDenseRank"),
  maxPurchaseQuantity.alias("maxPurchaseQuantity")).show()
~~~
- sql 에서는 `GROUPING SETS`, dataFrame에서 `rollup` 과 `cube` 함수 사용
- `rollup` -> 그룹화 키로 설정된 조합 + 데이터셋에서 볼 수 있는 실제 조합을 모두 볼 수 있음  
  ex) 그룹화를 날짜, 국가로 지정했을 때 날짜의 총합, 날짜별 총합, 날짜별 국가별 총합 확인 가능
~~~scala
val rolledUpDF = dfNoNull.rollup("Date", "Country").agg(sum("Quantity"))
.selectExpr("Date", "Country", "`sum(Quantity)` as total_Quantity")
.orderBy("Date")
rolledUpDF.show()
~~~
- `cube` -> rollup을 조금 더 고차원적으로 확인  
  Date와 Country를 그룹으로 지정했을 때, `groupby("Date", "Country")` + `groupby("Date")` + `groupby("Country")` 의 결과가 나온다고 생각하면 이해하기 쉬움
- 큐브와 롤업의 집계 수준 조회시 `grouping_id` 사용
- 그룹화 id의 수가 높아질수록 groupby를 무시한 총량 계산 한다고 생각하자 
- `pivot`
  - 로우를 컬럼으로 변환
~~~scala
val pivoted = dfWithDate.groupBy("date").pivot("Country").sum()
pivoted.where("date > '2011-12-05'").select("date", "`USA_sum(Quantity)`").show()
~~~
- Spark는 입력 데이터의 중간 결과를 단일 AggregationBuffer에 저장해 관리  
  
## chapter08
- Spark 조인 타입
  - 내부 조인(inner join), 외부 조인(outer join), 왼쪽 외부 조인(left outer join), 오른쪽 외부 조인(right outer join)
  - 왼쪽 세미 조인(left semi join) : 키가 일치하는 경우에 키가 일치하는 왼쪽 데이터셋만 유지
  - 왼쪽 안티 조인(left anti join) : 키가 일치하지 않는 경우에는 키가 일치하지 않는 왼쪽 데이터셋만 유지
  - 자연 조인(natural join) : 두 데이터셋에서 동일한 이름을 가진 컬럼을 암시적으로 결합하는 조인을 수행
  - 교차 조인(cross join) 또는 카테시안 조인(Cartesian join) : 왼쪽 데이터셋의 모든 로우와 오른쪽 데이터셋의 모든 로우를 조합
- 조인 예시 코드
~~~scala
// val joinType = "outer", "inner", "left_outer", "right_outer", "left_semi", "left_anti"
val joinExpression = person.col("graduate_program") === graduateProgram.col("id")
person.join(graduateProgram, joinExpression, joinType).show()
~~~
- dataFrame의 컬럼은 카탈리스트 엔진에 고유 Id가 있음  
  카탈리스트 엔진에 있는 고유 id는 직접 참조할 수 있는 값은 아님.
- Spark의 조인 수행 방식을 이해하기 위해서는 핵심적인 두 가지 핵심 전략을 이해해야 함
  - 노드간 네트워크 통신 전략과 노드별 연산 전략
- 네트워크 통신 전략 
  - Spark는 조인시 두 가지 클러스터 통신방식 활용
  - 전체 노드간의 통신을 유발하는 셔플 조인과 통신을 유발하지 않는 브로드케스트 조인 방식이 있음
- 셔플 조인
  - 두 개의 큰 데이터를 조인할 때 적용되는 조인 방식
  - 전체 노드간 통신이 발생. 
  - 특정 키나 키 집합을 어떤 노드에서 가지고 있느냐에 따라 해당 노드와 데이터를 공유
  - 통신 방식 때문에 네트워크는 복잡해지고 많은 자원을 사용 
- 브로드캐스트 조인
  - 테이블이 단일 워커 노드의 메모리 하나에 로딩이 가능할 정도로 작은 경우 수행
  - 작은 DataFrame을 전체 워커 노드에 복제함. 이렇게 하면 자원을 많이 사용하는 것처럼 보이지만  
    전체 노드의 네트워크가 일어나지 않으므로 자원을 아낄 수 있음  
    데이터를 복제할 때 한 번만 네트워크가 수행되고, 뒤에는 노드간 네트워크가 수행되지 않음
  - 모든 단일 노드에서 개별적으로 조인이 수행되므로, CPU가 가장 큰 병목 구간이 됨
  - `explain` 함수를 사용하면 조인시 어떤 네트워크  통신 전략을 사용했는지 확인 가능 
- DataFrame API를 사용하면 조인시 hint를 줄 수 있음
~~~
person.join(broadcast(graduateProgram), joinExpr).explain()
~~~
- SQL 역시 조인 수행에 필요한 힌트를 줄 수 있음
- 너무 큰 데이터를 브로드케스트 하면 메모리 상에 해당 데이터를 다 로딩할 수 없어 driver process가 강제 종료될 수 있음
- 아주 작은 테이블 끼리의 조인은 spark가 자동으로 처리하도록 두는 것이 좋음
- 조인 전에 데이터를 적절히 분할하면 셔플이 계획되어 있더라도 피할 수 있고 효율적으로 실행 가능

## chapter09
- JSON(JavaScropt Object Notation)
- 파케이는 컬럼에 복합 데이터 구조도 가능
- ORC는 하둡 워크로드를 위해 설계된 self-decribing 구조이며, 데이터 타입을 인식할 수 있는 컬럼 기반 데이터 타입 포맷
- DB에서 데이터를 읽고 쓰려면 jar파일의 JDBC 드라이버가 존재해야 함
- dbTable에 단일 테이블 대신에 서브쿼리도 사용 가능
- <b> Query pushDown </b> : DataFrame에서 트랜스포메이션을 수행하기 전에 쿼리에서 해당 트랜스포메이션(ex) filtering)을 수행할 수 있음  
  DataFrame의 filter 함수를 Spark는 DB query로 자체 위임해 최적화하는 기능이 있음
- DB에서 `numPartition` 수를 지정하여 최대 파티션 수를 지정할 수 있음
- 병렬로 데이터를 읽고 쓰는 것이 최적화에 효율적인데, 일반적으로 데이터 포맷은 파케이, 압축 방식은 GZIP이 효율적임
- 읽을 때는 하나의 익스큐터에서 하나의 파티션(file)을 읽을 수 있으며, 쓸 때는 파티션 하나당 하나의 파일이 생성됨
- 파티셔닝(partitioning)  
  - 데이터를 어디서, 어떻게 저장 하는지를 제어하는 기능. 
    ex) 디렉토리 별로 컬럼 데이터를 저장할 수 있음 --> 데이터를 전부 읽어보지 않고 특정 컬럼의 데이터를 가져올 수 있음  
    ex) 날짜별로 디렉토리에 저장하여(날짜를 partition parameter로 지정) 과거 날짜의 데이터를 가져오는 등을 수행할 수 있음
- 버켓팅(bucketing)
  - 버켓팅을 수행하게 되면 파티션 별로 동일한 버켓ID가 부여되여 저장되며, 이를 다시 읽을 때 셔플이 일어나지 않음
- 너무 작은 파일을 여러개 만들면 메타 데이터의 부하가 발생하여 Spark는 이를 잘 다루지 못함  
  너무 큰 데이터도 마찬가지로 메모리 상에 부하가 발생할 수 있음
- JDBC(Java DataBase Connectivity)  
  - Java에서 DB 프로그래밍을 하기 위한 API. Database 모듈에 상관이 없음
- JDBC 드라이버  
  - Java Class로써 DBMS와의 통신을 담당
- ORC(Optimized Row Columnar)
  - 컬럼 기반의 파일 저장방식 
  - Hadoop, Hive, Pig, Spark에 적용 가능
  - ORC는 컬럼 단위로 데이터를 저장하기 때문에 압축 효율이 좋고, 성능이 빠름
- 이스케이프 처리  
  - 특수 문자 앞에 `\`를 붙여 특수 문자를 정상 문자로 처리하고 싶을 때 이스케이프 처리를 한다고 함
- 트랜젝션 격리 수준(transaction isolation level)
  - read uncommited : commit 되지 않아도 table의 데이터가 수정되면 수정된 값이 그대로 반영 
  - read commited   : commit된 정보만 table에 반영되어 나타남
  - repeatable read : 트랜잭션이 시작되기 전 정보 이후의 갱신된 데이터는 나타나지 않음
  - serializable    : 트랜잭션이 시작되면 읽기 작업에도 잠금이 설정되어 다른 작업 수행시 해당 table에 접근하지 못함
- TRUNCATE : DELETE 명령어와 같이 특정 Table의 레코드를 삭제하는 역할을 함  
  - DELETE와는 달리 Table DROP 후 CREATE
  - 속도는 TRUNCATE가 더 빠르지만 복구가 불가함


## chapter15
- driver는 cluster에서 실행 중인 application 상태를 유지(SparkSession..?)
- driver는 spark application을 제어, cluster 상태 정보 유지
- driver는 executor 실행을 위한 클러스터 매니저와 네트워크 및 물리적 컴퓨팅 자원 확보
- executor는 실행 결과를 driver에게 보고
- 클러스터 매니저는 spark application 을 실행할 물리적 머신을 유지
- 클러스터 매니저는 드라이버(= 마스터)와 워커라는 개념을 가지고 있으며, <b>프로세스가 아닌 물리적 머신과 연결되는 개념임</b> 
- spark application이 실행되지 않고 있는 클러스터 구성에 대한 이해 필요
  - Master 노드는 개별 워커 노드를 실행 관리하는 데몬 프로세스가 있음
- 실행 모드는 application을 실행할 때 요청한 자원의 물리적인 위치를 결정  
  크게 client mode, clutser mode, local mode
- 클러스터 모드는 가장 흔한 방식이며 컴파일된 JAR 파일, R/Python Script를 클러스터에게 전달해야 함
- 클러스터는 파일을 받은 다음 워커 노드에 driver/executor process 실행
- 클러스터에서 Spark Application을 실행할 때 수행되는 단계  
  - client가 spark-submit을 통해 application을 제출
  - spark-submit은 드라이버 프로그램 실행. 사용자가 정의한 main 함수 실행
  - 드라이버 프로그램은 클러스터 매니저에게 executor 실행을 위한 리소스 요청
  - 클러스터 매니저는 드라이버 프로그램을 대신해 executor 실행
  - 드라이버는 program에 작성된 RDD의 트랜스포메이션, 액션 기반의 작업 내역(Job)을 태스크로 나눠 executor에게 전달
  - 단위 작업들은 executor들에 의해 실행되고 결과를 다시 driver에게 전달
  - driver의 main이 종료되거나, SparkContext.Stop()이 수행되면 executor들은 중지되고 자원들은 다시 반환됨
- <b>꼭 기억해야 할 것은 driver/executor process가 전부 worker node에서 실행된다는 것</b> 
- 클라이언트 모드에는 driver process가 client 머신에 존재  
  클라이언트 모드 수행시, Spark Application을 실행했던 콘솔을 닫아버리거나 기타 다른 방법으로 client process를 종료시키면  
  Spark Context도 같이 종료되면서 Spark Job이 종료
- 로컬모드로 설정된 경우 단일 머신에서 spark application이 실행됨
- 로컬모드에서는 병렬 처리를 위해 단일 머신의 스레드를 활용
- 클러스터 관점 Spark Application Life cycle
  - 클라이언트 요청 : Spark Application을 제출하는 것(컴파일된 JAR FILE), driver process가 클러스터에 배치됨
  - 사용자 코드 실행, 이때 사용자 코드에는 Spark Cluster를 초기화 시키는 SparkSession이 있어야 함  
    SparkSession은 CM과 네트워크 통신을 통해 executor 실행 요청  
  - Spark 클러스터가 생성되었으므로, 코드를 실행  
    드라이버와 워커(CM)는 코드를 실행하고 데이터를 이동하는 과정에서 서로 통신함  
    드라이버는 각 워커에 Task를 할당함
  - 완료시 driver process는 성공/실패 중 하나의 상태로 종료됨  
    드라이버가 속한 Spark 클러스터의 모든 익스큐터를 종료시킴  
- 모든 Spark Application은 가장 먼저 SparkSession을 생성
- <b>대부분의 대화형 모드에서는 자동으로 생성되지만, 애플리케이션을 만드는 경우라면 직접 생성해 주어야 함</b>
- SparkSession의 builder 메서드를 이용해서 생성해야 Spark 애플리케이션에서 다수의 라이브러리가 세션을 생성하는 상황에서  
  충돌 방지
- SparkSession을 생성하면 Spark code를 실행할 수 있음
- SparkSession은 2.x 버전 이상에서만 사용 가능하며, 과거 버전에서는 SparkContext + SQLContext를 이용
- SparkContext는 클러스터의 연결을 나타내며, 이를 이용해 RDD와 같은 저수준 API를 사용 가능
- 대부분의 경우 SparkSession 으로 SparkContext 접근 가능하여 SparkContext를 초기화 시켜줄 필요 없음
- 직접 초기화하는 가장 좋은 방법은 `getOrCreate` 메서드 사용
- `collect`와 같은 액션을 호출하면 개별 Stage와 task로 이루어진 Spark Job이 실행됨
- <b>액션 하나당 하나의 Spark Job이 수행되며,</b> 액션은 항상 같은 결과를 반환함
- Spark Job은 일련의 Stage로 나뉘며, Stage 수는 셔플 작업이 얼마나 많이 일어나느냐에 따라 다름
- Spark의 스테이지는 다수의 머신에서 동일한 테스크를 수행하는 태스크의 그룹
- 셔플 작업이 수행된 후에는 반드시 새로운 스테이지가 수행됨
- 셔플은 데이터의 물리적 재분배 과정임
- 경험적으로 클러스터의 익스큐터 수보다 파티션 수를 높게 지정하는 것이 좋음
- 로컬 머신에서는 병렬로 처리할 수 있는 태스크 수가 제한적임을 생각해야 함
- 태스크는 단일 익스큐터에서 실행할 데이터의 블록과 트랜스포메이션의 조합으로 볼 수 있음 
- 1000개의 파티션 --> 1000개의 태스크로 만들어 병렬 실행
- <b>태스크는 데이터 연산 단위라고 생각할 수 있음</b>
- map + map 연산시 자연스레 태스크와 스테이지가 연결
- 모든 셔플 작업 수행시 안정적인 저장소에 저장하므로 여러 Job에서 재사용 가능 
- <b>Spark를 인메모리 컴퓨팅 도구로 만들어주는 핵심은 메모리나 디스크에 쓰기 전 최대한 많은 연산을 수행해 준다는 것</b>
- 파이프라이닝 기법은 노드 간의 데이터 이동 없이 각 노드내에서 연산만을 모아서 태스크의 단일 스테이지로 만듬
- Spark 런타임에서 파이프라이닝을 자동으로 수행해줌
- reduce-by-key 같은 연산 수행시 소스 태스크의 스테이지 수행 동안 셔플 파일을 디스크에 기록  
  저장된 셔플 파일은 재사용됨(뒷단의 연산은 재수행되지 않음)
- 이러한 자동 최적화 기능은 워크로드의 시간 최적화에 도움이 됨  

## chapter 16
- Spark application은 빌드 도구로 apache maven이나 sbt를 사용할 수 있음
- Spark application 테스트에 대한 전략적 원칙
  - 입력 데이터에 대한 유연성
    - 입력 데이터가 일부 바뀌더라도 유연하게 대응할 수 있어야 함
    - 아니면 오류 상황을 유연하고 적절하게 대응할 수 있어야 함
  - 비즈니스 로직 변경에 대한 유연성
  - 결과의 유연성과 원자성
    - 결과가 원하는대로 반환되는지 확인해야 함
    - 데이터가 생성되면 해당 데이터는 거의 반드시 실행되므로, 갱신 날짜 등을 확인하여 테스트 해야함
- 테스트 하네스(harness)란 시스템과 시스템 컴포넌트를 테스트하는 환경의 일부  
  테스트를 지원하는 목적하에 생성된 코드와 데이터
- 의존성 주입(Dependency Injection) 방식  
  - 클래스 별로 의존적인 상황을 막아줌
  - https://velog.io/@wlsdud2194/what-is-di
- 코드를 단위 테스트 하려면 테스트 하네스에서 테스트마다 SparkSession을 생성하고 제거하는 것이 좋음
- 가능하면 테스트 코드에서 데이터 소스는 운영 환경 연결 X  
  데이터소스가 변경되더라도 고립된 환경에서 개발자가 쉽게 테스트 가능
- 비즈니스 로직을 가지고 있는 함수가 데이터 소스에 직접 접근하면 안됨  
  데이터 소스 함수를 따로 만들어야 함
- 드라이버와 익스큐터 간의 네트워크 지연시간을 줄이기 위해 클러스터 모드 추천
- Master option 값을 local 이나 local[*]로 바꾸면 로컬 모드에서 실행 가능
- `Spark` 속성은 대부분 애플리케이션 파라미터를 제어하며 `SparkConf` 객체를 이용해 스파크 속성 설정 가능
- IP 주소같은 환경변수는 Spark 클러스터 노드의 `conf/spark-env.sh` 스크립트를 사용해 머신별로 설정할 수 있음  
  해당 shell이 없을 수도 있기 때문에 없다면 template을 참고하여 생성 해야 함
- `log4j.properties` 파일을 변경해 로그 관련 설정을 변경할 수 있음
- `SparkConf` 객체는 개별 Spark application의 속성값과 클러스터 구성값을 control 하는 용도로 사용
- 이 절에서 Job은 해당 액션을 수행하기 위하여 수행되어야 할 여러 태스크와 액션의 집합을 의미함
- Spark application에서 별도의 스레드를 이용해 여러 job을 병렬 처리 할 수 있음
- Spark는 모든 Job 클러스터 자원을 거의 동일하게 사용하는 '라운드 로빈'방식으로 여러 Spark Job에 태스크를 할당
- 페어 스케줄러(Fair Scheduler)는 제출된 작업이 동등하게 수행될 수 있도록 지원  
  페어 스케줄러 사용시 pool의 그룹화를 수행할 수 있게되고, 그룹별로 자원 할당의 가중치 부여가 가능 
- 컴파일은 빌드의 부분집합이라 할 수 있음

## chapter 17
- Spark 애플리케이션 실행을 위한 클러스터 환경은 크게 두 가지로 나눌 수 있음
  - on-premise 환경
  - public cloud 환경
- on-premise cluster 환경은 자체 데이터 센터가 구축되어 있는 경우 적합함  
- on-premise cluster 구축시 사용 중인 하드웨어를 완전히 제어할 수 있으므로 특정 워크로드의 성능 최적화가 가능
- 하지만 Spark 같은 데이터 분석 워크로드에서는 on-premise 환경이 몇 가지 문제를 발생시킬 수 있음
- on premise cluster의 크기는 제한적
- 클러스터의 크기가 너무 작으면 신규 ML 모델을 학습하는 Job을 실행시키기 어렵고, 너무 크면 남용 자원이 많아짐
- on-premise cluster는 HDFS나 분산 키-값 저장소 같은 자체 저장소 시스템을 선택하고 운영해야함  
  상황에 따라 geo-replication 및  disaster recovery 체계도 함께 구축해야 함
- on-premise cluster 사용시 자원 활용 문제를 해결할 수 있는 가장 좋은 방법은 클러스터 매니저 사용
- 클러스터 매니저를 사용하면 다수의 Spark application을 사용할 수 있고, 자원을 동적으로 할당이 가능
- Spark가 지원하는 클러스터 매니저는 Spark 어플리케이션 뿐만 아니라 동시에 다른 여러 어플리케이션을 실행할 수 있음
- 자원 공유적인 측면에서 on-premise / public cloud는 큰 자이가 있는데, on-premise는 여러 종류의 저장소를 내가 스스로 선택이 가능하고,  
  public cloud는 애플리케이션 크기에 맞는 규모의 클러스터 얻을 수 있음
- 클라우드 환경에서 빅데이터 워크로드를 실행하면 얻을 수 있는 장점이 존재
  - 자원을 탄력적이게 늘이고 줄이는 것이 가능
  - 일반적인 연산을 수행하는 경우에도 애플리케이션마다 다른 유형의 머신과 클러스터 규모를 선택할 수 있어 가격 대비 뛰어난 성능을 얻을 수 있음
    ex) 딥러닝 작업이 필요한 경우에만 GPU 인스턴스를 할당받아 사용 가능
  - 많은 클라우드로 전환하려는 기업들은 기존의 클러스터를 운영하는 방식 그대로 가져가려고 하는데, 
    고정된 크기의 클러스터는 자원의 효율성을 가져가지 못함
  - 따라서 Amazon S3, Azure blob, GCS 와 같이 클러스터와 분리된 글로벌 저장소 시스템을 사용하고  
    Spark 워크로드마다 별도의 클러스터를 동적으로 할당하는 것이 좋음
- 연산 클러스터와 저장소 클러스터를 분리하면 연산이 필요한 경우에만 클러스터 비용을 지불하면 됨
- 동적으로 크기 조절이 가능하며 하드웨어 종류가 다른 것들을 섞어서 사용도 가능함
- 클라우드 환경에서는 연산 관련 프로그램 및 환경을 관리할 필요가 없음  
- stand-alone 클러스터 매니저는 Spark workload용으로 특별히 제작된 경량화 플랫폼
- stand-alone은 spark application만 실행할 수 있음
- YARN은 Job 스케줄링과 클러스터 자원 관리용 프레임워크
- Spark은 하둡 에코시스템의 일부로 착각해 잘못 분류하는 경우가 많은데, Spark은 하둡과 거의 관련이 없음 
- `spark-submit` 명령의 `--master` 인수를 yarn으로 지정해 하둡 YARN 클러스터에서 Spark Job 실행 가능
- YARN 클러스터와 다른 배포 환경과의 가장 큰 차이점은 `--master` 인수 값을 `yarn`으로 지정한다는 것
- `HADOOP_CONF_DIR`과 `YARN_CONF_DIR` 환경변수를 통해 YARN 설정 파일을 찾아냄  
  이 환경변수를 하둡의 환경 설정 디렉터리 경로에 설정하면 `spark-sumbit` 명령 실행 가능
- YARN에서는 두 가지 배포 모드로 spark application 실행 가능
  - cluster 모드 : YARN 클러스터에서 Spark 드라이버 프로세스를 관리하며 클라이언트는 애플리케이션을 생성한 즉시 종료
  - client 모드 : 드라이버가 클라이언트 프로세스에서 실행되고 YARN은 마스터 노드를 관리하지 않으며 application의 익스큐터 자원을 배분하는 역할을 함
- cluster 모드에서는 Spark application이 실행된 노드가 아닌 다른 노드에서 Spark job이 실행될 수 있음  
  그러므로 외부 라이브러리 + jar 파일을 해당 클러스터 노드에 배포하거나 <b>spark-submit 명령에 --jars 인수에 명시해 배포해야함 </b>
- 보안 관련 설정은 주로 통신 방식과 관련이 있는데, 인증, 네트워크 구간 암호화, SSL, TLS 설정 등이 있음
- Spark는 여러 연산 과정에서 필요한 자원을 스케줄링 할 수 있는 몇가지 기능을 제공함
- Spark application은 독립적인 익스큐터 프로세스를 실행  
  클러스터 매니저는 spark application 전체 스케줄링 기능 제공
- Spark application에서 여러 개의 Job을 여러 스레드가 제출했을 경우 동시에 실행 가능  
- Spark는 application 자원 스케줄링을 위한 fair 스케줄링 기능 제공
- 단일 클러스터를 공유하는 여러 사용자가 동시에 spark application을 실행할 경우,  
  클러스터 매니저에 따라 자원을 할당할 수 있는 여러 옵션들이 존재
- 모든 클러스터 매니저가 할 수 있는 가장 간단한 방법은 각각 application 마다 동일한 자원 할당  
  이러면 spark application은 Job 수행이 끝날때까지 자원을 점유함
- `spark-submit` 명령에는 특정 application에 자원 할당을 위한 여러 옵션이 있음
- 동적할당 기능을 사용하면 대기 중인 태스크 수에 따라 동적으로 자원을 할당 할 수 있음
- 동적 할당은 application이 사용하지 않은 자원을 클러스터에 반환하고, 필요할 때 제공되는 방식을 의미
- 동적 할당은 다수의 spark application이 spark 클러스터 자원을 공유해야 할 때 특히 유용
- 클러스터 매니저의 동적 할당 기능의 default는 사용하지 않음임
- <b>배포 환경 선정시 가장 중요한 것은 application의 개수와 유형</b>  
- YARN은 HDFS를 사용하는 경우에는 적합하고, 그 외에는 잘 사용하지 않음  
  YARN은 HDFS의 정보를 사용하도록 설계되었기 때문에 클라우드 환경에서는 잘 사용할 수 없음  
  또한 저장소 클러스터와 연산용 클러스터가 강하게 결합되어 있기 때문에 연산 + 저장소 클러스터를 동시에 확장해야 함
- 메소스 클러스터는 YARN의 확장형 버전으로 다양한 application을 지원하지만  
  spark application만 수행하려고 메소스 클러스터를 사용하는 것은 바람직하지 못함
- 각 노드별 spark 버전을 따로 관리하는 것은 어려우므로, 버전 통일을 하는 것이 좋음
- YARN과 메소스는 기본적으로 로그 기능을 제공
- 일반적으로 Spark는 셔플 블록을 특정 노드의 디스크에 저장함
- bus system : 기본적으로 데이터를 통신할 수 있도록 해주는 시스템
- URI : 인터넷에 있는 고유 자료 id 라고 생각하면 됨
- SSH : 네트워크 프로토콜 중 하나. 컴퓨터와 컴퓨터가 인터넷과 같은 Public Network를 통해 서로 통신을 할 때
보안적으로 안전하게 통신하기 위해 사용하는 프로토콜
- 프록시(proxy)  
  '대리'라는 의미로, 주로 보안상의 이유로 직접 통신할 수 없는 두 점 사이에서 통신을 할 경우
그 상이에 있어서 중계기로서 대리로 통신을 수행하는 기능을 가리킴

## chapter 18
- Spark Job의 오류 발생 지점을 파악하려면 Spark Job을 모니터링 해야하는데 모니터링 대상은 크게 다음과 같음
  - Spark application Job
  - JVM
  - OS와 머신
  - 클러스터 매니저 
- Spark Job 오류 발생시 가장 먼저 Spark UI와 Spark Log를 확인 해야 함   
  해당 UI는 SDD와 쿼리실행계획같은 개념적 수준의 정보를 제공
- Spark의 익스큐터는 개별 JVM에서 실행되는데, 코드가 실행되는 과정을 이해하기 위해 가상머신을 모니터링 해야함
- JVM 도구에는 stackTrace를 생성하는 jstack, heap dump를 생성하는 jmap, 시계열 통계 report 제공하는 jstat 등이 있는데  
  이는 JVM 내부 작동 방식을 이해하는데 도움이 됨  
  저수준의 디버깅이 필요하다면 JVM 도구가 유용
- CPU, 네트워크, I/O 등의 자원에 대한 모니터링도 함께 해야함  
  이것들은 클러스터 수준의 모니터링 솔루션에서 확인 가능
- YARN, 메소스 같은 클러스터 매니저도 모니터링 해야함  
  강글리아(Ganglia), 프로메테우스(Prometheus)가 있음
- 모니터링 대상은 크게 2가지로 나눌 수 있음
  - 실행중인 사용자 어플리케이션 프로세스(CPU, 메모리 사용률 등)
  - 프로세스 내부 쿼리실행계획(Job, task)
- Spark application 모니터링시, driver를 유심히 관찰해야함
- 단일 머신 또는 단일 JVM을 모니터링 해야하는 경우 드라이버를 모니터링 해야함
- `$SPARK_HOME/conf/metrics.properties` 파일을 생성해 매트릭 시스템 구성 가능. 이는 강글리아 같은 모니터링 시스템에 내보낼 수 있음
- SparkUI 등을 통하여 Query, Job, Stage, Task 모니터링도 중요
- Spark를 가장 상세하게 모니터링 하는 방법은 Log File을 참조 하는 것  
- python은 자바 기반의 로깅 프레임워크를 사용할 수 없으므로, logging 모듈이나 print 구문을 사용해 표준 오류로 결과를 확인해야 함  
- `spark.sparkContext.setLogLevel`을 통해 Log Level 수정
- 로깅 프레임워크 사용시 Spark Log와 사용자가 필요한 정보까지 함께 로그로 기록할 수 있음 --> Spark + Spark application 모두 점검 가능
- local mode에서 어플리케이션 실행시 로그 자체가 표준 오류로 출력되지만 cluster mode로 실행시 클러스터 매니저로 파일에 로그 저장 가능  
  클러스터 매니저 공식 문서에 로그 파일을 찾는 방법이 나와 있음  
  일반적으로 해당 클러스터 매니저의 Web UI로 로그 조회 가능
- 클라우드 환경에서 실행되는 경우 머신이 고장나거나 전원이 내려갔다면 로그가 기록된 머신에서 기록된 로그를 내려 받아  
  문제점 확인 가능
- Spark UI는 실행 되었거나 실행 중인 application과 워크로드에 대한 평가지표를 모니터링 할 수 있음
- 모든 SparkContext는 application 실행시 유용한 정보를 제공하는 Web UI를 4040 port로 제공  
  다수의 application 실행시 port 번호를 순차적으로 증가시켜 가면서 확인 가능
- 다음은 Spark UI에서 확인 가능한 Tab 
  - Job, Stages, Storage(application의 caching 정보, 데이터 정보), Environment, Executors, SQL 
- Spark UI는 SparkSession이 실행되는 동안 사용할 수 있음
- 정상/비정상적으로 종료된 Spark application의 정보를 확인하려면 Spark 히스토리 서버를 이용해야함
- 이벤트 로그를 저장하도록 Spark application을 설정하면 Spark 히스토리 서버를 통해 Spark UI, REST API 재구성 가능    
- Spark 히스토리 서버 사용시 spark application 설정하여 특정 경로에 이벤트 로그 저장
- 설정이 제대로 구성되면 Spark history 서버는 저장된 이벤트 로그 기반으로 Web UI를 자동으로 재구성함
- 일부 클러스터 매니저와 클라우드 서비스에는 로깅을 자동으로 구성해줌

### 디버깅 및 Spark 응급 처치 
- Spark application 이 실행되지 않는 경우 
  - 대부분 머신, 클러스터 영역 등 설정 관련 문제
  - 사용자 application에서 익스큐터 자원을 클러스터 유휴 자원 이상으로 요청하는 경우가 있음 
  - 설정한 port로 클러스터 간 통신이 가능한지 확인 필요
  - 간단한 spark application 실행이 되는지 확인 
  - 클러스터 매니저의 UI로 유휴 자원을 확인한다음 `spark-submit` 명령에 할당할 메모리 설정
- Spark application 실행 전에 오류 발생한 경우
  - Spark 실행 계획(DataFrame API 사용한 경우)을 만드는 과정에서 오류 발생
  - 잘못된 입력 파일 경로, 컬럼명 등 확인 필요
  - 클러스터의 드라이버, 워커, 저장소 시스템간 연결 확인
  - 잘못된 버전 라이브러리, classpath 확인
- Spark application 실행 중에 오류 발생한 경우
  - Spark Job 중간에 발생하거나, 특정 batch 성 Job에 특정 날짜에 error 발생
  - 데이터 포멧 확인, 쿼리 실행 즉시 오류 발생하는 경우 쿼리실행계획 제작시 오류 발생일 경우일 수 있음
  - 쿼리 내에 컬럼명, 테이블 명 확인
  - StackTrace에서 정확한 원인을 파악해야 함
- 느리거나 뒤처진 태스크
  - 머신 간의 균등하게 Job이 배분이 되지 않은 경우 발생
  - 느린 테스크를 낙오자(straggler) 라고 부르기도 함  
  - 파티션별 데이터양을 줄이기 위해 파티션 수를 증가해 볼 수 있음
  - 다른 컬럼을 조합해 파티션을 재분배 하는 것도 하나의 방법
  - null값이 많은 경우 치우침 현상이 발생할 수 있으므로 null처리를 해보는 것도 방법 중 하나
  - 특정 클러스터의 disk 확인, 또는 익스큐터의 메모리 증가도 하나의 방법
  - UDF, UDAF 확인(로직 확인, 성능 확인)
- 느린 집계 속도
  - 집계 처리 끝나고 다음 태스크가 느리다면 repartition 수행하자
  - null 값 대신에 ""나 대체값으로 수행되는지 확인. null값이면 건너띄고 최적화를 수행하는데 대체되면 그렇지 못함
  - 특정 집계 함수는 매우 느림. 집계 함수인 `collect_list`, `collect_set`은 일치하는 모든 객체를 드라이버에 전송
- 느린 조인 속도
  - 조인 연산은 셔플 부하를 일으킴
  - SELECT, Filter 연산 등이 조인 연산 앞에 와야함
  - 브로드케스트, 셔플 조인의 선택을 수정해 보아야 함
- 느린 읽기와 쓰기 속도
  - 투기적 실행(spark.speculation -> true로 설정)을 사용하면 느린 읽기와 쓰기 속도 개선 가능  
    해당 기능 사용시 일관성 기능을 제공하는지 확인 필요
  - Spark 클러스터와 저장소 간의 네트워크 대역폭 확인 필요
- 드라이버 OutOfMemoryError 또는 응답 없음
  - Spark application이 비정상적으로 종료되므로 심각한 문제
  - `collect` 같은 함수 수행시 너무 큰 데이터셋을 드라이버에 전송하려고 하는 경우 발생 가능성 있음
  - 브로드캐스트 조인 확인
  - Spark의 최대 브로드케스트 조인 설정을 통해 조인 크기를 제어할 수 있음
  - 드라이버의 가용 메모리 증가
  - JVM 메모리 부족 현상은 언어가 달라 데이터 변환 과정에서 주로 발생함
  - 다른 사용자와 SparkContext를 공유하는 상황이라면 여러 사용자가 동시에 대량의 데이터를 메모리에 올리지 못하도록 막아야 함
- 익스큐터 outOfMemoeryError 또는 응답 없음  
  - 특정 노드의 느린 태스크가 복구되지 않음
- 의도하지 않은 null 값이 있는 결과 데이터
  - 트랜스포메이션이 실행된 결과에 의도치 않은 null값 발생
  - 비즈니스 로직 변경이 안되었다면 데이터 포멧 확인
  - 어큐물레이터로 정상과 비정상 레코드 수를 확인할 수 있음. 결과에 따라 맞는 명령 수행 가능
  - 트렌스포매이션이 실제 유효한 쿼리 실행 계획을 생성하는지 확인
- 디스크 공간 없음 오류(no space left on disk)
- 투기적 실행(speculative execution) --> 하나의 Job을 중복된 태스크로 구성하여 동시에 수행시키는 작업   

## Methods, function 
- `spark.sql` : SQL 쿼리 실행
 
