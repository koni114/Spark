# Dataset
- Dataset은 구조적 API의 기본 데이터 타입  
  DataFrame은 Row 타입의 Dataset(`DataFrame = Dataset[Row]`)임
- Spark가 지원하는 다양한 언어에서 사용할 수 있음  
  Dataset은 자바 가상 머신을 사용하는 언어인 <b>스칼라와 자바</b>에서만 사용 가능
- Dataset을 사용해 데이터셋의 각 로우를 구성하는 객체를 정의함
- 스칼라에서는 스키마가 정의된 케이스 클래스 객체를 사용해 Dataset을 정의함
- 자바에서는 java bean 객체를 사용해 Dataset을 정의
- Spark 유경험자들은 <b>'타입형 API'</b>라고 부르기도 함
- Spark는 `StringType`, `BigIntType`, `StructType`과 같은 다양한 데이터 타입을 제공
- Spark가 지원하는 다양한 언어의 String, Integer, Double과 같은 데이터 타입을 Spark의 특정 데이터 타입으로 매핑할 수 있음
- DataFrame API를 사용할 때 String 이나 Integer 데이터 타입의 객체를 생성하지는 않지만, Spark는 Row 객체를 변환해 데이터를 처리
- 사실 스칼라는 자바를 사용할 때 <b>모든 DataFrame은 Row 타입의 Dataset을 의미함</b>
- 도메인별 특정 객체를 효과적으로 지원하기 위해 '인코더(encoder)'라 부르는 특수 개념이 필요
- 인코더는 도메인별 특정 객체 T를 스파크의 내부 데이터 타입으로 매핑하는 시스템을 의미함
- 예를 들어 `name(string)`과 `age(int)`의 두 개의 필드를 가진 Person 클래스가 있다고 가정해보자  
인코더는 런타임 환경에서 Person 객체를 바이너리 구조로 직렬화하는 코드를 생성하도록 Spark에 지시
- DataFrame이나 '표준' 구조적 API를 사용한다면 Row 타입을 직렬화된 바이너리 구조로 변환
- 도메인에 특화된 객체를 만들어 사용하려면 스칼라의 케이스 클래스 또는 자바의 자바빈(JavaBean) 형태로 사용자 정의 데이터 타입을 정의해야 함
- Spark에서는 Row 타입 대신 사용자가 정의한 데이터 타입을 분산 방식으로 다룰 수 있음
- Dataset API를 사용한다면 Spark는 Dataset에 접근할 때마다 Row 포맷이 아닌 사용자 정의 데이터 타입으로 변환함
- 이 변환 작업은 느리긴 하지만 사용자에게 더 많은 유연성을 제공할 수 있음
- <b>사용자 정의 데이터 타입을 사용하면 성능이 느려지게 됨</b>
- <b>그 이유는 프로그래밍 언어를 전환하는 것이 사용자 정의 데이터 타입을 사용하는 것보다 훨씬 더 느리기 때문</b>

## 11.1 Dataset을 사용할 시기
- Dataset을 사용하면 성능이 떨어진다는데 사용할 필요가 있을까? 라는 생각이 들 수 있지만 크게 2가지 이유에서 사용
  - DataFrame 기능만으로는 수행할 연산을 표현할 수 없는 경우
  - 성능 저하를 감수하더라도 타입 안정성(type-safe)를 가진 데이터 타입을 사용하고 싶은 경우
- Dataset을 사용해야 하는 이유를 좀 더 자세히 알아보자
- 구조적 API를 사용해 표현할 수 없는 몇 가지 작업이 있음
- 흔하진 않지만, 복잡한 비즈니스 로직을 SQL이나 DataFrame대신 단일 함수로 인코딩해야 하는 경우가 있음. 이런 경우 Dataset을 사용하는 것이 적합
- 또한 Dataset API는 타입 안정성이 있음. 예를 들어 두 문자열을 사용해 뺄셈 연산을 하는 것처럼 데이터 타입이 유효하지 않은 작업은 런타임이 아닌 컴파일 타임에 오류가 발생
- <b>만약 정확도와 방어적 코드를 가장 중요시한다면, 성능을 조금 희생하더라도 Dataset을 사용하는 것이 가장 좋은 선택이 될 수 있음</b>
- Dataset API를 사용하면 잘못된 데이터로부터 애플리케이션을 보호할 수는 없지만 보다 우아하게 데이터를 제어하고 구조화할 수 있음
- 단일 노드의 워크로드와 Spark 워크로드에서 전체 로우에 대한 다양한 트랜스포메이션을 재사용하려면 Dataset을 사용하는 것이 적합
- Scala를 사용해본 경험이 있다면 Spark의 API는 스칼라 Sequence 타입의 API가 일부 반영되어 있지만 분산 방식으로 동작한다는 것을 알 수 있음
- <b>결국 Dataset을 사용하는 장점 중 하나는 로컬과 분산 환경의 워크로드에서 재사용할 수 있다는 것</b>
- 케이스 클래스로 구현된 데이터 타입을 사용해 모든 데이터와 트랜스포메이션을 정의하면 재사용할 수 있음
- 또한 올바른 클래스와 데이터 타입이 지정된 DataFrame을 로컬 디스크에 저장하면 다음 처리 과정에서 사용할 수 있어 쉽게 데이터를 다룰 수 있음
- 더 적합한 워크로드를 만들기 위해 DataFrame과 Dataset을 동시에 사용해야 할 때가 있음
- 하지만 성능과 타입 안정성 중 하나는 반드시 희생할 수밖에 없음
- 이러한 방식은 대량의 DataFrame 기반의 ETL 트랜스포메이션의 마지막 단계에서 사용할 수 있음
- 예를 들어 드라이버로 데이터를 수집해 단일 노드의 라이브러리로 수집된 데이터를 처리하는 경우임
- 반대로 트랜스포메이션의 첫 번째 단계에서 사용할 수도 있음. 예를들어 Spark SQL에서 필터링 전에 로우 단위로 데이터를 파싱하는 경우

## 11.2 Dataset 생성
- Dataset을 생성하는 것은 수동 작업이므로 정의할 스키마를 미리 알고 있어야 함

### 11.2.1 자바: Encoders
- 자바 인코더는 매우 간단. 데이터 타입 클래스를 정의한 다음 DataFrame의 (Dataset<Row> 타입)에 지정해 인코딩할 수 있음

### 11.2.2 스칼라:케이스 클래스
- 스칼라에서 Dataset을 생성하려면 스칼라 `case class`구문을 사용해 데이터 타입을 정의해야 함
- 케이스 클래스는 다음과 같은 특징을 가진 정규 클래스(regular class)임
  - 불변성
  - 패턴 매칭으로 분해 가능
  - 참조값 대신 클래스 구조를 기반(값)으로 비교
  - 사용하기 쉽고 다루기 편함
- 이러한 특징으로 케이스 클래스를 판별할 수 있으므로 데이터 분석시 유용 
- 케이스 클래스는 불변성을 가지며 값 대신 구조로 비교할 수 있음
- 스칼라 문서는 케이스 클래스를 다음과 같이 설명함
  - 불변성이므로 객체들이 언제 어디서 변경되었는지 추적할 필요가 없음
  - 값으로 비교하면 인스턴스를 마치 원시 데이터 타입의 값처럼 비교.  
    그러므로 클래스 인스턴스가 값으로 비교되는지, 참조로 비교되는지 더는 불확실해하지 않아도 됨
  - 패턴 매칭은 로직 분기를 단순화해 버그를 줄이고 가독성을 좋게 만듬
- Spark에서 케이스 클래스를 사용할 때도 이러한 장점은 유지됨
- Dataset을 생성하기 위해 예제 데이터셋 중 하나를 case class로 정의
~~~scala
case class Flight(DEST_COUNTRY_NAME:String, ORIGIN_COUNTRY_NAME:String, count: BigInt)
~~~
- 앞서 데이터셋의 레코드를 표현할 case class를 정의함. 즉, Flight 데이터 타입의 Dataset 생성
- Flight 데이터 타입은 스키마만 정의되어 있을 뿐 아무런 메서드도 정의되어 있지 않음
- 데이터를 읽으면 DataFrame이 반환. 그리고 as 메서드를 사용해 Flight 데이터 타입으로 변환함
~~~scala
val flightsDF = spark.read.parquet("C:/Spark-The-Definitive-Guide-master/data/flight-data/parquet/2010-summary.parquet/")
val flights = flightsDF.as[Flight]
~~~

## 11.3 액션
- 지금까지 Dataset의 강력한 힘을 보았음
- 하지만 Dataset과 DataFrame에 `collect`, `take`, `count` 같은 액션을 적용할 수 있다는 사실이 더 중요하니 반드시 기억하자
~~~scala
flights.show(2)
~~~
- 또한 케이스 클래스에 실제로 접근할 때 어떠한 데이터 타입도 필요하지 않다는 사실을 알고 있어야 함
- case class의 속성명을 지정하면 속성에 맞는 값과 데이터 타입 모두를 반환함
~~~scala
flights.first.DEST_COUNTRY_NAME // United States

// 결과
res2: String = United States
~~~

## 11.4 트랜스포메이션
- Dataset의 트랜스포메이션은 DataFrame과 동일함. DataFrame의 모든 트랜스포메이션은 Dataset에서 사용할 수 있음
- Dataset을 사용하면 원형의 JVM 데이터 타입을 다루기 때문에 DataFrame만 사용해서 트랜스포메이션을 수행하는 것보다 좀 더 복잡하고 강력한 데이터 타입으로 트랜스포메이션을 사용 할 수 있음
- 원형 객체를 다루는 방법을 설명하기 위해 이전 예제에서 만든 Dataset에 필터를 적용해보자

### 11.4.1 필터링
- `Flight` 클래스를 파라미터로 사용해 불리언값을 반환하는 함수를 만들어보자
- 불리언 값은 출발지와 도착지가 동일한지 나타냄. 이 함수는 사용자 정의 함수(Spark SQL은 예제와 같은 방식으로 사용자 정의 함수를 정의함)가 아닌 일반 함수
- 다음 예제에서는 필터가 정의된 함수를 생성. 이 방식은 <b>지금까지 했던 방식과 다르므로 매우 중요</b>. Spark는 정의된 함수를 사용해 모든 로우를 평가
- 따라서 매우 많은 자원을 사용함. 그러므로 단순 필터라면 SQL 표현식을 사용하는 것이 좋음. SQL 표현식을 사용하면 데이터 필터링 비용이 크게 줄어들 뿐만 아니라 다음 처리 과정에서 Dataset으로 데이터를 다룰 수 있음
~~~scala
def originIsDestination(flight_row: Flight):Boolean = {
  return flight_row.ORIGIN_COUNTRY_NAME == flight_row.DEST_COUNTRY_NAME
}
~~~
- 위의 정의된 함수를 `filter` 메서드에 적용해 각 행이 true를 반환하는지 평가하고 데이터셋을 필터링할 수 있음
~~~scala
flights.filter(flight_row => originIsDestination(flight_row)).first()

// 결과
 Flight = Flight(United States,United States,348113)
~~~
- 이 함수는 Spark 코드에서 호출하지 않아도 됨
- Spark 코드에서 사용하기 전에 사용자 정의 함수처럼 로컬 머신의 데이터를 대상으로 테스트 할 수 있음
- 예를 들어 다음 예제의 데이터셋은 드라이버에 모을 수 있을 만큼 아주 작기 때문에 동일한 필터 연산을 수행할 수 있음
~~~scala
flights.collect().filter(flight_row => originIsDestination(flight_row))
~~~
- 결과는 다음과 같음
~~~scala
Array[Flight] = Array(Flight(United States,United States,348113))
~~~
- 함수를 사용했을 때와 동일한 결과가 반환됨

### 11.4.2 매핑
- 필터링은 단순한 트랜스포메이션. 때로는 특정 값을 다른 값으로 매핑해야 함
- 이전 예제에서 함수를 정의하면서 이미 매핑 작업을 해봤음
- 정의한 함수는 Flight 데이터 타입을 입력으로 사용해 불리언값을 반환함
- 하지만 실무에서는 값을 추출하거나 값을 비교하는 것과 같은 정교한 처리가 필요
- Dataset을 다루는 가장 간단한 예제는 로우의 특정 컬럼값을 추출하는 것
- DataFrame에서 매핑 작업을 수행하는 것은 Dataset의 `select`메서드를 사용하는 것과 같음
- 다음은 목적지 컬럼을 추출하는 예제
~~~scala
val destinations = flights.map(f => f.DEST_COUNTRY_NAME)
~~~
- 최종적으로 String 데이터 타입의 Dataset을 반환
- Spark는 결과로 반환할 JVM 데이터 타입을 알고 있기 때문에 컴파일 타임에 데이터 타입의 유효성을 검사할 수 있음
- 드라이버는 결괏값을 모아 문자열 타입의 배열로 반환
~~~scala
val localDestinations = destinations.take(5)
~~~
- 매핑 작업은 사소하고 불필요하다고 생각할 수 있음
- DataFrame을 사용해도 대다수 매핑 작업을 수행할 수 있음 . 사실 매핑 작업보다 더 많은 장점을 얻을 수 있으므로 DataFrame을 사용할 것을 추천
- DataFrame을 사용하면 코드 생성 기능과 같은 장점을 얻음. 하지만 매핑 작업을 사용한다면 훨씬 정교하게 로우 단위로 처리할 수 있음

## 11.5 조인
- DataFrame에서와 마찬가지로 Dataset에서도 동일하게 적용
- 하지만 Dataset은 `joinwith`처럼 정교한 메서드를 제공
- `joinwith` 메서드는 co-group과 거의 유사하며 Dataset 안쪽에 다른 두 개의 중첩된 Dataset으로 구성
- 각 컬럼은 단일 Dataset이므로 Dataset 객체를 컬럼처럼 다룰 수 있음
- 그러므로 조인 수행 시 더 많은 정보를 유지할 수 있으며, 고금 맵이나 필터처럼 정교하게 데이터를 다룰 수 있음
- `joinWith` 메서드를 설명하기 위해 가짜 항공운항 메타데이터 Dataset을 생성
~~~scala
case class FlightMetadata(count:BigInt, randomData:BigInt)
val flightsMeta = spark.range(500).map(x => (x, scala.util.Random.nextLong)).withColumnRenamed("_1", "count").withColumnRenamed("_2", "randomData")
.as[FlightMetadata]

val flights2 = flights.joinWith(flightsMeta, flights.col("count") === flightsMeta.col("count"))
~~~
- 최종적으로 로우는 Flight와 FlightMetadata로 이루어진 일종의 키-값 형태의 Dataset을 반환함
- Dataset이나 복합 데이터 타입의 DataFrame으로 데이터를 조회할 수 있음
~~~scala
flights2.selectExpr("_1.DEST_COUNTRY_NAME")
~~~
- 이전 예제처럼 드라이버로 데이터를 모은 다음 결과를 반환함
~~~scala
flights2.take(2)
~~~
- 일반 조인 역시 잘 작동하지만, DataFrame을 반환하므로 JVM 데이터 타입 정보를 모두 잃게됨
~~~scala
val flights2 = flights.join(flightsMeta, Seq("count"))
~~~
- 이 정보를 다시 얻으려면 다른 Dataset을 정의해야 함
- DataFrame과 Dataset을 조인하는 것은 아무런 문제가 되지 않으며 최종적으로 동일한 결과를 반환
~~~scala
val flights2 = flights.join(flightsMeta.toDF(), Seq("count"))
~~~

## 11.6 그룹화와 집계
- `groupBy`, `rollup`, `cube`메서드를 여전히 사용할 수 있음
- 하지만 Dataset 대신 DataFrame을 반환하기 때문에 데이터 타입 정보를 잃게 됨
~~~scala
flights.groupBy("DEST_COUNTRY_NAME").count()
~~~
- 데이터 타입 정보를 잃는 것은 큰 문제는 아니지만, 이를 유지할 수 있는 그룹화와 집계 방법이 있음
- 한 가지 예로 `groupByKey` 메서드는 Dataset의 특정 키를 기준으로 그룹화하고 형식화된 Dataset을 반환
- 하지만 이 함수는 컬럼명 대신 함수를 파라미터로 사용해야 함
- 따라서 다음 예제와 같이 훨씬 더 정교한 그룹화 함수를 사용할 수 있음
~~~scala
flights.groupByKey(x => x.DEST_COUNTRY_NAME).count()
~~~
- groupByKey 메서드의 파라미터로 함수를 사용함으로써 유연성을 얻을 수 있음
- 하지만 Spark는 함수와 JVM 데이터 타입을 최적화할 수 없으므로 트레이드오프가 발생
- 이로 인해 성능 차이가 발생할 수 있으며 실행 계획으로 그 이유를 확인할 수 있음
- 다음 예제는 `groupByKey` 메서드를 사용해 DataFrame에 새로운 컬럼을 추가한 다음 그룹화 수행 
~~~scala
flights.groupByKey(x => x.DEST_COUNTRY_NAME).count().explain()

// 결과
== Physical Plan ==
*(3) HashAggregate(keys=[value#55], functions=[count(1)])
+- Exchange hashpartitioning(value#55, 200)
   +- *(2) HashAggregate(keys=[value#55], functions=[partial_count(1)])
      +- *(2) Project [value#55]
         +- AppendColumns <function1>, newInstance(class $line16.$read$$iw$$iw$Flight), [staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, input[0, java.lang.String, true], true, false) AS value#55]
            +- *(1) FileScan parquet [DEST_COUNTRY_NAME#0,ORIGIN_COUNTRY_NAME#1,count#2L] Batched: true, Format: Parquet, Location: InMemoryFileIndex[file:/C:/Spark-The-Definitive-Guide-master/data/flight-data/parquet/2010-summar..., PartitionFilters: [], PushedFilters: [], ReadSchema: struct<DEST_COUNTRY_NAME:string,ORIGIN_COUNTRY_NAME:string,count:bigint>
~~~
- Dataset의 키를 이용해 그룹화를 수행한 다음 다음 결과를 키-값 형태로 함수에 전달해 원시 객체 형태로 그룹화된 데이터를 다룰 수 있음
~~~scala
def grpSum(countryName:String, values: Iterator[Flight]) = {
  values.dropWhile(_.count < 5).map(x => (countryName, x))
}
flights.groupByKey(x => x.DEST_COUNTRY_NAME).flatMapGroups(grpSum).show(5)

def grpSum2(f:Flight):Integer ={
  1
}
flights.groupByKey(x => x.DEST_COUNTRY_NAME).mapValues(grpSum2).count().take(5)
~~~


## 용어 정리
- 직렬화(Serialization)
  - 데이터 스토리지 문맥에서 데이터 구조나 오브젝트 상태를 동일하거나 다른 컴퓨터 환경에 저장하고, 나중에 재구성할 수 있는 포맷으로 변환하는 과정 