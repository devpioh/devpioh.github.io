---
title: "interface의 암묵적 / 명시적 구현"
date: 2022-07-25
last_modified_at: 2025-05-12

toc: true
toc_sticky: true

categories:
    - csharp
tags:
    - [csharp, c#, interface]
---

## 개요

마소 c# 문서에서는 인터페이스를 **"여러 형식에 대한 동작 정의"**라고 제목에 넣어 두고 있다.

"여러 형식"이란 우리가 정의하는 구조체와 클래스를 말하는 것이고, 다시 적으면 "객체에 대한 동작의 정의"라는 뜻이 된다.

하지만 이미 공통된 동작에 대한 처리는 상속으로 구현하면 되는데 왜 인터페이스라는 기능으로 만든것일까?

### 다중 상속의 문제

c#은 하나의 클래스만을 상속하는것을 허용하고, 클래스의 다중 상속은 언어 차원에서 허용하지 않는다.

다중상속의 가장 대표적인 문제가 아래의 도식처럼 구조가 형성되는 다이아몬드 문제이다.

![diamond problem](https://upload.wikimedia.org/wikipedia/commons/thumb/8/8e/Diamond_inheritance.svg/440px-Diamond_inheritance.svg.png)

최상위 A클래스를 상속하는 B와 C클래스가 있고, B와 C를 상속하는 D라는 클래스가 있다.
  
여기서 아래와 같이 구현 했을때, 생기는 문제점들을 한번 알아보자.

 ```cpp
  class A
  {
    public:
        A()                 { cout<<"A"<<endl; }
        void MethodA()      { cout<<"Call MethodA()"<<endl; }
  }

  class B : public A
  {
    public:
        int m_data;
        B() : m_data(1)     { cout<<"B"<<endl; }
        void MethodA()      { cout<<"Call B.MethodA()"<<endl; }
  }

  class C : public A
  {
    public:
        int m_data;
        C() : m_data(2)     { cout<<"C"<<endl; }
        void MethodA()      { cout<<"Call C.MethodA()"<<endl; }
  }

  class D : public B, public C
  {
    public:
        D() : B(), C()      { cout<<"D"<<endl; }
        int GetData()       { return m_data; }
        void CallMethod()   { MethodA(); }
  }
 ```

#### 1. D 의 생성자는 어떻게 호출될까?

  ```txt
  A
  B
  A
  C
  D
  ```

* D 는 A를 상속받은 B와 C를 상속받으므로 생성자 호출 시 각 B와 C의 부모클래스인 A를 호출해야된다.
  
#### 2. D 의 GetData()를 호출하면 어떻게 될까?

* 컴파일러는 B 와 C 중 어느 쪽 m_data를 호출해야 되는지 알지 못해 불만을 터트리며 컴파일 에러를 뱉어 낼 것이다.

#### 3. D 의 CallMethod()를 호출하면 어떻게 될까?

* 2번 상황과 마찬가지로 어느쪽 메서드를 호출해야된지 알지 못하므로 컴파일 에러가 발생된다.

상속으로 인해 코드의 재사용성과 확장이 늘어났지만, 중복 코드의 문제와 구조의 복잡성이 늘어나게 된 것이다.

그래서 c#은 상속은 단일 클래스로만 가능하게 했다.
  
하지만 단일 클래스의 상속만으로는 객체지향의 특징인 다형성을 표현하지 못하므로 인터페이스를 이용해 동작에 대한 확장을 적용하므로써 다형성을 구현한다.

c#은 공통된 속성과 대한 처리는 기본 클래스의 상속, 기능의 확장으로는 인터페이스의 상속으로 다형성을 표현한 것으로 나는 생각한다.
  
그렇지만 클래스의 다중 상속을 막았다고 해서 다중 상속 메서드명 중복의 문제는 여전히 남아있다.

### 암묵적 / 명시적 구현

c# 에서는 인터페이스의 구현을 2가지로 할 수 있다.

 1. public 접근지정자로 구현하는 **"암묵적 구현"**
 2. 접근지정자를 생략하고 *인터페이스명.메서드*로 구현하는 **"명시적 구현"**

```cs
public interface IAction
{
  void Action();
  void Reaction();
}

public class Button : IAction
{
  public void Action()
  {
    // 암묵적 구현
  }

  void IAction.Reaction()
  {
    // 명시적 구현
  }
}

public class App
{
  public static void Main()
  {
    var button = new Button();
    button.Action();        // 객체로 호출 가능
    //button.Reaction();    // 객체로 호출 불가!

    IAction act = button;
    act.Action();        // 인터페이스로 호출 가능
    act.Reaction();      // 인터페이스로 호출 가능  

    ((IAction)button).Reaction(); // IAction 으로 캐스팅 해야지만 호출 가능
  }
}
```

암묵적 구현은 일반적인 인터페이스의 구현 방법이다.

public 접근 지정자로 인터페이스의 메서드를 구현하는 방법으로 인터페이스를 상속한 객체의 public 멤버로 접근이 가능하다.

명시적 구현은 인터페이스를 상속한 객체의 멤버로는 접근을 할 수 없지만 해당 상속한 인터페이스로 캐스팅으로 접근이 가능한 형태이다.

바로 이 명시적 구현이 c#에서 메서드 명 중복을 해결한다.

혹시 `IEnumerable<T>`를 상속해 본적이 있다면, `GetEnumerator()`의 구현을 암묵적과 명시적 구현을 했을 것이다.
  
c#에 아직 제네릭이 도입되기전에는 `IEnumerable`의 `GetEnumerator()`이 있었고 이는 상당한 Boxing이 발생된다.

추후 제네릭을 도입하여 Boxing 문제를 해결한게 `IEnumerable<T>`의 `GetEnumerator()`지만 문제는 여기서 생긴다.

c#은 하위 버전의 호환성을 위해 `IEnumerable<T>`는 `IEnumerable`을 상속하는데, `IEnumerable`의 `GetEnumerable()`와 모호성이 생긴다.

클래스에서 `IEnumerable<T>`를 상속하는경우 어느 쪽의 `GetEnumerable()`을 쓸 것 인지를 CLR이 알지 못하게 되어 버린것이다.

c#은 이 메서드 중복의 문제를 상속 받는 클래스에서 *외부에 노출하고 싶은 인터페이스의 메서드*는 **암묵적 구현**으로 구현하고,

*외부에 노출시키면 위험하거나 혹은 숨기고 싶은 인터페이스의 메서드*는 **명시적 구현** 으로 해결했다.

`IEnumerable<T>`의 경우로 보자면 `IEnumerable`의 `GetEnumerable()`는 박싱의 위험이 있으니 외부의 노출을 막는것과 낮은 버전의 .Net에서도 동작이 가능해야되므로

`IEnumerable`의 `GetEnumerable()`는 명시적 구현을 통해 처리하고, 제네릭 버전의 `IEnumerable<T>`의 `GetEnumerable()`을 암묵적 구현으로 처리했다.

즉, 인터페이스의 명시적 구현을 통해서 다중상속의 메서드 중복문제와 하위 버전의 호환성 문제를 동시에 해결한 것이다.

암묵적/명시적 구현을 통해 다중상속의 문제를 해결했다고 하지만 과도한 인터페이스의 상속은 코드의 복잡도를 높히고 가독성을 떨어트리는 문제를 가진다.

최대한 단일 기능에 대한 처리(SRP) 원칙을 준수하도록 처리하여 가독성과 유지보수가 쉽게 구현하도록 노력해야된다.

## 출처 및 같이 보기

* [c# 인터페이스](https://docs.microsoft.com/ko-kr/dotnet/csharp/fundamentals/types/interfaces)
* [위키 다중상속](https://ko.wikipedia.org/wiki/%EB%8B%A4%EC%A4%91_%EC%83%81%EC%86%8D)
* [인터페이스의 암묵적/명시적 구현](https://www.csharpstudy.com/DevNote/Article/4)