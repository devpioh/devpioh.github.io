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

"yield return" 키워드를 이용해 정지가 필요한 지점에서 대기 정보(null, WaitForSeconds, WaitForEndOfFrame등)을 리턴 시키고,
 대기 처리가 끝나면 이어서 다음 구문을 실행한다.

 코루틴은 비동기 흐름 제어로 인해 별도 스레드에서 실행되는 것 처럼 보이지만, 메인 스레드에서 동작되는 시분할(time-sharing) 시스템의 일종이다.

 메인스레드에서 동작되므로 유니티 API에 안전하다.

## 출처 및 같이 보기
[Monobehaviour](https://docs.unity3d.com/6000.1/Documentation/ScriptReference/MonoBehaviour.html)      
[Unity Order of execution for event functions](https://docs.unity3d.com/6000.1/Documentation/Manual/execution-order.html)