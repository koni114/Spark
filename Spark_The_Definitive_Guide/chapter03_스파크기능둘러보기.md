## chapter03 스파크 기능 둘러보기
- Spark는 기본 요소인 <b>저수준 API</b>와 <b>고수준 API</b>, 추가 기능을 제공하는 <b>표준 라이브러리</b>로 구성

#### Spark의 기능

~~~
---------------------------------------------------------
| 구조적 스트리밍 | 고급 분석 | 라이브러리 및 에코시스템 |
---------------------------------------------------------
|                     구조적 API                         |
|    DataSet      |   DataFrame   |         SQL          |
----------------------------------------------------------
|                     저수준 API                         |
|         RDD                        분산형 변수         |
----------------------------------------------------------
~~~
- Spark의 라이브러리는 그래프 분석, 머신러닝, 스트리밍 등 다양한 작업 지원하며  
  컴퓨팅 및 스토리지 시스템과의 통합을 돕는 역할을 함
- chapter03 에서는 다음과 같은 내용 설명
  - spark-submit 명령으로 운영용 애플리케이션 실행
  - Dataset : 타입 안정성(type-safe)를 제공하는 구조적 API
  - 구조적 스트리밍
  - 머신러닝과 고급 분석
  - RDD : Spark의 저수준 API
  - SparkR
  - 3rd Party 에코 시스템

### 3.1 운영용 애플리케이션 실행
- spark-submit 명령을 사용하여 대화형 셸에서 개발한 프로그램을 운영용 애플리케이션으로 쉽게 전환할 수 있음
- spark-submit 명령은 애플리케이션 코드를 클러스터에 전송해 실행시키는 역할을 함
- 클러스터에 제출된 애플리케이션은 작업이 종료되거나 에러가 발생할 때까지 실행됨
- Spark 애플리케이션은 stand-alone, 메소스, YARN 클러스터 매니저를 이용해 실행
- spark-submit 명령에 애플리케이션 실행에 필요한 자원, 실행 방식, 다양한 옵션 지정 가능
- 사용자는 Spark가 지원하는 프로그래밍 언어로 애플리케이션을 개발한 다음 실행 할 수 있음  
  
#### 스칼라 애플리케이션 예제
- Spark를 내려받은 디렉토리에서 다음 예제 명령 수행
~~~
// -1 --class ~~  : 예제 class 지정
// -2 --master local : local에서 수행하고자 하는 JAR file 지정
// -3 10 : 입력하고자 하는 인수 지정 
spark-submit --class org.apache.spark.examples.SparkPi --master local C:\spark-2.3.2-bin-hadoop2.7\examples\jars\spark-examples_2.11-2.3.2.jar 10
~~~
- 위의 예제는 Pi 값을 특정 자리수까지 계산
- spark-submit 명령에 예제 클래스를 지정하고 로컬 머신에서 실행되도록 설정
- 실행에 필요한 JAR 파일과 관련된 인수도 함께 지정
- spark-submit 명령 중 master 옵션의 인수값을 변경하면 스파크가 지원하는 Spark-stand-alone, 메소스, YARN 클러스터 매니저에서 동일한 애플리케이션을 실행  

### 3.2 Dataset : 타입 안정성을 제공하는 구조적 API
- Dataset은 Java와 Scala의 정적 데이터 타입에 맞는 코드, 즉 정적 타입 코드를 지원하기 위해  
  고안된 Spark의 구조적 API
- Dataset은 타입 안정성을 지원하며 동적 타입 언어인 R, python에서는 사용할 수 없음
- Dataset API는 DataFrame의 레코드를 사용자가 Java나 Scala로 정의한 클래스에 할당하고,  
  자바의 ArrayList 또는 Scala의 Seq 객체 등의 고정 타입형 컬렉션으로 다룰 수 있는 기능 제공
- Dataset API는 <b>타입 안정성</b>을 지원하므로 초기화에 사용한 클래스 대신  
  다른 클래스를 사용해 접근할 수 없음
- Dataset API는 다수의 SW 엔지니어가 잘 정의된 인터페이스로 상호작용하는 대규모 애플리케이션을 개발하는데 특히 유용
- Dataset 클래스는 내부 객체의 데이터 타입을 매개변수로 사용  
  (scala : Dataset[T], ex) Dataset[Person] --> Person 객체)

#### 예제 : 타입 안정성 함수와 DataFrame을 사용해 비즈니스 로직을 작성
~~~
// Scala 코드
// -1 Flight class 정의
case class Flight(DEST_COUNTRY_NAME:String,
                  ORIGIN_COUNTRY_NAME: String, 
                  count: BigInt)

// -2 flightDF DataFrame read
val flightDF = spark.read
.parquet("C:\Spark-The-Definitive-Guide-master\data\flight-data\parquet\2010-summary.parquet
")

// -3 dataset 생성
val flights = flightDF.as[Flight] 

// -4 filter, map 함수 적용
flights
.filter(flight_row => flight_row.ORIGIN_COUNTRY_NAME != "Canada")
.map(flight_ row => flight_row)
.take(5)
~~~
- Dataset의 장점은 collect 메소드나 take 메소드를 사용하면 DataFrame을 구성하는 Row 타입의 객체가 아닌, Dataset에 매개변수로 지정한 타입의 객체를 반환함  

### 3.3 구조적 스트리밍
- 구조적 스트리밍은 Spark 2.2 버전에서 안정화된 스트림 처리용 고수준 API
- 구조적 스트리밍을 사용하면 구조적 API로 개발된 배치 모드의 연산을 스트리밍 방식으로 실행 가능하고, 지연 시간을 줄이고 증분 처리 할 수 있음  
- 배치 처리용 코드를 일부 수정하여 스트리밍 처리를 수행하고 값을 빠르게 얻을 수 있다는 장점이 있음
- 프로토타입을 batch job 으로 개발한 다음 streaming job으로 변환할 수 있으므로, 개념 잡기가 수월함  

### 예제 : retail 데이터셋을 이용해서 구조적 스트리밍 만들기
- 데이터 셋에는 특정 날짜와 시간 정보가 있음
- 여러 프로세스에서 데이터가 꾸준히 생성되는 상황이고, 저장소로 꾸준히 전송되고 있다고 가정  
~~~
// Scala
// 1- data read
val staticDataFrame = spark.read.format("csv")
.option("header", "true")
.option("inferSchema", "true")
.load("C:\Spark-The-Definitive-Guide-master\data\retail-data\by-day\*.csv")

// 2- table 생성
staticDataFrame.createOrReplaceTempView("retail_data")

// 3- schema 생성
val staticSchema = staticDataFrame.schema
~~~

- 시계열 데이터이기 때문에 데이터를 그룹화하고 집계하는 방법을 알아보자
- 특정 고객이 대량으로 구매하는 영업시간을 살펴보자  
  총 구매비용 컬럼을 추가하고, 고객이 가장 많이 소비한 날을 찾아보자
- window 함수는 집계 시에 시계열 컬럼을 기준으로 각 날짜에 대한 전체 데이터를 가지는 윈도우를 구성
~~~
// Scala
import org.apache.spark.sql.functions.{window, col}

// -1 selectExpr : 추가적인 산술식을 통해 파생변수 등을 생성. + column select
// -2 groupBy    : 그룹별 집계값을 위해 적용
// -3 sum        : total_cost sum
// -5 show       :  
staticDataFrame
.selectExpr(
    "CustomerId",
    "(UnitPrice * Quantity) as total_cost",
    "InvoiceDate")
.groupBy(
    col("CustomerId"), window(col("InvoiceDate"), "1 day"))
.sum("total_cost")
.show(5)
~~~

- 로컬 실행시를 위해 파티션 수 조정 200(default) --> 5
~~~
spark.conf.set("spark.sql.shuffle.partitions", "5")
~~~
- 스트리밍 코드를 적용해보자
- read 메서드 --> readStream 메서드 변환
- maxFilesPerTrigger 옵션 추가 지정 --> 한 번에 읽을 파일 수 설정 가능
- 이번 예제는 '스트리밍'답게 만드는 옵션이지만, 운영 환경에 적용하는 것은 비추천
~~~
val streamingDataFrame = spark.readStream
.schema(staticSchema)
.option("maxFilesPerTrigger", 1)
.format("csv")
.option("header", "true")
.load("C:\Spark-The-Definitive-Guide-master\data\retail-data\by-day\*.csv")
~~~

- streaming 유형인지 확인
~~~
streamingDataFrame.isStreaming
~~~

- 앞선 DataFrame 처리와 동일한 비즈니스 로직 적용
~~~
val purchaseByCustomerPerHour = streamingDataFrame
.selectExpr(
    "CustomerId",
    "(UnitPrice * Quantity) as total_cost",
    "InvoiceDate")
.groupBy(
    col("CustomerId"), window(col("InvoiceDate"), "1 day"))
.sum("total_cost")
~~~
- 이 작업 역시 지연 연산이므로, 데이터 플로를 실행하기 위해 스트리밍 액션 호출 해야함
- 스트리밍 액션은 어딘가에 데이터를 채워 넣어야 하므로 count 메서드와 같은 일반적인 정적 액션과는 조금 다른 특성을 가짐
- 스트리밍 액션은 트리거가 실행된 다음 데이터를 갱신하게 될 인메모리 테이블에 데이터 저장
- 예제에서는 파일마다 트리거를 실행
- total_price maximum 5를 계산할 떄 항상 더 큰 값이 발생한 경우에만 인메모리 테이블을 갱신
~~~
// Scala
purchaseByCustomerPerHour.writeStream
.format("memory")                // memory = 인메모리 테이블에 저장
.queryName("customer_purchases") // 인메모리에 저장될 테이블명
.outputsMode("complete")         // complete = 모든 카운트 수행 결과를 테이블에 저장
.start()
~~~
- 스트림이 시작되면 쿼리 실행 결과가 어떠한 형태로 인메모리 테이블에 기록되는지 확인할 수 있음
~~~
// scala
spark.sql("""
SELECT *
FROM custome_purchases
ORDER BY 'sum(total_cost)' DESC
""")
.show(5)
~~~
- 더 많은 데이터를 읽을수록 테이블 구성이 바뀐다는 것을 알 수 있음
- 상황에 따라 결과를 콘솔에 출력 가능  
~~~
// Scala
purchaseByCustomerPerHour.writeStream
.format("console")                // console = 결과를 콘솔에 출력  
.queryName("customer_purchases_2") 
.outputsMode("complete")
.start()
~~~  

### 3.4 ML과 고급 분석
- 내장된 머신러닝 알고리즘 라이브러리인 MLlib을 사용해 대규모 머신러닝을 수행 할 수 있음
- MLlib 사용시 전처리, 멍잉(Wrangling), 모델 학습, 예측을 할 수 있음
- 구조적 스트리밍에서 예측하고자 할 때도 MLlib에서 학습시킨 다양한 예측 모델 사용
- k-means를 이용한 예제를 진행해보자

#### 예제) 트렌스포메이션 -> 모델 예측 수행
- 앞서 사용된 retail-DF를 가지고 모델 예측(K-means)를 수행해보자
- printSchema 함수를 통해 data-type 확인
~~~
staticDataframe.printSchema()
~~~
- 날짜형, 문자형 등 데이터를 수치형으로 변환
- 몇가지 DataFrame의 트랜스포메이션을 사용해 날짜 데이터를 다루는 예제
~~~
// Scala
import org.apache.spark.sql.functions.data_format

// 1- 요일 추출
// 2-  partition 개수 조정 --> 5 (파티션 조정 시 shuffle 사용 안함)
val preppendDataFrame = staticDataFrame
.na.fill(0)
.withColumn("day_of_week", date_format($"InvoiceDate", "EEEE")) // -1
.coalesce(5) // -2 
~~~

- 데이터를 특정 날짜를 기준으로 train/test 분리
- 액션을 통해 데이터를 분리 --> count
~~~
// Scala
val trainDataFrame = preppendDataFrame
.where("InvoiceDate < 2011-07-11")
val testDataFrame = preppendDataFrame
.where("InvoiceDate >= 2011-07-11")

trainDataFrame.count()
testDataFrame.count()
~~~

- StringIndexer MLlib를 사용하여 요일 -> 수치형으로 변환
~~~
import org.apache.spark.ml.feature.StringIndexer

val indexer = new StringIndexer()
.setInputCol("day_of_week")
.setOutputCol("day_of_week_index")
~~~
- one-hot encoding 수행
- 인코딩 결과 컬럼은 벡터 타입을 구성할 컬럼 중 하나로 사용    
- Spark의 모든 ML 알고리즘은 수치형 벡터 타입을 입력으로 사용    
  --> 이산형도 encoding을 해서 수행한다는 의미
~~~
import org.apache.spark.ml.feature.OntHotEncoder
val encoder = new OneHotEncoder()
.setInputCol("day_of_week_index")
.setOutputCol("day_of_week_encoded")
~~~
- 입력값으로 들어올 데이터가 같은 프로세스를 거쳐 변환되도록 파이프라인 설정
~~~
// Scala
import org.apache.spark.ml.feature.VectorAssembler

val VectorAssembler = new VectorAssembler()
.setInputCols(Array("UnitPrice", "Quantity", "day_of_week_encoded"))
.setOutputCol("features")

import org.apache.spark.ml.Pipeline

// pipeLine 설정
val transformationPipeline = new Pipeline()
.setStages(Array(indexer, encoder, vectorAssembler))
~~~
- 학습 준비 과정을 2단계로 이루어짐
  - transformer를 데이터셋에 적합(fit)
  - StringIndexer는 인덱싱할 고윳값의 수를 알아야 함  
    만약 모른다면 고윳값을 조사하고 인덱싱해야함
~~~
// pipeline에 dataFrame fit
val fittedPipeline = transformationPipeline.fit(trainDataFrame)

// 데이터 변환
val trainsformedTraining = fittedPipeline.transform(trainDataFrame)
~~~
- 동일한 트랜스포메이션을 계속 반복할 수 없으므로,  
  그 대신 모델에 일부 hyper-parameter 튜닝값을 적용
- <b>캐싱</b>을 사용하면 중간 변환된 데이터셋의 복사본을 메모리에 저장하므로,  
  전체 파이프라인을 재실행하는 것보다 훨씬 빠르게 반복적으로 데이터셋에 접근 가능
- 다음은 cashe를 적용하는 코드임
~~~
transformedTraining.cache()
~~~
- 모델을 학습해보자
- ML 모델을 사용하려면 관련 클래스를 임포트하고 인스턴스를 생성
~~~
import org.apache.spark.ml.clustering.KMeans

val kmeans = new KMeans()
.setK(20)
.setSeed(1L)
~~~
- 모델 학습 수행(k-means)
- Spark에서 모델 학습은 크게 두가지로 수행
  - 아직 학습되지 않는 모델을 초기화
  - 해당 모델 학습
- MLlib의 DataFrame에서 제공하는 모든 알고리즘은 항상 두 가지 유형으로 구성되어 있으며  
  다음의 명명규칙을 따름
  - 학습 전 알고리즘 명칭 : Algorithm
  - 학습 후 알고리즘 명칭 : AlgorithmModel
- 이번 예제에서는 KMeans, KMeansModel 이라고 명명
- MLlib의 DataFrame API에서 제공하는 estimator는 앞서 사용한 전처리 변환자(ex) StringIndexer)와 거의 동일한 인터페이스를 가지고 있음
- 이 인터페이스를 통해 학습 과정을 단순화 할 수 있지만 이번 예제에서는 단계별 설명을 위해 생략함
~~~
// Scala
val kmModel = kmeans.fit(transformedTraining)

kmModel.computeCost(transformedTraining)
val transformedTest = fittedPipeline.transform(testDataFrame) 
kmModel.computeCost(transformedTest)
~~~

### 3.5 저수준 API
- Spark는 RDD를 통해 자바와 파이썬 객체를 다루는 데 필요한 다양한 기본 기능(저수준 API)를 제공
- <b>Spark의 거의 모든 기능은 RDD를 기반으로 만들어짐</b>
- DataFrame 연산도 RDD 기반으로 만들어졌으며, 편리하고 효율적인 분산 처리를 위해 저수준 명령으로 컴파일됨
- 원시 데이터를 읽거나 다루는 용도로 RDD를 사용할 수 있지만  
  대부분 구조적 API를 사용하는 것이 좋음  
- RDD를 이용해 파티션과 같은 물리적 실행 특성을 결정할 수 있으므로  
  DataFrame보다 좀 더 세밀한 제어 가능
- 드라이버 시스템의 메모리에 저장된 원시 데이터를 병렬처리하는 데 RDD 사용 할 수 있음
- 다음은 간단한 숫자를 이용해 병렬화해 RDD를 생성하고 DataFrame으로 변환하는 예제
~~~
// Scala
spark.sparkContext.parallelize(Seq(1, 2, 3)).toDF()
~~~
- RDD 세부 구현 방식에 있어 언어적 특성의 차이를 보임

### 3.6 SparkR
- SparkR은 Spark를 R 언어로 사용하기 위한 기능
- SparkR은 스파크가 지원하는 모든 언어에 적용된 원칙을 동일하게 따르고 있음
- SparkR을 설치하고 코드를 실행하면 됨
~~~
library(SparkR)
sparkDF <- read.df("/data/flight-data/csv/2015-summary.csv",
                  source = "csv",
                  header = "true",
                  inferSchema = "true")
~~~

### 3.7 Spark의 에코시스템과 패키지
- 커뮤니티가 만들어낸 패키지 에코시스템과 다양한 기능 존재
- 이러한 기능 중 일부는 Spark 코어 프로젝트에 포함되어 널리 사용되고 있음
- spark-packages.org에서 확인 가능


### 용어 정리
- 스트리밍 데이터
  - 수 천개의 데이터 소스에서 연속적으로 생성되는 데이터  
  - 모바일이나 웹 애플리케이션을 사용하는 고객이 생성하는 로그 파일, 전자 상거래 구매,   
    게임 내 플레이어 활동, 소셜 네트워크의 정보, 주식 거래소, 지리공간 서비스,  
    연결된 디바이스의 텔레메트리, 데이터 센터의 계측 등 다양한 데이터가 포함  
- 스트림 처리
  - 스트림 처리는 데이터의 시퀀스를 수집하고, 수신되는 각 데이터 레코드에 대한 응답하고 지표, 보고서 및 요약 통계를 증분식으로 업데이트 함 
  - 실시간 모니터링 및 응답 기능에 적합 
  - 데이터 범위 : 롤링 타임 윈도우 내 데이터 또는 가장 최신 데이터 레코드의 데이터를 쿼리하거나 처리  
  - 데이터 크기 : 일부 레코드로 구성된 마이크로 배치 또는 개별 레코드
  - 성능 : 몇 초 또는 몇 밀리초의 지연 시간이 필요
  - 분석 : 간단한 응답 기능, 수집 및 롤링 지표
  - 배치 모드와 대비됨

- 증분 처리(incremental load)
  - 데이터 원본에 새로 추가되거나 변경된 데이터를 대상에 반영하는 작업

- 멍잉(munging = wrangling)
  - 원본 데이터를 다른 형태로 변환하거나 매핑하는 과정을 의미

- RDD(Resilient Distributed DataSet)
  - 여러 분산 노드에 걸쳐서 저장되는 변경이 불가능한 데이타(객체)의 집합
  - 각각의 RDD는 여러개의 파티션으로 분리가 됨
  - 쉽게 말하면 Spark에서 사용되는 Data가 SDD
