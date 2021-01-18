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
    - `SaveMode`, `Overwrite`가 활성화되면 스파크는 기존 테이블을 삭제하거나 재생성하는 대신 DB의 truncate 명령을 실행함
    - 기본값이 false임
  - createTableOptions
    - 해당 옵션 지정시 테이블 생성 시 특정 테이블의 DB와 파티션 옵션을 설정 할 수 있음
    - 쓰기에만 적용됨
  - createTableColumnTypes
    - 테이블 생성시 기본값 대신 사용할 DB 컬럼 데이터 타입을 정의함
    - ex) name CHAR(64), comments VARCHAR(1024) 등
    - 지정한 데이터 타입은 유효한 스파크 SQL 데이터 타입이어야 함

### 9.6.1 SQL 데이터베이스 읽기
- 다른 데이터소스와 거의 동일
- 포맷과 옵션을 지정한 후 데이터를 읽어들임
~~~scala
val driver = "org.sqlite.JDBC"
val path = "C:/Spark-The-Definitive-Guide-master/data/flight-data/jdbc/my-sqlite.db"
val url = s"jdbc:sqlite:${path}"
val tablename = "flight_info"
~~~
- 접속 관련 속성을 정의한다음, 정상적으로 DB에 접속되는지 테스트해 해당 연결이 유효한지 
확인 가능
- 이것은 Spark driver가 DB에 접속할 수 있는지 확인할 수 있는 훌륭한 문제 해결 기술
- 예를들어, MySQL 같은 DB를 사용하는 경우 다음과 같은 코드를 사용해 접속 테스트 가능
~~~scala
import java.sql.DriverManager
val connection = DriverManager.getConnection(url)
connection.isClosed()
connection.close()
~~~
- 접속에 성공하면 다음 예제 진행 가능. SQL 테이블을 읽어 DataFrame을 만들어 보겠음
~~~scala
val dbDataFrame = spark.read.format("jdbc")
.option("url", url)
.option("dbtable", tablename)
.option("driver", driver)
.load()
~~~
- SQLite는 설정이 간단함. PostgreSQL과 같은 데이터베이스에는 더 많은 설정이 필요
- 다음 예제는 PostgreSQL을 이용해 위 예제와 동일한 데이터 읽기 작업을 수행함
~~~scala
val pgDF = spark.read
.format("jdbc")
.option("driver", "org.postgresql.Driver")
.option("url", "jdbc:postgresql://satabase_server")
.option("dbtable", "schema.tablename")
.option("user", "username").option("password", "my-secret-password")
.load()
~~~  
- 생성한 DataFrame은 기존 예제에서 생성한 DataFrame과 전혀 다르지 않음
- 아무 문제없이 조회, 변환할 수 있으며 이미 스키마가 적용되어 있음 
- Spark는 DB Table에서 스키마 정보를 읽어 테이블에 존재하는 컬럼의 데이터 타입을 Spark의 데이터 타입으로 변환하기 때문
- 원하는 대로 쿼리를 수행할 수 있는지 확인하기 위해 중복 데이터가 제거된 목록 조회해보자
~~~scala
dbDataFrame.select("DEST_COUNTRY_NAME").distinct().show(5)
~~~
- 다음 챕터는 더 진행하기 전에 알아두면 좋은 몇 가지 세부사항에 대해 짚고 넘어가자

### 9.6.2 쿼리 푸시다운
- <b>Spark는 DataFrame을 만들기 전에 DB 자체에서 데이터를 필터링하도록 만들 수 있음</b>
- 예를 들어 이전 예제에서 사용한 쿼리의 실행 계획을 들여다보면 테이블의 여러 컬럼 중 관련 있는 컬럼만 선택한다는 것을 알 수 있음
~~~scala
dbDataFrame.select("DEST_COUNTRY_NAME").distinct().explain

~~~
- Spark는 특정 유형의 쿼리를 더 나은 방식으로 처리할 수 있음
- 예를 들어 <b>DataFrame의 Filter를 명시하면 Spark는 해당 필터에 대한 처리를 DB로 위임(push down)함</b>
- 실행 계획의 PushedFilters 부분에서 관련 내용을 확인할 수 있음
~~~scala
dbDataFrame.filter("DEST_COUNTRY_NAME in ('Anguilla', 'Sweden')").explain
~~~
- Spark는 모든 스파크 함수를 사용하는 SQL DB에 맞게 변환하지는 못함
- 따라서 때로는 전체 쿼리를 DB에 전달해 결과 DataFrame으로 받아야 하는 경우도 있음
- 테이블명 대신 SQL 쿼리를 명시하면 됨
- 이렇게 하려면 괄호로 쿼리를 묶고 이름을 변경하는 특수한 방식을 사용해야 함
- 다음 예제에서는 테이블명을 별칭으로 사용함
~~~scala
val pushdownQuery = """(SELECT DISTINCT(DEST_COUNTRY_NAME) FROM flight_info)
AS flight_info"""

val dbDataFrame = spark.read.format("jdbc")
.option("url", url).option("dbtable", pushdownQuery).option("driver", driver)
.load()
~~~
- 이제 이 테이블에 쿼리할 때 실제로는 `pushdownQuery` 변수에 명시한 쿼리를 사용해 수행
- 이 사실은 실행 계획에서 확인할 수 있음
- <b>Spark는 테이블의 실제 스키마와 관련된 정보를 알지 못하며 단지 쿼리의 결과에 대한 스키마만 알 수 있음</b>
~~~scala
dbDataFrame.explain()
~~~
#### 데이터베이스 병렬로 읽기
- 이 책 전반에 걸쳐 파티셔닝과 데이터 처리시 파티셔닝의 중요성에 관해 이야기 했음
- Spark는 파일 크기, 파일 유형, 압축 방식에 따른 '분할 가능성'에 따라 여러 파일을 읽어 하나의 파티션으로 만들거나 여러 파티션을 하나의 파일로 만드는 기본 알고리즘을 가지고 있음
- 파일이 가진 이런 유연성은 SQL DB에도 존재하지만 몇 가지 수동 설정이 필요
- <b>이전 옵션 목록 중 numPartitions 옵션을 사용해 읽기 및 쓰기용 동시 작업 수를 제한할 수 있는 최대 파티션 수를 설정할 수 있음</b>
~~~scala
val dbDataFrame = spark.read.format("jdbc")
.option("url", url)
.option("driver", driver)
.option("numPartitions", 10)
.load()
~~~
- 지금은 데이터가 그리 많지 않기 때문에 여전히 한 개의 파티션만 존재
- 하지만 이러한 설정을 활용해 DB에 일어날 수 있는 과도한 쓰기나 읽기를 막을 수 있음
~~~scala
dbDataFrame.select("DEST_COUNTRY_NAME").distinct().show()
~~~
- 안타깝게도 다른 API 셋에서만 사용할 수 있는 몇 가지 최적화 방법이 있음
- DB연결을 통해 명시적으로 조건절을 SQL DB에 위임할 수 있음
- 이 최적화 방법은 조건절을 명시함으로써 특정 파티션에 특정 데이터의 물리적 위치를 제어할 수 있음
- 전체 데이터 중 Anguilla와 Sweden 두 국가의 데이터만 필요한데,  
  두 국가에 대한 필터를 DB에 위임해 처리된 결과를 반환할 수도 있지만  
  Spark 자체 파티션에 결과 데이터를 저장함으로써 더 많은 처리를 할수도 있음
- 데이터소스 생성 시 조건절 목록을 정의해 spark 자체 파티션에 결과 데이터를 저장할 수 있음
~~~scala
val props = new java.util.Properties
props.setProperty("driver", "org.sqlite.JDBC")
val predicates = Array(
  "DEST_COUNTRY_NAME" = 'Sweden' OR ORIGIN_COUNTRY_NAME = 'Sweden'",
  "DEST_COUNTRY_NAME" = 'Anguilla' OR ORIGIN_COUNTRY_NAME = 'Anguilla'")
spark.read.jdbc(url, tablename, predicates, props).show()
spark.read.jdbc(url, tablename, predicates, props).rdd.getNumPartitions  
~~~ 
- 연관성 없는 조건절을 정의하면 중복 로우가 많이 발생할 수 있음
- 다음은 중복 로우를 발생시키는 조건절 예제
~~~scala
val props = new java.util.Properties
props.setProperty("driver", "org.sqlite.JDBC")
val predicates = Array(
  "DEST_COUNTRY_NAME" != 'Sweden' OR ORIGIN_COUNTRY_NAME != 'Sweden'",
  "DEST_COUNTRY_NAME" != 'Anguilla' OR ORIGIN_COUNTRY_NAME != 'Anguilla'")
spark.read.jdbc(url, tablename, predicates, props).count()
~~~

#### 슬라이딩 원도우 기반의 파티셔닝
- 조건절을 기반으로 분할할 수 있는 방법을 알아보자
- 예제에서는 수치형 count 컬럼을 기준으로 분할함
- 여기서는 처음과 마지막 파티션 사이의 최솟값과 최댓값을 사용
- 이 범위 밖의 모든 값은 첫 번째 또는 마지막 파티션에 속함
- 그런 다음 전체 파티션 수를 설정 함. 이 값은 병렬 수준을 의미함
- Spark는 DB에 병렬로 쿼리를 요청하며 numPartitions에 설정한 값만큼 파티션을 반환함
- 그리고 파티션에 값을 할당하기 위해 상한값과 하한값을 수정
- 이전 예제와 마찬가지로 필터링은 발생하지 않음
~~~scala
val colName = "count"
val lowerBound = 0L
val upperBound = 348113L // 예제 데이터베이스의 데이터 최대 개수
val numPartitions = 10

spark.read.jdbc(url, tablename, colName, lowerBound, upperBound, numPartitions, props).count() // 255
~~~

### 9.6.3 SQL 데이터베이스 쓰기
- SQL 데이터베이스에 데이터를 쓰는 것은 읽기 만큼이나 쉬움
- URI를 지정하고 지정한 쓰기 모드에 따라 데이터를 쓰면 됨
- 다음 예제에서는 전체 테이블을 덮어쓰는 overwrite 쓰기 모드를 사용함
- 처리를 위해 이전에 정의해놓은 CSV DataFrame을 사용함
~~~scala
val newPath = "jdbc:sqlite://tmp/my-sqlite-db"
csvFile.write.mode("overwrite").jdbc(newPath, tablename, props)
~~~
- 실행 결과는 다음과 같음
~~~scala
spark.read.jdbc(newPath, tablename, props).count()
~~~
- 새로운 테이블에 아주 쉽게 데이터을 추가할 수 있음
~~~scala
csvFile.write.mode("append").jdbc(newPath, tablename, props)
~~~
- 레코드 수 증가 확인
~~~scala
spark.read.jdbc(newPath, tablename, props).count() // 765
~~~

## 9.7 텍스트 파일
- Spark는 일반 텍스트 파일도 읽을 수 있음
- 파일의 각 줄은 DataFrame의 레코드가 됨
- 그러므로 변환하는 것도 마음대로 할 수 있음. Apache log file을 구조화된 포맷으로 파싱하거나, 자연어 처리를 위해 일반 텍스트를 파싱하는 경우를 예로 들 수 있음 
- 텍스트 파일은 기본 데이터 타입의 유연성을 활용할 수 있으므로 Dataset API에서 사용하기 매우 좋은 포맷

### 9.7.1 텍스트 파일 읽기
- 텍스트 파일을 읽는 것은 매우 간단
- `textFile` 메서드에 텍스트 파일을 지정하기만 하면 됨
- `textFile` 메서드는 파티션 수행 결과로 만들어진 디렉터리명을 무시함
- 파티션된 텍스트 파일을 읽거나 쓰려면 읽기 및 쓰기 시 파티션 수행 결과로 만들어진 디렉터리를 인식할 수 있도록 `text` 메서드를 사용해야 함
~~~scala
spark.read.textFile("/data/flight-data/csv/2010-summary.csv")
.selectExpr("split(value, ',') as rows").show()
~~~

### 9.7.2 텍스트 파일 쓰기
- <b>텍스트 파일을 쓸 때는 문자열 컬럼이 하나만 존재해야 함</b>. 그렇지 않으면 작업 실패
~~~scala
csvFile.select("DEST_COUNTRY_NAME").write.text("/tmp/simple-text-file.txt")
~~~ 
- 텍스트 파일에 데이터를 저장할 때 파티셔닝 작업을 수행하면 더 많은 컬럼을 저장할 수 있음
- 하지만 모든 파일에 컬럼을 추가하는 것이 아니라 텍스트 파일이 저장되는 디렉토리에 폴더별로 컬럼을 저장
~~~scala
csvFile.limit(10).select("DEST_COUNTRY_NAME", "count")
.write.partitionBy("count").text("/tmp/file-csv-files2.csv")
~~~

## 9.8 고급 I/O 개념
- 쓰기 작업 전에 파티션 수를 조절함으로써 병렬로 처리할 파일 수를 제어할 수 있음
- 또한 <b>버켓팅</b>과 <b>파티셔닝</b>을 조절함으로써 데이터의 저장 구조를 제어할 수 있음
- 버켓팅과 파티셔닝은 잠시 후에 알아보자

### 9.8.1 분할 가능한 파일 타입과 압축 방식
- 특정 파일 포맷은 기본적으로 분할을 지원함. 따라서 Spark에서 전체 파일이 아닌 쿼리에 필요한 부분만 읽을 수 있으므로 성능 향상에 도움이 됨
- 게다가 하둡 분산 파일 시스템 같은 시스템을 사용한다면 분할된 파일을 여러 블록으로 나누어 분산 저장하기 때문에 훨씬 최적화할 수 있음
- 이와 함께 압축 방식도 관리해야 함. 모든 압축 방식이 분할 압축을 지원하지는 않음
- 데이터를 저장하는 방식에 따라 스파크 잡이 원할하게 동작 하는데 막대한 영향을 끼칠 수 있음
- 추천하는 파일 포맷과 압축 방식은 파케이 파일 포맷과 GZIP 압축 방식

### 9.8.2 병렬로 데이터 읽기
- 여러 익스큐터가 같은 파일을 동시에 읽을 수는 없지만 여러 파일을 동시에 읽을 수는 있음
- 다수의 파일이 존재하는 퐅더를 읽을 때 폴더의 개별 파일은 DataFrame의 파티션이 됨
- 따라서 사용 가능한 익스큐터를 이용해 병렬로 파일을 읽음  
  (익스큐터 수를 넘어가는 파일은 처리 중인 파일이 완료될 때까지 대기)

### 9.8.3 병렬로 데이터 쓰기
- 파일이나 데이터 수는 데이터를 쓰는 시점에 DataFrame이 가진 파티션 수에 따라 달라질 수 있음
- 기본적으로 데이터 파티션당 하나의 파일이 작성됨
- 따라서 옵션에 지정된 파일명은 실제로는 다수의 파일을 가진 디렉터리
- 그리고 디렉터리 안에 파티션당 하나의 파일로 데이터를 저장
- 예를 들어 다음 코드는 폴더 안에 5개의 파일을 생성
~~~scala
csvFile.repartition(5).write.format("csv").save("/tmp/ multiple.csv")
~~~

#### 파티셔닝
- 파티셔닝은 어떤 데이터를 어디에 저장할 것인지 제어할 수 있는 기능
- 파티셔닝된 디렉터리 또는 테이블에 파일을 쓸 때 디렉터리별로 컬럼 데이터를 인코딩해 저장함
- 그러므로 데이터를 읽을 때 전체 데이터셋을 스캔하지 않고 필요한 컬럼의 데이터만 읽을 수 있음
- 이 방식은 모든 파일 기반의 데이터소스에서 지원함
~~~scala
csvFile.limit(10).write.mode("overwrite").partitionBy("DEST_COUNTRY_NAME").save("/tmp/partitioned-files.parquet")
~~~
- 각 폴더는 조건절을 폴더명으로 사용하며 조건절을 만족하는 데이터가 저장된 파케이 파일을 가지고 있음
- 파티셔닝은 필터링을 자주 사용하는 테이블을 가진 경우에 사용할 수 있는 가장 손쉬운 최적화 방식
- 예를 들어 전체 데이터를 스캔하지 않고 지난주 데이터만 보려면 날짜를 기준으로 파티션을 만들 수 있음
- 이 기법을 사용하면 빠른 속도로 데이터를 읽어 들일 수 있음

#### 버켓팅
- 버켓팅(bucketing)은 각 파일에 저장된 데이터를 제어할 수 있는 또 다른 파일 조직화 기법
- 이 기법을 사용하면 동일한 버킷 ID를 가진 데이터가 하나의 물리적 파티션에 모두 모여 있기 때문에 데이터를 읽을 때 셔플을 피할 수 있음
- 즉 데이터가 이후의 사용 방식에 맞춰 사전에 파티셔닝되므로 조인이나 집계 시 발생하는 고비용의 셔플을 피할 수 있음
- 특정 컬럼을 파티셔닝하면 수억 개의 디렉터리를 만들어낼 수도 있음. 이런 경우 데이터를 버켓팅할 수 있는 방법을 찾아야 함
- 다음은 '버켓' 단위로 데이터를 모아 일정 수의 파일로 저장하는 예제
~~~scala
// 기본적으로 /user/hive/warehouse 디렉터리 하위에 버켓팅 파일을 기록함
// 그러므로 user/hive/warehouse 디렉터리를 먼저 생성해야 함
val numberBuckets = 10
val columnToBucketBy = "count"

csvFile.write.format("parquet").mode("overwrite")
.bucketBy(numberBuckets, columnBucketBy).saveAsTable("bucketedFiles")
~~~

### 9.8.4 복합 데이터 유형 쓰기
- Spark는 다양한 자체 데이터 타입을 제공하는데, 이러한 데이터 타입은 Spark에서는 잘 동작하지만 모든 데이터 파일 포맷에 적합한 것은 아님 
- 예를 들어 CSV 파일은 복합 데이터 타입을 지원하지 않지만 파케이나 ORC는 복합 데이터 타입을 지원

### 9.8.5 파일 크기 관리 
- 파일 크기를 관리하는 것은 데이터를 저장할 때는 중요한 요소가 아님
- 하지만 데이터를 읽을 때는 중요
- 작은 파일을 많이 생성하면 메타데이터에 엄청난 관리 부하가 발생
- HDFS 같은 파일 시스템은 작은 크기의 파일을 잘 다루지 못함
- Spark는 특히 더 그런데, 이런 상황을 작은 크기의 파일 문제라고 함
- 하지만 그 반대의 경우도 문제가 됨. 몇 개의 로우가 필요하더라도 전체 데이터 블록을 읽어야 하기 때문에 비효율적임. 즉 너무 큰 파일도 좋지 않음
- Spark 2.2 버전에서는 자동으로 파일 크기를 제어할 수 있는 새로운 방법이 도입됨
- 이전 예제에서 결과 파일 수는 파일을 쓰는 시점에서의 파티션 수에서 파생되었음을 알 수 있었음
- 이제 결과 파일을 최적의 크기로 제한할 수 있는 새로운 기능을 활용해 보자
- 이 기능을 사용하려면 `maxRecordsPerFile` 옵션에 파일당 레코드 수를 지정해야 함
- 각 파일에 기록될 레코드 수를 조절할 수 있으므로 파일 크기를 더 효과적으로 제어할 수 있음
- 만약 파일 쓰기 객체에 다음과 같은 옵션을 설정했다면 Spark는 파일당 최대 5000개의 로우를 포함하도록 보장할 수 있음
~~~scala
df.write.option("maxRecordsPerFile", 5000)
~~~


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
- 트랜잭션 격리 수준(transaction isolation Level)
  - 동시에 여러 트랜잭션이 처리 될 때, 트랜잭션이 다른 트랜잭션이 DB Table 내역을 수정하거나 변경했을 때 볼 수 있도록 허용할지 말지를 결정하는 것
  - 격리 수준은 크게 4가지로 나뉨
    - READ UNCOMMITTED
      - 어떤 트랜잭션의 변경내용이 COMMIT이나 ROLLBACK과 상관없이 다른 트랜잭션에 보여짐
      - ex) A 트랜잭션에서 10번의 사원 나이를 27 -> 28살로 변경  
       아직 커밋하지 않음  
      B 트랜잭션에서 10번 나이를 조회함 --> 28살이 조회됨
      - 데이터 정합성에 매우 문제가 많으므로, RDBMS 표준에서는 격리 수준으로 인정 안해줌
    - READ COMMITTED
      - 트랜잭션의 변경 내용이 COMMIT 되어야만 다른 트랜잭션에서 조회 가능
      - Oracle RDBMS에서 기본으로 사용하고 있고, 온라인 서비스에서 가장 많이 선택되는 격리 수준
      - 위의 예시에서 그대로 27살로 조회됨
      - 하나의 트랜잭션내에서 똑같은 SELECT를 수행했을 경우 항상 같은 결과를 반환해야 한다는 REPEATABLE READ 정합성에 어긋남
    - REPEATABLE READ
      - 트랜잭션이 시작되기 전에 커밋된 내용에 대해서만 조회할 수 있는 격리수준  
        (트랜잭션은 동일한 테이블을 여러번 SELECT 할 수 있음)
      - 자신의 트랜잭션 번호보다 낮은 트랜잭션 번호에서 변경된(+커밋된) 것만 보게 되는 
      - MySQL DBMS에서 기본으로 사용
      - 한 트랜잭션의 실행시간이 길어질수록 해당 시간만큼 계속 멀티 버전을 관리해야 하는 단점이 있음
      - 하지만 실제로 영향을 미칠 정도로 오래 지속되는 경우는 거의 없어서 READ COMMITTED와 REPETABLE READ의 성능차이는 거의 없다고 함
    - SERIALIZABLE
      - 격리수준이 SERIALIZABLE일 경우 읽기 작업에도 공유 잠금을 설정하게 되고, 이러면 동시에 다른 트랜잭션에서 이 레코드를 변경하지 못하게 됨
  - TRUNCATE 명령
    - DELETE 명령어와 같이 특정 Table 및 recode를 삭제하는 역할을 함
    - TRUNCATE vs DELETE
      - `DELETE`는 조건절 부여 가능, `TRUNCATE`는 조건절 부여 불가능
      - `DELETE`는 데이터를 순차적으로 삭제하고, `TRUNCATE`는 DROP 후 CREATE함
      - 속도는 `TRUNCATE`가 더 빠르지만, 복구가 불가능함
