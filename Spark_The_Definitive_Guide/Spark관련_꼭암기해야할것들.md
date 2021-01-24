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

## Methods, function 
- `spark.sql` : SQL 쿼리 실행
 