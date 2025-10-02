---
title: "셰이더 베리언트 자동 수집"
date: 2025-10-02
last_modified_at: 2025-10-02

toc: true
toc_sticky: true

categories:
    - unity
tag:
    - [unity, shader, shader variants]
---

## 개요

많은 사용 셰이더를 사용하는데 각 상용 셰이더마다 각기 다른 셰이더 키워드를 사용한다.

이로 인해 셰이더 컴파일로 인하여 빌드 시간이 증대되고 관리도 힘든 상황이 자주 발생.

조금 간편하게 처리 하는 방법이 없나 찾아보니 유니티에서 셰이더별 배리언트를 자동으로 추적해 기록해두는 방법이 있었다.

### 설정

1. 먼저 에디터에서 처음부터 끝가지 사용된 셰이더가 있는 컨텐츠를 플레이 한다.
2. 그 후 Edit-Project Settigs-Graphics-Shader Settings 항목을 찾는다.
3. 해당 항목에서 Currently tracked: 라는 항목이 있는데 해당 플레이에서 추적된 셰이더의 배리언트를 확인 가능
4. 추적된 셰이더 배리언트를 배리언트 콜렉션 파일로 저장도 가능하다.

## 출처 및 같이 보기
