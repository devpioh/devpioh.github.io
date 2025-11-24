---
title: "BundleTool 사용법"
date: 2025-11-21
last_modified_at: 2025-11-21

toc: true
toc_sticky: true

categories:
    - etc
tags:
    - [bundletool]
---

## 개요

맨날 잊어먹어서 자주쓰는 명령어 기록.

android bundle tool 을 이용하여 Play Asset delivery 테스트.

### 현재 상황

1. 아직 Adressable을 도입을 하지 않은 상황.
2. Unity 기본 제공 옵션(Build App Bundle, Split Application Binary)를 이용하여 빌드

### 자바 설치 및 bundletool 버전 확인

```ps
java --version  // 자바 설치 및 버전 확인

java -jar [bundle tool path] version  // 버전 툴 버전 확인
```

- java 가 설치 되지 않은 경우 **[jdk](https://www.oracle.com/kr/java/technologies/downloads/)** 설치
- bundletool 이 없는 경우 **[bundletool](https://github.com/google/bundletool/releases)** 다운로드

### Bundle Tool을 간편하게 쓰기 위해 프로필 등록

```ps
Add-Content $PROFILE @'
>>> function bundletool {
>>> param(
>>> [Parameter(ValueFromRemainingArgument = $true)]
>>> [string[]] $Args
>>> )
>>>
>>> java -jar "[bundle tool path]" @Args
>>> }
>>> `@
```

파워쉘 프로필에 위의 함수를 등록하여 bundletool로 **java -jar "[bundle tool path]"**을 **bundletool**로 접근하도록 처리.

### Bundle Tool을 이용하여 device spec 출력

```ps
java -jar [bundle tool path] get-device-spec --output=[디바이스 spec json 출력 위치]
```

연결된 디바이스 스펙을 json 형태로 저장하여 추후 apks 시 해당 스펙에 맞게 apks를 빌드 가능.

### Bundle Tool을 이용하여 apks 빌드

```ps
java -jar [번들 툴 path] build-apks  
--bundle=[대상 aab 파일 경로] 
--output=[빌드된 apks 파일 경로] 
--local-testing                     // Play Asset Delivery 로컬 테스트 옵션 or 옵션 대신 
--connected-device                  // 연결된 디바이스 스펙으로 빌드
--ks=[keystore 위치]                
--ks-key-alias=[keystore alias]     
--key-pass=pass:[keystore password] // 키스토어 패스워드
```

- **--local-testing** 옵션을 이용해서 PAD(Play Assets Delivery) 테스팅이 가능.
- **--connected-deivce** 옵션을 이용해 컴퓨터에 연결된 디바이스에 맞는 apks로 빌드. 만약 따로 get-device-spec을 이용해 디바이스 spec json을 뽑아 놨다면 **--devic-spec** 옵션을 이용해 해당 스펙으로 빌드 가능.

### Bundle Tool을 이용하여 apks 설치

```ps
java -jar [번들 툴 path] build-apks --apks=[빌드된 apks 파일 경로] 
```

### Bundle Tool을 이용하여 빌드된 apks를 각각의 apk로 분리

```ps
java -jar [번들 툴 path] extract-apks
--apks=[빌드된 apks 파일 경로] 
--output-dir=[추출한 apk 경로] 
--device-spec=[타게팅 디바이스 스팩]
```

합처진 apks를 각각의 apk로 분리 하여 확인이 필요할때 사용.

## 출처 및 같이 보기

- [번들툴 사용법](https://developer.android.com/tools/bundletool?hl=ko)
- [PAD 테스트](https://developer.android.com/guide/playcore/feature-delivery/on-demand?hl=ko#local-testing)
