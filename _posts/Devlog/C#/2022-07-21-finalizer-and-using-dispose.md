---
title: "종료자와 using()/Dispose()"
date: 2022-07-21
last_modified_at: 2022-07-21

toc: true
toc_sticky: true

categories:
    - csharp
tags:
    - [csharp, c#, IDisposable, Using()]
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

## IDisposable

종료자는 CLR이 관리되지 않는 리소스(Unmanaged Resources)를 해제 하기 위하여 사용된다.

> 관리되지 않는 리소스는 아래와 같이 정리 할 수 있다.
    1. 파일핸들(FileStream, StreamWriter 등)
    2. 네트워크 소켓(Socket, TcpClient 등)
    3. 데이터베이스 연결(SqlConnection)
    4. 유니티 엔진의 경우 ComputeBuffer, Texture, Font등 네이티브 메모리를 사용하는 객체들

하지만 종료자가 언제 호출될지는 개발자가 알 수 없기 때문에 관리되지 않는 리소스를 실수로 다시 호출 하게 되면 해당 리소스는 메모리에서 해제 되지 못해 메모리 누수가 발생하게 된다.

이런 문제를 해결하기 위해 c#에서는 IDisposable 패턴을 사용한다.

### IDisposable 인터페이스

IDisposable 패턴은 아래의 인터페이스를 구현으로 적용가능하다.

```cs
public interface IDisposable
{
    void Dispose();
}
```

사용 예시:

```cs

using(var stream = new FileStream("data.txt", FileMode.Open))
{
    // 파일 작업 수행

} // Using 블럭이 끝나는 경우 자동으로 stream.Dispose() 호출


// 또는

var streamCall = new FileStream("data2.txt", FileMode.Open);

// 파일 작업 수행 후

streamCall.Dispose(); // Dispose()를 호출.
```

일반적으로 IDisposable 패턴은 Using() 블럭과 함께 사용하지만, "streamCall" 객체처럼 Dispose()를 호출 하므로써 명시적으로 관리되지 않는 리소스를 해제 할 수 있다.

## Using()

c# 에서는 Using()문과 같이 사용하는 경우 Dispose()를 자동으로 호출한다.

```cs
using (var reader = new StreamReader("data.txt"))
{
    string message = reader.ReadToEnd();
}
```

위의 구문은 아래의 구문과 완전히 동일하다.

```cs
var reader = new StreamReader("data.txt");

try
{
    string message = reader.ReadToEnd();
}
finally
{
    reader.Dispose();
}
```

Using()구문은 c#에서 자원을 안전하게 해제하기 위한 문법적 설탕(syntactic sugar)으로 IDisposable를 구현한 객체는 Using()블럭으로 감싸진 경우 컴파일러에 의해 try...finally 구조로 변경되어 Dispose()를 자동으로 호출하게 된다.

## 출처 및 같이 보기

- [종료자](https://learn.microsoft.com/ko-kr/dotnet/csharp/programming-guide/classes-and-structs/finalizers)
