---
title: "Quick Sort"
date: 2022-08-01
last_modified_at: 2022-08-02

toc: true
toc_sticky: true

categories:
    - algorithm
tag:
    - [algorithm, sort, quick]
---

## 개요
병합 정렬(Merge Sort)과 함께 대표적인 정렬 알고리즘 중 하나이다.
   
병합 정렬과 동일하게 평균적으로 O(nlogn)의 속도를 가지지만, 최악의 경우(미리 정렬된 상태)는 O(N^2)의 속도를 가진다.

최악의 경우의 속도가 O(N^2)가 될 수 있다는 부분은 회피가 가능하다.

대부분의 순열들은 정렬된 상태를 가지지 않고, 적절한 피봇(부분 순열의 중간 값)이 선정하는 처리로 회피할 수 있다. 

그리고 *메모리 참조가 지역화되어 있기 때문에 CPU 캐시의 히트율이 높아* 더 효률적으로 CPU가 작업 가능하므로, 다른 O(nlogn)속도의 정렬 알고리즘보다 더 나은 속도를 보여준다.

정렬로 인해 삽입 순서가 흐트러질 수 있어 불안정 정렬에 속한다.

### 알고리즘
1. 리스트에서 정렬 기준이 되는 원소를 고른다. 해당 원소는 피봇이라 한다.
2. 피봇을 기준으로 작은 값은 피봇 앞으로, 큰 값은 피봇의 뒤로 이동시킨다
3. 이동 된 값들은 피봇 값을 기준으로 리스트는 두 부분으로 분할된다.
4. 둘로 나눠진 리스트를 1 ~ 3번 과정을 재귀적으로 처리한다. 리스트의 크기가 0, 1이 될 때까지 반복한다.

### 구현
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
- <a href="https://ko.wikipedia.org/wiki/%ED%80%B5_%EC%A0%95%EB%A0%AC">위키 퀵 정렬</a>
- <a href="https://www.geeksforgeeks.org/quick-sort/">GeeksForGeeks 퀵 정렬</a>
- <a href="https://na982.tistory.com/109?category=92742">na982님의 병합 정렬과 퀵 정렬 설명(추천)</a>
- <a href="https://velog.io/@sparkbosing/%ED%80%B5%EC%A0%95%EB%A0%AC-%ED%80%B5%EC%A0%95%EB%A0%AC%EC%9D%98-%EC%B5%9C%EC%95%85%EC%9D%B4-n2%EC%9D%B8-%EC%9D%B4%EC%9C%A0">퀵 정렬 - 최악의 경우</a>
- <a href="https://stackoverflow.com/questions/70402/why-is-quicksort-better-than-mergesort">why quick sort better than merge sort?</a>
- <a href="https://www.acmicpc.net/problem/2750">백준 2750</a>