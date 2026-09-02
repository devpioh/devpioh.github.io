---
title: "Difference Array and Prefix Sum(차분 배열과 누적합)"
date: 2025-05-16
last_modified_at: 2026-09-02

toc: true
toc_sticky: true

categories:
    - algorithm
tags:
    - [algorithm, difference array, prefix sum]
---

## 개요

**누적합(Prefix Sum)**은 배열의 앞에서부터 값을 누적하여 구간 합 쿼리를 빠르게 처리하는 방법이다.

반대로 **차분 배열(Difference Array)**은 인접한 원소의 차이를 저장하여 여러 번의 구간 변경을 빠르게 기록하는 방법이다.

| 방법 | 전처리 또는 복원 | 구간 연산 |
|----|----|----|
| 누적합 | `O(n)` | 구간 합 `O(1)` |
| 차분 배열 | `O(n)` | 구간 덧셈 `O(1)` |

두 방법은 서로 반대되는 관계를 가진다. 원본 배열을 누적하면 누적합이 되고, 차분 배열을 누적하면 원본 배열이 복원된다.

## 누적합(Prefix Sum)

배열 `A`에 대해 누적합 배열 `P`를 다음과 같이 정의한다.

```text
P[0] = 0
P[i + 1] = P[i] + A[i]
```

`P[i]`에는 `A[0]`부터 `A[i - 1]`까지의 합이 저장된다. 앞에 0을 하나 추가하면 `left = 0`인 경우도 별도의 예외 처리 없이 닫힌 구간 `[left, right]`의 합을 구할 수 있다.

```text
sum(left, right) = P[right + 1] - P[left]
```

예를 들어 `A = [3, 1, 4, 2]`라면 `P = [0, 3, 4, 8, 10]`이다. 구간 `[1, 3]`의 합은 `P[4] - P[1] = 10 - 3 = 7`이 된다.

```cs
public static long[] BuildPrefixSum(IReadOnlyList<int> values)
{
    var prefix = new long[values.Count + 1];

    for (int i = 0; i < values.Count; i++)
        prefix[i + 1] = prefix[i] + values[i];

    return prefix;
}

// left와 right를 모두 포함하는 구간의 합을 반환한다.
public static long RangeSum(long[] prefix, int left, int right)
{
    return prefix[right + 1] - prefix[left];
}
```

누적합 생성에는 `O(n)`이 필요하지만, 생성한 뒤에는 각 구간 합을 `O(1)`에 구할 수 있다. 따라서 원본 배열이 바뀌지 않고 구간 합 쿼리가 여러 번 주어질 때 효과적이다.

## 차분 배열(Difference Array)

원본 배열 `A`의 차분 배열 `D`는 다음과 같이 정의할 수 있다.

```text
D[0] = A[0]
D[i] = A[i] - A[i - 1]
```

닫힌 구간 `[left, right]`의 모든 원소에 `value`를 더하려면 경계만 변경한다.

```text
D[left] += value
D[right + 1] -= value
```

`left`에서 변화가 시작되고 `right + 1`에서 변화가 취소되도록 표시하는 것이다. 모든 변경을 기록한 후 차분 배열을 한 번 누적하면 최종 배열이 만들어진다.

```cs
public static long[] ApplyRangeAdds(
    IReadOnlyList<int> values,
    IEnumerable<(int Left, int Right, long Value)> updates)
{
    int n = values.Count;
    var diff = new long[n + 1];

    // 원본 배열을 차분 배열로 변환한다.
    long previous = 0;
    for (int i = 0; i < n; i++)
    {
        diff[i] = values[i] - previous;
        previous = values[i];
    }

    // 각 구간 변경은 두 경계만 표시한다.
    foreach (var update in updates)
    {
        diff[update.Left] += update.Value;
        diff[update.Right + 1] -= update.Value;
    }

    // 차분 배열을 누적하여 최종 배열을 복원한다.
    var result = new long[n];
    long current = 0;

    for (int i = 0; i < n; i++)
    {
        current += diff[i];
        result[i] = current;
    }

    return result;
}
```

구간 변경이 `q`번이라면 각 변경을 `O(1)`에 기록하고 마지막에 `O(n)`으로 복원하므로 전체 시간 복잡도는 **O(n + q)**이다.

## 함께 사용하는 경우

여러 구간 변경이 끝난 뒤 최종 배열의 구간 합 쿼리까지 처리해야 한다면 다음 순서로 두 방법을 함께 사용할 수 있다.

1. 차분 배열에 모든 구간 변경을 기록한다.
2. 차분 배열을 누적하여 최종 배열을 복원한다.
3. 최종 배열의 누적합을 만든다.
4. 각 구간 합 쿼리를 `O(1)`에 처리한다.

## 주의할 점

- 문제의 구간이 닫힌 구간 `[left, right]`인지 반열린 구간 `[left, right)`인지 먼저 확인한다.
- 차분 배열에서 `right + 1`을 사용하므로 크기가 `n + 1`인 배열을 준비하면 경계 처리가 단순해진다.
- 원소 하나는 `int` 범위여도 전체 합은 범위를 넘을 수 있으므로 누적값은 `long`을 사용하는 편이 안전하다.
- 값의 변경과 구간 합 쿼리가 번갈아 주어지고 즉시 답해야 한다면 펜윅 트리나 세그먼트 트리를 고려한다.

## 출처 및 같이 보기

- [Prefix sum](https://en.wikipedia.org/wiki/Prefix_sum)
- [Difference array](https://en.wikipedia.org/wiki/Difference_array)
- [Fenwick tree](https://en.wikipedia.org/wiki/Fenwick_tree)
