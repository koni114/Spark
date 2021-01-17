## chapter09 데이터소스
- 이 장에서는 스파크에서 기본으로 지원하는 데이터소스뿐만 아니라 훌륭한 커뮤니티에서 만들어낸 수많은 데이터소스를 소개함
- Spark는 6가지 핵심 데이터소스와 커뮤니티에서 만든 수백 가지 외부 데이터소스가 있음 
- 다양한 데이터소스를 읽고 쓰는 능력과 데이터소스를 커뮤니티에서 자체적으로 만들어내는 능력은 스파크를 가장 강력하게 만들어주는 힘이라고 할 수 있음
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
- 외부 데이터소스에서 데이터를 읽다 보면 자연스레 형식에 맞지 않는 데이터를 만나게 됨 
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

### 9.2 CSV(Comma-Seperated Values) 파일
- CSV는 콤마(,)로 구분된 값을 의미함. CSV는 각 줄이 단일 레코드가 되며  
  레코드의 각 필드를 콤마로 구분하는 일반적인 텍스트 파일 포멧
- CSV 파일은 구조적으로 보이지만, 가장 까다로운 파일 포맷 중 하나
- 그 이유는 운영 환경에서는 어떤 내용이 들어 있는지, 어떠한 구조로 되어있는지 등 다양한  
  전제를 만들어낼 수 없기 때문
- 그러한 이유로 CSV reader는 많은 수의 옵션을 만들어냄
- 예를 들어 CSV 파일 내 컬럼 내용에 콤마가 들어가 있거나 비표준적인 방식으로 null 값이 기록된 경우 특정 문자를 이스케이프 처리하는 옵션을 사용해 문제 해결 가능

#### 9.2.1 CSV 옵션
> |읽기/쓰기|  키 | 사용 가능한 값 | 기본값 | 설명
> |---|:---:|:---:|:---:|:---:|
> |모두|sep|단일 문자| , | 각 필드와 값을 구분하는데 사용되는 단일 문자   
> |모두|header|true, false| false | 첫 번째 줄이 컬럼명인지 나타내는 불리언값
> |읽기|escape|모든 문자열|"\ "  |Spark가 파일에서 이스케이프 처리할 문자|
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
~~~scala
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
~~~scala
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
- 이 장 마지막 부분에서 이러한 방식의 트레이드오프를 알아볼 것임

## JSON 파일
- 자바스크립트 세상에서 온 파일 형식들은 자바스크립트 객체 표기법, 즉 JSON(JavaScript Object Notation)으로 더 친숙하게 알려져 있음
- 이런 종류의 데이터를 다룰 때 조심해야 할 사항들을 먼저 알아보자
- Spark는 JSON 파일을 사용할 때 <b>줄로 구분된 JSON</b>을 기본적으로 사용
- 다음은 줄로 구분된 JSON Format 예시
~~~JSON
{"ORIGIN_COUNTRY_NAME":"Romania","DEST_COUNTRY_NAME":"United States","count":1}
{"ORIGIN_COUNTRY_NAME":"Ireland","DEST_COUNTRY_NAME":"United States","count":264}
~~~
- 이런 방식은 큰 JSON 객체나 배열을 하나씩 가지고 있는 파일을 다루는 것과는 대조적인 부분
- multiLine 옵션을 사용해 줄로 구분된 방식과 여러 줄로 구성된 방식을 선택적으로 사용할 수 있음
- 이 옵션을 true로 설정하면 전체 파일을 하나의 JSON 객체로 읽을 수 있음
- 스파크는 JSON 파일을 파싱한 다음에 DataFrame을 생성함
- 줄로 구분된 JSON은 전체 파일을 읽어 들인 다음 저장하는 방식이 아니므로 새로운 레코드를 추가할 수 있음
- 다른 포맷에 비해 훨씬 더 안정적인 포맷이므로 이 방식을 사용하는 것이 좋음
- 줄로 구분된 JSON이 인기 있는 또 다른 이유는 구조화되어 있고, 최소한의 기본 데이터 타입이 존재하기 때문(JSON은 자바스크립트를 토대로 만들어짐)
- 따라서 Spark는 적합한 데이터 타입을 추정할 수 있어 원할하게 처리 가능
- JSON은 객체이기 때문에 CSV보다 옵션 수가 적음

### JSON 옵션
> |읽기/쓰기|  키 | 사용 가능한 값 | 기본값 | 설명
> |---|:---:|:---:|:---:|:---:|
> |모두|compression 또는 codec|none, uncompressed, bzip2, deflate, gzip, lz4, snappy| none | 스파크가 파일을 읽고 쓸 때 사용하는 압축 코덱 정의  
> |모두|dateFormat|자바의 SimpleDateFormat을 따르는 모든 문자열 또는 문자| yyyy-MM-dd | 날짜 데이터 타입인 모든 필드에서 사용할 날짜 형식을 정의함
> |모두|timestampFormat|자바의 SimpleDateFormat을 따르는 모든 문자열 또는 문자|yyyy-MM-dd HH:mm:SS  |타임스탬프 데이터 타입인 모든 필드에서 사용할 날짜 형식을 정의|
> |읽기|primitiveAsString|true, false|false|모든 primitive값을 문자열로 추정할지 정의
> |모두|allowComments|true, false|false|JSON 레코드에서 자바나 C++ 스타일로 된 코멘트를 무시할지 정의함  
> |모두|allowUnquotedFieldNames|true, false| false |인용부호로 감싸여 있지 않는 JSON 필드명을 허용할지 정의
> |읽기|allowSingleQuotes|true, false|  | 인용부호로 큰따옴표(") 대신 작은따옴표(')를 허용할지 정의
> |읽기|allowNumericLeadingZeros|true,false|false|숫자 앞에 0을 허용할지 정의(ex) 00012)
> |읽기|allowBackslashEscapingAnyCharacter|true,false|false|백슬래시 인용부호 매커니즘을 사용한 인용부호를 허용할지 정의
> |읽기|columnNameOfCorruptRecord|모든 문자열|속성의 설정값|permissive 모드에서 생산된 비정상 문자열을 가진 새로운 필드명을 변경할 수 있음. 이 값을 설정하면 spark.sql.columnNameOfCorruptRecord 설정값 대신 적용됨
> |읽기|multiLine|true,false|false|줄로 구분되지 않는 JSON 파일의 읽기를 허용할지 정의

- 줄로 구분된 JSON 파일을 읽는 방법은 데이터 파일 포맷 설정과 옵션을 지정하는 방식만 다름
~~~scala
spark.read.format("json")
~~~

### 9.3.2 JSON 파일 읽기
- JSON 파일을 읽는 방법과 앞에서 살펴본 옵션을 비교해볼 수 있는 예제를 살펴보자
~~~scala
spark.read.format("json")
.option("mode", "FAILFAST")
.schema(myManualSchema)
.load("/data/flight-data/json /2010-summary.json")
.show(5)
~~~

### 9.3.3 JSON 파일 쓰기
- JSON 파일 쓰기는 읽기와 마찬가지로 간단함
- 데이터소스에 관계없이 JSON 파일에 저장할 수 있음
- 이전에 만들었던 CSV DataFrame을 JSON 파일의 소스로 재사용 할 수 있음 
- 이 작업 역시 이전 규칙 그대로 따름
- 파티션당 하나의 파일을 만들며 전체 DataFrame을 단일 폴더에 저장
- JSON 객체는 한 줄에 하나씩 기록됨
~~~scala
csvFile.write.format("json")
.mode("overwrite")
.save("/tmp/my-json-file.json")
~~~

## 9.4 파케이 파일
- 파케이는 다양한 스토리지 최적화 기술을 제공하는 오픈소스로 만들어진 컬럼 기반의 데이터 저장 방식
- 특히 분석 워크로드에 최적화되어 있음
- 저장소 공간을 절약할 수 있고 전체 파일을 읽는 대신 개별 컬럼을 읽을 수 있으며, 컬럼 기반의 압축 기능 제공
- 특히 Apache Spark와 잘 호환되기 때문에 Spark의 기본 파일 포맷이기도 함
- 파케이 파일은 읽기 연산 시 JSON이나 CSV보다 훨씬 효율적으로 동작하므로 장기 저장용 데이터는 파케이로 저장하는 것이 좋음
- 파케이의 또 다른 장점은 <b>복합 데이터 타입을 지원</b>하는 것. 컬럼이 배열, 맵, 구조체 데이터 타입이라 해도 문제없이 읽고 쓸 수 있음
- 단 CSV에서는 배열을 사용할 수 없음. 파케이를 읽기 포맷으로 지정하는 방법은 다음과 같음
~~~scala
spark.read.format("parquet")
~~~  

### 파케이 파일 읽기
- 파케이는 옵션이 거의 없음. 데이터를 저장할 때 자체 스키마를 사용해 데이터를 저장하기 때문
- 따라서 포맷을 설정하는 것만으로도 충분함
- DataFrame을 표현하기 위해 정확한 스키마가 필요한 경우에만 스키마 설정
- 하지만 이런 작업은 거의 필요가 없음. 그 이유는 CSV 파일에서 `inferSchema`를 사용하는 것과 유사하게 읽는 시점에 스키마를 알 수 있기 때문(스키마-온-리드)
- 파케이 파일은 스키마가 파일 자체에 내장되어 있으므로 추정이 필요 없음. 그러므로 이 방법이 더 효과적임
- 다음은 파케이 파일을 읽는 간단한 예제
~~~scala
spark.read.format("parquet")

spark.read.format("parquet")
.load("/data/flight-data/parquet/2010-summary.parquet").show(5)
~~~

#### 파케이 옵션
- 파케이에는 옵션이 거의 없음. 정확히 2개의 옵션이 존재
- 단 2개의 옵션만 존재하는 이유는 스파크의 개념에 아주 잘 부합하고 알맞게 정의된 명세를 가지고 있기 때문
- 다른 버전 Spark를 사용한다면(구버전) 파케이 파일을 저장할 때는 조심해야 함

> |읽기/쓰기|  키 | 사용 가능한 값 | 기본값 | 설명
> |---|:---:|:---:|:---:|:---:|
> |모두|compression 또는 codec|none, uncompressed, bzip2, deflate, gzip, lz4, snappy| none | 스파크가 파일을 읽고 쓸 때 사용하는 압축 코덱 정의  
> |읽기|mergeSchema|true, false| mergeSchema 속성의 설정값 | 동일한 테이블이나 폴더에 신규 추가된 파케이 파일에 컬럼을 점진적으로 추가할 수 있음. 이러한 기능을 활성화하거나 비활성화하기 위해 옵션을 사용

### 9.4.2 파케이 파일 쓰기
- 파케이 파일 쓰기는 읽기만큼 쉬움. 파일의 경로만 명시하면 됨
- 분할 규칙은 다른 포맷과 동일하게 적용
~~~scala
csvFile.write.format("parquet").mode("overwrite")
.save("/tmp/my-parquet-file.parquet")
~~~

## 9.5 ORC 파일
- ORC는 하둡 워크로드를 위해 설계된 자기 기술적(self-describing)이며 데이터 타입을 인식할 수 있는 컬럼 기반의 파일 포맷
- 이 포맷은 <b>대규모 스트리밍 읽기에 최적화되어 있을 뿐만 아니라,필요한 로우를 신속하게 찾아낼 수 있는 기능이 통합되어 있음</b>
- Spark는 ORC 파일포맷을 효율적으로 사용할 수 있으므로 별도의 옵션 지정 없이 데이터를 읽을 수 있음
- ORC와 파케이의 차이점은 파케이는 Spark에 최적화된 반면, ORC는 하이브에 최적화되어 있음

### 9.5.1 ORC 파일 읽기
~~~scala
spark.read.format("orc").load("/data/flight-data/orc/2010-summary.orc").show(5)
~~~ 

### 9.5.2 ORC 파일 쓰기
- 파일 쓰는 것은 동일한 사용 패턴을 따르며, 포맷을 지정한 다음 파일을 저장함 
~~~scala
csvFile.write.format("orc").mode("overwrite").save("/tmp/my-orc-file.orc")
~~~

## 9.6 SQL Database
- SQL 데이터소스는 매우 강력한 커넥터 중 하나
- 사용자는 SQL을 지원하는 다양한 시스템에 SQL 데이터소스를 연결할 수 있음
- 예를 들어 MySQL, PostgreSQL, Oracle DB에 접속할 수 있음
- 또한 예제에서 사용할 SQLite에도 접속할 수 있음. DB는 원시 파일 형태가 아니므로 고려해야 할 옵션이 더 많음
- 예를 들어 데이터 인증 정보나 접속과 관련된 옵션이 필요
- 그리고 스파크 클러스터에서 데이터 베이스 시스템에 접속 가능한지 네트워크 상태를 확인해야함
- DB를 설정하는 번거로움을 없애고 이 책의 목적에 충실하기 위해 SQLite 실행을 위한 참고용 샘플 제공
- 모든 예제는 분산 환경이 아닌 로컬 머신에서도 충분히 테스트 할 수 있어야 함
- SQLite는 접속 시 사용자 인증이 필요 없음
- <b>DB의 데이터를 읽고 쓰기 위해서는 spark classpath에 DB JDBC 드라이버를 추가하고 적절한 JDBC 드라이버 jar 파일을 제공해야 함</b>
- 예를 들어 PostgreSQL DB에 데이터를 읽거나 쓰려면 다음과 같이 실행함
~~~scala
./bin/spark-shell\
--driver-class-path postgresql-9.4.1207.jar\
---jars postgresql-9.4.1207.jar
~~~
- 다른 데이터소스와 마찬가지로 SQL DB에서 데이터를 읽고 쓸 때 사용할 수 있는 몇가지 옵션이 있음
- JDBC 데이터베이스를 사용할 때 설정할 수 있는 모든 옵션을 제공
  - url  
    - 접속을 위한 JDBC URL
  - dbtable 
    - 읽을 JDBC Table 설정
    - FROM절에 유효한 모든 것을 사용할 수 있음
    - <b>전체 Table 대신, 서브쿼리 사용 가능</b>
  - driver 
    - 지정한 URL 접속할 때, 사용할 JDBC 드라이버 클래스명 지정
  - <b>partitionColumn, lowerBound, upperBound</b>
    - 세 가지 옵션은 항상 같이 지정해야 함
    - numPartitions도 반드시 지정해야 함
    - 다수의 워커에서 병렬로 테이블을 나눠 읽는 방법을 정의함
    - partitionColumn은 반드시 해당 테이블의 수치형 컬럼이어야 함
    - lowerBound, upperBound는 각 파티션의 범위를 결정하는 데 사용
    - 테이블의 모든 로우는 분할되어 반환됨
    - 읽기에만 적용
  - numPartitions
    - 테이블의 데이터를 병렬로 읽거나 쓰기 작업에 사용될 수 있는 최대 파티션 수를 결정
    - 이 속성은 최대 동시 JDBC 연결 수를 결정함
    - 쓰기에 사용되는 파티션 수가 이 값을 초과하는 경우 쓰기 연산 전에 `coalesce`를 실행해 파티션 수를 이 값에 맞게 줄이게 됨 
  - fetchsize
    - 한 번에 얼마나 많은 로우를 가져올지 결정하는 JDBC 패치 크기를 설정함
    - 패치 크기가 작게 설정된 JDBC 드라이버의 성능을 올리는데 도움이 됨(oracle의 경우 패치 크기가 10)
    - 이 옵션은 읽기에만 적용됨
  - batchsize
    - 한 번에 얼마나 많은 로우를 <b>저장</b>할지 결정하는 JDBC 배치 크기 설정
    - 이 옵션은 JDBC 드라이버의 성능을 향상시킬 수 있음. 쓰기에만 적용됨  
    - 기본값은 1000
  - isolationLevel
    - 현재 연결에 적용되는 트랜잭션 격리 수준을 정의
    - 이 값은 JDBC Connection 객체에서 정의하는 표준 트랜잭션 격리 수준에 해당하는 NONE, READ_COMMITED, READ_UNCOMMITED, REPEATABLE_READ, SERIALIZABLE 중 하나가 될 수 있음
    - 기본값은 READ_UNCOMMITED. 이 옵션은 쓰기에만 적용됨
  - truncate
    - JDBC writer 관련 옵션임
  - createTableOptions
  - createTableColumnTypes

### 용어 정리
- JDBC(Java Database Connectivity)
  - Java에서 DB 프로그래밍을 하기위한 API  
  - Database 종류에 상관없음
- JDBC 드라이버
  - DBMS와 통신을 담당하는 자바 클래스
- ODBC(Open Database Connectivity)
  - 어떤 응용프로그램을 사용하는지 관계없이, DB를 자유롭게 사용하기 위해 만든 응용프로그램의 표준 방법
- ORC(Optimized Row Columnar)
  - 칼럼 기반의 파일 저장방식
  - Hadoop, Hive, Pig, Spark 등에 적용가능
  - DB의 Data를 읽을 때 보통 컬럼단위의 데이터를 이용하는 경우가 많음
  - ORC는 컬럼단위로 데이터를 저장하기 때문에 컬럼의 데이터를 찾는 것이 빠르고, 압축효율이 좋음     
- 이스케이프 처리
  - 앞에 `\`를 붙여 특수 문자를 정상적인 문자로 호출하고 싶을 때  
    이스케이프 처리를 한다고 함 