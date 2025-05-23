---
title: "Value Type과 Referance Type"
date: 2022-07-10
last_modified_at: 2022-07-10

toc: true
toc_sticky: true

categories:
    - csharp
tags:
    - [csharp, c#]
---

## 개요

.Net 프레임워크에서 동작하는 언어(닷넷 호완 언어)는 한가지 지켜야 되는 규칙이 있다.

**CTS(Common Type Systme) - 공통 형식 시스템**이라고 불리는 것으로 이는 언어가 지켜야되는 표준 타입 규격이다.

CTS의 모든 타입들은 최상위 타입인 System.Object를 상속받아 구현되어져 있으며 다시 ValueType과 ReferanceType으로 나눠진다.

![ValueType_ReferanceType](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/types/media/index/value-reference-types-common-type-system.png)

### Value Type

- System.Object에서 파생된 System.ValueType을 상속받아 구현된 타입들.
- System.Int32, System.Boolean, System.Byte 등 시스템 기본 데이터 타입들.
- 모든 구조체 및 열거형 타입 (사용자 정의 포함)도 여기에 해당된다.
- 같은 값타입의 변수에 대입 시 기본적으로 값을 복사 한다.
- 스택 메모리에 할당되므로 함수가 종료될 시 메모리에서 해제된다.

### Referance Type

- System.Object를 직접 상속받아 구현된 타입들.
- System.String, System.Array 등과 모든 인터페이스와 사용자가 정의한 클래스들.
- 메모리에 할당시 관리되는 힙에 할당된다.
- 힙에 할당된 객체는 GC에 의해 관리되고 해제된다.
- 같은 참조 타입 변수에 대입 시 메모리 주소를 복사 한다.

## 출처 및 같이 보기

- [ValueType ReferanceType](https://docs.microsoft.com/ko-kr/dotnet/csharp/fundamentals/types/)