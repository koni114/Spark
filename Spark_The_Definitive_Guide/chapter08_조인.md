## chapter08 조인
- 앞장에서는 단일 데이터셋의 집계 방법을 알아봤는데, Spark 어플리케이션에서는 다양한 데이터셋을 함께 결합해 사용하는 경우가 더 많음
- 따라서 조인은 거의 모든 Spark 작업에 필수적으로 사용됨
- Spark는 서로 다른 데이터를 조합할 수 있으므로, 데이터를 처리할 때 기업의 여러 데이터소스를 활용할 수 있음
- 이번 장에서는 <b>Spark가 지원하는 조인 타입과 사용법</b> 그리고 <b>Spark가 클러스터에서 어떻게 조인을 실행하는지</b>  
  생각해볼 수 있도록 기본적인 내부 동작 방식을 다룸
- 이러한 기초 지식은 메모리 부족 상황을 회피하는 방법과 이전에 풀지 못했던 문제를 해결하는데 도움이 됨

### 8.1 조인 표현식
- Spark는 왼쪽과 오른쪽 데이터셋에 있는 하나 이상의 키값을 비교하고 왼쪽 데이터셋과  
  오른쪽 데이터셋의 결합 여부를 결정하는 조인 표현식(join expression)의 평가 결과에 따라 두 개의  
  데이터셋을 조인함
- 가장 많이 사용되는 조인 표현식은 왼쪽과 오른쪽 데이터셋에 지정된 키가 동일한지 비교하는 동등 조인(equi-join)임
- 키가 일치하면 Spark는 왼쪽과 오른쪽 데이터셋을 결합
- 일치하지 않으면 데이터셋을 결합하지 않음
- <b>Spark는 일치하는 키가 없는 로우는 조인에 포함시키지 않음</b>
- Spark는 동등 조인뿐만 아니라 더 복잡한 조인정책도 지원. 또한 복합 데이터 타입을 조인에 사용할 수 있음
- 예를 들어 배열 타입의 키에 조인할 키가 존재하는지 확인해 조인을 수행할 수 있음

### 8.2 조인타입
- 조인 표현식은 두 로우의 조인 여부를 결정
- 조인 타입은 결과 데이터셋에 어떤 데이터가 있어야 하는지 결정함
- Spark에서 사용할 수 있는 조인 타입은 다음과 같음
  - 내부 조인(inner join)
  - 외부 조인(outer join)
  - 왼쪽 외부 조인(left outer join)
  - 오른쪽 외부 조인(right outer join)
  - 왼쪽 세미 조인(left semi join) : 키가 일치하는 경우에 키가 일치하는 왼쪽 데이터셋만 유지
  - 왼쪽 안티 조인(left anti join) : 키가 일치하지 않는 경우에는 키가 일치하지 않는 왼쪽 데이터셋만 유지
  - 자연 조인(natural join) : 두 데이터셋에서 동일한 이름을 가진 컬럼을 암시적으로 결합하는 조인을 수행
  - 교차 조인(cross join) 또는 카테시안 조인(Cartesian join) : 왼쪽 데이터셋의 모든 로우와 오른쪽 데이터셋의 모든 로우를 조합
- 조인 예제를 해보기 위한 간단한 데이터셋을 만들자
~~~scala
val person = Seq(
    (0, "Bill Chambers", 0, Seq(100)),
    (1, "Matei Zaharia", 1, Seq(500, 250, 100)),
    (2, "Michael Armbrust", 1, Seq(250, 100))).toDF("id", "name", "graduate_program", "spark_status")

val graduateProgram = Seq(
    (0, "Masters", "School of Information", "UC Berkeley"),
    (2, "Masters", "EECS", "UC Berkeley"),
    (1, "Ph.D", "EECS", "UC Berkeley")).toDF("id", "degree", "department", "school")

val sparkStatus = Seq(
    (500, "Vice president"),
    (250, "PMC Member"),
    (100, "Contributor")).toDF("id", "status")
~~~
- 생생한 데이터셋을 이 장 전체 예제에서 사용하기 위해 테이블로 등록
~~~scala
person.createOrReplaceTempView("person")
graduateProgram.createOrReplaceTempView("graduateProgram")
sparkStatus.createOrReplaceTempView("sparkStatus")
~~~

### 8.3 내부 조인
- 내부 조인은 DataFrame이나 테이블에 존재하는 키를 평가함  
  그리고 참(true)로 평가되는 로우만 결합함
- 다음은 graduateProgram DataFrame과 person DataFrame을 조인해 새로운 DataFrame을 만드는 예제
~~~scala
val joinExpression = person.col("graduate_program") === graduateProgram.col("id")
~~~
- 두 DataFrame 모두에 키가 존재하지 않으면 결과 DataFrame에서 볼 수 없음
- 예를들어 다음과 같은 표현식을 사용하면 비어 있는 결과 DataFrame을 얻게 됨
~~~scala
// name 컬럼과 school 컬럼의 값이 일치하는 것이 없으므로, 비어있는 DataFrame return
val wrongJoinExpression = person.col("name") === graduateProgram.col("school")
~~~
- 내부 조인은 기본 조인 방식이므로 JOIN 표현식에 왼쪽 DataFrame과 오른쪽 DataFrame을 지정하기만 하면 됨
~~~scala
person.join(graduateProgram, joinExpression).show()
~~~
- `join` 메서드의 세 번째 파라미터(joinType)으로 조인 타입을 명확하게 지정 가능
~~~scala
val joinType = "inner"
person.join(graduateProgram, joinExpression, joinType).show()
~~~

### 8.4 외부 조인
- 외부 조인은 DataFrame이나 테이블에 존재하는 키를 평가하여 참(true)이나 거짓(false)으로 평가한 로우를 포함함
- 왼쪽이나 오른쪽 DataFrame에 일치하는 로우가 없다면 <b>Spark는 해당 위치에 null을 삽입</b>
~~~scala
joinType = "outer"
person.join(graduateProgram, joinExpression, joinType).show()
~~~

### 8.5 왼쪽 외부 조인
- 왼쪽 로우의 키가 기준이 되어 오른쪽에 일치하는 키가 없으면 null 삽입
~~~scala
joinType = "left_outer"
person.join(graduateProgram, joinExpression, joinType).show()
~~~

### 8.6 오른쪽 외부 조인
- 오른쪽 로우의 키가 기준이 되어 왼쪽에 일치하는 키가 없으면 null 삽입 
~~~scala
joinType = "right_outer"
person.join(graduateProgram, joinExpression, joinType).show()
~~~

### 8.7 왼쪽 세미 조인
- 왼쪽 세미 조인은 오른쪽 DataFrame의 어떤 값도 포함하지 않기 때문에 다른 조인 타입과는 약간 다름
- 단지 두 번째 DataFrame은 값이 존재하는지 확인하기 위해 값만 비교하는 용도로 사용  
  만약 값이 존재한다면 왼쪽 DataFrame에 중복 키가 존재하더라도 해당 로우는 결과에 포함
- 왼쪽 세미 조인은 기존 조인과는 달리 DataFrame의 필터 정도로 볼 수 있음
~~~scala
joinType = "left_semi"
graduateProgram.join(person, joinExpression, joinType).show()

val gradProgram2 = graduateProgram.union(Seq(
  (0, "Masters", "Duplicated Row", "Duplicated School")).toDF())
~~~

### 8.8 왼쪽 안티 조인
- 왼쪽 세미 조인과 반대되는 개념으로, 두 번째 DataFrame에서 관련된 <b>키를 찾을 수 없는 로우</b>만 결과에 포함
- 안티 조인은 SQL의 NOT IN과 같은 스타일의 필터로 볼 수 있음
~~~scala
joinType = "left_anti"
graduateProgram.join(person, joinExpression, joinType).show()
~~~

### 8.9 자연 조인
- 자연 조인은 조인하려는 컬럼을 암시적으로 추정
- 왼쪽과 오른쪽 그리고 외부 자연 조인을 사용 할 수 있음
- 항상 암시적인 처리는 위험하므로, 조심해서 사용해야 함

### 8.10 교차 조인(카테시안 조인)
- 간단하게 말해 교차 조인은 조건절을 기술하지 않은 내부 조인을 의미함
- 교차 조인은 왼쪽 DataFrame의 모든 로우를 오른쪽 DataFrame의 모든 로우와 결합
- 1000개의 로우가 존재하는 두 개의 DataFrame에 교차 조인을 수행하면 1,000,000개의 결과 로우가 생성됨
- 따라서 반드시 키워드를 이용해 교차 조인을 수행한다는 것을 명시적으로 선언해야 함
~~~scala
joinType = "cross"
graduateProgram.join(person, joinExpression, joinType).show()
~~~
- 교차 조인이 필요한 경우 다음과 같이 명시적으로 메소드 호출 가능
~~~scala
person.crossJoin(graduateProgram).show()
~~~
- 교차 조인은 정말 위험함!

### 8.11 조인 사용시 문제점
- 몇 가지 문제점과 Spark의 조인 수행 방식에 대해서 알아보자
- 이 내용에서 최적화와 관련된 몇가지 힌트를 얻을 수 있음

#### 8.11.1 복합 데이터 타입의 조인
- 복합 데이터 타입의 조인은 어려워 보이지만 그렇지 않음
- <b>불리언을 반환하는 모든 표현식은 조건 표현식으로 간주할 수 있음</b>
~~~scala
import org.apache.spark.sql functions.expr

person.withColumnRenamed("id", "personId")
.join(sparkStatus, expr("array_contains(spark_status, id)")).show()
~~~

#### 8.11.2 중복 컬럼명 처리
- 조인을 수행할 때 가장 까다로운 것 중 하나는 결과 DataFrame에서 중복된 컬럼명을 다루는 것
- <b>DataFrame의 각 컬럼은 스파크 SQL 엔진인 카탈리스트 내에 고유 ID가 있음</b> 
- 고유 ID는 카탈리스트 내부에만 사용할 수 있으며 직접 참조할 수 있는 값은 아님
- 그러므로 중복된 컬럼명이 존재하는 DataFrame을 사용할 때는 특정 컬럼을 참조하기 매우 어렵
- 이런 문제를 일으키는 상황은 다음과 같음
  - 조인에 사용할 DataFrame의 특정 키가 동일한 이름을 가지며, 키가 제거되지 않도록  
    조인 표현식에 명시하는 경우
  - 조인 대상이 아닌 두 개의 컬럼이 동일한 이름을 가진 경우
- 이러한 상황을 설명하기 위해 잘못된 데이터셋을 만들어보자
~~~scala
val gradProgramDupe = graduateProgram.withColumnRenamed("id", "graduate_program")
val joinExpr = gradProgramDupe.col("graduate_program") === person.col("graduate_program")
~~~
- 두 개의 graduate_program 컬럼이 존재  
  이러한 컬럼 중 하나를 참조할 때 문제가 발생
~~~scala
person.join(gradProgramDupe, joinExpr).select("graduate_program").show()
~~~
- 위 예제를 실행하면 오류 발생

#### 해결 방법 1 : 다른 조인 표현식 사용
- 가장 쉬운 조치 방법 중 하나는 불리언 형태의 조건 표현식(joinExpr)을 문자열이나 시퀀스 형태로 바꾸는 것
- 이렇게 되면 조인을 할 때 두 컬럼 중 하나가 자동으로 제거
~~~scala
person.join(gradProgramDupe, "graduate_program").select("graudate_program").show()
~~~

#### 해결 방법 2 : 조인 후 컬럼 제거
- 조인 후에 문제가 되는 컬럼을 제거하는 방법도 있음
- 이 경우에는 원본 DataFrame을 참조해 사용해야 함. 조인 시 동일한 키 이름을 사용하거나  
  원본 DataFrame에 동일한 컬럼명이 존재하는 경우에 사용
~~~scala
person.join(gradProgramDupe, joinExpr).drop(person.col("graduate_program"))
.select("graduate_program").show()
~~~
- 이 방법은 Spark의 SQL 분석 프로세스의 특성을 이용
- Spark는 명시적으로 참조된 컬럼(col("name"))을 검증할 필요가 없으므로 스파크 코드 분석 단계를 통과함. 위 예제에서 column 함수 대신 col 메서드를 사용한 부분을 주목할 필요가 있음
- col 메서드를 사용함으로써 컬럼 고유의 ID로 해당 컬럼을 암시적으로 지정 할 수 있음

#### 해결 방법 3 : 조인 전 컬럼명 변경
~~~scala
val gradProgram3 = graduateProgram.withColumnRenamed("id", "grad_id")
val joinExpr = person.col("graduate_program") === gradProgram3.col("grad_id")
person.join(gradProgram3, joinExpr).show()
~~~

### 8.12 Spark의 조인 수행 방식
- Spark의 조인 수행 방식을 이해하기 위해서는 실행에 필요한 두 가지 핵심 전략을 이해해야 함
  - 노드간 네트워크 통신 전략
  - 노드별 연산 전략
- 이런 내부 동작은 해결하고자 하는 비즈니스 문제와는 관련이 없을 수 있음
- 하지만 Spark 조인 수행 방식을 이해하면 빠르게 완료되는 작업과 절대 완료되지 않는 작업 간의 차이를 이해할 수 있음

#### 8.12.1 네트워크 통신 전략
- Spark는 조인 시 두 가지 클러스터 통신 방식 활용
- 전체 노드간 통신을 유발하는 <b>셔플 조인(shuffle join)</b>과 그렇지 않은 <b>브로드캐스트 조인(broadcast join)</b>임
- 이런 내부 최적화 기술은 시간이 흘러 비용 기반 옵티마이저(cost-based optimizer)가 개선되고 더 나은 통신 전략이 도입되는 경우 바뀔 수 있음
- 일반적인 상황에서 정확히 어떤 일이 일어나는지 이해할 수 있도록 고수준 예제를 알아보자  
  그러면 워크로드 성능을 쉽고 빠르게 최적화 할 수 있는 방법을 알 수 있음
- 이제부터 사용자가 Spark에서 사용하는 테이블의 크기가 아주 크거나 아주 작다고 가정
- 물론 실전에서 다루는 테이블의 크기는 다양하므로 중간 크기의 테이블을 활용하는 상황에서 설명과 다르게 작동할 수 있음
- 하지만 이해를 돕기위해 동전의 앞뒷면처럼 단순하게 정의하겠음

#### 큰 테이블과 큰 테이블 조인
- 하나의 큰 테이블과 다른 큰 테이블을 조인하면 셔플 조인이 발생   
[!img](C:/Spark/Spark_The_Definitive_Guide/imgs/shuffle_join.img)
- 셔플 조인은 전체 노드간 통신이 발생
- 그리고 조인에 사용한 특정 키나 키 집합을 어떤 노드가 가졌는지에 따라 해당 노드와 데이터를 공유
- 이런 통신 방식 때문에 네트워크는 복잡해지고 많은 자원을 사용
- 특히 데이터가 잘 나뉘어 있지 않다면 더 심해짐
- 셔플 조인 과정은 큰 테이블의 데이터를 다른 큰 테이블의 데이터와 조인하는 과정을 잘 나타냄
- 예를 들어 사물인터넷(IOT) 환경에서 매일 수십억 개의 메세지를 수신하고 일별 변경사항을 식별해야 한다면 deviceID, messageType 그리고 data와 date - 1을 나타내는 컬럼을 이용해 조인할 수 있음
- DataFrame 1 과 2는 모두 큰 DataFrame 임. 즉 전체 조인 프로세스가 진행되는 동안 모든 워커 노드에서 통신이 발생함을 의미

#### 큰 테이블과 작은 테이블 조인
- 테이블이 단일 워커 노드의 메모리 크기에 적합할 정도로 충분히 작은 경우 조인 연산을 최적화 할 수 있음
- 큰 테이블 사이에 조인에 사용한 방법도 유용하지만 브로드케스트 조인이 훨신 효율적
- <b>이 방법은 작은 DataFrame을 클러스터의 전체 워커 노드에 복제하는 것을 의미함</b>
- 이렇게 하면 자원을 많이 사용하는 것처럼 보이지만 조인 프로세스 내내 전체 노드가 통신하는 현상을 막을 수 있음
- 그림에서와 같이 시작 시 단 한번만 복제가 수행되며 그 이후로는 개별 워커가 다른 워커 노드를 기다리거나 통신할 필요 없이 작업 수행
- 브로드캐스트 조인은 이전 조인 방식과 마찬가지로 대규모 노드간 통신이 발생하지만  
  그 이후로는 노드 사이에 추가적인 통신이 발생하지 않음
- 따라서 모든 단일 노드에서 개별적으로 조인이 수행되므로 CPU가 가장 큰 병목 구간이 됨
- 다음 예제와 같이 실행 계획을 살펴보면 Spark가 자동으로 데이터셋을 브로드캐스트 조인으로  
  설정한 것을 알 수 있음
~~~scala
val joinExpr = person.col("graduate_program") === graduteProgram.col("id")
person.join(graduateProgram, joinExpr).explain()
~~~
- DataFrame API를 사용하면 옵티마이저에서 브로드캐스트 조인을 사용할 수 있도록 힌트를 줄 수 있음
- 힌트를 주는 방법은 broadcast 함수에 작은 크기의 DataFrame을 인수로 전달하는 것
- 다음 예제는 우리가 앞서 보았던 예제와 동일한 실행 계획을 세움. 하지만 항상 동일한 실행 계획을 세우는 것은 아님
~~~scala
import org.apache.spark.sql.functions.broadcast
val joinExpr = person.col("graduate_program") === graduateProgram.col("id")
person.join(broadcast(graduateProgram), joinExpr).explain()
~~~
- SQL 역시 조인 수행에 필요한 힌트를 줄 수 있음
- 하지만 강제성이 없으므로 옵티마이저가 이를 무시할 수 있음
- 특수 주석 구문을 사용해 MAPJOIN, BROADCAST, BROADCASTJOIN 등의 힌트를 설정할 수 있음
- 이를 모두 동일한 작업을 수행하며 모두 힌트로 사용할 수 있음
~~~SQL
SELECT /*+ MAPJOIN(graduateProgram) */ * FROM person JOIN graduateProgram
ON person.graduate_program = graduateProgram.id
~~~
- 물론 단점도 있음. 너무 큰 데이터를 브로드캐스트하면 고비용의 수집 연산이 발생하므로  
  드라이버 노드가 비정상적으로 종료될 수 있음
- 이러한 현상은 향후 개선되어야 하는 영역

#### 아주 작은 테이블 사이의 조인
- 아주 작은 테이블 사이의 조인은 Spark가 결정하도록 내버려두는 것이 좋음
- 필요한 경우 브로드캐스트 조인을 강제로 지정 가능

### 8.13 정리
- 한 가지 중요하게 고려해야 할 사항이 있는데, 조인 전에 데이터를 적절히 분할하면  
  셔플이 계획되어 있더라도 동일한 머신에 두 DataFrame이 있을 수 있음
- 따라서 셔플을 피할 수 있고 훨씬 더 효율적으로 실행할 수 있음
- 일부 데이터를 사전에 분할해 조인 수행 시 성능이 향상되는지 확인해 보아라!
