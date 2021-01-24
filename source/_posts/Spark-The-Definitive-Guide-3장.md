---
title: '&#039;Spark The Definitive Guide&#039; 3장 - 일단 좀 더 잡솨봐 (PART 1 끝)'
date: 2021-01-24 23:13:51
categories: spark
tags:
	- spark
	- apache
	- book
	- study
---

<br/>

#### [Part 1] END

파트 1 까지 끝내고 나니 이제 어찌저찌 조금씩 맛은 본 것 같은데, 이번 장은 따라가기 살짝 힘들었다.

> 나 : 뭔 말이에요
> ?? : *'XX 장에서 자세히 알아보겠습니다.'*
>
> 나 : 이건 또 뭐여
> ?? : *'이와 관련된 내용은 XX 부에서 자세히 알아보겠습니다.'*

<img width="150" alt="angry" src="https://user-images.githubusercontent.com/26691216/105637781-85fbe680-5eb2-11eb-8956-b7de17f122de.png
">
<center><i style="color:lightgray">이toRL들이..</i></center>



예제 위주로 모르는 부분도 찾아가며 겨우 이해는 했지만
그 다음 파트는 또 `구조적 API`라 오늘 본 거 대부분은 한참 뒤에야 다시 볼텐데..
아 이거 백프로 다 까먹을텐디... 킹받네... 🤦🏻‍♀️  

<center><h2>_ _ _</h2></center>

<br/>

---


# CHAPTER 3 스파크 기능 둘러보기
> 스파크 = 기본 요소 (저수준 API + 구조적 API) + 추가 기능 (일련의 표준 라이브러리)
> 
> - 구조적 스트리밍, 고급 분석, 라이브러리 및 에코시스템
> - 구조적 API : Dataset, DataFrame, SQL
> - 저수준 API : RDD, 분산형 변수

CHAPTER 2 는 구조적 API의 핵심개념을 소개했다면
CHAPTER 3 은 나머지 API 와 주요 라이브러리, 스파크의 다양한 기능 소개

### 3.1 운영용 애플리케이션 실행하기

<details><summary class="point-color-can-hover">[3.1] 예제 펼치기</summary>

```bash
# /spark-2.4.7-bin-hadoop2.7

# scalar example
$ ./bin/spark-submit --class org.apache.spark.examples.SparkPi --master local ./examples/jars/spark-examples_2.11-2.4.7.jar 10
...
21/01/24 13:41:15 INFO DAGScheduler: Job 0 finished: reduce at SparkPi.scala:38, took 0.968079 s
Pi is roughly 3.1414071414071416 # 돌릴때마다 다르게 나온다

# python example
$ ./bin/spark-submit --master local ./examples/src/main/python/pi.py 10
Pi is roughly 3.139084 # 이럴거면 args 는 대체 왜 넣으란걸까
```

</details>

- `spark-submit` 명령
  - 애플리케이션 코드를 클러스터에 전송해 실행시키는 역할
  - 대화형 쉘에서 개발한 프로그램 -> 운영용 애플리케이션으로 전환 가능
  - 스파크 애플리케이션은 standalone, Mesos, YARN 클러스터 매니저를 이용해 실행됨 (`--master` 옵션)


### 3.2 Dataset : 타입 안정성을 제공하는 구조적 API

<details><summary class="point-color-can-hover">[3.2] 예제 펼치기</summary>

```scala
import spark.implicits._

case class Flight(DEST_COUNTRY_NAME: String,
                  ORIGIN_COUNTRY_NAME: String,
                  count: BigInt)

val flightsDF = spark.read.parquet("/data/flight-data/parquet/2010-summary.parquet/")
val flights = flightsDF.as[Flight] // DataFrame -> Dataset[Flight]


(flights
  .filter(flight_row => flight_row.ORIGIN_COUNTRY_NAME != "Canada")
  .map(flight_row => flight_row)
  .take(5))

(flights
  .take(5)
  .filter(flight_row => flight_row.ORIGIN_COUNTRY_NAME != "Canada")
  .map(fr => Flight(fr.DEST_COUNTRY_NAME, fr.ORIGIN_COUNTRY_NAME, fr.count + 5)))

// res45: Array[Flight] = Array(Flight(United States,Romania,1), Flight(United States,Ireland,264), Flight(United States,India,69), Flight(Egypt,United States,24), Flight(Equatorial Guinea,United States,1))
// res46: Array[Flight] = Array(Flight(United States,Romania,6), Flight(United States,Ireland,269), Flight(United States,India,74), Flight(Egypt,United States,29), Flight(Equatorial Guinea,United States,6))
```

</details>

- **Dataset** : Java와 Scala의 정적 데이터 타입에 맞는 코드(statically typed code)를 지원하기 위한 스파크의 구조적 API
  - Python, R 사용 X
- Dataset API 는 **DataFrame 레코드 => Java나 Scala로 정의한 클래스에 할당**, Collection 으로 다룰 수 있는 기능 등을 제공
  - DataFrame : 다양한 데이터 타입의 테이블형 데이터를 보관할 수 있는 Row 타입 객체로 구성된 분산 컬렉션 ([2장 참고](https://minsw.github.io/2021/01/24/Spark-The-Definitive-Guide-2%EC%9E%A5/#2-6-DataFrame))
  - **타입 안정성을 지원** 하므로 초기화에 사용한 클래스 외 다른 클래스를 사용한 접근은 X
  - 여러 명이 개발하고 잘 정의된 인터페이스로 상호작용하는 대규모 애플리케이션 개발에 유용
    ~~잘 정의된 인터페이스 부터가 실패다 이말이야~~
- Dataset 클래스 (Java `Dataset<T>`, Scala `Dataset[T]`)
  - 내부 객체 타입을 매개변수로 사용 (T) => 해당 클래스 객체만 가질 수 있음
  - 스파크 2.0 에서는 자바의 JavaBean 패턴, 스칼라의 케이스 클래스 유형으로 정의된 클래스 지원
  - 타입 T를 분석해서 Dataset 스키마를 생성해야하므로 타입을 제한할 수 밖에 없음
- 장점
  - 필요한 경우 선택적으로 사용 가능하고, map, filter 등 함수 사용 가능
  - 코드 변경 없이 타입 안정성을 보장할 수 있고, 안전하게 데이터 다루기 가능
    - `collect()` 나 `take()` 호출 시 DataFrame의 row 타입 객체가 아닌 Dataset의 지정된 타입(T)의 객체로 반환
- Dataset의 자세한 내용은 CHAPTER 11 에서 이어서


### 3.3 구조적 스트리밍



<details><summary class="point-color-can-hover">[3.3] 예제 펼치기 (정적 DataFrame 버전) </summary>

```scala
// 정적 DataFrame 버전
val staticDataFrame = (spark.read.format("csv")
  .option("header", "true")
  .option("inferSchema", "true")
  .load("/data/retail-data/by-day/*.csv"))

staticDataFrame.createOrReplaceTempView("retail_data")
val staticSchema = staticDataFrame.schema


import org.apache.spark.sql.functions.{window, column, desc, col}

// '특정 고객(CustomerId)가 대량으로 구매하는 영업 시간' 구하기
(staticDataFrame
  .selectExpr(
    "CustomerId",
    "(UnitPrice * Quantity) as total_cost",
    "InvoiceDate")
  .groupBy(
    col("CustomerId"), window(col("InvoiceDate"), "1 day")) // 관련 날짜 데이터 그룹화 & 집계
  .sum("total_cost")
  .show(5))

// +----------+--------------------+------------------+
// |CustomerId|              window|   sum(total_cost)|
// +----------+--------------------+------------------+
// |   14075.0|[2011-12-05 00:00...|316.78000000000003|
// |   18180.0|[2011-12-05 00:00...|            310.73|
// |   15358.0|[2011-12-05 00:00...| 830.0600000000003|
// |   15392.0|[2011-12-05 00:00...|304.40999999999997|
// |   15290.0|[2011-12-05 00:00...|263.02000000000004|
// +----------+--------------------+------------------+

```

> 로컬 모드 사용 시 셔플 파티션 수 (default 200) 줄이기를 권장. `spark.conf.set("spark.sql.shuffle.partitions", "5")`

</details>

<br/>

<details><summary class="point-color-can-hover">[3.3] 예제 펼치기 (Streaming 버전) </summary>


```scala
// Streaming 버전
val streamingDataFrame = (spark.readStream  // read => readStream
    .schema(staticSchema)
    .option("maxFilesPerTrigger", 1)  // maxFilesPerTrigger (한번에 읽을 파일 수 설정) => 파일별로 트리거 수행
    .format("csv")
    .option("header", "true")
    .load("/data/retail-data/by-day/*.csv"))

streamingDataFrame.isStreaming // returns true


val purchaseByCustomerPerHour = (streamingDataFrame
  .selectExpr(
    "CustomerId",
    "(UnitPrice * Quantity) as total_cost",
    "InvoiceDate")
  .groupBy(
    $"CustomerId", window($"InvoiceDate", "1 day"))
  .sum("total_cost"))

// 1) 스트림 시작 & 인메모리 테이블에 저장
(purchaseByCustomerPerHour.writeStream
    .format("memory") // memory = 인메모리 테이블에 저장
    .queryName("customer_purchases") // 인메모리에 저장될 테이블명
    .outputMode("complete") // complete = 모든 카운트 수행 결과를 테이블에 저장
    .start())

// 인메모리 테이블 확인 (데이터를 많이 읽으면 읽을수록 테이블 구성이 변경)
(spark.sql("""
  SELECT *
  FROM customer_purchases
  ORDER BY `sum(total_cost)` DESC
  """)
  .show(5))

// +----------+--------------------+------------------+
// |CustomerId|              window|   sum(total_cost)|
// +----------+--------------------+------------------+
// |   17450.0|[2011-09-20 00:00...|          71601.44|
// |      null|[2011-11-14 00:00...|          55316.08|
// |      null|[2011-11-07 00:00...|          42939.17|
// |      null|[2011-03-29 00:00...| 33521.39999999998|
// |      null|[2011-12-08 00:00...|31975.590000000007|
// +----------+--------------------+------------------+


// 2) 처리결과 콘솔에 출력
(purchaseByCustomerPerHour.writeStream
    .format("console") // console = 콘솔에 결과 출력
    .queryName("customer_purchases_2")
    .outputMode("complete")
    .start())

```

</details>

- **구조적 스트리밍** : 스트림 처리용 고수준 API
  - 구조적 API로 개발된 배치 모드 연산을 **스트리밍 방식으로** 실행 가능하며, 지연 시간을 줄이고 증분 처리 가능
  - 즉 스트리밍 처리로 <u>빠르게 값을 얻을 수 있고</u>, 모든 작업에서 데이터를 <u>증분 처리</u>하면서 수행된다
  - 배치 잡으로 프로토타입 개발 후에 스트리밍 잡으로 변환도 가능
  - 스파크 2.2 버전부터 안정화 (production-ready)
- 데이터를 그룹화하고 집계하는 방법 (시계열 time-series 데이터  처리)
  - `window()` : 집계 시에, 시계열 컬럼 기준으로 각 날짜에 대한 전체 데이터를 가지는 윈도우 구성 => 날짜, 타임스탬프 처리에 유용
- 정적 DataFrame 코드 vs 스트리밍 코드
  - `read` vs `readStream`
  -  일반적인 정적 액션 vs **스트리밍 액션**
  - 스트리밍 액션은 어딘가에 데이터를 채워넣어야함. **트리거**가 실행된 후 데이터를 갱신
    - (인메모리 테이블에 저장 시 - 스파크는 이전 집계값보다 더 큰 값이 발생할 때만 인메모리 테이블 갱신)
- [예제 retail 데이터 셋](http://bit.ly/2PvOwCS)
  - by-day 하루 치 데이터 사용
  - 예제는 인메모리 테이블에 저장 / 파일마다 트리거 실행
  - 예제의 두가지 방식 (메모리/콘솔 출력, 파일별 트리거 수행)은 운영 환경에서는 권장 X
- 데이터 처리 시점이 아닌 이벤트 시간에 따라 윈도우를 구성하는 방식에 주목
  - 기존 스파크 스트리밍의 단점 => **구조적 스트리밍으로 보완** 가능
  - 스트림 처리과정의 스키마 추론방법 및 구조적 스트리밍은 CHAPTER 5 에서 자세히


### 3.4 머신러닝과 고급 분석

<details><summary class="point-color-can-hover">[3.4] 예제 펼치기 </summary>


```bash
# MLlib 머신러닝 알고리즘 : 수치형 데이터 필요
# 예제의 (정적) 데이터 => 수치형으로 변환

staticDataFrame.printSchema()

root
 |-- InvoiceNo: string (nullable = true)
 |-- StockCode: string (nullable = true)
 |-- Description: string (nullable = true)
 |-- Quantity: integer (nullable = true)
 |-- InvoiceDate: timestamp (nullable = true)
 |-- UnitPrice: double (nullable = true)
 |-- CustomerID: double (nullable = true)
 |-- Country: string (nullable = true)
```

```scala
import org.apache.spark.sql.functions.date_format
val preppedDataFrame = (staticDataFrame
  .na.fill(0) // 0인 경우 null로 채움
  .withColumn("day_of_week", date_format($"InvoiceDate", "EEEE")) // Sunday, Monday, ..
  .coalesce(5)) // 파티션 개수 줄임 (default, shuffle = false)

// (1) 데이터 => 학습 데이터셋, 테스트 데이터셋으로 분리 (2011-07-01 기준)
val trainDataFrame = (preppedDataFrame
  .where("InvoiceDate < '2011-07-01'"))
val testDataFrame = (preppedDataFrame
  .where("InvoiceDate >= '2011-07-01'"))

trainDataFrame.count()
// res110: Long = 245903
testDataFrame.count()   
// res111: Long = 296006


// (2-1) 요일(Sunday, Monday,..)을 수치형(0,1, ..)으로 반환
import org.apache.spark.ml.feature.StringIndexer
val indexer = (new StringIndexer()
  .setInputCol("day_of_week")
  .setOutputCol("day_of_week_index"))

// (2-2) 숫자로 표현된 범주형 데이터 인코딩 (해당 요일인지 Boolean 타입으로 확인 가능)
import org.apache.spark.ml.feature.OneHotEncoder
val encoder = (new OneHotEncoder()
  .setInputCol("day_of_week_index")
  .setOutputCol("day_of_week_encoded"))

// (2-3) 수치형 벡터 타입을 입력으로 사용 (가격, 수량, 특정 날짜의 요일)
import org.apache.spark.ml.feature.VectorAssembler
val vectorAssembler = (new VectorAssembler()
  .setInputCols(Array("UnitPrice", "Quantity", "day_of_week_encoded"))
  .setOutputCol("features"))

// (3) 파이프라인 설정
import org.apache.spark.ml.Pipeline
val transformationPipeline = (new Pipeline()
  .setStages(Array(indexer, encoder, vectorAssembler)))

// (4) 변환자(transformer) 를 데이터셋에 적합(fit) => 'fitted pipeline'
// 일관되고 반복된 방식으로 데이터 변환 가능. 학습 데이터셋 생성 완료
val fittedPipeline = transformationPipeline.fit(trainDataFrame)

val transformedTraining = fittedPipeline.transform(trainDataFrame)

// 캐싱 사용시 중간 변화된 데이터셋의 복사본을 메모리에 저장. 전체 파이프라인 재실행보다 훨씬 빠르다
// 근데 왜때문에 나는 더 느린 것..? ㅎ.. CHAPTER 4 에서 다시 확인하자
transformedTraining.cache()


// (5) 모델 학습 (kmeans 모델 설정 과정은 생략..)
import org.apache.spark.ml.clustering.KMeans
val kmeans = (new KMeans()
  .setK(20)
  .setSeed(1L))

val kmModel = kmeans.fit(transformedTraining)

// (6) 학습 데이터셋에 대한 비용 (군집 비용) 계산
kmModel.computeCost(transformedTraining)
// res146: Double = 8.455373996537486E7

// 테스트 데이터셋과 비교
// 모델 개선 방법은 CHAPTER 6 에서
val transformedTest = fittedPipeline.transform(testDataFrame)
kmModel.computeCost(transformedTest)
// res150: Double = 5.175070947222117E8

```

</details>

- 내장된 머신러닝 알고리즘 라이브러리 MLlib 사용한 대규모 머신러닝 가능
  - 대용량 데이터 대상의 전처리(proprocessing), 멍잉(munging), 모델 학습(model training), 예측(prediction)
  - 구조적 스트리밍에서 예측하고자 할때도 예측 모델 사용 가능
- 스파크는 분류(classification), 회귀(regression), 군집화(clustering), 딥러닝(deep learning) 같은 머신러닝 관련 정교한 API 제공
  - 두유 노- `k-평균` ? : 군집화 표준 알고리즘. 센트로이드(centroid)라는 중심점을 사용해서.. `p.99 참고`
- k-평균을 사용한 예제
  - 원본 데이터를 올바른 포맷으로 만드는 트렌스포메이션 정의. 실제 모델 학습 후 다음 예측 수행

- 스파크 (MLlib DataFrame API) 에서 머신러닝 모델 학습 과정 2단계
  1. 아직 학습되지 않은 모델 초기화
  2. 해당 모델을 학습

> 알고리즘 명명 규칙 
> - 학습 전 알고리즘 명칭 : {Algorithm_name}
> - 학습 후 알고리즘 명칭 : {Algorithm_name} + 'Model'


### 3.5 저수준 API
- 스파크는 **RDD** 를 통해 자바와 파이썬 객체를 다루는데 필요한 다양한 기본 기능 (저수준 API) 제공
  - DataFrame을 포함해서 스파크의 거의 모든 기능이 RDD 기반
  - 저수준 명령으로 컴파일 => 편리하고 매우 효율적인 분산처리
- 원시 데이터를 다루는 용도로도 쓸 수는 있지만, 대부분 구조적 API 사용이 더 낫다
  - 대신 파티션과 같은 **물리적 실행 특성을 결정** 할 수 있어, 세밀한 제어가 가능
  - 비정형 데이터, 정제되지 않은 원시 데이터 처리에 사용
- 언어에 따라 RDD 세부 구현에 차이가 있음
  - Scala, Python 모두 사용 가능하지만 RDD가 동일하지 X
  - (<-> 언어에 관계없이 동일한 실행 특성의 DataFrame API)

```scala
// 메모리에 저장된 원시 데이터를 병렬 처리 (parallize) 하여 RDD[Int] 생성 후 DataFrame으로 변환
spark.sparkContext.parallelize(Seq(1, 2, 3)).toDF()
```

### 3.6 SparkR
- SparkR : 스파크를 R언어로 사용하기 위한 기능
  - 파이썬 API 와 유사하고, 파이썬에서 사용할 수 있는 기능은 대부분 사용 가능
  - R 라이브러리 사용하여 스파크 트랜스포메이션 과정을 R과 유사하게 만들 수 있음
  - CHAPTER 7에서 자세히 알아보자

### 3.7 스파크의 에코시스템과 패키지
- 스파크의 최고 자랑 = 커뮤니티가 만들어낸 패키지 에코시스템 & 다양한 기능
  - 스파크 패키지 저장소 : https://spark-packages.org/
  - 그 외 깃헙, 기타 웹사이트 ...

### 3.8 정리
- 스파크를 비즈니스와 기술적 문제 해결에 적용할 수 있는 다양한 방법
  - 단순하고 강력한 프로그래밍 모델, 손쉬운 적용
  - 다양한 패키지는 여러 비즈니스 문제를 성공적으로 해결할 수 있는 스파크의 능력에 대한 증거
  - 더 성장하도록 더 많은 패키지가 만들어질거다~


### 📒 단어장
- 정적 타입 코드/언어 (Statically typed) : 자료형이 고정된 언어. 컴파일 때 변수 타입이 결정 (ex. Java, Scala, C, C++ 등)
  - <-> 동적 타입 언어 (Dynamically typed) : 런타임에 변수 타입이 결정 (ex. Python, JavaScript 등)
- 멍잉 (munging) : =data wrangling. 원본 데이터를 다른 형태로 변환하거나 매핑하는 과정
