---
title: '&#039;Spark The Definitive Guide&#039; 6장 - 데이터 타입 (비)공식 가이드북'
date: 2021-02-02 00:46:40
categories: spark
tags:
  - spark
  - apache
  - book
  - study
---

<br/>

<center><p style="color:lightgray">라떼 시절엔,, 가이드북이 하나면 든-든했다,,, 이말이야,,, 총총 @}----</p>
<img width="300" alt="maple" src="https://user-images.githubusercontent.com/26691216/106801127-b149b700-66a4-11eb-9c8f-0802771ebe5f.jpg">
<i>'아파치 스파크' 미인증 비공식 가이드 북<br/>
[전원 증정] 50.00 페이지 포인트 (캐시 아이템 구매 가능)</i></center>


<br/>

<center><h2>_ _ _</h2></center>

<br/>

---

# CHAPTER 6 다양한 데이터 타입 다루기

CHAPTER 5 는 DataFrame의 기본 개념과 핵심 추상화 개념을 소개
CHAPTER 6 는 스파크의 구조적 연산에서 가장 중요한 내용인 **표현식 만드는 방법** 소개 + 다양한 데이터 타입 다루는 방법

> 다양한 데이터 타입
>
> - Boolean 타입
> - 수치 타입
> - 문자열 타입
> - date와 timestamp 타입
> - null 값 다루기
> - 복합 데이터 아입
> - 사용자 정의 함수


### 6.1 API는 어디서 찾을까
- 오늘은 언젠가 내일이 된다
  - 버전 바뀌면 책의 내용도 다 예전 내용이다~ 이말이야
  - => 따라서 <u>데이터 변환용 함수 찾는 방법</u> 을 알아야함
- 어떻게 찾나?
  - DataFrame (Dataset) 메서드
    - DatasFrame은 Row타입을 가진 Dataset => [Dataset API](http://bit.ly/2rKkALY)
    - 다양한 메서드를 제공하는 Dataset 하위 모듈 (ex. [DataFrameStateFunctions](http:bit.ly/2DPYhJC) - 통계적 함수 제공, [DataFrameNaFunctions](http://bit.ly/2DPAqd3) - null 데이터 제어)
  - Column 메서드
    - `alias` `contains` 등의 컬럼 관련 메서드 제공 => [Columns API](http://bit.ly/2FloFbr)
    - `org.apache.spark.sql.functions`는 데이터 타입 관련 다양한 함수 제공 (ex. [SQL, DataFrame 함수 등](http://bit.ly/2DPAycx))
- 모든 함수는 데이터 로우의 특정 포맷이나 구조를 다른 형태로 변환하기 위해 존재
  - 함수로 더 많은 로우를 만들거나 줄일 수 O

<details><summary class="point-color-can-hover">[6.1] 예제 펼치기 - 분석용 DataFrame 생성 예제</summary>

```scala
val df = (spark.read.format("csv")
  .option("header", "true")
  .option("inferSchema", "true")
  .load("/data/retail-data/by-day/2010-12-01.csv"))
df.printSchema()
df.createOrReplaceTempView("dfTable")
// df: org.apache.spark.sql.DataFrame = [InvoiceNo: string, StockCode: string ... 6 more fields]
// root
//  |-- InvoiceNo: string (nullable = true)
//  |-- StockCode: string (nullable = true)
//  |-- Description: string (nullable = true)
//  |-- Quantity: integer (nullable = true)
//  |-- InvoiceDate: timestamp (nullable = true)
//  |-- UnitPrice: double (nullable = true)
//  |-- CustomerID: double (nullable = true)
//  |-- Country: string (nullable = true)

```

</details>


### 6.2 스파크 데이터 타입으로 변환하기
- `lit()` : 데이터 타입 변환
  - 다른 프로그래밍 언어 고유 데이터 타입 => **스파크 데이터 타입** 변환

<details><summary class="point-color-can-hover">[6.2] 예제 펼치기</summary>

```scala
import org.apache.spark.sql.functions.lit

df.select(lit(5), lit("five"), lit(5.0))
// res9: org.apache.spark.sql.DataFrame = [5: int, five: string ... 1 more field]
```

```sql
-- SQL (SQL은 스파크 데이터 타입 변환 필요 X. 직접 값 입력)
SELECT 5, "five", 5.0
```

</details>

### 6.3 불리언 데이터 타입 다루기

- 불리언은 모든 필터링 작업의 기반 (데이터 분석에 필수)
- 불리언 구문 : `and`, `or`, `true`, `false`
  - 불리언 구문으로 논리 문법(true/false) 생성
- **스칼라** 사용 시 동등 여부
  - `===` (일치) / `=!=` (불일치)
  - `not()`, `equalTO()` 사용 가능
- 파이썬, 스칼라 모두 사용할 수 있는
  - 가장 명확한 방법? => <u>문자열 표현식에 조건절 명시</u> (ex. `where("InvoiceNo = 536353)`)
    <details><summary class="point-color-can-hover">[6.3-1] 예제 펼치기 - 문자열 표현식에 조건절 명시 </summary>

    ```scala
    import org.apache.spark.sql.functions.col

    // 같은 표현식 (in Scala)
    (df.where(col("InvoiceNo").equalTo(536365))
      .select("InvoiceNo", "Description")
      .show(5, false))
    (df.where(col("InvoiceNo") === 536365)
      .select("InvoiceNo", "Description")
      .show(5, false))
    // +---------+-----------------------------------+
    // |InvoiceNo|Description                        |
    // +---------+-----------------------------------+
    // |536365   |WHITE HANGING HEART T-LIGHT HOLDER |
    // |536365   |WHITE METAL LANTERN                |
    // |536365   |CREAM CUPID HEARTS COAT HANGER     |
    // |536365   |KNITTED UNION FLAG HOT WATER BOTTLE|
    // |536365   |RED WOOLLY HOTTIE WHITE HEART.     |
    // +---------+-----------------------------------+


    // 문자열 표현식에 조건절 명시 (가장 명확한 방법) 사용
    (df.where("InvoiceNo = 536365")
      .show(5, false))

    (df.where("InvoiceNo <> 536365")
      .show(5, false))

    // +---------+---------+-----------------------------------+--------+-------------------+---------+----------+--------------+
    // |InvoiceNo|StockCode|Description                        |Quantity|InvoiceDate        |UnitPrice|CustomerID|Country       |
    // +---------+---------+-----------------------------------+--------+-------------------+---------+----------+--------------+
    // |536365   |85123A   |WHITE HANGING HEART T-LIGHT HOLDER |6       |2010-12-01 08:26:00|2.55     |17850.0   |United Kingdom|
    // |536365   |71053    |WHITE METAL LANTERN                |6       |2010-12-01 08:26:00|3.39     |17850.0   |United Kingdom|
    // |536365   |84406B   |CREAM CUPID HEARTS COAT HANGER     |8       |2010-12-01 08:26:00|2.75     |17850.0   |United Kingdom|
    // |536365   |84029G   |KNITTED UNION FLAG HOT WATER BOTTLE|6       |2010-12-01 08:26:00|3.39     |17850.0   |United Kingdom|
    // |536365   |84029E   |RED WOOLLY HOTTIE WHITE HEART.     |6       |2010-12-01 08:26:00|3.39     |17850.0   |United Kingdom|
    // +---------+---------+-----------------------------------+--------+-------------------+---------+----------+--------------+

    // +---------+---------+-----------------------------+--------+-------------------+---------+----------+--------------+
    // |InvoiceNo|StockCode|Description                  |Quantity|InvoiceDate        |UnitPrice|CustomerID|Country       |
    // +---------+---------+-----------------------------+--------+-------------------+---------+----------+--------------+
    // |536366   |22633    |HAND WARMER UNION JACK       |6       |2010-12-01 08:28:00|1.85     |17850.0   |United Kingdom|
    // |536366   |22632    |HAND WARMER RED POLKA DOT    |6       |2010-12-01 08:28:00|1.85     |17850.0   |United Kingdom|
    // |536367   |84879    |ASSORTED COLOUR BIRD ORNAMENT|32      |2010-12-01 08:34:00|1.69     |13047.0   |United Kingdom|
    // |536367   |22745    |POPPY'S PLAYHOUSE BEDROOM    |6       |2010-12-01 08:34:00|2.1      |13047.0   |United Kingdom|
    // |536367   |22748    |POPPY'S PLAYHOUSE KITCHEN    |6       |2010-12-01 08:34:00|2.1      |13047.0   |United Kingdom|
    // +---------+---------+-----------------------------+--------+-------------------+---------+----------+--------------+
    ```

    </details>

- 불리언 표현식 사용하는 경우
  - 항상 모든 표현식을 `and` 메서드로 묶어 차례대로 필터 적용 해야 함
  - why?
    - 스파크 내부적으로 필터 사이에 `and` 구문 추가 시
    - => 모든 필터를 하나의 문장으로 변환하여 **동시에 모든 필터 처리** 하기 때문
  - `and` 구문 사용 시
    - `and` 구문으로 조건문을 만들 수는 있으나,
    - 차례대로 조건 나열하는게 가독성이 좋음
  - `or` 구문 사용시
    - 반드시 동일한 구문에 조건 정의해야 함

    <details><summary class="point-color-can-hover">[6.3-2] 예제 펼치기 - 불리언 표현식으로 필터링 적용 </summary>

    ```scala
    val priceFilter = col("UnitPrice") > 600
    val descripFilter = col("Description").contains("POSTAGE")
    (df.where(col("StockCode").isin("DOT")).where(priceFilter.or(descripFilter))
      .show())
    // priceFilter: org.apache.spark.sql.Column = (UnitPrice > 600)
    // descripFilter: org.apache.spark.sql.Column = contains(Description, POSTAGE)
    // +---------+---------+--------------+--------+-------------------+---------+----------+--------------+
    // |InvoiceNo|StockCode|   Description|Quantity|        InvoiceDate|UnitPrice|CustomerID|       Country|
    // +---------+---------+--------------+--------+-------------------+---------+----------+--------------+
    // |   536544|      DOT|DOTCOM POSTAGE|       1|2010-12-01 14:32:00|   569.77|      null|United Kingdom|
    // |   536592|      DOT|DOTCOM POSTAGE|       1|2010-12-01 17:06:00|   607.49|      null|United Kingdom|
    // +---------+---------+--------------+--------+-------------------+---------+----------+--------------+
    ```

    ```sql
    -- SQL (동일 표현)
    SELECT * FROM dfTable WHERE StockCode in ("DOT") AND(UnitPrice > 600 OR
        instr(Description, "POSTAGE") >= 1)
    ```

    </details>

- 불리언 표현식은...
  - 필터링 조건에만 사용? => 🙅🏻‍♀️. 불리언컬럼으로 DataFrame 필터링도 가능
  - 반드시 표현식으로 정의해야? => 🙅🏻‍♀️. 별도 작업없이 컬럼명만 사용해서 정의도 가능
  - 사실 SQL로 표현하는게 더 익숙할지도.. (성능저하 X)
- NULL 값 데이터 처리?
  - => **null-safe** 동치(equivalence) 테스트
  - ex. `df.where(col("Description").eqNullSafe("hello")).show()`
- SQL의 `IS [NOT] DISTINCT FROM` 구문
  - 과 동일한 기능이 뭘 말하나.. => `isNotDistinctFrom()` `isDistinctFrom()`? (지금도 사용하는지?)
  - since Spark 2.3 ([이슈 참고](https://bit.ly/2x47Obk))

  <details><summary class="point-color-can-hover">[6.3-3] 예제 펼치기 - 불리언컬럼으로 DataFrame 필터링</summary>

  ```scala
  // DataFrame 필터링 예제
  val DOTCodeFilter = col("StockCode") === "DOT"
  val priceFilter = col("UnitPrice") > 600
  val descripFilter = col("Description").contains("POSTAGE")
  (df.withColumn("isExpensive", DOTCodeFilter.and(priceFilter.or(descripFilter)))
    .where("isExpensive")
    .select("unitPrice", "isExpensive").show(5))
  // DOTCodeFilter: org.apache.spark.sql.Column = (StockCode = DOT)
  // priceFilter: org.apache.spark.sql.Column = (UnitPrice > 600)
  // descripFilter: org.apache.spark.sql.Column = contains(Description, POSTAGE)
  // +---------+-----------+
  // |unitPrice|isExpensive|
  // +---------+-----------+
  // |   569.77|       true|
  // |   607.49|       true|
  // +---------+-----------+
  ```
  ```sql
  -- SQL (동일 표현)
  SELECT UnitPrice, (StockCode = 'DOT' AND
    (UnitPrice > 600 OR instr(Description, "POSTAGE") >= 1)) as isExpensive
  FROM dfTable
  WHERE (StockCode = 'DOT' AND
         (UnitPrice > 600 OR instr(Description, "POSTAGE") >= 1))
  ```

  ```scala
  // 필터는 SQL로 사용시 더 편리할 수도. (아래 두 문장 동일하게 처리됨)
  import org.apache.spark.sql.functions.{expr, not, col}

  (df.withColumn("isExpensive", not(col("UnitPrice").leq(250)))
    .filter("isExpensive")
    .select("Description", "UnitPrice").show(5))
  (df.withColumn("isExpensive", expr("NOT UnitPrice <= 250"))
    .filter("isExpensive")
    .select("Description", "UnitPrice").show(5))

  ```

  </details>


### 6.4 수치형 데이터 타입 다루기

- `count()` 
  - 빅데이터 처리 시, 필터링 다음으로 많이 수행하는 작업
  - 수치형 데이터 타입을 사용한 연산 방식 정의
- 자주 사용하는 수치형 함수
  - `pow(밑, 지수)` (거듭제곱)
  - `round()` (반올림), `bround()` (내림)
  - `corr()` => 피어슨 상관계수 계산 (= 두 컬럼의 상관관계)
  - `describe()` => 관련 컬럼에 대한 집계(count), 평균(mean), 표준편차(stddev), 최솟값(min), 최댓값(max) 등 계산
    - 하나 이상의 컬럼에대한 요약 통계 계산
    - 그러나 콘솔 확인용으로만 사용해야함 (통계 스키마는 변경 될 수 있음)
    - 정확한 수치 필요 시 => 해당 함수 임포트해서 적용하는 방식으로 **직접 집계**
- **StatFunction** 패키지 => 다양한 통계 함수 제공
  - 다양한 통계값 계산에 사용하는 DataFrame 메서드 => `df.stat` 으로 접근
  - `approxQuantile()` : 데이터 백분위수 계산 (정확하게 or 근사치로?)
  - `crosstab()` : 교차표(cross-tabulation) 확인
  - `freqItems()` : 자주 사용하는 항목 쌍 확인
    - crosstab, freqItems 등은 결과가 너무 크면 다 출력 X
  - `monotonically_increasing_id()` : 모든 로우에 고유 ID 값 추가 (0 ~ )
- 스파크 새로운 버전 나올 때마다 새로운 함수 생김
  - => 스파크 공식 문서 참조
    - ex. `rand()`, `randn()` (임의 데이터 생성 함수)
  - 최신 버전 StatFunction 패키지는 여러 고급 기법 관련 함수 제공하기도
    - bloom 필터링, sketching algorithms ..
    - 자세한 내용은 [API docs](http://bit.ly/2ptAiY2)
    - (사실 현시점 최신버전은 아니고 책기준 최신 2.2 버전인 듯)

<details><summary class="point-color-can-hover">[6.4] 예제 펼치기</summary>

```scala
import org.apache.spark.sql.functions.{expr, pow}

// 두 컬럼 모두 수치형 => 곱셈 연산 가능 (+ 덧셈, 뺄셈)
val fabricatedQuantity = pow(col("Quantity") * col("UnitPrice"), 2) + 5
df.select(expr("CustomerId"), fabricatedQuantity.alias("realQuantity")).show(2)
// +----------+------------------+
// |CustomerId|      realQuantity|
// +----------+------------------+
// |   17850.0|239.08999999999997|
// |   17850.0|          418.7156|
// +----------+------------------+

df.selectExpr(
  "CustomerId",
  "(POWER((Quantity * UnitPrice), 2.0) + 5) as realQuantity").show(2)
// +----------+------------------+
// |CustomerId|      realQuantity|
// +----------+------------------+
// |   17850.0|239.08999999999997|
// |   17850.0|          418.7156|
// +----------+------------------+


// 반올림(round) 예제
import org.apache.spark.sql.functions.{round, bround}
df.select(round(col("UnitPrice"), 1).alias("rounded"), col("UnitPrice")).show(5)
// +-------+---------+
// |rounded|UnitPrice|
// +-------+---------+
// |    2.6|     2.55|
// |    3.4|     3.39|
// |    2.8|     2.75|
// |    3.4|     3.39|
// |    3.4|     3.39|
// +-------+---------+

// 내림(bround) 예제
import org.apache.spark.sql.functions.lit
df.select(round(lit("2.5")), bround(lit("2.5"))).show(2)
// +-------------+--------------+
// |round(2.5, 0)|bround(2.5, 0)|
// +-------------+--------------+
// |          3.0|           2.0|
// |          3.0|           2.0|
// +-------------+--------------+


// 피어슨 상관계수 계산 예제
import org.apache.spark.sql.functions.{corr}
df.stat.corr("Quantity", "UnitPrice")
df.select(corr("Quantity", "UnitPrice")).show()
// res52: Double = -0.04112314436835551
// +-------------------------+
// |corr(Quantity, UnitPrice)|
// +-------------------------+
// |     -0.04112314436835551|
// +-------------------------+
```

```sql
-- SQL (동일 표현)
SELECT customerId, (POWER((Quantity * UnitPrice), 2.0) + 5) as realQuantity
FROM dfTable

SELECT round(2.5), bround(2.5)

SELECT corr(Quantity, UnitPrice) FROM dfTable
```


```scala
// 콘솔용 요약 통계 (describe)
df.describe().show()
// +-------+-----------------+------------------+--------------------+------------------+------------------+------------------+--------------+
// |summary|        InvoiceNo|         StockCode|         Description|          Quantity|         UnitPrice|        CustomerID|       Country|
// +-------+-----------------+------------------+--------------------+------------------+------------------+------------------+--------------+
// |  count|             3108|              3108|                3098|              3108|              3108|              1968|          3108|
// |   mean| 536516.684944841|27834.304044117645|                null| 8.627413127413128| 4.151946589446603|15661.388719512195|          null|
// | stddev|72.89447869788873|17407.897548583845|                null|26.371821677029203|15.638659854603892|1854.4496996893627|          null|
// |    min|           536365|             10002| 4 PURPLE FLOCK D...|               -24|               0.0|           12431.0|     Australia|
// |    max|          C536548|              POST|ZINC WILLIE WINKI...|               600|            607.49|           18229.0|United Kingdom|
// +-------+-----------------+------------------+--------------------+------------------+------------------+------------------+--------------+

// '직접 집계'' 필요 시 => 함수 임포트
import org.apache.spark.sql.functions.{count, mean, stddev_pop, min, max}
```

```scala
// StatFunctions package (다양한 통계 함수) 예제
// approxQuantile() : 데이터 백분위수 계산
val colName = "UnitPrice"
val quantileProbs = Array(0.5)
val relError = 0.05
df.stat.approxQuantile("UnitPrice", quantileProbs, relError)
// res61: Array[Double] = Array(2.51)

// 1) crosstab() : 교차표 확인
df.stat.crosstab("StockCode", "Quantity").show()
// +------------------+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
// |StockCode_Quantity| -1|-10|-12| -2|-24| -3| -4| -5| -6| -7|  1| 10|100| 11| 12|120|128| 13| 14|144| 15| 16| 17| 18| 19|192|  2| 20|200| 21|216| 22| 23| 24| 25|252| 27| 28|288|  3| 30| 32| 33| 34| 36|384|  4| 40|432| 47| 48|480|  5| 50| 56|  6| 60|600| 64|  7| 70| 72|  8| 80|  9| 96|
// +------------------+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
// |             22578|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|
// |             21327|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  2|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|
// |             22064|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|
// |             21080|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|
// |             22219|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  3|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|
// |             21908|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|
// |             22818|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|
// |           15056BL|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|
// |             72817|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|
// |             22545|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|
// |             22988|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|
// |             22274|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|
// |             20750|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  2|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|
// |            82616C|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|
// |             21703|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|
// |             22899|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  2|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|
// |             22379|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  2|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|
// |             22422|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  2|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|
// |             22769|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|
// |             22585|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|  1|  0|  0|  1|  0|  0|  0|  0|  0|  0|  0|  0|  0|  0|
// +------------------+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

// 2) freqItems() : 자주 사용하는 항목 쌍 확인
df.stat.freqItems(Seq("StockCode", "Quantity")).show()
// +--------------------+--------------------+
// | StockCode_freqItems|  Quantity_freqItems|
// +--------------------+--------------------+
// |[90214E, 20728, 2...|[200, 128, 23, 32...|
// +--------------------+--------------------+

// 3) monotonically_increasing_id() : 로우에 고유 ID 값 추가
import org.apache.spark.sql.functions.monotonically_increasing_id
df.select(monotonically_increasing_id()).show(2)
// +-----------------------------+
// |monotonically_increasing_id()|
// +-----------------------------+
// |                            0|
// |                            1|
// +-----------------------------+
```

</details>

### 6.5 문자열 데이터 타입 다루기
- 문자열 다루기 = 거의 모든 데이터 처리 과정에서 발생
  - 로그 파일에 정규 표현식을 사용한 데이터 추출, 데이터 치환, 문자열 존재 여부, 대/소문자 변환 처리 등
- 대소문자 변환 작업
  - `initcap()` => 공백으로 구분된 모든 단어의 첫 글자 대문자로 변환
  - `lower()` (전체 소문자로 변환) / `upper()` (전체 대문자로 변환)
- 문자열 주변 공백 제거/추가
  - `lpad()`, `ltrim()`, `rpad()`, `rtrim()`, `trim()`

    <details><summary class="point-color-can-hover">[6.5-1] 예제 펼치기 - 문자열 변환</summary>

    ```scala
    import org.apache.spark.sql.functions.{initcap}
    df.select(initcap(col("Description"))).show(2, false)
    // +----------------------------------+
    // |initcap(Description)              |
    // +----------------------------------+
    // |White Hanging Heart T-light Holder|
    // |White Metal Lantern               |
    // +----------------------------------+


    import org.apache.spark.sql.functions.{lower, upper}
    df.select(col("Description"),
      lower(col("Description")),
      upper(lower(col("Description")))).show(2)
    // +--------------------+--------------------+-------------------------+
    // |         Description|  lower(Description)|upper(lower(Description))|
    // +--------------------+--------------------+-------------------------+
    // |WHITE HANGING HEA...|white hanging hea...|     WHITE HANGING HEA...|
    // | WHITE METAL LANTERN| white metal lantern|      WHITE METAL LANTERN|
    // +--------------------+--------------------+-------------------------+

    import org.apache.spark.sql.functions.{lit, ltrim, rtrim, rpad, lpad, trim}
    df.select(
        ltrim(lit("    HELLO    ")).as("ltrim"),
        rtrim(lit("    HELLO    ")).as("rtrim"),
        trim(lit("    HELLO    ")).as("trim"),
        lpad(lit("HELLO"), 3, " ").as("lp"),
        rpad(lit("HELLO"), 10, " ").as("rp")).show(2)
    // +---------+---------+-----+---+----------+
    // |    ltrim|    rtrim| trim| lp|        rp|
    // +---------+---------+-----+---+----------+
    // |HELLO    |    HELLO|HELLO|HEL|HELLO     |
    // |HELLO    |    HELLO|HELLO|HEL|HELLO     |
    // +---------+---------+-----+---+----------+
    ```

    </details>

- 정규표현식
  - 스파크는 **자바 정규 표현식 문법** 사용
  - `regexp_extract()`, `regexp_replace()` => 값 추출 및 치환
  - `translate(column, from_string, to_string)` 사용한 치환 가능
  - 값 존재 여부 확인 방법?
    - 스칼라 사용 시 `contains()` 사용
    - 파이썬, SQL 사용 시 `instr()` 사용
  - 동적으로 인수의 개수가 변하는 상황에서는
    - 스칼라 고유 기능 `varargs()` 사용
    - 파이썬은 `locate()` (문자열 위치를 정수로 반환. 위치는 1 ~) + 위치 정보 불리언으로 변환

    <details><summary class="point-color-can-hover">[6.5-2] 예제 펼치기 - 문자열 변환</summary>

    ```scala
    import org.apache.spark.sql.functions.regexp_replace
    val simpleColors = Seq("black", "white", "red", "green", "blue")
    val regexString = simpleColors.map(_.toUpperCase).mkString("|")
    // the | signifies `OR` in regular expression syntax
    //
    // df.select(
    //   regexp_replace(col("Description"), regexString, "COLOR").alias("color_clean"),
    //   col("Description")).show(2)
    // regexString: String = BLACK|WHITE|RED|GREEN|BLUE
    // +--------------------+--------------------+
    // |         color_clean|         Description|
    // +--------------------+--------------------+
    // |COLOR HANGING HEA...|WHITE HANGING HEA...|
    // | COLOR METAL LANTERN| WHITE METAL LANTERN|
    // +--------------------+--------------------+

    import org.apache.spark.sql.functions.translate
    (df.select(translate(col("Description"), "LEET", "1337"), col("Description"))
      .show(2))
    // +----------------------------------+--------------------+
    // |translate(Description, LEET, 1337)|         Description|
    // +----------------------------------+--------------------+
    // |              WHI73 HANGING H3A...|WHITE HANGING HEA...|
    // |               WHI73 M37A1 1AN73RN| WHITE METAL LANTERN|
    // +----------------------------------+--------------------+

    import org.apache.spark.sql.functions.regexp_extract
    val regexString = simpleColors.map(_.toUpperCase).mkString("(", "|", ")")
    // the | signifies OR in regular expression syntax
    df.select(
         regexp_extract(col("Description"), regexString, 1).alias("color_clean"),
         col("Description")).show(2)
    // +-----------+--------------------+
    // |color_clean|         Description|
    // +-----------+--------------------+
    // |      WHITE|WHITE HANGING HEA...|
    // |      WHITE| WHITE METAL LANTERN|
    // +-----------+--------------------+

    val containsBlack = col("Description").contains("BLACK")
    val containsWhite = col("DESCRIPTION").contains("WHITE")
    (df.withColumn("hasSimpleColor", containsBlack.or(containsWhite))
      .where("hasSimpleColor")
      .select("Description").show(3, false))
    // +----------------------------------+
    // |Description                       |
    // +----------------------------------+
    // |WHITE HANGING HEART T-LIGHT HOLDER|
    // |WHITE METAL LANTERN               |
    // |RED WOOLLY HOTTIE WHITE HEART.    |
    // +----------------------------------+


    val simpleColors = Seq("black", "white", "red", "green", "blue")
    val selectedColumns = simpleColors.map(color => {
       col("Description").contains(color.toUpperCase).alias(s"is_$color")
    }):+expr("*") // Column 타입이여야 합니다
    (df.select(selectedColumns:_*).where(col("is_white").or(col("is_red")))
      .select("Description").show(3, false))
    // +----------------------------------+
    // |Description                       |
    // +----------------------------------+
    // |WHITE HANGING HEART T-LIGHT HOLDER|
    // |WHITE METAL LANTERN               |
    // |RED WOOLLY HOTTIE WHITE HEART.    |
    // +----------------------------------+
    ```

    </details>


### 6.6 날짜와 타임스탬프 데이터 타입 다루기
- 날짜/시간 사용 시 시간대 (timezone) 와 포맷의 유효성 확인 필요
  - => 스파크는 두 가지 정보만 집중적으로 관리
  - **날짜** (date) & **타임스탬프** (timestamp)
  - inferSchema 옵션 활성화된 경우, 두 정보를 포함해 데이터 타입을 최대한 정확히 식별
  - 스파크는 특정 날짜 포맷 명시 없이도 자체적으로 식별
- 날짜, 시간을 문자열로 저장 <-> 런타임에 날짜 타입으로 변환
  - 텍스트, CSV 파일 다룰 시 많이 발생하는 방식
  - 스파크 2.1 이하) 시간대 미지정 시, 시스템 시간대 기준으로 파싱
    - 시간대 설정? => `spark.conf.sessionLocalTimeZone` 속성을 로컬 시간대로 지정 - [Java TimeZone 포맷](https://bit.ly/2NcW6p2) 따름)
  - 스파크 2.3 이상) `spark.conf.set("spark.sql.session.timeZone", "UTC")` 으로 사용 가능
- TimestampType 클래스는 초 단위 정밀도까지만 지원
  - 밀리세컨드(ms), 마이크로세컨드(μs) 지원 X => 필요 시 Long 데이터타입 사용해서 우회
  - 특이한 포맷의 날짜/시간 데이터를 다뤄야한다면 => 각 단계별 데이터타입과 포맷 정확히 파악 후 트랜스포메이션 적용 해야함
- 스파크는 특정 시점에 데이터 포맷이 특이하게 변할 수 있다
  - 싫다면 파싱이나 변환 작업 필요
  - 스파크는 **자바의 날짜와 타임스탬프** 사용 (표준 체계)
- 자주 사용하는 함수
  - 오늘 기준으로 N일 전후 날짜 구하기
    - `date_sub(컬럼, 뺄 날짜 수)` (책에는 sum, 오타?)
    - `date_add(컬럼, 더할 날짜 수)` 
  - 두 날짜 사이 차이 구하기
    - `datediff(컬럼1, 컬럼2)` : 두 날짜 사이 일 수 반환
    - `months_between(컬럼1, 컬럼2)` : 두 날짜 사이 개월 수 반환

    <details><summary class="point-color-can-hover">[6.6-1] 예제 펼치기 - 날짜 구하기 및 비교</summary>

    ```scala
    // df.printSchema()
    // root
    //  |-- InvoiceNo: string (nullable = true)
    //  |-- StockCode: string (nullable = true)
    //  |-- Description: string (nullable = true)
    //  |-- Quantity: integer (nullable = true)
    //  |-- InvoiceDate: timestamp (nullable = true)
    //  |-- UnitPrice: double (nullable = true)
    //  |-- CustomerID: double (nullable = true)
    //  |-- Country: string (nullable = true)


    // 예제1) 오늘 날짜 / 현재 타임스탬프 값 구하기
    import org.apache.spark.sql.functions.{current_date, current_timestamp}
    val dateDF = (spark.range(10)
      .withColumn("today", current_date())
      .withColumn("now", current_timestamp()))
    dateDF.createOrReplaceTempView("dateTable")

    dateDF.printSchema()
    // root
    //  |-- id: long (nullable = false)
    //  |-- today: date (nullable = false)
    //  |-- now: timestamp (nullable = false)


    // 예제2) 오늘 기준으로 5일 전 날짜 구하기
    // -- SQL : SELECT date_sub(today, 5), date_add(today, 5) FROM dateTable
    import org.apache.spark.sql.functions.{date_add, date_sub}
    dateDF.select(date_sub(col("today"), 5), date_add(col("today"), 5)).show(1)
    // +------------------+------------------+
    // |date_sub(today, 5)|date_add(today, 5)|
    // +------------------+------------------+
    // |        2021-02-01|        2021-02-11|
    // +------------------+------------------+


    // 예제3) 두 날짜 사이 차이 일수(개월수) 구하기
    // -- SQL : SELECT to_date('2016-01-01'), months_between('2016-01-01', '2017-01-01'),
    // datediff('2016-01-01', '2017-01-01')
    // FROM dateTable
    import org.apache.spark.sql.functions.{datediff, months_between, to_date}
    (dateDF.withColumn("week_ago", date_sub(col("today"), 7))
      .select(datediff(col("week_ago"), col("today"))).show(1))
    (dateDF.select(
        to_date(lit("2016-01-01")).alias("start"),
        to_date(lit("2017-05-22")).alias("end"))
      .select(months_between(col("start"), col("end"))).show(1))
    // +-------------------------+
    // |datediff(week_ago, today)|
    // +-------------------------+
    // |                       -7|
    // +-------------------------+

    // +--------------------------+
    // |months_between(start, end)|
    // +--------------------------+
    // |              -16.67741935|
    // +--------------------------+
    ```

    </details>

- 날짜 변환 및 파싱
  - 올바른 포맷과 타입 사용 시 매우 쉬움
  - 날짜나 타임스탬프 타입 사용 or 'yyy-MM-dd' 포맷에 맞는 문자열 지정
  - `to_date()` : 문자열 => 날짜로 변환 (option. 날짜 포맷 지정 가능)
    - 날짜 포맷 : 자바의 [SimpleDateFormat 클래스 지원 포맷 ](https://bit.ly/2Mz21Qc)사용
  - `to_timestamp()` : 날짜 포맷 필수 (미지정시 'yyyy-MM-dd HH:mm:ss' 포맷 default)
- 날짜 파싱 실패 시?
  - => **null 반환** (에러 X)
  - 예상치 못한 포맷의 데이터가 나타날 수 있으므로 디버깅 어려움
  - 문제 회피할 수 있는 방식
    - 1\. 자바 [SimpleDateFormat](https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html) 표준에 맞춰 날짜 포맷 지정
    - 2\. `to_date()`, `to_timestamp()` 사용
  - 암시적 형변환(implicit type casting)은 위험 => 명시적으로 데이터 타입 변환해서 사용할 것

    <details><summary class="point-color-can-hover">[6.6-2] 예제 펼치기 - 날짜 변환 및 파싱</summary>

    ```scala
    // to_date(문자열) : 문자열 -> 날짜 
    import org.apache.spark.sql.functions.{to_date, lit}
    (spark.range(5).withColumn("date", lit("2017-01-01"))
      .select(to_date(col("date"))).show(1))
    // +---------------+
    // |to_date(`date`)|
    // +---------------+
    // |     2017-01-01|
    // +---------------+

    // SimpleDateFormate 클래스 지원 포맷 사용해야
    dateDF.select(to_date(lit("2016-20-12")),to_date(lit("2017-12-11"))).show(1)
    +---------------------+---------------------+
    |to_date('2016-20-12')|to_date('2017-12-11')|
    +---------------------+---------------------+
    |                 null|           2017-12-11|
    +---------------------+---------------------+
    ```

    ```scala
    // to_date(문자열, 날짜 포맷) => 날짜포맷 Option
    // to_timestamp(문자열, 날짜 포맷) => 날짜포맷 필수
    import org.apache.spark.sql.functions.to_date
    val dateFormat = "yyyy-dd-MM"
    val cleanDateDF = spark.range(1).select(
        to_date(lit("2017-12-11"), dateFormat).alias("date"),
        to_date(lit("2017-20-12"), dateFormat).alias("date2"))
    cleanDateDF.createOrReplaceTempView("dateTable2")

    // cleanDateDF.show()
    // +----------+----------+
    // |      date|     date2|
    // +----------+----------+
    // |2017-11-12|2017-12-20|
    // +----------+----------+


    import org.apache.spark.sql.functions.to_timestamp
    cleanDateDF.select(to_timestamp(col("date"), dateFormat)).show()
    // +----------------------------------+
    // |to_timestamp(`date`, 'yyyy-dd-MM')|
    // +----------------------------------+
    // |               2017-11-12 00:00:00|
    // +----------------------------------+
    ```
    ```sql
    -- SQL (같은 표현, to_date(), to_timestamp())
    SELECT to_date(date, 'yyyy-dd-MM'), to_date(date2, 'yyyy-dd-MM'), to_date(date)
    FROM dateTable2

    SELECT to_timestamp(date, 'yyyy-dd-MM'), to_timestamp(date2, 'yyyy-dd-MM')
    FROM dateTable2
    ```

    </details>

  - 날짜 <-> 타임스탬프 변환
    - SQL (easy)
      ```sql
      SELECT cast(to_date("2017-01-01", "yyyy-dd-MM") as timestamp)
      ```
    - 올바른 포맷과 타입의 날짜와 타임스탬프 사용 시에는 매우 쉽게 비교할 수 있다
      ```scala
      // 날짜, 타임스탬프 타입 사용 or "yyyy-MM-dd" 포맷 문자열 사용
      cleanDateDF.filter(col("date2") > lit("2017-12-12")).show()

      // 스파크가 리터럴로 인식하는 문자열 지정해서 비교도 가능
      cleanDateDF.filter(col("date2") > "'2017-12-12'").show()

      // +----------+----------+
      // |      date|     date2|
      // +----------+----------+
      // |2017-11-12|2017-12-20|
      // +----------+----------+
      ```


### 6.7 null 값 다루기
- DataFrame 에서 빈 값은 **NULL** 로 표현하는게 좋다
  - 스파크에서는 null 을 사용해야 최적화 수행 (빈 문자열 X, 대체값 X)
- DataFrame 에서 null 을 다루는 기본 방식 => `.na`
  - DataFrame의 하위 패키지
  - 연산 수행 중 null 값 제어 방식을 명시적으로 지정하는 함수는 => 5.4.15 로우정렬하기 / 6.3 불리언 데이터 타입 다루기 참조
- null 값을 다루는 두가지 방식
  - 1\. 명시적으로 null 값 제거
  - 2\. 전역 or 컬럼 단위로 null 값을 특정 값으로 채우기
  > null 값은 명시적으로 사용 권장.
  > 그러나 null 값을 허용하지 않는 컬럼 선언해도 **강제성 없음**
  >
  > - 즉, notnull 컬럼이여도 null 값이 있을 수 있다
  > - nullable 속성은 스파크 SQL 옵티마이저가 해당 컬럼을 제어하는 동작을 단순하게 도울 뿐

- `coalesce()`
  - 인수의 여러 컬럼 중 null 이 아닌 첫번째 값 반환
  - 모든 컬럼이 null이 아닌 값을 가지면 첫 번째 컬럼 값 반환

    <details><summary class="point-color-can-hover">[6.7-1] 예제 펼치기 - coalesce()</summary>

    ```scala
    import org.apache.spark.sql.functions.coalesce

    // Description 컬럼 값 null 체크
    //  1. null이면 CustomerId 값 반환
    //  2. null이 아니면 Description 컬럼 값 반환
    df.select(coalesce(col("Description"), col("CustomerId"))).show()
    ```

    </details>

- SQL 함수
  - `ifnull()` : 첫 번째 값이 null이면 두 번째 값 반환, null이 아니면 첫 번째 값 반환
  - `nullif()` : 두 값이 같으면 null 반환, 다르면 첫 번째 값 반환
  - `nvl()` : 첫 번째 값이 null이면 두 번째 값 반환, null이 아니면 첫 번째 값 반환
  - `nvl2()` : 첫 번째 값이 null이 아니면 두 번째 값 반환, null이면 세 번째 값 반환
  ```kotlin
  // 이해하는 용도.. like this
  fun ifnull(first: Any, default: Any) = if (first != null) first else default
  fun nullif(first: Any, second: Any) = if (first != second) first else null
  fun nvl(first: Any, default: Any) = if (first != null) first else default
  fun nvl2(first: Any, notnull_return: Any, null_return: Any) = if (first != null) notnull_return else null_return
  ```

    <details><summary class="point-color-can-hover">[6.7-2] 예제 펼치기 - SQL 함수</summary>

    ```sql
    SELECT
      ifnull(null, 'return_value'),
      nullif('value', 'value'),
      nvl(null, 'return_value'),
      nvl2('not_null', 'return_value', "else_value")
    FROM dfTable LIMIT 1
    
    -- +------------+----+------------+------------+
    -- |           a|   b|           c|           d|
    -- +------------+----+------------+------------+
    -- |return_value|null|return_value|return_value|
    -- +------------+----+------------+------------+
    ```

    </details>


- `drop()`
  - null 값을 가진 로우를 모든 로우 제거
  - 인수
    - `any` (하나라도 null이면 제거) / `all` (모든 컬럼이 null 또는 NaN이면 제거)
    - 배열 형태 컬럼도 인수로 전달 가능
  - SQL 사용 시 컬럼별로 수행해야함

    <details><summary class="point-color-can-hover">[6.7-3] 예제 펼치기 - drop()</summary>

    ```scala
    df.na.drop()
    df.na.drop("any") // 하나라도 컬럼이 null (또는 NaN) 이면 로우 제거
    df.na.drop("all") // 모든 컬럼이 null (또는 NaN) 이면 로우 제거

    df.na.drop("all", Seq("StockCode", "InvoiceNo")) // 컬럼(배열형태) 전달 가능
    ```
    ```sql
    -- SQL 사용 시 컬럼 별 수행해야
    SELECT * FROM dfTable WHERE Description IS NOT NULL
    ```

    </details>

- `fill()`
  - 하나 이상의 컬럼을 특정 값으로 채움
  - 인수
    - 채워넣을 값, 컬럽 집합으로 구성된 맵
    - 컬럼명 배열로 인수 사용 및 다수 컬럼에도 적용 가능 (ex. `df.na.fill(5:Integer)`, `df.na.fill(5:Double)`)
  - 스칼라 Map 타입 사용도 인수로 가능 (key:value = 컬럼명:null값을 채울 값)

    <details><summary class="point-color-can-hover">[6.7-4] 예제 펼치기 - fill()</summary>

    ```scala
    df.na.fill("All Null values become this string")

    // 다수 컬럼에 적용
    df.na.fill(5, Seq("StockCode", "InvoiceNo"))

    // Map 타입으로 다수 컬럼에 적용
    val fillColValues = Map("StockCode" -> 5, "Description" -> "No Value")
    df.na.fill(fillColValues)
    ```

    </details>

- `relace()`
  - 조건에 따라 다른 값으로 대체
  - 단, 변경하고자하는 값 == 원래 값 데이터 타입 같아야

    <details><summary class="point-color-can-hover">[6.7-5] 예제 펼치기 - replace()</summary>

    ```scala
    df.na.replace("Description", Map("" -> "UNKNOWN"))
    ```

    </details>

### 6.8 정렬하기
- `asc_nulls_first()`, `desc_nulls_first()`, `asc_nulls_last()`, `desc_nulls_last()`
  - DataFrame 정렬 시 null 값 표시 기준 지정 가능
- (=> [5.4.15 - 로우정렬하기](https://minsw.github.io/2021/01/26/Spark-The-Definitive-Guide-5%EC%9E%A5/#15-%EB%A1%9C%EC%9A%B0-%EC%A0%95%EB%A0%AC%ED%95%98%EA%B8%B0) 다시 참고~)

### 6.9 복합 데이터 타입 다루기
- 복합 데이터 타입 : 구조체 (struct), 배열 (array), 맵(map)

- 구조체 = DataFrame 내부의 DataFrame
  - 쿼리문에서 다수의 컬럼을 괄호로 묶어서 => 구조체 만듦
  - 복합 데이터 타입을 가진 DataFrame 사용
    - => 다른 DataFrame 조회하는것과 동일하게 사용 가능
    - 차이점은 문법에 점 (.) 사용 or `getField()` 사용
    - `*` 문자로 모든 값 조회 가능 (모든 컬럼을 최상위 수준으로 끌어올리기 가능?)

  <details><summary class="point-color-can-hover">[6.8,1] 예제 펼치기 - 구조체</summary>

  ```scala
  df.selectExpr("(Description, InvoiceNo) as complex", "*")
  df.selectExpr("struct(Description, InvoiceNo) as complex", "*")

  import org.apache.spark.sql.functions.struct
  val complexDF = df.select(struct("Description", "InvoiceNo").alias("complex"))
  complexDF.createOrReplaceTempView("complexDF")

  // 조회 시 점(.) 또는 getField() 사용
  complexDF.select("complex.Description")
  complexDF.select(col("complex").getField("Description"))

  // * 로 모든 값 조회 가능
  complexDF.select("complex.*")
  ```
  ```sql
  SELECT complex.* FROM complexDF
  ```

  </details>

- 배열
  - example) 해당하는 컬럼의 모든 단어를 하나의 로우로 변환
  - `split(target, delimiter)` : 구분자 기준으로 나누어 배열로 변환
    - 복합 데이터 타입을 또 다른 컬럼처럼 다룰 수 있는 기능
  - 배열의 길이 : 배열 size 조회해서 길이 알 수 있음
  - `array_contains()` : 배열에 특정 값 존재하는지 확인 가능
    - 하지만 시나리오 완성은 불가능
  - `explode(배열타입 칼럼)` : 인수의 컬럼 배열 갑셍 포함된 모든 값을 로우로 변환 (나머지 컬럼 값은 중복되어 표시)

  <details><summary class="point-color-can-hover">[6.8.2] 예제 펼치기 - 배열</summary>

  ```scala
  // split() : 배열로 변환
  import org.apache.spark.sql.functions.split
  df.select(split(col("Description"), " ")).show(2)
  // +---------------------+
  // |split(Description,  )|
  // +---------------------+
  // | [WHITE, HANGING, ...|
  // | [WHITE, METAL, LA...|
  // +---------------------+

  (df.select(split(col("Description"), " ").alias("array_col"))
    .selectExpr("array_col[0]").show(2))
  // +------------+
  // |array_col[0]|
  // +------------+
  // |       WHITE|
  // |       WHITE|
  // +------------+


  // size() : 배열의 길이
  import org.apache.spark.sql.functions.size
  df.select(size(split(col("Description"), " "))).show(2) // shows 5 and 3
  // +---------------------------+
  // |size(split(Description,  ))|
  // +---------------------------+
  // |                          5|
  // |                          3|
  // +---------------------------+


  // array_contains() : 배열에 특정값 존재하는지 확인
  import org.apache.spark.sql.functions.array_contains
  df.select(array_contains(split(col("Description"), " "), "WHITE")).show(2)
  // +--------------------------------------------+
  // |array_contains(split(Description,  ), WHITE)|
  // +--------------------------------------------+
  // |                                        true|
  // |                                        true|
  // +--------------------------------------------+


  // explode() : 입력된 컬럼의 배열값(split(Description) 결과물) 에 포함된 모든 값을 로우로 변환 (나머지값(InvoiceNo)은 중복)
  import org.apache.spark.sql.functions.{split, explode}
  (df.withColumn("splitted", split(col("Description"), " "))
    .withColumn("exploded", explode(col("splitted")))
    .select("Description", "InvoiceNo", "exploded").show(2))
  // +--------------------+---------+--------+
  // |         Description|InvoiceNo|exploded|
  // +--------------------+---------+--------+
  // |WHITE HANGING HEA...|   536365|   WHITE|
  // |WHITE HANGING HEA...|   536365| HANGING|
  // +--------------------+---------+--------+
  ```

  ```sql
  -- SQL

  -- split()
  SELECT split(Description, ' ') FROM dfTable

  SELECT split(Description, ' ')[0] FROM dfTable

  -- array_contains()
  SELECT array_contains(split(Description, ' '), 'WHITE') FROM dfTable

  -- explode()
  SELECT Description, InvoiceNo, exploded
  FROM (SELECT *, split(Description, " ") as splitted FROM dfTable)
  LATERAL VIEW explode(splitted) as exploded
  ```

  </details>

- 맵
  - `map()` + 키-값 쌍
  - 적합한 키로 데이터 조회 가능, 없을 시 null 반환
  - map 타입 분해 -> 컬럼 변환 가능

  <details><summary class="point-color-can-hover">[6.8.3] 예제 펼치기 - 맵</summary>

  ```scala
  import org.apache.spark.sql.functions.map
  df.select(map(col("Description"), col("InvoiceNo")).alias("complex_map")).show(2)
  // +--------------------+
  // |         complex_map|
  // +--------------------+
  // |[WHITE HANGING HE...|
  // |[WHITE METAL LANT...|
  // +--------------------+

  // 키로 데이터 조회 (없을 시 null 반환)
  (df.select(map(col("Description"), col("InvoiceNo")).alias("complex_map"))
    .selectExpr("complex_map['WHITE METAL LANTERN']").show(2))
  // +--------------------------------+
  // |complex_map[WHITE METAL LANTERN]|
  // +--------------------------------+
  // |                            null|
  // |                          536365|
  // +--------------------------------+

  // map 타입 분해 -> 컬럼으로 변환 가능
  (df.select(map(col("Description"), col("InvoiceNo")).alias("complex_map"))
    .selectExpr("explode(complex_map)").show(2))
  // +--------------------+------+
  // |                 key| value|
  // +--------------------+------+
  // |WHITE HANGING HEA...|536365|
  // | WHITE METAL LANTERN|536365|
  // +--------------------+------+
  ```

  ```sql
  -- SQL
  SELECT map(Description, InvoiceNo) as complex_map FROM dfTable
  WHERE Description IS NOT NULL
  ```

  </details>

### 6.10 JSON 다루기
- 스파크에서 JSON 데이터 다루기 위한 고유 기능 제공
  - 문자열 형태 JSON 조작, JSON 파싱, JSON 객체로 변환 등
- `get_json_object()`
  - JSON 객체 (딕셔너리, 배열) 인라인 쿼리로 조회 가능
  - 중첩 없는 단일 JSON일 시, `json_tuble` 사용 가능
- `to_json()` : StructType -> JSON 문자열. 데이터 소스와 동일한 형태의 딕셔너리(맵) 인자로 사용 가능
- `from_json()` : JSON 문자열 -> 객체. 단 스키마 지정 필수 (option. 맵 데이터 타입 옵션)

<details><summary class="point-color-can-hover">[6.10] 예제 펼치기</summary>

```scala
// JSON 컬럼 생성
val jsonDF = spark.range(1).selectExpr("""
  '{"myJSONKey" : {"myJSONValue" : [1, 2, 3]}}' as jsonString""")

// get_json_object() 로 JSON 객체 조회
import org.apache.spark.sql.functions.{get_json_object, json_tuple}
jsonDF.select(
    get_json_object(col("jsonString"), "$.myJSONKey.myJSONValue[1]") as "column",
    json_tuple(col("jsonString"), "myJSONKey")).show(2)

// => SQL 사용한 처리 (동일 표현)
jsonDF.selectExpr(
  "get_json_object(jsonString, '$.myJSONKey.myJSONValue[1]') as column", 
  "json_tuple(jsonString, 'myJSONKey')").show(2)
// +------+--------------------+
// |column|                  c0|
// +------+--------------------+
// |     2|{"myJSONValue":[1...|
// +------+--------------------+


// to_json() : StructType -> JSON 문자열
import org.apache.spark.sql.functions.to_json
(df.selectExpr("(InvoiceNo, Description) as myStruct")
  .select(to_json(col("myStruct"))))


// from_json() : JSON 문자열 -> 객체
import org.apache.spark.sql.functions.from_json
import org.apache.spark.sql.types._
val parseSchema = new StructType(Array(
  new StructField("InvoiceNo",StringType,true),
  new StructField("Description",StringType,true)))
(df.selectExpr("(InvoiceNo, Description) as myStruct")
  .select(to_json(col("myStruct")).alias("newJSON"))
  .select(from_json(col("newJSON"), parseSchema), col("newJSON")).show(2))
// +----------------------+--------------------+
// |jsontostructs(newJSON)|             newJSON|
// +----------------------+--------------------+
// |  [536365, WHITE HA...|{"InvoiceNo":"536...|
// |  [536365, WHITE ME...|{"InvoiceNo":"536...|
// +----------------------+--------------------+
```

</details>

### 6.11 사용자 정의 함수
- **사용자 정의 함수** (user defined function, **UDF**)
  - 스파크의 가장 강력한 기능 중 하나
  - 파이썬, 스칼라, 외부 라이브러리등 사용 => 사용자가 원하는 형태로 트랜스포메이션 생성
- 특징
  - 하나 이상의 컬럼을 입력/반환 가능
  - 스파크 UDF는 다양한 언어로 개발 가능
  - 레코드 별로 데이터를 처리하는 함수이므로, 독특하거나 도메인 특화 (DSL) 언어 사용 X
  - => 기본적으로 특정 SparkSession이나 Context에서 사용할 수 있게 <u>임시 함수 형태로 등록</u>
- 다양한 언어로 UDF 개발 가능
  - 그러나 언어별로 성능에 영향 존재
    - 예제 참고
    - 함수를 만들고 모든 워커 노드에서 해당 함수를 사용할 수 있도록 스파크에 등록
      - 스파크는 드라이버에서 함수 직렬화 -> 네트워크 통해서 모든 익스큐터 프로세스로 전달
      - (언어에 관계없이 발생하는 과정)
    - 함수를 개발한 언어에 따라 기본적으로 동작하는 방식이 달라짐
      - 애초에 스칼라, 자바 사용 시 JVM 환경에서만 사용 가능
        - 스파크 내장함수 장점 사용 X => 성능 ↓
        - 많은 객체 생성시에도 성능 문제
      - 파이썬 사용 시 모든 데이터를 직렬화하고, 파이썬 프로세스에 있는 데이터의 로우마다 함수 실행 및 JVM과 스파크에 처리 결과를 반환
        - 일단 직렬화 과정에서 큰 부하 발생
        - 데이터가 파이썬으로 전달되면 스파크에서 워커 메모리 관리의 어려움
    - => 따라서 사용자 정의 함수는 **자바나 스칼라로 작성** 을 권장
- 기본은 사용자 정의 함수(UTF)는 DataFrame에서만 사용 가능 (문자열 표현식 X)
  - **스파크 SQL 함수 등록하면?**
  - => 모든 프로그래밍 언어와 SQL 에서 사용자 정의 함수 사용 가능
    - 파이썬에서도 우회적으로 사용 가능하지만 DataFrame 함수 대신 SQL 표현식으로 사용해야함
    - 스파크는 자체 데이터 타입(파이썬X)을 사용하므로 **변환 타입 지정 권!장!**
    - 반환될 타입과 다른 데이터 타입 지정시 => null 반환
- 사용자 정의 함수에서 선택적 값 반환
  - 파이썬 = `None` / 스칼라 = `Option` 반환

  <details><summary class="point-color-can-hover">[6.11] 예제 펼치기</summary>

  ```scala
  val udfExampleDF = spark.range(5).toDF("num")
  def power3(number:Double):Double = number * number * number
  power3(2.0) // 8.0


  // UDF 실행 + 함수 등록 및 사용 (=> DataFrame에서 사용 가능)
  import org.apache.spark.sql.functions.udf
  val power3udf = udf(power3(_:Double):Double)

  udfExampleDF.select(power3udf(col("num"))).show()
  // +--------+
  // |UDF(num)|
  // +--------+
  // |     0.0|
  // |     1.0|
  // |     8.0|
  // |    27.0|
  // |    64.0|
  // +--------+


  // UDF를 스파크 SQL로 등록하면 => 모든 프로그래밍 언어, SQL 에서 사용 가능 (+문자열 표현식)
  spark.udf.register("power3", power3(_:Double):Double)
  udfExampleDF.selectExpr("power3(num)").show(2)
  +-------------------------------+
  |UDF:power3(cast(num as double))|
  +-------------------------------+
  |                            0.0|
  |                            1.0|
  +-------------------------------+
  ```

  ```python
  %spark.pyspark
  udfExampleDF.selectExpr("power3(num)").show(2)
  # => Scala로 등록된 UDF 사용

  # 반환 데이터타입이 Integer인데 DoubeType() 으로 변환 시 => null 반환
  from pyspark.sql.types import IntegerType, DoubleType
  spark.udf.register("power3py", power3, DoubleType())
  ```

  ```sql
  -- SQL 에서도 등록된 UDF 사용 가능
  SELECT power3(12), power3py(12) -- 반환 데이터 타입 문제로 동작하지 않음
  ```

  </details>


### 6.12 Hive UDF
- 하이브 문법을 사용해서 만든 함수 => **UDF**, **UDAF** 사용 가능
  - UDF (User Defined Function)
  - UDAF (User Defined Aggregate Function)
- 단, 하이브 지원 기능 활성화 필요
  - => `SparkSession.builder().enableHiveSupport()` 명시
  - 하이브 지원 활성화 되면 SQL로 UDF 등록 가능
  - 사전에 컴파일된 스칼라, 자바 패키지만 지원 (라이브러리 의존성 명시 필요)

  ```sql
  -- TEMPORARY 키워드 제거 시 => 하이브 메타스토어에 영구(permanent) 함수로 등록
  CREATE TEMPORARY FUNCTION myFunc AS 'com.organization.hive.udf.FunctionName'
  ```


### 6.13 정리
- 스파크 SQL을 사용목적에 맞게 확장하는 방식
  - 간단한 함수만으로도 확장 가능 (DSL X)
- 스파크 SQL은 복잡한 비즈니스 로직 구현에 사용할 수 있는 강력한 기능

### 📒 단어장