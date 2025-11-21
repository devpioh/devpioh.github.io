---
title: "ADB 명령어"
date: 2024-06-19
last_modified_at: 2024-07-03

toc: true
toc_sticky: true

categories:
    - etc
tags:
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
adb logcat -c               // 이전 로그지우기
adb logcat --pid=[pid]      // 해당 pid로 로그캣 출력

// 만약 상세 로그 필요시 디바이스에 찍히는 모든 로그를 기록할 필요.

adb logcat > logcat-logging.txt // txt 파일로 남기기, ctrl + c를 이용해 로깅 종료.
```

## 출처 및 같이 보기
