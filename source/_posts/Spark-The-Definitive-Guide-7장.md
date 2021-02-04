---
title: '&#039;Spark The Definitive Guide&#039; 7장 - 집계해라 애송이'
date: 2021-02-03 21:42:12
categories: spark
tags:
  - spark
  - apache
  - book
  - study
---

<br/>

<p style="color:lightgray">벌써부터 슬슬 포스트 포멧 헷갈리기 시작하죠? 망했죠?<br/>
나중에 한번에 맞춰야지 생각해놓고 절대 수정안하죠? ㅎ..</p>

<br/>

<img width="300" alt="counting" src="https://user-images.githubusercontent.com/26691216/106792995-3b405280-669a-11eb-9f64-fc4d576200cd.gif">

<center><h2>_ _ _</h2></center>

<br/>

---

# CHAPTER 7 집계 연산
*집계(aggregation)은 무언갈 함께 모으는 행위이며 빅데이터 분석의 초석이다.*

스파크는 모든 데이터 타입 다루는 것 + 다음과 같은 **그룹화 데이터 타입** 생성 가능하고,
지정된 집계 함수에 따라 그룹화된 결과는 RelationalGroupedDataset 을 반환.

> - select 구문에서 집계 수행, DataFrame 전체 데이터 요약 (가장 간단한 그룹화)
> - 'group by' : 하나 이상의 키 지정. 다른 집계 함수 사용해서 값을 가진 컬럼 변환 가능
> - '윈도우(window)' : 하나 이상의 키 지정. 다른 집계 함수로 컬럼 변환 가능. + 단, 함수 입력으로 사용할 로우는 현재 로우와 **연관성** 있어야 함
> - '그룹화 셋(grouping set)' : 서로 다른 레벨 값 집계 (SQL, DataFrame의 롤업, 큐브)
> - '롤업(rollup)' : 하나 이상의 키 지정. 다른 집계 함수로 컬럼 변환 가능. + **계층적으로 요약된 값** 추출
> - '큐브(cube)' : 하나 이상의 키 지정. 다른 집계 함수로 컬럼 변환 가능. + **모든 컬럼 조합에 대한 요약 값** 계산
>
> <i style="color:lightgray">(=> 사실상 7장 요약)</i> 


📌 중요한 건, **어떤 결과를 만들지** 정확히 파악해야 한다는 것.
(정확한 답 계산 = 높은 비용 요구 → 빅데이터의 경우 근사치가 효율적일 수 있음)

<!-- 
count 예제.. (데이터셋 크기 출력 및 캐싱용도)
이질적으로 느껴질 수 있음. 왜나면...
- 함수가 아닌 메서드 형태
- 트랜스포메이션같은 지연 연산이아닌 즉시 연산(액션)
-->

### 7.1 집계 함수
- 모든 집계는 특별한 경우를 제외하고는 **함수** 사용
  - => **집계 함수** ([org.apache.spark.sql.functions](https://bit.ly/2tRMYus) 패키지)
  - 예외) DataFrame의 .stat 속성 이용 (6장 참고)
  - 스칼라, 파이썬에서 임포트 할 수 있는 함수와 SQL에서 사용가능한 함수는 약간 다름 (매 릴리즈마다 조금씩 변함)

#### 집계 함수
> 집계 함수 : 키나 그룹을 지정하고 + 하나 이상의 컬럼을 변환하는 방법을 지정 (여러 입력값이 주어지면 그룹 별로 결과 생성)
> - 수치형 데이터 요약 (ex. 그룹의 평균값 구하기)
> - 합산, 곱셈, 카운팅 등의 작업
> - 복합 데이터 타입(배열, 리스트, 맵)을 사용한 집계 수행 가능 

- `count(컬럼명)` : 전체 로우 수 카운트
  - 액션이 아닌 **트랜스포메이션**으로 동작
  - 두가지 방식으로 사용 가능
    - `count(특정 컬럼)` : null 값 포함 X
    - `count(*)` or `count(1)` : null 값 가진 로우 포함해서 카운트
  - <details><summary class="only-hover">[7.1.1] 예제 펼치기</summary>

    ```scala
    import org.apache.spark.sql.functions.count
    df.select(count("StockCode")).show() // 541909
    ```
    ```sql
    SELECT COUNT(*) FROM dfTable
    ```

    </details>

- `countDistinct(컬럼명)` : 고유 (distinct) 레코드 수 카운트
  - <details><summary class="only-hover">[7.1.2] 예제 펼치기</summary>

    ```scala
    import org.apache.spark.sql.functions.countDistinct
    df.select(countDistinct("StockCode")).show() // 4070
    ```
    ```sql
    SELECT COUNT(DISTINCT *) FROM DFTABLE
    ```

    </details>

- `approx_count_distinct(컬럼명, 최대추정오류율)` : 근사치 고유 레코드 수 카운트
  - 대규모 데이터셋 다룰 시 정확한 개수 무의미함 => 근사치로 효율
  - `최대 추정 오류율 (maximum estimation error)` 파라미터 설정
  - <details><summary class="only-hover">[7.1.3] 예제 펼치기</summary>

    ```scala
    import org.apache.spark.sql.functions.approx_count_distinct
    df.select(approx_count_distinct("StockCode", 0.1)).show() // 3364
    ```
    ```sql
    SELECT approx_count_distinct(StockCode, 0.1) FROM DFTABLE
    ```
    
    </details>


- `first(컬럼명)`, `last(컬럼명)` : 첫 번째 값, 마지막 값 추출
  - DataFrame 값이 아닌 로우 기반 동작
  - <details><summary class="only-hover">[7.1.4] 예제 펼치기</summary>

    ```scala
    df.select(first("StockCode"), last("StockCode")).show()
    // +-----------------------+----------------------+
    // |first(StockCode, false)|last(StockCode, false)|
    // +-----------------------+----------------------+
    // |                 85123A|                 22138|
    // +-----------------------+----------------------+
    ```
    ```sql
    SELECT first(StockCode), last(StockCode) FROM dfTable
    ```

    </details>

- `min(컬럼명)`, `max(컬럼명)` : 최솟값, 최댓값 추출
  - <details><summary class="only-hover">[7.1.5] 예제 펼치기</summary>

    ```scala
    import org.apache.spark.sql.functions.{min, max}
    df.select(min("Quantity"), max("Quantity")).show()
    // +-------------+-------------+
    // |min(Quantity)|max(Quantity)|
    // +-------------+-------------+
    // |       -80995|        80995|
    // +-------------+-------------+
    ```
    ```sql
    SELECT min(Quantity), max(Quantity) FROM dfTable
    ```

    </details>

- `sum(컬럼명)` : 특정 컬럼의 모든 값 합산
  - <details><summary class="only-hover">[7.1.6] 예제 펼치기</summary>

    ```scala
    import org.apache.spark.sql.functions.sum
    df.select(sum("Quantity")).show() // 5176450
    // +-------------+
    // |sum(Quantity)|
    // +-------------+
    // |      5176450|
    // +-------------+
    ```
    ```sql
    SELECT sum(Quantity) FROM dfTable
    ```

    </details>


- `sumDistinct(컬럼명)` : 특정 컬럼의 고유 (distinct) 값 합산
  - <details><summary class="only-hover">[7.1.7] 예제 펼치기</summary>

    ```scala
    import org.apache.spark.sql.functions.sumDistinct
    df.select(sumDistinct("Quantity")).show() // 29310
    // +----------------------+
    // |sum(DISTINCT Quantity)|
    // +----------------------+
    // |                 29310|
    // +----------------------+
    ```
    ```sql
    SELECT SUM(Quantity) FROM dfTable -- 29310
    ```

    </details>


- `avg(컬럼명)` : 평균 값
  - == `sum()/count()` == `expr("mean(컬럼명)")`
  - \+ `distinct()` => 고윳값 평균 구하기도 가능
  - <details><summary class="only-hover">[7.1.8] 예제 펼치기</summary>

    ```scala
    import org.apache.spark.sql.functions.{sum, count, avg, expr}
    (df.select(
        count("Quantity").alias("total_transactions"),
        sum("Quantity").alias("total_purchases"),
        avg("Quantity").alias("avg_purchases"),
        expr("mean(Quantity)").alias("mean_purchases"))
      .selectExpr(
        "total_purchases/total_transactions",
        "avg_purchases",
        "mean_purchases").show())
    // +--------------------------------------+----------------+----------------+
    // |(total_purchases / total_transactions)|   avg_purchases|  mean_purchases|
    // +--------------------------------------+----------------+----------------+
    // |                      9.55224954743324|9.55224954743324|9.55224954743324|
    // +--------------------------------------+----------------+----------------+
    ```

    </details>

- 분산과 표준편차
  - 평균(`m`) 주변에 데이터가 분포된 정도를 측정
    - 분산 : 평균과의 차이를 제곱한 결과의 평균 (`v = avg((x-m)^2)`)
    - 표준편차 : 분산의 제곱근 (`σ = v^(1/2)`)
  - 스파크는 표본표준편차(sample standard deviation), 모표준편차(population standard deviation) 방식 지원
    - => 아예 다르므로 **잘 구분해서 사용해야함**
  - 표본표준분산, 표본표준편차 방식 사용 시 => `variance()`, `stddev()`
  - 모표준분산, 모표준편차 방식 사용 시 => `var_pop()`, `stddev_pop()`
  - <details><summary class="only-hover">[7.1.9] 예제 펼치기</summary>

    ```scala
    df.select(var_pop("Quantity"), var_samp("Quantity"),
      stddev_pop("Quantity"), stddev_samp("Quantity")).show()
    // +-----------------+------------------+--------------------+---------------------+
    // |var_pop(Quantity)|var_samp(Quantity)|stddev_pop(Quantity)|stddev_samp(Quantity)|
    // +-----------------+------------------+--------------------+---------------------+
    // |47559.30364660923| 47559.39140929892|  218.08095663447835|   218.08115785023455|
    // +-----------------+------------------+--------------------+---------------------+
    ```
    ```sql
    SELECT var_pop(Quantity), var_samp(Quantity),
    stddev_pop(Quantity), stddev_samp(Quantity)
    FROM dfTable
    ```

    </details>

- 비대칭도와 첨도
  - 데이터의 변곡점(extreme point) 를 측정하는 방법
    - `skewness(컬럼명)` : 비대칭도 (데이터 평균의 비대칭 정도) 측정
    - `kurtosis(컬럼명)` : 첨도 (데이터 끝 부분의 뾰족한 정도) 측정
  - 확률변수(random variable)의 확률분포(probability distribution)로 데이터 모델링 시에 중요
  - 수학적인 내용은 따로 알아서... 흠흠..
  - <details><summary class="only-hover">[7.1.10] 예제 펼치기</summary>

    ```scala
    import org.apache.spark.sql.functions.{skewness, kurtosis}
    df.select(skewness("Quantity"), kurtosis("Quantity")).show()
    // +--------------------+------------------+
    // |  skewness(Quantity)|kurtosis(Quantity)|
    // +--------------------+------------------+
    // |-0.26407557610528376|119768.05495530753|
    // +--------------------+------------------+
    ```
    ```sql
    SELECT skewness(Quantity), kurtosis(Quantity) FROM dfTable
    ```

    </details>

- 공분산과 상관관계
  - 두 컬럼값 사이의 영향도 비교
  - `cov(컬럼1, 컬럼2)` : 공분산(covariance) 계산
    - 데이터 입력값에 따라 다른 범위를 가짐
    - var 함수처럼 표본공분산(sample covariance)이나 모공분산(population covariance) 방식으로도 계산 가능 => `covar_samp()`, `covar_pop()`
  - `corr(컬럼1, 컬럼2)` : 상관관계(correlation) 계산
    - 피어슨 상관계수 (Pearson correlation coefficient) 측정 (-1 <= `r` <= 1)
    - 모집단이나 표본에 대한 계산 개념 X
  - <details><summary class="only-hover">[7.1.11] 예제 펼치기</summary>

    ```scala
    import org.apache.spark.sql.functions.{corr, covar_pop, covar_samp}
    df.select(corr("InvoiceNo", "Quantity"), covar_samp("InvoiceNo", "Quantity"),
        covar_pop("InvoiceNo", "Quantity")).show()
    // +-------------------------+-------------------------------+------------------------------+
    // |corr(InvoiceNo, Quantity)|covar_samp(InvoiceNo, Quantity)|covar_pop(InvoiceNo, Quantity)|
    // +-------------------------+-------------------------------+------------------------------+
    // |     4.912186085640497E-4|             1052.7280543915997|            1052.7260778754955|
    // +-------------------------+-------------------------------+------------------------------+
    ```
    ```sql
    SELECT corr(InvoiceNo, Quantity), covar_samp(InvoiceNo, Quantity),
    covar_pop(InvoiceNo, Quantity)
    FROM dfTable
    ```

    </details>

- 복합 데이터 타입의 집계
  - 스파크는 수식을 통한 집계 외에도 **복합 데이터 타입**을 사용한 집계 가능 (ex. 특정 컬럼 값 => List, Set .. 등으로 수집)
  - 수집된 데이터는 다양한 프로그래밍 방식으로 다루거나 활용 가능
  - <details><summary class="only-hover">[7.1.12] 예제 펼치기</summary>

    ```scala
    import org.apache.spark.sql.functions.{collect_set, collect_list}
    df.agg(collect_set("Country"), collect_list("Country")).show()
    // +--------------------+---------------------+
    // |collect_set(Country)|collect_list(Country)|
    // +--------------------+---------------------+
    // |[Portugal, Italy,...| [United Kingdom, ...|
    // +--------------------+---------------------+
    ```
    ```sql
    SELECT collect_set(Country), collect_set(Country) FROM dfTable
    ```

    </details>

### 7.2 그룹화
- <u>데이터 **그룹** 기반의 집계</u> 에 대한 내용
  - ([7.1] 은 DataFrame 수준의 집계 내용)
  - 카테고리형 데이터(categorical data) 사용
  - => 단일 컬럼의 데이터를 그룹화, 해당 그룹의 다른 여러 컬럼을 사용해서 계산
- 그룹화 작업의 2 단계
  - 1\) 하나 이상의 컬럼 그룹화 (여러개 지정도 가능)
  - 2\) 집계 연산 수행
- 표현식을 이용한 그룹화
  - 카운팅은 메서드, 함수 둘 다 사용 가능 🤔
    - 메서드 보다 `count()` 함수 사용 추천
    - select 구문의 표현식 지정보다 `agg()` 메서드 사용 추천
  - `agg()` : 여러 집계 처리 한번에 지정 & 집계에 표현식 사용 가능
    - 트랜스포메이션 완료 컬럼에 `alias` 사용 가능
- 맵을 이용한 그룹화
  - 맵(map) 타입 사용 : Key = 컬럼 / Value = 수행할 집계 함수의 문자열
  - 수행할 집계함수를 한 줄로 작성 시 => 여러 컬럼명 재사용 가능
    - `agg(Key -> Value, Key -> Value, ...)`

### 7.3 윈도우 함수
- **윈도우 함수** 도 집계에 사용 가능
- 윈도우 함수
  - 데이터의 특정 '윈도우(window)' 대상으로 고유의 집계 연산 수행
  - 데이터의 '윈도우' => 현재 데이터에 대한 참조(reference)를 사용해 정의
  - 윈도우 명세(window specification) => 함수에 전달될 로우 결정
- 스파크가 지원하는 윈도우 함수
  - 랭크 함수 (ranking function)
  - 분석 함수 (analytic function)
  - 집계 함수 (aggragate function)
- 윈도우 함수 vs group-by 함수
  - 윈도우 함수 : **프레임**에 입력되는 모든 로우에 대해 결과값 계산
  - group-by 함수 : 모든 로우 레코드가 단일 그룹으로만 이동
- 프레임(frame) : 로우 그룹 기반의 테이블
  - 각 로우는 하나 이상의 프레임에 할당 가능
    <img width="300" alt="row - window frame" src="https://user-images.githubusercontent.com/26691216/106770674-8b5eeb00-6681-11eb-929f-09e7c373f9a2.png">
  - 프레임 정의 방법은 예제 참고
- ex. 하루를 나타내는 값의 롤링 평균(rolling average) 구하기
  - 개별 로우가 7개의 다른 프레임으로 구성되어야 함

<details><summary class="point-color-can-hover">[7.3] 예제 펼치기</summary>

```scala
// 1) 주문 일자(InvoiceDate) => 'date' 컬럼으로 변환 (날짜 정보만 포함)
import org.apache.spark.sql.functions.{col, to_date}

val dfWithDate = df.withColumn("date", to_date(col("InvoiceDate"),
  "MM/d/yyyy H:mm"))
dfWithDate.createOrReplaceTempView("dfWithDate")


// 2) 윈도우 명세 만들기
import org.apache.spark.sql.expressions.Window
import org.apache.spark.sql.functions.col

val windowSpec = (Window
  .partitionBy("CustomerId", "date")  // 그룹을 어떻게 나눌지 결정
  .orderBy(col("Quantity").desc)    // 파티션 정렬 방식
  .rowsBetween(Window.unboundedPreceding, Window.currentRow)) // 프레임 명세 (=> 첫 로우 ~ 현재 로우까지 확인)


// 3) 집계 함수로 분석
// => 컬럼 or 표현식 반환 시 DataFrame.select() 에서 사용 가능

// 예시 1. maxPurchaseQuantity = 시간대별 최대 구매 개수
import org.apache.spark.sql.functions.max
val maxPurchaseQuantity = max(col("Quantity")).over(windowSpec)
// maxPurchaseQuantity: org.apache.spark.sql.Column = max(Quantity) OVER (PARTITION BY CustomerId, date ORDER BY Quantity DESC NULLS LAST ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)

// 예시 2. purchase(Dense)Rank = 구매량 순위 
import org.apache.spark.sql.functions.{dense_rank, rank}
val purchaseDenseRank = dense_rank().over(windowSpec) // 순위가 비지않도록 dense_rank() 사용
val purchaseRank = rank().over(windowSpec)

// DataFrame.select()로 윈도우 값 확인
import org.apache.spark.sql.functions.col

(dfWithDate.where("CustomerId IS NOT NULL").orderBy("CustomerId")
  .select(
    col("CustomerId"),
    col("date"),
    col("Quantity"),
    purchaseRank.alias("quantityRank"),
    purchaseDenseRank.alias("quantityDenseRank"),
    maxPurchaseQuantity.alias("maxPurchaseQuantity")).show())
// +----------+----------+--------+------------+-----------------+-------------------+
// |CustomerId|      date|Quantity|quantityRank|quantityDenseRank|maxPurchaseQuantity|
// +----------+----------+--------+------------+-----------------+-------------------+
// |     12346|2011-01-18|   74215|           1|                1|              74215|
// |     12346|2011-01-18|  -74215|           2|                2|              74215|
// |     12347|2010-12-07|      36|           1|                1|                 36|
// |     12347|2010-12-07|      30|           2|                2|                 36|
// |     12347|2010-12-07|      24|           3|                3|                 36|
// |     12347|2010-12-07|      12|           4|                4|                 36|
// |     12347|2010-12-07|      12|           4|                4|                 36|
// |     12347|2010-12-07|      12|           4|                4|                 36|
// |     12347|2010-12-07|      12|           4|                4|                 36|
// |     12347|2010-12-07|      12|           4|                4|                 36|
// |     12347|2010-12-07|      12|           4|                4|                 36|
// |     12347|2010-12-07|      12|           4|                4|                 36|
// |     12347|2010-12-07|      12|           4|                4|                 36|
// |     12347|2010-12-07|      12|           4|                4|                 36|
// |     12347|2010-12-07|      12|           4|                4|                 36|
// |     12347|2010-12-07|      12|           4|                4|                 36|
// |     12347|2010-12-07|      12|           4|                4|                 36|
// |     12347|2010-12-07|      12|           4|                4|                 36|
// |     12347|2010-12-07|       6|          17|                5|                 36|
// |     12347|2010-12-07|       6|          17|                5|                 36|
// +----------+----------+--------+------------+-----------------+-------------------+
```

```sql
-- SQL
SELECT CustomerId, date, Quantity,
  rank(Quantity) OVER (PARTITION BY CustomerId, date
                       ORDER BY Quantity DESC NULLS LAST
                       ROWS BETWEEN
                         UNBOUNDED PRECEDING AND
                         CURRENT ROW) as rank,

  dense_rank(Quantity) OVER (PARTITION BY CustomerId, date
                             ORDER BY Quantity DESC NULLS LAST
                             ROWS BETWEEN
                               UNBOUNDED PRECEDING AND
                               CURRENT ROW) as dRank,

  max(Quantity) OVER (PARTITION BY CustomerId, date
                      ORDER BY Quantity DESC NULLS LAST
                      ROWS BETWEEN
                        UNBOUNDED PRECEDING AND
                        CURRENT ROW) as maxPurchase
FROM dfWithDate WHERE CustomerId IS NOT NULL ORDER BY CustomerId
```

</details>

> Window 메서드
> - `partitionBy()` : 그룹을 어떻게 나눌지 결정 (지금까지 파티셔닝 스키마 개념이랑 관련 X)
> - `orderBy()` : 파티션 정렬 방식 정의
> - `rowsBetween(from, to)` : 입력된 로우의 참조 기반으로 프레임에 로우가 포함될 수 있는지 결정
>
> row_number vs **rank** vs **dense_rank**
> - `row_number()` : 순서대로 넘버링 (1,2,3,4 ...)
> - `rank()` : 순서대로 넘버링 + 같은 값일 경우 같은 숫자 (1,1,3,4 ...)
> - `dense_rank()` : rank 와 동일하되, 빈값 없이 증가하게끔 넘버링 (1,1,2,3, ...)

### 7.4 그룹화 셋
- 컬럼의 값을 이용해 여러 컬럽 집계 => `group-by` 표현식
  - 그러면 **여러 그룹**에 걸쳐 집계는? => **그룹화셋**  사용
- 그룹화 셋 : 여러 집계를 결합하는 저수준 기능
  - `GROUPING SETS` 구문은 SQL에서만 사용 가능
  - DataFrame에서 동일 연산하려면? => **롤업**, **큐브** 메서드 사용
- 주의 사항
  - 그룹화 셋, 롤업, 큐브 사용 시 **null 제거 필수**
  - null에 따라 집계 수준이 달라짐 (=> null 미제거시 부정확한 결과)
  

<details><summary class="point-color-can-hover">[7.4.0] '그룹화 셋' 예제 펼치기</summary>

```scala
// 그룹화 셋 사용 시 null 제거 필수
val dfNoNull = dfWithDate.drop()
dfNoNull.createOrReplaceTempView("dfNoNull")
```

```sql
-- SQL 예제 1) 재고 코드(StockCode)와 고객(CustomerId) 별 총 수량 구하기
SELECT CustomerId, stockCode, sum(Quantity) FROM dfNoNull
GROUP BY customerId, stockCode
ORDER BY CustomerId DESC, stockCode DESC

-- (그룹화 셋 사용한 동일 표현)
SELECT CustomerId, stockCode, sum(Quantity) FROM dfNoNull
GROUP BY customerId, stockCode GROUPING SETS((customerId, stockCode))
ORDER BY CustomerId DESC, stockCode DESC
```

```sql
-- SQL 예제 2) 예제 1 + 재고 코드나 고객 상관없이 총 수량 합산 결과 추가 => group-by로 처리 불가
-- 그룹화 셋으로 집계 방식 지정
SELECT CustomerId, stockCode, sum(Quantity) FROM dfNoNull
GROUP BY customerId, stockCode GROUPING SETS((customerId, stockCode),())
ORDER BY CustomerId DESC, stockCode DESC
```

```bash
# output
+----------+---------+-------------+
|CustomerId|stockCode|sum(Quantity)|
+----------+---------+-------------+
|     18287|    85173|           48|
|     18287|   85040A|           48|
|     18287|   85039B|          120|
...
|     18287|    23269|           36|
+----------+---------+-------------+
```

</details>

- 롤업(rollup)
  - `group-by` 스타일의 다양한 연산을 수행할 수 있는 다차원 집계 기능
  - `rollup(그룹화 키)` => 다양한 컬럼을 그룹화 키로 설정 가능
  - 롤업된 컬럼값이 모두 null 인 로우 = 해당 컬럼에 속한 레코드의 전체 합계

<details><summary class="point-color-can-hover">[7.4.1] '롤업' 예제 펼치기</summary>

```scala
// 시간(신규 Date 컬럼), 공간(Country) 을 축으로 하는 롤업
// => '모든 날짜 총합', '날짜별 총합', '날짜별 국가별 총합' 포함하는 DataFrame 생성
val rolledUpDF = (dfNoNull.rollup("Date", "Country").agg(sum("Quantity"))
  .selectExpr("Date", "Country", "`sum(Quantity)` as total_quantity")
  .orderBy("Date"))
rolledUpDF.show()
// +----------+--------------+--------------+
// |      Date|       Country|total_quantity|
// +----------+--------------+--------------+
// |      null|          null|       5176450| => 전체 합계
// |2010-12-01|   Netherlands|            97|
// |2010-12-01|       Germany|           117|
// |2010-12-01|     Australia|           107|
// |2010-12-01|        France|           449|
// |2010-12-01|          EIRE|           243|
// |2010-12-01|United Kingdom|         23949|
// |2010-12-01|          null|         26814|
// |2010-12-01|        Norway|          1852|
// |2010-12-02|          EIRE|             4|
// |2010-12-02|          null|         21023|
// |2010-12-02|       Germany|           146|
// |2010-12-02|United Kingdom|         20873|
// |2010-12-03|        France|           239|
// |2010-12-03|      Portugal|            65|
// |2010-12-03|       Germany|           170|
// |2010-12-03|       Belgium|           528|
// |2010-12-03|         Spain|           400|
// |2010-12-03|         Italy|           164|
// |2010-12-03|   Switzerland|           110|
// +----------+--------------+--------------+


// Country, Date 둘 다 null 인 로우 => 전체 합계 나타냄
rolledUpDF.where("Country IS NULL").show()
rolledUpDF.where("Date IS NULL").show()
// +----+-------+--------------+
// |Date|Country|total_quantity|
// +----+-------+--------------+
// |null|   null|       5176450|
// +----+-------+--------------+
```

</details>

- 큐브(cube)
  - 롤업의 고차원적 사용 (호출 방식도 유사)
  - 요소들을 계층적으로 다루는 대신 모든 차원에 대해 동일한 작업 수행
  - ex. 전체 기간에 대한 날짜와 국가별 결과 구하기
  - `cube(그룹화 키)` => 요약 정보 테이블 만들기 가능

<details><summary class="point-color-can-hover">[7.4.2] '큐브' 예제 펼치기</summary>

```scala
// 시간(신규 Date 컬럼), 공간(Country) 을 축으로 하는 큐브
// => '전체 기간에 대한 날짜와 국가별 결과' 포함하는 DataFrame 생성
//     (외에도 전체 날짜와 모든 국가에 대한 합계, 모든 국가의 날짜별 합계, 날짜별 국가별 합계, 전체 날짜의 국가별 합계, ... 가능)
(dfNoNull.cube("Date", "Country").agg(sum(col("Quantity")))
  .select("Date", "Country", "sum(Quantity)").orderBy("Date").show())
// +----+--------------------+-------------+
// |Date|             Country|sum(Quantity)|
// +----+--------------------+-------------+
// |null|               Japan|        25218|
// |null|            Portugal|        16180|
// |null|             Germany|       117448|
// |null|                 RSA|          352|
// |null|           Australia|        83653|
// |null|           Hong Kong|         4769|
// |null|              Cyprus|         6317|
// |null|             Finland|        10666|
// |null|United Arab Emirates|          982|
// |null|                null|      5176450|
// |null|         Unspecified|         3300|
// |null|               Spain|        26824|
// |null|           Singapore|         5234|
// |null|     Channel Islands|         9479|
// |null|             Lebanon|          386|
// |null|                 USA|         1034|
// |null|             Denmark|         8188|
// |null|              Norway|        19247|
// |null|      Czech Republic|          592|
// |null|  European Community|          497|
// +----+--------------------+-------------+
```

</details>

- 그룹화 메타데이터
  - 큐브, 롤업 사용 시 집계 수준에 따라 쉽게 필터링하고자 하면 => **집계 수준 조회** 필요
  - `grouping_id()` : 결과 데이터셋의 집계 수준을 명시하는 컬럼 제공

<details><summary class="point-color-can-hover">[7.4.3] grouping_id() 예제 펼치기</summary>

```scala
import org.apache.spark.sql.functions.{grouping_id, sum, expr}

(dfNoNull.cube("customerId", "stockCode").agg(grouping_id(), sum("Quantity"))
.orderBy(col("grouping_id()").desc)
.show())
// => 4개의 개별 그룹화 ID 값 (0,1,2,3) 반환됨
// +----------+---------+-------------+-------------+
// |customerId|stockCode|grouping_id()|sum(Quantity)|
// +----------+---------+-------------+-------------+
// |      null|     null|            3|      5176450| => 3 : 가장 높은 계층의 집계 결과. 전체 총 수량 (customerId, stockCode 관계 X)
// |      null|    84226|            2|           17| => 2 : 개별 stockCode 별 총 수량 (customerId 관계 X)
// |      null|    22856|            2|          518|
// |      null|    22352|            2|         3077|
//            ...
// |     14907|     null|            1|         1686| => 1 : customerId 기반 총 수량 제공 (구매 물품 관계 X)
// |     14543|     null|            1|          600|
//            ...
// |     13047|    22749|            0|           12| => 0 : customerId - stockCode 별 조합에 따라 총 수량
// |     15311|    22083|            0|          169|
// +----------+---------+-------------+-------------+
```

</details>

- 피벗(pivot)
  - `pivot()` : 로우 → 컬럼으로 변환 가능
  - => 컬럼의 모든 값을 단일 그룹화하여 계산 가능
    - 그러나 데이터 탐색방식에 따라 피벗 수행 결과값이 감소할 수 있음
  - 특정 컬럼 cardinality가 낮으면 피벗으로 다수 컬럼으로 변환 추천   => 스키마, 쿼리 대상 확인 가능

<details><summary class="point-color-can-hover">[7.4.4] '피벗' 예제 펼치기</summary>

```scala
val pivoted = dfWithDate.groupBy("date").pivot("Country").sum() // 집계 수행 => 수치형 컬럼으로 나옴
// pivoted.printSchema()
// root
//  |-- date: date (nullable = true)
//  |-- Australia_sum(Quantity): long (nullable = true)
//  |-- Australia_sum(UnitPrice): double (nullable = true)
//  |-- Australia_sum(CustomerID): long (nullable = true)
//  |-- Austria_sum(Quantity): long (nullable = true)
//  |-- Austria_sum(UnitPrice): double (nullable = true)
//  |-- Austria_sum(CustomerID): long (nullable = true)
//  |-- Bahrain_sum(Quantity): long (nullable = true)
//  |-- Bahrain_sum(UnitPrice): double (nullable = true)
//  |-- Bahrain_sum(CustomerID): long (nullable = true)
//  ...

pivoted.where("date > '2011-12-05'").select("date" ,"`USA_sum(Quantity)`").show()
// +----------+-----------------+
// |      date|USA_sum(Quantity)|
// +----------+-----------------+
// |2011-12-06|             null|
// |2011-12-09|             null|
// |2011-12-08|             -196|
// |2011-12-07|             null|
// +----------+-----------------+
```

</details>

### 7.5 사용자 정의 집계 함수
- 사용자 정의 집계 함수 (**UDAF**, user-defined aggregation function)
  - 직접 제작한 함수나 비즈니스 규칙에 기반한 자체 집계 함수 정의 방법
  - UDAF 사용 => 입력 데이터 그룹에 직접 개발한 연산 수행 가능
  - 스파크는 입력 데이터의 모든 그룹 중간 결과를 단일 AggregationBuffer에 저장/관리
- UDAF는 현재 스칼라, 자바로만 사용 가능
  - Spark 2.3 에서는 UDF/UDAF => 함수 등록 가능 ([6.12] 참고)
- 생성 방법
  - 기본 클래스 UserDefinedAggregateFunction 상속 + 메서드 정의

> UDAF 생성 시 정의해야할 메서드
> - inputScheme : `UDAF 입력 파라미터의 스키마`를 StructType 로 정의
> - bufferSchema : `UDAF 중간 결과의 스키마`를 StructType 로 정의
> - dataType : `반환될 값의 DataType` 정의
> - deterministic : `UDAF가 동일한 입력값에 대해 항상 동일한 결과를 반환하는지` Boolean 값으로 정의
> - initialize : `집계용 버퍼 값 초기화 로직` 정의
> - update : 입력받은 로우 기반으로 `내부 버퍼 업데이트 로직` 정의
> - merge : 두 개의 `집계용 버퍼 병합 로직` 정의
> - evaluate : `집계 최종 결과 생성 로직` 정의

<details><summary class="point-color-can-hover">[7.5] 예제 펼치기</summary>

```scala
// UDAF 예제 - 'BoolAnd' Class : 입력된 모든 로우의 컬럼이 true인지 판단
class BoolAnd extends org.apache.spark.sql.expressions.UserDefinedAggregateFunction {
  import org.apache.spark.sql.types.{StructType, StructField, BooleanType, DataType}
  import org.apache.spark.sql.expressions.MutableAggregationBuffer
  import org.apache.spark.sql.Row

  def inputSchema: org.apache.spark.sql.types.StructType =
    StructType(StructField("value", BooleanType) :: Nil)
  def bufferSchema: StructType = StructType(
    StructField("result", BooleanType) :: Nil
  )
  def dataType: DataType = BooleanType
  def deterministic: Boolean = true
  def initialize(buffer: MutableAggregationBuffer): Unit = {
    buffer(0) = true
  }
  def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
    buffer(0) = buffer.getAs[Boolean](0) && input.getAs[Boolean](0)
  }
  def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
    buffer1(0) = buffer1.getAs[Boolean](0) && buffer2.getAs[Boolean](0)
  }
  def evaluate(buffer: Row): Any = {
    buffer(0)
  }
}


// 함수로 등록 및 사용
val ba = new BoolAnd
spark.udf.register("booland", ba)
import org.apache.spark.sql.functions._
(spark.range(1)
  .selectExpr("explode(array(TRUE, TRUE, TRUE)) as t")
  .selectExpr("explode(array(TRUE, FALSE, TRUE)) as f", "t")
  .select(ba(col("t")), expr("booland(f)"))
  .show())
// +----------+----------+
// |booland(t)|booland(f)|
// +----------+----------+
// |      true|     false|
// +----------+----------+
```

</details>


### 7.6 정리
- 스파크에서 사용 가능한 여러 유형의 집계 연산
- 그룹화, 윈도우 함수, 롤업, 큐브

### 📒 단어장
- 비대칭도(skewness) : 실숫값 확률변수의 확률분포 비대칭성을 나타내는 지표 (=왜도)
- 첨도(kurtosis) : 확률분포의 뾰족한 정도를 나타내는 척도
    