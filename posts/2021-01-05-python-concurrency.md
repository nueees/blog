---
aliases:
- /python/2021/01/05/python-concurrency
categories:
- python
date: '2021-01-05'
description: section5
layout: post
title: Python Concurrency
toc: true

---

Python Intermediate Programming

---

## 5.1. Generator Pattern
작은 메모리 조각 사용  
단위 실행 가능한 코루틴(Coroutine) 구현과 연동  

### iter function
파이썬 반복 가능한 타입: for, collections, text file, List, Dict, Set, Tuple, unpacking, \*args  

### next

```
class WordSplitGenerator:
    def __init__(self, text):
        self._text = text.split(' ')
    
    def __iter__(self):
        # print('Called __iter__')
        for word in self._text:
           yield word # 제네레이터
        return
    
    def __repr__(self):
        return 'WordSplit(%s)' % (self._text)


wg = WordSplitGenerator('One shot No break')
print(wg)
> WordSplit(['One', 'shot', 'No', 'break'])
wt = iter(wg)
print(wt)
> <generator object WordSplitGenerator.__iter__ at 0x000002142675C200>

print(next(wt)) # 내부적으로 iter 호출
> Called __iter__
> 'One'
print(next(wt)) # iter에서 yield 시점부터 재 호출
> 'shot'
print(next(wt))
> 'No'
print(next(wt))
> 'break'
print(next(wt))
>Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
  StopIteration

```
### genetator 생성

```
def generator_ex1():
    print('Start')
    yield 'A Point.'
    print('continue')
    yield 'B Point.'
    print('End')
```

```
temp2 = [x * 3 for x in generator_ex1()]

print(temp2)
> ['A Point.A Point.A Point.', 'B Point.B Point.B Point.']

for i in temp2:
    print(i)
> A Point.A Point.A Point.
> B Point.B Point.B Point.
```

```
temp3 = (x * 3 for x in generator_ex1())
print(temp3)
> <generator object <genexpr> at 0x000002142685DA50>

for i in temp3:
    print(i)
> Start
> A Point.A Point.A Point.
> continue
> B Point.B Point.B Point.
> End
```

---

## 5.2. Generator Functions

```
import itertools
```

### takewhile
```
gen2 = itertools.takewhile(lambda n : n < 3, [1,2,3,4,5,5,4,3,2,1])
for v in gen2:
    print(v)
> 1
> 2
```

### filter랑 비교   
```
gen1 = filter(lambda n : n < 3, [1,2,3,4,5,5,4,3,2,1])
for v in gen1:
    print(v)
> 1
> 2
> 2
> 1
```

### filterfalse
```
gen3 = itertools.filterfalse(lambda n : n < 3, [1,2,3,4,5,5,4,3,2,1])
for v in gen3:
    print(v)
> 3
> 4
> 5
> 5
> 4
> 3
```

### accumulate
```
gen4 = itertools.accumulate([x for x in range(1, 6)])
for v in gen4:
    print(v)
> 1
> 3
> 6
> 10
> 15
```

### chain
```
gen5 = itertools.chain('ABCDE', range(1,11,2))
print(list(gen5))
> ['A', 'B', 'C', 'D', 'E', 1, 3, 5, 7, 9]
gen6 = itertools.chain(enumerate('ABCDE'))
print(list(gen6))
> [(0, 'A'), (1, 'B'), (2, 'C'), (3, 'D'), (4, 'E')]
```

### product
```
gen7 = itertools.product('ABCDE')
print(list(gen7))
> [('A',), ('B',), ('C',), ('D',), ('E',)]

gen8 = itertools.product('ABCDE', repeat=2) # permutation 5P2 경우의 수 전부 출력
print(list(gen8))
> [('A', 'A'), ('A', 'B'), ('A', 'C'), ('A', 'D'), ('A', 'E'), ('B', 'A'), ('B', 'B'), ('B', 'C'), ('B', 
'D'), ('B', 'E'), ('C', 'A'), ('C', 'B'), ('C', 'C'), ('C', 'D'), ('C', 'E'), ('D', 'A'), ('D', 'B'), ('D', 'C'), ('D', 'D'), ('D', 'E'), ('E', 'A'), ('E', 'B'), ('E', 'C'), ('E', 'D'), ('E', 'E')]
```


### groupby
```
gen9 = itertools.groupby('AAABBCCCCDDEEE')
print(list(gen9))
> [('A', <itertools._grouper object at 0x0000021426987B20>)
, ('B', <itertools._grouper object at 0x0000021426987700>)
, ('C', <itertools._grouper object at 0x0000021426987BB0>)
, ('D', <itertools._grouper object at 0x00000214269879A0>)
, ('E', <itertools._grouper object at 0x0000021426987A90>)]

gen9 = itertools.groupby('AAABBCCCCDDEEE') # 위에 iter 다 돌아서 다시 생성해서 돌려줌
for chr, group in gen9:
    print(chr, ' : ', list(group))
> A  :  ['A', 'A', 'A']
> B  :  ['B', 'B']
> C  :  ['C', 'C', 'C', 'C']
> D  :  ['D', 'D']
> E  :  ['E', 'E', 'E']

```

---

## 5.3. Coroutine
동시성 프로그래밍을 위해 routine 실행 중 중지  
coroutine: multi thread 코드는 thread간 deadlock, context switching, resource 소비 등을 신경써야 하며 bug 발생률이 높아짐  
single thread로 동작하도록 만든게 Coroutine  

### yield
main <-> sub  
왼쪽(y)에 있으면 main->sub로 받는 것 오른쪽(x)에 있으면 sub->main으로 주는 것  
ex) y = yield x  

```
from inspect import getgeneratorstate

def coroutine2(x): # sub
    cnt = 1
    print('>>> coroutine started : {}, {}'.format(x,cnt))
    y = yield x # main으로 x(10)전달 # 다음 실행시 main에서 y(20)받음
    cnt += 1
    print('>>> coroutine y received : {}, {}'.format(y,cnt))
    z = yield x - y # main으로 x-y(10-20) # 다음 실행시 main에서 z(30)받음
    cnt += 1 
    print('>>> coroutine z received : {}, {}'.format(z,cnt))
    
# main
cr3 = coroutine2(10) 
print(getgeneratorstate(cr3))
> GEN_CREATED

print(next(cr3)) # x 반환하므로
> 10
print(getgeneratorstate(cr3))
> GEN_SUSPENDED

print(cr3.send(20)) # x-y 반환하므로
> -10

print(cr3.send(30)) #  error 
> coroutine received : 30, 3
> StopIteration error

```
---

## 5.4. Futures

### map
```
import os
import time
from concurrent import futures

WORK_LIST = [100000, 1000000, 10000000, 10000000]

# concurrency control
def sum_generator(n):
    return sum(n for n in range(1, n+1))

def main():
    # Worker Count
    worker = min(10, len(WORK_LIST))
    # 시작 시간
    start_tm = time.time()
    # 결과 건수
    # ProcessPoolExecutor
    with futures.ThreadPoolExecutor() as excutor:
        # map -> 작업 순서 유지, 즉시 실행
        result = excutor.map(sum_generator, WORK_LIST)
    # 종료 시간
    end_tm = time.time() - start_tm
    # 출력 포멧
    msg = '\n Result -> {} Time : {:.2f}s'
    # 최종 결과 출력
    print(msg.format(list(result), end_tm))


if __name__ == '__main__':
    main()
> Result -> [5000050000, 500000500000, 50000005000000, 50000005000000] Time : 4.53s
```
### wait, as_completed
```
def sum_generator(n):
    return sum(n for n in range(1, n+1))

# wait
# as_completed
def main():
    # Worker Count
    worker = min(10, len(WORK_LIST))
    
    # 시작 시간
    start_tm = time.time()
    # Futures
    futures_list = []

    # 결과 건수
    # ProcessPoolExecutor
    with ThreadPoolExecutor() as excutor:
        for work in WORK_LIST:
            # future 반환
            future = excutor.submit(sum_generator, work)
            # 스케쥴링
            futures_list.append(future)
            # 스케쥴링 확인
            print('Scheduled for {} : {}'.format(work, future))
            # print()
        
        # wait 결과 출력
        # result = wait(futures_list, timeout=7)
        # # 성공
        # print('Completed Tasks : ' + str(result.done))
        # # 실패
        # print('Pending ones after waiting for 7seconds : ' + str(result.not_done))
        # # 결과 값 출력
        # print([future.result() for future in result.done])
        
        # as_completed 결과 출력
        for future in as_completed(futures_list):
            result = future.result()
            done = future.done()
            cancelled = future.cancelled
            
            # future 결과 확인
            print('Future Result : {}, Done : {}'.format(result, done))
            print('Future Cancelled : {}'.format(cancelled))
        
    # 종료 시간
    end_tm = time.time() - start_tm
    # 출력 포멧
    msg = '\n Time : {:.2f}s'
    # 최종 결과 출력
    print(msg.format(end_tm))

if __name__ == '__main__':
    main()

> Scheduled for 10000 : <Future at 0x20268c4a280 state=finished returned int>
  Scheduled for 100000 : <Future at 0x20268c4a400 state=pending>
  Scheduled for 1000000 : <Future at 0x20268d7c1c0 state=running>
  Scheduled for 10000000 : <Future at 0x20268d7c190 state=pending>
  Future Result : 5000050000, Done : True
  Future Cancelled : <bound method Future.cancelled of <Future at 0x20268c4a400 state=finished returned int>>
  Future Result : 50005000, Done : True
  Future Cancelled : <bound method Future.cancelled of <Future at 0x20268c4a280 state=finished returned int>>
  Future Result : 500000500000, Done : True
  Future Cancelled : <bound method Future.cancelled of <Future at 0x20268d7c1c0 state=finished returned int>>
  Future Result : 50000005000000, Done : True
  Future Cancelled : <bound method Future.cancelled of <Future at 0x20268d7c190 state=finished returned int>>

  Time : 2.10s
```


---


## 5.5. asyncio


### async
```
import asyncio
import timeit
from urllib.request import urlopen
from concurrent.futures import ThreadPoolExecutor
import threading

# 실행 시작 시간
start = timeit.default_timer()
# 서비스 방향이 비슷한 사이트로 실습 권장(예 : 게시판성 커뮤니티)
urls = ['http://daum.net', 'https://naver.com', 'http://mlbpark.donga.com/', 'https://tistory.com', 'https://wemakeprice.com/']


async def fetch(url, executor):
    # 쓰레드명 출력
    print('Thread Name :', threading.current_thread().getName(), 'Start', url)
    # 실행
    res = await loop.run_in_executor(executor, urlopen, url)
    print('Thread Name :', threading.current_thread().getName(), 'Done', url)
    # 결과 반환
    return res.read()[0:5]

async def main():
    # 쓰레드 풀 생성
    executor = ThreadPoolExecutor(max_workers=10)

    # future 객체 모아서 gather에서 실행
    futures = [
        asyncio.ensure_future(fetch(url, executor)) for url in urls
    ]
    
    # 결과 취합
    rst = await asyncio.gather(*futures)

    print()
    print('Result : ', rst)

if __name__ == '__main__':
    # 루프 초기화
    loop = asyncio.get_event_loop()
    # 작업 완료 까지 대기
    loop.run_until_complete(main())
    # 수행 시간 계산
    duration = timeit.default_timer() - start
    # 총 실행 시간
    print('Total Running Time : ', duration)
> Thread Name : MainThread Start http://daum.net
    Thread Name : MainThread Start https://naver.com
    Thread Name : MainThread Start http://mlbpark.donga.com/
    Thread Name : MainThread Start https://tistory.com
    Thread Name : MainThread Start https://wemakeprice.com/
    Thread Name : MainThread Done http://mlbpark.donga.com/
    Thread Name : MainThread Done https://naver.com
    Thread Name : MainThread Done http://daum.net
    Thread Name : MainThread Done https://tistory.com
    Thread Name : MainThread Done https://wemakeprice.com/

    Result :  [b'<!DOC', b'\n<!do', b'<!DOC', b'\n\t<!d', b'\x1f\x8b\x08\x00\x00']
    Total Running Time :  6.3161305

```

### await

```
import asyncio
import timeit
from urllib.request import urlopen
from concurrent.futures import ThreadPoolExecutor
import threading
from bs4 import BeautifulSoup

import sys
import io
sys.stdout = io.TextIOWrapper(sys.stdout.detach(), encoding = 'utf-8')
sys.stderr = io.TextIOWrapper(sys.stderr.detach(), encoding = 'utf-8')

# 실행 시작 시간
start = timeit.default_timer()
# 서비스 방향이 비슷한 사이트로 실습 권장(예 : 게시판성 커뮤니티)
urls = ['http://daum.net', 'https://naver.com', 'http://mlbpark.donga.com/', 'https://tistory.com', 'https://www.inflearn.com/']


async def fetch(url, executor):
    # 쓰레드명 출력
    print('Thread Name :', threading.current_thread().getName(), 'Start', url)
    # 실행
    res = await loop.run_in_executor(executor, urlopen, url)
    
    soup = BeautifulSoup(res.read(), 'html.parser')
    
    # 전체 페이지 소스 확인
    # print(soup.prettify())
    # 이 부분에서 BeautifulSoup Selector(선택자)를 활용해서 다양한 정보 가져오기 가능
    # 현 예제에서는 페이지 타이틀 정보 수집
    result_data = soup.title

    print('Thread Name :', threading.current_thread().getName(), 'Done', url)
    # 결과 반환
    return result_data

async def main():
    # 쓰레드 풀 생성
    executor = ThreadPoolExecutor(max_workers=10)

    # future 객체 모아서 gather에서 실행
    futures = [
        asyncio.ensure_future(fetch(url, executor)) for url in urls
    ]
    
    # 결과 취합
    rst = await asyncio.gather(*futures)

    print()
    print('Result : ', rst)

if __name__ == '__main__':
    # 루프 초기화
    loop = asyncio.get_event_loop()
    # 작업 완료 까지 대기
    loop.run_until_complete(main())
    # 수행 시간 계산
    duration = timeit.default_timer() - start
    # 총 실행 시간
    print('Total Running Time : ', duration)
 
 > Thread Name : MainThread Start http://daum.net
    Thread Name : MainThread Start https://naver.com
    Thread Name : MainThread Start http://mlbpark.donga.com/
    Thread Name : MainThread Start https://tistory.com
    Thread Name : MainThread Start https://www.inflearn.com/
    Thread Name : MainThread Done http://mlbpark.donga.com/
    Thread Name : MainThread Done https://www.inflearn.com/
    Thread Name : MainThread Done https://naver.com
    Thread Name : MainThread Done http://daum.net
    Thread Name : MainThread Done https://tistory.com

    Result :  [<title>Daum</title>, <title>NAVER</title>, <title>↗ 파크에 오면 즐겁다 MLBPARK</title>, <title>TISTORY</title>, <title>인프런 - 프로가 되는 온라인 클래스 | 온라인 강의 플랫폼</title>]
    Total Running Time :  6.4905200999999995
 
 
 ```



---
