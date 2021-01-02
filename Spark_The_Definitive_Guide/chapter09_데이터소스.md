## chapter09 데이터소스
- 이 장에서는 스파크에서 기본으로 지원하는 데이터소스뿐만 아니라 훌륭한 커뮤니티에서 만들어낸 수많은 데이터소스를 소개함
- Spark는 6가지 핵심 데이터소스와 커뮤니티에서 만든 수백 가지 외부 데이터소스가 있음 
- 다양한 데이터소스를 읽고 쓰는 능력과 데이터소스를 커뮤니티에서 자체적으로 만들어내는 능력은 스파크를 가장 강력하게 만들어주는 힘이라고 할 수 있습니다
- 다음은 Spark의 핵심 데이터소스임
  - CSV
  - JSON
  - 파케이
  - ORC
  - JDBC/ODBC 연결
  - 일반 텍스트 파일
- Spark에는 커뮤니티에서 만든 수많은 데이터소스가 존재. 그 중 일부는 다음과 같음
  - 카산드라
  - HBase
  - 몽고디비
  - AWS Redshift
  - XML
  - 기타 수많은 데이터소스
- 이 장의 목표는 핵심 데이터소스를 이용해 데이터를 읽고 쓰는 방법을 터득하고,  
  서드파티 데이터소스와 스파크를 연동할 때 무엇을 고려해야 하는지 배우는 것 

### 9.1 데이터소스 API의 구조
- 특정 포맷을 읽고 쓰는 방법을 알아보기 전에 데이터소스 API의 전체적인 구조에 대해서 알아보자

#### 9.1.1 읽기 API의 구조
- 데이터 읽기의 핵심 구조는 다음과 같음
~~~scala
DataFrameReader.format(...).option("key", "value").schema(...).load()
~~~
- 모든 데이터소스를 읽을 때 위와 같은 형식을 사용
- `format` 메서드는 선택적으로 사용할 수 있으며 기본값은 파케이
- 그리고 `option` 메서드를 사용해 데이터를 읽는 방법에 대한 파라미터를 키-값 쌍으로 설정 가능
- 마지막으로 `schema` 메서드는 데이터 소스에서 스키마를 제공하거나, 스키마 추론 기능을 사용하려는 경우에 선택적으로 사용할 수 있음
- 당연히 데이터 포맷별로 필요한 몇 가지 옵션 존재

#### 9.1.2 데이터 읽기의 기초
- Spark에서 데이터를 읽는 경우에는 기본적으로 `DataFrameReader`를 사용
- `DataFrameReader`는 sparkSession의 read 속성으로 접근
~~~scala
spark.read
~~~
- `DataFrameReader`를 얻은 다음에는 다음과 같은 값을 지정해야 함
  - 포맷
  - 스키마
  - 읽기 모드
  - 옵션
- 포맷, 스키마, 그리고 옵션은 트랜스포메이션을 추가로 정의할 수 있는 DataFrameReader를 반환함
- 읽기모드를 제외한 세 가지 항목은 필요한 경우에만 선택적으로 지정할 수 있음
- 데이터소스마다 데이터를 읽는 방식을 결정할 수 있는 옵션 제공
- 사용자는 반드시 DataFrameReader에 데이터를 읽을 경로를 지정해야 함
- 전반적인 코드 구성은 다음과 같음
~~~
spark.read.format("csv")
.option("mode", "FAILFAST")
.option("inferschema", "true")
.option("path", "path/to/file(s)")
.schema(someSchema)
.load()
~~~ 
- 옵션을 설정할 수 있는 다양한 방법이 존재. 예를 들어 설정값을 가진 맵 객체를 전달할 수 있음
- 하지만 당분간의 위의 포맷을 사용하자

#### 읽기모드
- 외부 데이터소스에서 데이터를 읽다 보면 자여느레 형식에 맞지 않는 데이터를 만나게 됨 
- 특히 반정형 데이터소스를 다룰 때 많이 발생 
- 읽기 모드는 스파크가 형식에 맞지 않는 데이터를 만났을 때 동작 방식을 지정하는 옵션
- 다음은 읽기 모드의 종류를 나타냄
  - permissive  
    - 오류 레코드의 모든 필드를 null로 설정하고 모든 오류 레코드를 `_corrupt_record`라는 문자열 컬럼에 기록
  - dropMalformed
    - 형식에 맞지 않는 레코드가 포함된 로우 제거
  - failFast
    - 형식에 맞지 않는 레코드를 만나면 즉시 종료

#### 9.1.3 쓰기 API 구조
- 데이터 쓰기의 핵심 구조는 다음과 같음
~~~scala
DataFrameWriter.format(...).option(...).partitionBy(...).bucketBy(...).sortBy(...).save()
~~~
- 모든 데이터소스에 데이터를 쓸 때 위와 같은 포맷 사용
- `format` 메서드는 선택적으로 사용가능하며 기본값은 파케이 포맷
- 그리고 `option` 메서드를 사용해 데이터 쓰기 방법을 설정할 수 있음
- `partitionBy`, `bucketBy`, `sortBy` 메서드는 파일 기반의 데이터소스에서만 동작하며  
  이 기능으로 최종 파일 배치 형태(layout)을 제어할 수 있음

#### 9.1.4 데이터 쓰기의 기초
- 데이터 쓰기는 읽기와 매우 유사하며, DataFrameReader 대신 DataFrameWriter 사용
- 데이터소스에 데이터를 기록해야 하기 때문에 DataFrame의 write 속성을 이용해 DataFrame별로 DataFrameWriter에 접근해야 함
~~~scala
dataFrame.write
~~~
- DataFrameWriter를 얻은 다음에는 포맷, 옵션, 저장 모드를 지정해야하며, 데이터가 저장될 경로를 반드시 지정해야함
- 데이터소스별로 다양한 옵션이 존재하며, 개별 데이터소스를 다룰 때 자세히 알아보자
~~~scala
dataFrame.write.format("csv")
.option("mode", "OVERWRITE")
.option("dateFormat", "yyyy-MM-dd")
.option("path", "path/to/file(s)")
.save()
~~~

#### 저장모드
- 저장 모드는 스파크가 지정된 위치에서 동일한 파일이 발견됐을 때 동작 방식을 지정하는 옵션
- 저장 모드 종류는 다음과 같음
  - append
    - 해당 경로에 이미 존재하는 파일 목록에 결과 파일을 추가
  - overwrite
    - 이미 존재하는 모든 데이터를 완전히 덮어씀 
  - errorIfExists
    - 해당 경로에 데이터나 파일이 존재하는 경우 오류를 발생시키면서 쓰기 작업이 실패
  - ignore
    - 해당 경로에 데이터나 파일이 존재하는 경우 아무런 처리도 하지 않음
- <b>기본값은 `errorIfExists` 임</b>

### 9.2 CSV 파일
- CSV는 콤마(,)로 구분된 값을 의미함. CSV는 각 줄이 단일 레코드가 되며  
  레코드의 각 필드를 콤마로 구분하는 일반적인 텍스트 파일 포멧
- CSV 파일은 구조적으로 보이지만, 가장 까다로운 파일 포맷 중 하나
- 그 이유는 운영 환경에서는 어떤 내용이 들어 있는지, 어떠한 구조로 되어있는지 등 다양한  
  전제를 만들어낼 수 없기 때문
- 그러한 이유로 CSV reader는 많은 수의 옵션을 만들어냄
- 예를 들어 CSV 파일 내 컬럼 내용에 콤마가 들어가 있거나 비표준적인 방식으로 null 값이 기록된 경우 특정 문자를 이스케이프 처리하는 옵션을 사용해 문제 해결 가능

#### 9.2.1 CSV 옵션
> |읽기/쓰기|  키 | 사용 가능한 값 | 가본값 | 설명
> |---|:---:|:---:|:---:|:---:|
> |모두|sep|단일 문자| , | 각 필드와 값을 구분하는데 사용되는 단일 문자   
> |모두|header|true, false| false | 첫 번째 줄이 컬럼명인지 나타내는 불리언값
> |읽기|escape|모든 문자열|"\ "  |Spark가 파일에성 이스케이프 처리할 문자|
> |읽기|inferSchema|true,false|false|Spark가 파일을 읽을 때 컬럼의 데이터 타입을 추론할 지 정의
> |읽기|ignoreLeadingWhiteSpace|true, false|false|값을 읽을 때 선행 공백 제거 여부|
> |읽기|ignoreTrailingWhiteSpace|true, false|false|값을 읽을 때 후행 공백 무시 여부|
> |모두|nullValue|모든문자열|""|파일에서 null값을 나타내는 문자|
> |모두|nanValue|모든문자열|NaN|CSV 파일에서 NaN이나 값없음을 나타내는 문자 선언|
> |모두|positiveInf|모든 문자열 또는 문자|Inf|양의 무한 값을 나타내는 문자를 선언|
> |모두|negativeInf|모든 문자열 또는 문자|-Inf|음의 무한 값을 나타내는 문자열을 선언|
> |모두|compression 또는 codec|none. uncompressed. bzip2, deflate..|none|Spark가 파일을 읽고 쓸때 사용하는 압축 코덱을 정의|
> |모두|dateFormat|SimpleDateFormat을 따르는 모든 문자열 또는 문자|yyyy-MM-dd|날짜 데이터 타입인 모든 필드에서 사용할 날짜 형식|
> |모두|timeStampFormat|SimpleDateFormat을 따르는 모든 문자열 또는 문자|yyyy-MM-dd HH:mm:ss|타임스탬프 데이터 타입인 모든 필드에서 사용할 날짜 형식|
> |읽기|maxColumns|모든 정수|20480|파일을 구성하는 최대 컬럼 수를 선언|
> |읽기|maxCharPerColumn|모든 정수|1000000|컬럼의 문자 최대 길이를 선언|
> |읽기|escapeQuotes|true, false|true|Spark가 파일의 라인에 포함된 인용부호를 이스케이프 할지 선언|
> |읽기|maxMalformedLogPerPartition|모든 정수|10|스파크가 각 파티션별로 비정상적인 레코드를 발견했을 때 기록할 최대 수, 이 숫자를 초과하는 비정상적인 레코드는 무시됨|
> |쓰기|quoteAll|true, false|false|인용부호 문자가 있는 값을 이스케이프 처리하지 않고, 전체 값을 인용 부호로 묶을지 여부|
> |읽기|multiline|true, false|false|하나의 논리적 레코드가 여러 줄로 이루어진 CSV 파일 읽기를 허용 할지 여부|

#### 9.2.2 CSV 파일 읽기
- 다른 포맷과 마찬가지로 CSV 파일을 읽으려면 먼저 CSV용 DataFrameReader를 생성해야 하며 예제는 다음과 같음
~~~scala
spark.read.format("csv")
~~~
- 그다음에는 스키마와 읽기 모드 지정
- 이제 몇 가지 옵션을 지정해 보겠음
~~~scala
dataFrame.write.format("csv")
.option("header", "true")
.option("mode", "FAILFAST")
.option("inferschema", "true")
.load("some/path/to/file.csv")
~~~
- 앞서 언급했듯이 비정상적인 데이터를 얼마나 수용할 수 있을지 읽기 모드로 지정할 수 있음
- 예를 들어 다음과 같이 읽기 모드와 5장에서 생성한 스키마를 파일의 데이터가 예상한 형태로 이루어져 있음을 검증하는 용도로 사용할 수 있음
~~~scala
import org.apache.spark.sql.types.{StructField, StructType, StringType, LongType}
val myManualSchema = new StructType(Array(
    new StructField("DEST_COUNTRY_NAME", StringType, true),
    new StructField("ORIGIN_COUNTRY_NAME", StringType, true),
    new StructField("count", LongType, false)
))

spark.read.format("csv")
.option("header", "true")
.option("mode", "FAILFAST")
.schema(myManualSchema)
.load("C:/Spark-The-Definitive-Guide-master/data/flight-data/csv/2010-summary.csv")
.show(5)
~~~
- 다행히도 위의 예제는 잘 동작함
- 데이터가 기대했던 데이터 포맷이 아니였다면 문제가 발생했을 것임
- 예를 들어 현재 스키마의 모든 컬럼의 데이터 타입을 LongType으로 변경
- <b>실제 스키마와 일치하지 않지만 Spark는 어떤 문제도 찾아내지 못함</b> 
- 문제는 Spark가 실제로 데이터를 읽어 들이는 시점에 발생
- 데이터가 지정된 스키마에 일치하지 않으므로 스파크 잡은 시작하자마자 종료
~~~
import org.apache.spark.sql.types.{StructField, StructType, StringType, LongType}
val myManualSchema = new StructType(Array(
    new StructField("DEST_COUNTRY_NAME", LongType, true),
    new StructField("ORIGIN_COUNTRY_NAME", LongType, true),
    new StructField("count", LongType, false)
))

spark.read.format("csv")
.option("header", "true")
.option("mode", "FAILFAST")
.schema(myManualSchema)
.load("C:/Spark-The-Definitive-Guide-master/data/flight-data/csv/2010-summary.csv")
.show(5)
~~~
- Spark는 <b>지연 연산</b> 특성이 있으므로 DataFrame 정의 시점이 아닌 잡 실행 시점에만 오류가 발생
- 예를 들어 DataFrame을 정의하는 시점에는 존재하지 않는 파일을 지정해도 오류가 발생하지 않음

### 9.2.3 CSV 파일 쓰기
- 데이터 읽기와 마찬가지로 CSV 파일을 쓸 때 사용할 수 있는 다양한 옵션이 있음
- 예제는 다음과 같음
~~~
val csvFile = spark.read.format("csv")
.option("header", "true")
.option("mode", "FAILFAST")
.schema(myManualSchema)
.l
oad("C:/Spark-The-Definitive-Guide-master/data/flight-data/csv/2010-summary.csv")

csvFile.write.format("csv").mode("overwrite").option("sep", "\t").save("test.csv")
~~~
- 데이터를 쓰는 시접에 DataFrame의 파티션 수를 반영
- 만약 사전에 데이터를 분할했다면 파일 수가 달라졌을 것임


### 용어 정리
