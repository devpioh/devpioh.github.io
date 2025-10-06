---
title: "coroutine"
date: 2022-07-22
last_modified_at: 2025-05-08

toc: true
toc_sticky: true

categories:
    - unity
tags:
    - [unity, coroutine]
---

## 개요

유니티에서 비동기 작업을 위한 함수.

프레임단위의 조작이나 일정 시간의 동작을 분산시키거나, 비동기적 흐름 제어에 유용하다.

`yield return` 키워드를 사용하여, 정지가 필요한 시점에서 대기 객체(null, WaitForSeconds, WaitForEndOfFrame등)를 반환한다.

이 후 대기 조건이 끝나면 이어서 다음 구문을 실행한다.

코루틴은 마치 비동기적 병렬 실행되는 것처럼 보이지만, 실제로는 Unity 메인 스레드의 프레임 루프 내에서 순차적으로 실행된다.

즉, *멀티 스레드가 아닌 단일 스레드 기반의 협력형(concurrent) 실행 모델* 이다.

## 동작 구조

Unity 엔진에서 코루틴 동작 흐름

1. StartCoroutine() -> 코루틴 등록
2. IEnumerator.MoveNext() 호출
3. yield return (대기 객체 반환)
4. Unity가 반환된 객체 타입에 따라 대기 상태 결정
5. 조건이 충족되면 다시 IEnumerator.MoveNext() 호출
6. Coroutine 종료 시 코루틴 제거.

Unity의 코루틴은 C#의 `IEnumerator`를 기반으로 한 *상태 머신(State Machine) 형태의 객체*.

`StartCoroutine()`을 호출하면 Unity 엔진이 `IEnumerator`를 내부 리스트(코루틴 매니저)에 등록하고, 매프레임 마다 `MoveNext()`를 호출하여 다음 `yield` 지점까지 실행한다.

## 기본적인 사용

### 1. 메서드 시그니처

반환형은 반드시 IEnumerator 이어야 된다.

```cs
private IEnumerator WaitJob()
{
    // 실행 코드

    yield return null;  // 다음 프레임 이 후 아래 코드가 실행된다.
    
    // 이어서 실행될 코드
}

```

### 2. 실행

StartCoroutine() 메서드로 실행.

```cs
StartCoroutine(WaitJob());
```

### 3. 중지

* StopCoroutine("WaitJob");
* StopAllCoroutines();

```cs

// 실행 시 StartCoroutine() 반환을 저장.
var stop =  StartCoroutine(WaitJob()); 
/*
 * var stop = WaitJob();
 * StartCoroutine(stop); 
*/

// 코루틴 작업을 취소
StopCoroutine(stop);
```

## yield return

yield return 의 대기 반환은 여러 종류가 있지만 자주 사용되는 몇가지를 정리해 본다.

| 표현식 | 설명 |
|-------|-----|
| yield return null | 다음 프레임까지 대기 |
| yield return new WaitForSeconds(t); | t 초 동안 대기 |
| yield return new WaitForEndOfFrame(); | 모든 렌더링이 끝난 End of Frame까지 대기 |
| yield return new WaitUntil(() => isTrue); | 함수 또는 람다식의 return 값이 true로 평가 될 때까지 대기 |
| yield return StartCoroutine(OtherCoroutine()); | OtherCoroutine() 이 끝날 때 까지 대기 |

## 출처 및 같이 보기

* [StartCoroutine](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/MonoBehaviour.StartCoroutine.html)
* [StopCoroutine](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/MonoBehaviour.StopCoroutine.html)