---
title: "Preprocessing(전처리)"
date: 2025-05-16
last_modified_at: 2026-09-02

toc: true
toc_sticky: true

categories:
    - algorithm
tags:
    - [algorithm, preprocess]
---

## 개요

알고리즘에서 **전처리(Preprocessing)**는 입력을 미리 계산하거나 다른 형태로 변환하여 이후 연산이나 쿼리를 더 빠르게 처리하는 방법이다.

한 번의 답을 구하는 데는 전처리 비용이 더 클 수 있지만, 같은 데이터를 대상으로 많은 쿼리를 처리한다면 전체 실행 시간을 크게 줄일 수 있다.

```text
전체 비용 = 전처리 비용 + 쿼리 수 * 쿼리 한 번의 비용
```

예를 들어 배열의 구간 합을 쿼리마다 직접 더하면 `O(Qn)`이 필요하지만, 누적합을 `O(n)`에 만들어 두면 모든 쿼리를 `O(n + Q)`에 처리할 수 있다.

## 대표적인 전처리

| 전처리 | 전처리 비용 | 이후 연산 |
|----|----|----|
| 정렬 | `O(n log n)` | 이분 탐색 `O(log n)` |
| 누적합 | `O(n)` | 구간 합 `O(1)` |
| 빈도 배열 또는 해시 테이블 | `O(n)` | 존재 여부, 개수 조회 평균 `O(1)` |
| 에라토스테네스의 체 | `O(M log log M)` | `M` 이하의 소수 여부 `O(1)` |
| 최단 거리 테이블 | 알고리즘에 따라 다름 | 여러 최단 거리 쿼리를 빠르게 처리 |

전처리는 시간을 줄이는 대신 결과를 저장할 **추가 메모리**를 사용하는 경우가 많다. 문제의 쿼리 수, 입력 변경 여부, 메모리 제한을 함께 고려해야 한다.

## 예시: 소수 판별 테이블

`0`부터 `M`까지의 수에 대해 소수인지 묻는 쿼리가 반복된다면 각 숫자마다 나누어 보지 않고, 에라토스테네스의 체로 소수 여부를 미리 계산할 수 있다.

```cs
public static bool[] BuildPrimeTable(int max)
{
    if (max < 0)
        throw new ArgumentOutOfRangeException(nameof(max));

    var isPrime = new bool[max + 1];
    Array.Fill(isPrime, true);

    isPrime[0] = false;
    if (max >= 1)
        isPrime[1] = false;

    for (int number = 2; (long)number * number <= max; number++)
    {
        if (!isPrime[number])
            continue;

        // 더 작은 배수는 앞선 소수에서 이미 제거되었다.
        for (int multiple = number * number; multiple <= max; multiple += number)
            isPrime[multiple] = false;
    }

    return isPrime;
}
```

```cs
bool[] isPrime = BuildPrimeTable(1_000_000);

bool firstAnswer = isPrime[97];
bool secondAnswer = isPrime[100];
```

전처리에 `O(M log log M)`의 시간과 `O(M)`의 공간을 사용하고, 이후 각 소수 여부 쿼리는 `O(1)`에 처리한다.

## 전처리를 판단하는 기준

다음과 같은 조건에서는 전처리를 우선 고려할 수 있다.

- 같은 입력에 대한 쿼리가 많이 주어진다.
- 입력이 쿼리 도중 변경되지 않거나 변경 횟수가 적다.
- 반복해서 계산하는 값의 범위가 미리 정해져 있다.
- 전처리 결과를 저장할 메모리가 충분하다.

반대로 쿼리가 한두 번뿐이거나 입력이 계속 바뀐다면 전처리 비용을 회수하지 못할 수 있다. 변경과 조회가 섞여 있다면 매번 전체를 다시 전처리하기보다 펜윅 트리, 세그먼트 트리, 동적 계획법의 메모이제이션처럼 변경 가능한 구조를 고려한다.

## 주의할 점

- 입력의 최댓값이 아니라 실제로 필요한 범위까지만 계산한다.
- 전처리 시간도 전체 시간 제한에 포함된다.
- 테스트 케이스마다 전처리를 반복해야 하는지, 한 번 만든 결과를 공유할 수 있는지 확인한다.
- 배열 인덱스와 정수 곱셈이 범위를 넘지 않는지 확인한다.
- 원본 데이터가 변경되면 전처리 결과가 더 이상 유효하지 않을 수 있다.

## 출처 및 같이 보기

- [Precomputation](https://en.wikipedia.org/wiki/Precomputation)
- [Sieve of Eratosthenes](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes)
- [Prefix sum](https://en.wikipedia.org/wiki/Prefix_sum)
