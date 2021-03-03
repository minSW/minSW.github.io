---
title: '&#039;Spark The Definitive Guide&#039; 10장 - SQL 너마저'
date: 2021-02-23 15:42:28
categories: spark
tags:
  - spark
  - apache
  - book
  - study
---


<img src="https://user-images.githubusercontent.com/26691216/109770707-3e9a1000-7c3f-11eb-9e63-1a6235580fa8.jpg" width=300/>

<center><h2>_ _ _</h2></center>

<br/>

---

# CHAPTER 10 스파크 SQL
**스파크 SQL**은 스파크에서 가장 중요하고 강력한 기능 중 하나.
스파크 SQL은 DataFrame, Dataset API에 통합되어 있다
(=> 즉 SQL, DataFrame 기능 모두 사용 가능 & 동일한 실행 코드로 컴파일)
- DB에 생성된 뷰(view)나 테이블에 SQL쿼리 실행 가능
- 시스템 함수 사용, 사용자 정의 함수 정의 가능
- 쿼리 실행 계획 분석 가능 (=> 워크로드 최적화)

### 10.1 SQL이란
- SQL (Structured Query Launguage, 구조적 질의 언어)
  - 데이터에 대한 관계형 연산을 표한하기 위한 **도메인 특화 언어**
  - <U>모든 관계형 데이터베이스</U>에서 사용
  - NoSQL 데이터베이스에서도 쉽게 사용 가능한 변형된 자체 SQL 제공
  - <p style="color:lightgray"> 없어질거다~ 진다~ 해도 없어질 수가 없는 데이터 처리 언어 계의 짱짱맨 </p>
- 스파크 SQL
  - [ANSI SQL:2003](https://bit.ly/2NcpcVl) 의 일부를 구현
  - => 대부분의 SQL DB에서 채택하고 있는 표준 (so 유명 벤치마크도 통과)
 
### 10.2 빅데이터와 SQL: 아파치 하이브
- 스파크 등장 전
  - 사실상 빅데이터 SQL 접근 계층의 표준 => **Hive** (made by Facebook)
  - 하둡을 다양한 사업군으로 진출하는데 도움
- 스파크는 RDD를 이용하는 범용  처리 엔진으로 시작했지만, 현재는 많은 이용자가 
스파크 SQL 사용 중

### 10.3 빅데이터와 SQL: 스파크 SQL
- Hive 지원하는 상위 호환 기능 지원
  - 스파크 2.0 버전. 자체 개발된 SQL 파서 포함
  - => ANSI-SQL, HiveQL 모두 지원
- 스파크 SQL은 DataFrame과의 뛰어난 호환성
  - 다양한 기업에서 강력한 기능으로 자리매김할 수 있는 이유
  - ex. Facebook 의 [스파크 워크로드 (2016)](https://bit.ly/2bLrvJG) (블로그 내용 `p.280-281`)
- 스파크 SQL의 강력함을 만드는 핵심 요인
  - SQL 분석가) Thrift Server 나 SQL 인터페이스에 접속해, 스파크 연산 능력 활용 가능
  - 데이터 엔지니어 & 과학자) 전체 데이터 처리 파이프라인에 스파크 SQL 사용 가능
- 스파크 SQL (통합형 API) 으로 인해 가능해진 전체 과정
  - SQL 로 데이터 조회
  - => DataFrame으로 변환
  - => 스파크 MLlib (대규모 머신러닝 알고리즘) 수행 결과를 다른 데이터 소스에 저장
- 단, 스파크 SQL은 **OLAP** 데이터베이스로 동작 (OLTP X)
  - **OLAP** (OnLine Analytic Processing, 온라인 분석용)
  - OLTP (OnLine Transaction Processing, 온라인 트랜잭션 처리)
  - => 따라서 매우 낮은 지연 시간이 필요한 쿼리 수행용도로는 사용 X
  - 언젠가는 인플레이스 수정(in-place modification) 방식을 지원할수는 있으나, 현재는 **사용 불가**

#### 스파크와 하이브의 관계
- 스파크 SQL은 Hive 메타스토어 사용 => 하이브와 연동 good
  - Hive 메타스토어는 여러 세션에서 사용할 테이블 정보 보관
  - (Hive 사용 시) 스파크 SQL 는 Hive Metastore에 접속해서 조회할 파일 수를 최소화 하기 위해 메타데이터 참조함
    - => 기존 하둡 환경의 모든 워크로드를 스파크로 이관하기 좋음
- Hive Metastore 
  - 접속하기위해 필요한 속성
    - `spark.sql.hive.metastore.version` (default 1.2.1)
    - `spark.sql.hive.metastore.jars` : HiveMetastoreClient 초기화 방식 변경
  - 스파크는 기본 버전 사용 (호환을 위해 클래스패스에 정의도 가능)
  - Hive Metastore가 저장된 다른 데이터베이스 접속 시
    - 적합한 클래스 접두사 정의 필요 (ex. MySQL => `com.mysql.jdbc`)
    - 접두사 정의 후 `spark.sql.hive.metastore.sharedPrefixes` 속성에 설정하여 스파크, 하이브에서 공유
  - 사용하던 하이브 메타스토어와 연동 시, [스파크 공식 문서](http://bit.ly/2DFlcrL) 에서 부가 정보 확인

### 10.4 스파크 SQL 쿼리 실행 방법
- 스파크가 SQL 쿼리 실행을 위하여 제공하는 인터페이스
- (1) 스파크 SQL CLI
- (2) 스파크의 프로그래밍 SQL 인터페이스
- (3) 스파크 SQL 쓰리프트 JDBC/ODBC 서버

### 10.5 카탈로그
- **카탈로그** (catalog) : 스파크 SQL에서 가장 높은 추상화 단계
  - 테이블에 저장된 데이터에 대한 메타데이터 + 데이터베이스, 테이블, 함수, 뷰에 대한 정보 까지 추상화
- `org.apache.spark.sql.catalog.Catalog` 패키지
  - 테이블, 데이터베이스, 함수 조회 등 여러 유용한 함수 제공
- 스파크 SQL을 사용하는 또 다른 방식의 <U>프로그래밍 인터페이스</U> 이다
  - => `saprk.sql` 함수를 사용해 관련 코드 실행 가능

### 10.6 테이블
- 스파크 SQL 사용을 위해서는 테이블 정의 부터 필요
- 테이블은 '명령을 실행할 데이터의 구조'
  - DataFrame과 논리적으로 동일
  - => 테이블로 조인, 필터링, 집계 등의 여러 데이터 변환 작업 수행 가능
- 테이블 vs DataFrame
  - 테이블은 데이터베이스에서 정의, DataFrame은 프로그래밍 언어로 정의
  - 스파크에서 테이블 생성시 => <U>**default** 데이터 베이스에 등록</U>
- 테이블은 **'항상 데이터를 가지고 있다'** (스파크 2.x 버전)
  - 임시 테이블 개념 X
  - 데이터를 가지지않은 뷰만 존재 (테이블 X)
  - 테이블 제거 시 모든 데이터 제거됨

#### 테이블 다루기

- 스파크 관리형 테이블
  - 테이블은 두 가지 중요 정보 저장 => 테이블의 데이터 & 테이블에 대한 데이터 (**메타데이터**)
  - 관리형 테이블 & 외부 테이블 개념 알기
    - **관리형 테이블** : DataFrame의 `saveAsTable()` 로 생성 (스파크가 관련된 모든 정보를 추적 가능)
    - **외부 테이블** : 디스크에 저장된 파일을 이용해 테이블을 정의 (파일의 메타데이터 관리)
  - `saveAsTable()` 메서드
    - 테이블을 읽고, 데이터를 스파크 포맷으로 변환 후, 새로운 경로에 저장 (Hive 기본 웨어하우스 경로)
  - 저장 경로에서 데이터베이스 목록 확인 가능
      - 기본 저장 경로 : /user/hive/warehouse
      - 저장 경로 변경? => SparkSession 생성 시 `spark.sql.warehouse.dir` 에 설정
  - `show tables IN {databaseName}` 쿼리로 특정 데이터베이스 테이블 확인 가능

- 테이블 생성하기
  - 다양한 데이터소스로 테이블 생성 가능
  - 스파크는 SQL에서 전체 데이터소스 API 재사용 가능
    - 즉, 테이블 정의 후 테이블에 데이터 적재 필요 X (?)
    - 실행 즉시 테이블 생성
  - 파일에서 데이터 읽을 때 모든 종류의 정교한 옵션 지정 가능
    - `USING` (Hive : `STORED AS`)
    - 스파크는 포맷 미 지정 시, 기본적으로 하이브 SerDe 설정 사용 가능 (스파크 자체 직렬화보다 훨씬 느림)  
  - 테이블 특정 컬럼에 코멘트(`COMMENT`) 추가 가능
  - SELECT 쿼리 결과로 테이블 생성 가능
    - => CTAS (Create Table .. As .. Select ..) 패턴
  - 파티셔닝 데이터셋을 저장해 데이터 레이아웃 제어 가능 ([9장](https://minsw.github.io/2021/02/16/Spark-The-Definitive-Guide-9%EC%9E%A5/) 참고)
  - 스파크에 접속한 세션에서도 생성된 테이블 사용 가능
    - 사용자가 임시 뷰를 만들어서 사용 가능 (스파크에는 임시 테이블 X)

- 외부 테이블 생성하기
  - 스파크 SQL은 HiveQL 과 완벽하게 호환 
    - HiveQL 대부분 그대로 사용 가능
  - 스파크는 외부 테이블의 메타데이터 관리 (스파크에서 데이터 파일은 관리 X)
  - `CREATE EXTERNAL TABLE` 구문
  - 마찬가지로 쿼리 결과로 생성도 가능 (CTAS 패턴)

- 테이블에 데이터 삽입하기
  - `INSERT INTO` 구문 (표준 SQL 문법 사용)
  - 특정 파티션에만 저장하고 싶다면? => 파티션 명세 추가
    - 쓰기 연산은 파티셔닝 스키마에 맞게 데이터 저장 (단, 매우 느리게 동작할 수도)

- 테이블 메타데이터 확인하기
  - `DESCRIBE` 구문 : 테이블의 메타데이터 정보 반환
    - 테이블 생성 시 추가된 코멘트 확인 가능
  - `SHOW PARTITIONS` 으로 파티셔닝 스키마 정보 확인 가능 (파티션된 테이블인 경우)

- 테이블 메타데이터 갱신하기
  - 가장 최신의 데이터셋을 읽고 있다는 것을 보장하려면? => "테이블 메타데이터 유지" (중요!)
  - `REFRESH TABLE` 구문 : 테이블과 관련된 모든 캐싱된 항목 갱신 (기본적으로 파일)
    - 테이블이 이미 캐싱된 경우, 다음번 스캔 동작하는 시점에 다시 캐싱
  - `REPAIR TABLE` 구문 : 카탈로그에서 관리하는 테이블의 파티션 정보 새로고침
    - 새로운 파티션 정보 수집에 초점
    - ex. 수동으로 신규파티션을 만들면 테이블을 수리(repair) 해야함

- 테이블 제거하기
  - 테이블은 삭제(delete)할 수 없고, 오로지 **제거(drop)** 만 가능하다
  - `DROP TABLE` 구문
    - 관리형 테이블 제거 시, <U>**데이터와 테이블 정의 모두 제거**됨</U>
    - 존재하지 않는 테이블 제거 시 오류 발생 
    - => 테이블 존재 시에만 제거하는 `DROP TABLE IF EXISTS` 대신 사용
  - 외부 테이블 제거하기
    - 데이터는 삭제되지 않으나, 더는 외부 테이블 명으로 조회 불가

- 테이블 캐싱하기
  - `CACHE TABLE` 로 테이블을 캐시 가능
  - `UNCACHE TABLE` 로 캐시에서 제거 가능

### 10.7 뷰
- 뷰는 기존 테이블에 여러 트랜스포메이션 작업 지정
  - 기본적으로 뷰는 '단순 쿼리 실행 계획'
  - 쿼리 로직 체계화 & 재사용하기 편리
- 스파크가 가진 뷰에 관련된 다양한 개념
  - 데이터베이스에 설정하는 전역 뷰
  - 세션별 뷰
- 최종사용자 입장에서 뷰는 테이블처럼 보인다
  - 신규 경로에 모든 데이터를 다시 저장 X
  - 대신 단순하게 쿼리시점에 데이터소스에 트랜스포메이션 수행
  - 트랜스포메이션 예) filter, select, 대규모 group by, rollup

#### 뷰 다루기

- 뷰 생성하기
  - `CREATE VIEW` 구문 사용
  - 임시 뷰 생성 : `CREATE TEMP VIEW`
    - 현재 세션에서만 사용할 수 있는 임시 뷰
    - (테이블처럼 데이터베이스에 등록 X)
  - 전역적 임시 뷰(global temp view) 생성 : `CREATE GLOBAL TEMP VIEW`
    - 데이터베이스에 상관없이 사용 가능. 전체 스파크 애플리케이션에서 볼 수 O
    - 단, 세션 종료 시 뷰도 제거
  - 뷰 덮어쓰기 : `CREATE OR REPLACE TEMP VIEW` (임시뷰, 일반뷰 모두)
  - 뷰는 실질적으로 트랜스포메이션 이다
    - 스파크는 쿼리가 실행될때만 뷰에 정의된 트랜스포메이션 수행
    - 즉, 테이블의 데이터를 실제로 조회하는 경우에만 필터 적용
  - 뷰는 <U>기존 DataFrame에서 새로운 DataFrame 만드는 것과 동일</U>

- 뷰 제거하기
  - `DROP VIEW (IF EXISTS)` 사용
  - 뷰 제거 vs 테이블 제거
    - 뷰 정의는 제거되지만, **어떤 데이터도 제거되지 않음**
    - (테이블 제거는 데이터도 모두 제거됨)

### 10.8 데이터베이스
- 데이터 베이스 = 여러 테이블을 조직화하기 위한 도구
- 데이터베이스 미정의 시 스파크는 기본 데이터베이스 사용 (default)
- 스파크에서 실행하는 모든 SQL 명령문은 실행 중인 데이터베이스 범위에서 실행
  - <p style="color:lightgray"> 데이터베이스 변경 시, 이전에 생성한 모든 사용자 테이블은 변경 전 데이터베이스에 속해 있으므로 다르게 쿼리해야함 (뭔 소린가 했더니..)</p> 
  - => 즉, 다른 데이터베이스 사용 시 `데이터베이스 명 + 테이블 명` 으로 조회할 것
- `SHOW DATABASES` 으로 전체 데이터베이스 목록 확인

#### 데이터베이스 다루기

- 데이터베이스 생성하기
  - `CREATE DATABASE` 사용
- 데이터베이스 설정하기
  - `USE {databaseName}` 으로 쿼리 수행할 데이터베이스 설정
  - 다른 테이블에 쿼리 수행시 접두사 사용
  - `SELECT current_database()` : 현재 사용중인 데이터베이스 확인
- 데이터베이스 제거하기
  - `DROP DATABASE (IF EXISTS)` 사용
  - Q. 스파크 데이터베이스 삭제 시, 데이터는?

### 10.9 select 구문
- 스파크 SQL은 ANSI-SQL 요건 충족
  - SELECT 표현식 구조는 `p.296` 참고
- case...when...then 구문
  - SQL 쿼리 값을 조건에 맞게 처리 가능
  - (like if-else statement)

### 10.10 고급 주제
- 데이터 쿼리 방법 알아보기
  - SQL 쿼리 : 특정 명령 집합을 실행하도록 요청하는 SQL 구문
  - <U>조작, 정의, 제어</U> 와 관련된 명령 정의 가능
  - => 본 책은 대부분 **조작** 관련
- 복합 데이터 타입
  - 표준SQL에는 존재하지 않는 강력한 기능
  - 스파크 SQL의 핵심 복합 데이터 타입 3가지 => **구조체, 리스트, 맵**
  - 구조체 : 중첩 데이터 생성/쿼리 방법 제공
    - 맵에 가까움
  - 리스트 : 값의 배열이나 리스트 사용
    - 집계 함수 `collection_list()`, `collect_set()` 으로 생성 가능 (단, 집계 연산 시에만 사용 가능) 
    - `ARRAY` 로 컬럼에 직접 배열 생성 가능
    - `explode()` : 저장된 배열의 모든 값을 단일 로우 형태로 분해 (<-> `collect()`)
- 함수
  - 스파크 SQL은 다양한 고급 함수 제공 ([DataFrame 함수 문서](http://bit.ly/2DPAycx) 에서 확인)
  - `SHOW FUNCTIONS` 으로 스파크 SQL이 제공하는 전체 함수 목록 확인 가능  
    - `SHOW SYSTEM FUNCTIONS` : 스파크에 내장된 시스템 함수 목록
    - `SHOW USER FUNCTIONS` : 누군가가 스파크 환경에 공개한 함수 목록 (=  **사용자 정의 함수**)
    - 와일드카드 문자(*)로 필터링 가능
    - `LIKE` 키워드 사용 가능
  - `DESCRIBE FUNCTION {functionName}` : 개별 함수의 설명과 사용법 반환 
  - 사용자 정의 함수
    - 스파크는 사용자 정의 함수를 정의하여 분산 환경에서 사용할 수 있는 기능 제공
    - 특정 언어로 함수 개발 후 등록하여 정의
    - 함수 등록/정의 방법 => `spark.udf.register()`, Hive의 `CREATE TEMPORARY FUNCTION`
- 서브 쿼리
  - 서브 쿼리 (subquery) : 쿼리안에 쿼리 지정
    - SQL 내 정교한 로직 명시 가능
    - => 하나 (스칼라 서브쿼리) 이상의 결과 반환 가능?
  - 스파크의 기본 서브쿼리 2가지
    - **상호연관 서브쿼리** (correlated subquery) : 쿼리 외부 범위에 있는 일부 정보 사용 가능
    - **비상호연관 서브쿼리** (uncorrelated subquery) : 외부 범위 정보 사용 X
  - **조건절 서브쿼리** (predicate subquery) 도 지원 => 값에 따라 필터링
  - 비상호 스칼라 쿼리(uncorrelated scalar query) 사용 시
    - 기존에 없던 일부 부가 정보 가져오기 가능
    - (?)

### 10.11 다양한 기능
- 지금까지의 내용과 잘 안맞는 몇가지 특징?
  - SQL 코드 성능 최적화, 디버깅 케이스 등..
- 설정
  - 스파크 SQL 환경 설정 값 (`p.302-303 [표 10-1]` 참고)
  - 애플리케이션 초기화 시점 or 실행 시점에 설정 가능
- SQL에서 설정값 지정하기
  - `SET` 사용하여 SQL을 사용하여 환경 설정 가능 (단, 스파크 SQL 관련 설정에 한함)
  - SQL 설정은 15장에서 자세히

### 10.12 정리
- 스파크 SQL 관련 세부사항
- 스파크 SQL과 DataFrame은 매우 밀접한 연관

### 📒 단어장
- ANSI-SQL : 미국 표준 협회 ANSI (American National Standards Institute)에서 정립한 표준 SQL문
- 워크로드 (workloads) : 고객 대면 애플리케이션이나 백엔드 프로세스 같이 비즈니스 가치를 창출하는 리소스 및 코드 모음. 또는 말그대로 시스템이 실행해야할 할당 작업량.
- Thrift Server
  - Apache Thrift (RPC Framework)? Hive의 Thrift Server?
  - 스파크의 Thrift => 여러 사용자의 JDBC(or OCBC) 접속을 받아 사용자 쿼리를 원격으로 스파크 SQL 세션으로 실행하는 스파크 애플리케이션
    - [참고 링크](https://thebook.io/006908/part02/ch05/03/03/)
- OLTP vs OLAP
  - OLTP (OnLine Transaction Processing, 온라인 트랜잭션 처리) 는 DB서버에서 각각의 작업요청의 트랜잭션 처리 (CUD 무결성 보장) & 결과 READ 과정
  - OLAP (OnLine Analytic Processing, 온라인 분석용) 는 저장된 데이터를 바탕으로 요구와 목적에 맞는 분석 정보 제공
  - 즉, OLTP는 데이터 처리 중심, OLAP는 저장된 데이터 분석 중심
- 인플레이스 수정(in-place modifiction) 방식
  - in-place update?