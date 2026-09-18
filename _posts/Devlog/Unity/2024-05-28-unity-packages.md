---
title: "Unity Packages"
date: 2024-05-28
last_modified_at: 2026-09-18

toc: true
toc_sticky: true

categories:
    - unity
tags: [unity, packages, UPM]
---

## 개요

여러 프로젝트에서 사용하는 유틸리티나 에디터 도구를 `Assets` 폴더에 복사하여 관리하면, 수정할 때마다 각 프로젝트에 변경 사항을 반영해야 한다. 공통 기능을 패키지로 분리하면 코드와 에셋을 재사용하고, 프로젝트에서 사용할 버전과 종속성을 명시할 수 있다.

Unity Package Manager(UPM)는 패키지를 설치하고 종속성을 처리하는 시스템이다. 공식 패키지뿐 아니라 직접 만든 커스텀 패키지도 로컬 폴더, Git 저장소, 패키지 레지스트리 등을 통해 사용할 수 있다. [Unity Package Manager](https://docs.unity3d.com/kr/2023.2/Manual/Packages.html)

이 글은 **Unity 2023.2 문서**를 기준으로 패키지의 구조, 로컬 패키지 설치, 커스텀 패키지 제작 방법을 정리한다. 에디터 버전에 따라 Package Manager의 메뉴 이름은 다를 수 있다.

## UPM 패키지와 에셋 패키지

Unity에서 사용하는 패키지는 형식에 따라 설치와 관리 방식이 다르다.

| 구분 | UPM 패키지 | 에셋 패키지 |
| ------ | ------------ | ------------- |
| 구성 | `package.json`을 포함하는 폴더 구조 | `.unitypackage` 파일 |
| 설치 | Package Manager에서 패키지를 참조 | 파일 안의 에셋을 프로젝트에 임포트 |
| 관리 | 매니페스트로 버전과 패키지 종속성을 선언 | 임포트한 에셋을 프로젝트에서 관리 |

아래에서 만드는 것은 UPM 패키지다. `Assets > Export Package`로 만드는 `.unitypackage`와는 별도의 형식이며, 해당 파일을 만드는 과정은 필요하지 않다. [에셋 패키지](https://docs.unity3d.com/kr/2023.2/Manual/AssetPackages.html)

## 프로젝트에서의 패키지 구조

먼저 프로젝트가 패키지를 참조하는 파일과, 패키지 자체를 설명하는 파일을 구분한다.

```text
MyProject/
├── Assets/
├── Packages/
│   ├── manifest.json
│   └── packages-lock.json
├── ProjectSettings/
└── Library/
    └── PackageCache/
```

| 경로 | 역할 |
| ------ | ------ |
| `Packages/manifest.json` | 프로젝트에서 직접 사용하는 패키지와 버전 또는 설치 경로를 선언 |
| `Packages/packages-lock.json` | UPM이 계산한 종속성 해결 결과를 기록 |
| `Library/PackageCache` | 다운로드한 패키지의 프로젝트 캐시 |
| 각 패키지의 `package.json` | 해당 패키지의 이름, 버전, 설명, 종속성 등을 선언 |

`manifest.json`은 **프로젝트가 무엇을 사용할지**, `package.json`은 **패키지가 무엇인지**를 설명한다. Package Manager에서 패키지를 설치하면 프로젝트 매니페스트에도 해당 참조가 반영된다. [프로젝트 매니페스트](https://docs.unity3d.com/kr/2023.2/Manual/upm-manifestPrj.html)

`packages-lock.json`은 Unity가 관리하므로 직접 편집하지 않는다. 팀에서 같은 종속성 해결 결과를 공유할 수 있도록 `manifest.json`과 함께 버전 관리에 포함한다. 다만 로컬 폴더의 파일 내용까지 특정 시점으로 고정해 주는 것은 아니다. [잠금 파일](https://docs.unity3d.com/kr/2023.2/Manual/upm-conflicts-auto.html)

Project 창의 `Packages`에 보이는 항목이 모두 디스크의 `MyProject/Packages` 안에 저장되는 것은 아니다. 로컬 패키지는 연결한 원본 폴더에 있고, 레지스트리에서 받은 패키지는 캐시에서 읽는다. 캐시의 코드를 직접 수정하는 대신 로컬 패키지나 아래의 Embedded 패키지로 작업한다. [내장 종속성](https://docs.unity3d.com/kr/2023.2/Manual/upm-embed.html)

## 로컬 패키지 설치

로컬 패키지는 컴퓨터에 있는 패키지 폴더를 프로젝트에 연결하는 방식이다. 패키지를 별도로 배포하기 전에 수정하고 확인하거나, 여러 프로젝트에서 같은 소스를 사용할 때 활용할 수 있다.

다음과 같이 프로젝트와 패키지 폴더를 배치했다고 가정한다. 패키지 루트에는 유효한 `package.json`이 있어야 한다. 아직 패키지가 없다면 뒤의 커스텀 패키지 예제를 먼저 만든다.

```text
Workspace/
├── MyProject/
│   ├── Assets/
│   ├── Packages/
│   │   └── manifest.json
│   └── ProjectSettings/
└── SharedPackages/
    └── com.devpioh.tools/
        ├── package.json
        ├── Runtime/
        └── Editor/
```

### 1. Package Manager에서 설치

1. Unity에서 **Window > Package Manager**를 연다.
2. 툴바의 설치 버튼을 누르고 **Install package from disk**를 선택한다. 이전 버전에서는 `+ > Add package from disk`로 표시될 수 있다.
3. `SharedPackages/com.devpioh.tools/package.json`을 선택한다.
4. 패키지 목록과 Project 창의 `Packages`에서 설치된 패키지를 확인한다.

선택할 파일은 **패키지 루트의 `package.json`**이다. 프로젝트의 `Packages/manifest.json`을 선택하는 것이 아니다. [로컬 폴더에서 패키지 설치](https://docs.unity3d.com/kr/2023.2/Manual/upm-ui-local.html)

설치한 뒤에는 연결된 원본 폴더를 수정한다. 같은 폴더를 참조하는 다른 프로젝트도 변경된 소스를 사용하므로, 프로젝트마다 독립적인 버전이 필요하다면 배포한 버전을 참조하는 구성이 적합하다.

### 2. manifest.json에서 직접 연결

Package Manager 대신 프로젝트의 `Packages/manifest.json`을 수정할 수도 있다. 기존 `dependencies`에 다음 항목을 추가한다. 아래는 설명을 위해 다른 패키지를 생략한 예제이므로, 기존 파일 전체를 덮어쓰지 않는다.

```json
{
  "dependencies": {
    "com.devpioh.tools": "file:../../SharedPackages/com.devpioh.tools"
  }
}
```

키는 패키지의 `package.json`에 적힌 `name`과 같아야 한다. 값은 `file:` 뒤에 **패키지 폴더 경로**를 작성하며, 끝에 `package.json`을 붙이지 않는다.

**상대 경로의 기준은 프로젝트 루트가 아니라 `Packages` 폴더다.** 위 구조에서는 `MyProject/Packages`에서 두 단계 올라가야 `Workspace`에 도달하므로 `../../SharedPackages/...`를 사용한다. [로컬 경로 지정](https://docs.unity3d.com/kr/2023.2/Manual/upm-localpath.html)

| 패키지 위치 | manifest.json에 작성할 값 |
| ------------- | -------------------------- |
| `MyProject/LocalPackages/com.devpioh.tools` | `file:../LocalPackages/com.devpioh.tools` |
| `Workspace/SharedPackages/com.devpioh.tools` | `file:../../SharedPackages/com.devpioh.tools` |
| `E:/UnityPackages/com.devpioh.tools` | `file:E:/UnityPackages/com.devpioh.tools` |

Windows에서도 경로 구분자로 `/`를 사용하면 JSON의 백슬래시 이스케이프를 신경 쓸 필요가 없다. 팀에서 사용할 때는 상대 경로와 폴더 배치를 함께 맞춘다. 매니페스트만 공유하고 패키지 원본을 전달하지 않으면 다른 컴퓨터에서 패키지를 찾을 수 없다.

로컬 패키지를 프로젝트 안에 둘 경우에는 `LocalPackages`처럼 별도 폴더를 사용한다. `Assets`에 놓고 로컬 패키지로도 연결하면 이중 임포트 문제가 발생할 수 있다. `Library`와 `ProjectSettings`도 보관 위치로 사용하지 않는다. [로컬 패키지의 위치 제한](https://docs.unity3d.com/kr/2023.2/Manual/upm-ui-local.html)

### 3. Embedded 패키지와의 차이

패키지 폴더를 프로젝트의 `Packages` 바로 아래에 배치하면 **Embedded 패키지**로 인식한다.

```text
MyProject/
└── Packages/
    ├── manifest.json
    ├── packages-lock.json
    └── com.devpioh.tools/
        ├── package.json
        ├── Runtime/
        └── Editor/
```

이 경우에는 해당 패키지를 사용하기 위해 `manifest.json`에 `file:` 참조를 추가할 필요가 없다. 같은 이름의 패키지가 매니페스트에 등록되어 있어도 Embedded 패키지가 우선한다. [내장 종속성](https://docs.unity3d.com/kr/2023.2/Manual/upm-embed.html)

여러 프로젝트에서 하나의 폴더를 참조하려면 로컬 패키지로, 특정 프로젝트와 소스를 함께 관리하려면 Embedded 패키지로 구성할 수 있다. Embedded 패키지를 사용하는 경우 패키지 폴더도 프로젝트 저장소에 포함한다.

## 커스텀 패키지 만들기

공통 로그 함수와 에디터 메뉴를 제공하는 `com.devpioh.tools` 패키지를 만들어 본다. 위의 `SharedPackages` 아래에 폴더를 만들거나, Embedded 방식으로 시작하려면 프로젝트의 `Packages` 아래에 같은 구조를 만든다. [커스텀 패키지 생성](https://docs.unity3d.com/kr/2023.2/Manual/CustomPackages.html)

### 1. 폴더 구성

```text
com.devpioh.tools/
├── package.json
├── README.md
├── CHANGELOG.md
├── LICENSE.md
├── Runtime/
│   ├── DevPioh.Tools.asmdef
│   └── PackageLogger.cs
├── Editor/
│   ├── DevPioh.Tools.Editor.asmdef
│   └── PackageToolsMenu.cs
├── Samples~/
│   └── Basic/
│       └── PackageExample.cs
└── Documentation~/
    └── index.md
```

| 파일 또는 폴더 | 용도 |
| ---------------- | ------ |
| `package.json` | 패키지 식별 정보와 종속성 |
| `Runtime` | 게임 실행 중에도 사용하는 코드와 에셋 |
| `Editor` | 커스텀 Inspector, 메뉴 등 에디터 전용 코드 |
| `Samples~` | 사용자가 선택하여 임포트할 예제 |
| `Documentation~` | 패키지 사용 문서 |
| `README.md` | 패키지 소개, 설치 방법, 개발 안내 |
| `CHANGELOG.md` | 버전별 변경 내역 |
| `LICENSE.md` | 패키지의 사용 조건 |

문서와 샘플 등은 필요에 따라 추가한다. 테스트가 있다면 `Tests/Editor`, `Tests/Runtime`에 별도의 테스트 어셈블리와 함께 구성할 수 있다. [권장 패키지 레이아웃](https://docs.unity3d.com/kr/2023.2/Manual/cus-layout.html)

폴더 이름 끝의 `~`는 Unity가 해당 폴더의 내용을 일반 에셋처럼 임포트하지 않도록 한다. 따라서 `Samples~`에 있는 코드는 패키지 설치만으로 컴파일되지 않고, 샘플을 프로젝트에 임포트한 뒤 사용한다.

### 2. package.json 작성

패키지 루트에 다음 파일을 만든다. `samples`는 뒤에서 작성할 `Samples~/Basic` 폴더를 가리킨다.

```json
{
  "name": "com.devpioh.tools",
  "version": "1.0.0",
  "displayName": "DevPioh Tools",
  "description": "공통 로그 함수와 에디터 도구를 제공하는 예제 패키지.",
  "unity": "2023.2",
  "author": {
    "name": "DevPioh"
  },
  "dependencies": {},
  "samples": [
    {
      "displayName": "Basic",
      "description": "PackageLogger 사용 예제",
      "path": "Samples~/Basic"
    }
  ]
}
```

| 프로퍼티 | 설명 |
| ---------- | ------ |
| `name` | 필수. `com.조직명.패키지명` 형태의 고유 식별자 |
| `version` | 필수. `major.minor.patch` 형식의 패키지 버전 |
| `displayName` | Package Manager에 표시할 이름 |
| `description` | 패키지 설명 |
| `unity` | 호환 가능한 최소 Unity 버전. `major.minor` 형식 |
| `dependencies` | 이 패키지가 사용하는 다른 패키지와 버전 |
| `samples` | Package Manager에서 임포트할 샘플 목록 |

`name`과 `displayName`은 용도가 다르다. 프로젝트 매니페스트의 키에는 `DevPioh Tools`가 아니라 `com.devpioh.tools`를 사용한다. 예제의 `unity`는 `2023.2`로 선언했으며, 하위 버전까지 지원하려면 해당 버전에서 확인한 뒤 값을 정한다. [패키지 매니페스트](https://docs.unity3d.com/kr/2023.2/Manual/upm-manifestPkg.html)

### 3. Runtime 어셈블리와 코드 작성

패키지의 C# 스크립트는 `.asmdef`로 어셈블리를 정의한다. 먼저 `Runtime/DevPioh.Tools.asmdef`를 작성한다.

```json
{
  "name": "DevPioh.Tools",
  "rootNamespace": "DevPioh.Tools",
  "references": [],
  "includePlatforms": [],
  "excludePlatforms": [],
  "autoReferenced": true
}
```

`includePlatforms`와 `excludePlatforms`가 비어 있으므로 특정 플랫폼으로 제한하지 않는다. `autoReferenced`는 `Assembly-CSharp` 같은 사전 정의 어셈블리에서 이 어셈블리를 참조할 수 있도록 한다. 직접 만든 다른 `.asmdef`에서도 자동으로 참조된다는 의미는 아니다. [어셈블리 정의](https://docs.unity3d.com/kr/2023.2/Manual/ScriptCompilationAssemblyDefinitionFiles.html)

`Runtime/PackageLogger.cs`에는 공통으로 사용할 함수를 작성한다.

```cs
using UnityEngine;

namespace DevPioh.Tools
{
    public static class PackageLogger
    {
        public static void Log(string message)
        {
            Debug.Log($"[DevPioh] {message}");
        }
    }
}
```

### 4. Editor 어셈블리 분리

`Editor/DevPioh.Tools.Editor.asmdef`를 작성한다.

```json
{
  "name": "DevPioh.Tools.Editor",
  "rootNamespace": "DevPioh.Tools.Editor",
  "references": [
    "DevPioh.Tools"
  ],
  "includePlatforms": [
    "Editor"
  ],
  "autoReferenced": true
}
```

Editor 어셈블리가 Runtime의 함수를 호출하므로 `references`에 `DevPioh.Tools`를 추가한다. 여기에 들어가는 값은 패키지 이름이나 네임스페이스가 아니라 **참조할 어셈블리의 `name`**이다.

패키지 안에서 `Editor`라는 폴더 이름만으로 빌드 제외를 처리한다고 가정하지 않는다. `includePlatforms`를 `Editor`로 제한하여 에디터 전용 코드를 분리하고, Runtime 어셈블리에서는 Editor 어셈블리를 참조하지 않는다. [어셈블리 정의 및 패키지](https://docs.unity3d.com/kr/2023.2/Manual/cus-asmdef.html), [패키지 폴더 규칙](https://docs.unity3d.com/kr/2023.2/Manual/cus-layout.html)

`Editor/PackageToolsMenu.cs`를 작성한다.

```cs
using DevPioh.Tools;
using UnityEditor;

namespace DevPioh.Tools.Editor
{
    public static class PackageToolsMenu
    {
        [MenuItem("Tools/DevPioh/Print Package Message")]
        private static void PrintPackageMessage()
        {
            PackageLogger.Log("에디터에서 패키지 호출");
        }
    }
}
```

이 구조에서는 `UnityEditor`를 사용하는 코드가 에디터 전용 어셈블리에만 포함된다. Runtime 코드에서 `UnityEditor` API를 사용하면 플레이어 빌드 시 오류가 발생할 수 있다.

### 5. 샘플 작성과 동작 확인

`Samples~/Basic/PackageExample.cs`를 작성한다.

```cs
using DevPioh.Tools;
using UnityEngine;

public class PackageExample : MonoBehaviour
{
    private void Start()
    {
        PackageLogger.Log("프로젝트에서 패키지 호출");
    }
}
```

1. 앞서 설명한 방법으로 패키지의 `package.json`을 선택하여 로컬 설치한다. Embedded 방식으로 만들었다면 Unity에서 패키지가 인식되는지 확인한다.
2. 컴파일 후 **Tools > DevPioh > Print Package Message**를 선택하고 Console의 출력을 확인한다.
3. Package Manager에서 **DevPioh Tools**를 선택하고 **Samples**의 **Basic**을 임포트한다.
4. `Assets` 아래로 복사된 `PackageExample`을 씬의 GameObject에 추가한다.
5. 플레이 모드에서 `[DevPioh] 프로젝트에서 패키지 호출`이 출력되는지 확인한다.
6. 대상 플랫폼으로 빌드하여 에디터 코드가 런타임 빌드에 포함되지 않는지도 확인한다.

샘플 임포트는 `Samples~`의 내용을 프로젝트의 `Assets` 아래로 복사하는 작업이다. 임포트한 코드를 수정해도 패키지 원본 샘플이 함께 수정되지는 않는다. [패키지용 샘플 생성](https://docs.unity3d.com/kr/2023.2/Manual/cus-samples.html)

위 예제는 샘플 코드가 기본 `Assembly-CSharp`에서 컴파일되는 구성을 전제로 한다. 사용하는 프로젝트 코드에 별도의 `.asmdef`가 있다면 해당 어셈블리의 **Assembly Definition References**에도 `DevPioh.Tools`를 추가한다.

## 종속성과 버전 관리

패키지 종속성과 C# 어셈블리 참조는 각각 설정해야 한다.

| 설정 | 의미 |
| ------ | ------ |
| `package.json`의 `dependencies` | 필요한 다른 패키지와 버전을 UPM에 선언 |
| `.asmdef`의 `references` | C# 코드가 사용할 다른 어셈블리를 컴파일러에 선언 |

다른 패키지를 `dependencies`에 추가해도 그 안의 어셈블리가 자동으로 참조되지는 않는다. 반대로 `.asmdef`에 이름만 추가한다고 해당 패키지가 설치되는 것도 아니다. [어셈블리 정의 및 패키지](https://docs.unity3d.com/kr/2023.2/Manual/cus-asmdef.html)

Unity 2023.2의 패키지 매니페스트에서는 종속성 버전에 `1.0.0`처럼 특정 SemVer 값을 사용한다. npm에서 사용하는 `^1.0.0` 같은 버전 범위는 지원하지 않는다. 로컬 경로는 프로젝트의 `manifest.json`에서 지정하며, 패키지 간 Git 종속성도 지원하지 않으므로 Git URL은 프로젝트 매니페스트에 작성한다. [패키지 매니페스트](https://docs.unity3d.com/kr/2023.2/Manual/upm-manifestPkg.html), [Git 종속성](https://docs.unity3d.com/kr/2023.2/Manual/upm-git.html)

로컬 패키지의 `version`을 바꾸는 것만으로 이전 소스가 보관되지는 않는다. 개발 중에는 원본을 Git으로 관리하고, 다른 프로젝트에 배포할 때는 태그나 커밋을 지정한 Git 패키지 또는 레지스트리의 버전으로 공유할 수 있다.

`Runtime`, `Editor`의 에셋과 `.asmdef` 등에 Unity가 생성한 **`.meta` 파일도 함께 관리**한다. 기존 코드를 패키지로 옮길 때 `.meta`를 보존해야 GUID를 사용하는 에셋 참조가 끊어지는 일을 줄일 수 있다. 위 폴더 예제에서는 가독성을 위해 `.meta` 파일을 생략했다. [패키지 레이아웃의 메타 파일 규칙](https://docs.unity3d.com/kr/2023.2/Manual/cus-layout.html)

## 자주 확인할 문제

| 증상 | 확인할 내용 |
| ------ | ------------- |
| 로컬 패키지를 찾지 못함 | `file:` 경로가 `Packages` 폴더 기준인지, 원본 폴더와 `package.json`이 존재하는지 확인 |
| 패키지가 정상적으로 로드되지 않음 | JSON 문법, 필수 `name`·`version`, 프로젝트 매니페스트의 패키지 이름 확인 |
| 설치했지만 코드에서 타입을 찾지 못함 | 호출하는 코드의 `.asmdef`에서 패키지 어셈블리를 참조하는지 확인 |
| 에디터에서는 되지만 빌드에 실패함 | Runtime의 `UnityEditor` 사용과 Editor 어셈블리의 플랫폼 제한 확인 |
| 지정한 버전과 다른 코드가 사용됨 | 같은 이름의 Embedded 패키지가 `Packages` 아래에 있는지 확인 |
| 샘플이 나타나지 않음 | `package.json`의 `samples`와 실제 `Samples~` 경로가 일치하는지 확인 |

## 출처 및 같이 보기

- [Unity Package Manager](https://docs.unity3d.com/kr/2023.2/Manual/Packages.html)
- [로컬 폴더에서 패키지 설치](https://docs.unity3d.com/kr/2023.2/Manual/upm-ui-local.html)
- [로컬 폴더 또는 타르볼 경로](https://docs.unity3d.com/kr/2023.2/Manual/upm-localpath.html)
- [내장 종속성](https://docs.unity3d.com/kr/2023.2/Manual/upm-embed.html)
- [커스텀 패키지 생성](https://docs.unity3d.com/kr/2023.2/Manual/CustomPackages.html)
- [패키지 레이아웃](https://docs.unity3d.com/kr/2023.2/Manual/cus-layout.html)
- [패키지 매니페스트](https://docs.unity3d.com/kr/2023.2/Manual/upm-manifestPkg.html)
- [어셈블리 정의](https://docs.unity3d.com/kr/2023.2/Manual/ScriptCompilationAssemblyDefinitionFiles.html)
- [패키지용 샘플 생성](https://docs.unity3d.com/kr/2023.2/Manual/cus-samples.html)
