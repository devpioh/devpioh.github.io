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
Merge Sort와 함께 대표적인 정렬 알고리즘 중 하나이다.
   
속도는 머지정렬과 동일하게 평균적으로 O(nlogn)이며, 최악의 경우(미리 정렬된 상태) O(N^2)의 속도를 가진다.
    



### 구현
```cs
// na982님의 퀵정렬 구현이 읽기도 편하고 구현도 쉬워서 가져왔다.
public void Quick(List<int> list, int left, int right)
{
    if(left >= right)
        return;

    var l = left - 1;
    var r = right + 1;
    var pivot = list[right];         //  적절한 중간 값을 선정하는게 퀵 정렬에서 가장 중요하다.

    while(true)
    {
        while(list[++l] < pivot);   // 왼쪽에서 오른쪽으로
        while(list[--r] > pivot);   // 오른쪽에서 왼쪽으로
        if(l >= r)  break;          // 비교 인덱스가 동일한경우 종료한다.

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
- <a herf="https://www.geeksforgeeks.org/quick-sort/">Quick Sort</a>
- <a href="https://na982.tistory.com/109?category=92742">na982님의 머지소트와 퀵소트 설명(추천)</a>
- <a href="https://velog.io/@sparkbosing/%ED%80%B5%EC%A0%95%EB%A0%AC-%ED%80%B5%EC%A0%95%EB%A0%AC%EC%9D%98-%EC%B5%9C%EC%95%85%EC%9D%B4-n2%EC%9D%B8-%EC%9D%B4%EC%9C%A0">퀵정렬 - 최악의 경우</a>
- <a href="https://www.acmicpc.net/problem/2750">백준 2750</a>
- <a href="https://www.acmicpc.net/problem/2750">백준 2751(문자열 처리 주의))</a>