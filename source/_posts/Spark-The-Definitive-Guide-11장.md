---
title: '&#039;Spark The Definitive Guide&#039; 11장 - 셋뚜셋뚜 데이터셋뚜 (PART 2 끝)'
date: 2021-03-01 00:56:15
categories: spark
tags:
  - spark
  - apache
  - book
  - study
---

<br/>

#### [Part 2] END

드디어 길고 길었던 [구조적 API](https://minsw.github.io/2021/01/26/Spark-The-Definitive-Guide-4%EC%9E%A5/) 파트 끝 🎉
예제가 많아서 계속 차근차근 따라가면서 읽다보니 한 챕터 당 소요시간이 훨씬 길었다.. 후...

남은 파트들은 템포가 좀 짧았으면 좋겠다 (´；ω；`)
<p style="color:lightgray"><i>(해치웠나..?)</i></p>

<br/>

<img src="https://user-images.githubusercontent.com/26691216/109769822-1362f100-7c3e-11eb-9f22-9a1d3be261dd.png" width=400/>
<center>쎘뚜쎘뚜~ 데이터프레임이랑 셋뚜~</center>


<center><h2>_ _ _</h2></center>

<br/>

---

# CHAPTER 11 Dataset
Dataset은 구조적 API의 기본 데이터 타입 (DataFrame = Row 타입의 Dataset `Dataset[Row]`)
스팤잘알들은 '타입형 API' 라고 부르기도 한다

- Dataset은 JVM 사용하는 Scala, Java에서만 사용 가능 (DataFrame은 다양한 언어 사용 가능)
  - Scala는 스키마가 정의된 케이스 클래스 객체로 정의
  - Java는 자바 빈 객체로 정의

#### 인코더 (encoder) ?
- 인코더 : 도메인별 특정 객체 T → 스파크의 내부 데이터 타입으로 매핑하는 시스템 
  - 인코더가 스파크에게 지시 (런타임 환경) : 객체 (클래스)  → 바이너리 구조로 직렬화 코드 생성 
  - DataFrame 이나 '표준' 구조적 API 사용 시 :  Row 타입 → 바이너리 구조로 변환
- 도메인에 특화된 객체를 만들어 사용하려면 <U>**사용자 정의 데이터 타입**</U> 정의 필요
  - Scala의 `case class`, Java의 `JavaBean` 형태로
  - 스파크는 Row 타입 대신 사용자 정의 데이터 타입을 분산 방식으로 다루기 가능
- Dataset API 사용 시
  - 스파크가 데이터셋에 접근할 때마다 사용자 정의 데이터 타입으로 변환 (Row 포맷 X)
  - => **느리다 (성능 ↓)** / 대신 더 많은 유연성
  - 사용자 정의 함수 (Python) 이랑 비슷?
    - 사용자 정의 데이터 타입 >>>>>> 사용자 정의 함수 (언어 전환이 훨씬 느림)

<br/>

### 11.1 Dataset을 사용할 시기

> ~~*"그럼 Dataset 성능 구린데 왜 쓰나?"*~~
>
> **Dataset을 사용해야하는 2가지 이유**
>   1) DataFrmae 기능만으로는 수행할 연산을 표현할 수 없는 경우
>  2) 성능 저하를 감수하더라도, 타입 안정성(type-safe)를 가진 데이터 타입을 사용하고 싶은 경우

- (1) 구조적 API로 표현할 수 없는 작업들
  - ex. 복잡한 비즈니스로직을 단일 함수(SQL X, DataFrame X) 로 인코딩해야하는 경우
- (2) 타입 안정성 (정확도↑, 방어적 코드)
  - 데이터 타입이 유효하지 않은 작업 => 컴파일 타임에 오류 발생 (런타임 X)
  - 잘못된 데이터로부터 보호는 X
  - 그러나 보다 우아하게 데이터를 제어, 구조화 가능
- \+ **로컬과 분산 환경의 워크로드에서 재사용 가능**
  - When?
    - 단일 노드의 워크로드와 스파크 워크로드에서 전체 로우에 대한 다양한 트랜스포메이션을 재사용하고자할 때
  - How? 
    - 스파크 API = Scala의 Sequence 타입 API 가 일부 반영 + 분산 방식 동작 ([Scala 만든 형 said](http://bit.ly/2NFqKGZ))
    - 케이스 클래스로 구현된 데이터 타입으로 모든 데이터와 트랜스포메이션을 정의하면 재사용 가능
  - 또한 올바른 클래스, 데이터 타입이 지정된 DataFrame을 로컬 디스크에 저장하면 다음 처리 과정에서 사용 가능 (more easy)
- DataFrame + Dataset 동시 사용
  - 성능 <-> 타입 안정성 (Trade-Off)
  - 대량의 DataFrame 기반의 ETL 트랜스포메이션의 <U>마지막 단계</U>에서 사용 가능
    - ex. 드라이버로 데이터를 수집 후, 단일 노드의 라이브러리로 수집된 데이터 처리하는 경우
  - 트랜스포메이션 <U>첫 단계</U>에서 사용도 가능
    - ex. 스파크 SQL에서 필터링 전에 로우 단위로 데이터 파싱하는 경우

### 11.2 Dataset 생성
- Dataset 생성은 수동 작업
  - 정의할 스키마를 미리 알고 있어야 함
- 자바 (Java) : Encoders
  - 데이터 타입 클래스 정의 후,  DataFrame (`= Dataset<Row>`) 에 지정해서 인코딩
    ```java
    import org.apache.spark.sql.Encoders;

    public class Flight implements Serializable {
      String DEST_COUNTRY_NAME;
      String ORIGIN_COUNTRY_NAME;
      Long DEST_COUNTRY_NAME;
    }

    Dataset<Flight> flights = spark.read
      .parquet("/data/flight-data/parquet/2010-summary.parquet/")
      .as(Encoders.bean(Flight.class)); // Encoders 지정
    ```
- 스칼라 (Scala) : 케이스 클래스
  - Scala의 케이스 클래스 (`case class`) 는 정규 클래스(regular class)
  - 특징
    - **불변성**
    - 패턴 매칭으로 분해가능
    - 참조값 대신 클래스 구조를 기반으로 비교
    - 사용하기 쉽고 다루기 편함
  - 장점
    - 불변성 => 객체의 변경 추적 필요 X
    - 값 대신 구조로 비교 가능 => 클래스 인스턴스가 값으로 비교되는지 참조로 비교되는지 걱정 X (값 비교시 인스턴스를 primitive 데이터 타입 값 처럼 비교함)
    - 패턴 매칭 => 로직 분기 단순화 (버그 ↓ 가독성 ↑)
    - (더 자세한 건 [스칼라 문서](https://bit.ly/2xdwSfd) 참고)
  - case class 로 데이터 타입 정의. DataFrame의 `as()` 로 변환
    ```scala
    case class Flight(DEST_COUNTRY_NAME: String,
                      ORIGIN_COUNTRY_NAME: String, count: BigInt)
    
    val flightsDF = spark.read
      .parquet("/data/flight-data/parquet/2010-summary.parquet/")
    val flights = flightsDF.as[Flight] // Dataset
    ```


### 11.3 액션
- Dataset, DataFrame의 힘보다 "액션을 적용할 수 있다"는 사실이 더 중요하다
  - `collect()`, `take()`, `count()` ...
- 케이스 클래스에 실제로 접근 시 "어떠한 데이터 타입도 필요하지 않다"는 사실도 알아라
  - '속성 명' 으로 해당 값 & 데이터 타입 모두 반환됨

<p style="color:lightgray">말투 무엇..</p>

### 11.4 트랜스포메이션
- Dataset의 트랜스포메이션 == DataFrame의 트랜스포메이션
  - Dataset은 원형의 JVM 데이터 타입을 다루므로, 더 복잡하고 강력한 데이터 타입으로 사용 가능
  - 원형 객체 다루는 법? => **필터링 & 매핑**
- 필터링
  - 불리언 값을 반환하는 함수 (=> **일반 함수**) 정의
    - 스파크 SQL은 **사용자 정의 함수** 정의
    - 스파크는 정의된 함수로 모든 로우를 평가 (자원 사용량 ↑)
    - 단순 필터의 경우 SQL 표현식 사용 권장
     - 데이터 필터링 비용 ↓ 다음 처리과정에서 Dataset으로 데이터 다루기 가능
  - 단순 트랜스포메이션
- 매핑
  - 특정 값을 다른 값으로 매핑 작업 (값 추출/비교 등의 정교한 처리)
  - DataFrame의 매핑 = Dataset의 `select()`
  - 컴파일 타임에 데이터 타입 유효성 검사 가능 (스파크가 결과로 반환되는 JVM 데이터 타입을 알고 있기 때문)
- 사실 DataFrame 사용을 권장함
  - 매핑 작업보다 더 많은 장점.. (ex. 코드 생성 기능) 대다수의 매핑작업 수행가능..
  - 하지만 훨씬 정교하게 로우 단위 처리가 필요하다면 Dataset

### 11.5 조인
- 조인도 DataFrame과 동일하게 제공
  - Dataset은 정교한 메서드 제공 => `joinWith()`
- `joinWith()`
  - co-group (RDD) 과 거의 유사
  - Dataset 안쪽에 다른 두 개의 중첩된 Dataset으로 구성
    - 각 컬럼은 단일 Dataset => Dataset 객체를 컬럼처럼 다루기 가능
  - => 조인 시 더 많은 정보 유지 가능. 고급 맵이나 필터처럼 정교한 데이터 다루기 가능
- 일반 조인 (`join()`)
  - => 결과가 DataFrame으로 반환. JVM 데이터 타입 정보를 잃음
  - DataFrame + Dataset 조인도 문제 X (동일 결과 반환) 

### 11.6 그룹화와 집계
- 동일한 기본 표준을 따름
  - `groupBy()`, `rollup()`, `cube()` 그대로 사용 가능
  - 단, DataFrame을 반환 (데이터 타입 정보 잃음)
- 데이터 타입 정보를 유지하려면?
  - ex. `groupByKey()`
    - Dataset 특정 키 기준으로 그룹화하고 형식화된 Dataset 반환
    - 파라미터는 함수 사용 (컬럼명 X) => 유연성 good / 최적화 X
      - 스파크는 함수랑 JVM 데이터 타입 최적화 X
      - (성 능 차 이)
  - groupByKey vs groupBy
    - 데이터 스캔 직후에 집계를 수행하는 groupBy 보다 처리비용이 더 비쌈
    - 따라서  사용자가 정의한 인코딩으로 세밀한 처리가 필요하는 등의 필요한 경우에만 사용할 것
- Dataset은 빅데이터 처리 파이프라인의 처음과 끝에서 주로 사용 할 것

### 11.7 정리
- Dataset의 기초와 Dataset 사용이 적합한 경우
- Dataset을 사용하기 위한 기본 지식 및 사용 방법
- Dataset = 고수준의 구조적 API + 저수준 RDD API


### 📒 단어장
