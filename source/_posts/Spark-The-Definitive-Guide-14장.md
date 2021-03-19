---
title: '&#039;Spark The Definitive Guide&#039; 14장 - 분산형 공유 변수 (PART 3 끝)'
date: 2021-03-20 01:02:00
categories: spark
tags:
  - spark
  - apache
  - book
  - study
---

<br/>

#### [Part 3] END

> *["... 완벽 가이드라니까 일단 파트 3까지는 아묻따 따라가보자."](https://minsw.github.io/2021/01/20/Spark-The-Definitive-Guide-1%EC%9E%A5/)*

마침내 🎈약속의 파트 쓰리! 🎈   🙌🏻
빠르지는 않았지만 꾸준함에 의미를 두고 싶다.

스파크 너란 녀석.. 이젠 뭔지 조금은 알지도...?
<i style="color:lightgray">( 『5252.. 이제부턴 '실전'이다ㅡ 』 )</i>

<img alt="dooly" src="https://user-images.githubusercontent.com/26691216/111810920-dd9a5980-8919-11eb-8d2d-2b26434a01ae.jpg" width=300/>
<center style="color:lightgray">선 넘네..</center>

<br/>

<center><h2>_ _ _</h2></center>

<br/>

---

# CHAPTER 14 분산형 공유 변수
스파크의 저수준 API의 두 번째 유형, **'분산형 공유 변수'**
분산형 공유 변수 타입이 만들어지게 된 계기, 사용 방법 소개

- 클러스터 실행 시 특별한 속성을 가진 사용자 정의함수 (ex. RDD, DataFrame을 다루는 map 함수)에서 사용 가능
- 분산형 공유 변수의 2가지 타입 : **브로드캐스트 변수**, **어큐뮬레이터**

### 14.1 브로드캐스트 변수
> 브로드캐스트 변수는 **모든 워커노드에 큰 값 저장** => 재 전송 없이 많은 스파크 액션에서 재사용 가능

<img width="400" alt="bv" src="https://user-images.githubusercontent.com/26691216/111816566-543a5580-8920-11eb-88eb-ba873e1384fc.png">

- **브로드캐스트 변수**
  - 변하지 않는 값(불변성 값)을 클로저(closure) 함수의 변수로 캡슐화하지 않고, 클러스터에서 효율적으로 공유하는 방법 제공
- \(1) 태스크에서 드라이버 노드의 변수 사용 시 <U>클로저 함수 내부에서 단순 참조</U>
    - => **비효율적**. 특히 큰 변수 사용시 (ex. 룩업 데테이블, 머신러닝 모델)
      - Why?
      - 클로저 함수에서 변수 사용 시 => 워커 노드에서 여러 번 (태스크당 한번) 역직렬화 발생
      - 여러 스파크 액션과 잡에서 동일 변수 사용시 => 잡 실행 때마다 워커로 큰 변수 재전송
    - 그러면 어떻게 하나?
      - => <U>**브로드캐스트 변수 사용**해라!</U>
- \(2) 브로드 캐스트 변수 사용 시
  - 모든 태스크마다 직렬화 X => **클러스터의 모든 머신에 캐시하는 불변성 공유 변수**
  - 익스큐터 메모리에 맞는 조회용 테이블을 전달하고 함수에서 사용
- How to use ?
  - `spark.sparkContext.broadcast()` 로 참조 (= 불변성 값)
    - 액션을 실행할때 클러스터 모든 노드에 지연 처리 방식으로 복제됨
  - `value` 메서드로 브로드캐스트된 값 참조 가능
    - 직렬화된 함수에서 브로드캐스트된 데이터를 직렬화 하지않아도 접근 가능
    - 데이터를 보다 효율적으로 전송. **직렬화/역직렬화 부하 ↓**

- 클로저에 담아 전달 vs 브로드캐스트 변수 사용
  - 말해뭐해. **브로드캐스트 변수가 훨씬 더 효율적**
    - 데이터 총량, 익스큐터에 따라 다를 수는 있지만.. (작은 데이터를 작은 클러스터에서 돌릴 땐 별 차이 X)
  - 브로드캐스트 변수에 작은 크기의 딕셔너리(dictionary) 타입 사용 시 부하 크게 발생 X (?)
    - 훨씬 큰 크기 데이터 사용 시 전체 데이터 직렬화 시 발생 부하 커질 수도
- RDD 영역에서 브로드캐스트 변수 사용 (UDF, Dataset도 사용 가능. 동일 효과)

<details><summary class="point-color-can-hover">[14.1] '브로드캐스트 변수' 예제 펼치기 </summary>

```scala
// '단어의 값과 목록 + 수 KB~GB 크기를 가지는 다른 정보' 브로드캐스트 후 RDD로 변환하는 예제
val myCollection = ("Spark The Definitive Guide : Big Data Processing Made Simple"
  .split(" "))
val words = spark.sparkContext.parallelize(myCollection, 2)

// (== SQL Right JOIN)
val supplementalData = Map("Spark" -> 1000, "Definitive" -> 200,
                           "Big" -> -300, "Simple" -> 100)


// 해당 구조체 브로드캐스트 => suppBroadcast 변수로 참조 가능 (불변)
val suppBroadcast = spark.sparkContext.broadcast(supplementalData)
// suppBroadcast: org.apache.spark.broadcast.Broadcast[scala.collection.immutable.Map[String,Int]] = Broadcast(0)

// value 메서드로 직렬화 하지 않아도 접근 가능
suppBroadcast.value
// res11: scala.collection.immutable.Map[String,Int] = Map(Spark -> 1000, Definitive -> 200, Big -> -300, Simple -> 100)

// 브로드캐스트된 데이터로 키-값 RDD 변환 가능 (+ 값 없을 시 0 으로 처리)
(words.map(word => (word, suppBroadcast.value.getOrElse(word, 0)))
  .sortBy(wordPair => wordPair._2)
  .collect())
// res14: Array[(String, Int)] = Array((Big,-300), (The,0), (Guide,0), (:,0), (Data,0), (Processing,0), (Made,0), (Simple,100), (Definitive,200), (Spark,1000))
```

</details>

### 14.2 어큐뮬레이터
> 어큐뮬레이터는 **모든 태스크 데이터를 공유 결과에 추가 가능** (ex. 잡의 입력 레코드 파싱하면서 오류 발생 확인 카운터 구현 가능)

<img width="400" alt="av" src="https://user-images.githubusercontent.com/26691216/111816573-569caf80-8920-11eb-8906-13a25b3707d1.png">

- **어큐뮬레이터**
  - 트랜스포메이션 내부의 다양한 값 갱신하는데 사용
  - 내고장성 보장 + 효율적인 방식으로 드라이버에 값 전달 가능
- 어큐뮬레이터는 스파크 클러스터에서 <U>로우 단위로 안전하게 값을 갱신할 수 있는 변경 가능한 변수</U> 제공
  - **디버깅용, 저수준 집계 생성용**으로 사용 가능 (ex. 파티션별로 특정 변수 값 추적 용도)
  - 결합성, 가환성을 가진 연산을 통해서만 더할 수 있는 변수 => 병렬 처리 과정에서 효율적 사용 가능
  - 카운터(==맵리듀스의 카운터)나 합계 구하는 용도로 사용 가능
- 스파크는 기본적으로 수치형 어큐뮬레이터 지원. 사용자 정의 어큐뮬레이터 만들어서 사용도 OK
- 어큐뮬레이터의 값은 **액션** 처리 과정에서만 갱신됨
  - 스파크는 <U>**각 태스크에서 어큐뮬레이터를 한 번만 갱신**</U>하도록 제어 (재시작한 태스크는 갱신X)
  - 트랜스포메이션에서 태스크 or 잡 스테이지 재처리 시? => 각 태스크 갱신 작업이 두 번 이상 적용될 수도
- 어큐뮬레이터는 스파크의 지연 연산 모델에 영향 X
  - 어큐뮬레이터가 RDD 처리 중 갱신되면? => RDD 연산이 실제로 수행된 지점에 딱 한번만 값 갱신  (해당 or 부모 RDD에 액션을 실행하는 시점)
  - 지연 처리 형태의 트랜스포메이션 (ex. `map()`) 에서 어큐뮬레이터 갱신 작업 수행시? => 실제 실행 전까지는 갱신 X
- 어큐뮬레이터 이름은 선택적 지정 가능
  - 생성 시 파리미터로 이름을 붙이거나 `spark.sparkContext.register(어큐뮬레이터, 이름)` 사용
  - 이름이 지정된 (named) 어큐뮬레이터의 실행결과는 스파크 UI에 표시
  - 이름이 지정되지 않은 어큐뮬레이터는 표시 X

- **사용자 정의 어큐뮬레이터**
  - 어큐뮬레이터 직접 정의 시 => `AccumulatorV2` 클래스 상속하여 구현
  - 파이썬 사용 시 [AccumulatorParam](https://bit.ly/2x7RnL7) 상속

<details><summary class="point-color-can-hover">[14.2.1] 
'어큐뮬레이터' 예제 펼치기 </summary>

```scala
// Dataset API 사용 (RDD API X)
// 출발지나 도착지가 중국인 항공편의 수를 구하는 어큐뮬레이터 생성하기 예제
case class Flight(DEST_COUNTRY_NAME: String,
                  ORIGIN_COUNTRY_NAME: String, count: BigInt)
val flights = (spark.read
  .parquet("/data/flight-data/parquet/2010-summary.parquet")
  .as[Flight])

import org.apache.spark.util.LongAccumulator

// 이름이 지정되지 않은 어큐뮬레이터 생성
val accUnnamed = new LongAccumulator
val acc = spark.sparkContext.register(accUnnamed)

// 이름이 지정된 어큐뮬레이터 생성
val accChina = new LongAccumulator
val accChina2 = spark.sparkContext.longAccumulator("China")
spark.sparkContext.register(accChina, "China")		// register() 로 이름 지정 가능
// accChina: org.apache.spark.util.LongAccumulator = Un-registered Accumulator: LongAccumulator
// accChina2: org.apache.spark.util.LongAccumulator = LongAccumulator(id: 101, name: Some(China), value: 0)


// 어큐뮬레이터에 값 더하는 방법 정의
def accChinaFunc(flight_row: Flight) = {
  val destination = flight_row.DEST_COUNTRY_NAME
  val origin = flight_row.ORIGIN_COUNTRY_NAME
  if (destination == "China") {
    accChina.add(flight_row.count.toLong)
  }
  if (origin == "China") {
    accChina.add(flight_row.count.toLong)
  }
}

// foreach() 는 액션 => DataFrame의 매 로우마다 함수 한번씩 적용 (어큐뮬레이터 갱신)
flights.foreach(flight_row => accChinaFunc(flight_row))

accChina.value // 953
```

> Spark UI (이름이 지정된 어큐뮬레이터)

<img width="400" alt="china-result" src="https://user-images.githubusercontent.com/26691216/111823196-00336f00-8928-11eb-93b4-14ae1c731ab2.png">

</details>


<details><summary class="point-color-can-hover">[14.2.2] 
사용자 정의 어큐뮬레이터 구현 예제 펼치기 </summary>

```scala
// 사용자 정의 어큐뮬레이터 (AccumulatorV2 상속 클래스 구현)
import scala.collection.mutable.ArrayBuffer
import org.apache.spark.util.AccumulatorV2

val arr = ArrayBuffer[BigInt]()

class EvenAccumulator extends org.apache.spark.util.AccumulatorV2[BigInt, BigInt] {
  private var num:BigInt = 0
  def reset(): Unit = {
    this.num = 0
  }
  def add(intValue: BigInt): Unit = {
    if (intValue % 2 == 0) {
      this.num += intValue
    }
  }
  def merge(other: AccumulatorV2[BigInt,BigInt]): Unit = {
    this.num += other.value
  }
  def value():BigInt = {
    this.num
  }
  def copy(): AccumulatorV2[BigInt,BigInt] = {
    new EvenAccumulator
  }
  def isZero():Boolean = {
    this.num == 0
  }
}
val acc = new EvenAccumulator
val newAcc = sc.register(acc, "evenAcc")


acc.value // 0
flights.foreach(flight_row => acc.add(flight_row.count))
acc.value // 31390
```

</details>

### 14.3 정리
- 분산형 공유 변수 (브로드캐스트 변수, 어큐뮬레이터)
- 분산형 공유 변수는 디버깅이나 최적화 작업에 유용한 도구

### 📒 단어장
- 내고장성 (Fault tolerance) : 시스템 일부에 문제가 발생하여도 정상 동작이 가능한 
- 결합성 (Associative) : 둘 이상의 이항연산 중첩시, 연산 결과가 순서에 관계없이 동일
    > *(a+b)+c = a+(b+c) = a+b+c*
- 가환성 (Commutative) : 연산 결과가 순서에 관계없이 동일 (=> [2장 단어장](https://minsw.github.io/2021/01/24/Spark-The-Definitive-Guide-2%EC%9E%A5/#%F0%9F%93%92-%EB%8B%A8%EC%96%B4%EC%9E%A5))
    > *a+b = b+a*
