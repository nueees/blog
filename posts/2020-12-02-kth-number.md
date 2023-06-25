---
aliases:
- /algorithm/python/2020/12/02/kth-number
categories:
- algorithm
- python
date: '2020-12-02'
description: section2
layout: post
title: Kth number
toc: true

---

Python Algorithm Practice
[진수와진법/최대공약수/최소공배수/소인수분해,재귀함수](https://youtu.be/t_Ezo4NksZY)

---

## 2.1. Nth Prime Number
두 개의 자연수 N과 K가 주어졌을 때, N의 약수들 중 K번째로 작은 수를 출력하는 프로그램을 작성하세요.  
```
n, k = map(int, input().split())
print(n,k)

cnt = 0

for i in range(1,n+1):
    if n%i==0:
        cnt+=1
    if cnt==k:
        print(i)
        break
else:
    print(-1)
```

---


## 2.2. Nth Number
N개의 숫자로 이루어진 숫자열이 주어지면 해당 숫자열중에서 s번째부터 e번째까지의 수를 오름차순 정렬했을 때 k번째로 나타나는 숫자를 출력하는 프로그램을 작성하세요.  

```
T = int(input())
for i in range(T):    
    n, s, e, k = map(int, input().split())
    a = list(map(int, input().split()))
    # print(n,s,e,k)

    cnt = 0
    a = a[s-1:e]
    a.sort()
    print(a[k-1])
```
---

## 2.3. Nth Biggest Number
1부터 100사이의 자연수가 적힌 N장의 카드를 가지고 있습니다. 같은 숫자의 카드가 여러장 있을 수 있습니다.   
이 중 3장을 뽑아 각 카드에 적힌 수를 합한 값을 기록하려고 합니다.   
3장을 뽑을 수 있는 모든 경우를 기록합니다.   
기록한 값 중 K번째로 큰 수를 출력하는 프로그램을 작성하세요.  
```
n, k = map(int, input().split())
a = list(map(int, input().split()))

myset = set() # to remove duplication

for i in range(n):
    for j in range(i+1, n):
        for l in range(j+1):
            myset.add(a[i]+a[j]+a[l])

res = list(myset)
res.sort(reverse=True)
print(res[k-1])
```
---

## 2.4. Closest Representative value
N명의 학생의 수학점수가 주어집니다. N명의 학생들의 평균(소수 첫째자리 반올림)을 구하고, N명의 학생 중 평균에 가장 가까운 학생은 몇 번째 학생인지 출력하는 프로그램을 작성하세요.   
평균과 가장 가까운 점수가 여러 개일 경우 먼저 점수가 높은 학생의 번호를 답으로 하고, 높은 점수를 가진 학생이 여러 명일 경우 그 중 학생번호가 빠른 학생의 번호를 답으로 합니다.  
```
n = int(input())
a = list(map(int, input().split()))
avg = round(sum(a)/n)
min = 2**10000
for idx, x in enumerate(a):
    tmp = abs(x-avg)
    if tmp < min:
        min = tmp
        score = x
        res = idx+1
    elif tmp == min:
        if x > score:
            score = x
            res = idx+1
print(avg, res)
```
---

## 2.5. Regular Polyhedron
두 개의 정 N면체와 정 M면체의 두 개의 주사위를 던져서 나올 수 있는 눈의 합 중 가장 확률이 높은 숫자를 출력하는 프로그램을 작성하세요.  
정답이 여러 개일 경우 오름차순으로 출력합니다.  
```
n, m = map(int, input().split())
cnt = [0]*(n+m+1)
cnt

for i in range(1, n+1):
    for j in range(1, m+1):
        cnt[i+j] += 1

cnt

max = -1
for i in range(1,len(cnt)):
    if cnt[i]>max:
        max=cnt[i]

for i in range(1,len(cnt)):
    if cnt[i]==max:
        print(i, end=' ')
```
---

## 2.6. Sum the Digits of a Number 
N개의 자연수가 입력되면 각 자연수의 자릿수의 합을 구하고, 그 합이 최대인 자연수를 출력하는 프로그램을 작성하세요.   
```
n = int(input())
a = list(map(int, input().split()))

def digit_sum(x):
    sum = 0
    while x>0:
        sum += x%10
        x = x//10
    return sum

max = 0
for x in a:
    tot = digit_sum(x)
    if tot > max:
        max = tot
        res = x

print(res)
```
---

## 2.7. Prime Number
자연수 N이 입력되면 1부터 N까지의 소수의 개수를 출력하는 프로그램을 작성하세요.   
예를 들어 20이 입력되면 1부터 20까지의 소수는 2, 3, 5, 7, 11, 13, 17, 19로 총 8개입니다. (제한시간은 1초)   
```
n = int(input())
plist = [0]*(n+1)

cnt = 0
for i in range(2,n+1):
    if plist[i] == 0:
        cnt += 1
        for j in range(i,n+1,i):
            plist[j] = 1

print(cnt)
```
---

## 2.8. Prime with digits reversed
N개의 자연수가 입력되면 각 자연수를 뒤집은 후 그 뒤집은 수가 소수이면 그 수를 출력하는 프로그램을 작성하세요.   
예를 들어 32를 뒤집으면 23이고, 23은 소수이다. 그러면 23을 출력한다.   
단 910를 뒤집으면 19로 숫자화 해야 한다. 첫 자리부터의 연속된 0은 무시한다.  
```
n = int(input())
a = list(map(int, input().split()))

def reverse(x):
    res = 0
    while x>0:
        t = x%10
        res = res*10+t
        x = x//10
    return res

def isPrime(x):
    if x==1:
        return False
    for i in range(2,x//2+1):
        if x%i==0:
            return False
    else: 
        return True

for x in a:
    tmp = reverse(x)
    if isPrime(tmp):
        print(tmp, end=' ')
```
---

## 2.9. Dice game
1에서부터 6까지의 눈을 가진 3개의 주사위를 던져서 다음과 같은 규칙에 따라 상금을 받는 게임이 있다.  
규칙(1) 같은 눈이 3개가 나오면 10,000원+(같은 눈)\*1,000원의 상금을 받게 된다.   
규칙(2) 같은 눈이 2개만 나오는 경우에는 1,000원+(같은 눈)\*100원의 상금을 받게 된다.   
규칙(3) 모두 다른 눈이 나오는 경우에는 (그 중 가장 큰 눈)\*100원의 상금을 받게 된다.   
예를 들어, 3개의 눈 3, 3, 6이 주어지면 상금은 1,000+3\*100으로 계산되어 1,300원을 받게 된다.   
또 3개의 눈이 2, 2, 2로 주어지면 10,000+2\*1,000 으로 계산되어 12,000원을 받게 된다.   
3개의 눈이 6, 2, 5로 주어지면 그 중 가장 큰 값이 6이므로 6\*100으로 계산되어 600원을 상금으로 받게 된다.  
N 명이 주사위 게임에 참여하였을 때, 가장 많은 상금을 받은 사람의 상금을 출력하는 프로그램을 작성하세요.  
```
n = int(input())
sum = 0
for i in range(n):
    tmp = input().split()
    tmp.sort(reverse=True)
    a,b,c = map(int, tmp)
    if a==b and a==c:
        sum += 10000 + (a * 1000)
    elif a==b:
        sum += 1000 + (a * 100)
    elif b==c:
        sum += 1000 + (b * 100)
    else:
        sum += (a * 100)
    
    print(sum)
```
---

## 2.10. Calculate a score
OX 문제는 맞거나 틀린 두 경우의 답을 가지는 문제를 말한다.   
여러 개의 OX 문제로 만들어진 시험에서 연속적으로 답을 맞히는 경우에는 가산점을 주기 위해서 다음과 같이 점수 계산을 하기로 하였다.  
1번 문제가 맞는 경우에는 1점으로 계산한다. 앞의 문제에 대해서는 답을 틀리다가 답이 맞는 처음 문제는 1점으로 계산한다.  
또한, 연속으로 문제의 답이 맞는 경우에서 두 번째 문제는 2점, 세 번째 문제는 3점, ..., K번째 문제는 K점으로 계산한다.  
틀린 문제는 0점으로 계산한다.   
예를 들어, 아래와 같이 10 개의 OX 문제에서 답이 맞은 문제의 경우에는 1로 표시하고, 틀린 경우에는 0으로 표시하였을 때, 점수 계산은 아래 표와 같이 계산되어, 총 점수는 1+1+2+3+1+2=10 점이다.  
  
|채점|1|0|1|1|1|0|0|1|1|0|
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|점수|1|0|1|2|3|0|0|1|2|0|
  
시험문제의 채점 결과가 주어졌을 때, 총 점수를 계산하는 프로그램을 작성하세요.   
```
n = int(input())
a = list(map(int, input().split()))

sum = 0
cnt = 0
for x in a:
    if x == 1:
        cnt += 1
        sum += cnt
    else:
        cnt = 0
print(sum)
```
---


