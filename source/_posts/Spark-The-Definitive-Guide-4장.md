---
title: '&#039;Spark The Definitive Guide&#039; 4장 - 눈 떠, 구조적 API 들어간다'
date: 2021-01-26 02:24:59
categories: spark
tags:
	- spark
	- apache
	- book
	- study
---

<br/>

#### 📌 [Part 2] 구조적 API : DataFrame, SQL, Dataset

#### 들어가기전, 스파크 기본 개념 살짝 복습 👀
> 스파크는 `트랜스포메이션의 처리 과정을 정의하는 분산 프로그래밍 모델`.
>
> (사용자가 정의한) 다수의 **트랜스포메이션**은 → **DAG** (지향성 비순환 그래프) 로 표현되는 명령을 만들고
> **액션**은 하나의 잡을 클러스터에서 실행하기 위해 → 스테이지와 태스크로 나누고 DAG 처리 프로세스를 실행
>
> 이런 트랜스포메이션과 액션으로 다루는 논리적 구조? => **<u>DataFrame, Dataset</u>**

<img width="300" alt="start" src="https://user-images.githubusercontent.com/26691216/105742251-98dbed00-5f7e-11eb-98c3-96ff8f8679dc.gif"/>

<center><h2>_ _ _</h2></center>

<br/>

---


# Chapter 4 구조적 API 개요

반드시 이해해야 (한다고) 하는 아래 세 가지 기본개념을 설명한다.

- `타입형(typed)` / `비타입형(untyped)` API 개념과 차이점
- 핵심 용어
- 스파크가 구조적 API의 데이터 흐름을 해석하고 클러스터에서 실행하는 방식


### 구조적 API 의 특징
- 데이터 흐름을 정의하는 기본 추상화 개념
- 다양한 유형의 데이터 처리 가능
  - 비정형 로그파일, 반정형 CSV 파일, 정형적 Parquet (파-케이) 파일 등
- 배치(batch), 스트리밍 (streaming) 처리에서 사용 가능
  - 배치 작업 <-> 스트리밍 작업 : 쉽게 변환 가능
- 구조적 API 의 세 가지 분산 컬렉션 API
  - Dataset
  - DataFrame
  - SQL 테이블과 뷰

### 4.1 DataFrame과 Dataset
> DataFrame (코드 사용) == 테이블/뷰 (SQL 사용)

- 스파크의 구조화된 컬렉션 개념 : [DataFrame [2.6]](https://minsw.github.io/2021/01/24/Spark-The-Definitive-Guide-2%EC%9E%A5/#2-6-DataFrame) / [Dataset [3.2]](https://minsw.github.io/2021/01/24/Spark-The-Definitive-Guide-3%EC%9E%A5/#3-2-Dataset-%ED%83%80%EC%9E%85-%EC%95%88%EC%A0%95%EC%84%B1%EC%9D%84-%EC%A0%9C%EA%B3%B5%ED%95%98%EB%8A%94-%EA%B5%AC%EC%A1%B0%EC%A0%81-API)
- 그래서 이것들이 뭔데?
  - 잘 정의된 로우(row), 컬럼(column)을 갖는 분산 테이블 형태의 컬렉션
    - 각 컬럼은 다른 컬럼과 동일한 수의 로우를 가짐 (값 없음은 null)
    - 모든 로우는 같은 데이터 타입 정보
  - 지연 연산의 실행 계획
    - 결과를 생성하기 위해 어떤 데이터에 어떤 연산을 적용해야하는지 정의
  - 불변성 (Immutability)

### 4.2 스키마
- 스키마 : DataFrame의 컬럼명과 데이터 타입 정의
  - 직접 정의 or Schema-on-read (데이터소스에서 얻는 것)
  - 여러 데이터 타입으로 구성

### 4.3 스파크의 구조적 데이터 타입 개요
- 스파크가 사용하는 **카탈리스트 (Catalyst)** 엔진 
  - 다양한 실행 최적화 기능 제공
  - 실행 계획과 처리에 사용하는 자체 데이터 타입 정보를 가짐
- 스파크는 사실상 '프로그래밍 언어'
  - 여러 언어 API와 직접 매핑 (각 언어에 대한 매핑 테이블을 가짐)
  - 파이썬이나 R 로 구조적 API 사용해도 => 대부분의 연산은 ~~각 언어의 자체 데이터 타입~~ 이 아닌 **스파크의 데이터 타입** 사용 (카탈리스트 엔진에서 변환)

    ```scala
    val df = spark.range(500).toDF("number")

    // scala가 아닌 'spark'의 덧셈 연산 수행
    df.select(df.col("number") + 10)
    ```

- `비타입형(untyped)` DataFrame vs `타입형(typed)` Dataset
  - DataFrame, Dataset 모두 데이터 타입은 존재
  - 그러나 스키마에 명시된 데이터 타입 **일치 여부를 확인하는 시점** 차이
    - <u>DataFrame은 **런타임**에</u> 일치 여부 확인 => `비타입형(untyped)`
    - <u>Dataset은 **컴파일 타임**에</u> 일치 여부 확인 => `타입형(typed)`
  - DataFrame 은 Row 타입 사용
    - **Row 타입** : 스파크가 사용하는 '연산에 최적화된 인메모리 포멧' 의 내부적 표현 방식 => 언어와 무관한 동일한 효과/효율성
    - 매우 효율적인 연산 (vs Dataset의 JVM 데이터 타입 : GC, 객체 초기화 부하 .. )
  - Dataset은 지정된 데이터 타입(T) 사용
    - 엄격한 데이터 타입 검증 => CHAPTER 11 참고
- 컬럼 (column)
  - 단순 데이터 타입 (정수형, 문자열), 복합 데이터 타입 (배열, 맵), null 값 표현
  - 스파크는 데이터 타입의 모든 정보를 추적, 다양한 컬럼 변환 방법 제공
  - 스파크의 컬럼 == 테이블의 컬럼
- 로우 (row)
  - 데이터 레코드 (DataFrame의 레코드는 Row 타입)
  - 로우는 SQL, RDD, 데이터 소스에서 얻거나 직접 만들거나
    ```scala
    spark.range(2).toDF().collect()
    // res165: Array[org.apache.spark.sql.Row] = Array([0], [1])
    ```
- 스파크의 **데이터 타입**
  - 스파크는 여러가지 내부 데이터 타입을 가짐
  - `p.116 ~ 118` [표 4-1 ~ 3] 스파크가 지원하는 언어별 매핑 정보 (파이썬/스칼라/자바 데이터 타입 매핑)
  - 최신 데이터 타입은 [스파크 공식 문서](http://bit.ly/2EdflXW) 참고
  - 예시) 언어 별 데이터 타입 초기화 및 정의 방법
    ```scala
    // scala
    import org.apache.spark.sql.types._
    val b = ByteType
    ```

    ```java
    // java (Factory method)
    import org.apache.spark.sql.types.DataTypes;
    ByteType x = DataTypes.ByteType;
    ```

    ```python
    # python
    from pyspark.sql.types import *
    b = ByteType()
    ```


### 4.4 구조적 API의 실행 과정
> 구조적 API 쿼리 (사용자 코드) → 실제 실행 코드 변환 과정
>
> 1. [👩🏻‍💻] DataFrame/Dataset/SQL을 이용해 코드 작성
> 2. [Spark ✨] 코드 => **논리적 실행 계획** 으로 변환 (정상적 코드일 경우)
> 3. [Spark ✨] 논리적 실행 계획 => **물리적 실행 계획** 으로 변환. 추가적인 최적화 가능한지 확인
> 4. [Spark ✨] 클러스터에서 **물리적 실행 계획 (RDD 처리)** 실행

- 좀 더 가깝게 설명하자면..
  - [본인] 이 스파크 코드 작성하고 console 이나 `spark-submit` 으로 실행
  - -> [카탈리스트 옵티마이저] 가 코드를 받고 실제 실행 계획 (물리적) 생성
  - -> [스파크] 는 코드 실행 & 결과 반환

- 논리적 실행 계획
    <img width="700" alt="mermaid-1" src="https://user-images.githubusercontent.com/26691216/105745516-b494c280-5f81-11eb-972a-827a5936fe0f.png"/>
    <details><summary style="color:lightgray"> theme (neutral) </summary>

    ```mermaid
    graph LR
    A[/코드/]-- 1_ -->B[검증 전 ...]
    B-- 2_ Analzer -->C[검증된 ...]
    C-- 3_ Optimizer -->D[최적화된 논리적 실행 계획]
    ```

    </details>

  1) 사용자 코드 → 검증 전 논리적 실행 계획 (unresolved logical plan)
      - 추상적 트랜스포메이션만 표현 (드라이버, 익스큐터 고려 X)
      - 사용자의 표현식을 최적화된 표현으로 변환
  2) 분석기 (Analyzer) : **카탈로그**, 모든 테이블 저장소, DataFrame 정보 => 컬럼과 테이블 검증
  3) 논리적 최적화 (Catalyst Optimizer) : 논리적 실행 계획을 최적화하는 규칙의 모음 (predicate pushing down, 선택절 구문)
- 물리적 실행 계획
    <img width="700" alt="mermaid-2" src="https://user-images.githubusercontent.com/26691216/105745527-b8284980-5f81-11eb-8d36-554b97959946.png"/>
    <details><summary style="color:lightgray"> theme (neutral) </summary>

    ```mermaid
    graph LR
    A[최적화된 논리적 실행 계획] --> B[물리적]
    A --> C[실행계획s]
    B --> D{비용 모델 비교}
    C --> D
    D --> E
    [최적의 물리적 실행 계획]
    E --> F>클러스터에서 처리]
    ```
    </details>

  - (== 스파크의 실행 계획)
  - 논리적 실행 계획을 클러스터 환경에서 실행하는 방법을 정의
  - 비용 모델 비교 후 최적의 전략 선택 (ex. 조인 연산 수행 비용 계산)
  - => RDD, 트랜스포메이션으로 변환하고 컴파일 (*이래서 스파크를 '컴파일러'라고 부르기도*)
- 실행
  - 물리적 실행 계획 선정 후 RDD (저수준 인터페이스) 대상으로 모든 코드 실행
  - 추가 최적화 수행 : 런타임에 자바 바이트 코드 생성 (=> 전체 테스크나 스테이지 제거 가능)
  - => 처리 결과 사용자에게 반환

 

### 4.5 정리
- Spark와 구조적 API
- 사용자 코드 -> 물리적 실행 코드 변환 과정

### 📒 단어장
- 카탈리스트 엔진 (Catalyst Optimizer) : DataFrame DSL, SQL 을 하위 레벨의 RDD 연산으로 변환하는 Optimizer
  - 책에서 언급한 카탈리스트 엔진 구현 영상 - [youtube1](https://youtu.be/5ajs8EIPWGI), [youtube2](https://youtu.be/GDeePbbCz2g)
  - ref. https://thebook.io/006908/part02/ch05/05/
  - ref. https://medium.com/@leeyh0216/spark-sql-6dc3d645cc31


