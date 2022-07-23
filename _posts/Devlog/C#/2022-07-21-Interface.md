---
title: "interface를 왜 사용하나요?"
date: 2022-07-21
last_modified_at: 2022-07-21

toc: true
toc_sticky: true

categories:
    - csharp
tag:
    - [csharp, c#, interface]
---

## 개요
면접 때 인터페이스가 뭐고 왜 사용하는지를 물어봤는데 명확하게 대답하지 못해서 다시 정리 해보려고 한다.

마소 c# 문서에서는 인터페이스를 **"여러 형식에 대한 동작 정의"**라고 제목에 넣어 두고 있다.

"여러 형식"이란 우리가 정의하는 구조체와 클래스를 말하는 것이고, 다시 적으면 "객체에 대한 동작의 정의"라는 뜻이 된다.

하지만 이미 공통된 동작에 대한 처리는 상속으로 구현하면 되는데 왜 인터페이스라는 기능으로 만든것일까?

이미 다들 아시다 시피 c#은 하나의 클래스만을 상속하는것을 허용하고, 클래스의 다중상속은 언어 차원에서 허용하지 않는다.
  
다중상속에서 가장 큰 문제는 **"모호성"** 인데, 
   
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
 1. D 의 생성자는 어떻게 호출될까?
  ``` 
  A
  B
  A
  C
  D
  ```
  - D 는 A를 상속받은 B와 C를 상속받으므로 생성자 호출 시 각 B와 C의 부모클래스인 A를 호출해야된다.
 2. D 의 GetData()를 호출하면 어떻게 될까?
  - 컴파일러는 B 와 C 중 어느 쪽 m_data를 호출해야 되는지 알지 못해 불만을 터트리며 컴파일 에러를 뱉어 낼 것이다.
 3. D 의 CallMethod()를 호출하면 어떻게 될까?
  - 2번 상황과 마찬가지로 어느쪽 메서드를 호출해야된지 알지 못하므로 컴파일 에러가 발생된다.

사용자가 의도치 않게 혹은 
   


## 암묵적 구현
## 명시적 구현
## 출처 및 같이 보기
- <a href="https://docs.microsoft.com/ko-kr/dotnet/csharp/fundamentals/types/interfaces">c# 인터페이스</a>
- <a herf="https://ko.wikipedia.org/wiki/%EB%8B%A4%EC%A4%91_%EC%83%81%EC%86%8D">위키 다중상속</a>
- <a href="https://www.csharpstudy.com/DevNote/Article/4">인터페이스의 암묵적/명시적 정의</a>

