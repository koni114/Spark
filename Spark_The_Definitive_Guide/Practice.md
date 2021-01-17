# Practice
## chatper05 - 구조적 API 기본 연산
1. `C:/Spark-The-Definitive-Guide-master/data/flight-data/json` 의 경로에 있는  
  2015-summary.json file을 loading해 df에 저장하세요
2. 1번 데이터의 데이터 스키마를 확인하세요
3. 1번 데이터에서 DataFrame에 직접 스키마를 만들고 적용해 보세요
4. 직접 `Seq`와 `Row`를 이용해서 dataFrame 을 생성해보세요  
   (toDF 사용, sparkContext 사용 2가지 방법)
5. df의 DEST_COUNTRY_NAME 컬럼의 head 2 row를 확인해 보세요
6. `expr`를 이용해서 DEST_COUNTRY_NAME -> destination으로 컬럼명 변경 후,  다시 DEST_COUNTRY_NAME로 변경해보세요  
(hint : as, alias 사용)
7. DataFrame에 출발지와 도착지가 같은지 나타내는 withinCountry 컬럼 추가를 해보세요
8. DataFrame에 상수 1을 새로운 컬럼 `Constant`에 추가해 보세요
9. `withcolumn` 함수를 이용해서 출발지와 도착지가 같은지 나타내는 withinCountry 컬럼을 추가해보세요
10. `withColumnRenamed`를 이용해서 컬럼명을 변경해 보세요  
    (DEST_COUNTRY_NAME -> destination)
11. 예약문자가 포함된 컬럼을 생성(되도록이면 안하는게 좋음) 하려면? 
12. Spark에서는 기본적인 대소문자 처리는 어떻게 되는가?   
    만약 대소문자를 구분하게 하려면 어떻게 해야 하는가? 
13. 컬럼을 제거하려면 어떻게 해야하는가? 
14. DEST_COUNTRY_NAME이 "United_States" 인 dataFrame 추출해 보세요
15. DEST_COUNTRY_NAME의 unique 값들을 추출해 보세요
16. `sample` 함수를 이용해서 무작위 샘플을 생성해보세요
17. `randomSplit` 함수를 이용해서 30:70으로 임의 분할하여 확인해보세요
18. df에서 20개만 제한하여 확인해 보세요
19. 