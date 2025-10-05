---
title: "Monobehaviour"
date: 2022-07-21
last_modified_at: 2025-05-08

toc: true
toc_sticky: true

categories:
    - unity
tags:
    - [unity, monobehaviour lifcycle]
---

## 개요

Unity 엔진에서 스크립트 작성에 가장 기본이 되는 베이스 클래스.

Unity 엔진의 내부 코드는 c++로 구현되어 있으며, c# 개발자가 c++ 기반의 오브젝트를 제어하기 위한 스크립트 인터페이스라고 볼 수 있다.

Unity에서 작성하는 스크립트들은 c# 코드로 구성되지만, 실제 동작하는 대상은 엔진 내부의 네이티브 오브젝트이다.

작성된 `Monobehaviour` 스크립트는 단일로는 동작하지 않으며, `GameObject`에 컴포넌트로 붙여 동작시킨다.

## 유니티 엔진 오브젝트의 상속 구조

```markdown
UnityEngine.Object
    |----- Component
            |----- Behaviour
                    |----- MonoBehaviour
```

| 클래스 | 설명 |
|--------|-----------|
| UnityEngine.Object | 모든 Unity 오브젝트의 기반 클래스, 엔진 내부의 C++ 객체를 감싸는 래퍼(Wrapper) |
| Component | GameObject에 부착 가능한 엔진 구성요소, Transform, Renderer, Collider등이 이해 해당 |
| Behaviour | Component중에서도 '활성화/비활성화' 속성을 가지는 클래스 |
| MonoBehaviour | Behaviour를 c# 스크립트로 확장한 클래스, 개발자가 직접 코드를 작성할 수 있는 컴포넌트 타입 |

`MonoBehaviour`는 Unity 엔진의 C++ 네이티브 객체를 제어하는 c# 인터페이스 계층으로 볼 수 있다.

## UnityEngine.Object의 특별한 동작

1. `==` 연산자 오버라이드

    ```cs
    if(null == myBehaviour)
    {
        Debug.Log("이미 Destory() 된 오브젝트 입니다.");
    }
    ```

    이 비교는 "c# 참조가 null인지?"를 체크하는 것이 아니다.
    Unity는  `==`연산자를 오버라이드 하여 *c++ 객체가 파괴된 상태*인지를 판별한다.
    즉, 이미 `Destroy()`가 호출 된 오브젝트라도 *c# 인스턴스는 남아 있을 수 있지만, 엔진 내부적으로는 null처럼 동작* 한다.(Fake null 상태)

2. `Destroy()`와 메모리 해제 시점

    - `Desotry(obj)`를 호출하면, C++ 엔진 레벨의 오브젝트는 즉시 파괴되지만, *c# 인스턴스는 다음 프레임의 GC 시점까지 유지*된다.
    - 따라서 `null` 체크는 *C++ 객체의 존재 여부*를 확인 하는 기능으로 동작.

3. new 연산자의 금지

    ```cs
    var script = new MyScript(); // 금지
    ```

    `MonoBehaviour`는 `new` 키워드로 생성하는 것을 금지한다.
    반드시 `AddComponent<T>()`를 통해 GameObject에 추가되어야 된다.
    이는 Unity 엔진이 내부적으로 C++ 객체를 생성하고 그에 대응하는 c# 객체를 연결해야만 올바른 참조가 생기기 때문이다.

## Monobehaviour Property

`Monobehaviour`는 부착된 `GameObject`에 대한 여러 편의 기능을 프로퍼티로 제공한다.

- gameObject : 현재 스크립트가 부착된 게임 오브젝트.
- transform : 현재 스크립트가 부착된 게임 오브젝트의 Transform 컴포넌트.
- name : 현재 스크립트가 부착된 게임 오브젝트의 이름.
- tag : 현재 스크립트가 부착된 게임 오브젝트의 Tag.
- enabled : 컴포넌트를 활성화(OnEnable()) / 비활성화 (OnDisable()), 비활성화 되는 경우 해당 컴포넌트의 업데이트(Update()) 호출이 중지 된다.

## Monobehaviour Lifecycle

`Monobehaviour`는 유니티 엔진에서 정의된 메세지 함수가 호출하며 이 메세지 함수는 일정한 호출 시점을 가지므로 생명주기(lifecycle)로 불린다.

| 함수명 | 호출 시점 | 용도 |
|--------|-----------|------|
| Awake() | 인스펙터 로드 직후, 오브젝트가 생성된 시점(할당 후 단 한 번 호출) | 컴포넌트 캐싱, 의존성 초기화, 설정 정보 읽기 |
| OnEnable() | 컴포넌트가 활성화 될때(enabled = true) | 실질적인 컴포넌트 진입점, 활성화 될때 처리되는 작업(이벤트/리스너 등록등) |
| Start() | 첫 번째 Update()가 호출되는 시점(할당 후 단 한 번 호출) | 컴포넌트의 초기화, 다른 컴포넌트의 값 접근등 |
| Update() | 매프레임 마다 | 주요 로직(이동/입력/애니메이션등) |
| FixedUpdate() | 물리 프레임 마다(Project Setting에 설정된 시간(초)) | 유니티 물리 계산(RigidBody등) |
| LateUpdate() | 모든 Update() 이후 | 후처리 로직 및 카메라 처리등 |
| OnDisable() | 컴포넌트가 비활성화 될때 (enabled = false) | 비활성화 될때 처리되는 작업(이벤트/리스너 해제등) |
| OnDestroy() | 오브젝트 파괴 될 때 | 컴포넌트가 참조하던 리소스 해제 및 정리 작업 |

> 간단한 흐름으로 보면 아래와 같다.
> Awake() → OnEnable() → Start() → Update() → OnDisable() → OnDestroy()

## 출처 및 같이 보기

- [Monobehaviour](https://docs.unity3d.com/6000.1/Documentation/ScriptReference/MonoBehaviour.html)
- [Unity Order of execution for event functions](https://docs.unity3d.com/6000.1/Documentation/Manual/execution-order.html)
