---
title: '&#039;Spark The Definitive Guide&#039; 12장 - RDD'
date: 2021-03-07 15:44:49
categories: spark
tags:
  - spark
  - apache
  - book
  - study
---

<br/>

#### 📌 [ Part 3 저수준 API ]
> 대부분의 상황에서는 구조적 API 사용이 좋음 (Part 2)
>
> 그러나 이러한 고수준(hive-level) API로 처리할 수 없는 비즈니스나 기술적 문제는? => <U>**저수준 API**</U> 사용
>
> - RDD
> - SparkContext
> - 분산형 공유 변수 (distributed shared variables)
>   - 어큐뮬레이터 (accumulator)
>   - 브로드캐스트 변수 (broadcast variable)

<br/>

<img src="https://user-images.githubusercontent.com/26691216/110234092-04e14600-7f6c-11eb-966f-153324981b5c.jpg" width=300/>

<center><h2>_ _ _</h2></center>

<br/>

---

# CHAPTER 12 RDD

### 12.1 저수준 API란
- 스파크의 저수준 API 종류 2가지
  - 분산 데이터 처리를 위한 **RDD**
  - **분산형 공유 변수**를 배포하고 다루기위한 API 
- When?
  - 고수준 API에서 제공하지 않는 기능이 필요한 경우
    - ex. 클러스터의 물리적 데이터 배치를 세밀하게 제어
  - RDD를 사용해 개발된 기존 코드 유지
  - 사용자가 정의한 공유 변수를 다뤄야하는 경우 (=> 14장)
- 장점
  - 스파크의 모든 워크로드는 저수준 기능을 사용하는 기초 형태로 컴파일됨 => 이해하여 워크로드 디버깅 작업에 도움
  - 세밀한 제어 방법 제공 (치명적인 실수 방지)
- How?
  - 저수준 API 기능 사용의 진입 지점 => `SparkContext`
  - 자세한 내용은 15장에서
    ```scala
    # 스파크 연산 수행 시 필요한 도구 'SparkSession' 으로 접근 가능
    spark.sparkContext
    ```


### 12.2 RDD 개요
- RDD는 스파크 1.x 의 핵심 API 
  - 2.x 부터는 잘 사용안하지만, 모든 DataFrame/Dataset 코드는 RDD로 컴파일
  - 스파크 UI에서도 RDD단위로 잡 수행
- **RDD** 란
  - 불변성을 가지고 병렬로 처리할 수 있는 파티셔닝된 레코드의 모음
    - DataFrame의 레코드 : 스키마를 알고 있는 필드로 구성된 구조화된 로우
    - RDD의 레코드 : 그저 프로그래머가 선택하는 자바/스칼라/파이썬의 객체
  - RDD의 모든 레코드는 객체이므로 완벽하게 제어 가능
    - 개발자가 강력한 제어권을 가짐
    - 모든 값을 다루거나 값 사이 상호작용에 대해 반드시 수동 정의 필요
    - => <U>*"Reinvent the wheel"*</U>
  - 내부 구조를 파악할 수 없음 (구조적 API는 가능)
    - 최적화에 많은 수작업 요구
  - => **결론은 스파크 구조적 API 사용을 강력하게 권고**
- RDD API 는 Dataset과 유사
  - 단, RDD는 구조화된 데이터 엔진을 사용해서 데이터를 저장하거나 다루지 X
  - RDD <-> Dataset 전환 쉬움
  - 두 API의 각각의 장점 동시에 활용 가능

#### RDD 유형
- 수많은 하위 클래스 존재
  - 대부분 DataFrame API의 최적화된 물리적 실행 계획 만드는데 사용
- RDD 타입 2 가지 (객체의 컬렉션 표현)
  - 제네릭 RDD 타입
  - 키-값 RDD (특수연산 + 키를 이용한 사용자 지정 파티셔닝 개념 포함)
- RDD 의 주요 속성 5가지
  - 파티션의 목록
  - 각 조각을 연산하는 함수
  - 다른 RDD와의 의존성 목록
  - 부가적으로 키-값 RDD를 위한 Partitioner (ex. RDD는 hash partitioned)
  - 부가적으로 각 조각을 연산하기 위한 기본 위치 목록 (Ex. 하둡 분산 파일 시스템의 블록 위치)
- 각 속성은 사용자 프로그램을 스케쥴링, 실행 하는 스파크의 모든 처리방식을 결정
  - 속성 구현체로 새로운 데이터 소스 정의도 가능
  - **Partitioner** 는 RDD 사용하는 주된 이유 중 하나
    - 올바른 사용자 정의 Partitioner => 성능, 안정성 ↑
    - 자세한 내용은 13장
- RDD는 스파크 프로그래밍 패러다임을 그대로 따른다
  - 지연 처리 방식의 <U>트랜스포메이션</U>, 즉시 실행 방식의 <U>액션</U> 제공
  - DataFrame, Dataset과 동일한 방식으로 동작하지만 **'로우' 개념이 없음**
    - 개별 레코드는 객체일뿐 (구조적 API가 제공하는 함수들의 동작을 수동으로 처리해야함)
- RDD API는 스칼라, 자바, 파이썬 모두 사용 가능
  - 스칼라, 자바 => 대부분 비슷한 성능 (단, 원형 객체 사용시 성능 ↓)
  - 파이썬 => 상당한 성능 저하 (높은 오버헤드)

#### RDD는 언제 사용할까
- A. 물리적으로 분산된 데이터 (자체 구성한 데이터 파티셔닝) 에 세부적인 제어가 필요할 때 => RDD 사용
- 하지만 정말 필요한 경우 빼고는 수동 RDD 생성 비권장
  - 구조적 API의 최적화 기법 사용 X
  - DataFrame이 훨씬 효율적, 안정적, 좋은 표현력

#### Dataset과 RDD의 케이스 클래스
- Dataset vs 케이스 클래스로 만들어진 RDD ?
- Dataset은 구조적 API가 제공하는 풍부한 기능과 최적화 기법 제공
  - JVM 데이터 타입 / 스파크 데이터 타입 중 고민 필요 X (성능 동일)

### 12.3 RDD 생성하기
- DataFrame, Dataset으로 RDD 생성하기
  - 가장 쉬운 방법
  - DataFrame이나 Dataset의 `rdd()` 호출
    ```scala
    // (Scala) Dataset[Long] => RDD[Long] 변환
    spark.range(500).rdd
    ```
    ```python
    # (python) Row타입의 RDD 반환 (Python은 Dataset X)
    sprak.range(10).rdd
    ```
  - Row 타입 : 스파크가 구조적 API에서 데이터를 표현하는 데 사용하는 내부 카탈리스트 포맷
    - 상황에 따라 구조적 API <-> 저수준 API
  - Dataset API와 RDD API의 유사성

- 로컬 컬렉션으로 RDD 생성하기
  - `sparkContext.parallelize()` : 컬렉션 객체 => RDD
    - 단일 노드에 있는 컬렉션을 병렬 컬렉션으로 전환
    - 파티션 수 명시적 지정 가능
  - RDD에 이름 지정 시 스파크 UI에 해당 이름으로 노출

- 데이터소스로 RDD 생성하기
  - DataSource API 사용
    - 데이터소스나 텍스트 파일을 이용한 직접 생성보다 바람직
    - 데이터를 읽는 가장 좋은 방법
  - RDD에는 DataFrame이 제공하는 'Datasource API' 개념 X
    - `sparkContext` 를 사용하여 RDD로 읽기 가능

### 12.4 RDD 다루기
- DataFrame을 다루는 방식과 매우 유사
- RDD vs DataFrame, Dataset
  - 스파크 데이터 타입이 아닌 자바, 스칼라 객체를 다룬다는 점이 가장 큰 차이
  - '헬퍼' 메서드(연산 단순화) 나 함수 부족
  - => 필터, 맵 함수, 집계 등의 다양한 함수를 직접 정의해야함

### 12.5 트랜스포메이션
- 대부분의 RDD 트랜스 포메이션 => 구조적 API에서 사용 가능
  - RDD에도 트랜스포메이션을 지정해서 새로운 RDD 생성 가능 + 의존성 정의 (like DataFrame, Dataset)
- `distinct()` : RDD의 중복된 데이터 제거
- `filter()` : 조건 함수를 만족하는 레코드만 반환
  - 모든 로우는 어떤 경우라도 입력값을 가져야 함
  - 함수는 각 레코드를 개별적으로 처리 후, 결과를 사용한 언어의 데이터 타입으로 반환한다 (like Dataset API) => 강제로 Row 타입으로 변환할 필요가 없어서?
- `map()` : 주어진 입력을 원하는 값으로 반환하는 함수를 레코드 별로 적용
  - `flatMap()` : 단일 로우 → 여러 로우로 변환
- `sortBy()` : 함수의 추출 값 기준으로 RDD 정렬
- `randomSplit()` : RDD를 임의로 분할해 배열로 만듦
  - 가중치, 난수시드(random seed) 배열을 파라미터로 사용

### 12.6 액션
- 지정된 트랜스포메이션 연산을 시작하기 위해 '액션' 사용
  - 데이터를 드라이버로 모으거나, 외부 데이터 소스로 내보내기 가능
- `reduce()` : RDD의 모든 값을 하나로 만듦
  - 파티션에 대한 리듀스 연산은 비결정적 (not deterministic) => 결과가 매번 달라질 수 있음
- `count()` : RDD의 전체 로우 수 반환
  - `countApprox()` : 제한시간 내에 count 근사치 계산 (confidence = 신뢰도 = 오차율)
  - `countApproxDistinct()` 는 2가지 구현체 존재
    - 상대 정확도(relative accuracy)를 파라미터로 사용 : 더 많은 메모리 공간 사용. (설정값 >= 0.000017)
    - 일반 (regular) 데이터를 위한 파라미터 & 희소표현을 위한 파라미터 사용
  - `countByValue()` : RDD값의 개수 반환
    - 연산결과가 메모리에 모두 적재되므로 결과가 작을때만 사용 
  - `countByValueApprox()` : 제한시간 내에 countByValue의 근사치 계산 (+ confidence)
- `first()` : 첫 번째 값 반환
- `max()`, `min()` : 각각 최댓값, 최솟값 반환
- `take()` : 지정한 개수 만큼 값 반환
  - 먼저 하나의 파티션 스캔 후, 결과 수를 이용해 필요한 추가 파티션 수를 예측
  - 다양한 유사함수 존재 (`takeOrdered()`, `takeSample()`, `top()` ..)

### 12.7 파일 저장하기
- 데이터 처리 결과를 일반 텍스트 파일로 쓰는 것
  - RDD는 일반적 의미의 데이터소스에 '저장' 할 수 없음
    - 각 파티션의 내용을 저장하려면 전체 파티션을 순회하면서 외부 데이터베이스에 저장
    - (== 고수준 API의 내부 처리 과정)
  - 스파크는 각 파티션의 데이터를 파일로 저장
- `saveAsTextFile()` : 지정한 경로로 텍스트 파일 저장
  - 필요 시 압축 코덱 사용 가능 (코덱은 [org.apache.hadoop.io.compress](https://bit.ly/2p3dnT3) 참고)
- 시퀀스 파일
  - 바이너리 키-값 쌍으로 구성된 플랫 파일
  - 맵리듀스의 입출력 포맷으로 알려짐
  - => `saveAsObjectFile()`이나 명시적 키-값 쌍 데이터 저장 방식으로 시퀀스 파일 작성 가능
- 하둡 파일
  - 데이터를 저장하는데 사용할 수 있는 여러 하둡 파일 포맷 존재
  - 하둡 파일 포맷 사용 시 클래스, 출력 포맷, 하둡 설정, 압축 방식 지정 가능 ('하둡 완벽 가이드' 도서 참고)

### 12.8 캐싱
- DataFrame, Dataset 캐싱과 동일 원칙 적용
  - RDD 는 캐시, 저장(persist) 가능
    ```scala
    words.cache()
    ```
  - 기본적으로 캐시와 저장은 메모리에 있는 데이터가 대상
  - `setName()` : 캐시된 RDD에 이름 지정 가능
- 저장소 수준
  - 싱글턴 객체 `org.apache.spark.storage.StorageLevel` 의 속성(메모리, 디스크, 메모리+디스크 조합, off-heap) 중 하나로 지정 가능
  - 저장소 수준은 `getStorageLevel()` 로 조회 가능

### 12.9 체크포인팅
- **체크포인팅 (checkpointing)** : RDD => 디스크에 저장
  - DataFrame API에서 사용할 수 없는 기능 중 하나
  - 저장된 RDD 참조 시, 원본 데이터소스를 다시 계산해 디스크에 저장된 중간 결과 파티션을 참조 (RDD 생성 X)
  - 캐싱과 유사 (단, 메모리가 아닌 디스크에 저장)
  - 반복적인 연산 수행시 유용 (최적화 good)
- `checkpoint()` 설정 => RDD 참조 시 데이터소스의 데이터 대신 체크포인트에 저장된 RDD 사용

### 12.10 RDD를 시스템 명령으로 전송하기
- `pipe()`
  - 파이핑 요소로 생성된 RDD => 외부 프로세스로 전달
  - 외부 프로세스는 파티션마다 한번씩 처리 => 결과 RDD 생성
    - 입력 파티션 : 각 입력 파티션의 모든 요소는 개행 문자 단위로 분할, 여러 줄의 입력 데이터로 변경 => 프로세스의 표준 입력 (stdin) 으로 전달
    - 결과 파티션 => 프로세스의 표준 출력 (stdout) 에 전달. 표준 출력의 각 줄은 출력 파티션의 하나의 요소가 됨
  - 비어있는 파티션 처리 시에도 프로세스는 실행됨
  - 사용자가 정의한 두 함수를 인수로 전달 시, 출력 방식을 원하는대로 변경 가능 ([API docs](https://bit.ly/2OhgRwd) 참고)
- `mapPartitions()`
  - 스파크는 실제 코드 실행 시 파티션 단위로 동작
    - `map()` 이 반환하는 RDD => `MapPartitionsRDD`
    - <U>map은 mapPartitions의 로우 단위 처리를 위한 별칭일 뿐이다</U>
  - mapPartitions는 개별 파티션(이터레이터로 표현) 에 대해 `map` 연산 수행 가능
    - 클러스터에서 물리적 단위로 개별 파티션을 처리하기 때문 (로우 단위 처리 X)
  - 기본적으로 파티션 단위로 작업 수행
    - => **전체** 파티션에 대한 연산 수행 가능
    - RDD의 전체 하위 데이터셋에 원하는 연산 수행 가능
  - `mapPartitionsWithIndex()` : 인덱스 (파티션 범위의 인덱스, RDD의 파티션 번호) 와 파티션의 모든 아이템을 순회하는 이터레이터를 가진 함수를 인수로 사용
- `foreachPartition()`
  - 개별 파티션에서 특정 작업을 수행하는데 적합
  - `mapPartitions()` vs `foreachPartition()`
    - mapPartitions 는 처리 결과 반환, foreachPartition 는 결과 반환 X (순회만)
- `glom()`
  - 데이터셋의 모든 파티션 => 배열로 변환
  - 데이터를 드라이버로 모으고, 데이터가 존재하는 파티션의 배열이 필요한 경우 유용
  - (단, 파티션이 크거나 많으면 드라이버가 비정상적으로 종료)

### 12.11 정리
- 단일 RDD 처리 방법
- RDD API의 기초

### 📒 단어장