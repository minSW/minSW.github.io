---
title: '&#039;Spark The Definitive Guide&#039; 8장 - 조인'
date: 2021-02-15 02:27:10
categories: spark
tags:
  - spark
  - apache
  - book
  - study
---

<br/>

<img src="https://user-images.githubusercontent.com/26691216/108165034-3ae19600-7135-11eb-8b40-8175f455337b.gif" width=400/>
<center><h2>_ _ _</h2></center>

<br/>

---

# CHAPTER 8 조인

CHAPTER 7 은 단일 데이터셋의 집계 방법 소개
CHPATER 8 은 다양한 데이터 셋을 합께 결합하는 **조인** 타입과 사용법, 실제 동작방식 소개
<i style="color:lightgray">(=> 메모리 부족 상황을 회피하는 등의 문제 상황 해결에 도움이 되는 기초 지식이 될지도)</i> 

### 8.1 조인 표현식
- **조인 표현식(join expression)** 
  - 스파크의 왼쪽과 오른쪽 데이터셋의 하나 이상의 키값 을 비교하여 **결합 여부를 결정** (=> 평과 결과)
- 스파크 지원 조인 정책
  - ex. 동등 조인(equal-join) : `왼쪽 키 == 오른쪽 키` 일때만 데이터셋 결합
  - 복합 데이터 타입(배열, 리스트 ..) 사용하는 등의 복잡한 조인 정책도 가능

### 8.2 조인 타입

<details><summary class="point-color-can-hover">[8.2] 예제용 데이터셋 생성 예제 펼치기</summary>

```scala
// 예제용 데이터셋 생성
val person = (Seq(
    (0, "Bill Chambers", 0, Seq(100)),
    (1, "Matei Zaharia", 1, Seq(500, 250, 100)),
    (2, "Michael Armbrust", 1, Seq(250, 100)))
  .toDF("id", "name", "graduate_program", "spark_status"))
val graduateProgram = (Seq(
    (0, "Masters", "School of Information", "UC Berkeley"),
    (2, "Masters", "EECS", "UC Berkeley"),
    (1, "Ph.D.", "EECS", "UC Berkeley"))
  .toDF("id", "degree", "department", "school"))
val sparkStatus = (Seq(
    (500, "Vice President"),
    (250, "PMC Member"),
    (100, "Contributor"))
  .toDF("id", "status"))

// 테이블로 등록
person.createOrReplaceTempView("person")
graduateProgram.createOrReplaceTempView("graduateProgram")
sparkStatus.createOrReplaceTempView("sparkStatus")
```

</details>

- **조인 타입** : **결과 데이터셋*에 어떤 데이터가 있어야 하는지 결정
- 스파크 지원 조인 타입
  - 내부 조인(inner join) : 왼쪽 and 오른쪽에 키가 있는 로우 유지
  - 외부 조인(outer join) : 왼쪽 or 오른쪽에 키가 있는 로우 유지
  - 왼쪽 외부 조인(left outer join) : 왼쪽에 키가 있는 로우 유지
  - 오른쪽 외부 조인(right outer join) : 오른쪽에 키가 있는 로우 유지
  - 왼쪽 세미 조인(left semi join) : 왼쪽 키가 오른쪽에 있는 경우, 키가 일치하는 왼쪽 데이터셋만 유지
  - 왼쪽 안티 조인(left anti join) : 왼쪽 키가 오른쪽에 없는 경우, 키가 일치하지 않는 왼쪽 데이터셋만 유지
  - 자연 조인(natural join) : 두 데이터셋에서 동일한 이름 가진 컬럼을 암시적(Implicit)으로 결합하는 조인
  - 교차 조인(corss join) 또는 카테시안 조인(Cartesian join) : 왼쪽 모든 로우와 오른쪽 모든 로우 조합
 - 조인 타입 예제는 [다음](#스파크-조인-타입-예제) 참고


#### # 스파크 조인 타입 예제 
<details><summary class="point-color-can-hover">[8.3~10] 조인 비교 예제 펼치기</summary>

```scala
// val joinExpression = person.col("graduate_program") === graduateProgram.col("id")
// joinExpression: org.apache.spark.sql.Column = (graduate_program = id)

// 3. 내부 조인
// == INNER JOIN
var joinType = "inner"
person.join(graduateProgram, joinExpression, joinType).show()
// +---+----------------+----------------+---------------+---+-------+--------------------+-----------+
// | id|            name|graduate_program|   spark_status| id| degree|          department|     school|
// +---+----------------+----------------+---------------+---+-------+--------------------+-----------+
// |  0|   Bill Chambers|               0|          [100]|  0|Masters|School of Informa...|UC Berkeley|
// |  2|Michael Armbrust|               1|     [250, 100]|  1|  Ph.D.|                EECS|UC Berkeley|
// |  1|   Matei Zaharia|               1|[500, 250, 100]|  1|  Ph.D.|                EECS|UC Berkeley|
// +---+----------------+----------------+---------------+---+-------+--------------------+-----------+

// 4. 외부 조인
// == FULL OUTER JOIN
joinType = "outer"
person.join(graduateProgram, joinExpression, joinType).show()
// +----+----------------+----------------+---------------+---+-------+--------------------+-----------+
// |  id|            name|graduate_program|   spark_status| id| degree|          department|     school|
// +----+----------------+----------------+---------------+---+-------+--------------------+-----------+
// |   1|   Matei Zaharia|               1|[500, 250, 100]|  1|  Ph.D.|                EECS|UC Berkeley|
// |   2|Michael Armbrust|               1|     [250, 100]|  1|  Ph.D.|                EECS|UC Berkeley|
// |null|            null|            null|           null|  2|Masters|                EECS|UC Berkeley|
// |   0|   Bill Chambers|               0|          [100]|  0|Masters|School of Informa...|UC Berkeley|
// +----+----------------+----------------+---------------+---+-------+--------------------+-----------+


// 5. 왼쪽 외부 조인
// == LEFT OUTER JOIN
joinType = "left_outer"
graduateProgram.join(person, joinExpression, joinType).show()
// +---+-------+--------------------+-----------+----+----------------+----------------+---------------+
// | id| degree|          department|     school|  id|            name|graduate_program|   spark_status|
// +---+-------+--------------------+-----------+----+----------------+----------------+---------------+
// |  0|Masters|School of Informa...|UC Berkeley|   0|   Bill Chambers|               0|          [100]|
// |  2|Masters|                EECS|UC Berkeley|null|            null|            null|           null|
// |  1|  Ph.D.|                EECS|UC Berkeley|   2|Michael Armbrust|               1|     [250, 100]|
// |  1|  Ph.D.|                EECS|UC Berkeley|   1|   Matei Zaharia|               1|[500, 250, 100]|
// +---+-------+--------------------+-----------+----+----------------+----------------+---------------+


// 6. 오른쪽 외부 조인
// == RIGHT OUTER JOIN
joinType = "right_outer"
person.join(graduateProgram, joinExpression, joinType).show()
// +----+----------------+----------------+---------------+---+-------+--------------------+-----------+
// |  id|            name|graduate_program|   spark_status| id| degree|          department|     school|
// +----+----------------+----------------+---------------+---+-------+--------------------+-----------+
// |   0|   Bill Chambers|               0|          [100]|  0|Masters|School of Informa...|UC Berkeley|
// |null|            null|            null|           null|  2|Masters|                EECS|UC Berkeley|
// |   2|Michael Armbrust|               1|     [250, 100]|  1|  Ph.D.|                EECS|UC Berkeley|
// |   1|   Matei Zaharia|               1|[500, 250, 100]|  1|  Ph.D.|                EECS|UC Berkeley|
// +----+----------------+----------------+---------------+---+-------+--------------------+-----------+


// 7. 왼쪽 세미 조인
// == LEFT SEMI JOIN
joinType = "left_semi"
graduateProgram.join(person, joinExpression, joinType).show()
// +---+-------+--------------------+-----------+
// | id| degree|          department|     school|
// +---+-------+--------------------+-----------+
// |  0|Masters|School of Informa...|UC Berkeley|
// |  1|  Ph.D.|                EECS|UC Berkeley|
// +---+-------+--------------------+-----------+

// 세미조인은 => 값이 존재하면, 중복 키(id=0) 가 존재해도 로우 포함
val gradProgram2 = graduateProgram.union(Seq(
    (0, "Masters", "Duplicated Row", "Duplicated School")).toDF())
gradProgram2.createOrReplaceTempView("gradProgram2")

gradProgram2.join(person, joinExpression, joinType).show()
// +---+-------+--------------------+-----------------+
// | id| degree|          department|           school|
// +---+-------+--------------------+-----------------+
// |  0|Masters|School of Informa...|      UC Berkeley|
// |  1|  Ph.D.|                EECS|      UC Berkeley|
// |  0|Masters|      Duplicated Row|Duplicated School|
// +---+-------+--------------------+-----------------+


// 8. 왼쪽 안티 조인
// == LEFT ANTI JOIN
joinType = "left_anti"
graduateProgram.join(person, joinExpression, joinType).show()
// +---+-------+----------+-----------+
// | id| degree|department|     school|
// +---+-------+----------+-----------+
// |  2|Masters|      EECS|UC Berkeley|
// +---+-------+----------+-----------+


// 9. 자연 조인
// == NATURAL JOIN


// 10. 교차 조인
// == CROSS JOIN
joinType = "cross"
graduateProgram.join(person, joinExpression, joinType).show()
// +---+-------+--------------------+-----------+---+----------------+----------------+---------------+
// | id| degree|          department|     school| id|            name|graduate_program|   spark_status|
// +---+-------+--------------------+-----------+---+----------------+----------------+---------------+
// |  0|Masters|School of Informa...|UC Berkeley|  0|   Bill Chambers|               0|          [100]|
// |  1|  Ph.D.|                EECS|UC Berkeley|  2|Michael Armbrust|               1|     [250, 100]|
// |  1|  Ph.D.|                EECS|UC Berkeley|  1|   Matei Zaharia|               1|[500, 250, 100]|
// +---+-------+--------------------+-----------+---+----------------+----------------+---------------+

// 교차 조인은 => 명시적메서드 호출도 가능 (하단 예제는 키워드 없이 전체 교차 호출)
person.crossJoin(graduateProgram).show()
// +---+----------------+----------------+---------------+---+-------+--------------------+-----------+
// | id|            name|graduate_program|   spark_status| id| degree|          department|     school|
// +---+----------------+----------------+---------------+---+-------+--------------------+-----------+
// |  0|   Bill Chambers|               0|          [100]|  0|Masters|School of Informa...|UC Berkeley|
// |  1|   Matei Zaharia|               1|[500, 250, 100]|  0|Masters|School of Informa...|UC Berkeley|
// |  2|Michael Armbrust|               1|     [250, 100]|  0|Masters|School of Informa...|UC Berkeley|
// |  0|   Bill Chambers|               0|          [100]|  2|Masters|                EECS|UC Berkeley|
// |  1|   Matei Zaharia|               1|[500, 250, 100]|  2|Masters|                EECS|UC Berkeley|
// |  2|Michael Armbrust|               1|     [250, 100]|  2|Masters|                EECS|UC Berkeley|
// |  0|   Bill Chambers|               0|          [100]|  1|  Ph.D.|                EECS|UC Berkeley|
// |  1|   Matei Zaharia|               1|[500, 250, 100]|  1|  Ph.D.|                EECS|UC Berkeley|
// |  2|Michael Armbrust|               1|     [250, 100]|  1|  Ph.D.|                EECS|UC Berkeley|
// +---+----------------+----------------+---------------+---+-------+--------------------+-----------+
```

```sql
-- SQL
-- 3. 내부 조인
SELECT * FROM person JOIN graduateProgram
  ON person.graduate_program = graduateProgram.id

SELECT * FROM person INNER JOIN graduateProgram
  ON person.graduate_program = graduateProgram.id

-- 4. 외부 조인
SELECT * FROM person FULL OUTER JOIN graduateProgram
  ON graduate_program = graduateProgram.id

-- 5. 왼쪽 외부 조인
SELECT * FROM graduateProgram LEFT OUTER JOIN person
  ON person.graduate_program = graduateProgram.id

-- 6. 오른쪽 외부 조인
SELECT * FROM person RIGHT OUTER JOIN graduateProgram
  ON person.graduate_program = graduateProgram.id

-- 7. 왼쪽 세미 조인
SELECT * FROM gradProgram2 LEFT SEMI JOIN person
  ON gradProgram2.id = person.graduate_program

-- 8. 왼쪽 안티 조인
SELECT * FROM graduateProgram LEFT ANTI JOIN person
  ON graduateProgram.id = person.graduate_program

-- 9. 자연 조인
SELECT * FROM graduateProgram NATURAL JOIN person

-- 10. 교차 조인
SELECT * FROM graduateProgram CROSS JOIN person
  ON graduateProgram.id = person.graduate_program

SELECT * FROM graduateProgram CROSS JOIN person
```

</details>


### 8.3 내부 조인


<details><summary class="point-color-can-hover">[8.3] 내부 조인 (기본 조인) 예제 펼치기</summary>

```scala
// 내부 조인은 기본 조인 방식
val joinExpression = person.col("graduate_program") === graduateProgram.col("id")

person.join(graduateProgram, joinExpression).show()
// +---+----------------+----------------+---------------+---+-------+--------------------+-----------+
// | id|            name|graduate_program|   spark_status| id| degree|          department|     school|
// +---+----------------+----------------+---------------+---+-------+--------------------+-----------+
// |  0|   Bill Chambers|               0|          [100]|  0|Masters|School of Informa...|UC Berkeley|
// |  2|Michael Armbrust|               1|     [250, 100]|  1|  Ph.D.|                EECS|UC Berkeley|
// |  1|   Matei Zaharia|               1|[500, 250, 100]|  1|  Ph.D.|                EECS|UC Berkeley|
// +---+----------------+----------------+---------------+---+-------+--------------------+-----------+

// wrongJoinExpression (두 DF 모두에 키가 존재하지않으면 => 빈 결과 DataFrame 반환)
val wrongJoinExpression = person.col("name") === graduateProgram.col("school")
person.join(graduateProgram, wrongJoinExpression).show()
// +---+----+----------------+------------+---+------+----------+------+
// | id|name|graduate_program|spark_status| id|degree|department|school|
// +---+----+----------------+------------+---+------+----------+------+
// +---+----+----------------+------------+---+------+----------+------+
```

```sql
-- SQL
SELECT * FROM person JOIN graduateProgram
  ON person.graduate_program = graduateProgram.id
```

</details>

- 내부 조인
  - DataFrame이나 테이블에 존재하는 키 평가, `True` 로 평가되는 로우만 결합
  - 둘 다 키가 존재하지 않으면? => 빈 값
  - **기본 조인 방식**
    - ex. `왼DF.join(오DF, 왼DF.col('A') === 오DF.col('B'), inner)`
  - joinType (Optional) 으로 조인 타입 명확하게 지정도 가능

- `join(DF, joinExpress, (joinType))`

### 8.4 외부 조인

- 외부 조인
  - DataFrame이나 테이블에 존재하는 키를 평가, 평과 결과(`True`, `False`) 로우를 포함하여 조인
  - 둘 다 일치하는 로우가 없으면? => null 삽입

### 8.5 왼쪽 외부 조인

- 왼쪽 외부 조인
  - DataFrame이나 테이블에 존재하는 키를 평가, <u>왼쪽 DataFrame의 모든 로우</u>와 <u>왼쪽과 일치하는 오른쪽 DataFrame의 로우</u> 포함
  - 오른쪽에 일치 로우 없으면? => null 삽입

### 8.6 오른쪽 외부 조인

- 오른쪽 외부 조인
  - DataFrame이나 테이블에 존재하는 키를 평가, <u>오른쪽 DataFrame의 모든 로우</u>와 <u>오른쪽과 일치하는 왼쪽 DataFrame의 로우</u> 포함
  - 왼쪽 일치 로우 없으면? => null 삽입
- 왼쪽 외부 조인 반대지 뭐..

### 8.7 왼쪽 세미 조인

- 왼쪽 세미 조인
  - 오른쪽 DataFrame의 어떤 값도 **포함 X** (단지 값이 존재하는지 확인하는 용도)
  - 오른쪽에 값이 존재하면 => 왼쪽에 중복 키가 존재하더라도 해당 로우는 결과에 포함
- 기존 조인 기능보다는 달리 **DataFrame 필터** 에 가깝다


### 8.8 왼쪽 안티 조인

- 왼쪽 안티 조인 <-> 왼쪽 세미 조인 
  - 오른쪽 DataFrame의 어떤 값도 **포함 X** (== 세미)
  - 단, 오른쪽 값이 존재하면 유지하는 대신 => 오른쪽에서 **관련된 키가 없는 로우**만 결과에 포함
  - (== SQL 의 `NOT IN`)


### 8.9 자연 조인

- 자연 조인
  - 조인하려는 컬럼을 암시적 추정 (=> 일치하는 컬럼을 찾고 결과를 반환)
  - 왼쪽, 오른쪽, 외부 자연 조인 사용 가능
- 하지만 암시적 조인은 위험. 조심해서 사용할 것

### 8.10 교차 조인(카테시안 조인)

- 교차 조인
  - 조건절을 기술하지 않은 내부조인
  - 왼쪽 모든 로우 X 오른쪽 모든 로우 결합
- 교차 조인도 매우 위험. (키워드 미사용 시 왼쪽X오른쪽 로우 수 만큼 생성)
  - 반드시 필요한 경우에, **명시적으로 정의** 할 것
  - (참고) `spark.sql.crossJoin.enable = true` : 교차 조인 시 발생 경고 제거. 스파크가 교차조인을 다른 조인으로 처리하지 않도록 강제


### 8.11 조인 사용 시 문제점

- [문제 1] 복합 데이터 타입의 조인
  - 불리언을 반환하는 모든 표현식 => 조인 표현식으로 사용 O

    <details><summary class="point-color-can-hover">[8.11.1] '복합 데이터 타입의 조인' 예제 펼치기</summary>

    ```scala
    import org.apache.spark.sql.functions.expr

    (person.withColumnRenamed("id", "personId")
      .join(sparkStatus, expr("array_contains(spark_status, id)")).show())
    // +--------+----------------+----------------+---------------+---+--------------+
    // |personId|            name|graduate_program|   spark_status| id|        status|
    // +--------+----------------+----------------+---------------+---+--------------+
    // |       0|   Bill Chambers|               0|          [100]|100|   Contributor|
    // |       1|   Matei Zaharia|               1|[500, 250, 100]|500|Vice President|
    // |       1|   Matei Zaharia|               1|[500, 250, 100]|250|    PMC Member|
    // |       1|   Matei Zaharia|               1|[500, 250, 100]|100|   Contributor|
    // |       2|Michael Armbrust|               1|     [250, 100]|250|    PMC Member|
    // |       2|Michael Armbrust|               1|     [250, 100]|100|   Contributor|
    // +--------+----------------+----------------+---------------+---+--------------+
    ```
    ```sql
    -- SQL
    SELECT * FROM
      (select id as personId, name, graduate_program, spark_status FROM person)
      INNER JOIN sparkStatus ON array_contains(spark_status, id)
    ```

    </details>

- [문제 2] 중복 컬럼명 처리
  - 결과 DataFrame에서 중복된 컬럼명 다루기
    - 각 컬럼은 스파크 SQL 엔진(카탈리스트) 내에 고유 ID 존재 (직접 참조는 불가)
    - 중복 컬럼명이 존재하는 DataFrame 사용 시 특정 컬럼 참조가 어려움
  - 문제 발생 상황
    - 조인에 사용할 DataFrame의 특정 키가 동일한 이름을 가지며, 키가 제거 되지않도록 조인 표현식에 명시한 경우
    - 조인 대상이 아닌 두 개의 컬럼이 동일 이름을 가진 경우
  - Solution 1) 다른 조인 표현식 사용
    - ~~불리언 형태 조인 표현식~~ => 문자열이나 시퀀스 형태로 변경
    - 조인할 때 두 컬럼 중 하나가 자동 제거됨
  - Solution 2) 조인 후 컬럼 제거
    - 조인 시 동일한 키 이름 사용하거나 원본 DataFrame에 동일한 컬럼명 존재 시 => 원본 DataFrame을 사용해 컬럼을 참조
    - 스파크의 SQL 분석 프로세스 특성 활용 - "명시적으로 참조된 컬럼은 검증 필요 X. 스파크 분석 단계 패스"
    - `col()` 메서드로 컬럼 고유 ID로 해당 컬럼을 암시적 지정 가능 (예제 참고, ~~`column()`~~ X)
  - Solution 3) 조인 전 컬럼명 변경

    <details><summary class="point-color-can-hover">[8.11.2] '중복 컬럼명 처리' 예제 펼치기</summary>

    ```scala
    // 잘못된 데이터셋 생성 (두 개의 graduate_program 컬럼 존재)
    val gradProgramDupe = graduateProgram.withColumnRenamed("id", "graduate_program")
    val joinExpr = gradProgramDupe.col("graduate_program") === person.col(
      "graduate_program")

    person.join(gradProgramDupe, joinExpr).show()
    // +---+----------------+----------------+---------------+----------------+-------+--------------------+-----------+
    // | id|            name|graduate_program|   spark_status|graduate_program| degree|          department|     school|
    // +---+----------------+----------------+---------------+----------------+-------+--------------------+-----------+
    // |  0|   Bill Chambers|               0|          [100]|               0|Masters|School of Informa...|UC Berkeley|
    // |  2|Michael Armbrust|               1|     [250, 100]|               1|  Ph.D.|                EECS|UC Berkeley|
    // |  1|   Matei Zaharia|               1|[500, 250, 100]|               1|  Ph.D.|                EECS|UC Berkeley|
    // +---+----------------+----------------+---------------+----------------+-------+--------------------+-----------+

    // 중복된 컬럼 중 하나를 참조하면 에러 발생 ("Reference 'graduate_program' is ambiguous ..")
    person.join(gradProgramDupe, joinExpr).select("graduate_program").show()
    ```
    ```scala
    // * 중복 컬럼명 처리 Solution *
    // Solution 1) 다른 조인 표현식 사용
    person.join(gradProgramDupe,"graduate_program").select("graduate_program").show()
    // +----------------+
    // |graduate_program|
    // +----------------+
    // |               0|
    // |               1|
    // |               1|
    // +----------------+


    // Solution 2) 조인 후 컬럼 제거
    (person.join(gradProgramDupe, joinExpr).drop(person.col("graduate_program"))
      .select("graduate_program").show())
    // +----------------+
    // |graduate_program|
    // +----------------+
    // |               0|
    // |               1|
    // |               1|
    // +----------------+

    val joinExpr = person.col("graduate_program") === graduateProgram.col("id")
    person.join(graduateProgram, joinExpr).drop(graduateProgram.col("id")).show()
    // +---+----------------+----------------+---------------+-------+--------------------+-----------+
    // | id|            name|graduate_program|   spark_status| degree|          department|     school|
    // +---+----------------+----------------+---------------+-------+--------------------+-----------+
    // |  0|   Bill Chambers|               0|          [100]|Masters|School of Informa...|UC Berkeley|
    // |  2|Michael Armbrust|               1|     [250, 100]|  Ph.D.|                EECS|UC Berkeley|
    // |  1|   Matei Zaharia|               1|[500, 250, 100]|  Ph.D.|                EECS|UC Berkeley|
    // +---+----------------+----------------+---------------+-------+--------------------+-----------+


    // Solution 3) 조인 전 컬럼명 변경
    val gradProgram3 = graduateProgram.withColumnRenamed("id", "grad_id")
    val joinExpr = person.col("graduate_program") === gradProgram3.col("grad_id")
    person.join(gradProgram3, joinExpr).show()
    // +---+----------------+----------------+---------------+-------+-------+--------------------+-----------+
    // | id|            name|graduate_program|   spark_status|grad_id| degree|          department|     school|
    // +---+----------------+----------------+---------------+-------+-------+--------------------+-----------+
    // |  0|   Bill Chambers|               0|          [100]|      0|Masters|School of Informa...|UC Berkeley|
    // |  2|Michael Armbrust|               1|     [250, 100]|      1|  Ph.D.|                EECS|UC Berkeley|
    // |  1|   Matei Zaharia|               1|[500, 250, 100]|      1|  Ph.D.|                EECS|UC Berkeley|
    // +---+----------------+----------------+---------------+-------+-------+--------------------+-----------+
    ```
    
    </details>

### 8.12 스파크의 조인 수행 방식

- 두 가지 핵심 (내부) 전략
  - 노드간 네트워크 통신 전략
  - 노드별 연산 전략
- 스파크 조인 수행 방식 이해하면 뭐가 좋은데?
  - 빠르게 완료되는 작업 vs 절대 완료되지 않는 작업간의 차이 이해 가능
- **네트워크 통신 전략**
  - 스파크는 조인 시 두 가지 클러스터 통신방식 활용
    - **셔플 조인(shuffle join)** => 전체 노드간 통신 유발
    - **브로드캐스트 조인(broadcast join)** => 필요없음
  - 사실 이런 내부 최적화 기술은 CBO 개선 후 더 나은 통신 전략이 생기면 얼마든지 바뀔 수 있고 ...
  - 따라서 일반적 상황에서 정확히 어떤일이 일어나는지 고수준 예제로 이해해보자
  - 예제 설정은 이해를 돕기위한 데이터 크기가 극단적인 상황 가정 (아주 크거나, 아주 작거나)
- (1) 큰 테이블 + 큰테이블 조인
  - **셔플 조인** 발생 (전체 노드간 통신)
    - 조인에 사용한 특정 키나 키 집합을 어떤 노드가 가졌는 지에 따라 해당 노드와 데이터 공유
    - => 즉 높은 네트워크 비용, 많은 자원 사용
    - <a style="color:lightgray">(IoT 예제는 무슨 소린지 약간 애매..)</a>
  - *"즉, **전체** 조인 프로세스가 진행되는 동안 (데이터 파티셔닝 없이) 모든 워커 노드(그리고 모든 파티션)에서 통신이 발생함을 의미"*
- (2) 큰테이블 + 작은 테이블 조인
  - 테이블이 단일 워커 노드의 메모리에서 감당할정도로 작으면 조인 연산 최적화 가능
  - **브로드캐스트 조인** 이 훨씬 효율적
    - 작은 DataFrame을 클러스터의 전체 워커 노드에 복제
    - 자원을 많이 쓰는거 같아도, 프로세스 중 전체 노드가 통신하는 현상 방지 & 다른 워커 기다림없이 작업 수행 가능
    - => 즉 대규모 통신은 발생하지만 노드 간 추가적인 통신 발생 X
  - 따라서 모든 단일 노드에서 개별적 조인 수행
    - CPU가 가장 큰 병목 구간
  - 브로드 캐스트 사용
    - DataFrame API : `broadcast(작은DF)` 로 사용 힌트 제공 가능 => but 항상 동일한 실행계획은 아님
    - SQL : `/*_ MAPJOIN() */` 힌트 제공 가능 (=`MAPJOIN` = `BROADCAST` = `BROADCASTJOIN`) => but **강제성 X** 
    
    <details><summary class="point-color-can-hover">[8.12] (2) '브로드캐스트 조인' 확인 예제 펼치기</summary>

    ```scala
    // [큰테이블 - 작은 테이블 조인]
    // => 스파크가 자동으로 데이터셋을 브로드캐스트 조인으로 설정
    val joinExpr = person.col("graduate_program") === graduateProgram.col("id")

    person.join(graduateProgram, joinExpr).explain()
    // joinExpr: org.apache.spark.sql.Column = (graduate_program = id)
    // == Physical Plan ==
    // *(1) BroadcastHashJoin [graduate_program#11], [id#26], Inner, BuildLeft
    // :- BroadcastExchange HashedRelationBroadcastMode(List(cast(input[2, int, false] as bigint)))
    // :  +- LocalTableScan [id#9, name#10, graduate_program#11, spark_status#12]
    // +- LocalTableScan [id#26, degree#27, department#28, school#29]


    // DataFrame API => 옵티마이저에게 브로드캐스트 조인하도록 힌트 전달 가능
    import org.apache.spark.sql.functions.broadcast

    val joinExpr = person.col("graduate_program") === graduateProgram.col("id")
    person.join(broadcast(graduateProgram), joinExpr).explain()
    // joinExpr: org.apache.spark.sql.Column = (graduate_program = id)
    // == Physical Plan ==
    // *(1) BroadcastHashJoin [graduate_program#11], [id#26], Inner, BuildRight
    // :- LocalTableScan [id#9, name#10, graduate_program#11, spark_status#12]
    // +- BroadcastExchange HashedRelationBroadcastMode(List(cast(input[0, int, false] as bigint)))
    //    +- LocalTableScan [id#26, degree#27, department#28, school#29]
    ```
    ```sql
    -- SQL : 조인 수행 힌트 주기 (강제성 X) 
    SELECT /*+ MAPJOIN(graduateProgram) */ * FROM person JOIN graduateProgram
      ON person.graduate_program = graduateProgram.id
    ```

    </details>

  - 단점
    - 큰 데이터 브로드 캐스트 시 고비용 수집 연산으로 드라이버 노드 비정상적 종료 가능성
- (3) 아주 작은 테이블 사이 조인
  - 스파크가 알아서 하게 두는게 BEST
  - 필요하면 브로드캐스트 조인 강제 지정 가능하긴 한데..

### 8.13 정리

- 이번엔 정리보다는 팁 느낌...
- 조인 전 데이터를 적절히 분할 시 셔플이 계획되어있어도 동일 머신에 두 DataFrame 의 데이터가 있을 수 있다.
  - => 셔플 피하고 훨씬 효율적 조인 가능
  - 따라서 일부 데이터를 실험용으로 사전 분할 해 조인 수행 시 성능 향상되는지 확인 해볼 것
- 데이터소스는 조인 순서를 결정하는데 부가적 영향 미칠 수 있다. => 다음 챕터 참고
- 일부 조인은 필터 임무를 수행하므로, 네트워크 교환 데이터를 줄여 워크로드 성능 향상 쉽게 할 수 있다.

### 📒 단어장
- 비용 기반 옵티마이저 (cost-based optimizer, CBO)
