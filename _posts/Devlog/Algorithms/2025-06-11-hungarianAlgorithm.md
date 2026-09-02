---
title: "Assignment Problem and Hungarian Algorithm(할당 문제와 헝가리안 알고리즘)"
date: 2025-06-11
last_modified_at: 2026-09-02

toc: true
toc_sticky: true

categories:
    - algorithm
tags:
    - [algorithm, assignment, hungarian]
---

## 개요

**할당 문제(Assignment Problem)**는 `n`명의 작업자에게 서로 다른 작업을 하나씩 배정할 때 전체 비용이 최소가 되는 조합을 찾는 문제이다.

비용 행렬 `cost[i, j]`는 작업자 `i`가 작업 `j`를 수행하는 비용을 나타낸다. 각 작업자는 하나의 작업만 선택하고, 같은 작업을 두 명에게 중복으로 배정할 수 없다.

모든 순열을 확인하면 `O(n!)`, 비트마스크 동적 계획법은 `O(n * 2^n)`이 필요하다. **헝가리안 알고리즘(Hungarian Algorithm)**은 정사각 비용 행렬의 최소 비용 완전 매칭을 **O(n^3)**에 구할 수 있다.

## 핵심 아이디어

비용 행렬의 각 행이나 열에 같은 값을 더하거나 빼더라도 가능한 모든 완전 할당의 상대적인 비용 차이는 변하지 않는다.

고전적인 설명에서는 다음 과정을 반복한다.

1. 각 행의 최솟값을 그 행의 모든 원소에서 뺀다.
2. 각 열의 최솟값을 그 열의 모든 원소에서 뺀다.
3. 값이 0인 간선만 사용하여 최대 매칭을 시도한다.
4. 완전 매칭을 만들지 못하면 잠재값을 조정하여 새로운 0을 만든다.
5. 모든 작업자를 서로 다른 작업에 연결할 때까지 반복한다.

아래 구현은 같은 원리를 **잠재값(Potential)**과 증가 경로로 표현한다. `u`와 `v`는 각 행과 열의 잠재값이고, `minValue[j]`는 현재 증가 경로에서 열 `j`까지 도달하는 최소 여유 비용을 저장한다.

## 구현

아래 코드는 작업자의 수 `n`이 작업의 수 `m`보다 작거나 같은 직사각 행렬을 지원한다. 반환하는 `assignment[i]`는 작업자 `i`에 배정된 작업의 인덱스이다.

```cs
public static (long Cost, int[] Assignment) Hungarian(long[,] cost)
{
    int n = cost.GetLength(0);
    int m = cost.GetLength(1);

    if (n > m)
        throw new ArgumentException("작업의 수는 작업자의 수 이상이어야 합니다.");

    // 알고리즘 내부에서는 1부터 시작하는 인덱스를 사용한다.
    var rowPotential = new long[n + 1];
    var columnPotential = new long[m + 1];
    var matchedRow = new int[m + 1];
    var previousColumn = new int[m + 1];
    const long Infinity = long.MaxValue / 4;

    for (int row = 1; row <= n; row++)
    {
        matchedRow[0] = row;
        int column0 = 0;
        var minValue = Enumerable.Repeat(Infinity, m + 1).ToArray();
        var used = new bool[m + 1];

        do
        {
            used[column0] = true;
            int row0 = matchedRow[column0];
            long delta = Infinity;
            int column1 = 0;

            for (int column = 1; column <= m; column++)
            {
                if (used[column])
                    continue;

                long reducedCost = cost[row0 - 1, column - 1]
                    - rowPotential[row0]
                    - columnPotential[column];

                if (reducedCost < minValue[column])
                {
                    minValue[column] = reducedCost;
                    previousColumn[column] = column0;
                }

                if (minValue[column] < delta)
                {
                    delta = minValue[column];
                    column1 = column;
                }
            }

            for (int column = 0; column <= m; column++)
            {
                if (used[column])
                {
                    rowPotential[matchedRow[column]] += delta;
                    columnPotential[column] -= delta;
                }
                else
                {
                    minValue[column] -= delta;
                }
            }

            column0 = column1;
        }
        while (matchedRow[column0] != 0);

        // 찾은 증가 경로를 따라 매칭을 뒤집는다.
        do
        {
            int column1 = previousColumn[column0];
            matchedRow[column0] = matchedRow[column1];
            column0 = column1;
        }
        while (column0 != 0);
    }

    var assignment = Enumerable.Repeat(-1, n).ToArray();

    for (int column = 1; column <= m; column++)
    {
        if (matchedRow[column] != 0)
            assignment[matchedRow[column] - 1] = column - 1;
    }

    long minimumCost = 0;
    for (int row = 0; row < n; row++)
        minimumCost += cost[row, assignment[row]];

    return (minimumCost, assignment);
}
```

```cs
long[,] cost =
{
    { 9, 2, 7 },
    { 6, 4, 3 },
    { 5, 8, 1 }
};

var result = Hungarian(cost);

// result.Cost: 9
// result.Assignment: [1, 0, 2]
```

위 결과는 첫 번째 작업자에게 작업 1, 두 번째 작업자에게 작업 0, 세 번째 작업자에게 작업 2를 배정하며 전체 비용은 `2 + 6 + 1 = 9`이다.

## 시간 복잡도

작업자마다 하나의 증가 경로를 찾고, 그 과정에서 모든 작업 열을 확인한다. 작업자가 `n`, 작업이 `m`이고 `n <= m`일 때 시간 복잡도는 **O(n^2 * m)**, 공간 복잡도는 **O(n + m)**이다.

정사각 행렬에서는 시간 복잡도가 **O(n^3)**이 된다.

## 주의할 점

- 작업자의 수가 작업보다 많다면 행과 열을 바꾸거나 더미 작업을 추가하여 조건을 맞춘다.
- 최대 이익을 구하는 문제는 점수를 음수 비용으로 바꾸거나 `최댓값 - 점수` 형태의 비용으로 변환한다.
- 일부 배정을 하지 않아도 되는 문제는 배정하지 않는 비용을 가진 더미 행이나 열을 추가한다.
- 같은 최소 비용의 답이 여러 개라면 구현에 따라 그중 하나를 반환한다.
- 잠재값과 비용의 덧셈 및 뺄셈이 있으므로 값의 범위에 여유가 있는 `long`을 사용한다.

## 출처 및 같이 보기

- [Assignment problem](https://en.wikipedia.org/wiki/Assignment_problem)
- [Hungarian algorithm](https://en.wikipedia.org/wiki/Hungarian_algorithm)
- [Harold W. Kuhn - The Hungarian Method for the Assignment Problem](https://doi.org/10.1002/nav.3800020109)
- [Bipartite graph](https://en.wikipedia.org/wiki/Bipartite_graph)
