---
aliases:
- /python/2021/01/04/python-firstclassfunction
categories:
- python
date: '2021-01-04'
description: section4
layout: post
title: Python First Class Function
toc: true

---

Python Intermediate Programming
[스쿨오브웹](https://schoolofweb.net/blog/posts/)

---

## 4.1. First Class Function
an instance of the Object type  
이미 정의된 여러 함수를 간단히 재활용 가능  

### Python Function 특징
1.런타임 초기화   
2.변수 할당  
3.함수 인수 전달  
4.함수 결과 반환  

```
def factorial(n):
    '''Factorial Function -> n : int'''
    if n == 1: # n < 2
        return 1
    return n * factorial(n-1)

class A:
    pass

print(factorial(5))
> 120

print(factorial.__name__) # 이름
> factorial

print(factorial.__code__) # 코드 위치와 객체 정보
> <code object factorial at 0x00000209FFE98EA0, file "<stdin>", line 1>

print(type(factorial), type(A)) # factorial 함수는 객체 타입이 function이고, A 함수는 객체 타입이 type으로 나옴
> <class 'function'> <class 'type'>
```

### 변수 할당  
```
var_func = factorial
print(factorial, var_func)
> <function factorial at 0x00000209FFDBAC10> <function factorial at 0x00000209FFDBAC10>
print(var_func(5))
> 120
print(list(map(var_func, range(1,6))))
> [1, 2, 6, 24, 120]
```

### 함수 인수 전달 및 함수 결과 반환
고위 함수(Higher-order function):  map, filter, reduce...  

```
print(list(map(var_func, filter(lambda x: x % 2, range(1,6))))) # map함수 첫 인자로 계산할 함수 var_func 전달
> [1, 6, 120]

from functools import reduce
from operator import add
print(reduce(add, range(1,11))) # reduce함수 첫 인자로 계산할 함수 add 전달, 누적
> 55
```

### Callable 확인
```
print(set(sorted(dir(factorial))) - set(sorted(dir(A)))) # function 객체의 variables와 method 중 type 객체꺼 빼서 보면 __call__이 있음 factorial.__call(5) 계산 가능
> {'__globals__', '__name__', '__annotations__', '__call__', '__code__', '__kwdefaults__', '__qualname__', '__get__', '__defaults__', '__closure__'}

print(callable(str), callable(A), callable(var_func), callable(3.14))
> True True True False

from inspect import signature

sg = signature(var_func)

print(sg)
> (n)
print(sg.parameters)
> OrderedDict([('n', <Parameter "n">)])
```
callable: 함수 생성 후 해당 함수가 method 형태로 호출이 가능한지 확인 할 수 있음   
signature: 어떤 파라메터를 받고 좀 더 상세하게 확인 가능한 lib  

### partial
인수를 고정해서 함수 사용 시 편리
```
from operator import add
from functools import partial

print(add(10,10))
> 20
five = partial(add, 5)
print(five(10)) # 5+10
> 15

# 추가
six = partial(five, 6)
print(six()) # 5+6
> 11

print([five(i) for i in range(1,11)])
> [6, 7, 8, 9, 10, 11, 12, 13, 14, 15]
print(list(map(five, range(1,11))))
> [6, 7, 8, 9, 10, 11, 12, 13, 14, 15]
```


---

## 4.2. Closure
동시성 제어  
자신의 영역 밖에서 호출된 함수의 변수값과 레퍼런스를 복사하고 저장한 뒤, 이 후에도 값들에 액세스할 수 있게 도와줌  
다른 함수의 지역변수를 그 함수가 종료된 이후에도 상태 기억  
프리변수(free variable): 클로저가 만들어지는 당시의 값과 레퍼런스에 맵핑하여 주는 역할  

ex) 결과 누적하는 함수들 (sum, reduce...)

### Closure 특징
1. 메모리 공간에 여러 자원이 접근 시 교착상태(Dead Lock) -> 동시성(Concurrency)제어 필요  
2. 하나의 Thread 안에서 메모리를 공유하지 않고 "메시지 전달"  
3. "메시지 전달"로 처리하기 위한 공유하되 변경되지 않는 Immutable, Read Only 적극적으로 사용 (함수형 프로그래밍)  
4. closure는 Immutable 자료구조, STM(stands for Software Transactional Memory) -> multi thread programming  
5. Python에서는 Coroutine이란 개념을 이용해서 single thread로 병행성 제어  


### Class vs Closure 비교

```
class Averager(): # class
    def __init__(self):
        self._series = []

    def __call__(self, v):
        self._series.append(v)
        print('inner >>> {} / {}'.format(self._series, len(self._series)))
        return sum(self._series) / len(self._series)

averager_cls = Averager() # init
print(averager_cls(15)) # call
> inner >>> [15] / 1
print(averager_cls(35))
> inner >>> [15, 35] / 2
print(averager_cls(40))
> inner >>> [15, 35, 40] / 3
```

```
def closure_ex1(): # closure
    series = [] # Free variable
    def averager(v):
        series.append(v)
        print('inner >>> {} / {}'.format(series, len(series)))
        return sum(series) / len(series)
    return averager # 함수 자체 리턴

avg_closure1 = closure_ex1() # create closure
print(avg_closure1) # closure 확인
> <function closure_ex1.<locals>.averager at 0x000002142617EE50>

print(avg_closure1(15)) # call
> inner >>> [15] / 1
print(avg_closure1(35))
> inner >>> [15, 35] / 2
print(avg_closure1(40))
> inner >>> [15, 35, 40] / 3
```

### 함수 내부 확인  
```
print(dir(avg_closure1))
> ['__annotations__', '__call__', '__class__', '__closure__', '__code__', '__defaults__', '__delattr__', 
'__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__get__', '__getattribute__', '__globals__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__kwdefaults__', '__le__', '__lt__', '__module__', '__name__', '__ne__', '__new__', '__qualname__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__']

print(dir(avg_closure1.__code__))
> ['__class__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'co_argcount', 'co_cellvars', 'co_code', 'co_consts', 'co_filename', 'co_firstlineno', 'co_flags', 'co_freevars', 'co_kwonlyargcount', 'co_lnotab', 'co_name', 'co_names', 'co_nlocals', 'co_posonlyargcount', 'co_stacksize', 'co_varnames', 'replace']

print(avg_closure1.__code__.co_freevars) # 위 리스트에서 co_freevars 확인
> ('series',)

print(dir(avg_closure1.__closure__[0]))
> ['__class__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'cell_contents']

print(avg_closure1.__closure__[0].cell_contents) # 위 리스트에서 실제값 확인을 위해 cell_contents 확인
> [15, 35, 40]

```

### closure 실행 순서 확인  
```
def outer_func(): # 1
    message = 'Hi' # 3

    def inner_func(): # 4
        print(message) # 6 

    return inner_func() # 5

outer_func() # 2
> Hi
```

5에서 괄호를 지워면 Hi가 출력되지 않음  

```
def outer_func(): # 1
    message = 'Hi' # 3

    def inner_func(): # 4
        print(message) # 6 

    return inner_func # 5

outer_func() # 2
> <function outer_func.<locals>.inner_func at 0x00000214261B5280>
```

### nonlocal 변수 선언

```
def outer_func():
    cnt = 0 
    def inner_func():
        nonlocal cnt 
        cnt = cnt + 1 # inner에서 cnt를 nonlocal선언 안하고 쓰면 여기서 error남 
        print(cnt)

    return inner_func

func1 = outer_func()
func1()
> 1
func1()
> 2
func1()
> 3
```


---

## 4.3. Decorator
기존의 코드에 여러가지 기능을 추가하는 구문  
함수를 다른 함수의 인자로 전달한다는 점이 특징  

### Decorator 특징
#### 장점
1. 중복 제거, 코드 간결, 공통 함수 작성  
2. 로깅, 프레임워크, 유효성 체크 -> 공통 기능  
3. 조합해서 사용 용이  

#### 단점
1. 가독성 감소  
2. 특정 기능에 한정된 함수는 -> 단일 함수로 작성하는 것이 유리  
3. 디버깅 불편  


### Decorator 사용 vs 미사용

```
def outer_function(msg): # 1
    def inner_function(): # 5
        print(msg) # 7

    return inner_function # 6

hi_func = outer_function('Hi') # 2
bye_func = outer_function('Bye')

hi_func() # 3
> Hi
bye_func()
> Bye
```

```
def decorator_function(original_function):
    def wrapper_function(*msg):  #1
        print('{} before calling function.'.format(original_function.__name__))
        return original_function(*msg)  #2
    return wrapper_function


@decorator_function
def display():
    print('calling display function.')


@decorator_function
def display_info(msg):
    print('calling display_info({}) function.'.format(msg))


display()
> display before calling function.
> calling display function.

display_info('Hi')
> display_info before calling function.
> calling display_info(Hi) function.
display_info('Bye')
> display_info before calling function.
> calling display_info(Bye) function.
```

### Decorator 로깅 예제
logging for debugging  

```
import time

def perf_clock(func):
    def perf_clocked(*args):
        st = time.perf_counter() 
        result = func(*args)
        et = time.perf_counter() - st 
        name = func.__name__
        arg_str = ', '.join(repr(arg) for arg in args)
        print('[%0.5fs] %s(%s) -> %r' % (et, name, arg_str, result)) 
        return result 
    return perf_clocked

@perf_clock
def time_func(seconds):
    time.sleep(seconds)

@perf_clock
def sum_func(*numbers):
    return sum(numbers)
```
```
# no decorator
none_deco1 = perf_clock(time_func)
none_deco2 = perf_clock(sum_func)

print(none_deco1)
> <function perf_clock.<locals>.perf_clocked at 0x0000021426669280>
print(none_deco1.__code__.co_freevars)
> ('func',)

none_deco1(1.5) # time_func
> [1.50070s] time_func(1.5) -> None

none_deco2(100, 150, 250, 300, 350) # sum_func
> [0.00000s] sum_func(100, 150, 250, 300, 350) -> 1150
```
```
# decorator
time_func(1.5) 
> [1.50811s] time_func(1.5) -> None
sum_func(100, 150, 250, 300, 350)
> [0.00001s] sum_func(100, 150, 250, 300, 350) -> 1150
```

---
