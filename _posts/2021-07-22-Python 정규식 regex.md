---
layout: post
title: "Python 정규식: re"
# cover-img: /assets/img/path.jpg
thumbnail-img: /assets/img/regex.png
# share-img: /assets/img/path.jpg
tags: [python, regex]
comments: true
categories: [Python]
# gh-repo: yeonggilgo/pyspark-exercise
---

# Python Regex

 정규식은 regex 패턴이라고도 불리는 일치시키려는 문자열 집합에 대한 규칙을 지정할 수 있다. Pyhton에서는 `re` 모듈로 정규식을 사용한다. `re.complie`함수는 정규식 패턴을 입력으로 받아 정규식 객체 `re.RegexObject`를 반환 한다. [Official Docs](https://docs.python.org/ko/3/howto/regex.html#search-and-replace) 로 학습한 내용을 기록하였다. 



## 1. 문자 일치

 대부분 문자는 단순히 자신과 일치 합니다. 예외적으로 특수한 **메타 문자**를 이용하여 일반적이지 않은 것을 매치할 수 있다.



### 메타 문자

| 패턴       | 설명                                                         | 예제                                                  |
| :--------- | :----------------------------------------------------------- | :---------------------------------------------------- |
| ^          | 이 패턴으로 시작해야 함                                      | ^abc : abc로 시작해야 함 (abcd, abc12 등)             |
| $          | 이 패턴으로 종료되어야 함                                    | xyz$ : xyz로 종료되어야 함 (123xyz, strxyz 등)        |
| [문자들]   | 문자들 중에 하나이어야 함. 가능한 문자들의 집합을 정의함.    | [Pp]ython : "Python" 혹은 "python"                    |
| [^문자들]  | [문자들]의 반대로 피해야할 문자들의 집합을 정의함.           | [^aeiou] : 소문자 모음이 아닌 문자들                  |
| \|         | 두 패턴 중 하나이어야 함 (OR 기능)                           | a \| b : a 또는 b 이어야 함                           |
| ?          | 앞 패턴이 없거나 하나이어야 함 (Optional 패턴을 정의할 때 사용) | \d? : 숫자가 하나 있거나 없어야 함                    |
| +          | 앞 패턴이 하나 이상이어야 함                                 | \d+ : 숫자가 하나 이상이어야 함                       |
| *          | 앞 패턴이 0개 이상이어야 함                                  | \d* : 숫자가 없거나 하나 이상이어야 함                |
| 패턴{n}    | 앞 패턴이 n번 반복해서 나타나는 경우                         | \d{3} : 숫자가 3개 있어야 함                          |
| 패턴{n, m} | 앞 패턴이 최소 n번, 최대 m 번 반복해서 나타나는 경우 (n 또는 m 은 생략 가능) | \d{3,5} : 숫자가 3개, 4개 혹은 5개 있어야 함          |
| \d         | 숫자 0 ~ 9                                                   | \d\d\d : 0 ~ 9 범위의 숫자가 3개를 의미 (123, 000 등) |
| \w         | 문자를 의미                                                  | \w\w\w : 문자가 3개를 의미 (xyz, ABC 등)              |
| \s         | 화이트 스페이스를 의미하는데, [\t\n\r\f] 와 동일             | \s\s : 화이트 스페이스 문자 2개 의미 (\r\n, \t\t 등)  |
| .          | 뉴라인(\n) 을 제외한 모든 문자를 의미                        | .{3} : 문자 3개 (F15, 0x0 등)                         |



- 정규식에서 `( )`는 Group을 의미합니다

```python
import re
 
text = "문의사항이 있으면 032-232-3245 으로 연락주시기 바랍니다."
 
regex = re.compile(r'(\d{3})-(\d{3}-\d{4})')
matchobj = regex.search(text)
areaCode = matchobj.group(1)
num = matchobj.group(2)
fullNum = matchobj.group()
print(areaCode, num) # 032 232-3245
```



### 반복

 아래의 메타 문자들로 반복적인 문자열을 매치할 수 있습니다.

| 패턴          | 의미              |
| ------------- | ----------------- |
| `*`           | 0부터 무한대 개수 |
| `+`           | 1번 이상          |
| `{m}, {m, n}` | m번, m~n번        |
| `?`           | 0 or 1            |



## 2. 정규식 사용하기

### 문자열 검색

| 메서드/어트리뷰트 | 목적                                                         |
| :---------------- | :----------------------------------------------------------- |
| `match()`         | 문자열의 시작 부분에서 RE가 일치하는지 판단                  |
| `search()`        | 이 RE가 일치하는 위치를 찾으면서, 문자열을 훑음              |
| `findall()`       | RE가 일치하는 모든 부분 문자열을 찾아 리스트로 반환          |
| `finditer()`      | RE가 일치하는 모든 부분 문자열을 찾아 [이터레이터](https://docs.python.org/ko/3/glossary.html#term-iterator)로 반환 |



#### match

match 메서드는 문자열의 처음부터 정규식과 매치되는지 조사한다. 위 패턴에 match 메서드를 수행해 보자.

```python
>>> m = p.match("python")
>>> print(m)
<_sre.SRE_Match object at 0x01F3F9F8>
```

"python" 문자열은 `[a-z]+` 정규식에 부합되므로 match 객체를 돌려준다.

```python
>>> m = p.match("3 python")
>>> print(m)
None
```

"3 python" 문자열은 처음에 나오는 문자 3이 정규식 `[a-z]+`에 부합되지 않으므로 None을 돌려준다.

match의 결과로 match 객체 또는 None을 돌려주기 때문에 파이썬 정규식 프로그램은 보통 다음과 같은 흐름으로 작성한다.

```python
p = re.compile(정규표현식)
m = p.match( 'string goes here' )
if m:
    print('Match found: ', m.group())
else:
    print('No match')
```

즉 match의 결괏값이 있을 때만 그다음 작업을 수행하겠다는 것이다.



#### search

컴파일된 패턴 객체 p를 가지고 이번에는 search 메서드를 수행해 보자.

```python
>>> m = p.search("python")
>>> print(m)
<_sre.SRE_Match object at 0x01F3FA68>
```

"python" 문자열에 search 메서드를 수행하면 match 메서드를 수행했을 때와 동일하게 매치된다.

```python
>>> m = p.search("3 python")
>>> print(m)
<_sre.SRE_Match object at 0x01F3FA30>
```

"3 python" 문자열의 첫 번째 문자는 "3"이지만 search는 문자열의 처음부터 검색하는 것이 아니라 문자열 전체를 검색하기 때문에 "3 " 이후의 "python" 문자열과 매치된다.

이렇듯 match 메서드와 search 메서드는 문자열의 처음부터 검색할지의 여부에 따라 다르게 사용해야 한다.



#### findall

이번에는 findall 메서드를 수행해 보자.

```python
>>> result = p.findall("life is too short")
>>> print(result)
['life', 'is', 'too', 'short']
```

"life is too short" 문자열의 'life', 'is', 'too', 'short' 단어를 각각 `[a-z]+` 정규식과 매치해서 리스트로 돌려준다.



#### finditer

이번에는 finditer 메서드를 수행해 보자.

```python
>>> result = p.finditer("life is too short")
>>> print(result)
<callable_iterator object at 0x01F5E390>
>>> for r in result: print(r)
...
<_sre.SRE_Match object at 0x01F3F9F8>
<_sre.SRE_Match object at 0x01F3FAD8>
<_sre.SRE_Match object at 0x01F3FAA0>
<_sre.SRE_Match object at 0x01F3F9F8>
```

finditer는 findall과 동일하지만 그 결과로 반복 가능한 객체(iterator object)를 돌려준다. 반복 가능한 객체가 포함하는 각각의 요소는 match 객체이다.



### Match Object의 Method

| 뷰트      | 목적                                          |
| :-------- | :-------------------------------------------- |
| `group()` | RE와 일치하는 문자열을 반환                   |
| `start()` | 일치의 시작 위치를 반환                       |
| `end()`   | 일치의 끝 위치를 반환                         |
| `span()`  | 일치의 (시작, 끝) 위치를 포함하는 튜플을 반환 |



다음 예로 확인해 보자.

```python
>>> m = p.match("python")
>>> m.group()
'python'
>>> m.start()
0
>>> m.end()
6
>>> m.span()
(0, 6)
```

예상한 대로 결괏값이 출력되는 것을 확인할 수 있다. match 메서드를 수행한 결과로 돌려준 match 객체의 start()의 결괏값은 항상 0일 수밖에 없다. 왜냐하면 match 메서드는 항상 문자열의 시작부터 조사하기 때문이다.

만약 search 메서드를 사용했다면 start() 값은 다음과 같이 다르게 나올 것이다.

```python
>>> m = p.search("3 python")
>>> m.group()
'python'
>>> m.start()
2
>>> m.end()
8
>>> m.span()
(2, 8)
```



> 패턴을 만들지 않고 아래 처럼 축약하여 사용도 가능하다
>
> ```m = re.match('[a-z]+', "python")```



### 3. 다른 유용한 기능

#### \b

 단어의 경계에 매치한다. 다음 예제는 완전한 단어의 `class` 만 매치한다.

```python
>>> p = re.compile(r'\bclass\b')
>>> print(p.search('no class at all'))
<re.Match object; span=(3, 8), match='class'>
>>> print(p.search('the declassified algorithm'))
None
>>> print(p.search('one subclass is'))
None
```



#### Group naming

`(P<name>...)` 으로 각 그룹에 이름을 붙일 수 있다. `groupdict()` 로도 확인 가능하다.

```python
>>> p = re.compile(r"(?P<name>\w+)\s+((\d+)[-]\d+[-]\d+)")
>>> m = p.search("park 010-1234-1234")
>>> print(m.group("name"))
park
```



> Assertion에 관한 내용이 복잡하여 다음글로 다뤄보겠다.



## 참조

- [Official Docs](https://docs.python.org/ko/3/howto/regex.html#search-and-replace)
- [예제로 배우는 python 프로그래밍](http://pythonstudy.xyz/python/article/401-정규-표현식-Regex)
- [Jump to Python](https://wikidocs.net/4308)

