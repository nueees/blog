---
aliases:
- /python/2021/01/02/python-magicmethod
categories:
- python
date: '2021-01-02'
description: section2
layout: post
title: Python Magic Method
toc: true

---

Python Intermediate Programming
[docs.python.org](https://docs.python.org/3/reference/datamodel.html#special-method-names)

---

## 2.1. Built-In Functions

### Basic Data Types
basic numeric, string, boolean  
```
print(int)
print(float)
print(str)
print(bool)
> <class 'str'> # 모두 클래스로 만들어져 있음
print(dir(str)) # 모든 속성 메소드 확인  
> ['__abs__', '__add__', '__and__', '__bool__', '__ceil__', '__class__', '__delattr__', '__dir__', '__divmod__', '__doc__', '__eq__', '__float__', '__floor__', '__floordiv__', '__format__', '__ge__', '__getattribute__', '__getnewargs__', '__gt__', '__hash__', '__index__', '__init__', '__init_subclass__', '__int__', '__invert__', '__le__', '__lshift__', '__lt__', '__mod__', '__mul__', '__ne__', '__neg__', '__new__', '__or__', '__pos__', '__pow__', '__radd__', '__rand__', '__rdivmod__', '__reduce__', '__reduce_ex__', '__repr__', '__rfloordiv__', '__rlshift__', '__rmod__', 
'__rmul__', '__ror__', '__round__', '__rpow__', '__rrshift__', '__rshift__', '__rsub__', '__rtruediv__', '__rxor__', '__setattr__', '__sizeof__', '__str__', '__sub__', '__subclasshook__', '__truediv__', '__trunc__', '__xor__', 'as_integer_ratio', 'bit_length', 'conjugate', 'denominator', 'from_bytes', 'imag', 'numerator', 'real', 'to_bytes'] 
```

```
n = 10
print(n.__doc__) # 위처럼 숫자 할당 시 처리 방법
> int([x]) -> integer
  int(x, base=10) -> integer

  Convert a number or string to an integer, or return 0 if no arguments
  are given.  If x is a number, return x.__int__().  For floating point
  numbers, this truncates towards zero.

  If x is not a number or if base is given, then x must be a string,
  bytes, or bytearray instance representing an integer literal in the
  given base.  The literal can be preceded by '+' or '-' and be surrounded
  by whitespace.  The base defaults to 10.  Valid bases are 0 and 2-36.
  Base 0 means to interpret the base from the string as an integer literal.
  >>> int('0b100', base=0)
  4


print(n + 100, n.__add__(100))
> 110 110
print(n * 100, n.__mul__(100))
> 1000 1000

print(n.__bool__(), bool(n))
> True True

```


## 2.2. Magic Methods

```
class Vector(object):
    def __init__(self, *args):
        '''Create a vector, example : v = Vector(5,10)'''
        if len(args) == 0:
            self._x, self._y = 0, 0
        else:
            self._x, self._y = args

    def __repr__(self):
        '''Returns the vector infomations'''
        return 'Vector(%r, %r)' % (self._x, self._y)

    def __add__(self, other):
        '''Returns the vector addition of self and other'''
        return Vector(self._x + other._x, self._y + other._y)
    
    def __mul__(self, y):
        return Vector(self._x * y, self._y * y)

    def __bool__(self):
        return bool(max(self._x, self._y))


# create instance
v1 = Vector(5,7)
v2 = Vector(23, 35)
v3 = Vector()
```

### ge/le/add/sub/mul/div/str


```
print(Vector.__init__.__doc__)
> Create a vector, example : v = Vector(5,10)

print(Vector.__repr__.__doc__)
> Returns the vector infomations

print(Vector.__add__.__doc__)
> Returns the vector addition of self and other

print(v1, v2, v3)
> Vector(5, 7) Vector(23, 35) Vector(0, 0)

print(v1 + v2)
> Vector(28, 42)
print(v1 * 3)
> Vector(15, 21)
print(bool(v1), bool(v2), bool(v3)
> True True False

```


---

## 2.3. Named Tuple

### Tuple 사용
```
from math import sqrt

pt1 = (1.0, 5.0)
pt2 = (2.5, 1.5)

l_leng1 = sqrt((pt2[0] - pt1[0]) ** 2 + (pt2[1] - pt1[1]) ** 2)
print(l_leng1)
```
인덱스 에러 가능성 있음  

### Named Tuple 사용
```
from collections import namedtuple
Point = namedtuple('Point', 'x y')
pt3 = Point(1.0, 5.0)
pt4 = Point(2.5, 1.5)
l_leng2 = sqrt((pt4.x - pt3.x) ** 2 + (pt4.y - pt3.y) ** 2)
print(l_leng2)
```

### Named Tuple 선언
```
Point = namedtuple('Point', 'x y')
Point1 = namedtuple('Point', ['x', 'y'])
Point2 = namedtuple('Point', 'x, y')
Point3 = namedtuple('Point', 'x y x class', rename=True) # 자동으로 중복되는 필드네임 _2, _3으로 바뀜
Point4 = namedtuple('Point', 'x y x', rename=False) # 필드네임 중복 에러남 default
```

### Unpacking
```
p1 = Point1(45, y=20)
print(p1)
> Point(x=45, y=20)
x, y = p1
print(x, y)
> 45 20
```

### \_make
```
temp = [10,20,30,40] 
p2 = Point3._make(temp)
print(p2)
> Point(x=10, y=20, _2=30, _3=40)
```
instance 생성  

### \_fields
```
print(p1._fields, p2._fields)
> ('x', 'y') ('x', 'y', '_2', '_3')
```
field name 확인  

### \_asdict
```
print(p1._asdict(), p2._asdict())
> {'x': 45, 'y': 20} {'x': 10, 'y': 20, '_2': 30, '_3': 40}
```
field name의 value 확인 (OrderedDict으로 return)  


## 2.4. Named Tuple 예제

```
Classes = namedtuple('Classes', ['rank', 'number'])
# List Comprehension으로
students2 = [Classes(rank, number) 
                    for rank in 'A B C D'.split() 
                        for number in [str(n) 
                            for n in range(1,21)]]

for s in student2:
    print(s)

```

## 2.5. List Comprehension 예제

```
things = [3, 4, 6, 7, 0, 1]
#chaining together filter and map:
# first, filter to keep only the even numbers
# double each of them
map_obj = map(lambda x: x*2, filter(lambda y: y % 2 == 0, things))
print(list(map_obj))
] [8, 12, 0]

# equivalent version using list comprehension
print([x*2 for x in things if x % 2 == 0])
] [8, 12, 0]

```


## 2.6. Zip function 예제

list들을 tuple로 만들어 줌
```
L1 = [3, 4, 5, 8]
L2 = [1, 2, 3]
L4 = list(zip(L1, L2)) # 튜플->리스트로 캐스팅해서 재정의
print(L4)
] [(3, 1), (4, 2), (5, 3)]
```



---
