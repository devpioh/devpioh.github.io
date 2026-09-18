---
title: "MonoBehaviour"
date: 2022-07-21
last_modified_at: 2026-09-18

toc: true
toc_sticky: true

categories:
    - unity
tags: [unity, monobehaviour, lifecycle]
---

## 개요

`MonoBehaviour`는 Unity에서 GameObject에 부착하는 C# 스크립트를 작성할 때 사용하는 기반 클래스다. Unity가 제공하는 생명주기 메시지를 통해 초기화, 프레임별 갱신, 정리 작업을 구현할 수 있다.

Unity의 엔진 객체는 C++ 네이티브 영역과 C# 관리 영역에 걸쳐 존재한다. `MonoBehaviour`도 엔진의 컴포넌트와 연결되어 동작하므로, 일반 C# 객체와 생성·파괴 방식이 다르다.

모든 C# 클래스가 `MonoBehaviour`를 상속해야 하는 것은 아니다. 데이터 처리나 계산만 담당하는 클래스는 일반 C# 클래스로 작성할 수 있다.

이 글의 생명주기 설명은 Unity 6.1 문서를 기준으로 하며, 일반적인 플레이 모드의 동작을 다룬다.

## 유니티 엔진 오브젝트의 상속 구조

```text
UnityEngine.Object
    |----- Component
            |----- Behaviour
                    |----- MonoBehaviour
```

| 클래스 | 설명 |
| -------- | ----------- |
| UnityEngine.Object | GameObject, Component, 에셋 등 Unity 엔진 객체의 기반 클래스 |
| Component | GameObject에 부착되는 구성요소. Transform, Renderer, Collider 등이 해당 |
| Behaviour | Component 중 `enabled` 속성을 가지는 클래스 |
| MonoBehaviour | 개발자가 C# 스크립트로 동작을 구현하는 컴포넌트의 기반 클래스 |

`UnityEngine.Object`는 모든 C# 객체의 기반 타입인 `System.Object`와 구분해야 한다.

## UnityEngine.Object의 특별한 동작

### 1. `==` 연산자 오버로딩과 null 검사

Unity는 `==` 연산자를 오버로딩하여 C# 참조가 null인지에 더해, 연결된 네이티브 객체가 파괴되었는지도 검사한다.

```cs
// MonoBehaviour를 상속한 클래스 안에서 사용하는 예제.
private void CheckTarget(MonoBehaviour target)
{
    if (target == null)
    {
        Debug.Log("참조가 없거나, 연결된 Unity 객체가 파괴되었습니다.");
    }
}
```

네이티브 객체가 파괴된 뒤에도 C# 객체는 메모리에 남아 있을 수 있다. 이때 `target == null`은 참이지만 `object.ReferenceEquals(target, null)`은 거짓일 수 있다. 흔히 이 상태를 *fake null*이라고 부른다.

`is null`, `?.`, `??`는 Unity의 `==` 연산자를 사용하지 않는다. 따라서 Unity 객체의 파괴 여부까지 확인해야 할 때는 이 표현들을 `target == null`의 대체로 사용하면 안 된다. 변수의 정적 타입이 `object`인 경우에도 Unity의 연산자 오버로딩이 적용되지 않으므로 주의한다.

관련 구현은 [UnityEngine.Object의 비교 연산자](https://github.com/Unity-Technologies/UnityCsReference/blob/master/Runtime/Export/Scripting/UnityEngineObject.bindings.cs)에서 확인할 수 있다.

### 2. `Destroy()`와 메모리 해제 시점

`Destroy(obj)`는 호출한 자리에서 객체를 즉시 파괴하지 않는다. 실제 파괴는 현재 Update 루프가 끝난 뒤, 렌더링 전에 처리된다. `Destroy(obj, t)`처럼 지연 시간을 지정할 수도 있다.

네이티브 객체의 파괴와 C# 객체의 메모리 회수는 별개다. C# 객체는 더 이상 도달 가능한 참조가 없을 때 GC의 수거 대상이 되며, 다음 프레임에 반드시 수거되는 것은 아니다.

- `Destroy(component)`는 해당 컴포넌트를 제거한다.
- `Destroy(gameObject)`는 GameObject와 부착된 컴포넌트, 자식 GameObject를 함께 파괴한다.
- `Destroy()`를 호출했다는 사실과 파괴 처리가 완료되었다는 사실을 구분해야 한다.

자세한 실행 시점은 [Object.Destroy 공식 문서](https://docs.unity3d.com/6000.1/Documentation/ScriptReference/Object.Destroy.html)를 참고한다.

### 3. 컴포넌트 생성

`MonoBehaviour`를 상속한 클래스를 `new`로 생성하면 정상적인 Unity 컴포넌트가 되지 않는다. 기존 GameObject에 추가하려면 `AddComponent<T>()`를 사용한다.

```cs
// MyBehaviour가 MonoBehaviour를 상속한 클래스라고 가정한다.
var script = gameObject.AddComponent<MyBehaviour>();
```

에디터의 Add Component로 추가하거나, 해당 컴포넌트가 포함된 GameObject·프리팹을 `Instantiate()`로 복제하는 방법도 있다. Unity가 네이티브 객체와 C# 객체의 연결을 관리하도록 생성해야 한다.

## 주요 프로퍼티

`MonoBehaviour`에서는 기반 클래스로부터 상속받은 프로퍼티를 통해 부착된 GameObject에 접근할 수 있다.

| 프로퍼티 | 의미 |
| ---------- | ------ |
| `gameObject` | 현재 컴포넌트가 부착된 GameObject |
| `transform` | 해당 GameObject의 Transform |
| `name` | 해당 GameObject의 이름 |
| `tag` | 해당 GameObject의 태그 |
| `enabled` | 컴포넌트 자체의 활성화 설정 |
| `isActiveAndEnabled` | GameObject가 계층상 활성 상태이고, 컴포넌트도 활성화되어 있는지 |

`enabled = true`라도 GameObject 또는 부모가 비활성 상태라면 `Update()`는 호출되지 않는다. 컴포넌트만 비활성화하는 것과 `gameObject.SetActive(false)`로 GameObject를 비활성화하는 것은 구분해야 한다.

## MonoBehaviour 생명주기

Unity는 정해진 조건과 시점에 `Awake`, `Update` 등의 메시지 함수를 호출한다. 주요 함수의 역할은 다음과 같다.

| 함수명 | 호출 시점 | 용도 |
| -------- | ----------- | ------ |
| Awake() | 활성 GameObject의 스크립트 인스턴스가 초기화될 때 한 번. 비활성 GameObject는 활성화될 때까지 지연 | 컴포넌트 캐싱, 자신의 초기 상태 설정 |
| OnEnable() | GameObject와 컴포넌트가 모두 활성 상태가 될 때 | 이벤트 구독, 활성화마다 필요한 작업 시작 |
| Start() | 활성 상태인 스크립트의 첫 Update 직전에 한 번 | 다른 컴포넌트의 초기화 결과를 사용하는 작업 |
| Update() | GameObject와 컴포넌트가 활성 상태일 때 매 프레임 | 입력 및 일반 게임 로직 |
| FixedUpdate() | 활성 상태에서 고정 시간 간격의 업데이트 단계마다 | Rigidbody 등 물리 관련 로직 |
| LateUpdate() | 활성 상태에서 해당 프레임의 Update 호출들이 끝난 뒤 | 카메라 추적, 후처리 |
| OnDisable() | 활성 상태를 잃거나, 활성 컴포넌트가 파괴되는 과정 등 | 이벤트 구독 해제, 작업 중지 |
| OnDestroy() | 컴포넌트 또는 GameObject가 파괴될 때. 이전에 활성 상태였던 GameObject에 대해 호출 | 소유한 리소스의 정리 |

처음부터 활성 상태인 인스턴스의 일반적인 흐름은 다음과 같다.

```text
Awake → OnEnable → Start → 프레임별 갱신 반복
                           ↓ 비활성화
                        OnDisable
                           ↓ 다시 활성화
                        OnEnable → 프레임별 갱신 반복
```

`Awake`와 `Start`는 같은 인스턴스를 다시 활성화해도 반복 호출되지 않는다. `OnEnable`과 `OnDisable`은 활성 상태 변화에 따라 여러 번 호출될 수 있다.

생명주기를 이용할 때는 다음 조건도 함께 고려한다.

- 활성 GameObject에서는 컴포넌트의 `enabled`가 false여도 `Awake`가 호출된다. `Start`는 컴포넌트가 활성화될 때까지 지연된다.
- 서로 다른 GameObject의 `Awake` 호출 순서는 기본적으로 보장되지 않는다. 다른 객체의 `Awake`가 먼저 실행되었다고 가정하지 않는다.
- 씬 로드 시 활성 객체들의 초기화 순서와, 플레이 중 `Instantiate()`로 생성하는 객체의 초기화 시점은 구분해야 한다.
- `FixedUpdate`는 렌더링 프레임마다 정확히 한 번 호출되는 것이 아니다. 프레임에 따라 호출되지 않거나 여러 번 호출될 수 있다.
- `enabled = false`만으로 이미 실행 중인 코루틴이 중지되지는 않는다. 비활성화에 맞춰 끝내야 하는 작업은 `OnDisable`에서 명시적으로 정리한다.

조건별 세부 동작은 [이벤트 함수 실행 순서](https://docs.unity3d.com/6000.1/Documentation/Manual/execution-order.html)와 각 함수의 API 문서를 참고한다.

## 출처 및 같이 보기

- [MonoBehaviour](https://docs.unity3d.com/6000.1/Documentation/ScriptReference/MonoBehaviour.html)
- [Awake](https://docs.unity3d.com/6000.1/Documentation/ScriptReference/MonoBehaviour.Awake.html)
- [OnEnable](https://docs.unity3d.com/6000.1/Documentation/ScriptReference/MonoBehaviour.OnEnable.html)
- [Start](https://docs.unity3d.com/6000.1/Documentation/ScriptReference/MonoBehaviour.Start.html)
- [OnDestroy](https://docs.unity3d.com/6000.1/Documentation/ScriptReference/MonoBehaviour.OnDestroy.html)
- [StopCoroutine](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/MonoBehaviour.StopCoroutine.html)
- [Unity Order of execution for event functions](https://docs.unity3d.com/6000.1/Documentation/Manual/execution-order.html)
