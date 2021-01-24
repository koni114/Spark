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
- `spark-submit` 명령을 통해 spark application 코드를 클러스터에 전송에 실행시키는 역할을 함
- `spark-submit` 명령과 같이 application 실행에 필요한 자원, 실행 방식 등 다양한 옵션 지정 가능
- Dataset은 타입 안정성(type-safe)를 지원하는 고수준 API  
  R, python과 같은 동적 언어에서는 쓸 수 없음  
  초기화에 선언한 타입 외에는 사용할 수 없음, Java는 JavaBean class, Scala는 case class를 이용해 선언 가능 
- Dataset의 장점은 `collect`, `take` 메소드를 사용하면 DataFrame의 Row 객체가 반환되는 것이 아닌, 초기에 지정한 타입의 객체를 반환
- 
## Methods, function 
- `spark.sql` : SQL 쿼리 실행
- 