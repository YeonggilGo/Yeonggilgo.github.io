---
layout: post
title: "Spark Study 2. 스파크 기초"
thumbnail-img: /assets/img/spark.png
tags: [spark]
comments: true
categories: [Spark]
gh-repo: yeonggilgo/spark-practice
gh-badge: [star, fork, follow]
---

## 텍스트 파일 읽고 출력해보기

```python
# sc : Spark Context in pyspark shell
lines = sc.textFile("/path")
bsd_lines = lines.filter(lambda line: "BSD" in line)
bsd_lines.count()

from __future__ import print_function
bsd_lines.foreach(lambda b_line: print(b_line))

def isBSD(line):
  return "BSD" in line

bsd_lines = lines.filter(isBSD)
bsd_lines.count()
bsd_lines.foreach(lambda b_lines: print(b_line))
```



<br>

## RDD 개념

#### 성질

- 불변성(Immutable): Read-only
- 복원성(Resilient): 장애 내성
- 분산(distributed): 노드 한 개 이상에 저장된 데이터셋



 RDD는 데이터를 조작 할 수 있는 다양한 메서드를 제공하고 이 메서드들은 항상 새로운 RDD를 생성한다.(불변성) RDD의 목적은 분산 컬렉션의 성질과 장애 내성을 추상적화하고 직관적으로 연산 수행을 가능케 한다.

 RDD는 노드에 데이터셋을 중복 저장하지 않고 데이터셋을 만드는 데 사용된 변환 연산자의 로그를 남기는 방식으로 복원성을 제공한다. 노드에 장애가 발생하면 해당 노드가 가진 데이터 셋만 다시 계산해 RDD를 복원한다.

<br>

## RDD의 행동 연산자와 변환 연산자

 RDD 연산자는 크게 행동(action)과 변환(transformation), 두 유형으로 나뉜다
- Transformation: RDD의 데이터를 조작해 새로운 RDD를 생성(ex: filter, map)
- Action: 연산자를 호출한 프로그램으로 계산 결과를 반환하거나 RDD요소에 특정 작업을 수행하려고 실제 계산을 시작하는 역할(ex: count, foreach)

#### Lazy evaluation 
&nbsp;스파크의 변환 연산자의 지연 실행(Lazy evaluation)은 행동 연산자를 호출하기 전까지 변환 연산자의 계산을 실제로 실행하지 않는 것을 의미한다. action이 호출 되면 스파크는 해당 RDD계보를 보고 **연산 그래프**를 작성하여 action을 계산한다. 결국 transformation은 action을 호출 했을 때 무슨 연산이 어떤 순서로 실행 되어야 할지 알려주는 설계도라고 할 수 있다.



<br>

### `map`(trans)

RDD의 모든 요소에 임의의 함수를 적용하는 변환 연산자

```python
numbers = sc.parallelize(range(10, 51, 10))
numbers.foreach(lambda x: print(x))
numbersSquared = numbers.map(lambda num: num * num)
numbersSquared.foreach(lambda x: print(x))

reversed = numbersSquared.map(lambda x: str(x)[::-1])
reversed.foreach(lambda x: print(x))
```

<br>

### `distinct` & `flatMap`

`flatMap`은 `map`과 동일하지만 익명 함수가 반환한 배열의 중첩구조를 한단계 제거하고 모든 배열의 요소를 단일 컬렉션으로 병합한다.

`distinct`는 RDD의 중복을 제거해준다.

```python
# *************** terminal ******************
# echo "15,16,20,20
# 77,80,94
# 94,98,16,31
# 31,15,20" > ~/client-ids.log
# ************* end terminal *****************

lines = sc.textFile("/home/spark/client-ids.log")

idsStr = lines.map(lambda line: line.split(","))
idsStr.foreach(lambda x: print(x))
ids = lines.flatMap(lambda x: x.split(","))
intIds = ids.map(lambda x: int(x))

uniqueIds = intIds.distinct()
finalCount  = uniqueIds.count()
transactionCount = ids.count()
```

<br>

### `sample` & `take` & `takeSample`

`sample(replacement: bool, fraction: double, seed: long)`은 replacement에 따른 복원 샘플링 여부, fraction(복원에서는 샘플링 될 횟수의 기댓값, 비복원에서는 요소가 샘플링될 기대 확률, ex 0.3=30%)만큼 RDD 데이터를 샘플링 해준다. 비복원에서 8개의 데이터 중 0.3 확률로 sample하면 2.4개 이지만 확률이기 때문에 한개나 세개도 반환 할 수도 있다.

`takeSample(replacement: bool, num: int, seed: long) `은 정수형 변수로 항상 정확한 갯수를 샘플링 한다. `sample`은 변환 연산자 이지만 `takeSample`은 행동 연산자로 RDD를 반환한다.

 `take`는 지정된 개수를 모을 때 까지 RDD의 파티션을 하나씩 처리해 반환한다. RDD의 데이터를 엿볼 때 유용하다.

<br>

### Double RDD 전용함수

> Double 객체만 사용해 RDD를 구성하면 암시적 변환을 이용해 추가함수를 이용 가능하다.
>
> python에서는 암시적 변환이 좀 더 융통성 있어서 int, float 등에서도 사용이 가능한 것 같다.



- `mean` : 평균
- `sum` : 합계
- `variance` : 분산
- `stdev` : 표준편차
- `stats` : 위 네개 전부

<br>

### `histogram`

`histogram`은 데이터를 시각화 때 사용 한다. 행동연산자로 두가지 버전이 있다.

```python
# 첫 번째 버전
# 오름차순의 Double list를 입력받아 각 구간에 속한 요소의 개수를 반환한다.
intIds.histogram([1.0, 50.0, 100.0])

# 두 번째 버전
# 구간 개수를 입력받아 입력 데이터의 전체 범위를 균등하게 나눈 후 요소 두개로 구성된 튜플을 반환한다.
# (구간 개수로 계산된 경계의 배열, 요소 개수 배열)
intIds.histogram(3)
```

<br>

### `sumApprox`  & `meanApprox`

위 두 함수는 실험적으로 제공하는 메서드로 지정 제한 시간(`Long`) 동안 근사 합계, 근사 평균을 계산한다. 대규모 데이터셋을 다룰 때 유용하다.
