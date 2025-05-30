---
title: "Visual Stuio code 확장 다운로드"
date: 2025-5-30
last_modified_at: 2024-11-21

toc: true
toc_sticky: true

categories:
    - etc
tag:
    - [visual studio, vs, extension]
---

## 개요

회사 폐쇄망 정책으로 인해 비주얼 스튜디오 코드를 에디터에서 직접 검색해 설치가 불가능하다.

이전에는 [마켓플레이스](https://marketplace.visualstudio.com/vscode)에서 직접 적으로 다운로드 링크를 제공했지만, 어느새 이마저도 사라진 상황!

어쩔수 없이 마켓플레이스 REST API를 사용해 다운받는 방법을 정리해 보려고 한다.

## 준비

필요한것은 3가지

1. 확장 플러그인 이름
2. 퍼블리셔
3. 버전

일단 마켓플레이스에 접속해 다운받으려는 확장 플러그인을 찾아 URL을 확인한다.

예시 -> `https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one`

URL의 마지막 부분을 보면 `itemName=yzhang.markdown-all-in-one`이 있는데 `{퍼블리셔}.{확장 플러그인 이름}`의 구조로 되어있다.

필요한것은 갖춰졌으니 이제 다운로드 URL을 만들어 보자.

다운로드 API 구조는 아래와 같다.

`https://marketplace.visualstudio.com/_apis/public/gallery/publishers/{퍼블리셔}/vsextensions/{확장 플러그인 이름}/{버전}/vspackage`

이 부분에 내용을 채워 넣으면 vsix를 다운로드 받을수 있다.

## 출처 및 같이 보기

- [Can't download VSIX extensions from the web marketplace anymore?](https://www.reddit.com/r/vscode/comments/1i6k7gf/cant_download_vsix_extensions_from_the_web/)