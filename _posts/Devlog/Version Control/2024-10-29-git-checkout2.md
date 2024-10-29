---
title: "git checkout 특정 파일 되돌리기"
date: 2024-10-29
last_modified_at: 2024-10-29

toc: true
toc_sticky: true

categories:
    - version control
tag:
    - [version control, git]
---

이번 프로젝트에서 기존에 사용하던 svn이 아니라 git을 사용하므로, 자주 사용하는거 위주로 정리.

생각 날 때마다 업데이트  

## checkout
```
// 특정 커밋으로 파일을 되돌리기
git checkout <커밋 해시> -- <파일 경로>

// 다수의 파일일 경우
git checkout <커밋 해시> -- <파일1> <파일2> <파일3>

// 특정 파일의 커밋 해시를 알고 싶을 경우
git log -- <파일 경로>
```

만약 최신 커밋으로 되돌리고 싶은 경우 <커밋 해시> 대신 ***HEAD*** 키워드 사용

## 출처 및 같이 보기
 - <a href="https://git-scm.com/book/ko/v2/">git document</a>