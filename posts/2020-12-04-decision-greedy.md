---
aliases:
- /algorithm/python/2020/12/04/decision-greedy
categories:
- algorithm
- python
date: '2020-12-04'
description: section4
layout: post
title: Decision & Greedy algorithm
toc: false

---

Python Algorithm Practice

---

## 4.1. Binary search  
임의의 N개의 숫자가 입력으로 주어집니다. N개의 수를 오름차순으로 정렬한 다음 N개의 수 중 한 개의 수인 M이 주어지면 이분검색으로 M이 정렬된 상태에서 몇 번째에 있는지 구하는 프로그램을 작성하세요. 단 중복값은 존재하지 않습니다.  

```
n, m = map(int, input().split())
a = list(map(int, input().split()))
a.sort()
lt = 0
rt = n-1
while lt <= rt:
    mid=(lt+rt)//2
    if a[mid] == m:
        print(mid+1)
        break
    elif a[mid]>m:
        rt = mid-1
    else:
        lt = mid+1
```


---

## 4.2. Cutting LAN cable
한 학원은 자체적으로 K개의 랜선을 가지고 있다. 그러나 K개의 랜선은 길이가 제각각이다. 
선생님은 랜선을 모두 N개의 같은 길이의 랜선으로 만들고 싶었기 때문에 K개의 랜선을 잘라서 만들어야 한다.
예를 들어 300cm 짜리 랜선에서 140cm 짜리 랜선을 두 개 잘라내면 20cm 은 버려야 한다. (이미 자른 랜선은 붙일 수 없다.)
편의를 위해 랜선을 자를때 손실되는 길이는 없다고 가정하며, 기존의 K개의 랜선으로 N개의 랜선을 만들 수 없는 경우는 없다고 가정하자. 
그리고 자를 때는 항상 센티미터 단위로 정수 길이만큼 자른다고 가정하자. 
N개보다 많이 만드는 것도 N개를 만드는 것에 포함된다. 이때 만들 수 있는 최대 랜선의 길이를 구하는 프로그램을 작성하세요.  

```
k, n = map(int,input().split())
line = []
for i in range(k):
    line.append(int(input()))

line.sort()

def count(len):
    cnt = 0
    for x in line:
        cnt += (x//len)
    return cnt


largest = line[k-1] 

lt = 1
rt = largest
while lt <= rt:
    mid = (lt+rt)//2
    if count(mid) >= n:
        res = mid
        lt = mid+1
    else:
        rt = mid-1

print(res)
```


---

## 4.3. DVD recording
가수의 라이브 동영상을 DVD로 만들어 판매하려 한다. 
DVD에는 총 N개의 곡이 들어가는데, DVD에 녹화할 때에는 라이브에서의 순서가 그대로 유지되어야 한다.
즉, 1번 노래와 5번 노래를 같은 DVD에 녹화하기 위해서는 1번과 5번 사이의 모든 노래도 같은 DVD에 녹화해야 한다.
또한 한 노래를 쪼개서 두 개의 DVD에 녹화하면 안된다.
DVD 생산 최소화를 위해 M개가 한 세트인 DVD에 모든 동영상을 녹화하기로 하였다. 
이 때 DVD의 크기(녹화 가능한 길이)를 최소로 하려고 한다. 
그리고 M개의 DVD는 모두 같은 크기여야 제조원가가 적게 들기 때문에 꼭 같은 크기로 해야 한다.

3개의 DVD용량이 17분짜리이면 (1, 2, 3, 4, 5) (6, 7), (8, 9) 이렇게 3개의 DVD로 녹음을 할 수 있다.
17분 용량보다 작은 용량으로는 3개의 DVD에 모든 영상을 녹화할 수 없다.  

```
n, m = map(int, input().split())
music = list(map(int, input().split()))
max_music = max(music)

def count(capacity):
    cnt = 1
    sum = 0
    for x in music:
        if sum+x > capacity:
            cnt += 1
            sum = x
        else:
            sum += x
    return cnt

lt=1
rt=sum(music)

while lt <= rt:
    mid = (lt+rt)//2
    if mid > max_music and count(mid) <= m:
        res = mid
        rt = mid-1
    else:
        lt = mid+1
print(res)
```

---

## 4.4. Deciding Stables size
N개의 마구간이 수직선상에 있습니다. 각 마구간은 x1, x2, x3, ......, xN의 좌표를 가지며, 마구간 간에 좌표가 중복되는 일은 없습니다.
각 마구간에는 한 마리의 말만 넣을 수 있고, 가장 가까운 두 말의 거리가 최대가 되게 말을 마구간에 배치하고 싶습니다. 
C마리의 말을 N개의 마구간에 배치했을 때 가장 가까운 두 말의 거리가 최대가 되는 그 최대값을 출력하는 프로그램을 작성하세요.  

```
n, c = map(int,input().split())
stall = []
for _ in range(n):
    stall.append(int(input()))

def count(len):
    cnt = 1
    ep = stall[0]

    for i in range(1, n):
        if stall[i] - ep >= len:
            cnt += 1
            ep = stall[i]
    return cnt

stall.sort()
lt = 1
rt = stall[n-1]

while lt <= rt:
    mid = (lt+rt)//2
    if count(mid) >= c:
        res = mid
        lt = mid+1
    else:
        rt = mid-1

print(res)
```

---

## 4.5. Allocating meeting room
한 개의 회의실이 있는데 이를 사용하고자 하는 n개의 회의들에 대하여 회의실 사용표를 만들려고 한다.
각 회의에 대해 시작시간과 끝나는 시간이 주어져 있고, 각 회의가 겹치지 않게 하면서 회의실을 사용할 수 있는 최대수의 회의를 찾아라.
단, 회의는 한번 시작하면 중간에 중단될 수 없으며 한 회의가 끝나는 것과 동시에 다음 회의가 시작될 수 있다.  
```
n = int(input())
meeting = []
for i in range(n):
    s, e = map(int,input().split())
    meeting.append((s,e))

# meeting.sort()
meeting.sort(key= lambda x : (x[1], x[0])) # order by end time, start time 순

last_time = 0
cnt = 0
for s, e in meeting:
    if last_time <= s: # 다음 시작시간이 이전 끝나는 시간 보다 같거나 크면 회의실에서 회의 가능
        last_time = e
        cnt += 1
print(cnt)
```
---

## 4.6. Selecting Korean wrestling
현수는 씨름 감독입니다. 현수는 씨름 선수를 선발공고를 냈고, N명의 지원자가 지원을 했습니다.
현수는 각 지원자의 키와 몸무게 정보를 알고 있습니다.
현수는 씨름 선수 선발 원칙을 다음과 같이 정했습니다.
"다른 모든 지원자와 일대일 비교하여 키와 몸무게 중 적어도 하나는 크거나, 무거운 지원자만 뽑기로 했습니다."
만약 A라는 지원자보다 키도 크고 몸무게도 무거운 지원자가 존재한다면 A지원자는 탈락입니다.  

```
n = int(input())
candidate = []
for i in range(n):
    h, w = map(int, input().split())
    candidate.append((h,w))

candidate.sort(key=lambda x: (-x[0]))

cnt = 0
largest = 0
for h, w in candidate:
    if w > largest:
        largest = w
        cnt += 1
    
print(cnt)
```
---

## 4.7. Arranging warehouse storage
창고에 상자가 가로방향으로 일렬로 쌓여 있습니다.  
||||||||  
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|  
| | | | | | | |  
| | |9| | | | |  
| | |8| | |8| |  
| | |7|7| |7| |  
|6| |6|6| |6|6|  
|5| |5|5| |5|5|  
|4| |4|4| |4|4|  
|3|3|3|3| |3|3|  
|2|2|2|2|2|2|2|  
|1|1|1|1|1|1|1|  


만약 가로의 길이가 7이라면 1열은 높이가 6으로 6개의 상자가 쌓여 있고, 2열은 3개의 상자, 3열은 9개의 상자가 쌓여 있으며 높이는 9라고 읽는다.
창고 높이 조정은 가장 높은 곳에 상자를 가장 낮은 곳으로 이동하는 것을 말한다.
가장 높은 곳이나 가장 낮은 곳이 여러곳이면 그 중 아무거나 선택하면 된다.
위에 그림을 1회 높이 조정을 하면 다음과 같아진다.   
||||||||  
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|  
| | | | | | | |  
| | |X| | | | |  
| | |8| | |8| |  
| | |7|7| |7| |  
|6| |6|6| |6|6|  
|5| |5|5| |5|5|  
|4| |4|4| |4|4|  
|3|3|3|3|**3**|3|3|  
|2|2|2|2|2|2|2|  
|1|1|1|1|1|1|1|  


창고의 가로 길이(l)와 각 열의 상자 높이(a)가 주어집니다. m회의 높이 조정을 한 후 가장 높은 곳과 가장 낮은 곳의 차이를 출력하는 프로그램을 작성하세요.  

```
l = int(input())
a = list(map(int,input().split()))
m = int(input())

a.sort()
for _ in range(m):
    a[0] += 1
    a[l-1] -= 1
    a.sort()

print(a[l-1]-a[0])
```

---

## 4.8. Calculating a number of minimum lifeboat in sinking Titanic
침몰 중인 유람선에는 N명의 승객이 타고 있습니다. 
구명보트를 타고 탈출해야 하는데 타이타닉에 있는 구명보트는 2명 이하로만 탈 수 있으며, 보트 한 개에 탈 수 있는 총 무게도 M kg 이하로 제한되어 있습니다.
N명의 승객 몸무게가 주어졌을 때 승객 모두가 탈출하기 위한 구명보트의 최소 개수를 출력하는 프로그램을 작성하세요.  

```
n, limit = map(int, input().split())
a = list(map(int,input().split()))

a.sort()
cnt = 0
while a:
    if len(p)==1:
        cnt += 1
        break
    if a[0] + a[-1] > limit:
        a.pop()
        cnt += 1
    else:
        a.pop(0)
        a.pop()
        cnt += 1
print(cnt)
```

---

## 4.9. Incresing subsequence
1부터 N까지의 모든 자연수로 구성된 길이 N의 수열이 주어집니다. 
이 수열의 왼쪽 맨 끝 숫자 또는 오른쪽 맨 끝 숫자 중 하나를 가져와 나열하여 가장 긴 증가수열을 만듭니다.
이때 수열에서 가져온 숫자(왼쪽 맨 끝 또는 오른쪽 맨 끝)는 그 수열에서 제거됩니다.
예를 들어 2 4 5 1 3 이 주어지면 만들 수 있는 가장 긴 증가수열의 길이는 4입니다.
맨 처음 왼쪽 끝에서 2를 가져오고, 그 다음 오른쪽 끝에서 3을 가져오고, 왼쪽 끝에서 4, 왼쪽 끝에서 5를 가져와 2 3 4 5 증가수열을 만들 수 있습니다.  
```
n = int(input())
a = list(map(int,input().split()))

res = ""
tmp = []
last = 0
l, r = 0, n-1
while l <= r:
    if a[l] > last:
        tmp.append((a[l], "L"))
    if a[r] > last:
        tmp.append((a[r], "R"))
    tmp.sort()
    if len(tmp)==0:
        break
    else:
        res += tmp[0][1]
        last = tmp[0][0]
        if res[-1] == 'L':
            l += 1
        else:
            r -= 1
    tmp.clear()
print(len(res))
print(res)
```
---

## 4.10. Reverse sequence
1부터 n까지의 수를 한 번씩만 사용하여 이루어진 수열이 있을 때, 1부터 n까지 각각의 수 앞에 놓여 있는 자신보다 큰 수들의 개수를 수열로 표현한 것을 역수열이라 한다.  
예를 들어 다음과 같은 수열의 경우 
 4 8 6 2 5 1 3 7
1앞에 놓인 1보다 큰 수는 4, 8, 6, 2, 5. 이렇게 5개이고,
2앞에 놓인 2보다 큰 수는 4, 8, 6. 이렇게 3개,
3앞에 놓인 3보다 큰 수는 4, 8, 6, 5 이렇게 4개...
따라서 4 8 6 2 5 1 3 7의 역수열은 5 3 4 0 2 1 1 0 이 된다.
n과 1부터 n까지의 수를 사용하여 이루어진 수열의 역수열이 주어졌을 때, 원래의 수열을 출력하는 프로그램을 작성하세요.  

```
n = int(input())
a = list(map(int,input().split()))

ori = [0]*n

for i in range(n):
        cnt = 0
        for j in range(n):
                if ori[j] == 0 and cnt == a[i]:
                        ori[j] = i+1
                        break
                if ori[j] == 0:
                        cnt += 1
print(ori)
```

---
