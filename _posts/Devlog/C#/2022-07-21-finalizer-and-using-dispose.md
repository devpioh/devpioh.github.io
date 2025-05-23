---
title: "종료자와 using()/Dispose()"
date: 2022-07-21
last_modified_at: 2022-07-21

toc: true
toc_sticky: true

categories:
    - csharp
tags:
    - [csharp, c#, interface]
---

## 개요

c/c++의 경우 소멸자의 호출 시점은 개발자가 정확히 예상을 할 수 있지만. c#의 경우는 그렇지 않다.

c#의 종료자(소멸자)는 파이널라이저라고 불리는데, 호출 시점은 개발자가 제어할수 없고 가비지 수집기의 의해 결정된다.

이로 인해 몇가지 특징과 제약이 있는데 이를 한번 정리해 보려고 한다.

## 종료자의 특징

- 종료자는 구조체에서 정의할 수 없으며, 오직 클래스에서만 사용된다.
- 종료자는 클래스에서 하나만 있을 수 있다.
- 종료자는 상속하거나 오버로드 할 수 없다.
- 종료자는 명시적으로 호출 할 수 없고, 자동으로 호출된다.
- 종료자는 한정자(public / private등)를 사용할 수 없으며, 매개 변수를 갖지 않는다.

## 종료자의 선언

종료자는 아래와 같이 정의 한다.

```cs
class Man
{
    // finalizer
    ~Man() 
    {
        // 비관리 리소스 해제.
    }
}
```

이렇게 선언된 종료자는 컴파일러에 의해 IL에서는 아래와 같은 코드로 변환된다.

```cs
protected override void Fianlize()
{
    try
    {
        // 비관리 리소스 해제
    }
    finally
    {
        base.Fianlize();
    }
}
```

## 호출시점과 동작원리

종료자는 c/c++의 소멸자와는 달리 특별한 방식으로 호출된다.

1. 종료자의 등록
    종료자가 정의된 객체가 생성되면, CLR은 내부적으로 해당 객체들을 종료자 큐(finalization queue)라는 곳에 등록한다.
2. 가비지 수집 시
    GC가 "더 이상 참조되지 않는" 객체들을 발견하면,
    - 1차 단계 : 종료자 큐에 등록된 객체는 바로 해제 되지 않고,
    - 2차 단계 : 종료자 스레드(finalizer thread)가 해당 객체의 Fianlize 메서드를 호출한 뒤에 메모리가 해제된다.
3. 비결정적 실행
    - 가비지 수집기가 언제 실행되는지는 비결정적(non-deterministic)이다.
    - 프로그램이 종료된 직후, 또는 GC 타이밍(GC가 수행되는 시점은 메모리가 부족하다고 GC가 판단하는 시점. 특히나 세대별 GC인 경우 세대에 따라 GC가 실행되는 시점이 세대별로 다르므로 더욱 더 알기 힘들다.)에 따라 지연 될 수 있다.

## 출처 및 같이 보기

- [종료자](https://learn.microsoft.com/ko-kr/dotnet/csharp/programming-guide/classes-and-structs/finalizers)
