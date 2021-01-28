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
- 어려가지 액션이 존재
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
  테스트를 지원하기 위해 생성된 코드와 데이터를 의미
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
- 

## Methods, function 
- `spark.sql` : SQL 쿼리 실행
 