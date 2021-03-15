---
title: '&#039;Spark The Definitive Guide&#039; 13장 - RDD 심화반'
date: 2021-03-15 22:05:03
categories: spark
tags:
  - spark
  - apache
  - book
  - study
---

<br/>

RDD 쓰지 말래매...
자꾸 왜 이렇게 뭘 자세하게 알려주려고 하는데... 😔


<img alt="class" src="https://user-images.githubusercontent.com/26691216/111161260-4b731800-85de-11eb-8870-31d56a87f978.gif" width=400/>


<center><h2>_ _ _</h2></center>

<br/>

---

# CHAPTER 13 RDD 고급 개념
CHAPTER 12 는 RDD를 다루기 위한 기초적 내용과 생성하는 방법, 적합한 적용 케이스 등을 소개
CHAPTER 13 은 키-값 RDD 중심의 **RDD 고급 연산** 과 사용자 정의 파티션 등의 고급 주제 소개

- 집계와 키-값 형태의 RDD
- 사용자 정의 파티셔닝
- RDD 조인

### 13.1 키-값 형태의 기초(키-값 형태의 RDD)
- RDD에는 데이터를 key-value 형태로 다루는 다양한 메서드 존재
  - => `<연산명>ByKey()` (PairRDD 타입만 사용 가능)
- **PairRDD 타입**
  - 만드는 가장 쉬운 방법?
  - => RDD에 맵 연산해서 키-값 구조로 만들기
    - == RDD 레코드에는 두 개의 값이 존재한다
      ```scala
      words.map(word => (word.toLowerCase, 1))
      // org.apache.spark.rdd.RDD[(String, Int)]
      ```

- `KeyBy()` : 현재 값으로부터 키 생성
  - 원본 단어는 생성된 RDD **값** 으로 유지

- 값 매핑하기
  - 튜플 형태 데이터 사용 시 => 첫 번째를 키, 두 번째를 값 으로 추정 (`(key, value)`)
  - `mapValues()` : 튜플에서 값만 추출 가능 (키 제외)
  - `flatMap()` : 값에 대해 flatMap 적용하여 배열 반환 가능 (ex. 반환 결과의 각 로우가 문자를 나타내게)
    ```scala
    keyword.mapValues(word => word.toUpperCase).collect()
    // res12: Array[(String, String)] = Array((s,SPARK), (t,THE), (d,DEFINITIVE), (g,GUIDE), (:,:), (b,BIG), (d,DATA), (p,PROCESSING), (m,MADE), (s,SIMPLE))

    keyword.flatMapValues(word => word.toUpperCase).collect()
    // res15: Array[(String, Char)] = Array((s,S), (s,P), (s,A), (s,R), (s,K), (t,T), (t,H), (t,E), (d,D), (d,E), (d,F), (d,I), (d,N), (d,I), (d,T), (d,I), (d,V), (d,E), (g,G), (g,U), (g,I), (g,D), (g,E), (:,:), (b,B), (b,I), (b,G), (d,D), (d,A), (d,T), (d,A), (p,P), (p,R), (p,O), (p,C), (p,E), (p,S), (p,S), (p,I), (p,N), (p,G), (m,M), (m,A), (m,D), (m,E), (s,S), (s,I), (s,M), (s,P), (s,L), (s,E))
    ```

- 키와 값 추출하기
  - `.keys`, `.values` 메서드로 키나 값 전체 추출 가능

- `lookup()` : 특정 키에 관한 결과 찾기
  - 단, 오직 하나의 키만 찾을 수는 X (찾기 결과 전체 반환)

- `sampleByKey()` : 근사치/정확도로 RDD 샘플 생성
  - 특정 키를 부분 샘플링 가능
  - `sampleByKeyExtract()` : 99% 신뢰도를 가진 모든 키 값에 대해 RDD 추가 처리
  - `sampleByKey` vs `sampleByKeyExtract`
    - `sum(math.ceil(numItems * samplingRate))` 과 '거의' 동일한 크기냐, '완전히' 동일한 크기의 샘플을 만드느냐의 차이 
    - (.. 이게 무슨말이지..? => [API docs](https://spark.apache.org/docs/2.2.0/api/scala/index.html#org.apache.spark.rdd.PairRDDFunctions) 참고)
  - 선택적으로 비복원 추출 사용 가능
    - 비복원 추출 사용 시 => 샘플 크기 보장을 위해 RDD 한번 더 통과
    - 복원 추출 => RDD 두번 더 통과

### 13.2 집계
- 사용하는 메서드에 따라 **일반 RDD** or **PairRDD** 사용

- `countByKey()` : 각 키의 아이템 수 구하고 + 로컬 맵으로 결과 수집
  - Scala, Java 사용 시 => 제한 시간(timeout), 신뢰도 인수로 지정하여 근사치 구하기 가능 (=`countByApprox`)

- 집계 연산 구현 방식 이해하기
  - PairRDD (키-값 형태) 생성을 위한 구현 방식은 중요 (for 잡의 안정성)
  - `groupBy` vs `reduce` 구현 방식 비교 (동일한 기본 원칙)
    - 예제 - 각 키의 총 레코드 수 구하기
    - `groupByKey().map()` : 모든 익스큐터에서 함수 적용 전에 해당 키와 관련된 **모든 값을 메모리로** 읽음 (=> 파티션 쏠림 현상 시 OOM 발생 가능성)
    - `reduceByKey()` : 각 파티션에서 리듀스 작업 수행하고, 최종 리듀스 외 모든 작업은 개별 워커에서 처리 (연산 중 셔플 X) . 모든 값 메모리 유지 필요 없음
  - `groupByKey()`
    - 각 키에 대한 값의 크기가 일정하고, 익스큐터에 할당된 메모리에서 처리 가능한 정도의 크기일 경우에만 사용
    - 결과 순서 보장 O
  - `reduceByKey()`
    - 키별 그룹 RDD 반환 (개별요소 정렬X) => 결과의 순서가 중요한 경우에는 비적합
    - 안정성, 연산수행속도 빠름. 작업 부하 감소에 good

- 기타 집계 메서드
  - 구조적 API는 훨씬 간단하게 집계 가능 (권장)
    - 저수준 사용시, 사용자 워크로드에 따라 세부 구현 방법에 차이 존재
    - 그러나 아주 구체적이고 매우 세밀한 제어 가능
  - `aggregate(시작값)(func1, func2)`
    - null or 집계 시작값 필요 (두 함수 모두 시작값 사용)
    - func1 은 파티션 내에서 수행, func2는 모든 파티션에 걸쳐 수행
    - `aggregate` 는 드라이버에서 최종 집계
      - => 성능에 영향 有 (익스큐터 결과가 크면 OOM 발생가능성)
    - `treeAggregate()` 는 드라이버 최종 집계 전 익스큐터끼리 형성한 트리로 집계 처리 일부 하위 과정을 'push down' 방식으로 먼저 수행
      - 집계 처리를 여러 단계로 구성 (메모리 전체 소비 방지) => 안정적
  - `aggregateByKey(시작값)(func1, func2)`
    - aggregate와 동일. 단 파티션 대신 **키 기준**으로 연산 수행
  - `combineByKey(컴바이너, func1, func2, (파티션 수))`
    - 집계 함수대신 '컴바이너' 사용
    - **컴바이너(combiner)** : 키 기준으로 연산 수행 + 파라미터의 함수로 값 병합 + 여러 컴바이너 값을 병합한 결과 반환
    - 사용자 정의 파티셔너로 출력 파티션 수 지정 가능
  - `foldByKey(제로값, func)`
    - 결합 함수와 항등원인 '제로값' 으로 각 키의 값 병합
    - **제로값**은 여러번 사용될 수 있으나 결과 값을 변경할 수 없는 수 (ex. 덧셈의 0, 곱셈의 1)

### 13.3 cogroup
- `cogroup()`
  - RDD에 대한 그룹 기반의 조인 수행
    - 스칼라는 최대 3개, 파이썬은 최대 2개의 키-값 형태 RDD 그룹화 가능
    - 각 키 기준으로 값 결합
  - 사용자 정의 파티션 함수 파라미터로 사용 가능
    - => 출력 파티션 수, 클러스터에 데이터 분산 방식 정확하게 제어 가능
- 결과 => 키-값 형태의 배열 반환
  - key : 그룹화된 키
  - value : 키와 관련된 모든 값

### 13.4 조인
- 구조적 API 와 거의 동일한 조인 방식
  - 더 많은 부분에 사용자의 관여 필요
  - 조인 대상인 두 RDD (+ 출력 파티션 수, 사용자 정의 파티션 함수)
- 내부 조인
  - `RDD.join(RDD, 파티션 수)`로 출력 파티션 수 지정 가능
  - 다른 조인 함수도 동일한 기본 형식 따름 (=> [8장](https://minsw.github.io/2021/02/15/Spark-The-Definitive-Guide-8%EC%9E%A5/) 참고)
    - `fullOuterJoin` : 외부 조인
    - `leftOuterJoin` : 왼쪽 외부 조인
    - `rightOuterJoin` : 오른쪽 외부 조인
    - `cartesian` : 교차 조인 (danger⚠️)

- `zip()`
  - 진짜 조인은 아니지만, **두 RDD 결합**
    - zipper 잠그듯..🤐  => 연결된 PairRDD 반환
    - 한 RDD의 키 배열에 + 다른 RDD의 값이 지퍼처럼 연결
    - (예제로는 넘버링에 유용할 듯)
  - 두 RDD는 동일한 수의 요소 & 파티션을 가져야함

### 13.5 파티션 제어하기
- RDD를 사용한다는건..
  - => 클러스터 전체에 물리적으로 정확히 분산되는 방식 정의 가능
  - 일부 메서드는 구조적 API 메서드와 기본적으로 동일
  - 차이점은 *"**파티션 함수(사용자 지정 Partitioner)를 파라미터로** 사용할 수 있다는 것"*
- `coalesce(파티션 수)` :  동일한 워커에 존재하는 파티션 합침 (파티션 수 ↓)
  - 파티션 재분배 시 발생하는 데이터 셔플 X
- `repartition(파티션 수)` : 파티션 수 증감 가능
  - 처리 시 노드 간 셔플 발생 가능
  - 파티션 수 ↑ => 맵, 필터 타입 연산 시 병렬 처리 수준 ↑
- `repartitionAndSortWithinPartitions(파티셔너)` : 파티션 재분배 & 결과 파티션 정렬 방식 지정 가능
  - 파티셔닝, 키 비교 모두 사용자 지정 가능
  - [API docs](https://bit.ly/2NCHCOH) 참고


#### 사용자 정의 파티셔닝
- **사용자 정의 파티셔닝 (custom partitioning)**
  - RDD를 사용하는 가장 큰 이유 중 하나
    - 저수준 API의 세부적인 구현 방식
    - 구조적 API에서는 사용 불가 (논리적 대응책 X)
  - 잡이 성공적으로 동작되는지 여부에 상당한 영향
    - ex. 페이지랭크(PageRank) : 사용자 정의 파티셔닝 → 클러스터의 데이터 배치 구조 제어 및 셔플 회피
  - 목표 : **데이터 치우침(skew)** 문제 회피를 위한 데이터 균등 분배
- 구조적 API + RDD 장점 같이 활용하는 법
  - 구조적 API로 RDD 얻기
  - => 사용자 정의 파티셔너 적용
  - => 다시 DataFrame/Dataset 으로 변환
- How to use?
  - Partitioner 확장 클래스 구현 <i style="color:lightgray">(아 자신있으면 쓰라고 ㅋㅋ)</i>
  - 단일 값, 다수 값(다수 컬럼) 파티셔닝 시 => <U>DataFrame API</U> 사용 권장
- RDD, 구조적 API 모두 사용 가능한 **내장형 파티셔너** (기초적 기능 제공)
  - `HashPartitioner` : 이산형 값
  - `RangePartitioner` : 연속형 값
- 매우 큰 데이터나 심각하게 치우친 키를 다룰 경우 고급 파티셔닝 기능 필요 (=> 키 치우침)
  - **키 치우침** : 어떤 키가 다른 키에 비해 아주 많은 데이터를 가지는 현상
    - => 최대한 키 분할 필요 (병렬성 개선 + OOM 방지)
  - 키 분할이 필요한 예 : 키가 특정한 형태를 띠는 경우 (예제 참고)
    - *"우스꽝스러운 예제일 수 있지만, 실무에서 비슷한 상황 만날지도 모릅니다"*
- 이러한 **사용자 정의 키 분배로직**은 RDD 수준에서만 사용 가능
  - It's 임의의 로직을 사용해 물리적인 방식으로 클러스터에 데이터를 분배하는 강력한 방법

<details><summary class="point-color-can-hover">[13.5] 사용자 정의 키 분배로직 예제 펼치기</summary>

```scala
import org.apache.spark.Partitioner

// 분석 복잡도를 높이는 두 고객(17850, 12583) 과 다른 고객 정보 분리
class DomainPartitioner extends Partitioner {
 def numPartitions = 3
 def getPartition(key: Any): Int = {
   val customerId = key.asInstanceOf[Double].toInt
   if (customerId == 17850.0 || customerId == 12583.0) {
     return 0
   } else {
     return new java.util.Random().nextInt(2) + 1
   }
 }
}

// 각 파티션 수 확인
// (두 고객은 partition 0에, 나머지 고객들은 남은 partition 1, 2에 분산)
(keyedRDD
  .partitionBy(new DomainPartitioner).map(_._1).glom().map(_.toSet.toSeq.length)
  .take(5))
// res83: Array[Int] = Array(2, 4290, 4304)
```

> - `glom()` : 각 파티션의 모든 요소를 배열 하나로 모으고, 이 배열들을 요소로 포함하는 새로운 RDD를 반환
>
> (ref. https://thebook.io/006908/part01/ch04/02/04/02/)

</details>

### 13.6 사용자 정의 직렬화
- **Kryo 직렬화**
  - 병렬화 대상인 모든 객체나 함수는 직렬화 가능해야함
- "빠르다!"
  - 기본 직렬화 기능은 매우 느릴 수 있음
  - 스파크는 Kryo 라이브러리 (ver. 2) 사용 => **더 빠른 직렬화** (자바보다 성능 10배 이상 ↑, 간결)
- 단점
  - 모든 직렬화 유형 지원 X
  - 최상의 성능을 위해서는 사용할 클래스 사전에 등록 필요
- How to use ?
  - SparkConf 로 잡 초기화 시점에서 지정
    - `spark.serializer = org.apache.spark.serializer.KryoSerializer`
    - Kryo 자세한 내용은 4부 에서
  - `spark.serializer` : 워커 노드 간 데이터 셔플링과 RDD를 직렬화해 디스크에 저장하는 용도로 사용할 시리얼라이즈 지정
- Kryo가 기본 값이 아닌 이유?
  - 사용자가 직접 클래스를 등록해야 하므로
  - 네트워크에 민감한 애플리케이션에서 사용 권장
  - \+ (since 스파크 2.0.0)
    - 단순 데이터 타입, 단순 데이터 타입의 배열, 문자열 데이터 타입의 RDD 셔플링 시 => 내부적으로 KryoSerializer 사용
- 클래스 등록
  - 스파크는 [Twitter chill 라이브러리](https://github.com/twitter/chill) 의 `AllScalaRegistrar` 핵심 스칼라 클래스를 자동으로 KryoSerializer 에 등록
  - 사용자 정의 클래스 등록하려면? =>`registerKyroClasses()` 사용

### 13.7 정리
- RDD의 여러 고급 주제들
- 특히 ['사용자 정의 파티셔닝' [13.5.4]](#사용자-정의-파티셔닝) 주목

### 📒 단어장
- 항등원 (neutral) : 어떤 원소와 연산을 취해도 자기 자신이 되게하는 원소. (= neutral value = identity element)
- 이산형 값 (discrete variable) : 값을 하나 하나 셀 수 있는 (=이산) 변수 (Like 정수)
- 연속형 값 (continuous variable) : 구간 내 모든 값을 가질 수 있는 변수 (Like 실수) 
