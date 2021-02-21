---
title: '&#039;Spark The Definitive Guide&#039; 9장 - 쏘쓰는 역시 데이터소스'
date: 2021-02-16 00:11:36
categories: spark
tags:
  - spark
  - apache
  - book
  - study
---

<br/>

<img src="https://user-images.githubusercontent.com/26691216/108165351-c8bd8100-7135-11eb-9cbe-6ccfa0e63155.gif" width=400/>

<center><h2>_ _ _</h2></center>

<br/>

---

# CHAPTER 9 데이터소스
스파크 기본 6가지 '핵심' 데이터 소스 + 커뮤니티에서 만든 외부 데이터소스 소개

핵심데이터 소스를 통해 데이터를 읽고 쓰는 방법을 터득하고
서드파티 데이터소스와 스파크 연동 시 고려해야할 점을 배우는 것이 목표

#### 스파크의 핵심 데이터 소스
- CSV
- JSON
- 파케이(Parquet)
- ORC
- JDBC/ODBC 연결
- 일반 텍스트 파일

#### 커뮤니티에서 만든 데이터소스
- [카산드라](http://bit.ly/2DSafT8)
- [HBase](http://bit.ly/2FkKN5A)
- [몽고DB](http://bit.ly/2BwA7yq)
- [AWS Redshift](http://bit.ly/2GlMsJE)
- [XML](http://bit.ly/2GitGCK)
- 기타 수많은 데이터 소스

### 9.1 데이터소스 API의 구조
- 데이터 소스 API 전체 구조부터 이해하기
- ***읽기 API*** 구조
  - 핵심 구조 (모든 데이터 소스를 읽을 때 해당 형식 사용)  <i style="color:lightgray">// 요약 표기법도 존재</i>
    ```scala
    DataFrameReader.format(...).option("key", "value").schema(...).load()
    ```
  - `format()` : 포맷 설정은 Optional (default - Parquet 포멧)
  - `option()` : 데이터 읽는 방법에 대한 파라미터 키-값 쌍으로 설정
  - `schema()` : 데이터 소스에서 스키마를 제공하거나 추론 기능 사용 시. Optional
- 데이터 읽기의 기초
  - `DataFrameReader` : 스파크에서 데이터를 읽을 때 기본적으로 사용
    ```scala
    // DataFrameReader은 SparkSession의 read 속성으로 접근
    spark.read
    ```
  - DataFrameReader에 지정해야하는 값
    - 포맷
    - 스키마
    - 읽기 모드 (필수, default 값 존재)
    - 옵션
    - (+ **데이터 읽을 경로** 필수 지정)
  - **읽기 모드** : 스파크가 형식에 맞지않는 데이터를 만났을 때 동작 방식 지정하는 옵션
    - 반정형 데이터소스 다룰 시 많이 발생
  - 스파크의 읽기 모드 종류
    - `permissive` (default): 오류 레코드 모든 필드를 null로 지정하고 오류 레코드를 _corrupt_record (문자열 컬럼) 에 기록
    - `dropMalformed` : 형식에 맞지않는 레코드가 포함된 로우 제거
    - `failFast` : 형식에 맞지않는 레코드 만날 시 즉시 종료
    ```scala
    // 읽기 코드 구성 예제
    spark.read.format("csv")
      .option("mode", "FAILFAST")
      .option("inferSchema", "true")
      .option("path", "path/to/file(s)")
      .schema(someSchema)
      .load()
    ```
- ***쓰기 API*** 구조
  - 핵심 구조 (모든 데이터 소스를 읽을 때 해당 형식 사용)
    ```scala
    DataFrameWriter.format(...).option(...).partitionBy(...).bucketBy(...).sortBy(...).save()
    ```
  - `format()` : 포맷 설정은 Optional (default - Parquet 포멧)
  - `option()` : 데이터 쓰기 방법 설정
  - `partitionBy()`, `bucketBy()`, `sortBy()` : 최종 파일 배치 형태(layout) 제어 가능. 파일기반 데이터소스에만 동작
- 데이터 쓰기의 기초
  - 데이터 읽기와 매우 유사. Reader대신 Writer 사용
  - `DataFrameWriter` : 데이터 소스에 항상 데이터를 기록해야하고, DataFrame 별로 DataFramewriter에 접근해야함
    ```scala
    // DataFrame 의 write 속성을 이용해서 DataFrameWriter에 접근
    dataFrame.write
    ```
  - DataFrameWriter에 지정해야하는 값
    - 포맷
    - 옵션
    - 저장 모드
    - (+ **데이터 저장 경로** 필수 지정)
  - **저장 모드** : 스파크가 지정된 위치에서 동일한 파일을 발견했을 때 동작 방식 지정하는 옵션
  - 스파크의 저장 모드 종류
    - `append` : 해당 경로에 이미 존재하는 파일 목록에 결과 파일 추가
    - `overwrite` : 이미 존재하는 모든 데이터 덮어쓰기
    - `errorIfExists` (default) : 해당 경로에 데이터나 파일이 존재하면 오류 발생 및 쓰기 작업 실패
    - `ignore` : 아무런 처리 안함

    ```scala
    // 쓰기 코드 구성 예제
    spark.write.format("csv")
      .option("mode", "OVERWRITE")
      .option("dateFormat", "yyyy-MM-dd")
      .option("path", "path/to/file(s)")
      .save()
    ```


### 9.2 CSV 파일

- CSV(comma-separated values) : `,` 로 구분된 값
  - 각 줄이 단일 레코드, 레코드의 각 필드는 콤마로 구분하는 텍스트 파일 포멧
  - 구조적인 것 같아도 개 까다로운 포맷 (다양한 전제 생성 가능...)
  - => 따라서 CSV Reader 가 **많은 옵션** 제공 
- 옵션
  - CSV Reader, Writer 많은 옵션 제공 (`p.250-251 [표 9-3]` 참고)
  - maxColumns, inferSchema 등 쓰기에서는 적용되지 않는 옵션 빼고는 **읽기와 쓰기는 동일한 옵션** 제공
- CSV 파일 읽기
  - 예제 참고
    ```scala
    spark.read.format("csv")
    // in Scala
    /*
    (spark.read.format("csv")
      .option("header", "true")
      .option("mode", "FAILFAST")
      .option("inferSchema", "true")
      .load("some/path/to/file.csv"))
    */
    ```
  - 데이터가 기대한 데이터 포맷이 아닌경우?
    - **실제** 스키마와는 일치하지 않지만 스파크는 문제 인지 X
    - 스파크가 실제로 데이터를 읽어 들이는 시점에 문제 발생 (스파크 잡 즉시 종료)
    - 즉, 정의하는 시점에는 문제 X. 잡 실행 시점에만 오류 발생  => <u>스파크의 **지연 연산** 특성</u>
- CSV 파일 쓰기
  - 예제 참고

<details><summary class="point-color-can-hover">[9.2] CSV 파일 read 예제 펼치기</summary>

```scala
// CSV 파일 읽기
import org.apache.spark.sql.types.{StructField, StructType, StringType, LongType}

val myManualSchema = new StructType(Array(
  new StructField("DEST_COUNTRY_NAME", StringType, true),
  new StructField("ORIGIN_COUNTRY_NAME", StringType, true),
  new StructField("count", LongType, false)
))

(spark.read.format("csv")
  .option("header", "true")
  .option("mode", "FAILFAST")
  .schema(myManualSchema)
  .load("/data/flight-data/csv/2010-summary.csv")
  .show(5))
// +-----------------+-------------------+-----+
// |DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|
// +-----------------+-------------------+-----+
// |    United States|            Romania|    1|
// |    United States|            Ireland|  264|
// |    United States|              India|   69|
// |            Egypt|      United States|   24|
// |Equatorial Guinea|      United States|    1|
// +-----------------+-------------------+-----+


// 기대한 데이터 포맷이 아니라면?
// => 당장 에러 발생은 X. 스파크가 실제로 데이터를 읽어들이는 시점에 에러 발생 (지연 연산)
val myManualSchema = new StructType(Array(
                     new StructField("DEST_COUNTRY_NAME", LongType, true),
                     new StructField("ORIGIN_COUNTRY_NAME", LongType, true),
                     new StructField("count", LongType, false) ))

(spark.read.format("csv")
  .option("header", "true")
  .option("mode", "FAILFAST")
  .schema(myManualSchema)
  .load("/data/flight-data/csv/2010-summary.csv")
  .take(5))
// org.apache.spark.SparkException: Job aborted due to stage failure: Task 0 in stage 47.0 failed 1 times, most recent failure: Lost task 0.0 in stage 47.0 (TID 107, localhost, executor driver): org.apache.spark.SparkException: Malformed records are detected in record parsing. Parse Mode: FAILFAST.

// (참고용) 정상 스키마
// val myManualSchema = new StructType(Array(
//                      new StructField("DEST_COUNTRY_NAME", StringType, true),
//                      new StructField("ORIGIN_COUNTRY_NAME", StringType, true),
//                      new StructField("count", LongType, false) ))
//
// => res122: Array[org.apache.spark.sql.Row] = Array([United States,Romania,1], [United States,Ireland,264], [United States,India,69], [Egypt,United States,24], [Equatorial Guinea,United States,1])
```

</details>
<details><summary class="point-color-can-hover">[9.2] CSV 파일 write 예제 펼치기</summary>

```scala
// CSV 파일 쓰기 (CSV 파일 읽어서 TSV 파일로 내보내기)
val csvFile = (spark.read.format("csv")
  .option("header", "true").option("mode", "FAILFAST").schema(myManualSchema)
  .load("/data/flight-data/csv/2010-summary.csv"))

(csvFile.write.format("csv").mode("overwrite").option("sep", "\t")
  .save("/tmp/my-tsv-file.tsv"))
```

```bash
# 데이터를 쓰느 시점에 DataFrame의 파티션 수를 반영
$ ls /tmp/my-tsv-file.tsv/
part-00000-183b90fb-5828-434a-b948-55dd5732c7b0-c000.csv  _SUCCESS
```

</details>

### 9.3 JSON 파일

- JSON(JavaScript Object Notation)
  - 스파크는 **줄로 구분된 JSON** 을 기본적으로 사용
    - multiLine 옵션으로 줄로 구분 vs 여러 줄로 구성된 방식 선택 가능
    - `true` 로 설정 시 => 전체 파일을 하나의 JSON 파일로 읽기 가능
  - 그래도 줄로 구분된 JSON을 추천하는 이유?
    - 전체 파일을 읽어서 저장하는 방식이 아니므로 => 새로운 레코드 추가 가능 (안정적)
    - 구조화되어 있고, 최소한의 기본 데이터 타입이 존재 => 적합한 데이터타입 추정 가능
- 옵션
  - JSON은 객체. CSV(텍스트) 보다 옵션수 적음 (`p.255-256 [표 9-4]` 참고)
- JSON 파일 읽기
  - 예제 참고
    ```scala
    spark.read.format("json")
    ```
- JSON 파일 쓰기
  - 예제 참고
  - 데이터소스와 관계없이 JSON 파일로 저장 가능
    - ex. CSV DataFrame => JSON 파일
    - 이전의 규칙을 그대로 따른다? (예제이야기인지?)
    - 파티션당 하나의 파일을 만들고, 전체 DataFrame을 단일 폴더에 저장. JSON 객체는 한줄에 하나씩 기록.

<details><summary class="point-color-can-hover">[9.3] JSON 파일 read/write 예제 펼치기</summary>

```scala
// JSON 파일 읽기
// spark.read.format("json")
(spark.read.format("json").option("mode", "FAILFAST").schema(myManualSchema)
  .load("/data/flight-data/json/2010-summary.json").show(5))
// +-----------------+-------------------+-----+
// |DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|
// +-----------------+-------------------+-----+
// |    United States|            Romania|    1|
// |    United States|            Ireland|  264|
// |    United States|              India|   69|
// |            Egypt|      United States|   24|
// |Equatorial Guinea|      United States|    1|
// +-----------------+-------------------+-----+


// JSON 파일 쓰기 (CSV DataFrame => JSON 파일)
csvFile.write.format("json").mode("overwrite").save("/tmp/my-json-file.json")
```

```bash
# 파티션당 하나의 파일 만들고 전체 DataFrame은 단일폴더에 저장
$ ls /tmp/my-json-file.json/
part-00000-8a5f3d0c-2241-4508-ab3f-e2648f9a5ff4-c000.json  _SUCCESS

# JSON 객체는 한줄에 하나씩 기록
$ head -2 /tmp/my-json-file.json/part-00000-8a5f3d0c-2241-4508-ab3f-e2648f9a5ff4-c000.json
{"DEST_COUNTRY_NAME":"United States","ORIGIN_COUNTRY_NAME":"Romania","count":1}
{"DEST_COUNTRY_NAME":"United States","ORIGIN_COUNTRY_NAME":"Ireland","count":264}
```

</details>

### 9.4 파케이 파일
- 파케이(Parquet) : 다양한 스토리지 최적화 기술을 제공하는 오픈소스로 만들어진 **컬럼 기반의 데이터 저장 방식**
  - 분석 워크로드에 최적화
  - 저장소 공간 절약
  - 전체 파일 읽기 대신 개별 컬럼 읽기 가능
  - 컬럼 기반의 압축 기능 제공
  - 아파치 스파크와 특히 호환 good => 그래서 **스파크 기본 파일 포멧**
  - 복합 데이터 타입 지원 (CSV는 배열 사용 X)
- 읽기 연산이 CSV, JSON보다 훨씬 효율적 => 장기저장용 데이터는 파케이 권장
  - <a style="color:lightgray">걍 파케이가 짱짱맨이란 소리다</a>
- 옵션
  - 파케이는 옵션이 거의 없음. 단 2개  (`p.259 [표 9-5]` 참고)
    - 2개만 존재하는 이유는.. 그냥 모범생 포맷이기때문... (자체 스키마 사용해서 데이터 저장)
    - 그러나 '호환되지 않는 파케이 파일' 주의 => 트기 <u>다른 버전(구버전)의 스파크 사용 시 파케이 저장</u> 에 주의
- 파케이 파일 읽기
  - 예제 참고
    ```scala
    spark.read.format("parquet")
    ```
  - 포맷 설정만으로 충분
    - DataFrame 표현을 위해 정확한 스키마가 필요할 때만 스키마 지정 
    - 그렇지만 사실 거의 필요 X
  - 파케이 파일은 스키마가 파일 자체에 내장되어 추론 필요 X
    - 읽는 시점에 스키마를 알 수 있다 (Schema-on-read)
    - CSV 파일 inferSchema랑 비슷
- 파케이 파일 쓰기
  - 예제 참고
  - "읽기만큼 쉽다" => 파일의 경로만 명시하면 됨
    - 분할 규칙은 다른 포맷과 동일하게 적용

<details><summary class="point-color-can-hover">[9.4] Parquet 파일 read/write 예제 펼치기</summary>

```scala
// Parquet 파일 읽기
// spark.read.format("parquet")
(spark.read.format("parquet")
  .load("/data/flight-data/parquet/2010-summary.parquet").show(5))
// +-----------------+-------------------+-----+
// |DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|
// +-----------------+-------------------+-----+
// |    United States|            Romania|    1|
// |    United States|            Ireland|  264|
// |    United States|              India|   69|
// |            Egypt|      United States|   24|
// |Equatorial Guinea|      United States|    1|
// +-----------------+-------------------+-----+


// Parquet 파일 쓰기
(csvFile.write.format("parquet").mode("overwrite")
  .save("/tmp/my-parquet-file.parquet"))
```

```bash
# 다른 포멧과 동일한 분할 규칙
$ ls /tmp/my-parquet-file.parquet/
part-00000-7275ca33-21c1-4ce1-8e4f-93f9918a938d-c000.snappy.parquet  _SUCCESS
```

</details>

### 9.5 ORC 파일
- ORC(Optimized Row Columnar) : 하둡 워크로드를 위해 설계된 자기 기술적(self-describing)이며 데이터 타입을 인식할 수 있는 **컬럼 기반의 파일 포맷**
  - 대규모 스트리밍 읽기에 최적화
  - 필요한 로우를 신속하게 찾을 수 있는 기능 통합
  - 스파크에서 별도 옵션 지정 없이 데이터 읽기 가능
- ORC vs Parquet
  - 매우 유사하지만, 차이는 Parquet은 Spark에, ORC는 Hive에 최적화되어있음
  - ORC는 옵션은 따로 없는 듯?
- ORC 파일 읽기
  - 예제 참고
- ORC 파일 쓰기
  - 예제 참고

<details><summary class="point-color-can-hover">[9.5] ORC 파일 read/write 예제 펼치기</summary>

```scala
// ORC 파일 읽기
spark.read.format("orc").load("/data/flight-data/orc/2010-summary.orc").show(5)
// +-----------------+-------------------+-----+
// |DEST_COUNTRY_NAME|ORIGIN_COUNTRY_NAME|count|
// +-----------------+-------------------+-----+
// |    United States|            Romania|    1|
// |    United States|            Ireland|  264|
// |    United States|              India|   69|
// |            Egypt|      United States|   24|
// |Equatorial Guinea|      United States|    1|
// +-----------------+-------------------+-----+


// ORC 파일 쓰기
csvFile.write.format("orc").mode("overwrite").save("/tmp/my-json-file.orc")
```

```bash
# 다른 포멧과 동일한 분할 규칙
$ ls /tmp/my-json-file.orc/
part-00000-a45b9d23-eb06-48d1-935a-110cfdbddfdb-c000.snappy.orc  _SUCCESS
```

</details>

### 9.6 SQL 데이터베이스
> 예제 추가하면서 내용 보충 예정

- SQLite 샘플로 예제 (DB 설정과정 생략) _ 분산환경에서 사용해서는 X
- JDBC 데이터소스 옵션 (`p.262-263 [표 9-6]` 참고)
- SQL 데이터베이스 읽기
- 쿼리 푸시다운
  - 데이터베이스 병렬로 읽기
  - 슬라이딩 윈도우 기반의 파티셔닝
- SQL 데이터베이스 쓰기

### 9.7 텍스트 파일
- 일반 텍스트 파일(plain-text file) 도 읽기 가능
  - 각 줄이 DataFrame의 레코드
  - 변환은 마음대로 가능 (ex. 아파치 로그 파일 → 구조화된 포멧으로 파싱, 자연어 처리를 위한 일반 텍스트 파싱)
  - 기본 데이터 타입의 유연성 활용 가능 => Dataset API 활용 👍🏻
- 텍스트 파일 읽기
  - 예제 참고
  - `textFile(텍스트 파일)` 사용
  - `text()` : 파티션된 텍스트 파일을 읽고 쓸 경우 파티션 수행 결과로 생성된 디렉토리를 인식할 수 있음 (`textFile()`은 무시)
- 텍스트 파일 쓰기
  - 예제 참고
  - **문자열 컬럼이 하나만 존재**해야함 (아닐 경우 실패)
  - 파티셔닝 작업 수행 시 더 많은 컬럼 저장 가능
    - 단 모든 파일에 컬럼 추가 아님
    - 텍스트 파일이 저장되는 디렉토리에 폴더별로 컬럼 저장

<details><summary class="point-color-can-hover">[9.7] 텍스트 파일 read/write 예제 펼치기</summary>

```scala
// 텍스트 파일 읽기
(spark.read.textFile("/data/flight-data/csv/2010-summary.csv")
  .selectExpr("split(value, ',') as rows").show())
// +--------------------+
// |                rows|
// +--------------------+
// |[DEST_COUNTRY_NAM...|
// |[United States, R...|
// |[United States, I...|
// |[United States, I...|
// |[Egypt, United St...|
// |[Equatorial Guine...|
// |[United States, S...|
// |[United States, G...|
// |[Costa Rica, Unit...|
// |[Senegal, United ...|
// |[United States, M...|
// |[Guyana, United S...|
// |[United States, S...|
// |[Malta, United St...|
// |[Bolivia, United ...|
// |[Anguilla, United...|
// |[Turks and Caicos...|
// |[United States, A...|
// |[Saint Vincent an...|
// |[Italy, United St...|
// +--------------------+


// 텍스트 파일 쓰기
// * 문자열 컬럼이 하나만 존재해야 한다 (=> 아닐 시 작업 실패)
csvFile.select("DEST_COUNTRY_NAME").write.text("/tmp/simple-text-file.txt")

// 데이터 저장 시 파티셔닝 작업 수행하면 더 많은 컬럼 저장 가능
// 모든파일에 저장 X. 저장 디렉토리에 폴더 별로 컬럼 저장됨
(csvFile.limit(10).select("DEST_COUNTRY_NAME", "count")
  .write.partitionBy("count").text("/tmp/five-csv-files2.csv"))
```

```bash
# (Result)
$ ls /tmp/five-csv-files2.csv
count=1  count=24  count=25  count=264  count=29  count=44  count=477  count=54  count=69  _SUCCESS

$ ls /tmp/five-csv-files2.csv/count\=1/
part-00000-35cf6fc6-e27f-4861-9c88-d4ce2b913f80.c000.txt
```

</details>

### 9.8 고급 I/O 개념
- 고급 I/O
  - 쓰기 작업 전 파티션 수를 조절 => 병렬 처리 파일 수 제어 가능
  - **버케팅** & **파티셔닝** => 데이터 저장 구조 제어 가능
- 분할 가능한 파일 타입과 압축 방식
  - 특정 파일 포맷은 기본적으로 분할 지원
    - => 스파크가 전체 파일이 아닌 쿼리에 필요한 부분만 읽음
    - => 성능 향상
  - 하둡 분산 파일 시스템 (HDFS) 같은 시스템 : 분할된 파일을 여러 블록으로 나누어 분산 최적화
    - => 더 좋고요~ 최적화 ↑
  - 압축 방식 : 모든 압축 방식이 분할 압축을 지원하지는 X
    - 데이터 저장 방식이 스파크 잡의 원활한 동작에 영향이 큼
    - => **파케이 파일포맷 + GZIP 압축방식** 추천
- 병렬로 데이터 읽기
  - 여러 익스큐터가 동시에 같은 파일 읽기는 불가능. 여러 파일 읽기는 가능!
  - ex. 다수 파일이 존재하는 폴더를 읽는 상황
    - 폴더의 개별 파일 = DataFrame의 파티션
    - => 사용 가능한 익스큐터를 이용해서 병렬로 파일 읽기 O
- 병렬로 데이터 쓰기
  - 파일과 데이터 수? => 데이터를 쓰는 시점의 DataFrame 파티션 수에 따라 달라질 수 있음
  - 기본적으론 파티션 당 하나의 파일
  - 옵션에 지정하는 파일명은 실제론 다수의 파일을 가진 **디렉토리**
    - 해당 디렉토리 안에 파티션 당 하나의 파일로 데이터 저장 (1:1)

#### 파티셔닝 & 버케팅
- **파티셔닝(partitioning)** : 어떤 데이터를 어디에 저장할 것인지 제어
  - 파티셔닝된 디렉토리 or 테이블에 파일을 쓸 때, 디렉토리 별로 컬럼 데이터를 인코딩해서 저장
  - 즉, 데이터 읽기 시 전체 데이터 스캔 없이 **필요한 컬럼 데이터만 읽기** 가능
  - 특징
    - 모든 파일 기반 데이터소스에서 지원
    - 필터링 자주 사용하는 테이블 사용 시 => 가장 손쉬운 최적화 (읽기 속도 ↑)
  - 예제
    <details><summary class="point-color-can-hover">[9.8] '파티셔닝' 예제 펼치기</summary>

    ```scala
    (csvFile.limit(10).write.mode("overwrite").partitionBy("DEST_COUNTRY_NAME")
      .save("/tmp/partitioned-files.parquet"))
    ```
    ```bash
    # (Result)
    $ ls /tmp/partitioned-files.parquet
    DEST_COUNTRY_NAME=Costa Rica  DEST_COUNTRY_NAME=Egypt  DEST_COUNTRY_NAME=Equatorial Guinea  DEST_COUNTRY_NAME=Senegal  DEST_COUNTRY_NAME=United States  _SUCCESS

    # 각 폴더는 조건절을 폴더명으로 사용 (조건절 만족 데이터가 저장)
    $ ls /tmp/partitioned-files.parquet/DEST_COUNTRY_NAME\=Senegal/
    part-00000-547b6d60-db63-4b83-90e8-005cc890f6c5.c000.snappy.parquet
    ```

    </details>

- **버케팅(bucketing)** : 각 파일에 저장된 데이터를 제어할 수 있는 또 다른 파일 조직화 기법
  - 스파크 관리 테이블에서만 사용 가능
  - 동일한 버킷 ID 가진 데이터는 동일한 물리적 파티션에 존재
  - 즉, 데이터가 이후 사용 방식에 맞춰 사전에 파티셔닝. 조인이나 집계 시의 **고비용 셔플 회피** 가능
  - ex. 특정 컬럼을 파티셔닝해서 수억개 디렉토리 생성되면 => '버켓' 단위로 데이터를 모아 일정 수 파일로 저장
  - 버켓팅 파일 기본 경로 : `/user/hive/warehouse/`
  - 예제
    <details><summary class="point-color-can-hover">[9.8] '버케팅' 예제 펼치기</summary>
    
    ```scala
    val numberBuckets = 10
    val columnToBucketBy = "count"

    (csvFile.write.format("parquet").mode("overwrite")
      .bucketBy(numberBuckets, columnToBucketBy).saveAsTable("bucketedFiles"))
    ```
    ```bash
    # 기본적으로는 /user/hive/warehouse 디렉토리 하위에 버켓팅 파일 기록
    # (=> 디렉토리 먼저 생성해줘야함 `mkdir -p /user/hive/warehouse`)
    $ ls /user/hive/warehouse/

    # => 근데 예제 도커 환경에서는 해당 경로로 안감.. ㅋㅋㅋ;
    # $ find / -name bucketedfiles
    # /zeppelin/spark-warehouse/bucketedfiles

    $ ls /zeppelin/spark-warehouse/bucketedfiles
    part-00000-3aff3e3d-fb8d-4fb6-aeba-1df413ac3e9a_00000.c000.snappy.parquet  part-00000-3aff3e3d-fb8d-4fb6-aeba-1df413ac3e9a_00006.c000.snappy.parquet
    part-00000-3aff3e3d-fb8d-4fb6-aeba-1df413ac3e9a_00001.c000.snappy.parquet  part-00000-3aff3e3d-fb8d-4fb6-aeba-1df413ac3e9a_00007.c000.snappy.parquet
    part-00000-3aff3e3d-fb8d-4fb6-aeba-1df413ac3e9a_00002.c000.snappy.parquet  part-00000-3aff3e3d-fb8d-4fb6-aeba-1df413ac3e9a_00008.c000.snappy.parquet
    part-00000-3aff3e3d-fb8d-4fb6-aeba-1df413ac3e9a_00003.c000.snappy.parquet  part-00000-3aff3e3d-fb8d-4fb6-aeba-1df413ac3e9a_00009.c000.snappy.parquet
    part-00000-3aff3e3d-fb8d-4fb6-aeba-1df413ac3e9a_00004.c000.snappy.parquet  _SUCCESS
    part-00000-3aff3e3d-fb8d-4fb6-aeba-1df413ac3e9a_00005.c000.snappy.parquet
    ```

    </details>

- 더 자세한 내용은 [스파크 서밋 2017](https://bit.ly/2NJQfa2) 참고

- 복합 데이터 유형 쓰기
  - 스파크의 자체 데이터 타입([6장](https://minsw.github.io/2021/02/02/Spark-The-Definitive-Guide-6%EC%9E%A5/) 참고)은 스파크에서는 잘 동작하지만, 모든 데이터 파일 포맷에 적합하지는 X
  - ex. CSV 파일은 복합 데이터 타입 미지원
- 파일 크기 관리
  - 데이터 저장시에는 문제 없음. **읽을 때는 파일 크기는 중요 요소**
  - 작은 파일 多 => 메타데이터에 관리 부하 ↑↑
    - `작은 크기의 파일 문제` : 스파크, HDFS 등 많은 파일 시스템은 작은 크기 파일 잘 못 다룸
  - 그럼 큰 파일은 좋은가? => X
    - 몇개의 로우가 필요해도 전체 데이터 블록을 읽음. 비효율
    - 뭐든 '적당'한게 베스트
  - `maxRecordsPerFile` : 파일당 레코드 수 지정 옵션
    - **자동으로 파일 크기를 제어**할 수 있는 새로운 방법 (since 2.2)
    - 결과 파일 수 = 파일 쓰는 시점의 파티션 수 (파티셔닝 컬럼) 로 결정

### 9.9 정리
- 스파크에서 데이터를 읽고/쓸 때 사용할 수 있는 옵션
- 사용자 정의 데이터 소스 구현하는 방법은 개선 진행 중이므로 스킵. 궁금하다면 모범사례 참고 ([spark-cassandra-connector](https://github.com/datastax/spark-cassandra-connector)) 


### 📒 단어장