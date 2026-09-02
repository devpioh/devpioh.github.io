---
title: "Manhattan Distance(맨해튼 거리)"
date: 2025-06-11
last_modified_at: 2026-09-02

toc: true
toc_sticky: true

categories:
    - algorithm
tags:
    - [algorithm, manhattan]
---

## 개요

**맨해튼 거리(Manhattan Distance)**는 격자에서 가로와 세로 방향으로만 이동할 수 있을 때 두 점 사이의 거리를 측정하는 방법이다.

도로가 직각으로 배치된 도시에서 블록을 따라 이동하는 모습과 비슷하여 **Taxicab Distance**라고도 하며, 각 좌표 차이의 절댓값을 더해서 구한다.

2차원 평면의 두 점 `A(x1, y1)`, `B(x2, y2)` 사이의 맨해튼 거리는 다음과 같다.

```text
|x1 - x2| + |y1 - y2|
```

`d`차원에서는 모든 좌표축에 대해 같은 방식으로 확장한다.

```text
distance(a, b) = |a1 - b1| + |a2 - b2| + ... + |ad - bd|
```

## 구현

```cs
public static long ManhattanDistance(
    (long X, long Y) first,
    (long X, long Y) second)
{
    return Math.Abs(first.X - second.X)
        + Math.Abs(first.Y - second.Y);
}
```

장애물이 없는 4방향 격자에서 한 칸의 이동 비용이 모두 같다면 이 값은 두 점 사이의 최단 이동 횟수와 같다.

대각선 이동이 가능하거나 방향마다 비용이 다르다면 같은 식을 그대로 사용할 수 없다. 또한 장애물이 있으면 실제 최단 거리는 맨해튼 거리보다 길어질 수 있다.

## 유클리드 거리와 비교

| 구분 | 맨해튼 거리 | 유클리드 거리 |
|----|----|----|
| 식 | `|dx| + |dy|` | `sqrt(dx^2 + dy^2)` |
| 이동 형태 | 좌표축 방향 | 직선 방향 |
| 격자에서의 활용 | 4방향 이동 | 자유로운 평면 이동 |
| 제곱근 계산 | 필요 없음 | 필요함 |

맨해튼 거리는 A* 알고리즘에서 4방향 격자의 휴리스틱으로 자주 사용된다. 장애물을 무시한 최단 거리이므로 실제 남은 비용을 초과하지 않으며, 이동 비용이 동일한 경우 허용 가능한 휴리스틱이 된다.

## 여러 점 중 최대 거리

점이 `n`개일 때 모든 쌍의 거리를 직접 계산하면 `O(n^2)`이 필요하다. 2차원에서는 다음 항등식을 이용하여 최대 맨해튼 거리를 `O(n)`에 구할 수 있다.

```text
|x1 - x2| + |y1 - y2|
= max(
    |(x1 + y1) - (x2 + y2)|,
    |(x1 - y1) - (x2 - y2)|
  )
```

따라서 각 점의 `x + y`와 `x - y`의 최댓값과 최솟값만 알면 된다.

```cs
public static long MaxManhattanDistance(
    IReadOnlyList<(long X, long Y)> points)
{
    if (points.Count <= 1)
        return 0;

    long minSum = long.MaxValue;
    long maxSum = long.MinValue;
    long minDifference = long.MaxValue;
    long maxDifference = long.MinValue;

    foreach (var point in points)
    {
        long sum = point.X + point.Y;
        long difference = point.X - point.Y;

        minSum = Math.Min(minSum, sum);
        maxSum = Math.Max(maxSum, sum);
        minDifference = Math.Min(minDifference, difference);
        maxDifference = Math.Max(maxDifference, difference);
    }

    return Math.Max(
        maxSum - minSum,
        maxDifference - minDifference);
}
```

같은 원리는 더 높은 차원에서도 각 좌표의 부호 조합을 확인하는 방식으로 확장할 수 있다. 차원이 `d`라면 부호 조합의 수가 지수적으로 증가하므로, 점의 수뿐 아니라 차원의 크기도 확인해야 한다.

## 주의할 점

- `Math.Abs(int.MinValue)`는 `int`로 표현할 수 없으므로 좌표 범위가 크다면 계산 전에 `long`으로 변환한다.
- 좌표의 합과 차도 원래 좌표보다 큰 범위를 사용할 수 있다.
- 장애물이 있는 격자에서 맨해튼 거리는 실제 경로 길이가 아니라 두 점 사이 거리의 하한이다.
- 8방향 격자에서 대각선과 직선 이동 비용이 같다면 체비쇼프 거리가 더 적합하다.

## 출처 및 같이 보기

- [Taxicab geometry](https://en.wikipedia.org/wiki/Taxicab_geometry)
- [A* search algorithm](https://en.wikipedia.org/wiki/A*_search_algorithm)
- [Chebyshev distance](https://en.wikipedia.org/wiki/Chebyshev_distance)
