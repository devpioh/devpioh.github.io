---
title: "Coroutine"
date: 2022-07-22
last_modified_at: 2026-09-18

toc: true
toc_sticky: true

categories:
    - unity
tags: [unity, coroutine]
---

## 개요

코루틴은 실행을 중단했다가 특정 조건이 충족되면 이어서 실행하는 Unity의 흐름 제어 기능이다. 프레임에 걸친 연출, 일정 시간 대기, 비동기 로딩의 완료 대기 등에 사용한다.

`yield return`으로 `null`, `WaitForSeconds` 등의 값을 반환하면 Unity가 그 값에 맞춰 다음 실행 시점을 결정한다.

코루틴의 C# 코드는 일반적으로 Unity 메인 스레드에서 실행된다. 여러 코루틴이 번갈아 진행될 수 있지만, 코루틴 자체가 작업을 별도 스레드로 옮기지는 않는다. 따라서 `yield` 이전에 오래 걸리는 계산이나 동기 입출력을 수행하면 해당 프레임도 그만큼 지연된다.

이 글은 Unity 6.0의 코루틴 API를 기준으로 정리한다.

## 동작 구조

`yield`를 사용하는 C# 반복자 메서드는 컴파일러가 상태 머신으로 변환한다. 이 상태 머신은 실행 위치와 필요한 지역 변수를 보관하며, `IEnumerator`를 통해 다음 실행 구간으로 진행한다.

1. 반복자 메서드를 호출하여 `IEnumerator`를 얻는다.
2. `StartCoroutine()`에 전달하면 첫 번째 `yield` 또는 종료 지점까지 실행한다.
3. `yield return`으로 반환된 값에 따라 Unity가 대기 조건을 결정한다.
4. 조건이 충족되면 다음 `yield` 또는 종료 지점까지 실행을 재개한다.
5. 메서드가 끝나거나 `yield break`에 도달하면 코루틴이 종료된다.

코루틴 본문의 `MoveNext()`를 대기 조건과 관계없이 매 프레임 호출하는 것은 아니다. 예를 들어 `WaitForSeconds`를 반환했다면 지정된 시간이 경과하기 전까지 그 다음 구문을 실행하지 않는다.

## 기본적인 사용

### 1. 메서드 시그니처

`StartCoroutine()`에 전달할 반복자 메서드는 일반적으로 `System.Collections.IEnumerator`를 반환하도록 작성한다. 아래 코드는 `MonoBehaviour`를 상속한 클래스 안에서 사용한다.

```cs
private IEnumerator WaitJob()
{
    Debug.Log("대기 전");
    yield return null; // 다음 프레임에 재개한다.
    Debug.Log("대기 후");
}
```

### 2. 실행

`WaitJob()`만 호출하면 반복자 객체를 얻을 뿐, Unity가 실행을 관리하지 않는다. `StartCoroutine()`에 전달하여 실행한다.

```cs
Coroutine handle = StartCoroutine(WaitJob());
```

### 3. 중지

시작할 때 반환받은 `Coroutine` 핸들을 저장해 두면 해당 실행을 중지할 수 있다.

```cs
// 앞서 저장한 핸들로 해당 코루틴을 중지한다.
if (handle != null)
{
    StopCoroutine(handle);
}
```

다른 방법을 사용할 때도 시작한 실행을 정확히 지정해야 한다.

| 시작 방법 | 중지 방법 |
| ----------- | ----------- |
| `var handle = StartCoroutine(WaitJob());` | `StopCoroutine(handle);` |
| `var routine = WaitJob(); StartCoroutine(routine);` | 같은 인스턴스로 `StopCoroutine(routine);` |
| `StartCoroutine(nameof(WaitJob));` | `StopCoroutine(nameof(WaitJob));` |

문자열 방식으로 시작하지 않은 코루틴을 문자열로 중지하지 않는다. `StopCoroutine(WaitJob())`처럼 새 반복자 객체를 생성해 넘기는 것도 기존 실행을 가리키지 않는다.

`StopAllCoroutines()`는 호출한 **MonoBehaviour 인스턴스**에서 실행 중인 코루틴만 중지한다. 씬 전체나 같은 GameObject의 다른 컴포넌트까지 중지하는 기능은 아니다. [StopCoroutine](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/MonoBehaviour.StopCoroutine.html), [StopAllCoroutines](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/MonoBehaviour.StopAllCoroutines.html)

## 자주 사용하는 yield return

| 표현식 | 설명 |
| ------- | ----- |
| `yield return null;` | 다음 프레임까지 대기 |
| `yield return new WaitForSeconds(t);` | `Time.timeScale`의 영향을 받는 시간으로 t초 대기 |
| `yield return new WaitForSecondsRealtime(t);` | `Time.timeScale`의 영향을 받지 않는 시간으로 t초 대기 |
| `yield return new WaitForFixedUpdate();` | 다음 물리 업데이트 단계를 기다림 |
| `yield return new WaitForEndOfFrame();` | 카메라와 GUI 렌더링이 끝난 뒤, 프레임을 화면에 표시하기 전까지 대기 |
| `yield return new WaitUntil(() => isReady);` | 조건이 true가 될 때까지 대기 |
| `yield return OtherCoroutine();` | 중첩된 반복자가 끝날 때까지 대기 |
| `yield return StartCoroutine(OtherCoroutine());` | 별도로 시작한 코루틴의 완료를 대기 |

`WaitForSeconds`는 현재 프레임이 끝난 시점부터 대기를 시작하고, 시간이 경과한 뒤 재개할 수 있는 첫 프레임에 이어서 실행한다. 정확히 t초가 되는 순간의 실행을 보장하지 않는다. `Time.timeScale`이 0이면 이 대기도 진행되지 않으므로, 일시 정지 중에도 동작해야 하는 UI 등에는 `WaitForSecondsRealtime`을 고려한다. [WaitForSeconds 공식 문서](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/WaitForSeconds.html)

## 활성화 상태와 코루틴의 수명

GameObject가 비활성화되거나 실행 주체인 MonoBehaviour가 파괴되면 그 코루틴도 중지된다. 반면 **컴포넌트의 `enabled`만 false로 바꾸면 코루틴은 계속 실행된다.** 비활성화 시 중지해야 하는 작업은 직접 정리해야 한다.

다음 예제는 활성화될 때 작업을 시작하고, 비활성화될 때 중지하는 경우다. `CoroutineExample.cs`로 저장해 GameObject에 추가한다.

```cs
using System.Collections;
using UnityEngine;

public class CoroutineExample : MonoBehaviour
{
    private Coroutine running;

    private void OnEnable()
    {
        running = StartCoroutine(WaitJob());
    }

    private IEnumerator WaitJob()
    {
        Debug.Log("작업 시작");
        yield return new WaitForSeconds(1f);
        Debug.Log("작업 완료");
        running = null;
    }

    private void OnDisable()
    {
        if (running != null)
        {
            StopCoroutine(running);
            running = null;
        }
    }
}
```

중지된 코루틴은 GameObject를 다시 활성화한다고 이어서 실행되지 않는다. 위 예제는 `OnEnable`에서 **새 실행**을 시작한다. 또한 중지 시에는 대기 뒤에 작성한 정리 코드까지 도달하지 않을 수 있으므로, 필요한 정리를 코루틴 마지막 구문에만 맡기지 않는다.

## 메모리 할당

반복자 메서드를 호출해 상태 머신을 만들거나 `new WaitForSeconds(...)` 등의 대기 객체를 생성하면 관리 힙 할당이 발생할 수 있다. `yield return null` 자체는 새 대기 객체를 만들지 않지만, 코루틴 전체가 무할당이라는 의미는 아니다.

프레임마다 코루틴을 새로 시작하는 코드나 자주 반복되는 대기에서는 할당량을 Profiler로 확인한다. 상태를 가지는 대기 객체를 여러 실행이 무조건 공유하도록 바꾸는 것도 피한다.

## 출처 및 같이 보기

* [StartCoroutine](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/MonoBehaviour.StartCoroutine.html)
* [StopCoroutine](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/MonoBehaviour.StopCoroutine.html)
* [StopAllCoroutines](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/MonoBehaviour.StopAllCoroutines.html)
* [Coroutines](https://docs.unity3d.com/6000.0/Documentation/Manual/Coroutines.html)
* [WaitForSeconds](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/WaitForSeconds.html)
* [WaitForSecondsRealtime](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/WaitForSecondsRealtime.html)
* [WaitForEndOfFrame](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/WaitForEndOfFrame.html)
