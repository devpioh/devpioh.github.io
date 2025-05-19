---
title: "Boxing 과 Unboxing"
date: 2022-07-11
last_modified_at: 2022-07-11

toc: true
toc_sticky: true

categories:
    - csharp
tag:
    - [csharp, c#]
---

## 개요

CTS(공통 형식 시스템)에서는 값 타입과 참조 타입 2가지의 타입이 있다.
  
두 타입은 메모리에 할당 되는 방식에 차이가 있는데, 값 타입의 경우 스택에 할당되는 반면 참조 타입의 경우 힙 메모리에 할당된다.
  
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

 1. 힙 메모리에 변수 a의 타입의 크기만큼 메모리가 할당.
 2. 변수 a의 값을 복사해 힙 메모리에 저장.
 3. 스택에 변수 o가 할당.
 4. 힙에 할당된 a복사 데이터 주소를 변수 o에 저장.

![Boxing](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/types/media/boxing-and-unboxing/boxing-operation-i-o-variables.gif)

코드로 보면 단순히 변수끼리 대입을 한 것이지만 런타임에서 동작 시 상당히 많은 과정을 거치게 된다.

메모리에서 해제도 DoSomething() 함수가 종료 될때 내부의 지역 변수들은 스택에서 해제 되지만,

변수 o가 참조되는 데이터는 힙영역에 있으므로 단순히 해당 ~~데이터의 참조 카운터가 하나 내려가는 것 밖에 되지 않는다.~~

(이 부분은 잘못 되었다. c#은 참조 타입에 대한 메모리를 레퍼런스 카운터 방식이 아닌 참조 그래프를 만들어서 해당 객체가 참조되는지 아닌지를 파악한다.)

이는 추후 GC가 해당 데이터의 ~~참조 카운터~~ 참조 그래프를 보고 더이상 사용이 되지 않는다라고 판단되면 메모리에서 해제 한다.

이 부분은 다음 포스팅인 **[Garbage Collection]({% post_url /Devlog/C#/2022-07-11-Garbage-Collection %})**에서 더 알아보기로 한다.

### Unboxin

아래의 코드를 보자

```cs
private void DoSomthing()
{
    int a = 123;
    object o = a;       // Boxing 변환 발생

    var o2 = o;         // o가 참조하는 a의 데이터 123의 주소가 복사되어 o2에 저장
    int b = (int)o      // Unboxing 변환 발생
}
```

동일하게 변수 a에 데이터 123을 저장하고, 변수 o에 a를 대입 시키면서 박싱 변환이 일어 났다.

다시 변수 o2에 o를 대입 시키는데, 이 이때는 아무 변환도 일어나지 않는다.

컴파일러가 데이터 타입을 변수 o로 부터 추론하여 object 타입으로 o2의 타입을 결정짓는다.

o, o2 둘다 참조 타입이므로 o가 참조하는 힙역역의 주소가 복사되어 o2에 저장된다.

이제 다시 변수 o를 변수 b에 대입시키는데 이때 언박싱 변환이 일어나고, 박싱 변환과 마찬가지로 아래와 같은 과정을 거친다.

 1. 변수 o가 참조하는 힙 영역의 데이터의 크기만큼 스택에 메모리가 할당
 2. 힙 영역에 있는 데이터를 복사.
 3. 변수 b에 복사된 데이터를 저장.

![Unboxing](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/types/media/boxing-and-unboxing/unboxing-conversion-operation.gif)

Boxing과 Unboxing는 공통적으로 많은 비용을 소모하는 작업이다.

반복문 / 업데이트 함수에서 박싱과 언박싱이 많이 일어나게 될 경우 많은 양의 가비지를 발생시키고, 이는 성능저하로 이어진다.

보통은 타입을 잘 맞춰서 코딩을 하지만 사용자가 모르게 발생되는 경우가 많다.

특히 string.Format()의 경우 문자열에 데이터값을 표기 할때 많이 사용되는데 파라미터 값을 보면 object[] 로 데이터 값을 받는데,

이 때 그냥 그대로 데이터를 집어 넣게 되는 경우 박싱 변환이 일어나게 되므로 주의해야된다.

## 출처 및 같이 보기

- [Boxsing Unboxing](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/types/boxing-and-unboxing)