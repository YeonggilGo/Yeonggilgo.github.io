---
layout: post
title: "Python test Lib : Pytest"
# cover-img: /assets/img/path.jpg
thumbnail-img: /assets/img/pytest.jpg
# share-img: /assets/img/path.jpg
tags: [python, test]
comments: true
# gh-repo: yeonggilgo/pyspark-exercise
---

### Pytest is a mature full-featured Python testing tool that helps you write better programs.

### TDD (Test Driven Developement)
테스트를 먼저 작성하고 이에 적합한 로직을 개발하는 것을 의미 합니다.
애자일 방법론에 알맞은 개발 방식으로 함수, 기능 간 종속성 유지와 효율적인 디버깅을 할 수 있는 개발 방법론입니다. 

### Getting started
```shell
pip install pytest
pytest --version
```

<hr>

## Simple Example
`pytest {file_name}` or `python -m pytest {dir}`
```python
# first_test.py 

# 테스트를 해보고 싶은 함수 
def func(x): 
    return x + 1 

# 테스트 함수
def test_answer(): 
    assert func(3) == 5

```
```python
# Second test 
class TestClass: 
    def test_one(self): 
        x = "Hello, hi"
        assert "h" in x 
        
    def test_two(self): 
        x = "what"
        assert hasattr(x, "who")
```

## Pytest fixtures
*Fixture*
- 시스템의 필수 조건을 만족하는 테스팅 프로세스를 설정
- 같은 설정의 테스트를 쉽게 반복적으로 수행 할 수 있도록 돕는 

```python
# calculator.py 
class Calculator(object): 
    """Calculator class""" 
    def __init__(self): 
        pass 
    
    @staticmethod 
    def add(a, b): 
        return a + b 
        
    @staticmethod 
    def subtract(a, b): 
        return a - b 
        
    @staticmethod 
    def multiply(a, b):
        return a * b 
        
    @staticmethod
    def divide(a, b):
        return a / b
```

```python
# conftest.py 
import pytest 
from src.calculator import Calculator 
@pytest.fixture 
def calculator(): 
    calculator = Calculator() 
    return calculator
```

```python
# test_code.py 
def test_add(calculator): 
    """Test functionality of add.""" 
    assert calculator.add(1, 2) == 3 
    assert calculator.add(2, 2) == 4 
    assert calculator.add(9, 2) == 11 
    
def test_subtract(calculator): 
    """Test functionality of subtract"""
    assert calculator.subtract(5, 1) == 4 
    assert calculator.subtract(3, 2) == 1 
    assert calculator.subtract(10, 2) == 8 
        
def test_multiply(calculator): 
    """Test functionality of multiply."""
    assert calculator.multiply(2, 2) == 4 
    assert calculator.multiply(5, 6) == 30 
    assert calculator.multiply(9, 3) == 27
```

Fixture를 이용하여 Test code를 매우 단축 시킬 수 있다. setup, teardown과 비슷한 역할.

#### 참조
- [https://binux.tistory.com/47](https://binux.tistory.com/47)