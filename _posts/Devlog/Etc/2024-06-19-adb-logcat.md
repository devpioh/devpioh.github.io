---
title: "ADB 명령어"
date: 2024-06-19
last_modified_at: 2024-07-03

toc: true
toc_sticky: true

categories:
    - etc
tag:
    - [adb, logcat]
---

## 개요

맨날 잊어먹어서 자주쓰는 명령어 기록

### 연결 디바이스확인

```ps
adb devices [option]
```

### 타겟 디바이스에 apk설치

```ps
adb install [option] [apk 경로]
```

### 특정 apk pid 확인

```ps
adb shell ps | select-string [apk package name]
```

### pid로 로그캣 확인

```ps
adb logcat --pid=[pid]
```

## 출처 및 같이 보기
