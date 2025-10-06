---
title: "Merge Sort"
date: 2022-08-03
last_modified_at: 2025-10-06

toc: true
toc_sticky: true

categories:
    - algorithm
tags:
    - [algorithm, sort, merge]
---

## 개요

**병합 정렬(Merge Sort)**는 퀵 정렬(Quick Sort)와 함께 대표적인 **분할 정복(Divide and Conquer)** 기반의 정렬 알고리즘이다.

평균 및 최악의 경우 모두 **O(nlogn)**의 시간 복잡도를 가지며, 안정 정렬(Stable Sort)에 속한다.

이는 동일한 값을 가진 데이터의 상대적 순서가 유지된다는 뜻이다.

하지만 **추가적인 메모리 공간( O(n) )** 이 필요하다는 단점이 있다.

이 때문에 실제 시스템 구현에서는 퀵 정렬보다 느리게 동작할 수 있지만, 데이터가 크고 안정성이 중요한 경우(예 : 외부 정렬, 링크드 리스트 정렬 등)에서 자주 사용된다.

## 알고리즘

병합 정렬은 배열을 절반으로 계속 *분할(Divide)*하고, 각 부분 배열이 정렬된 상태가 되면 두 부분을 **병합(Merge)**하여 전체를 정렬하는 방식이다.

1. 리스트를 반으로 분할 한다.
2. 각 부분 리스트를 재귀적으로 정렬한다.
3. 두 개의 정렬된 리스트를 하나의 정렬된 리스트로 병합한다.
4. 리스트의 크기가 1 이하가 될 때까지 반복한다.

## 동작 과정

아래 예시는 `[38, 27, 43, 3, 9, 82, 10]`를 병합 정렬하는 과정을 단순화 한 것이다.

![wiki-divide-and-conquer-merge-sort](https://en.wikipedia.org/wiki/Divide-and-conquer_algorithm#/media/File:Merge_sort_algorithm_diagram.svg)

## 구현

```cs
public void MergeSort(List<int> list, int left, int right)
{
    if (left >= right)
        return;

    int mid = (left + right) / 2;

    // 1. 왼쪽 / 오른쪽 부분 리스트 재귀 정렬
    MergeSort(list, left, mid);
    MergeSort(list, mid + 1, right);

    // 2. 정렬된 두 부분을 병합
    Merge(list, left, mid, right);
}

private void Merge(List<int> list, int left, int mid, int right)
{
    int i = left;
    int j = mid + 1;
    var temp = new List<int>();

    // 1. 두 부분 리스트 비교 및 병합
    while (i <= mid && j <= right)
    {
        if (list[i] <= list[j])
            temp.Add(list[i++]);
        else
            temp.Add(list[j++]);
    }

    // 2. 남은 요소 추가
    while (i <= mid)
        temp.Add(list[i++]);
    while (j <= right)
        temp.Add(list[j++]);

    // 3. 병합된 결과를 원본에 복사
    for (int k = 0; k < temp.Count; k++)
        list[left + k] = temp[k];
}
```

## 출처 및 같이 보기

- [위키 분할정복](https://en.wikipedia.org/wiki/Divide-and-conquer_algorithm)
