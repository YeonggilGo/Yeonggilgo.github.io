---
layout: post
title: "Spark Study 3. UDF, Pair RDD, Partitioning"
thumbnail-img: /assets/img/spark.png
tags: [spark]
comments: true
categories: [Spark]
gh-repo: yeonggilgo/spark-practice
gh-badge: [star, fork, follow]
---

## UDF(User Defined  Function): 사용자 정의 함수

&nbsp; SparkSession 클래스에 사용자 정의 메서드를 등록 할 수 있다.

```python
sc   = SparkContext(conf=SparkConf())
ghLog = sqlContext.read.json(sys.argv[1])
pushes = ghLog.filter("type = 'PushEvent'")
grouped = pushes.groupBy("actor.login").count()
ordered = grouped.orderBy(grouped['count'], ascending=False)

# Broadcast the employees set	
employees = [line.rstrip('\n') for line in open(sys.argv[2])]
bcEmployees = sc.broadcast(employees)

def isEmp(user):
	return user in bcEmployees.value

sqlContext.udf.register("SetContainsUdf", isEmp, returnType=BooleanType())
filtered = ordered.filter("SetContainsUdf(login)")
filtered.write.format(sys.argv[4]).save(sys.argv[3])
```

<br>

## 공유 변수

&nbsp;공유 변수는 클러스터의 각 노드에 **정확히 한 번만** 전송한다. 또 노드의 메모리에 자동으로 캐시 되므로 프로그램 실행 중에 바로 접근 할 수 있다. 공유 변수는 `value` 를 사용해 접근 해야 한다.

`bcEmployees = sc.broadcast(employees)`

<br>

## Pair-RDD

> Key-value는 간단하고 확상성이 뛰어난 데이터 모델 이다. 파이썬에서는 Dictionary로 사용한다. 스파크에서는 key-value tuple으로 구성 된 RDD를 Pair RDD라고 한다. Pair RDD를 사용하면 데이터를 편리하게 집계 / 정렬 / 조인 할 수 있다.

<br>

### 생성

&nbsp; 스파크에서는 다양한 방법으로 Pair RDD를 생성 할 수 있으며 일부 메서드는 Pair RDD를 기본으로 반환한다. RDD의 ByKey 변환 연산자는 RDD요소로 키를 생성 하는 f 함수를 받고, 각요소를 `f(요소), 요소)`쌍의 튜플로 매핑한다. 

<br>

### 기본 함수

&nbsp;쇼핑몰에서 고객에게 선별적으로 사은품을 보내려고 한다. 기획자는 어제 날짜의 구매기록을 읽어 들여 다음의 규칙에 따라 사은품을 추가하는 프로그램을 개발해 달라고 요청했다. 

- 구매 횟수가 가장 많은 고객에게 곰인형
- 바비 쇼핑몰 놀이 세트를 두 개 이상 두개하면 5% 할인
- 사전을 5권 이상 구매하면 칫솔
- 가장 많은 금액을 지출한 고객에게 커플 잠옷 세트

&nbsp;사은품은 구매 금액이 0달러인 추가 거래로 기입해야 한다. 기획자는 사은품을 받는 고객이 어떤 상품을 구매 했는지 알고 싶다.

```python
# 파일 읽고 Pair RDD 생성
tranFile = sc.textFile("path/to/ch04_data_transactions.txt")
tranData = tranFile.map(lambda line: line.split("#"))
transByCust = tranData.map(lambda t: (int(t[2]), t)) # 고객 ID를 키로 설정

transByCust.keys().distinct().count() # 중복 제거 개수 : 고객수
```

<br>

### `countByKey`

 구매 횟수가 가장 많은 고객을 찾아 곰 인형을 보내자

```python
import operator
transByCust.countByKey()

>>>
defaultdict(<class 'int'>, {1: 9, 2: 15, 3: 13, 4: 11, 5: 11, 6: 7, 7: 10, 8: 10, 9: 7, 10: 7, 11: 8, 12: 7, 13: 12, 14: 8, 15: 10, 16: 8, 17: 13, 18: 9, 19: 6, 20: 8, 21: 13, 22: 10, 23: 13, 24: 9, 25: 12, 26: 11, 27: 7, 28: 11, 29: 9, 30: 5, 31: 14, 32: 14, 33: 9, 34: 14, 35: 10, 36: 5, 37: 7, 38: 9, 39: 11, 40: 10, 41: 12, 42: 7, 43: 12, 44: 8, 45: 11, 46: 9, 47: 13, 48: 5, 49: 8, 50: 14, 51: 18, 52: 9, 53: 19, 54: 7, 55: 13, 56: 17, 57: 8, 58: 13, 59: 9, 60: 4, 61: 8, 62: 6, 63: 12, 64: 10, 65: 10, 66: 11, 67: 5, 68: 12, 69: 7, 70: 8, 71: 10, 72: 7, 73: 7, 74: 11, 75: 10, 76: 15, 77: 11, 78: 11, 79: 13, 80: 7, 81: 9, 82: 13, 83: 12, 84: 9, 85: 9, 86: 9, 87: 10, 88: 5, 89: 9, 90: 8, 91: 13, 92: 8, 93: 12, 94: 12, 95: 8, 96: 8, 97: 12, 98: 11, 99: 12, 100: 12})
>>>
```

SQL에 익숙하여 `groupBy().count()` 를 자주 사용하였는데 훨씬 쉽게 사용 가능하다.

```python
sum(transByCust.countByKey().values())
>>> 
1000
```

`sum()` 을 사용하여 검산 가능하다.

```python
# 구매 횟수가 가장 많은 고객
(cid, purch) = sorted(transByCust.countByKey().items(), key=operator.itemgetter(1))[-1]
>>>
(53, 19)

# 곰 인형을 추가 구매 기록
complTrans = [["2015-03-30", "11:59 PM", "53", "4", "1", "0.00"]]
```

<br>

### `lookup`

`lookup` 을 사용하여 단일 키의 모든 구매 기록을 가져 올 수 있다.

```python
# 사은품 받을 고객의 구매 기록
transByCust.lookup(53)
for t in transByCust.lookup(53):
    print(", ".join(t))
```

<br>

### `mapValues`

&nbsp;바비 쇼피몰 놀이 세트를 두개 이상 구매하면 할인 해주기 위해 특정 라인의 value만 `mapValues` 를 사용하여 변경 하자.

```python
def applyDiscount(tran):
    if(int(tran[3])==25 and float(tran[4])>1):
        tran[5] = str(float(tran[5])*0.95)
    return tran
transByCust = transByCust.mapValues(lambda t: applyDiscount(t))
```

<br>

### `flatMapValues`

&nbsp;사전을 5권 이상 구매한 고객에게 칫솔을 보내자. `flatMapValues` 를 사용하면 키-값에 새로운 값을 넣거나 삭제, 추가가 가능하다.

```python
def addToothbrush(tran):
    if(int(tran[3]) == 81 and int(tran[4])>4):
        from copy import copy
        cloned = copy(tran)
        cloned[5] = "0.00"
        cloned[3] = "70"
        cloned[4] = "1"
        return [tran, cloned]
    else:
        return [tran]
transByCust = transByCust.flatMapValues(lambda t: addToothbrush(t))
```

<br>

### `reduceByKey` & `foldByKey`

&nbsp; `reduceByKey` 는 각 키의 모든 값을 동일한 타입의 단일 값으로 병합한다. merge 함수를 전달 받아 각 키별로 값.하나만 남을 때까지 merge함수를 호출 한다.

&nbsp;`foldByKey` 는 `reduceByKey` 와 기능을 같지만 merge 함수의 인자 목록 바로 앞에 zeroValue 인자를 담은 또 다른 인자 목록을 추가로 전달 받는다. zeroValue는 반드시 merge 함수의 항등원이어야 한다. 하지만 RDD 연산이 병렬로 실행 되기 때문에 zeroValue가 여러번 쓰일 수 있다는 점을 주의해야 한다.



```python
# 가장 구매 금액이 큰 고객
amounts = transByCust.mapValues(lambda t: float(t[5]))
totals = amounts.foldByKey(0, lambda p1, p2: p1 + p2).collect()
```

<br>

```python
# 구매 목록 합치기
complTrans += [["2015-03-30", "11:59 PM", "76", "63", "1", "0.00"]]
transByCust = transByCust.union(sc.parallelize(complTrans).map(lambda t: (int(t[2]), t)))

# 파일로 저장
transByCust.map(lambda t: "#".join(t[1])).saveAsTextFile("/destination")
```

<br>

### `aggregateByKey`

&nbsp;zeroValue를 받는 다는 점에서 `foldByKey` 와 비슷하지만 값 타입을 바꿀 수 있다는 점이 다르다.

```python
# 각 고객의 구매 목록을 리스트 타입으로
prods = transByCust.aggregateByKey([], lambda prods, tran: prods + [tran[3]],
    lambda prods1, prods2: prods1 + prods2)
prods.collect()
```



<br>

## 데이터 파티셔닝 & 셔플링 최소화

&nbsp;**데이터 파티셔닝**은 데이터를 여러 클러스터 노드로 분할하는 메커니즘을 의미한다. 스파크의 성능과 리소스 점유량을 크게 좌우 한다. **셔플링** 또한 RDD 연산자의 상당수가 연관된다. 

 RDD의 **파티션**은 RDD 데이터의 일부를 의미한다. RDD의 파티션 개수가 변환 연산을 실행할 태스크 개수와 직결되기 때문에 매우 중요하다. 너무 적으면 클러스터를 활용할 수 없고,  태스크가 처리할 데이터 분량이 실행자의 메모리를 초과 할 수도 있다. 따라서 클러스터의 코어 개수보다 서너 배 많은 파티션을 사용하는 것이 좋다. 너무 많으면 태스크 관리 작업에 병목이 일어난다. 

<br>

### Partitioner

&nbsp; 파티셔닝은 RDD 각 요소에 번호를 할당하는 Partitioner 객체가 수행한다. 

- HashPartitioner

  &nbsp;기본 파티셔너로 해시코드를 mod 공식에 대입해 파티션 번호를 계산한다. 번호를 거의 무작위로 결정하여 정확하게 같은 크기로 파티션을 분할할 가능성이 낮다. 하지만 대규모 데이터를 상대적으로 적은 수의 파티션으로 나누면 대체로 고르게 분산 시킬 수 있다.

- RangePartitioner

  &nbsp;정렬된 RDD의 데이터를 거의 같은 범위 간격으로 분할할 수 있다. 직접 사용할 일은 그리 많지 않다.
  
- 사용자 정의 Partitioner

  &nbsp;파티션의 데이터를 특정 기준에 따라 정확하게 배치해야 할 경우 사용한다. Pair RDD에서만 사용 가능하다.

<br>

### 불필요한 셔플링 줄이기

&nbsp; **셔플링**은 파티션간 물리적인 데이터 이동을 의미한다. 새로운 RDD의 파티션을 만들려고 여러 파티션의 데이터를 합칠 때 발생한다. 예를 들어 키를 기준으로 그루핑하려면 RDD의 파티션을 모두 살펴보고 키가 같은 요소를 전부 찾은 후 물리적으로 묶어서 새로운 파티션을 구성하는 과정을 수행해야 한다. 

 셔플링 전에 각 파티션에서 **맵** 태스크에서 부분적으로 변환하고 셔플링 후 **리듀스** 태스크에서 결과물을 파티션에 할당한다. 맵 태스크의 결과는 캐시에만 저장한다. RDD 변환 연산에는 대부분 셔플링이 필요하지 않지만 일부 연산에서는 특정 조건하에서 셔플링이 발생하기도 한다. 따라서 셔플링을 최소화 하려면 다음 특이 조건들을 잘 이해해야 한다.

- Partitioner를 명시적으로 변경하는 경우: 사용자 정의 Partitioner를 사용하면 반드시 셔플링 발생, 파티션 개수가 다른 파티셔너로 변경하면 셔플링 발생. 
- Partitioner를 제거하는 경우: `map` & `flatMap` 은 RDD의 파티셔너를 제거한다. 연산자 자체로는 셔플링을 발생하지 않지만 연산자의 결과 RDD에 다른 변환 연산자를 사용하면 셔플링이 발생한다. 아래는 셔플링이 발생하는 변환 연산자이다.
  - `reduceByKey, join, groupByKey, subtractByKey`
  - `subtract, intersection, groupWith`
  - `sortByKey`
  - `partitionBy, coalesce(shuffle=True)`
- 외부 셔플링 서비스로 셔플링 최적화: 중간 셔플파일을 읽을 수 있는 단일 지점을 제공하여 셔플링 최적화.



