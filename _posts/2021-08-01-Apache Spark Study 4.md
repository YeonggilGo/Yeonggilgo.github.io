---
layout: post
title: "Spark Study 4. DataFrame"
thumbnail-img: /assets/img/spark.png
tags: [spark]
comments: true
categories: [Spark]
gh-repo: yeonggilgo/spark-practice
gh-badge: [star, fork, follow]
---

&nbsp;RDD는 데이터를 직접 다룰 수 있는 스파크 하위 레벨 인터페이스, DataFrame은 칼럼이름과 타입이 지정된 테이블 형식의 분산 정형 데이터를 손쉽게 다룰 수 있는 상위 레벨 인터페이스를 제공한다. 
&nbsp;정형 데이터는 데이터 구조를 미리 파악 할 수 있고 컬럼으로 데이터를 참조하거나 SQL 쿼리를 이용해 데이터에 접근 할 수 있어 데이터를 손쉽게 다룰 수 있게 해준다.

<br>

### 생성 방법 : 파일(csv, parquet, txt...)으로부터

&nbsp;데이터프레임을 생성하는 방법은 RDD에서 변환, 스키마를 직접 설정하여 생성 등 여러가지 있지만 파일로 부터 생성하는 방법을 자주 사용하기 때문에 이 방법만 포스팅 하겠다. 먼저 `SparkSession` 객체가 필요하다. 스파크 셸에서는 `spark`라는 이름의 변수로 제공 되는 이 객체는 `SparkContext` 와 `SQLContext`를통합한 Wrapper 클래스의 인스턴스이다. 

```python
spark = SparkSession.builder().getOrCreate()
itPostsRows = spark.read.csv("itatlianPosts.csv")
```

<br>

## DataFrame API 

> &nbsp;DataFrame은 RDD와 마찬가지로 불변성과 지연 실행 특징을 가지고 있다. 매핑, 그루핑 등의 다양한 DataFrame 함수를 살펴보자.



### `select`

&nbsp;특정 칼럼을 선택한다.

```python
import spark.sql.functions as F

# 아래 세가지 방법 다 같은 결과
postsIdBody = postsDf.select("id", "body")
postsIdBody = postsDf.select(postsDf["id"], postsDf["body"])
postsIdBody = postsDf.select(F.col("id"), F.col("body"))
```

<br>

### `drop`

&nbsp;DataFrame에서 특정 컬럼을 삭제한다.

`postsIdBody.drop("body")`

<br>

### `filter` & `where`

&nbsp;데이터 필터링 함수로 동일하게 동작한다. 컬럼 객체 또는 문자열 표현식을 인수로 전달 받는다. 문자열 표현식에는 SQL 구문을 전달하면 된다. 

```python
import spark.sql.functions as F

# 문자열 포함
postsIdBody.filter(F.instr(postsIdBody["body"], "Italiano")).count()
postsIdBody.filter(F.col("body").contains("Italiano")).count()

# 조건식, 괄호를 적절히 사용하여야 함
noAnswer = postsDf.filter((postsDf["postTypeId"] == 1) & isnull(postsDf["acceptedAnswerId"]))
```

&nbsp; 이외에도 다양한 Column 클래스의 메서드들이 존재한다. [스파크 공식문서](http://mng.bz/a2Xt)

<br>

### `withColumn` & `withColumnRenamed`

&nbsp;컬럼이름과 표현식을 전달하여 신규 칼럼을 추가하거나 컬럼이름을 변경 할 수 있다.

```python
firstTenQsRn = firstTenQs.withColumnRenamed("ownerUserId", "owner")

postsDf.filter(postsDf.postTypeId == 1).withColumn("ratio", postsDf.viewCount / postsDf.score).where("ratio < 35").show()
```

<br>

### `orderBy` & `sort`

&nbsp;데이터를 한 개 이상의 컬럼 또는 표현식을 기준으로 정렬 할 수 있다. 오름차순이 기본값이다. 두개가 동일하게 동작한다.

```python
postsDf.filter(postsDf.postTypeId == 1).orderBy(postsDf.lastActivityDate.desc()).limit(10).show()
```

<br>

> 이외에도 스칼라 함수로 `add, sum, abs, log, length ` 등 다양한 함수를 제공한다. groupBy나 withColumn과 같이 사용 되는 일이 잦으며 너무 유명한 기능들은 포스팅에서 제외한다.

```python
# 날짜 차이로 정렬하는 예
postsDf.filter(postsDf.postTypeId == 1).withColumn("activePeriod", datediff(postsDf.lastActivityDate, postsDf.creationDate)).orderBy(desc("activePeriod")).head().body.replace("&lt;","<").replace("&gt;",">")
```

<br>



### 윈도 함수

&nbsp; 윈도 함수는 집계 함수와 유사하지만 로우를 단일 결과로만 그루핑하지 않고 윈도 함수에 따라 움직이는 그룹, 즉 **프레임**으로 그루핑한다. 프레임은 현재 처리하는 로우와 관련된 다른 로우 집합을 정의하며, 이 집합을 현재 로우 계산에 활용 할 수 있다. 이를 이용하여 서브 select쿼리나 조인 연산이 필요한 계산을 간단하게 계산할 수 있다. 

&nbsp;윈도 함수를 사용하려면 먼저 집계 함수 (`min, sum, avg, first, ntile, rank, row_number` 등)을 `Column`의`over` 함수에 인수로 전달해야 한다. `over`는 이 `WindowSpec`을 사용하는 윈도 컬럼을 정의해 반환한다.

&nbsp;`WindowSpec`은 세가지 방법으로 만들 수 있는데 `partitionBy`를 사용해 `groupBy`처럼 분할기준으로 칼럼을 분할하거나, `orderBy` 로 로우를 정렬할 기준을 지정하거나, 둘다 사용하는 것이다. `rowsBetween(from, to)` 를 사용해 프레임에 포함될 로우를 추가적으로 제한 할 수 있다. 

```python
# 사용자가 올린 질문 중 최고점수를 계산하고, 해당 사용자의 게시물 중 다른 질문의 점수과 최고 점수 간 차이
winDf = postsDf.filter(postsDf.postTypeId == 1)\
	.select(postsDf.ownerUserId, postsDf.acceptedAnswerId, postsDf.score, max(postsDf.score)\
          .over(Window.partitionBy(postsDf.ownerUserId)).alias("maxPerUser"))
  
winDf.withColumn("toMax", winDf.maxPerUser - winDf.score).show(10)
# +-----------+----------------+-----+----------+-----+
# |ownerUserId|acceptedAnswerId|score|maxPerUser|toMax|
# +-----------+----------------+-----+----------+-----+
# |        232|            2185|    6|         6|    0|
# |        833|            2277|    4|         4|    0|
# |        833|            null|    1|         4|    3|
# |        235|            2004|   10|        10|    0|
# |        835|            2280|    3|         3|    0|
# |         37|            null|    4|        13|    9|
# |         37|            null|   13|        13|    0|
# |         37|            2313|    8|        13|    5|
# |         37|              20|   13|        13|    0|
# |         37|            null|    4|        13|    9|
# +-----------+----------------+-----+----------+-----+

# 질문의 생성 날짜 기준, 질문자가 한 바로 전 질문과 바로 다음 질문의 ID 출력
# lag와 lead는 각각 현재 로우의 전, 후 로우를 가져온다.
postsDf.filter(postsDf.postTypeId == 1)\
	.select(postsDf.ownerUserId, postsDf.id, postsDf.creationDate,
          lag(postsDf.id, 1).over(Window.partitionBy(postsDf.ownerUserId).orderBy(postsDf.creationDate)).alias("prev"), 
          lead(postsDf.id, 1).over(Window.partitionBy(postsDf.ownerUserId).orderBy(postsDf.creationDate)).alias("next"))\
  .orderBy(postsDf.ownerUserId, postsDf.id).show()
# +-----------+----+--------------------+----+----+
# |ownerUserId|  id|        creationDate|prev|next|
# +-----------+----+--------------------+----+----+
# |          4|1637|2014-01-24 06:51:...|null|null|
# |          8|   1|2013-11-05 20:22:...|null| 112|
# |          8| 112|2013-11-08 13:14:...|   1|1192|
# |          8|1192|2013-11-11 21:01:...| 112|1276|
# |          8|1276|2013-11-15 16:09:...|1192|1321|
# |          8|1321|2013-11-20 16:42:...|1276|1365|
# |          8|1365|2013-11-23 09:09:...|1321|null|
# |         12|  11|2013-11-05 21:30:...|null|  17|
# |         12|  17|2013-11-05 22:17:...|  11|  18|
# |         12|  18|2013-11-05 22:34:...|  17|  19|
# |         12|  19|2013-11-05 22:38:...|  18|  63|
# |         12|  63|2013-11-06 17:54:...|  19|  65|
# |         12|  65|2013-11-06 18:07:...|  63|  69|
# |         12|  69|2013-11-06 19:41:...|  65|  70|
# |         12|  70|2013-11-06 20:35:...|  69|  89|
# |         12|  89|2013-11-07 19:22:...|  70|  94|
# |         12|  94|2013-11-07 20:42:...|  89| 107|
# |         12| 107|2013-11-08 08:27:...|  94| 122|
# |         12| 122|2013-11-08 20:55:...| 107|1141|
# |         12|1141|2013-11-09 20:50:...| 122|1142|
# +-----------+----+--------------------+----+----+
```

&nbsp;더 다양한 SQL 함수는 [스파크 공식문서](https://goo.gl/nu6mwT)를 참고하자.



### 사용자 정의 함수

&nbsp;간혹 스파크 SQL에서 지원하지 않는 특정 기능이 필요할 때는 사용자 정의 함수(UDF)를 사용하여 기능 확장이 가능하다.

```python
# '&lt'와 '&gt'로 둘러싸인 태그의 수를 세는 udf
countTags = F.udf(lambda (tags): tags.count("&lt;"), IntegerType())
postsDf.filter(postsDf.postTypeId == 1)\
	.select("tags", countTags(postsDf.tags).alias("tagCnt"))\
  .show(10, False)
# +-------------------------------------------------------------------+------+
# |tags                                                               |tagCnt|
# +-------------------------------------------------------------------+------+
# |&lt;word-choice&gt;                                                |1     |
# |&lt;english-comparison&gt;&lt;translation&gt;&lt;phrase-request&gt;|3     |
# |&lt;usage&gt;&lt;verbs&gt;                                         |2     |
# |&lt;usage&gt;&lt;tenses&gt;&lt;english-comparison&gt;              |3     |
# |&lt;usage&gt;&lt;punctuation&gt;                                   |2     |
# |&lt;usage&gt;&lt;tenses&gt;                                        |2     |
# |&lt;history&gt;&lt;english-comparison&gt;                          |2     |
# |&lt;idioms&gt;&lt;etymology&gt;                                    |2     |
# |&lt;idioms&gt;&lt;regional&gt;                                     |2     |
# |&lt;grammar&gt;                                                    |1     |
# +-------------------------------------------------------------------+------+
```



## 데이터 그루핑

&nbsp;SQL의 GROUP BY와 비슷하게 동작한다. `groupBy` 함수를 통해 Column 목록을 전달 받고 GroupedData를 반환한다. 이 객체는 집계 함수를 통해 DataFrame을 반환한다.

```python
postsDfNew.groupBy(postsDfNew.ownerUserId, postsDfNew.tags, postsDfNew.postTypeId).count().orderBy(postsDfNew.ownerUserId.desc()).show(10)
#+-----------+--------------------+----------+-----+
#|ownerUserId|                tags|postTypeId|count|
#+-----------+--------------------+----------+-----+
#|        862|                    |         2|    1|
#|        855|         <resources>|         1|    1|
#|        846|<translation><eng...|         1|    1|
#|        845|<word-meaning><tr...|         1|    1|
#|        842|  <verbs><resources>|         1|    1|
#|        835|    <grammar><verbs>|         1|    1|
#|        833|                    |         2|    1|
#|        833|           <meaning>|         1|    1|
#|        833|<meaning><article...|         1|    1|
#|        814|                    |         2|    1|
#+-----------+--------------------+----------+-----+

postsDfNew.groupBy(postsDfNew.ownerUserId).agg(max(postsDfNew.lastActivityDate), max(postsDfNew.score)).show(10)
postsDfNew.groupBy(postsDfNew.ownerUserId).agg({"lastActivityDate": "max", "score": "max"}).show(10)
# +-----------+---------------------+----------+
# |ownerUserId|max(lastActivityDate)|max(score)|
# +-----------+---------------------+----------+
# |        431| 2014-02-16 14:16:...|         1|
# |        232| 2014-08-18 20:25:...|         6|
# |        833| 2014-09-03 19:53:...|         4|
# |        633| 2014-05-15 22:22:...|         1|
# |        634| 2014-05-27 09:22:...|         6|
# |        234| 2014-07-12 17:56:...|         5|
# |        235| 2014-08-28 19:30:...|        10|
# |        435| 2014-02-18 13:10:...|        -2|
# |        835| 2014-08-26 15:35:...|         3|
# |         37| 2014-09-13 13:29:...|        23|
# +-----------+---------------------+----------+

postsDfNew.groupBy(postsDfNew.ownerUserId).agg(max(postsDfNew.lastActivityDate), max(postsDfNew.score) > 5).show(10)
# +-----------+---------------------+----------------+
# |ownerUserId|max(lastActivityDate)|(max(score) > 5)|
# +-----------+---------------------+----------------+
# |        431| 2014-02-16 14:16:...|           false|
# |        232| 2014-08-18 20:25:...|            true|
# |        833| 2014-09-03 19:53:...|           false|
# |        633| 2014-05-15 22:22:...|           false|
# |        634| 2014-05-27 09:22:...|            true|
# |        234| 2014-07-12 17:56:...|           false|
# |        235| 2014-08-28 19:30:...|            true|
# |        435| 2014-02-18 13:10:...|           false|
# |        835| 2014-08-26 15:35:...|           false|
# |         37| 2014-09-13 13:29:...|            true|
# +-----------+---------------------+----------------+
```

