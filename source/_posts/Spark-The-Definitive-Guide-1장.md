---
title: '&#039;Spark The Definitive Guide&#039; 1장 - Apache Spark (스파크) 는 뭘까'
date: 2021-01-20 09:20:29
categories: spark
tags:
	- spark
	- apache
	- book
	- study
	- summarization
	- mwolkka
---
<br/>

#### 'Spark The Definitive Guide'

<i>스파크 완벽 가이드 (& 하둡 완벽 가이드), O'REILLY</i>

전형적으로 묶어놔야 공부하는 타입이라.. 
어떻게든 마음의 부채를 쌓기 위해 일단 책부터 사고 일도 최대한 크게 벌리고 시작하기로 했다.

책 읽고 간단하게만 정리하는거라 내용이 별거 없긴 해도
또 대충 쓰다 어디 쑤셔 박아놓고서 못 찾지 말고 <i style="color:lightgray">("엄마~ 내 글 어딨어?")</i> 습관도 들일 겸 블로그에 남겨두는게 좋겠다.

**완벽** 가이드라니까 일단 파트 3까지는 아묻따 따라가보자. 자 드가자~!



> **Apache Spark™** is a unified analytics engine for large-scale data processing.
>
> Apache Saprk 공식 페이지 https://spark.apache.org/
> Latest Release Spark 3.0.1

<br/>

<img src="https://user-images.githubusercontent.com/26691216/105111053-699f2900-5b03-11eb-87c1-05d1c704f2d3.jpg" width=240 />


<center><h2>_ _ _</h2></center>

<br/>

---

# CHAPTER 1 아파치 스파크란

### 아파치 스파크 (Apache Spark) 란
통합 컴퓨팅 엔진이며 클러스터 환경에서 데이터를 병렬로 처리하는 라이브러리 집합.

- Spark 언어 4대 천왕 - Python, Java, Scala, R
- SQL, Library, ML 등 라이브러리 제공

### 1.1 아파치 스파크의 철학
> **빅데이터를 위한 통합 컴퓨팅 엔진과 라이브러리 집합.**

- 통합 (unified)
  - 다양한 처리 유형을 지원하기 위한 자체 API
- 컴퓨팅 엔진
  - 스파크는 데이터 저장 위치에 상관없이 처리에 집중
  - vs 하둡
    - 하둡은 저비용 저장장치를 사용하는 하둡 파일 시스템과 컴퓨팅 시스템(MR) 이 밀접하게 연관
    - 스파크는 하둡 저장소 뿐만 아니라 하둡 아키텍처를 사용할 수 없는 환경에서도 호환 가능
- 라이브러리
  - 궁극적으론 데이터 분석 작업에 필요한 통합 API를 제공하는 통합 엔진 기반의 자체 라이브러리
  - 수많은 외부 라이브러리도.. (https://spark-packages.org/)


### 1.2 스파크의 등장 배경
- 더 많은 연산과 대규모 데이터 처리를 프로세서의 성능 향상에 맡겼으나, H/W 성능 향상은 2005년까지
- 기술의 발전으로 데이터 수집 비용은 저렴해졌지만 데이터는 클러스터에서 처리해야할 만큼 거대해짐
- 새로운 프로그래밍 모델이 필요 => **아파치 스파크**

### 1.3 스파크의 역사
- UC버클리 대학에서 2009년 프로젝트로 시작
  - 당시 하둡 맵리듀스가 클러스터 환경용 병렬 프로그래밍 엔진의 대표주자
- '표준 라이브러리' 형태의 구현 방식
  - 조합형 API의 핵심 아이디어 진화
  - ~ 1.0 : 함수형 연산
  - 1.0 : 구조화된 데이터 기반의 스파크 SQL
  - 이 후에는 더 강력한 구조체 기반의 신규 API 들 (ex. DataFrame, 머신러닝 파이프라인, 구조적 스트리밍)

### 1.4 스파크의 현재와 미래
- 거대 규모 데이터셋이나 과학적 데이터 분석에 사용 중
- 인기 많다는 이야기

### 1.5 스파크 실행하기 -> 부록 A
> *... 특히 도커 환경에서 예제를 실행해보고 싶다면 부록 A를 참고하시기 바랍니다.*

```bash
$ docker pull docker.io/rheor108/spark_the_definitive_guide_practice
$ docker run -p 8080:8080 -p 4040:4040 --name spark_ex rheor108/spark_the_definitive_guide_practice
# Zeppelin UI : http://localhost:8080 
```
> [부록 A] Spark The Definitive Guide 예제 실행 환경 Docker image
> - https://hub.docker.com/r/rheor108/spark_the_definitive_guide_practice
>
> - Spark version: 2.3.2 Python version: 2.7 Zeppelin version: 0.8.0 (21.01 기준)
>
> Spark The Definitive Guide 저장소
> - 원서 : https://github.com/databricks/Spark-The-Definitive-Guide
> - 번역 예제 : https://github.com/FVBros/Spark-The-Definitive-Guide


```bash
# Docker image 실행하면 기본적으로 /spark-2.3.2-bin-hadoop2.7/bin 에 대화형 콘솔 존재
# + 2.4.7 다운로드 받기
wget https://downloads.apache.org/spark/spark-2.4.7/spark-2.4.7-bin-hadoop2.7.tgz
tar -xf spark-2.4.7-bin-hadoop2.7.tgz
```


### 1.6 정리
- 스파크의 개요 / 탄생 배경 / 환경 구성 방법
