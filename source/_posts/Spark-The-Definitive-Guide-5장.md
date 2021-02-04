---
title: '&#039;Spark The Definitive Guide&#039; 5장 - 구조적 API 기본 연산'
date: 2021-01-26 05:26:48
categories: spark
tags:
  - spark
  - apache
  - book
  - study
---

<br/>

오늘의 교훈.
도커 이미지에 예제 있다고 신나게 돌리고~ 돌리고~ 하다보면
터진다는걸 명심하도록 하자 🥺

```
java.io.IOException: No space left on device
```

<br/>

<img src="https://user-images.githubusercontent.com/26691216/106086744-1b72d100-6166-11eb-8d99-1deebfa68867.jpg" width="400" alt="jongman">
<center><i style="color:lightgray"> 인생은 실전이야 친구야</i></center>

<img width="300" alt="bomb" src="https://user-images.githubusercontent.com/26691216/106087261-fcc10a00-6166-11eb-80c2-d339e59bbc99.png">



<center><h2>_ _ _</h2></center>

<br/>

---

# CHAPTER 5 구조적 API 기본 연산
CHAPTER 4 는 구조적 API의 핵심 추상화 '개념'을 소개
CHAPTER 5 는 DataFrame과 그 데이터를 다루는 기본 '기능' 소개


> *'DataFrame = Row 타입의 **레코드** + 여러 **컬럼**'*
> (각 컬럼명과 데이터 타입은 **스키마**로 정의)
>
> DataFrame의 **파티셔닝** :  DataFrame (또는 Dataset)이 클러스터에서 물리적으로 배치되는 형태를 정의
>   - **파티셔닝 스키마** : 파티션을 배치하는 방법 정의
>   - 파티셔닝의 분할 기준? => 특정 컬럼 or 비결정론적(nondeterministically) 값 기반으로 설정

### 5.1 스키마

```scala
val df = spark.read.format("json").load("/data/flight-data/json/2015-summary.json")
df.printSchema()

// root
//  |-- DEST_COUNTRY_NAME: string (nullable = true)
//  |-- ORIGIN_COUNTRY_NAME: string (nullable = true)
//  |-- count: long (nullable = true)
```

- 스키마는 **DataFrame의 컬럼명과 데이터 타입을 정의**
  - 관련된 모든 것을 하나로 묶는 역할
- 데이터 소스에서 스키마를 얻거나 (Schema-on-read), 직접 정의 가능
  - 대부분의 비정형 분석 (ad-hoc analysis)에서 schema-on-read 잘 동작
  - 운영 환경 ETL 작업에 스파크 사용시 **직접 정의 필요** (샘플 데이터 타입에 따른 스키마 추론 방지)
- 스키마는 `StructType` 객체 
  - 복합 데이터 타입 `StructType` (=consistOf(`StructField` 객체))
  - 스파크는 자체 데이터 타입 정보를 사용 => 언어 별 데이터 타입으로 설정 X

    ```scala
    spark.read.format("json").load("/data/flight-data/json/2015-summary.json").schema
    // res176: org.apache.spark.sql.types.StructType
    //  = StructType(
    //        StructField(DEST_COUNTRY_NAME,StringType,true),
    //        StructField(ORIGIN_COUNTRY_NAME,StringType,true),
    //        StructField(count,LongType,true)
    //      )
    ```


<details><summary class="point-color-can-hover">[5.1] 예제 펼치기 - DataFrame에 스키마 적용 예제</summary>

```scala
// DataFrame에 스키마를 만들고 적용하는 예제
import org.apache.spark.sql.types.{StructField, StructType, StringType, LongType}
import org.apache.spark.sql.types.Metadata

val myManualSchema = StructType(Array(
  StructField("DEST_COUNTRY_NAME", StringType, true),
  StructField("ORIGIN_COUNTRY_NAME", StringType, true),
  StructField("count", LongType, false,
    Metadata.fromJson("{\"hello\":\"world\"}"))
))

val df = (spark.read.format("json").schema(myManualSchema)
  .load("/data/flight-data/json/2015-summary.json"))
```

</details>


### 5.2 컬럼과 표현식
- 스파크의 '컬럼' (=표현식)
  - 스프레드 시트, R의 dataframe, Pandas의 컬럼과 비슷
  - 사용자는 <u>**표현식**</u>으로 DataFrame의 컬럼을 선택, 조작, 제거 가능
  - 즉 표현식을 사용해 레코드 단위로 계산한 값을 나타내는 논리적 구조. 실제값을 얻으려면 로우 (=> DataFrame) 가 필요
  - 외부 접근시 **반드시 DataFrame 을 통해야 함**
- 컬럼 생성 & 참조 : `col()` `column()`
  - 컬럼이 DataFrame에 있는지 없는지는 모름 => **분석기**가 **카탈로그**에 저장된 정보랑 비교하기 전까지는 미확인 [[4.4] 참고](https://minsw.github.io/2021/01/26/Spark-The-Definitive-Guide-4%EC%9E%A5/#4-4-%EA%B5%AC%EC%A1%B0%EC%A0%81-API%EC%9D%98-%EC%8B%A4%ED%96%89-%EA%B3%BC%EC%A0%95)
  - 스칼라는 고유 기능 사용 가능 : `$"컬럼명"` `'컬럼명` (`'` : 틱 마크, 심벌)
  - 명시적 참조 : `DataFrame.col()` (조인시 유용)
    => 명시적 컬럼 정의 시, 분석기 실행 단계에서 컬럼 확인 절차 생략

  ```scala
  import org.apache.spark.sql.functions.{col, column}
  col("someCol")
  column("someCol")

  // in Scala
  $"someCol"
  'someCol

  // 명시적 참조
  df.col("someCol")
  ```

- **표현식** : `expr()`
  - DataFrame 레코드의 여러 값에 대한 트랜스포메이션 집합
  - 여러 컬럼을 입력받아 식별 -> 다양한 표현식을 각 레코드에 적용 -> **'단일값'** (복합 데이터 타입) 으로 출력 하는 함수
  - *DataFrame의 컬럼은 '표현식'이다*
    - `expr("someCol")` == `col("someCol")` (동일 동작)
    - 컬럼은 표현식의 일부 기능 제공
- 스파크는 연산 순서를 지정하는 논리적 트리로 컴파일
  - DataFrame 코드나 SQL 표현식 작성 시, 실행 시점에 동일한 논리 트리로 컴파일 되므로 동일한 성능 발휘
  - 예시는 `p.129` [그림 5-1] 논리적 트리 DAG 참고
  - `expr("someCol - 5")` == `col("someCol") - 5` == `expr("someCol") - 5`  (다 같은 트랜스포메이션 과정)

  ```scala
  // 동일한 표현 - col(), expr()
  import org.apache.spark.sql.functions.expr
  expr("(((someCol + 5) * 200) - 6) < otherCol")

  (((col("someCol") + 5) * 200) - 6) < col("otherCol")
  ```

  ```scala
  // DataFrame 컬럼 접근 (printScheme() 아닌 프로그래밍 방식)
  spark.read.format("json").load("/data/flight-data/json/2015-summary.json").columns
  // res0: Array[String] = Array(DEST_COUNTRY_NAME, ORIGIN_COUNTRY_NAME, count)
  ```


> '표현식' 과 '컬럼' 사이 핵심 내용
> - 컬럼은 단지 표현식일 뿐
> - 컬럼과 컬럼의 트랜스포메이션은 파싱된 표현식과 동일한 논리적 실행 계획으로 컴파일


### 5.3 레코드와 로우
- 스파크의 '로우' (=레코드)
  - 스파크에서 DataFrame의 각 로우는 하나의 레코드
  - 값을 생성하기 위해 컬럼 표현식으로 Row 객체를 다룸
  - Row 객체는 내부 바이트 배열을 가지는 인터페이스 => **오직 컬럼 표현식으로만** 다룰 수 있음 (외부 노출 X)
    ```scala
    // Row 확인
    scala> df.first()
    // res1: org.apache.spark.sql.Row = [United States,Romania,15]
    ```

- 로우 생성
  - 각 컬럼에 해당하는 값으로 직접 Row 객체 생성 가능
  - 그러나 Row 객체는 스키마 정보 X (=> 오직 DataFrame만 가짐)
  - => 스키마랑 같은 순서로 값 명시해야함
- 로우 데이터 접근하려면 => 원하는 위치 지정
  - Python, R 은 올바른 데이터 타입으로 알아서 변환됨
  - Scala, Java 는 헬퍼 메서드나 데이터타입 명시적 지정 필요 (Dataset API 사용 시 jvm 객체 데이터 셋 얻을 수 있음)
    ```scala
    import org.apache.spark.sql.Row
    val myRow = Row("Hello", null, 1, false)

    myRow(0) // type Any
    myRow(0).asInstanceOf[String] // String
    myRow.getString(0) // String
    myRow.getInt(2) // Int
    ```

    ```python
    # python 사용 시
    myRow[0]
    myRow[2]
    ```


### 5.4 DataFrame의 트랜스포메이션
> DataFrame을 다루는 방법 (주요 작업 4가지)
>  - 로우나 컬럼 추가
>  - 로우나 컬럼 제거
>  - 로우를 컬럼으로 변환하거나, 그 반대로 변환
>  - 컬럼값을 기준으로 로우 순서 변경
>
> 모든 유형의 작업은 트랜스포메이션으로 변환 가능 (ex. 모든 로우의 특정 컬럼값 변경 후 결과 반환)

- [(1) DataFrame 생성](#1-DataFrame-생성)
- [(2) select 와 selectExpr](#2-select-와-selectExpr)
- [(3) 스파크 데이터 타입으로 변환하기](#3-스파크-데이터-타입으로-변환하기)
- [(4) 컬럼 추가하기](#4-컬럼-추가하기)
- [(5) 컬럼명 변경하기](#5-컬럼명-변경하기)
- [(6) 예약 문자와 키워드](#6-예약-문자와-키워드)
- [(7) 대소문자 구분](#7-대소문자-구분)
- [(8) 컬럼 제거하기](#8-컬럼-제거하기)
- [(9) 컬럼의 데이터 타입 변경하기](#9-컬럼의-데이터-타입-변경하기)
- [(10) 로우 필터링하기](#10-로우-필터링하기)
- [(11) 고유한 로우 얻기](#11-고유한-로우-얻기)
- [(12) 무작위 샘플 만들기](#12-무작위-샘플-만들기)
- [(13) 임의 분할하기](#13-임의-분할하기)
- [(14) 로우 합치기와 추가하기](#14-로우-합치기와-추가하기)
- [(15) 로우 정렬하기](#15-로우-정렬하기)
- [(16) 로우 수 제한하기](#16-로우-수-제한하기)
- [(17) repartition과 coalesce](#17-repartition과-coalesce)
- [(18) 드라이버로 로우 데이터 수집하기](#18-드라이버로-로우-데이터-수집하기)


<br/>

#### (1) DataFrame 생성

<details><summary class="point-color-can-hover">[5.4-1] 예제 펼치기 </summary>

```scala

// A. 원시 데이터 소스 -> DataFrame
val df = (spark.read.format("json").load("/data/flight-data/json/2015-summary.json"))
// (임시 뷰 등록)
df.createOrReplaceTempView("dfTable")
``

// B. (Row 객체를 가지는) Seq 타입 -> DataFrame
import org.apache.spark.sql.Row
import org.apache.spark.sql.types.{StructField, StructType, StringType, LongType}

val myManualSchema = new StructType(Array(
  new StructField("some", StringType, true),
  new StructField("col", StringType, true),
  new StructField("names", LongType, false)))
val myRows = Seq(Row("Hello", null, 1L))
// sc 객체의 parallelize() 로 RDD 생성
val myRDD = spark.sparkContext.parallelize(myRows)
// createDataFrame()로 DataFrame 생성
val myDf = spark.createDataFrame(myRDD, myManualSchema)
myDf.show()

// +-----+----+-----+
// | some| col|names|
// +-----+----+-----+
// |Hello|null|    1|
// +-----+----+-----+


// Scala 사용 시 toDF() 사용 가능 (import spark.implicits._)
val myDF = Seq(("Hello", 2, 1L)).toDF("col1", "col2", "col3")
```
</details>

- 스파크의 implicits (import 필요, [참고](http://bit.ly/2xrFpML))
  - Scala 스파크 콘솔 사용 시 => Seq 데이터 타입에 `toDF()` 사용 가능
  - 그러나 null 타입과는 안맞으므로 운영환경 사용은 권장 X


> #### createDataFrame() vs toDF()
> - `createDataFrame(rowRDD: RDD[Row], schema: StructType) : DataFrame`
>   - 모든 schema customization 가능
>   - [API docs#SparkSession](https://spark.apache.org/docs/latest/api/scala/org/apache/spark/sql/SparkSession.html#createDataFrame(rowRDD:org.apache.spark.rdd.RDD[org.apache.spark.sql.Row],schema:org.apache.spark.sql.types.StructType):org.apache.spark.sql.DataFrame)
> - `toDF()`
>   - 스키마 지정 없음. schema 추론 (Dataset API)
>   - `import spark.implicits._` 필요
>   - [API docs#Dataset](https://spark.apache.org/docs/latest/api/scala/org/apache/spark/sql/Dataset.html#toDF():org.apache.spark.sql.DataFrame)

#### (2) select 와 selectExpr

<details><summary class="point-color-can-hover">[5.4-2] 예제 펼치기 </summary>

```sql
--- SQL
SELECT * FROM dataFrameTable
SELECT columnName FROM dataFrameTable
SELECT columnName * 10, otherColumn, someOtherCol as c FROM dataFrameTable
```

```bash
# select() == SELECT query
scala> df.select("DEST_COUNTRY_NAME").show(2)
+-----------------+
|DEST_COUNTRY_NAME|
+-----------------+
|    United States|
|    United States|
+-----------------+

scala> df.select("DEST_COUNTRY_NAME", "ORIGIN_COUNTRY_NAME").show(2)
+-----------------+-------------------+
|DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|
+-----------------+-------------------+
|    United States|            Romania|
|    United States|            Croatia|
+-----------------+-------------------+

--- SQL
SELECT DEST_COUNTRY_NAME FROM dfTable LIMIT 2
SELECT DEST_COUNTRY_NAME, ORIGIN_COUNTRY_NAME FROM dfTable LIMIT 2


# 다양한 컬럼 참조 방법
# df.select(col("DEST_COUNTRY_NAME"), "DEST_COUNTRY_NAME") => 에러
scala> import org.apache.spark.sql.functions.{expr, col, column}
scala> (df.select(
     |     df.col("DEST_COUNTRY_NAME"),
     |     col("DEST_COUNTRY_NAME"),
     |     column("DEST_COUNTRY_NAME"),
     |     'DEST_COUNTRY_NAME,
     |     $"DEST_COUNTRY_NAME",
     |     expr("DEST_COUNTRY_NAME"))
     |   .show(2))
+-----------------+-----------------+-----------------+-----------------+-----------------+-----------------+
|DEST_COUNTRY_NAME|DEST_COUNTRY_NAME|DEST_COUNTRY_NAME|DEST_COUNTRY_NAME|DEST_COUNTRY_NAME|DEST_COUNTRY_NAME|
+-----------------+-----------------+-----------------+-----------------+-----------------+-----------------+
|    United States|    United States|    United States|    United States|    United States|    United States|
|    United States|    United States|    United States|    United States|    United States|    United States|
+-----------------+-----------------+-----------------+-----------------+-----------------+-----------------+

```

```scala
// expr() 예시 - 컬럼명 DEST_COUNTRY_NAME -> destination -> DEST_COUNTRY_NAME
df.select(expr("DEST_COUNTRY_NAME AS destination")).show(2)
df.select(expr("DEST_COUNTRY_NAME as destination").alias("DEST_COUNTRY_NAME")).show(2)


// select() + expr() => selectExpr()
df.selectExpr("DEST_COUNTRY_NAME as newColumnName", "DEST_COUNTRY_NAME").show(2)
// +-------------+-----------------+
// |newColumnName|DEST_COUNTRY_NAME|
// +-------------+-----------------+
// |United States|    United States|
// |United States|    United States|
// +-------------+-----------------+

(df.selectExpr(
    "*", // include all original columns
    "(DEST_COUNTRY_NAME = ORIGIN_COUNTRY_NAME) as withinCountry")
  .show(2))
// +-----------------+-------------------+-----+-------------+
// |DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|withinCountry|
// +-----------------+-------------------+-----+-------------+
// |    United States|            Romania|   15|        false|
// |    United States|            Croatia|    1|        false|
// +-----------------+-------------------+-----+-------------+

df.selectExpr("avg(count)", "count(distinct(DEST_COUNTRY_NAME))").show(2)
// +-----------+---------------------------------+
// | avg(count)|count(DISTINCT DEST_COUNTRY_NAME)|
// +-----------+---------------------------------+
// |1770.765625|                              132|
// +-----------+---------------------------------+
```
</details>


> DataFrame을 다루기 위한 대부분의 트랜스포메이션 작업 해결 가능
>
> - `select()` : 컬럼이나 표현식을 사용
> - `selectExpr()` : 문자열 표현식을 사용
> - `select()`: 메서드로 사용할 수 없는 `org.apache.spark.sql.function` 에 포함된 다양한 함수

- DataFrame 컬럼 다룰 시, SQL 사용 가능
- 컬럼 참조 방법은 다양한 방법을 섞어서 사용할 수 있다. 5.2 재참고
  - Column 객체랑 문자열을 함께 섞어쓸수는 X (ex. `df.select(col("DEST_COUNTRY_NAME"), "DEST_COUNTRY_NAME")` => 컴파일 에러)
  - 가장 유연한 참조 방법 => `expr()`
- `select()` + `expr()` 패턴을 자주 사용 => **`selectExpr()`** (효율성 ↑)
  - <i style="color:gray">"?? : 크큭..스파크의 진정한 능력을 보여주지.."</i>
  - 새로운 DataFrame 생성하는 복잡한 표현식 간단하게 표현 가능
  - 모든 유효한 비집계형 (non-aggregating) SQL 지정 가능 (단, 컬럼 식별 가능해야)
  - 집계 함수 (avg, count 등) 사용 가능

#### (3) 스파크 데이터 타입으로 변환하기

<details><summary class="point-color-can-hover">[5.4-3] 예제 펼치기 </summary>


```scala
import org.apache.spark.sql.functions.lit

df.select(expr("*"), lit(1).as("One")).show(2)

// +-----------------+-------------------+-----+---+
// |DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|One|
// +-----------------+-------------------+-----+---+
// |    United States|            Romania|   15|  1|
// |    United States|            Croatia|    1|  1|
// +-----------------+-------------------+-----+---+
```
```sql
-- SQL 에서 리터럴은 상숫값 (동일 표현)
SELECT *, 1 as One FROM dfTable LIMIT 2
```

</details>

- **리터럴(literal)** : 프로그래밍 언어의 리터럴 값 => 스파크가 이해할 수 있는 값으로 변환
  - 때로는 명시적 값 (상수값, 비교에 사용할 무언가 등..) 을 스파크에 전달해야함 => 리터럴 사용
  - 리터럴은 표현식
  - 어떤 상수나 프로그래밍으로 생성된 변숫값을 특정 컬럼의 값과 비교할 때 사용

#### (4) 컬럼 추가하기

<details><summary class="point-color-can-hover">[5.4-4] 예제 펼치기 </summary>

```scala
df.withColumn("numberOne", lit(1)).show(2)
// +-----------------+-------------------+-----+---------+
// |DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|numberOne|
// +-----------------+-------------------+-----+---------+
// |    United States|            Romania|   15|        1|
// |    United States|            Croatia|    1|        1|
// +-----------------+-------------------+-----+---------+

(df.withColumn("withinCountry", expr("ORIGIN_COUNTRY_NAME == DEST_COUNTRY_NAME"))
  .show(2))
// +-----------------+-------------------+-----+-------------+
// |DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|withinCountry|
// +-----------------+-------------------+-----+-------------+
// |    United States|            Romania|   15|        false|
// |    United States|            Croatia|    1|        false|
// +-----------------+-------------------+-----+-------------+

// 컬럼명 변경도 가능 (DEST_COUNTRY_NAME -> Destination)
df.withColumn("Destination", expr("DEST_COUNTRY_NAME")).columns
// res18: Array[String] = Array(DEST_COUNTRY_NAME, ORIGIN_COUNTRY_NAME, count, Destination)
```

</details>

- `withColumn(컬럼명, 값을 생성할 표현식)` 사용
  - 공식적 컬럼 추가 방법
  - 컬럼명 변경하여 추가도 가능

#### (5) 컬럼명 변경하기

<details><summary class="point-color-can-hover">[5.4-5] 예제 펼치기 </summary>

```scala
// DEST_COUNTRY_NAME -> dest 로 변경
df.withColumnRenamed("DEST_COUNTRY_NAME", "dest").columns
// res21: Array[String] = Array(dest, ORIGIN_COUNTRY_NAME, count)
```

</details>

- `withColumnRenamed(컬럼명, 변경할 문자열)` 사용

#### (6) 예약 문자와 키워드

<details><summary class="point-color-can-hover">[5.4-6] 예제 펼치기 </summary>

```scala
// 이스케이핑 필요 없는 예시 - 새로운 컬럼명을 나타내는 문자열
import org.apache.spark.sql.functions.expr
val dfWithLongColName = df.withColumn(
  "This Long Column-Name",
  expr("ORIGIN_COUNTRY_NAME"))
// dfWithLongColName: org.apache.spark.sql.DataFrame = [DEST_COUNTRY_NAME: string, ORIGIN_COUNTRY_NAME: string ... 2 more fields]


// 이스케이핑 필요한 예시 - 표현식으로 해당 컬럼을 참조 
(dfWithLongColName.selectExpr(
    "`This Long Column-Name`",
    "`This Long Column-Name` as `new col`")
  .show(2))
// +---------------------+-------+
// |This Long Column-Name|new col|
// +---------------------+-------+
// |              Romania|Romania|
// |              Croatia|Croatia|
// +---------------------+-------+

dfWithLongColName.createOrReplaceTempView("dfTableLong")

// 같은 DataFrame 생성
dfWithLongColName.select(col("This Long Column-Name")).columns
dfWithLongColName.select(expr("`This Long Column-Name`")).columns
```
```sql
-- SQL (동일 표현)
SELECT `This Long Column-Name`, `This Long Column-Name` as `new col`
FROM dfTableLong LIMIT 2

```

</details>

- 예약 문자(공백, 하이픈 (-) 등..) 는 컬럼명 사용 불가
  - => 사용하려면 **`` ` `` (백틱문자)** 를 이용한 이스케이핑(escaping) 필요
- 예약 문자나 키워드를 사용하는 '표현식'에는 이스케이프 처리 필요
  - '문자열'로 명시적 컬럼 참조 시에는 리터럴로 해석 => 예약문자 없이도 참조 가능

#### (7) 대소문자 구분

<details><summary class="point-color-can-hover">[5.4-7] 예제 펼치기 </summary>

```sql
set spark.sql.caseSensitive true
```

</details>

- 기본적으로 스파크는 대소문자를 가리지 않음
- `set spark.sql.caseSenstive true` 설정 시 구분 가능

#### (8) 컬럼 제거하기

<details><summary class="point-color-can-hover">[5.4-8] 예제 펼치기 </summary>

```scala
df.drop("ORIGIN_COUNTRY_NAME").columns
// res30: Array[String] = Array(DEST_COUNTRY_NAME, count)

dfWithLongColName.drop("ORIGIN_COUNTRY_NAME", "DEST_COUNTRY_NAME")
// res32: org.apache.spark.sql.DataFrame = [count: bigint, This Long Column-Name: string]
```

</details>

- `drop(컬럼명...)` 사용
  - 여러개를 인수로 넣어 다수의 컬럼을 한번에 제거 가능
- `select()` 로도 제거 가능

#### (9) 컬럼의 데이터 타입 변경하기

<details><summary class="point-color-can-hover">[5.4-9] 예제 펼치기 </summary>

```scala
// count 컬럼 : Integer -> String 으로 형변환
df.withColumn("count2", col("count").cast("long"))
```
```sql
-- SQL (동일 표현)
SELECT *, cast(count as string) AS count2 FROM dfTable
```

</details>

- `cast()` 사용
  - 특정 데이터 타입 => 다른 데이터 타입으로 형변환

#### (10) 로우 필터링하기

<details><summary class="point-color-can-hover">[5.4-10] 예제 펼치기 </summary>

```scala
// 동일한 표현식
df.filter(col("count") < 2).show(2)
df.where("count < 2").show(2)
// +-----------------+-------------------+-----+
// |DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|
// +-----------------+-------------------+-----+
// |    United States|            Croatia|    1|
// |    United States|          Singapore|    1|
// +-----------------+-------------------+-----+

// 여러 필터 적용 시 (순서 무관, 동시 적용)
df.where(col("count") < 2).where(col("ORIGIN_COUNTRY_NAME") =!= "Croatia").show(2)
// +-----------------+-------------------+-----+
// |DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|
// +-----------------+-------------------+-----+
// |    United States|          Singapore|    1|
// |          Moldova|      United States|    1|
// +-----------------+-------------------+-----+
```

```sql
-- SQL (동일 표현)
SELECT * FROM dfTable WHERE count < 2 LIMIT 2
SELECT * FROM dfTable WHERE count < 2 AND ORIGIN_COUNTRY_NAME != "Croatia" LIMIT 2
```

</details>

- 필터링을 하려면 참/거짓 판별 표현식 필요
  - 문자열 표현식, 컬럼을 다루는 기능으로 표현식 만듦
- `where()` `filter()` 사용 가능
  - 두 메서드 모두 같은 파라미터 타입 및 같은 연산 수행
  - `where()` 는 SQL과 유사
  - `filter()` 는 Dataset API를 이용해서 사용하면 Dataset 각 레코드에 적용 할 함수를 사용 가능 (=> 자세한건 11장)
- 스파크는 필터의 순서와 상관없이 동시에 모든 필터링 작업 수행
  - 같은 표현식에 여러 필터 적용시
  - 차례대로 AND 필터 연결 후 판단은 스파크에게 맡겨야 함

#### (11) 고유한 로우 얻기

<details><summary class="point-color-can-hover">[5.4-11] 예제 펼치기 </summary>

```scala
df.select("ORIGIN_COUNTRY_NAME", "DEST_COUNTRY_NAME").distinct().count()
// res41: Long = 256

df.select("ORIGIN_COUNTRY_NAME").distinct().count()
// res44: Long = 125
```

```sql
-- SQL (동일 표현)
SELECT COUNT(DISTINCT(ORIGIN_COUNTRY_NAME, DEST_COUNTRY_NAME)) FROM dfTable
SELECT COUNT(DISTINCT ORIGIN_COUNTRY_NAME) FROM dfTable
```

</details>

- `distinct()` 사용
  - 고윳값이나 중복되지 않은 값을 얻는 연산

#### (12) 무작위 샘플 만들기

<details><summary class="point-color-can-hover">[5.4-12] 예제 펼치기 </summary>

```scala
val seed = 5
val withReplacement = false
val fraction = 0.5
df.sample(withReplacement, fraction, seed).count()
// res46: Long = 126
```

</details>

- `sample(복원추출 여부, 추출 비율, seed)` 사용
  - 표본 데이터 추출 비율 (<=1.0) 지정 가능
  - 복원 추출 (sample with replacement), 비복원 추출 (sample without replacement) 사용 여부 지정 가능

#### (13) 임의 분할하기

<details><summary class="point-color-can-hover">[5.4-13] 예제 펼치기 </summary>

```scala
// 총합이 1이 아닐 경우 설정되는 default
val dataFrames = df.randomSplit(Array(0.25, 0.75), seed)
dataFrames(0).count() > dataFrames(1).count()
// res51: Boolean = false
```

</details>

- 임의 분할 (random split) : 원본 DataFrame 을 임의 크기로 '분할'
  - 머신러닝 알고리즘 사용 시 학습셋, 검증셋, 테스트셋 만들때 주로 사용
- `randomSplit(분할 가중치 Array, seed)`
  - 임의성(randomized) 을 가지므로 시드값(seed) 필수
  - DataFrame 비율은 총합이 1이 되게 지정 (아닐 경우 예제 비율로 지정됨)

#### (14) 로우 합치기와 추가하기

<details><summary class="point-color-can-hover">[5.4-14] 예제 펼치기 </summary>

```scala
import org.apache.spark.sql.Row
val schema = df.schema
val newRows = Seq(
  Row("New Country", "Other Country", 5L),
  Row("New Country 2", "Other Country 3", 1L)
)
val parallelizedRows = spark.sparkContext.parallelize(newRows)
val newDF = spark.createDataFrame(parallelizedRows, schema)

// df + newDF => 로우가 추가된 새로운 객체
(df.union(newDF)
  .where("count = 1")
  .where($"ORIGIN_COUNTRY_NAME" =!= "United States")
  .show()) // get all of them and we'll see our new rows at the end

// schema: org.apache.spark.sql.types.StructType = StructType(StructField(DEST_COUNTRY_NAME,StringType,true), StructField(ORIGIN_COUNTRY_NAME,StringType,true), StructField(count,LongType,true))
// newRows: Seq[org.apache.spark.sql.Row] = List([New Country,Other Country,5], [New Country 2,Other Country 3,1])
// parallelizedRows: org.apache.spark.rdd.RDD[org.apache.spark.sql.Row] = ParallelCollectionRDD[74] at parallelize at <console>:29
// newDF: org.apache.spark.sql.DataFrame = [DEST_COUNTRY_NAME: string, ORIGIN_COUNTRY_NAME: string ... 1 more field]
// +-----------------+-------------------+-----+
// |DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|
// +-----------------+-------------------+-----+
// |    United States|            Croatia|    1|
// |    United States|          Singapore|    1|
// |    United States|          Gibraltar|    1|
// |    United States|             Cyprus|    1|
// |    United States|            Estonia|    1|
// |    United States|          Lithuania|    1|
// |    United States|           Bulgaria|    1|
// |    United States|            Georgia|    1|
// |    United States|            Bahrain|    1|
// |    United States|   Papua New Guinea|    1|
// |    United States|         Montenegro|    1|
// |    United States|            Namibia|    1|
// |    New Country 2|    Other Country 3|    1|
// +-----------------+-------------------+-----+

```

</details>

- DataFrame은 불변성 (immutability)
  - DataFrame을 변경하는 레코드 추가는 불가능
  - => 원본 DataFrame을 새로운 DataFrame과 **통합(union)** (결합)
  - 단, 통합하려는 두 DataFrame은 반드시 동일한 스키마와 컬럼 수를 가져야 함
- `union()`
  - 현재 스키마가 아닌 컬럼 위치 기반으로 동작 (자동 정렬 X)
  - 로우가 추가된 DataFrame 을 참조하려면 새롭게 만들어진 DataFrame 사용해야하지만, <u>뷰나 테이블로 등록 시에는 동적으로 참조 가능</u>
- 컬럼 표현식과 문자 비교열 비교 시
  - (컬럼 표현식이 아닌) 컬럼의 실제값을 비교 대상 문자열과 비교하려면
  - 스칼라 사용 시 반드시 **`=!=` 함수** 사용
    - `=!=`, `===` 는 스파크의 Column 클래스에 정의된 함수
  - 파이썬은 그대로 `!=`, `==`

#### (15) 로우 정렬하기

<details><summary class="point-color-can-hover">[5.4-15] 예제 펼치기 </summary>

```scala
df.sort("count").show(5)
df.orderBy("count", "DEST_COUNTRY_NAME").show(5)
df.orderBy(col("count"), col("DEST_COUNTRY_NAME")).show(5)

// 정렬 기준 지정 (desc 오름)
import org.apache.spark.sql.functions.{desc, asc}

df.orderBy(expr("count desc")).show(2)
// 이거 왜 내림차순이아니라 오름차순으로 나오나... expr("count desc") 설정 안되고 default 정렬 (asc)로 설정되서 나오는 듯한데..?
// +-----------------+-------------------+-----+
// |DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|
// +-----------------+-------------------+-----+
// |          Moldova|      United States|    1|
// |    United States|            Croatia|    1|
// +-----------------+-------------------+-----+

df.orderBy(desc("count"), asc("DEST_COUNTRY_NAME")).show(2)
// +-----------------+-------------------+------+
// |DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME| count|
// +-----------------+-------------------+------+
// |    United States|      United States|370002|
// |    United States|             Canada|  8483|
// +-----------------+-------------------+------+
```

```sql
-- SQL
SELECT * FROM dfTable ORDER BY count DESC, DEST_COUNTRY_NAME ASC LIMIT 2
```

```scala
// sortWithinPartitions() 로 파티션별 정렬
(spark.read.format("json").load("/data/flight-data/json/*-summary.json")
  .sortWithinPartitions("count"))

// explain() 시
// == Physical Plan ==
// *(1) Sort [count#450L ASC NULLS FIRST], false, 0
// +- *(1) FileScan json [DEST_COUNTRY_NAME#448,ORIGIN_COUNTRY_NAME#449,count#450L] Batched: false, Format: JSON, Location: InMemoryFileIndex[file:/data/flight-data/json/2015-summary.json, file:/data/flight-data/json/2012..., PartitionFilters: [], PushedFilters: [], ReadSchema: struct<DEST_COUNTRY_NAME:string,ORIGIN_COUNTRY_NAME:string,count:bigint>
```
</details>

- `sort()` `orderBy()` 사용
  - 두 메서드는 완전히 같은 방식으로 동작 (`orderBy()` 내부에서 `sort()` 사용)
  - 다수 컬럼 지정, 컬럼 표현식, 문자열 사용 가능
  - 정렬 기준 : `asc()`, `desc()` 로 명확한 지정 가능 (기본 동작은 오름차순)
- 정렬된 DataFrame 의 NULL 값 표시 기준
  - `asc_nulls_first`, `desc_nulls_first`, `asc_nulls_last`, `desc_nulls_last` 로 기준 지정 가능
- 파티션 별 정렬 => `sortWithinPartitions()`
  - 트랜스포메이션 처리 전 성능 최적화를 위함
  - 더 자세한 튜닝과 최적화 내용은 3부에서

> `df.orderBy(expr("count desc"))` ?
> - count 컬럼을 desc() (내림차순) 으로 정렬되야 맞나?
>   - 실제로는 그렇게 동작 하지 않음 (=> 오름차순으로 정렬됨)
> - 잘못된 예제인듯한데..
>   - 관련 stackoverflow 질문 [링크 1](https://stackoverflow.com/questions/63112281/pyspark-sort-dataframe-by-expression) / [링크 2](https://stackoverflow.com/questions/63373479/sorting-2-columns-in-opposite-direction-does-not-work-using-expr-function)

#### (16) 로우 수 제한하기

<details><summary class="point-color-can-hover">[5.4-16] 예제 펼치기 </summary>

```scala
df.limit(5).show()

df.orderBy(expr("count desc")).limit(6).show()
// +--------------------+-------------------+-----+
// |   DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|
// +--------------------+-------------------+-----+
// |               Malta|      United States|    1|
// |Saint Vincent and...|      United States|    1|
// |       United States|            Croatia|    1|
// |       United States|          Gibraltar|    1|
// |       United States|          Singapore|    1|
// |             Moldova|      United States|    1|
// +--------------------+-------------------+-----+
//
// 뒷구르기하면서 봐도 결과가 이렇게 나와야할거같은데...
// df.orderBy(desc("count")).limit(6).show()
// +-----------------+-------------------+------+
// |DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME| count|
// +-----------------+-------------------+------+
// |    United States|      United States|370002|
// |    United States|             Canada|  8483|
// |           Canada|      United States|  8399|
// |    United States|             Mexico|  7187|
// |           Mexico|      United States|  7140|
// |   United Kingdom|      United States|  2025|
// +-----------------+-------------------+------+
```

```sql
-- SQL
SELECT * FROM dfTable LIMIT 6
```

</details>

- `limit(로우 수)` 사용
  - 추출할 로우 수 제한하여 추출

#### (17) repartition과 coalesce

<details><summary class="point-color-can-hover">[5.4-17] 예제 펼치기 </summary>

```scala
// DataFrame 현재 파티션 수 구하기
df.rdd.getNumPartitions // 1

// 전체 데이터 셔플
df.repartition(5)
// df.repartition(5).rdd.getNumPartitions => 5

// 특정 컬럼 기준 파티션 재분배
df.repartition(col("DEST_COUNTRY_NAME"))
// df.repartition(col("DEST_COUNTRY_NAME")).rdd.getNumPartition => 200

// 특정 컬럼 지정 + 파티션 수 지정
df.repartition(5, col("DEST_COUNTRY_NAME"))
// df.repartition(5, col("DEST_COUNTRY_NAME")).rdd.getNumPartitions => 5

// coalesce() 로 셔플없이 파티션 병합 (1 -> 5 -> 2)
df.repartition(5, col("DEST_COUNTRY_NAME")).coalesce(2)
// df.repartition(5, col("DEST_COUNTRY_NAME")).coalesce(2).rdd.getNumPartitions => 2
```

</details>

- 또 다른 최적화 기법? => 자주 필터링하는 컬럼 기준으로 데이터 분할
  - 파티셔닝 스키마와 파티션 수를 포함한 클러스터 전반의 물리적 데이터 구성 제어 가능
- `repartition()` : 전체 데이터 셔플
  - 향후 사용할 파티션 수 > 현재 파티션 수 인 경우 사용 (파티션 수 ↑)
  - 컬럼 기준으로 파티션을 만드는 경우 사용
    - 자주 필터링되는 컬럼이 있다면 해당 컬럼 기준으로 파티션 재분배 추천
  - 선택적으로 파티션 수 지정 가능
- `coalesce()` : 전체 데이터 셔플 없이 파티션 병합
  - **파티션 수를 줄이려면** coalesce 사용 (~~repartition~~ X)
- DataFrame 파티션 수 확인은 `df.rdd.getNumPartitions` 로 확인

#### (18) 드라이버로 로우 데이터 수집하기

<details><summary class="point-color-can-hover">[5.4-18] 예제 펼치기 </summary>

```scala
val collectDF = df.limit(10)
collectDF.take(5) // take() 는 정수형 값을 인수로 사용
collectDF.show() // show() => 결과를 정돈된 형태로 출력
// +-----------------+-------------------+-----+
// |DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|
// +-----------------+-------------------+-----+
// |    United States|            Romania|   15|
// |    United States|            Croatia|    1|
// |    United States|            Ireland|  344|
// |            Egypt|      United States|   15|
// |    United States|              India|   62|
// |    United States|          Singapore|    1|
// |    United States|            Grenada|   62|
// |       Costa Rica|      United States|  588|
// |          Senegal|      United States|   40|
// |          Moldova|      United States|    1|
// +-----------------+-------------------+-----+


collectDF.show(5, false)
// +-----------------+-------------------+-----+
// |DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|
// +-----------------+-------------------+-----+
// |United States    |Romania            |15   |
// |United States    |Croatia            |1    |
// |United States    |Ireland            |344  |
// |Egypt            |United States      |15   |
// |United States    |India              |62   |
// +-----------------+-------------------+-----+

collectDF.collect()
```

</details>

- 스파크는 '드라이버'에서 클러스터 상태 정보 유지
  - 로컬 환경에서 데이터 다룰 때는 '드라이버'로 데이터 수집
- 사용해본 데이터 수집 메서드 일부
  - `collect()` : 전체 DataFrame의 모든 데이터 수집
  - `take()` : 상위 N개 로우 반환
  - `show()` : 여러 로우를 보기 좋게 출력
- `toLocalIterator()` : 전체 데이터셋에 대한 반복(iterate) 처리를 위해 '드라이버'로 로우를 모으는 방법
  - iterator(반복자) 로 모든 파티션의 데이터를 '드라이버'에 전달
  - 데이터셋의 파티션을 차례대로 반복 처리 가능
- 드라이버로 모든 데이터 컬렉션을 수집하는 건
  - => **매우 큰 비용** (CPU, 메모리, 네트워크)
  - 차례대로 처리하므로 처리 비용 엄청남 (병렬 연산 X)
- 따라서 대규모 데이터셋에 `collect()` 나 매우 큰 파티션에 대해 `toLocalIterator()` 사용 시 => 드라이버 비정상적 종료


### 5.5 정리
- DataFrame 기본 연산
- DataFrame 사용에 필요한 개념, 다양한 기능

### 📒 단어장
- 비결정론적(nondeterministically) : = 매번 변하는
- ETL : `추출(Extract)` - `변환(Transform)` - `적재(Load)`  <i style="color:lightgray">(친숙하쥬?)</i>
