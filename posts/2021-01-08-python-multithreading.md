---
aliases:
- /python/2021/01/08/python-multithreading
categories:
- python
date: '2021-01-08'
description: section8
layout: post
title: Python Multi-threading
toc: true

---

Python Intermediate Programming2

---

## 8.1. Process & Thread

### Process:  
- any program is in execution  
- 독립적인 Process Control Block(PCB), Stack, Address Space 가짐  
- Code, Data, Stack, Heap 독립적으로 할당  
- 최소 1개의 메인스레드 보유  
- 생성, 소멸, 통신(Context Switching via pipe, socket, file...)하는데 cost가 높음  
- 프로세스 전환시 OS 호출 필요  

### Thread:  
- segment of a process  
- 부모인 프로세스의 자원 사용  
- Stack만 별도 할당 나머지는 자원 공유(Code, Data, Heap)  
- 메모리 공유(변수 공유)  
- 한 스레드의 결과가 다른 스레드에 영향 끼침  
- 하나 스레드 block 시 다른 스레드도 blocked(디버깅 어려움, 동기화 주의)  
- 스레드 전환시 OS 호출 및 kernel 중단 불필요  

### Multi-process:  
- 한 개의 프로그램을 여러 프로세스로 구성하여 각 프로세스가 하나의 작업(task) 처리  
- 부모-자식 관계라고 해도 각각 독립적인 메모리 영역 가짐  
- fork를 통해 프로세스 복사  
- 프로세스 간 통신 시 IPC(Inter Process Communication: 세마포어, 큐, 공유메모리)라는 복잡한 통신 방식 사용  
- 캐시 체인지, context switching(캐시메모리 초기화) 비용 매우 높음(오버헤드)  

### Multi-thread:  
- 한 개의 프로그램을 여러 스레드로 구성 후 각 스레드가 하나의 작업(task) 처리  
- 스레드간 메모리 자원 공유(Stack 영역 제외), 시스템 자원 소모 감소(효율성), 처리량 증가(Cost 감소)  
- 디버깅 어려움, 동기화 이슈 발생, 교착 상태 발생, 하나의 스레드의 오류로 전체 프로세스에 문제 발생  
- 멀티 스레드는 하나의 프로세스에서 자원을 공유하므로 전역변수에 대해 두 개 이상의 스레드가 동시에 접근하려면 mutex[^1] 개념 필요

<br><br>
---

## 8.2. Multi-threading

python은 interpreter 언어로, single thread에 최적화되어 순차적으로 동작함 (한번에 하나의 작업)  
한번에 여러 작업을 하기 위해서(multi-threading) 기존의 CPython의 GIL이라는 개념을 가져와야 했음  

### GIL(Global Interpreter Lock):  
파이썬의 bytecode 실행 시 여러 스레드 중 하나의 스레드만 파이썬 객체에 접근할 수 있게 하는 mutex임  
하나의 프로세스 당 하나의 스레드만 점유

#### Multi-threading 단점
I/O작업(파일 읽기쓰기, 네트워크 통신)이 아닌 CPU작업(행렬연산, 이미지처리)을 많이 사용하는 작업은 멀티 스레드 사용 시 성능 떨어짐  

보완하기 위해서...  
- multi-threading 작업말고 프로세스를 사용하는 다양한 라이브러리(Numpy/Scipy)등으로도 효율적인 코딩 가능  
- 또한 병렬 처리는 Multiprocessing, asyncio 선택지 다양함  
- thread 동시성 처리를 위한 Jython, IronPython, Stackless Python 등이 존재  

#### 그럼에도 불구하고 Multi-threading 사용하는 이유
CPython과 달리 파이썬의 [메모리 관리 방식(Thread-safe)](https://hongl.tistory.com/262)은 오버헤드가 거의 없음  



semaphore[^2] 

[^1]뮤텍스(Mutex): 하나의 프로세스의 공유 자원을 하나의 스레드만이 점유하도록 하는 장치(Lock 사용)  
[^2]세마포어(Semaphore): 멀티 프로그래밍 환경에서 공유된 자원에 대한 접근을 제한하는 방법(최대 허용치 존재)  


### Daemon & Join


<br><br>
---
## 8.3. Multi-processing


<br><br>
---











