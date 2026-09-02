---
title: "배열 섞기(Shuffle Array) - Fisher-Yates Shuffle"
date: 2022-07-21
last_modified_at: 2026-09-02

toc: true
toc_sticky: true

categories:
    - algorithm
tags:
    - [algorithm, shuffle, Fisher-Yates]
---

## 개요

배열을 섞는다는 것은 원소의 순서를 무작위로 재배치하여 하나의 **순열(Permutation)**을 만드는 것이다.

원소가 `n`개라면 가능한 순열은 `n!`개이며, 공정한 셔플은 각 순열이 모두 `1 / n!`의 동일한 확률로 선택되어야 한다.

이를 만족하는 대표적인 방법이 **Fisher-Yates Shuffle**이다. 배열을 한 번만 순회하므로 시간 복잡도는 **O(n)**이고, 배열 내부에서 원소를 교환하면 추가 공간은 **O(1)**이다.

## 알고리즘

배열의 마지막 위치부터 앞으로 이동하며 다음 과정을 반복한다.

1. 현재 인덱스를 `i`라고 한다.
2. `0`부터 `i`까지의 인덱스 중 하나인 `j`를 균등한 확률로 선택한다.
3. `array[i]`와 `array[j]`를 교환한다.
4. `i`가 `1`이 될 때까지 반복한다.

각 단계에서 아직 확정되지 않은 원소 중 하나를 현재 위치에 배치하기 때문에 모든 순열이 같은 확률로 만들어진다.

> `j`의 범위에는 반드시 `i`가 포함되어야 한다. 자기 자신과 교환하는 경우도 있어야 현재 위치에 있던 원소가 그대로 남을 확률까지 공정해진다.

## 구현

```cs
public static void Shuffle<T>(IList<T> list, Random random)
{
    for (int i = list.Count - 1; i > 0; i--)
    {
        // Random.Next의 최댓값은 범위에 포함되지 않으므로 [0, i]가 된다.
        int j = random.Next(i + 1);

        T temp = list[i];
        list[i] = list[j];
        list[j] = temp;
    }
}
```

```cs
var numbers = new List<int> { 1, 2, 3, 4, 5 };
var random = new Random();

Shuffle(numbers, random);
```

## 주의할 점

- `OrderBy(_ => random.Next())`처럼 임의의 키로 정렬하면 키 충돌과 정렬 방식의 영향으로 모든 순열의 확률이 같다고 보장하기 어렵다.
- 매 반복마다 전체 배열에서 교환 대상을 고르면 이미 확정한 위치가 다시 변경되어 결과가 편향될 수 있다.
- 셔플을 여러 번 반복한다고 편향된 알고리즘이 공정해지는 것은 아니다.
- 같은 시드(seed)의 `Random`은 같은 순서를 생성한다. 재현 가능한 테스트에는 유용하지만 보안 목적에는 적합하지 않다.

## 출처 및 같이 보기

- [Fisher-Yates shuffle](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle)
- [Microsoft Learn - Random.Next Method](https://learn.microsoft.com/dotnet/api/system.random.next)
