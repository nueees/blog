---
aliases:
- /algorithm/python/2020/12/05/stack-queue
categories:
- algorithm
- python
date: '2020-12-05'
description: section5
layout: post
title: Stack, Queue, Hash, Heap algorithm
toc: true

---

Python Algorithm Practice
[stack/queue/deque](https://youtu.be/1aCzy4A7ToM)
[graph/tree/heap/bst](https://youtu.be/nVPDmM3jJdI)

---

## 5.1. Biggest number (stack)

선생님은 현수에게 숫자 하나를 주고, 해당 숫자의 자릿수들 중 m개의 숫자를 제거하여 가장 큰 수를 만들라고 했습니다. 
여러분이 현수를 도와주세요.(단 숫자의 순서는 유지해야 합니다)
만약 5276823 이 주어지고 3개의 자릿수를 제거한다면 7823이 가장 큰 숫자가 됩니다.
```
num, m = map(int, input().split())
num = list(map(int,str(num)))
stack = []

for i in num:
    while stack and m > 0 and stack[-1] < i:
        stack.pop()
        m -= 1
    stack.append(i)

if m != 0:
    stack = stack[:m]
res = ''.join(map(str, stack))
print(res)
```

---
## 5.2. Iron bar (stack)

여러 개의 쇠막대기를 레이저로 절단하려고 한다. 효율적인 작업을 위해서 쇠막대기를 아래에서 위로 겹쳐 놓고, 레이저를 위에서 수직으로 발사하여 쇠막대기들을 자른다. 쇠막대기와 레이저의 배치는 다음 조건을 만족한다.  
• 쇠막대기는 자신보다 긴 쇠막대기 위에만 놓일 수 있다.   
• 쇠막대기를 다른 쇠막대기 위에 놓는 경우 완전히 포함되도록 놓되, 끝점은 겹치지 않도록 놓는다.  
• 각 쇠막대기를 자르는 레이저는 적어도 하나 존재한다.  
• 레이저는 어떤 쇠막대기의 양 끝점과도 겹치지 않는다.   
아래 그림은 위 조건을 만족하는 예를 보여준다. 수평으로 그려진 굵은 실선은 쇠막대기이고, 
점은 레이저의 위치, 수직으로 그려진 점선 화살표는 레이저의 발사 방향이다.  
이러한 레이저와 쇠막대기의 배치는 다음과 같이 괄호를 이용하여 왼쪽부터 순서대로 표현할 수 있다.  
![]({{site.baseurl}}/images/post/ironbar.JPG)  
1. 레이저는 여는 괄호와 닫는 괄호의 인접한 쌍 ‘( )’ 으로 표현된다. 또한, 모든 ‘( )’는 반드시 레이저를 표현한다.  
2. 쇠막대기의 왼쪽 끝은 여는 괄호 ‘(’ 로, 오른쪽 끝은 닫힌 괄호 ‘)’ 로 표현된다.  
위 예의 괄호 표현은 그림 위에 주어져 있다.  
쇠막대기는 레이저에 의해 몇 개의 조각으로 잘려지는데, 위 예에서 가장 위에 있는 두 개의 쇠막대기는 각각 3개와 2개의 조각으로 잘려지고, 이와 같은 방식으로 주어진 쇠막대기들은 총 17개의 조각으로 잘려진다.  
쇠막대기와 레이저의 배치를 나타내는 괄호 표현이 주어졌을 때, 잘려진 쇠막대기 조각의 총 개수를 구하는 프로그램을 작성하시오.   


```
bar = input()
stack = []
cnt = 0
prev = ')'
for i in range(len(bar)):
    # print(bar[i])
    if bar[i] == '(':
        stack.append(bar[i])
    else: # ')'
        stack.pop()
        if bar[i-1] == '(':
            cnt += len(stack)
        else: # ')'
            cnt += 1

print(cnt)
```


---
## 5.3. Evaluation of Postfix Expression (stack)

중위표기식이 입력되면 후위표기식으로 변환하는 프로그램을 작성하세요.  
중위표기식은 우리가 흔히 쓰은 표현식입니다. 즉 3+5 와 같이 연산자가 피연산자 사이에 있으면 중위표기식입니다.   
후위표기식은 35\+ 와 같이 연산자가 피연산자 뒤에 있는 표기식입니다.   
예를 들어 중위표기식이 3\+5\*2 를 후위표기식으로 표현하면 352\*\+ 로 표현됩니다.  
만약 다음과 같이 연산 최우선인 괄호가 표현된 식이라면 (3\+5)\*2 이면 35\+2\* 로 바꾸어야 합니다.  

```
exp = input()
stack = []
res = ''
for x in exp:
    if x.isdecimal():
        res += x
    else:
        if x == '(':
            stack.append(x)
        elif x == ')':
            while stack and (stack[-1] != '('):
                res += stack.pop()
            stack.pop()
        elif x == '*' or x == '/':
            while stack and (stack[-1] == '*' or stack[-1] == '/'):
                res += stack.pop()
            stack.append(x)
        elif x == '+' or x == '-':
            while stack and (stack[-1] != '('):
                res += stack.pop()
            stack.append(x)
while stack:
    res += stack.pop()
print(res)
```

---
## 5.4. Evaluation of Postfix (stack)
후위연산식이 주어지면 연산한 결과를 출력하는 프로그램을 작성하세요.  
만약 3\*\(5\+2\)\-9 을 후위연산식으로 표현하면 352\+\*9\- 로 표현되며 그 결과는 21입니다.  
```
exp = input()
stack = []
for x in exp:
    if x.isdecimal():
        stack.append(int(x))
    else:
        n2 = stack.pop()
        n1 = stack.pop()
        if x == '*':
            stack.append(n1*n2)
        if x == '/':
            stack.append(n1/n2)
        if x == '+':
            stack.append(n1+n2)
        if x == '-':
            stack.append(n1-n2)
print(stack.pop())
```

---
## 5.5. Last survivor (queue)
왕은 왕자들을 나이 순으로 1번부터 N번까지 차례로 번호를 매긴다.  
그리고 1번 왕자부터 N번 왕자까지 순서대로 시계 방향으로 돌아가며 동그랗게 앉게 한다.  
그리고 1번 왕자부터 시계방향으로 돌아가며 1부터 시작하여 번호를 외치게 한다.  
한 왕자가 K(특정숫자)를 외치면 그 왕자는 밖으로 나오게 된다.  
그리고 다음 왕자부터 다시 1부터 시작하여 번호를 외친다.  
이렇게 해서 마지막까지 남은 왕자가 살아남는다.  
예를 들어 총 8명의 왕자가 있고, 3을 외친 왕자가 제외된다고 하자. 처음에는 3번 왕자가 3을 외쳐 제외된다.  
이어 6, 1, 5, 2, 8, 4번 왕자가 차례대로 제외되고 마지막까지 남게 된 7번 왕자가 살아 남습니다.  
N과 K가 주어질 때 살아남는 왕자의 번호를 출력하는 프로그램을 작성하시오.  

```
from collections import deque

n, k = map(int, input().split())

dequeue = list(range(1,n+1))
dq = deque(dequeue)
while dq:
    for _ in range(k-1):
        tmp = dq.popleft()
        dq.append(tmp)
    res = dq.popleft()

print(res)
```

---
## 5.6. Prioritizing in emergency room (queue)

메디컬 병원 응급실에는 의사가 한 명밖에 없습니다.  
응급실은 환자가 도착한 순서대로 진료를 합니다. 하지만 위험도가 높은 환자는 빨리 응급조치를 의사가 해야 합니다.  
이런 문제를 보완하기 위해 응급실은 다음과 같은 방법으로 환자의 진료순서를 정합니다.  
• 환자가 접수한 순서대로의 목록에서 제일 앞에 있는 환자목록을 꺼냅니다.  
• 대기 목록에서 꺼낸 환자 보다 위험도가 높은 환자가 존재하면 대기목록 제일 뒤로 다시 넣습니다. 그렇지 않으면 진료를 받습니다. 즉 대기목록에 자기 보다 위험도가 높은 환자가 없을 때 자신이 진료를 받는 구조입니다.  
현재 N명의 환자가 대기목록에 있습니다.  
N명의 대기목록 순서의 환자 위험도가 주어지면, 대기목록상의 M번째 환자는 몇 번째로 진료를 받는지 출력하는 프로그램을 작성하세요.  
대기목록상의 M번째는 대기목록의 제일 처음 환자를 0번째로 간주하여 표현한 것입니다.  

```
from collections import deque

n, p = map(int, input().split())
list(map(int, input().split()))
a = [(pos, val) for pos, val in enumerate(list(map(int, input().split())))]
dq = deque(a)
cnt = 0
while dq:
    tmp = dq.popleft()
    if any(tmp[1] < x[1] for x in dq):
        dq.append(tmp)
    else:
        cnt += 1
        if tmp[0] == p:
            break
print(cnt)
```

---
## 5.7. Making a curriculum (queue)
현수는 1년 과정의 수업계획을 짜야 합니다.  
수업중에는 필수과목이 있습니다. 이 필수과목은 반드시 이수해야 하며, 그 순서도 정해져 있습니다. 
만약 총 과목이 A, B, C, D, E, F, G가 있고, 여기서 필수과목이 CBA로 주어지면 필수과목은 C, B, A과목이며 이 순서대로 꼭 수업계획을 짜야 합니다.  
여기서 순서란 B과목은 C과목을 이수한 후에 들어야 하고, A과목은 C와 B를 이수한 후에 들어야 한다는 것입니다.  
현수가 C, B, D, A, G, E로 수업계획을 짜면 제대로 된 설계이지만 C, G, E, A, D, B 순서로 짰다면 잘 못 설계된 수업계획이 됩니다.  
수업계획은 그 순서대로 앞에 수업이 이수되면 다음 수업을 시작하다는 것으로 해석합니다.  
수업계획서상의 각 과목은 무조건 이수된다고 가정합니다.
필수과목순서가 주어지면 현수가 짠 N개의 수업설계가 잘된 것이면 "YES", 잘못된 것이면 "NO"를 출력하는 프로그램을 작성하세요.

```
from collections import deque

req = input()
dq = deque(req)
n = int(input())
for i in range(n):
    plan = input()
    for x in plan:
        if x in dq:
            if x != dq.popleft():
                print(f'{i}번째 NO')
                break
    else:
        if len(dq) == 0:
            print(f'{i}번째 NO')
        else:
            print(f'{i}번째 YES')
```


---
## 5.8. Finding words (hash)
시를 쓰기 전에 시에 쓰일 단어를 미리 노트에 적어둡니다.  
이번에는 N개의 단어를 노트에 적었는데 시에 쓰지 않은 단어가 하나 있다고 합니다.  
그 단어를 찾아 주세요.  
```
n = int(input())
dic = {}
for i in range(n):
    word = input()
    dic[word] = 0
for i in range(n-1):
    w = input()
    dic[w] += 1
for k, v in dic.items():
    if v == 0:
        print(k)
```


---
## 5.9. Anagram
두 문자열이 알파벳의 나열 순서를 다르지만 그 구성이 일치하면 두 단어는 Anagram이라고 합니다.  
예를 들면 AbaAeCe 와 baeeACA 는 알파벳을 나열 순서는 다르지만 그 구성을 살펴보면 A(2), a(1), b(1), C(1), e(2)로 알파벳과 그 개수가 모두 일치합니다.  
즉 어느 한 단어를 재 배열하면 상대편 단어가 될 수 있는 것을 아나그램이라 합니다.  
길이가 같은 두 개의 단어가 주어지면 두 단어가 아나그램인지 판별하는 프로그램을 작성하세요. 아나그램 판별시 대소문자가 구분됩니다.  

```
str1 = input()
str2 = input()

dic = {}
dic.get('C',0)
for i in str1:
    dic[i] = dic.get(i,0)+1

for i in str2:
    dic[i] -= 1

if any(x != 0 for x in dic.values()):
    print("NO")
else:
    print("YES")
```

---
## 5.10. Min-heap
최소힙은 완전이진트리로 구현된 자료구조입니다. 그 구성은 부모 노드값이 왼쪽자식과 오른쪽 자식노드의 값보다 작게 트리를 구성하는 것입니다.  
그렇게 하면 트리의 루트(root)노드는 입력된 값들 중 가장 작은 값이 저장되어 있습니다.  
예를 들어 5 3 2 1 4 6 7순으로 입력되면 최소힙 트리는 아래와 같이 구성됩니다.  
![]({{site.baseurl}}/images/post/minheap.JPG)  
최소힙 자료를 이용하여 다음과 같은 연산을 하는 프로그램을 작성하세요.  
1) 자연수가 입력되면 최소힙에 입력한다.  
2) 숫자 0 이 입력되면 최소힙에서 최솟값을 꺼내어 출력한다. (출력할 자료가 없으면 -1를 출력한다.)  
3) -1이 입력되면 프로그램 종료한다.  
```
import heapq as hq
a = []
while True:
    n = int(input())
    if n == -1:
        break
    if n == 0:
        if len(a) == 0:
            print(-1)
        else:
            print(hq.heappop(a))
    else:
        hq.heappush(a, n)
```

---
## 5.11. Max-heap
최대힙은 완전이진트리로 구현된 자료구조입니다. 그 구성은 부모 노드값이 왼쪽자식과 오른쪽 자식노드의 값보다 크게 트리를 구성하는 것입니다.  
그렇게 하면 트리의 루트(root)노드는 입력된 값들 중 가장 큰 값이 저장되어 있습니다.  
예를 들어 5 3 2 1 4 6 7순으로 입력되면 최대힙 트리는 아래와 같이 구성됩니다.  

![]({{site.baseurl}}/images/post/maxheap.JPG)  
최대힙 자료를 이용하여 다음과 같은 연산을 하는 프로그램을 작성하세요.  
1) 자연수가 입력되면 최대힙에 입력한다.  
2) 숫자 0 이 입력되면 최대힙에서 최댓값을 꺼내어 출력한다.(출력할 자료가 없으면 -1를 출력한다.)   
3) -1이 입력되면 프로그램 종료한다.  

```
import heapq as hq
a = []
while True:
    n = int(input())
    if n == -1:
        break
    if n == 0:
        if len(a) == 0:
            print(-1)
        else:
            print(-hq.heappop(a))
    else:
        hq.heappush(a, -n)
```

---




