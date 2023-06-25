---
aliases:
- /python/2021/01/03/python-sequence
categories:
- python
date: '2021-01-03'
description: section3
layout: post
title: Python Sequence
toc: true

---

Python Intermediate Programming

---

## 3.0. Sequence type

### 컨테이너 vs 플랫
Container: 서로다른 자료형: list, tuple, collections.deque  
Flat: 한 개의 자료형 str, bytes, bytearray, array.array, memoryview  

### 가변 vs 불변
Mutable: list, bytearray, array.array, memoryview, deque  
Immutable: tuple, str, bytes  

### 얕은 복사 vs 깊은 복사  

리스트 생성 시 for문으로 재귀적으로 생성과 \* 연산으로 생성 차이
```
marks1 = [['A'] * 5 for n in range(5)]
marks2 = [['A'] * 5] * 5

print(marks1)
> [['A', 'A', 'A', 'A', 'A'], ['A', 'A', 'A', 'A', 'A'], ['A', 'A', 'A', 'A', 'A'], ['A', 'A', 'A', 'A', 'A'], ['A', 
'A', 'A', 'A', 'A']]
print(marks2)
> [['A', 'A', 'A', 'A', 'A'], ['A', 'A', 'A', 'A', 'A'], ['A', 'A', 'A', 'A', 'A'], ['A', 'A', 'A', 'A', 'A'], ['A', 
'A', 'A', 'A', 'A']]

# 값 변경
marks1[0][1] = 'X'
marks2[0][1] = 'X'

print(marks1)
> [['A', 'X', 'A', 'A', 'A'], ['A', 'A', 'A', 'A', 'A'], ['A', 'A', 'A', 'A', 'A'], ['A', 'A', 'A', 'A', 'A'], ['A', 
'A', 'A', 'A', 'A']]
print(marks2)
> [['A', 'X', 'A', 'A', 'A'], ['A', 'X', 'A', 'A', 'A'], ['A', 'X', 'A', 'A', 'A'], ['A', 'X', 'A', 'A', 'A'], ['A', 
'X', 'A', 'A', 'A']]

# id값 차이 확인
print([id(i) for i in marks1])
> [1817473083648, 1817473082944, 1817473083136, 1817473083584, 1817473083712]
print([id(i) for i in marks2])
> [1817473087040, 1817473087040, 1817473087040, 1817473087040, 1817473087040]
```

shallow copy: 객체 껍데기만 복사하고 내용은 동일한 객체 참조  
mutable한 list의 경우 수정되고, immutable한 tuple의 경우 별도의 객체에 저장됨  
```
import copy

a = [1, [1, 2, 3]]
b = copy.copy(a)  # shallow copy
print(b)    
> [1, [1, 2, 3]]
b[0] = 100 # immutable tuple 수정
print(b) 
> [100, [1, 2, 3]]
print(a)  # immutable tuple의 경우 복사본만 수정
> [1, [1, 2, 3]] 

c = copy.copy(a)  # shallow copy
c[1].append(4)  # mutable list 수정
print(c)    
> [1, [1, 2, 3, 4]] 
print(a) # mutable list의 경우 둘 다 값이 변경됨
> [1, [1, 2, 3, 4]] 

```

deep copy: 객체 복사 후 내용도 재귀적으로 복사  
mutable, immutable 상관없이 별도의 객체에 저장됨  
```
a = [1, [1, 2, 3]]
b = copy.deepcopy(a)  # deep copy
print(b)    
> [1, [1, 2, 3]]
b[0] = 100
b[1].append(4)
print(b)    
> [100, [1, 2, 3, 4]] 
print(a)    
> [1, [1, 2, 3]]
```



## 3.1. Generator
한 번에 한 개의 항목을 생성(메모리 유지X)  


### Generator 생성

```
chars = 'abc'
tuple_g = (ord(s) for s in chars)
print(tuple_g)
> <generator object <genexpr> at 0x000001A729CF75F0>
type(tuple_g)
> <class 'generator'>
print(next(tuple_g))
> 97
print(next(tuple_g))
> 98
print(next(tuple_g))
> 99
print(next(tuple_g)) 
> StopIteration Error

```


---

## 3.2. Tuple Advanced

### unpacking
```
x, y, *rest = range(10)
print(x, y, rest)
> 0 1 [2, 3, 4, 5, 6, 7, 8, 9]
```


---

## 3.3. HashTable
적은 리소스로 많은 데이터를 효율적으로 관리
중복 허용 안되는 데이터 타입들 (Dict, Set)

### Set vs List

```
t1 = (10, 20, (30, 40, 50))
t2 = (10, 20, [30, 40, 50])
print( hash(t1))
> 465510690262297113
print( hash(t2)) # hash error남
```

### setdefault 미사용 (dict)
```
source = (('k1', 'val1'),
            ('k1', 'val2'),
            ('k2', 'val3'),
            ('k2', 'val4'),
            ('k2', 'val5'))

new_dict1 = {}

for k, v in source:
    if k in new_dict1:
        new_dict1[k].append(v)
    else:
        new_dict1[k] = [v]

print(new_dict1)
> {'k1': ['val1', 'val2'], 'k2': ['val3', 'val4', 'val5']}
```

### setdefault 사용
```
new_dict2 = {}
for k, v in source:
    new_dict2.setdefault(k, []).append(v)

print(new_dict2)
> {'k1': ['val1', 'val2'], 'k2': ['val3', 'val4', 'val5']}
```

---
## 3.4. Dict & Set Advanced

```
# immutable Dict
from types import MappingProxyType

d = {'key1': 'value1'}

# Read Only
d_frozen = MappingProxyType(d)

print(d, id(d))
> {'key1': 'value1'} 1817473569408
print(d_frozen, id(d_frozen))
> {'key1': 'value1'} 1817472793280
print(d is d_frozen, d == d_frozen)
> False True

# 수정 불가
d_frozen['key1'] = 'value2' # mappingproxy 객체는 assign지원하지 않는다는 error

d['key2'] = 'value2'

print(d)
> {'key1': 'value1', 'key2': 'value2'}

s1 = {'Apple', 'Orange', 'Apple', 'Orange', 'Kiwi'}
s2 = set(['Apple', 'Orange', 'Apple', 'Orange', 'Kiwi'])
s3 = {3}
s4 = set() # Not {}
s5 = frozenset({'Apple', 'Orange', 'Apple', 'Orange', 'Kiwi'})

# 추가
s1.add('Melon')

# 추가 불가
s5.add('Melon') # frozenset은 add 속성 존재하지 않는다는 error
print(s1, type(s1))
> {'Orange', 'Apple', 'Kiwi', 'Melon'} <class 'set'>
print(s2, type(s2))
> {'Orange', 'Apple', 'Kiwi'} <class 'set'>
print(s3, type(s3))
> {3} <class 'set'>
print(s4, type(s4))
> set() <class 'set'>
print(s5, type(s5))
> frozenset({'Orange', 'Apple', 'Kiwi'}) <class 'frozenset'>

```



---
