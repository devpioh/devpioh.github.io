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

인터페이스의 사용에 대한 내용은 웹에 많으므로 그것보단 "왜 이렇게 한걸까?"에 대한 내용을  
  
마소 c# 문서에서는 인터페이스를 **"여러 형식에 대한 동작 정의"**라고 제목에 넣어 두고 있다.

여러 형식이란 우리가 정의하는 구조체와 클래스를 말하는 것이고, 이것은 객체에 대한 동작의 정의라는 뜻도 된다.

하지만 이미 공통된 동작에 대한 처리는 추상 클래스의 상속으로 구현하면 되는데 왜 인터페이스라는 기능으로 만든것일까?

이미 다들 아시다 시피 c#은 하나의 클래스만을 상속하는것을 허용한다. 

c/c++과 같이 다중상속은 허용하지 않는데, 



여기서 나는 인터페이스가 사용자적인 측면 보다는 설계자의 입장에서 이 기능은 매우 유용한것 같다.


## 암묵적 구현
## 명시적 구현
## 출처 및 같이 보기
<a href="https://docs.microsoft.com/ko-kr/dotnet/csharp/fundamentals/types/interfaces">c# 인터페이스</a>
<a href="https://www.csharpstudy.com/DevNote/Article/4">인터페이스의 암묵적/명시적 정의</a>

