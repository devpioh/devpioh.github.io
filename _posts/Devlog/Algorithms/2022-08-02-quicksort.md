---
title: "Quick Sort"
date: 2022-08-02
last_modified_at: 2025-10-06

toc: true
toc_sticky: true

categories:
    - algorithm
tags:
    - [algorithm, sort, quick]
---

## 개요

**퀵 정렬(Quick Sort)**은 병합 정렬(Merge Sort)과 함께 대표적인 정렬 알고리즘 중 하나이다.

병합 정렬과 마찬가지로 *평균 시간 복잡도는 O(nlogn)*이지만, **입력이 이미 정렬된 경우(또는 역순 인 경우)**에는 *최악의 경우로 O(n^2)*의 성능으로 저하 될 수 있다.

하지만 이 경우 피벗을 적절하게 선택함으로써 이러한 경우를 대부분 회피할 수 있다.

그리고 **메모리 참조의 지역성(Locality)** 덕분에 CPU 캐시의 히트율이 높아져, 같은 O(nlogn)복잡도를 가지는 병합 정렬 보다 실제 속도가 더 빠른 경우가 많다.

정렬로 인해 삽입 순서가 흐트러질 수 있어 불안정 정렬에 속한다.

## 알고리즘

1. 피벗 선택(Pivot Selection)
    - 리스트에서 정렬 기준이 되는 원소(피벗)을 선택한다.
2. 분할(Partitioning)
    - 피벗보다 작은 값은 왼쪽, 큰 값은 오른쪽으로 이동시켜 리스트를 두 부분으로 분할한다.
3. 재귀 정렬(Recursion)
    - 분할된 각 부분 리스트에 대해 위 과정을 재귀적으로 반복한다.
    - 리스트의 크기가 0 또는 1이 될 때 종료한다.

## 구현

```cs
// na982님의 퀵 정렬 구현이 읽기도 편하고 구현도 쉬워서 가져왔다.
public void Quick(List<int> list, int left, int right)
{
    if(left >= right)
        return;

    var l = left - 1;
    var r = right + 1;

    // 적절한 중간 값을 선정하는게 퀵 정렬에서 가장 중요하다.
    // 리스트의 중간 인덱스로 잡아도 되고 처음 혹은 리스트의 마지막 요소를 피봇으로 선택한다.
    //var pivot = list[(l + r) / 2];         
    var pivot = list[right];         

    while(true)
    {
        while(list[++l] < pivot);   // 왼쪽에서 오른쪽으로
        while(list[--r] > pivot);   // 오른쪽에서 왼쪽으로
        if(l >= r)  break;          // 비교 인덱스가 동일한 경우 종료

        // l과 r의 위치를 교체한다.
        int temp = list[l];
        list[l] = list[r];
        list[r] = temp;
    }

    Quick(list, left, l - 1);
    Quick(list, r + 1, right);
}
```

## 출처 및 같이 보기

- [퀵 정렬](https://ko.wikipedia.org/wiki/%ED%80%B5_%EC%A0%95%EB%A0%AC)
- [GeeksForGeeks 퀵 정렬](https://www.geeksforgeeks.org/quick-sort/)
- [na982님의 병합 정렬과 퀵 정렬 설명](https://na982.tistory.com/109?category=92742)
- [why quick sort better than merge sort?(왜 퀵정렬이 병합정렬보다 나은가요?)](https://stackoverflow.com/questions/70402/why-is-quicksort-better-than-mergesort)
- [백준 2750](https://www.acmicpc.net/problem/2750)