---
title: "Garbage Collection"
date: 2022-07-11
last_modified_at: 2022-07-11

toc: true
toc_sticky: true

categories:
    - csharp
tag:
    - [csharp, c#, Garbage Collection, GC]
---

## 개요
이전 포스트 [Boxing-Unboxing]({% post_url /Devlog/C#/2022-07-11-Boxing-Unboxing %})에서 박싱과 언박싱 시 가비지가 발생되는 이유를 이야기 했다.

이번 포스트에서는 CLR의 가비지 컬렉션의 동작 원리를 간단하게 알아보고 GC로 인해 생기는 성능저하의 이유를 정리해 본다.

### 동작 원리
c/c++ 과는 달리 c#은 CLR(Common Language Runtime-공통 언어 런타임)이라는 가상머신에서 동작을 하는데, 
   
이 CLR의 특징중 하나가 쓰레기 수집 즉, 가비지 컬렉션 기능이다.

c/c++은 힙을 시스템 환경에서 관리하지 않고 사용자가 할당/해제를 해줘야된다. 
  
반면 C#과 같은 CLR에서 동작하는 언어의 경우 가상머신이 직접 힙 메모리를 관리하는데 이를 관리되는 힙(Managed Heap)이라 한다.

이 관리되는 힙의 메모리 할당은 상당히 간단한 방식인데 아래의 그림을 보도록하자.
   
![Managed Heap alloc memory]()
   
단순히 마지막에 할당한 메모리의 포인터 위치를 기억하고 있다가 할당 요청이 오면 해당 포인터에서 요청받은 크기만큼 메모리를 할당하고 0으로 초기화 한다.
  
이런 특성으로 인해 빈번한 메모리의 할당도 크게 부담스럽지 않다.(c/c++의 경우 잦은 할당/해제의 경우 메모리 파편화의 문제가 생긴다.)
  
그럼 이제 GC는 어떤 방식으로 동작을 하게 될까?

GC는 별도 쓰레드(GC 쓰레드)에서 동작이 되는데 평소 GC 쓰레드는 멈춰 있다가 메모리가 부족하다고 판단 시 모든 쓰레드를 중단시키고 GC 쓰레드를 동작시킨다.

이때 GC 쓰레드는 쓰레기를 판별하는 기준이 필요한데 이게 참조 그래프(루트)목록이다.

.Net 어플리케이션 실행 시 컴파일러는 모든 오브젝트의 참조 관계를 조사 하여 루트 목록을 만든다.
  
이 루트 참조는 현재 쓰레드의 로컬변수 / 정적 필드 / 전역 변수 / CPU 레지스터가 참조중인 변수 / 현재 사용중인 타입들 인데, 
   
GC는 이 참조 루트를 순회 하면서 각 루트를 참조하는 힙 객체들의 관계를 조사 하여 참조 관계에 걸려있지 않으면 쓰레기로 판단하여 수거한다.
   
![Collect Garbage and Memory Compaction](https://docs.microsoft.com/ko-kr/dotnet/standard/garbage-collection/media/loh/loh-figure-1.jpg)

위의 그림처럼 쓰레기로 판단되어 수거된 메모리의 위치에 생존한 메모리를 재배치 시키는데 이러한 작업을 메모리 컴팩션(Compaction)이라고 한다.

### 세대별 가비지 컬렉션(Generational Garbage Collection)
.net은 GC의 최적화 알고리즘들 중에서 세대별 가비지 컬렉션이라는 방식을 사용한다.

객체의 생존 시간에 따라 0세대부터 2세대까지 세대를 구분해 놓고 각 세대에 맞게 가비지 컬렉션을 수행한다.

가장 최근에 생성된 객체는 0세대, 0세대에서 가비지 컬렉션이 수행되고 살아 남은 객체는 1세대, 또 여기서 살아나은 객체는 2세대가 되는 방식이다.
   
빈번하게 가비지 컬렉션을 수행하는 세대는 0세대이고 세대가 증가될 수록 가비지 컬렉션의 수행은 거이 하지 않는다.

0세대만을 집중적으로 가비지 컬렉션을 하는 근거는 아래와 같다.
 * 최근 생성된 객체일수록 생존 시간은 짧다.(작은 객체일수록 생성 된 후 짧은 시간동안 사용되고 더 이상 사용되지 않는다.)
 * 오래된 객체일수록 생존 시간은 길다.
 * 최근에 생성된 객체들끼리는 서로 연관성(참조)이 높으며 비슷한 시점에서 자주 액세스 된다.
 * 일부분의 힙을 가비지 컬렉션 하는것이 전체를 가비지 컬렉션 하는것보다 빠르다.

물론 다른 세대들이 전혀 가비지 컬렉션을 수행하지 않는것은 아니다. 
   
1세대의 경우 0세대에서 살아남은 객체가 많다면 1세대로 넘어오게되는데 1세대의 메모리 허용량이 부족해지면 1세대에 대한 가비지 컬렉션을 수행한다.
   
여기서도 살아남은 객체들은 2세대로 넘어가는데 역시 2세대의 메모리 허용량이 부족해지면 이때는 모든 쓰레드의 동작을 멈추고 전체 오브젝트에 대한 가비지 컬렉션을 수행한다.
   
### LOH(Large Object Heap)
CLR은 객체의 크기를 85kb를 기준으로 크거나 같은 경우는 LOH(Large Object Heap)에 할당하고 작은 경우는 SOH(Small Object Heap)에 할당한다.
   
지금까지 알아본것은 85kb 미만의 객체들, SOH에 대한 GC이다. LOH는 SOH와는 다르게 GC가 동작한다.

SOH GC는 쓰레기가 수거된 메모리의 위치를 재조정하는 메모리 컴팩션 과정을 수행한다고 이야기 했었다.
   
하지만 LOH는 해당 메모리 컴팩션 과정을 수행하지 않는다. 크기가 큰 객체의 힙 메모리 위치에 대한 재조정은 득보다 실이 더 크기 때문이다.
  
그럼 LOH에서는 메모리와 GC가 어떻게 수행되는것일까? 아래의 그림을 보자.
   
![LOH Garbage Collection](https://docs.microsoft.com/ko-kr/dotnet/standard/garbage-collection/media/loh/loh-figure-2.jpg)
   
그림을 보면 Obj1과 Obj2가 쓰레기로 판단되어 수거되었다. 해당 메모리는 청소(Sweap)되어 그 크기만큼 다른 큰 객체를 받을수있는 공간이 생겼다.
  
이후 큰 객체 Obj4의 할당 요청이 오면 CLR은 LOH를 한번 훑어서 해당 객체가 할당 할 수 있는 메모리를 찾아 할당시켜준다.
  
이는 c/c++의 메모리 할당/해제와 유사하다. 이 말인즉 CRL 환경에서도 메모리 파편화 현상이 발생이 된다는 말이다.
   

## 출처 및 같이 보기
- <a href="https://docs.microsoft.com/ko-kr/dotnet/standard/garbage-collection/fundamentals">MSDN 가비지 컬렉션</a>
- <a herf="https://docs.microsoft.com/ko-kr/dotnet/standard/garbage-collection/large-object-heap">MSDN LOH</a>
- <a href="http://www.simpleisbest.net/post/2011/04/01/Review-NET-Garbage-Collection.aspx">SimpleIsBest.Net 가비지 컬렉션 포스트</a>   
(추천!! 오래된 포스팅이지만 기본 원리를 쉽고 재밌게 풀어서 설명해 주셨음) 
