---
title: "Coroutine"
date: 2022-07-21
last_modified_at: 2025-05-08

toc: true
toc_sticky: true

categories:
    - unity
tag:
    - [unity, Coroutine]
---

## 개요
유니티에서 비동기 처리 작업 처리를 위한 함수.

프레임단위의 조작이나 일정 시간의 동작을 분산시키거나, 비동기적 흐름 제어에 유용하다.

"yield return" 키워드를 이용해 정지가 필요한 지점에서 대기 정보(null, WaitForSeconds, WaitForEndOfFrame등)을 리턴 시키고, 대기 처리가 끝나면 이어서 다음 구문을 실행한다.

코루틴은 비동기 흐름 제어로 인해 별도 스레드에서 실행되는 것 처럼 보이지만, 메인 스레드에서 동작되는 시분할(time-sharing) 시스템의 일종이다.

메인스레드에서 동작되므로 유니티 API에 안전하다.

## 기본적인 사용
#### 1. 메서드 시그니처
반환형은 반드시 IEnermerator 이어야 된다.

```cs
private IEnumerator WaitJob()
{
    // 실행 코드

    yield return null;  // 다음 프레임 이 후 아래 코드가 실행된다.
    
    // 이어서 실행될 코드
}

```

#### 2. 실행
StartCoroutine() 메서드로 실행.

```cs
StartCoroutine(WaitJob());
```

#### 3. 중지
* StopCoroutine("WaitJob");
* StopAllCoroutine();

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

## "yield return"
yield return 의 대기 반환은 여러 종류가 있지만 자주 사용되는 몇가지를 정리해 본다.

| 표현식 | 설명 |
|-------|-----|
| yield return null | 다음 프레임까지 대기 |
| yield return new WaitForSeconds(t); | t 초 동안 대기 |
| yield return new WaitForEndOfFrame(); | 모든 렌더링이 끝난 End of Frame까지 대기 |
| yield return new WaitUntil(() => isTrue); | 함수 또는 람다식의 return 값이 true로 평가 될 때까지 대기 |
| yield return StartCoroutine(OtherCoroutine()); | OtherCoroutine() 이 끝날 때 까지 대기 |



## 출처 및 같이 보기
[StartCoroutine](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/MonoBehaviour.StartCoroutine.html)      
[StopCoroutine](hhttps://docs.unity3d.com/6000.0/Documentation/ScriptReference/MonoBehaviour.StopCoroutine.html)