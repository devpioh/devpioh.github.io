---
title: "Monobehaviour"
date: 2022-07-21
last_modified_at: 2025-05-08

toc: true
toc_sticky: true

categories:
    - unity
tag:
    - [unity, monobehaviour lifcycle]
---

## 개요

Unity 엔진에서 스크립트 작성에 가장 기본이 되는 베이스 클래스

작성된 Monobehaviour 스크립트는 단일로는 동작하지 않으며, Gameobject에 컴포넌트로 붙여 동작 시킨다.

## Monobehaviour Property

Monobehaviour는 GameObject에 컴포넌트로 붙여야지 동작하므로 부착된 GameObject에 대한 몇가지 편의 기능을 Property로 지원한다.

* gameObject : 컴포넌트가 부착된 게임 오브젝트. var go = GetComponent<GameObject>(); 접근과 동일.
* transform : 컴포넌트가 부착된 게임 오브젝트의 Transform 컴포넌트. var trans = GetComponent<Transform>(); 접근과 동일.
* name : 컴포넌트가 부착된 게임 오브젝트의 이름.
* tag : 컴포넌트가 부착된 게임 오브젝트의 Tag.
* enabled : 컴포넌트를 활성화(OnEnable()) / 비활성화 (OnDisable()), 비활성화 되는 경우 해당 컴포넌트의 업데이트(Updaate()) 호출이 중지 된다.

## Monobehaviour Lifecycle

Monobehaviour는 유니티 엔진에서 정의된 메세지 함수가 호출하며 이 메세지 함수는 일정한 호출 시점을 가지므로 생명주기(lifecycle)로 불린다.

| 함수명 | 호출 시점 | 용도 |
|--------|-----------|------|
| Awake() | 인스펙터 로드 직후, 오브젝트가 생성된 시점(할당 후 단 한 번 호출) | 컴포넌트 캐싱, 의존성 초기화, 설정 정보 읽기 |
| OnEnable() | 컴포넌트가 활성화 될때(enabled = true) | 실질적인 컴포넌트 진입점, 활성화 될때 처리되는 작업(이벤트/리스너 등록등) |
| Start() | 첫 번째 Update()가 호출되는 시점(할당 후 단 한 번 호출) | 컴포넌트의 초기화, 다른 컴포넌트의 값 접근등 |
| Update() | 매프레임 마다 | 주요 로직(이동/입력/애니메이션등) |
| FixedUpdate() | 물리 프레임 마다(Project Setting에 설정된 시간(초)) | 유니티 물리 계산(RigidBody등) |
| LateUpdate() | 모든 Update() 후 | 후처리 로직 및 카메라 처리등 |
| OnDisable() | 컴포넌트가 비활성화 될때 (enabled = false) | 비활성화 될때 처리되는 작업(이벤트/리스너 해제등) |
| OnDestory() | 오브젝트 파괴 시점 | 컴포넌트가 참조하던 리소스 해제 및 정리 작업 |

## 출처 및 같이 보기

[Monobehaviour](https://docs.unity3d.com/6000.1/Documentation/ScriptReference/MonoBehaviour.html)
[Unity Order of execution for event functions](https://docs.unity3d.com/6000.1/Documentation/Manual/execution-order.html)
