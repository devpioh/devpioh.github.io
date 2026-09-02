---
title: "균등한 확률의 주사위(Uniform Dice)"
date: 2022-07-22
last_modified_at: 2026-09-02

toc: true
toc_sticky: true

categories:
    - algorithm
tags:
    - [algorithm, probability, uniform, rejection sampling]
---

## 개요

`s`면체 주사위가 **균등한 분포(Uniform Distribution)**를 가진다는 것은 각 눈이 나올 확률이 모두 `1 / s`로 같다는 뜻이다.

C#의 `Random.Next(min, max)`처럼 원하는 범위의 값을 균등하게 만들어 주는 함수를 사용하면 주사위는 간단히 구현할 수 있다.

```cs
public static int RollDice(Random random, int sides)
{
    if (sides <= 0)
        throw new ArgumentOutOfRangeException(nameof(sides));

    // Random.Next(sides)는 0부터 sides - 1까지 반환한다.
    return random.Next(sides) + 1;
}
```

하지만 제한된 범위의 난수만 주어지고 이를 다른 범위로 바꾸어야 한다면 단순한 나머지 연산은 확률을 편향시킬 수 있다.

## 나머지 연산의 편향

`0`부터 `7`까지 균등하게 나오는 난수로 6면체 주사위를 만든다고 가정한다.

```cs
int dice = randomValue % 6 + 1;
```

이때 입력 `0`과 `6`은 모두 주사위 눈 `1`이 되고, 입력 `1`과 `7`은 모두 눈 `2`가 된다. 반면 나머지 눈에 대응하는 입력은 하나뿐이다.

| 주사위 눈 | 입력 | 확률 |
|----|----|----|
| 1 | 0, 6 | 2 / 8 |
| 2 | 1, 7 | 2 / 8 |
| 3 ~ 6 | 각각 하나 | 1 / 8 |

원본 범위의 크기가 주사위 면의 수로 나누어떨어지지 않아서 생기는 **Modulo Bias**이다.

## 거부 표본 추출(Rejection Sampling)

편향을 없애려면 모든 결과에 같은 개수의 입력을 배정하고 남는 입력은 버린 뒤 다시 뽑는다.

원본 범위가 `0`부터 `sourceRange - 1`이라면 다음과 같이 사용할 수 있는 가장 큰 구간을 구한다.

```text
limit = sourceRange - (sourceRange % faces)
```

`limit` 이상인 값은 거부하고, `limit`보다 작은 값만 나머지 연산으로 변환한다.

```cs
// next는 [0, sourceRange) 범위의 균등한 정수를 반환해야 한다.
public static int RollUniform(
    Func<int> next,
    int sourceRange,
    int faces)
{
    if (faces <= 0 || sourceRange < faces)
        throw new ArgumentOutOfRangeException();

    int limit = sourceRange - sourceRange % faces;
    int value;

    do
    {
        value = next();

        if (value < 0 || value >= sourceRange)
            throw new InvalidOperationException("난수가 약속한 범위를 벗어났습니다.");
    }
    while (value >= limit);

    return value % faces + 1;
}
```

```cs
var random = new Random();

// 0부터 7까지의 균등한 난수만 사용하여 6면체 주사위를 만든다.
int dice = RollUniform(() => random.Next(8), 8, 6);
```

위 예시에서는 `6`과 `7`을 거부한다. 허용한 여섯 입력이 주사위의 여섯 눈에 하나씩 대응하므로 결과도 균등하다.

## 주의할 점

- 원본 난수가 편향되어 있다면 거부 표본 추출만으로 결과가 자동으로 균등해지지는 않는다.
- 반복문의 종료 횟수에는 상한이 없지만, 한 번에 성공할 확률이 `limit / sourceRange`이므로 평균적으로는 빠르게 끝난다.
- 게임 플레이용 의사 난수와 보안용 난수는 목적이 다르다. 보안이 필요하면 `RandomNumberGenerator.GetInt32`와 같은 암호학적 난수 생성기를 사용한다.

## 출처 및 같이 보기

- [Microsoft Learn - Random.Next Method](https://learn.microsoft.com/dotnet/api/system.random.next)
- [Microsoft Learn - RandomNumberGenerator.GetInt32 Method](https://learn.microsoft.com/dotnet/api/system.security.cryptography.randomnumbergenerator.getint32)
- [Rejection sampling](https://en.wikipedia.org/wiki/Rejection_sampling)
