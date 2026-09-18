---
title: "셰이더 배리언트 자동 수집"
date: 2025-10-02
last_modified_at: 2026-09-18

toc: true
toc_sticky: true

categories:
    - unity
tags: [unity, shader, shader variants]
---

## 개요

여러 셰이더를 사용하다 보면 셰이더 키워드의 조합이 늘어나고, 그에 따라 컴파일할 배리언트도 많아질 수 있다. 이는 빌드 시간과 셰이더 데이터의 크기에 영향을 준다.

Unity 에디터에는 사용된 셰이더 배리언트를 추적하고 `ShaderVariantCollection` 에셋으로 저장하는 기능이 있다. 이 글에서는 Unity 6.0 문서를 기준으로 수집 절차와 활용 방법을 정리한다.

**배리언트 수집·저장은 사용 목록을 만드는 단계다.** 컬렉션을 저장하는 것만으로 빌드에서 미사용 배리언트가 자동으로 제거되지는 않는다. 빌드 시간을 줄이려면 별도로 스트리핑 설정과 빌드 결과를 확인해야 한다.

## 셰이더 배리언트와 컬렉션

셰이더 배리언트는 키워드 조합 등에 따라 만들어지는 셰이더 프로그램의 변형이다. 컬렉션은 셰이더, 패스 타입, 키워드 조합을 기록하며 필요한 배리언트를 관리하거나 미리 준비하는 데 사용할 수 있다. [Shader variant collections](https://docs.unity3d.com/6000.0/Documentation/Manual/shader-variant-collections.html)

| 작업 | 목적 |
|------|------|
| 수집 | 에디터에서 실제 사용된 배리언트 목록을 기록 |
| 예열(Warm-up) | 첫 사용 시 발생할 수 있는 셰이더 준비 지연을 줄임 |
| 스트리핑(Stripping) | 빌드에 불필요한 배리언트를 제거 |

## 에디터에서 자동 수집하기

1. 수집할 씬과 콘텐츠를 준비하고, 빌드 대상·렌더 파이프라인·품질 설정을 기록한다.
2. **Edit > Project Settings > Graphics**에서 **Shader Loading**의 추적 항목을 찾는다. Unity 버전에 따라 항목의 배치나 버튼 이름은 다를 수 있다.
3. **Clear**로 기존 추적 기록을 초기화한다. 초기화하지 않으면 이전에 에디터에서 사용한 배리언트가 섞일 수 있다.
4. 플레이 모드에서 필요한 씬과 콘텐츠를 실행한다. 화면에 등장하는 머티리얼뿐 아니라 그림자, 이펙트, 키워드를 바꾸는 연출 등도 확인한다.
5. **Currently tracked:**에서 추적된 셰이더와 배리언트의 수를 확인한다.
6. **Save to asset…** 또는 버전에 따라 **Create asset** 버튼으로 `ShaderVariantCollection` 에셋을 저장한다.
7. 저장된 에셋을 선택하고 Inspector에서 셰이더, 패스 타입, 키워드 조합을 검토한다.

메뉴와 저장 방법은 [배리언트 수 확인 문서](https://docs.unity3d.com/6000.0/Documentation/Manual/shader-how-many-variants.html), 초기화 항목은 [Graphics 설정 문서](https://docs.unity3d.com/kr/2023.2/Manual/class-GraphicsSettings.html)를 참고한다.

추적 결과는 **에디터가 사용한 목록**이다. Game 뷰뿐 아니라 Scene 뷰에서 사용된 배리언트도 포함될 수 있으므로, 해당 플레이의 런타임 사용 목록과 정확히 일치한다고 가정하지 않는다.

## 수집 범위의 한계

한 번 플레이하며 얻은 컬렉션이 프로젝트 전체에 필요한 배리언트를 빠짐없이 담고 있다고 보장할 수는 없다. 예를 들어 다음과 같은 사용 경로가 수집에서 빠질 수 있다.

- 아직 방문하지 않은 씬이나 나중에 로드하는 Addressables·AssetBundle의 콘텐츠.
- 낮은 확률로 발생하는 이펙트, 특정 상태에서만 켜는 키워드.
- 품질 설정이나 렌더 파이프라인 설정을 바꿨을 때의 렌더링 경로.
- 에디터와 다른 플랫폼·그래픽스 API에서 사용하는 경로.

따라서 이 목록만을 근거로 나머지 배리언트를 모두 제거하는 방식은 피한다. 수집 당시의 조건을 함께 기록하고, 실제 빌드 대상에서 렌더링을 검증해야 한다.

## 저장한 컬렉션 활용하기

### 1. 예열에 사용

DirectX 11, OpenGL, OpenGL ES, WebGL에서는 Graphics 설정의 **Preloaded shaders**에 컬렉션을 추가하거나, 로드한 컬렉션에 `WarmUp()`을 호출하는 방법을 사용할 수 있다. 아래는 로딩 단계에서 직접 예열하는 간단한 예제다. `ShaderWarmupExample.cs`로 저장하고 Inspector에서 컬렉션을 지정한다.

```cs
using UnityEngine;

public class ShaderWarmupExample : MonoBehaviour
{
    [SerializeField] private ShaderVariantCollection variants;

    private void Awake()
    {
        if (variants != null)
        {
            variants.WarmUp();
        }
    }
}
```

컬렉션이 크면 동기 예열 자체가 긴 지연을 만들 수 있으므로 실행 시점을 정해야 한다. Unity는 DX12, Vulkan, Metal에서는 이 방법을 권장하지 않는다. 실제 그래픽스 상태 정보가 부족해 필요한 파이프라인 상태 객체(PSO)와 다른 상태를 준비할 수 있기 때문이다. 해당 API에서는 PSO 추적·예열 방식을 검토해야 하며, 컬렉션 예열만으로 모든 첫 렌더링 지연이 사라진다고 보장하지 않는다. [셰이더 예열 문서](https://docs.unity3d.com/6000.0/Documentation/Manual/shader-prewarm-other.html)

### 2. 빌드 시간 최적화에 활용

수집한 목록은 어떤 셰이더와 키워드를 사용하는지 파악하는 자료로 활용할 수 있다. 실제 컴파일 대상을 줄이는 작업은 별도로 진행한다.

1. **Always Included Shaders**에 불필요한 셰이더가 등록되어 있는지 확인한다.
2. Graphics 설정 및 사용하는 렌더 파이프라인의 스트리핑 옵션을 확인한다.
3. URP·HDRP의 사용하지 않는 기능을 검토하고, 런타임에 변경하는 기능이 제거되지 않도록 범위를 정한다.
4. 설정만으로 처리하기 어려운 경우 `IPreprocessShaders.OnProcessShader`를 이용한 사용자 정의 스트리핑을 검토한다.

배리언트의 빌드 포함 여부와 예열 여부는 별개다. 컬렉션을 에셋으로 저장했다는 이유만으로 모든 빌드 경로에서 해당 배리언트가 보존되거나 예열된다고 가정해서는 안 된다. [셰이더 스트리핑 문서](https://docs.unity3d.com/6000.0/Documentation/Manual/shader-variant-stripping.html)

## 결과 확인

빌드 후 `Editor.log`에서 `Compiling shader`와 스트리핑 전후의 배리언트 수, 컴파일 시간을 확인한다. 변경 전후를 비교할 때는 빌드 대상과 품질 설정을 맞추고, 셰이더 캐시의 영향도 고려한다.

실제 기기에서는 주요 씬과 이펙트, 품질 전환을 다시 실행한다. 필요하면 **Strict shader variant matching**을 활성화한 검증용 빌드로 누락된 배리언트를 확인한다. 이 옵션은 누락 시 유사한 배리언트를 대신 사용하는 상황을 발견하는 데 도움이 된다. [배리언트 확인 및 누락 진단](https://docs.unity3d.com/6000.0/Documentation/Manual/shader-how-many-variants.html)

## 출처 및 같이 보기

- [Shader variant collections](https://docs.unity3d.com/6000.0/Documentation/Manual/shader-variant-collections.html)
- [Check how many shader variants you have](https://docs.unity3d.com/6000.0/Documentation/Manual/shader-how-many-variants.html)
- [Graphics settings](https://docs.unity3d.com/kr/2023.2/Manual/class-GraphicsSettings.html)
- [Strip shader variants](https://docs.unity3d.com/6000.0/Documentation/Manual/shader-variant-stripping.html)
- [Other methods to warm up shaders](https://docs.unity3d.com/6000.0/Documentation/Manual/shader-prewarm-other.html)
