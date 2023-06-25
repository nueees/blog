---
aliases:
- /python/2021/01/07/python-linkedlist
categories:
- python
date: '2021-01-07'
description: section7
layout: post
title: Python Linked List
toc: true

---

[참고](https://wayhome25.github.io/cs/2017/04/17/cs-19/)

---

## 7.1. Linked list

### 특징
![image](https://user-images.githubusercontent.com/83441376/145132920-b1950c11-b841-45af-bef2-cfc6a43a5905.png)

Array list와는 다르게 element 간의 연결(link)을 이용해서 List 구현  
Array list에서 element가 Linked list와 같이 연결된 element은 node(마디, 교점) 혹은 vertex(정점, 꼭지점)라고 부름  
python 의 경우 list 기본 자료형에 linked list 기능이 함께 포함


![linkedlist](https://user-images.githubusercontent.com/83441376/145132511-7937da1a-33f4-4843-a237-f353bcac980b.png)


### Node
Linked list의 단위  
data와 next node를 가리키는 reference 값으로 구성  
next node로 None을 가리키면 리스트의 끝  

```
class Node(object):
    def __init__(self, data, next = None):
        self.data = data
        self.next = next

```

### Single Linked list
```

class SList(object):
    def __init__(self):
        self.head = Node(None)
        self.size = 0
        
    def insert(self, value):
        if self.size == 0:
            self.head = Node(value) # first
            self.size += 1
        else:
            target = self.head # first
            while(target.next != None):
                target = target.next
            newNode = Node(value) # second
            target.next = newNode

            self.size += 1
        
            
    def printlist(self):
        target = self.head
        while target:
            if target.next != None:
                print(target.data, '-> ', end='')
                target = target.next
            else:
                print(target.data)
                target = target.next

s_list1 = SList()
s_list1.insert("first")
s_list1.printlist()
> first
s_list1.insert("second")
s_list1.printlist()
> first -> second
s_list1.insert("third")
s_list1.printlist()
> first -> second -> third
```



