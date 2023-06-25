---
aliases:
- /python/2021/01/06/python-abstract
categories:
- python
date: '2021-01-06'
description: section6
layout: post
title: Python Abstract & With
toc: true

---

Python Coding dojang

---

## 6.1. Abstract Class

```
from abc import *

class MyBookBase(metaclass=ABCMeta):
    
    @abstractmethod
    def display(self):
        pass

    @abstractmethod
    def get_amount(self):
        print("How many books sold?")

class MyBook(MyBookBase):
    cnt = 0
    def __init__(self, title, author, price):
        MyBook.cnt += 1
        self.title = title
        self.author = author
        self.price = price

    def display(self):
        print("Title: {}".format(self.title))
        print("Author: {}".format(self.author))
        print("Price: {}".format(self.price))

    def get_amount(self):
        super().get_amount()
        print(self.cnt)

new_novel1=MyBook('Human acts','Hangang',12000)
new_novel1.display()
> Title: Human acts
  Author: Hangang
  Price: 12000
new_novel1.get_amount()
> How many books sold?
> 1
new_novel2=MyBook('Vegitarian','Hangang',14000)
new_novel2.display()
> Title: Vegitarian
  Author: Hangang
  Price: 14000
new_novel2.get_amount()
> How many books sold?
> 2
```


## 6.2. With

```
class Hello:
    def __enter__(self):
        # 사용할 자원을 가져오거나 만든다(핸들러 등)
        print('enter...')
        return self # 반환값이 있어야 VARIABLE를 블록내에서 사용할 수 있다
        
    def sayHello(self, name):
        # 자원을 사용한다. ex) 인사한다
        print('hello ' + name)

    def __exit__(self, exc_type, exc_val, exc_tb):
        # 마지막 처리를 한다(자원반납 등)
        print('exit...')

with Hello() as h:
    h.sayHello('obama')
    h.sayHello('trump')
> enter...
  hello obama
  hello trump
  exit...
```


