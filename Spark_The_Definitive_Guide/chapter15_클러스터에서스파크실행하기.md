## chapter 15 클러스터에서 Spark 실행하기
- 지금까지는 프로그래밍 인터페이스 관점에서 Spark의 특성을 알아봄
- 구조적 API로 정의한 논리적인 연산을 논리적 실행계획으로 분해한 다음  
  물리적 실행 계획으로 변환하는 과정도 함께 알아봄
- 물리적 실행 계획은 클러스터의 머신에서 실행되는 단위인 RDD 작업으로 구성
- 이 장에서는 Spark에서 코드를 실행할 때 어떤 일이 발생하는지 알아보자
- 사용하는 클러스터 매니저나 실행하는 코드에 의존적일 수 있으므로, 구현 방식에 구애받지 않고 설명함
- 결국 모든 Spark 코드는 동일한 방식으로 실행됨
- 이 장에서 알아볼 주제는 다음과 같음
  - Spark application의 아키텍처와 컴포넌트
  - Spark 내/외부에서 실행되는 Spark application의 생애주기
  - 파이프라이닝과 같은 중요한 저수준 실행 속성
  - Spark application을 실행하는 데 필요한 사항  
- 우선 Spark application 아키텍처를 알아보자

## 15.1 스파크 어플리케이션의 아키텍처
- 2장에서 알아봤지만, 다시 한번 알아보자
### Spark 드라이버
- <b>물리적 머신의 프로세스이며, 클러스터에서 실행 중인 애플리케이션 상태를 유지함</b>
- 스파크 애플리케이션의 운전자 역할을 하는 프로세스
- 스파크 애플리케이션의 실행을 제어하고 스파크 클러스터의 모든 상태 정보 유지
- 물리적 컴퓨팅 자원 확보
- 익스큐터 실행을 위해 클러스터 매니저와 통신

### Spark 익스큐터
- Spark 드라이버가 할당한 테스크를 수행하는 프로세스
- 익스큐터는 드라이버가 할당한 테스크를 받아 실행하고 테스크의 상태와 결과(성공 또는 실패)를 드라이버에 보고
- 모든 Spark 애플리케이션은 개별 익스큐터 프로세스를 사용

### 클러스터 매니저
- Spark 드라이버와 익스큐터를 허공에 띄울 수는 없으므로, 클러스터 매니저가 필요
- 클러스터 매니저는 Spark 애플리케이션을 실행할 클러스터 머신을 유지
- 클러스터 매니저는 '드라이버'(<b>마스터</b>라고 부르기도 함)와 '워커'라는 개념을 가지고 있으며, 이 때문에 혼란스러울 수 있음  
<b>가장 큰 차이점은 프로세스가 아닌 물리적인 머신에 연결되는 개념이라는 점</b>
- 하단 [그림 15-1]에 기본적인 클러스터 구성을 나타냄
- 그림 왼쪽에 있는 머신은 클러스터 매니저의 드라이버 노드임. 원은 개별 워커 노드를 실행하고 관리하는 데몬 프로세스
- <b>그림에서 Spark 애플리케이션은 아직 실행되지 않음. 표시된 원들은 클러스터 매니저의 프로세스 일 뿐!</b>

![img](https://github.com/koni114/Spark/blob/main/Spark_The_Definitive_Guide/imgs/cluster_manager.jpg)

- Spark 애플리케이션을 실제로 실행할 때가 되면 우리는 클러스터 매니저에 자원 할당 요청
- 사용자 애플리케이션 설정에 따라 스파크 드라이버를 실행할 자원을 포함해 요청하거나 Spark 애플리케이션 실행을 위한 엑스큐터 자원을 요청할 수 있음
- Spark 애플리케이션의 실행 과정에서 클러스터 매니저는 애플리케이션이 실행되는 머신 관리
- Spark가 지원하는 클러스터 매니저는 다음과 같음
  - stand-alone 클러스터 매니저
  - apache 메소스
  - hadoop YARN
- Spark가 지원하는 클러스터 매니저가 늘어날 수 있음. 따라서 스파크 공식 문서에서 선호하는 클러스터 관리자를 지원하는지 확인
- 지금까지는 <b>애플리케이션의 기본 컴포넌트를 알아봄</b>
- 애플리케이션을 실행할 때 처음으로 선택해야 하는 '실행 모드'를 알아보자

### 15.1.1 실행 모드
- <b>실행 모드는 애플리케이션을 실행할 때 요청한 자원의 물리적인 위치 결정</b>
- 선택할 수 있는 실행 모드는 다음과 같음
  - 클러스터 모드
  - 클라이언트 모드
  - 로컬 모드
- [그림 15-1]을 기준으로 각 실행 모드를 자세히 알아보자
- 이어지는 절에서 실선으로 그려진 직사각형은 Spark 드라이버 프로세스를 나타내며, 점선으로 그려진 직사각형은 익스큐터 프로세스를 나타냄

#### 클러스터 모드
- 가장 흔하게 사용되는 스파크 애플리케이션 실행 방식은 클러스터 모드
- 클러스터 모드를 사용하려면 컴파일된 JAR 파일이나 파이썬 스크립트 또는 R 스크립트를 클러스터 매니저에 전달해야 함
- 클러스터 매니저는 파일을 받은 다음 워커 노드에 드라이버와 익스큐터 프로세스 실행
- Cluster에서 Spark Application을 실행할 때 발생하는 단계
  - client는 spark-submit을 이용하여 애플리케이션을 제출
  - spark-submit은 드라이버 프로그램을 실행, 사용자가 정의한 main() 메소드를 호출
  - Driver program은 Clutser Manager에게 executor 실행을 위한 리소스 요청
  - clutser manager는 Driver program을 대신해 executor를 실행
  - Driver는 program에 작성된 RDD의 트랜스포메이션, 액션을 기반하여 작업 내역을 Task로 나눠 Executor에게 보냄
  - 단위 작업들은 결과를 계산하고 저장하기 위해 executor들에 의해 실행
  - driver의 main()이 끝나거나 SparkContext.stop()이 호출된다면 executor들은 중지되고 클러스터 매니저에 사용했던 자원 반환

![img](https://github.com/koni114/Spark/blob/main/Spark_The_Definitive_Guide/imgs/Spark_clutserMode.jpg)

#### 클라이언트 모드
- 클라이언트 모드는 <b>애플리케이션을 제출한 클라이언트 머신에 Spark 드라이버가 위치</b>한다는 것을 제외하면 클러스터 모드와 동일
- 클라이언트 머신은 Spark 드라이버 프로세스를 유지하며, 클러스터 매니저는 익스큐터 프로세스를 유지
- [그림 15-3]을 보면 Spark 애플리케이션이 클러스터와 무관한 머신에서 동작하는 것을 알 수 있음
- 보통은 이런 머신을 <b>게이트웨이 머신(gateway machine)</b> 또는 <b>에지 노드(edge node)</b> 라고 부름
- [그림 15-3]을 보면 드라이버는 클러스터 외부의 머신에서 실행되며 나머지 워커는 클러스터에 위치하는 것을 알 수 있음
- <b>클라이언트 모드 수행시, Spark Application을 실행했던 콘솔을 닫아버리거나 기타 다른 방법으로 client process를 중지시키면, Spark Context도 함께 종료되면서 수행 중인 Spark job이 중지됨</b>
- 클러스터 매니저는 모든 Spark application과 관련된 프로세스를 유지하는 역할 수행
- [그림 15-2]는 하나의 워커 노드에 Spark 드라이버를 할당하고 다른 워커 노드에 익스큐터를 할당하는 모습을 나타냄
- 주로 개발과정에서 대화형 디버깅(Spark shell)을 할 때 사용
 
 ![img](https://github.com/koni114/Spark/blob/main/Spark_The_Definitive_Guide/imgs/Spark_clientMode.jpg)


#### 로컬 모드
- 로컬 모드는 앞서 알아본 두 모드가 상당히 다름
- 로컬 모드로 설정된 경우 <b>모든 스파크 애플리케이션은 단일 머신에서 실행</b>됨
- 로컬 모드는 애플리케이션의 병렬 처리를 위해 단일 머신의 스레드를 활용
- 이 모드는 Spark를 학습하거나 애플리케이션 테스트 그리고 개발 중인 애플리케이션을 반복적으로 실험하는 용도로 주로 사용
- 그러므로 운영용 애플리케이션을 실행할 때는 로컬 모드 사용을 권장하지 않음

## 15.2 스파크 애플리케이션의 생애주기(Spark 외부)
- 여기서 말하는 생애주기(life-cycle)은 시작 - 종료까지의 일련의 과정으로 볼 수 있음
- 지금까지는 Spark 애플리케이션을 다루는 데 필요한 용어를 알아보았음
- 이제 Spark 애플리케이션의 생애주기를 알아볼 차례
- 3장에서 소개한 spark-submit 명령을 사용해 애플리케이션을 실행하는 예제를 그림과 함께 설명할 것임 
- 하나의 드라이버 노드와 세 개의 워커 노드로 구성된 총 네 대 규모의 클러스터가 이미 실행되고 있다고 가정
- 이 시점에서 클러스터 매니저는 중요하지 않음
- 앞 절에서 알아본 용어를 사용해 초기화부터 종료까지 Spark 애플리케이션의 생애주기를 알아보겠음
- 그림 표현시 네트워크 통신을 나타내는 선이 존재하는데, 두꺼운 화살표 선은 Spark나 Spark관련 프로세스가 수행하는 통신을 표현  
  점선은 클러스터 매니저와의 통신 같은 일반적인 통신을 표현  

### 15.2.1 클라이언트 요청
- <b>첫 단계는 Spark 애플리케이션을 제출하는 것</b>
- 여기서 Spark 애플리케이션은 컴파일된 JAR나 라이브러리 파일을 의미함
- Spark 애플리케이션을 제출하는 시점에 로컬 머신에서 코드가 실행되어 클러스터 드라이버 노드에 요청
- 이 과정에서 <b>Spark 드라이버 프로세스</b>의 자원을 함께 요청
- 클러스터 매니저는 이 요청을 받아드리고 클러스터 노드 중 하나에 드라이버 프로세스를 실행
- Spark 잡을 제출한 클라이언트 프로세스는 종료되고 에플리케이션은 클러스터에서 실행됨

![img](https://github.com/koni114/Spark/blob/main/Spark_The_Definitive_Guide/imgs/Spark_requirement.jpg)

- Spark 애플리케이션을 제출하기 위해 터미널에서 다음과 같은 형태의 명령 실행
~~~
./bin/spark-submit \
--class <main-class> \
--master <master-url>  \
--deploy-mode cluster \
--conf <key>=<value> \
... # 다른 옵션
<application-jar>  \
[application-arguments]
~~~

### 15.2.2 시작
- 드라이버 프로세스가 클러스터에 배치되었으므로 사용자 코드를 실행할 차례([그림 15-5])
- <b>사용자 코드는 반드시 스파크 클러스터를 초기화 시키는 SparkSession이 포함되어 있어야 함</b>
- SparkSession은 클러스터 매니저와 통신(그림에서 어두운 선)해 Spark 익스큐터 프로세스의 실행(그림에서 어두운 선)을 요청 
- 사용자는 spark-submit을 실행할 때 사용하는 명령행 인수로 익스큐터 수와 설정값을 지정할 수 있음
- 클러스터 매니저는 익스큐터 프로세스를 시작하고 결과를 응답받아 익스큐터의 위치와 관련 정보를 드라이버 프로세스로 전송함
- 모든 작업이 정상적으로 완료되면 '스파크 클러스터'가 완성됨

[그림 15-5]  
![img](https://github.com/koni114/Spark/blob/main/Spark_The_Definitive_Guide/imgs/Spark_start.jpg)

### 15.2.3 실행
- '스파크 클러스터'가 생성되었으므로 [그림 15-6]과 같이 코드 실행
- 드라이버와 워커는 코드를 실행하고 데이터를 이동하는 과정에서 서로 통신함
- 드라이버는 각 워커에 태스크를 할당함
- 태스크를 할당받은 워커는 태스크의 상태와 성공/실패 여부를 드라이버에 전송함
- 이와 관련된 내용은 잠시 후에 자세히 알아보자

![img](https://github.com/koni114/Spark/blob/main/Spark_The_Definitive_Guide/imgs/Spark_execution.jpg)

### 15.2.4 완료
- Spark 애플리케이션의 실행이 완료되면 드라이버 프로세스가 성공이나 실패 중 하나의 상태로 종료됨
- 그런 다음 클러스터 매니저는 드라이버가 속한 Spark 클러스터의 모든 익스큐터를 종료시킴
- 이 시점에 Spark 애플리케이션의 성공/실패 여부를 클러스터 매니저에 요청해 확인할 수 있음

![img](https://github.com/koni114/Spark/blob/main/Spark_The_Definitive_Guide/imgs/Spark_end.jpg)

## 15.3 스파크 애플리케이션의 생애주기(스파크 내부)
- 지금까지는 클러스터 관점(Spark를 지원하는 인프라 관점)에서 스파크 애플리케이션의 생애주기를 알아봄
- 그러나 애플리케이션을 실행하면 스파크 내부에서 어떤 일이 발생하는지 알아야 함
- 여기서는 Spark 애플리케이션을 정의하는 실제 '사용자 코드'와 관련된 이야기를 함
- 스파크 애플리케이션은 하나 이상의 <b>Spark Job</b>으로 구성
- 스레드를 사용해 여러 액션을 병렬로 수행하는 경우가 아니라면 애플리케이션의 스파크 잡은 차례로 실행됨

### 15.3.1 SparkSession
- 모든 Spark 애플리케이션은 가장 먼저 SparkSession을 생성함
- 여러 대화형 모드에서는 자동으로 생성되지만, 애플리케이션을 만드는 경우라면 직접 생성해야 함
- 기존 코드에서는 `new SparkContext` 패턴을 사용  
  그러나 `SparkSession`의 빌더 메서드를 이용해 생성할 것을 추천
- 이 방식을 사용하면 Spark와 Spark SQL 컨텍스트를 `new SparkContext` 패턴을 사용해서 만드는 것보다 안전하게 생성할 수 있음
- 그리고 <b>Spark 애플리케이션에서 다수의 라이브러리가 세션을 생성하려는 상황에서 컨텍스트 충돌을 방지할 수 있음</b>
~~~scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder().appName("Databricks Spark Example")
.config("spark.sql.warehouse.dir", "/user/hive/warehouse")
.getOrCreate()
~~~
- `SparkSession`을 생성하면 스파크 코드를 실행할 수 있음
- `SparkSession`을 사용해 모든 저수준 API, 기존 컨텍스트 그리고 관련 설정 정보에 접근할 수 있음
- `SparkSession` 클래스는 스파크 2.x 버전에서만 사용할 수 있음
- 과거 버전의 코드에서는 구조적 API를 사용하기 위해 `SparkSession` 대신 `SparkContext` 와 `SQLContext`를 직접 생성

#### SparkContext
- SparkSession의 SparkContext는 스파크 클러스터에 대한 연결을 나타냄
- SparkContext를 이용해 RDD 같은 저수준 API를 사용할 수 있음. Spark 과거 버전의 예제나 문서를 보면 SparkContext는 일반적으로 sc 변수를 사용
- SparkContext로 RDD, 어큐물레이터 그리고 브로드캐스트 변수를 생성하고 코드 실행 가능
- 대부분의 경우 SparkSession으로 SparkContext에 접근할 수 있으므로 명시적으로 SparkContext를 초기화할 필요는 없음
- 직접 초기화하는 가장 일반적인 방법은 `getOrCreate` 메서드 사용
~~~scala
import org.apache.spark.SparkContext
val sc = SparkContext.getOrCreate()
~~~
- <b>SQLContext와 HiveContext는 거의 사용할 필요가 없음</b>
- SparkSession이 초기화되었다면 코드를 실행할 차례  
  모든 Spark 코드는 RDD 명령으로 컴파일됨. 따라서 일부 논리적 명령(DataFrame job)을 알아보고 어떤 일이 발생하는지 단계별로 알아보자

### 15.3.2 논리적 명령
- 책의 서두에서 보았듯이 Spark 코드는 트랜스포메이션과 액션으로 구성됨
- 사용자는 SQL, 저수준 RDD 처리, ML 알고리즘 등을 사용해 트랜스포메이션과 액션을 마음대로 구성 가능
- 그러므로 <b>DataFrame과 같은 선언적 명령을 사용하는 방법과 논리적 명령이 물리적 실행 계획으로 어떻게 변환되는지 이해하는 것은 중요</b>함
- 이를 기반으로 Spark 클러스터가 동작하는 방식을 이해할 수 있음
- 이 절에서는 잡, 스테이지, 테스크를 차례로 따라갈 수 있도록 Spark shell을 다시 띄워 코드를 실행함

#### 논리적 명령을 물리적 실행 계획으로 변경하기
- Spark가 사용자 코드를 어떻게 받아드리고 클러스터에 어떻게 명령을 전달하는지 다시 한번 되짚어보자
- 그리고 코드 예제를 한 줄씩 살펴보면서 그 안에서 일어나는 작업을 알아보자  
  그러면 Spark application 동작 방식을 더 깊이 이해하게 될 것임
- 모니터링 관련 내용을 다루는 다음 장에서 Spark UI를 사용해 Spark Job을 더 자세히 추적함
- 지금은 간단한 방식으로 Job을 추적할 것임
- 그리고 간단한 DataFrame을 이용해 파티션을 재분배하는 Job, 값을 트랜스포메이션하는 Job,  집계 및 최종 결과를 얻어내는 Job 이렇게 세 단계 Job을 수행함
- 아래 예제는 python 코드로 수행한 예제. 스칼라 코드로 실행해도 무방함. Job 수가 크게 달라지지는 않겠지만 물리적 실행 전략을 변화시키는 Spark 최적화 과정에 따라 Job 수가 달라질 수 있음
~~~python
df1 = spark.range(2, 10000000, 2)
df2 = spark.range(2, 10000000, 4)
step1 = df1.repartition(5)
step2 = df2.repartition(6)
step2 = step1.selectExpr("id * 5 as id")
step3 = step2.join(step12, ["id"])
step4 = step3.selectExpr("sum(id)")

step4.collect() # 결과는 2,500,000,000,000
~~~
- 위 코드 예제를 실행하면 액션으로 하나의 Spark Job이 완료되는 것을 확인할 수 있음
- 물리적 실행 계획에 대한 이해를 높이기 위해 실행 계획을 살펴보자
- 실행 계획 정보는 쿼리를 실제로 수행한 다음 Spark UI의 SQL 탭에서도 확인 가능
~~~
step4.explain()

== Physical Plan ==
*(7) HashAggregate(keys=[], functions=[sum(id#7L)])
+- Exchange SinglePartition
   +- *(6) HashAggregate(keys=[], functions=[partial_sum(id#7L)])
      +- *(6) Project [id#7L]
         +- *(6) SortMergeJoin [id#7L], [id#2L], Inner
            :- *(3) Sort [id#7L ASC NULLS FIRST], false, 0
            :  +- Exchange hashpartitioning(id#7L, 200)
            :     +- *(2) Project [(id#0L * 5) AS id#7L]
            :        +- Exchange RoundRobinPartitioning(5)
            :           +- *(1) Range (2, 10000000, step=2, splits=8)
            +- *(5) Sort [id#2L ASC NULLS FIRST], false, 0
               +- Exchange hashpartitioning(id#2L, 200)
                  +- Exchange RoundRobinPartitioning(6)
                     +- *(4) Range (2, 10000000, step=4, splits=8)
~~~
- `collect` 같은 액션을 호출하면 <b>개별 Stage와 task로 이루어진 Spark Job이 실행</b>됨
- 로컬 머신에서 Spark Job을 실행하면 localhost:4040에 접속해 Spark UI를 확인할 수 있음
-  Jobs 탭에서 Stage와 task로 이동해 자세한 내용을 확인할 수 있음

### 15.3.3 스파크 Job
- 보통 액션 하나당 하나의 Spark Job이 생성되며, 액션은 항상 같은 결과를 반환
- Spark Job은 일련의 스테이지로 나뉘며 스테이지 수는 셔플 작업이 얼마나 많이 발생하는지에 따라 달라짐
- 이전 예제의 Job은 다음과 같이 나타남
  - 스테이지 1 : 태스크 8개
  - 스테이지 2 : 태스크 8개
  - 스테이지 3 : 태스크 5개
  - 스테이지 4 : 태스크 6개
  - 스테이지 5 : 태스크 200개
  - 스테이지 6 : 태스크 1개
- '어떻게 저 숫자들이 나온거지?'라는 의문이 들었을 것임. 이해를 돕기 위해 다음 절에서 자세히 알아보자

### 15.3.4 스테이지
- <b>Spark의 스테이지는 다수의 머신에서 동일한 연산을 수행하는 태스크의 그룹</b>
- Spark는 가능한 한 많은 태스크(잡의 트랜스포메이션)를 동일한 스테이지로 묶으려 노력함
- 셔플 작업이 일어난 다음에는 반드시 새로운 스테이지를 시작함
- <b>셔플은 데이터의 물리적 재분배 과정</b>임
- 예를 들어 DataFrame 정렬이나 키별로 적재된 파일 데이터를 그룹화하는 작업과 같음
- 파티션을 재분배하는 과정은 데이터를 이동시키는 작업이므로 익스큐터 간의 조정이 필요
- Spark는 셔플이 끝난 다음 새로운 스테이지를 시작하며 최종 결과를 계산하기 위해 스테이지 실행 순서를 계속 추적함
- 이전 Job에서 처음 두 스테이지는 DataFrame 생성을 위해 사용한 `range`명령을 수행하는 단계
- `range`명령을 사용해 DataFrame을 생성하면 기본적으로 8개의 파티션을 생성함
- 다음은 파티션 재분배 단계. 이 단계에서는 데이터 셔플링으로 파티션 수를 변경함
- 두 개의 DataFrame은 스테이지 3, 4의 테스크 수에 해당하는 5개,6개의 파티션으로 재분배
- 스테이지 3,4는 개별 DataFrame에서 수행됨
- 마지막 두 스테이지는 조인(셔플)을 수행함. 
- "왜 갑자기 테스크 수가 200개로 변했을까요?"  
  그 이유는 스파크 SQL 설정 때문. `spark.sql.shuffle.partitions` 속성의 기본값은 200개의 셔플 파티션을 생성함
- `spark.sql.shuffle.partitions`속성을 원하는 값으로 변경할 수 있음
- 그러면 셔플을 수행할 때 생성되는 파티션 수가 변경됨. 이 값은 효율적인 실행을 위해 클러스터의 코어 수에 맞추어 설정함. 설정 방법은 다음과 같음
~~~scala 
spark.conf.set("spark.sql.shuffle.partitions", 50)
~~~
- 여러 요인의 영향을 받을 수 있지만, 경험적으로 보면 클러스터의 익스큐터 수보다 파티션 수를 더 크게 지정하는 것이 좋음
- 로컬 머신에서 코드를 실행하는 경우  병렬로 처리할 수 있는 태스크 수가 제한적이므로 이 값을 작게 설정해야 함
- 이 설정은 더 많은 익스큐터 코어를 사용할 수 있는 클러스터 환경을 위한 기본값
- 최종 스테이지에서는 드라이버로 결과를 전송하기 전에 파티션마다 개별적으로 수행한 결과를 단일 파티션으로 모으는 작업을 수행함
- 4부에서 이러한 속성을 설정하는 과정을 여러 번 보게 될 것임

### 15.3.5 태스크
- Spark의 스테이지는 테스크로 구성
- <b>각 태스크는 단일 익스큐터에서 실행할 데이터의 블록과 다수의 트랜스포메이션 조합으로 볼 수 있음</b>
- 만약 데이터셋이 거대한 하나의 파티션인 경우 하나의 테스크만 생성
- 약 1000개의 작은 파티션으로 구성되어 있다면 1000개의 태스크를 만들어 병렬로 실행 가능
- 즉, <b>태스크는 데이터 단위에 적용되는 연산 단위를 말함</b>
- 파티션 수를 늘리면 더 높은 병렬성을 얻을 수 있음, 물론 만병통치약은 아니지만 최적화를 위한 가장 간단한 방법임

## 15.4. 세부 실행 과정
- Spark의 스테이지와 태스크는 알아두면 좋을 만한 중요한 특성을 가지고 있음
- 첫째, Spark는 map 연산 후 다른 map 연산이 이어진다면 함께 실행될 수 있도록 스테이지와 태스크를 자동 연결
- 둘째, Spark는 모든 셔플을 작업할 때 데이터를 안정적인 저장소(ex) disk)에 저장하므로 여러 Job에서 재사용할 수 있음 
- 이러한 개념들은 Spark UI로 애플리케이션을 들여다보기 시작하면 만나게 될 내용들임

### 15.4.1 파이프라이닝
- <b>Spark를 '인메모리 컴퓨팅 도구'로 만들어주는 핵심 요소 중 하나는 맵리듀스와 같은 Spark 이전 기능과는 달리 메모리나 디스크에 데이터를 쓰기 전 최대한 많은 단계를 수행한다는 것</b>
- Spark가 수행하는 주요 최적화 기법 중 하나는 RDD나 RDD보다 더 아래에서 발생하는 파이프라이닝 기법
- <b>파이프라이닝</b> 기법은 노드 간의 데이터 이동 없이 각 노드가 데이터를 직접 공급할 수 있는 연산만 모아 태스크의 단일 스테이지로 만듬
-  예를 들어 `map`, -> `filter`, -> `map` 순서로 수행되는 RDD 기반의 프로그램을 개발했다면 개별 입력 레코드를 읽어 첫 번째 `map` 으로 전달한 다음 `filter` 하고 마지막 `map` 함수로 전달해 처리하는 과정을 태스크의 단일 스테이지로 만듬
- 따라서 파이프라인으로 구성된 연산 작업은 단계별로 메모리나 디스크에 중간 결과를 기록하는 방식보다 훨씬 처리 속도가 빠름  
- `select`, `filter`, `select`를 수행하는 DataFrame이나 SQL 연산에서도 동일한 파이프라이닝 유형이 적용됨
- <b>Spark 런타임</b>에서 파이프라이닝을 자동으로 수행하기 때문에 애플리케이션을 개발할 때는 눈에 보이지 않음
- 스파크 UI나 로그 파일로 애플리케이션을 확인해보면 다수의 RDD 또는 DataFrame 연산이 하나의 스테이지로 파이프라이닝 되어 있음을 확인 가능

### 15.4.2 셔플 저장
- 두 번째 특성은 셔플 저장(shuffle persistence)임
- Spark가 reduce-by-key(특정 그룹별로 reduce 연산 수행) 연산 같이 노드 간 복제를 유발하는 연산을 실행하면 엔진에서 파이프라이닝을 수행하지 못하므로 네트워크 셔플이 발생함
- 노드 간 복제를 유발하는 연산은 각 키에 대한 입력 데이터를 먼저 여러 노드로부터 복사함
- 항상 데이터 전송이 필요한 '소스'태스크를 먼저 수행하기 때문
- 그리고 소스 태스크의 스테이지가 실행되는 동안 <b>셔플 파일</b>을 로컬 디스크에 기록
- 그런 다음 그룹화나 리듀스를 수행하는 스테이지가 시작. 이 스테이지에서는 셔플 파일에서 레코드를 읽어 들인 다음 연산 수행
- 예를 들어 특정 범위의 키와 관련된 데이터를 읽고 처리함
- 만약 Job이 실패한 경우 셔플 파일을 디스크에 저장했기 때문에 '소스' 스테이지가 아닌 해당 스테이지부터 처리할 수 있음
- 따라서 '소스' 태스크를 재실행할 필요 없이 실패한 리듀스 태스크부터 다시 실행 가능
- 셔플 결과를 저장할 때 발생할 수 있는 부작용은 <b>이미 셔플된 데이터를 이용해 새로운 잡을 실행하면 '소스'와 관련된 셔플이 다시 실행되지 않는다는 것</b>
- Spark는 다음 스테이지를 실행하는 과정에서 디스크에 이미 기록되어 있는 셔플 파일을 다시 사용할 수 있다고 판단하기 때문에 이전 스테이지를 처리하지 않음
- Spark UI와 로그 파일이서 'skipped'라고 표시된 사전 셔플 스테이지(pre-shuffle-stage)를 확인할 수 있음 
- 이러한 자동 최적화 기능은 동일한 데이터를 활용해 여러 Job을 실행하는 워크로드의 시간을 절약할 수 있음
- 물론 더 나은 성능을 얻기 위해 DataFrame이나 RDD의 cache 메서드를 사용할 수 있음
- 그러면 사용자가 직접 캐싱을 수행할 수 있으며 정확히 어떤 데이터가 어디에 저장되는지 제어할 수 있음
- 집계된 데이터에 Spark 액션을 수행한 다음 Spark UI를 확인해보면 이러한 방식을 쉽게 이해 가능

## 15.5 정리
- Spark 애플리케이션을 클러스터에서 실행하면 어떤 일이 일어나는지 알아봄
- 즉, 클러스터에서 실제로 실행되는 방식과 Spark Application 내부에서 어떤 일이 일어나는지 알 수 있었음
- 이 내용으로 사용자 애플리케이션의 디버깅 시작 지점을 찾을 수 있음
- 다음 장에서는 Spark Application을 개발하는 방법과 고려사항을 알아보자