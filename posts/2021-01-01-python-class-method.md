---
aliases:
- /python/2021/01/01/python-class-method
categories:
- python
date: '2021-01-01'
description: section1
layout: post
title: Python Class & Method
toc: true

---

Python Intermediate Programming
(패키지 배포)[https://pj9039.tistory.com/m/4]

---

## 1.1. OOP (Object Oriented Programming)
객체 지향 프로그래밍: 코드 재사용  

### Using List
```
car_company_list = ['Ferrari', 'Bmw', 'Audi']
car_detail_list = [
    {'color' : 'White', 'horsepower': 400, 'price': 8000},
    {'color' : 'Black', 'horsepower': 270, 'price': 5000},
    {'color' : 'Silver', 'horsepower': 300, 'price': 6000}
]

car_company_list[1]
del car_detail_list[1]
```
관리가 불편하고 인덱스로 접근하기 때문에 오류 가능성이 높음  

### Using Dictionary 
```
cars_dicts = [
    {'car_company': 'Ferrari', 'car_detail': {'color' : 'White', 'horsepower': 400, 'price': 8000}},
    {'car_company': 'Bmw', 'car_detail': {'color' : 'Black', 'horsepower': 270, 'price': 5000}},
    {'car_company': 'Audi', 'car_detail': {'color' : 'Silver', 'horsepower': 300, 'price': 6000}}
]

del cars_dicts[1]
```
재사용성이 낮고, 중첩시 overwrite, 키 조회 예외 처리 따로 해야함  

### Using Class
```
class Car():
    def __init__(self, company, details):
        self._company = company
        self._details = details

    def __str__(self):
        return 'str : {} - {}'.format(self._company, self._details)

    def __repr__(self):
        return 'repr : {} - {}'.format(self._company, self._details)
    

car1 = Car('Ferrari', {'color' : 'White', 'horsepower': 400, 'price': 8000})
car2 = Car('Bmw', {'color' : 'Black', 'horsepower': 270, 'price': 5000})
car3 = Car('Audi', {'color' : 'Silver', 'horsepower': 300, 'price': 6000})

print(car1.__dict__)
> {'_company': 'Ferrari', '_details': {'color': 'White', 'horsepower': 400, 'price': 8000}}
```
재사용성 높고, 메소드 활용 가능  


### Class variables

```
class Car():
    """
    Car Class
    Author : Cho
    Date : 2021.01.01
    """

    # 클래스 변수
    car_count = 0

    def __init__(self, company, details):
        self._company = company
        self._details = details
        Car.car_count += 1

    def __str__(self):
        return 'str : {} - {}'.format(self._company, self._details)

    def __repr__(self):
        return 'repr : {} - {}'.format(self._company, self._details)

    def detail_info(self):
        print('Current Id : {}'.format(id(self)))
        print('Car Detail Info : {} {}'.format(self._company, self._details.get('price')))

    def __del__(self):
        Car.car_count -= 1
```

self 값
```
car1 = Car('Ferrari', {'color' : 'White', 'horsepower': 400, 'price': 8000})
car2 = Car('Bmw', {'color' : 'Black', 'horsepower': 270, 'price': 5000})
car3 = Car('Audi', {'color' : 'Silver', 'horsepower': 300, 'price': 6000})
print(car1)
> str : Ferrari - {'color': 'White', 'horsepower': 400, 'price': 8000}
```


id 값
```
print(id(car1))
> 1817466634144
```

dir 값
```
print(dir(car1))
> ['__class__', '__del__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_company', '_details', 'car_count', 'detail_info']
```


dict 값
```
print(car1.__dict__)
> {'_company': 'Ferrari', '_details': {'color': 'White', 'horsepower': 400, 'price': 8000}}
```

doc 값
```
print(Car.__doc__)
>   Car Class
    Author : Kim
    Date : 2019.11.08
```

user defined method 값
```
car1.detail_info()
> Current Id : 1817466634144
  Car Detail Info : Ferrari 8000

Car.detail_info() # self missing error 발생
Car.detail_info(car1)
> Current Id : 1817466634144
  Car Detail Info : Ferrari 8000
```

class 비교
```
print(car1.__class__, car2.__class__)
> <class '__main__.Car'> <class '__main__.Car'>
print(id(car1.__class__) == id(car2.__class__))
> True

```

### Instance variables
```
print(car1._company) # 직접접근 방법, 권장 X
> Ferrari
```

### Class variables
```
print(car1.car_count)
> 3
print(Car.car_count)
> 3
```

class 공유 확인
```
print(Car.__dict__)
> {'__module__': '__main__'
  , '__doc__': '\n
                  Car Class\n
                  Author : Kim\n
                  Date : 2019.11.08\n'
  , 'car_count': 2
  , '__init__': <function Car.__init__ at 0x000001A729721CA0>
  , '__str__': <function Car.__str__ at 0x000001A7297C0550>
  , '__repr__': <function Car.__repr__ at 0x000001A7297C05E0>
  , 'detail_info': <function Car.detail_info at 0x000001A7297C0670>
  , '__del__': <function Car.__del__ at 0x000001A7297C0700>
  , '__dict__': <attribute '__dict__' of 'Car' objects>
  , '__weakref__': <attribute '__weakref__' of 'Car' objects>}

print(car1.__dict__)
> {'_company': 'Ferrari', '_details': {'color': 'White', 'horsepower': 400, 'price': 8000}}
```

instance namespace 없으면 class(상위)에서 검색
```
print(car1.car_count)
> 2
print(Car.car_count)
> 2
```




---
## 1.2. Method

```
class Car(object):
    '''
    Car Class
    Author : Cho
    Date : 2021.01.01
    Description : Class, Static, Instance Method
    '''

    # Class Variable
    price_per_raise = 1.0

    def __init__(self, company, details):
        self._company = company # 인스턴스 변수 앞에 언더바 붙임
        self._details = details
        
 
    
    
    
    # Instance Method
    # self : 객체의 고유한 속성 값 사용
    def detail_info(self): # 첫변수 self로 받음
        print('Current Id : {}'.format(id(self)))
        print('Car Detail Info : {} {}'.format(self._company, self._details.get('price')))
        
    # Instance Method
    def get_price(self):
        return 'Before Car Price -> company : {}, price : {}'.format(self._company, self._details.get('price'))

    # Instance Method
    def get_price_culc(self):
        return 'After Car Price -> company : {}, price : {}'.format(self._company, self._details.get('price') * Car.price_per_raise)

    # Class Method
    @classmethod
    def raise_price(cls, per): # 첫변수 무조건 cls로 받음
        if per <= 1:
            print('Please Enter 1 or More')
            return
        cls.price_per_raise = per
        return 'Succeed! price increased.'
    

    # Static Method
    @staticmethod
    def is_bmw(inst): # 전달 변수 없어도 되나 차량확인시 필요 변수 inst 전달
        if inst._company == 'Bmw':
            return 'OK! This car is {}.'.format(inst._company)
        return 'Sorry. This car is not Bmw.'
        
        
```

```
car1 = Car('Bmw', {'color' : 'Black', 'horsepower': 270, 'price': 5000})
car2 = Car('Audi', {'color' : 'Silver', 'horsepower': 300, 'price': 6000})
```

### Class Method
데코레이터 \@classmethod 사용  
첫 변수로 cls 받아서 사용


### Instance Method
객체의 고유한 속성 값 self 변수 받아서 사용  

### Static Method
데코레이터 \@staticmethod 사용
class나 instance method처럼 cls나 self를 받지 않고 구현



