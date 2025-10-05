---
title: "Boxing과 Unboxing"
date: 2022-07-11
last_modified_at: 2025-05-20

toc: true
toc_sticky: true

categories:
    - csharp
tags:
    - [csharp, c#]
---

## 개요

> 2025-05-20 포스팅한 글에 잘못된 정보가 많아 해당 부분을 수정.

CTS(공통 형식 시스템)에서는 값 타입과 참조 타입 2가지의 타입이 있다.
  
두 타입은 메모리에 할당 되는 방식에 차이가 있는데, 값 타입의 경우 스택에 할당되는 반면 참조 타입의 경우 힙 메모리에 할당된다.

> 값 타입도 다음과 같은 경우 힙 메모리에 할당된다.
    1. 참조 타입의 객체 필드로 포함되는 경우.
    2. 배열과 같은 컬렉션의 요소로 저장.
    3. Jit 최적화나 레지스터 할당으로 인해 스택 되신에 cpu 레지스터에 보관되는 경우.
  
즉 값 타입의 경우 함수가 끝나는 동시에 스택 메모리에서 해제되지만, 참조 타입의 경우는 GC(가비지 컬렉터)에 의해 해제된다.
  
이러한 차이로 인해 값 타입이 참조 타입에 캐스팅 될때와 참조 타입에서 값타입으로 캐스팅 될때 복잡한 과정이 생기는데, 이를 Boxing / Unboxing 이라고 한다.

### Boxing

아래의 코드를 보자

```cs
private void DoSomething()
{
    int a = 123;
        
    int b = a;          // a의 값은 복사되어 b에 저장
    object o = a;       // Boxing 변환 발생
}
```

변수 a에 데이터 123이 저장되고, 다시 변수 b에 a를 대입 시킨다.

이 때는 둘다 값 타입이라 a의 값인 123이 복사되어 b에 저장된다.(둘다 스택 메모리에 할당)

다음 줄에서 참조 타입인 변수 o에 값 타입인 a를 대입시킨다.

이때, 박싱 변환이 일어나는데 아래와 같은 과정을 거치게 된다.

1. 힙 메모리에 객체 인스턴스 생성.
2. 값 타입 데이터 복사.
3. 로컬 변수 o에 해당 인스턴스의 참조(주소) 저장.

![Boxing](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/types/media/boxing-and-unboxing/boxing-operation-i-o-variables.gif)

코드로 보면 단순히 변수끼리 대입을 한 것이지만 런타임에서 동작 시 상당히 많은 과정을 거치게 된다.

메모리에서 해제도 DoSomething() 함수가 종료 될때 내부의 지역 변수들은 스택에서 해제 되지만,

변수 o가 참조되는 데이터는 힙영역에 있으므로 가비지 컬렉션 시점에 참조 그래프를 순회하여 더 이상 도달 할 수 없는 경우 객체를 해제한다.

>[.Net GC는 세대별(Gen0/1/2) tarcing 방식으로 Root에서 도달 불가 객체를 수집해 해제한다.]({% post_url /Devlog/C#/2022-07-11-garbage-collection %})

### Unboxing

아래의 코드를 보자

```cs
private void DoSomething()
{
    int a = 123;
    object o = a;        // Boxing 변환 발생

    var o2 = o;          // o가 참조하는 a의 데이터 123의 주소가 복사되어 o2에 저장
    int b = (int)o;      // Unboxing 변환 발생
}
```

동일하게 변수 a에 데이터 123을 저장하고, 변수 o에 a를 대입 시키면서 박싱 변환이 일어 났다.

다시 변수 o2에 o를 대입 시키는데, 이 이때는 아무 변환도 일어나지 않는다.

컴파일러가 데이터 타입을 변수 o로 부터 추론하여 object 타입으로 o2의 타입을 결정짓는다.

o, o2 둘다 참조 타입이므로 o가 참조하는 힙역역의 주소가 복사되어 o2에 저장된다.

이제 다시 변수 o를 변수 b에 대입시키는데 이때 언박싱 변환이 일어나고, 박싱 변환과 마찬가지로 아래와 같은 과정을 거친다.

 1. 변수 o가 참조하는 힙 영역의 데이터를 복사.
 2. 변수 b에 복사된 데이터를 저장.

![Unboxing](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/types/media/boxing-and-unboxing/unboxing-conversion-operation.gif)

### 자주 발생되는 상황

## 출처 및 같이 보기

- [Boxing Unboxing](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/types/boxing-and-unboxing)
