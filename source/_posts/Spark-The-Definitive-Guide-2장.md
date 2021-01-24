---
title: '&#039;Spark The Definitive Guide&#039; 2장 - 스파크 찍어먹기'
date: 2021-01-24 18:42:51
categories: spark
tags:
	- spark
	- apache
	- book
	- study
	- mwolkka
---


<img src="https://user-images.githubusercontent.com/26691216/105627228-e66e3200-5e78-11eb-9ea6-2e3662267b7a.jpg" width=200 />
<center> 도커 이미지 사용시 Zeppelin에 예제 코드가 있다 <br/>
나처럼 시력 검사&타자 연습 하느라 진빼지말고 Chapter2는 그냥 예제 코드를 쓰도록 하자... </center>


<center><h2>_ _ _</h2></center>

<br/>

---


# CHAPTER 2 스파크 간단히 살펴보기

DataFrame, SQL 을 사용해 클러스터, 스파크 애플리케이션, 구조적 API 를 살펴보고
스파크의 핵심용어와 개념, 사용법을 익힌다.


### 2.1 스파크의 기본 아키텍처
> 스파크 애플리케이션을 이해하기 위한 핵심사항
> - 스파크는 사용가능한 자원을 파악하기 위해 **클러스터 매니저** 사용
> - **드라이버** 프로세스는 주어직 작업을 완료하기위해, 드라이버 프로그램의 명령을 **익스큐터**에서 실행할 책임이 있음
>

- 스파크는 클러스터의 데이터 처리 작업을 관리 / 조율
  - 컴퓨터 클러스터는 여러 컴퓨터의 자원을 모아 하나의 컴퓨터 처럼 사용
  - 클러스터에서 작업을 조율할 수 있는 프레임워크 => **스파크**
- 스파크가 연산에 사용할 클러스터를 관리하는 **클러스터 매니저**
  - 스파크 standalone 클러스터 매니저, 하둡 YARN, Mesos
  - 역할
    - 사용자 : 스파크 애플리케이션 제출 (submit)
    - -> 클러스터 매니저 : 애플리케이션 실행에 필요한 자원 할당 
    - -> 할당받은 자원으로 작업 처리
- 스파크 애플리케이션 = `driver` 프로세스 + 다수의 `executor` 프로세스
  - `driver` 프로세스
    - 클러스터 노드 중 하나에서 실행. main() 함수 실행
    - 심장과 같은 존재로, 애플리케이션 생명 주기 동안 관련 정보 모두 유지
  - `executor` 프로세스
    - driver 가 할당한 작업 수행 & 진행 상황을 driver에게 보고
    - 대부분 스파크 코드를 실행하는 역할로, 스파크 언어 API를 통해 다양한 언어로 실행 가능

### 2.2 스파크의 다양한 언어 API
- 스파크는 모든 언어에 맞는 몇몇 '핵심 개념' 제공
  - 핵심개념 -> (클러스터 머신에서 실행되는) 스파크 코드 로 변환
  - 구조적 API만으로 작성된 코드는 언어에 무관하게 유사 성능
- 언어별 요약 정보
  - Scala : 스파크가 스칼라 기반. **스파크의 기본 언어**
  - Java : ~~자바 지원안해주면 난리칠거니까~~ 지원은 함
  - Python : 스칼라가 지원하는 거의 모든 구조 지원
  - SQL : ANSI SQL:2003 표준 중 일부 지원
  - R : 스파크 코어의 sparkR, R 커뮤니티 기반의 sparklyr
- SparkSession 객체
  - 사용자가 스파크 코드를 실행하기위해 진입점으로 사용 가능
  - Python, R 사용 시에도 사용자 대신 익스큐터의 JVM에서 실행할 수 있는 코드로 변환

### 2.3 스파크 API
- 다양한 언어로 사용할 수 있는 이유?
  - 스파크가 기본적으로 제공하는 2가지 API 때문
    - 저수준의 비구조적(unstructured) API
    - 고수준의 구조적(structured) API

### 2.4 스파크 시작하기
- Q. 스파크 애플리케이션을 개발하려면
  - A. 사용자 명령과 데이터를 스파크 애플리케이션에 전송하는 방법을 알아야
- SparkSession 생성 실습. 자 드가자~

### 2.5 SparkSession
- **SparkSession** : 스파크 애플리케이션을 제어하는 드라이버 프로세스
  - 사용자가 정의한 처리명령 -> 클러스터에 실행
  - 스파크 애플리케이션에 1:1 대응

```shell
# scala console
$ ./spark-2.4.7-bin-hadoop2.7/bin/spark-shell

scala> spark
res0: org.apache.spark.sql.SparkSession = org.apache.spark.sql.SparkSession@5b58f639
scala> val myRange = spark.range(1000).toDF("number")
myRange: org.apache.spark.sql.DataFrame = [number: bigint]
```

### 2.6 DataFrame
- **DataFrame** : 가장 대표적인 **구조적 API**
  - 테이블 데이터를 row, column 으로 단순하게 표현
    - scheme : column 과 column type 을 정의한 목록
  - DataFrame 은 수천 대의 컴퓨터에 분산 가능
  - vs 스프레드 시트
    - 비슷하다고 볼 수 있지만 스프레드 시트는 단일 컴퓨터 저장
  - vs Python (Pandas)의 DataFrame, R의 DataFrame
    - 마찬가지로 대부분 단일 컴퓨터에 존재
    - => 스파크 DataFrame으로 쉽게 변환 가능
- 스파크의 핵심 추상화 개념 (분산 데이터 모음)
  - Dataset, DataFrame, SQL 테이블, RDD
- DataFrame의 파티션
  - 익스큐터가 병렬로 작업을 수행할 수 있도록 데이터를 분할하는 청크 단위
  - 실행 중 데이터가 클러스터에서 물리적으로 분산되는 방식을 나타냄
    - 파티션 1 익스큐터 1000 => 병렬성 1
    - 파티션 1000 익스큐터 1 => 병렬성 1
  - 물리적 파티션에 데이터 변환용 함수 지정 시 스파크가 실제 처리 방법 결정 (파티션 수동 처리 필요 X)

### 2.7 트랜스포메이션
- 스파크의 핵심 데이터 구조 => **불변성 (immutable)**
  - DataFrame을 변경하려면?
  - 원하는 변경 방법을 스파크에게 알려줘야함 => **트랜스포메이션**
- 트랜스포메이션 : 스파크에서 비즈니스 로직을 표현하는 핵심 개념
  - 유형
    - 좁은 의존성 (narrow dependency)
      - 입력 파티션 : 출력 파티션 = 1 : 1
    - 넓은 의존성 (wide dependency)
      - 입력 파티션 : 출력 파티션 = 1 : N
- 지연 연산 (lazy evaluation) : 연산 그래프를 처리하기 직전까지 기다리는 동작 방식
  - 스파크는 연산 명령 즉시 데이터를 수정 X. 원시 데이터에 적용할 트랜스포메이션의 **실행 계획**을 생성
  - 마지막까지 대기하다 DataFrame 트랜스포메이션을 간결한 물리적 실행 계획으로 컴파일 => 전체 데이터 흐름 최적화
  - ex. DataFrame 의 predicate pushdown

### 2.8 액션
- 트랜스포메이션은 논리적 실행 계획
  - 트랜스포메이션을 선언해도 액션을 호출하지 않으면 수행 X
- 액션 (action) : 실제 연산을 수행
  - 유형
    - 콘솔에서 데이터를 보는 액션
    - 각 언어로 된 네이티브 객체에 데이터를 모으는 액션
    - 출력 데이터소스에 저장하는 액션
- 액션 지정 시 스파크 잡 시작
  - **스파크 잡 (job)**
    - 필터 (좁은 트랜스포메이션) 수행
    - -> 파티션 별로 레코드 수를 카운트 (넓은 트랜스포메이션)
    - -> 각 언어에 적합한 네이티브 객체에 결과 모음
  - 스파크 UI로 잡 모니터링 가능
  - *스파크 잡은 개별 액션에 의해 트리거되는 다수의 트랜스포메이션으로 이루어져 있다*

### 2.9 스파크 UI
- 드라이버 노드의 4040 포트
- 스파크 잡의 상태, 환경 설정, 클러스터 상태 등의 정보 확인 가능

### 2.10 종합 예제
- 미국 교통통계국의 항공운항 데이터 중 일부로 실습
  - [샘플 데이터](https://bit.ly/2yw2fCx) : 반정형(semi-structured), csv 포맷
  - (=> 부록 A의 도커 이미지 사용 시 이미 포함)
- 스파크는 다양한 데이터소스 지원
  - SparkSession의 DataFrameReader 클래스 사용해서 읽음
  - 예제는 **스키마 추론 (Schema inference)** 기능 추가
    - 스파크는 각 컬럼의 데이터 타입 추론을 위해 적은 양의 데이터를 읽음 
  - DataFrame 은 불특적 다수의 로우와 컬럼
    - 지연 연산 형태의 트렌스포메이션이므로 row 수 알 수 X

#### 예제 1

<details><summary class="point-color-can-hover">예제 1 펼치기</summary>

```bash

$ head /data/flight-data/csv/2015-summary.csv
DEST_COUNTRY_NAME,ORIGIN_COUNTRY_NAME,count
United States,Romania,15
United States,Croatia,1
..

# spark-shell (scala)
scala> val flightData2015 = spark.read.option("inferSchema", "true").option("header", "true").csv("/data/flight-data/csv/2015-summary.csv")
flightData2015: org.apache.spark.sql.DataFrame = [DEST_COUNTRY_NAME: string, ORIGIN_COUNTRY_NAME: string ... 1 more field]

scala> flightData2015.take(3)
res0: Array[org.apache.spark.sql.Row] = Array([United States,Romania,15], [United States,Croatia,1], [United States,Ireland,344])


scala> flightData2015.sort("count").explain()
== Physical Plan ==
*(2) Sort [count#12 ASC NULLS FIRST], true, 0
+- Exchange rangepartitioning(count#12 ASC NULLS FIRST, 200)
   +- *(1) FileScan csv [DEST_COUNTRY_NAME#10,ORIGIN_COUNTRY_NAME#11,count#12] Batched: false, Format: CSV, Location: InMemoryFileIndex[file:/data/flight-data/csv/2015-summary.csv], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<DEST_COUNTRY_NAME:string,ORIGIN_COUNTRY_NAME:string,count:int>


# 셔플 파티션 default 200개 => 5개
scala> spark.conf.set("spark.sql.shuffle.partitions", "5")

scala> flightData2015.sort("count").explain()
== Physical Plan ==
*(2) Sort [count#12 ASC NULLS FIRST], true, 0
+- Exchange rangepartitioning(count#12 ASC NULLS FIRST, 5)
   +- *(1) FileScan csv [DEST_COUNTRY_NAME#10,ORIGIN_COUNTRY_NAME#11,count#12] Batched: false, Format: CSV, Location: InMemoryFileIndex[file:/data/flight-data/csv/2015-summary.csv], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<DEST_COUNTRY_NAME:string,ORIGIN_COUNTRY_NAME:string,count:int>

scala> flightData2015.sort("count").take(2)
res3: Array[org.apache.spark.sql.Row] = Array([United States,Singapore,1], [Moldova,United States,1])

```

</details>

> - `take(n)` : Action
> - `sort()` :  Transformation (넓은) 
>   - DataFrame 을 변경하지 않고 새로운 DataFrame을 생성해 반환
> - `explain()` 
>   - DataFrame의 계보(lineage) 나 스파크 쿼리 실행 계획 출력

- 실행 계획? : 디버깅과 스파크의 실행과정을 이해하는데 도움을 주는 도구
  - 위에서 아래방향으로 읽는다
  - 최종 결과는 가장 위, 데이터소스는 가장 아래
- DataFrame의 계보
  - 트랜스포메이션의 논리적 실행 계획 -> DataFrame의 계보 정의
  - -> 계보를 통해 스파크가 입력데이터에 수행한 연산을 전체 파티션에서 어떻게 재연산하는지 알 수 있음
  - *함수형 프로그래밍의 핵심* (Pure Function, 같은 입력 -> 같은 출력)
- 사용자는 물리적 데이터를 직접 다루지 않고, 물리적 실행 특성을 제어
  - 스파크 UI (4040 포트) 에서 스파크 잡 물리적, 논리적 실행 특성 확인 가능 
<img width="500" alt="sparkui" src="https://user-images.githubusercontent.com/26691216/105624926-cfbfdf00-5e68-11eb-9407-e58a5f4688a9.png">


#### 예제 2 (SQL)

<details><summary class="point-color-can-hover">예제 2-1 펼치기</summary>

```bash
# 1) SQL 사용
scala> flightData2015.createOrReplaceTempView("flight_data_2015")
scala> val sqlWay = spark.sql("""
     | SELECT DEST_COUNTRY_NAME, count(1)
     | FROM flight_data_2015
     | GROUP BY DEST_COUNTRY_NAME
     | """)

scala> sqlWay.explain
== Physical Plan ==
*(2) HashAggregate(keys=[DEST_COUNTRY_NAME#10], functions=[count(1)])
+- Exchange hashpartitioning(DEST_COUNTRY_NAME#10, 5)
   +- *(1) HashAggregate(keys=[DEST_COUNTRY_NAME#10], functions=[partial_count(1)])
      +- *(1) FileScan csv [DEST_COUNTRY_NAME#10] Batched: false, Format: CSV, Location: InMemoryFileIndex[file:/data/flight-data/csv/2015-summary.csv], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<DEST_COUNTRY_NAME:string>


# 2) DataFrame 사용
scala> val dataFrameWay = flightData2015.groupBy("DEST_COUNTRY_NAME").count()
dataFrameWay: org.apache.spark.sql.DataFrame = [DEST_COUNTRY_NAME: string, count: bigint]

scala> dataFrameWay.explain
== Physical Plan ==
*(2) HashAggregate(keys=[DEST_COUNTRY_NAME#10], functions=[count(1)])
+- Exchange hashpartitioning(DEST_COUNTRY_NAME#10, 5)
   +- *(1) HashAggregate(keys=[DEST_COUNTRY_NAME#10], functions=[partial_count(1)])
      +- *(1) FileScan csv [DEST_COUNTRY_NAME#10] Batched: false, Format: CSV, Location: InMemoryFileIndex[file:/data/flight-data/csv/2015-summary.csv], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<DEST_COUNTRY_NAME:string>

```

</details>

- 스파크는 언어에 무관하게 같은 방식으로 트랜스포메이션 실행
  - SQL, DataFrame(R, Python, Scalar, Java) 에서 비즈니스 로직 표현
  - 스파크에서 코드 실행 전에 로직을 기본 실행계획(`explain`) 으로 컴파일
- 스파크 SQL 사용시 모든 DataFrame => 테이블, 뷰 (임시 테이블) 로 등록
  - 위에서 설명했듯 **같은 실행 계획**으로 컴파일하므로 성능차이 X

<details><summary class="point-color-can-hover">예제 2-2 펼치기</summary>

```bash
# '최대 비행 횟수' 구하기

# SQL 쿼리
scala> spark.sql("SELECT max(count) from flight_data_2015").take(1)
res9: Array[org.apache.spark.sql.Row] = Array([370002])

# DataFrame 구문 _ max 함수 (트랜스포메이션) 사용
scala> import org.apache.spark.sql.functions.max

scala> flightData2015.select(max("count")).take(1)
res10: Array[org.apache.spark.sql.Row] = Array([370002])

```

```bash
# '상위 5개의 도착 국가' 구하기

# SQL 쿼리
scala> val maxSql = spark.sql("""
     | SELECT DEST_COUNTRY_NAME, sum(count) as destination_total
     | FROM flight_data_2015
     | GROUP BY DEST_COUNTRY_NAME
     | ORDER BY sum(count) DESC
     | LIMIT 5
     | """)
maxSql: org.apache.spark.sql.DataFrame = [DEST_COUNTRY_NAME: string, destination_total: bigint]

scala> maxSql.show()
+-----------------+-----------------+
|DEST_COUNTRY_NAME|destination_total|
+-----------------+-----------------+
|    United States|           411352|
|           Canada|             8399|
|           Mexico|             7140|
|   United Kingdom|             2025|
|            Japan|             1548|
+-----------------+-----------------+


# DataFrame 구문
scala> import org.apache.spark.sql.functions.desc

scala> flightData2015.groupBy("DEST_COUNTRY_NAME").sum("count").withColumnRenamed("sum(count)", "destination_total").sort(desc("destination_total")).limit(5).show()
+-----------------+-----------------+
|DEST_COUNTRY_NAME|destination_total|
+-----------------+-----------------+
|    United States|           411352|
|           Canada|             8399|
|           Mexico|             7140|
|   United Kingdom|             2025|
|            Japan|             1548|
+-----------------+-----------------+

# 코드 수행 단계 : CSV 파일 => (1) read -> (2) groupBy -> (3) sum -> (4) withColumnRenamed -> (5) sort -> (6) limit -> (7) collect => Array(..)

# scala> ~.explain
== Physical Plan ==
TakeOrderedAndProject(limit=5, orderBy=[destination_total#108L DESC NULLS LAST], output=[DEST_COUNTRY_NAME#10,destination_total#108L])
+- *(2) HashAggregate(keys=[DEST_COUNTRY_NAME#10], functions=[sum(cast(count#12 as bigint))])
   +- Exchange hashpartitioning(DEST_COUNTRY_NAME#10, 5)
      +- *(1) HashAggregate(keys=[DEST_COUNTRY_NAME#10], functions=[partial_sum(cast(count#12 as bigint))])
         +- *(1) FileScan csv [DEST_COUNTRY_NAME#10,count#12] Batched: false, Format: CSV, Location: InMemoryFileIndex[file:/data/flight-data/csv/2015-summary.csv], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<DEST_COUNTRY_NAME:string,count:int>

```

</details>

- 실행계획은 트랜스포메이션의 **지향성 비순환 그래프 (Directed Acyclic Graph, DAG)**
  - 액션이 호출되면 결과를 만들어낸다
  - DAG의 각 단계는 불변성을 가진 신규 DataFrame을 생성
- 예제의 전체 코드 수행 단계 (7단계) 는 p.86 [그림 2-10] 참조
  - 실제 실행 계획 (`explain` 이 출력하는) 은 물리적인 실행 시점에서 수행하는 최적화로 인해 다를 수 있음
  - 직접 explain 해보면 책의 explain 과도 다르게 출력됨 


### 2.11 정리
- 트랜스포메이션, 액션, DataFrame 실행 계획 최적화 방법
  - 트랜스포메이션의 지향성 비순환 그래프(DAG) 를 지연 실행하여 최적화
- 예제를 통한 데이터가 파티션으로 구성되는 방법, 복잡한 트랜스포메이션 작업 실행 단계 확인

<br/>

### 📒 단어장
- 셔플 (Shuffle) : 스파크카 클러스터에서 파티션을 교환
  - 스파크는 셔플의 결과를 디스크에 저장
- 가환성 (Commutative) : 두 대상의 연산 결과가 순서와 관계없이 동일 (-> 교환 법칙)

