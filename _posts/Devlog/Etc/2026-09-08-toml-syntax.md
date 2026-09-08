---
title: "TOML 문법과 사용법 정리"
date: 2026-09-08
last_modified_at: 2026-09-08

toc: true
toc_sticky: true

categories:
    - etc
tags:
    - [toml, 문법, 설정파일, python]
---

## 개요

TOML(Tom's Obvious, Minimal Language)은 사람이 읽고 수정하기 쉽게 만든 설정 파일 형식이다. `키 = 값`을 기본으로 사용하며, 관련 설정은 테이블로 묶는다.

이 글에서는 자주 사용하는 문법과 설정 파일을 읽는 방법을 정리한다. 파일은 UTF-8 인코딩으로 저장하고, 확장자는 `.toml`을 사용한다. [TOML 공식 명세](https://toml.io/en/v1.0.0)

> 예제는 TOML 1.0 호환 문법으로 작성한다. TOML 1.1에서 추가된 문법은 마지막에 별도로 설명한다. 각 코드 블록은 독립된 예제이므로, 모두 하나의 파일에 합치면 키가 중복될 수 있다.

## 키와 값, 주석

등호(`=`) 왼쪽에 키, 오른쪽에 값을 작성한다. 문자열에는 따옴표가 필요하고, 키는 대소문자를 구분한다.

```toml
title = "설정 연습"
port = 8080
enabled = true

# 한 줄 주석
timeout = 30  # 값 뒤에도 주석을 쓸 수 있다.
color = "#ffffff"  # 따옴표 안의 #은 문자열이다.
```

따옴표 없는 키에는 영문자, 숫자, 밑줄(`_`), 하이픈(`-`)을 쓸 수 있다. 한글이나 공백 등이 포함되면 키도 따옴표로 감싼다.

```toml
app_name = "sample"
max-retries = 3
"표시 이름" = "샘플 앱"
"example.com" = "도메인 이름"
```

## 문자열

### 기본 문자열과 리터럴 문자열

큰따옴표(`"`)는 `\n`, `\t`, `\"`, `\\` 같은 이스케이프를 해석한다. 작은따옴표(`'`)는 역슬래시를 그대로 보존하므로 Windows 경로나 정규식에 편리하다.

```toml
message = "첫 번째 줄\n두 번째 줄"
quote = "그는 \"안녕하세요\"라고 말했다."
escaped_path = "C:\\workspace\\sample"

literal_path = 'C:\workspace\sample'
pattern = '^\d{4}-\d{2}-\d{2}$'
```

`escaped_path`와 `literal_path`는 같은 값이다. 한 줄 리터럴 문자열 안에 작은따옴표를 넣어야 한다면 큰따옴표 문자열을 사용하면 된다.

### 여러 줄 문자열

따옴표 세 개로 감싸면 여러 줄을 작성할 수 있다. 여는 따옴표 바로 다음의 첫 줄바꿈은 제거된다. `"""`는 이스케이프를 해석하고, `'''`는 그대로 보존한다.

```toml
description = """
TOML 문법을 정리한다.
설정 파일 예제를 함께 작성한다."""

paths = '''
C:\workspace\sample
C:\workspace\archive'''
```

## 숫자와 불리언

정수와 실수, 지수 표기법을 지원한다. 큰 숫자는 숫자 사이에 밑줄을 넣어 구분할 수 있다. 불리언은 소문자 `true`, `false`로 작성한다.

```toml
count = 12
offset = -5
max_size = 1_000_000
ratio = 0.75
timeout = 2.5e2  # 250.0

hex_value = 0xFF   # 255
octal_value = 0o755  # 493
binary_value = 0b1010  # 10

enabled = true
debug = false
```

10진수 정수에 `012`처럼 불필요한 앞자리 0을 붙일 수 없다. 실수는 `.5`, `5.` 대신 `0.5`, `5.0`처럼 작성한다.

## 날짜와 시간

날짜와 시간은 따옴표 없이 작성하면 전용 자료형으로 처리된다. 문자열로 유지하려면 따옴표를 붙인다.

```toml
published_at = 2026-09-08T09:00:00+09:00  # UTC 오프셋 포함
utc_time = 2026-09-08T00:00:00Z          # UTC
local_datetime = 2026-09-08T09:00:00     # 오프셋 없는 날짜와 시간
release_date = 2026-09-08                # 날짜
start_time = 09:00:00                   # 시간
date_text = "2026-09-08"                 # 문자열
```

오프셋 없는 날짜와 시간에는 시간대 정보가 없다. 예를 들어 `local_datetime`을 한국 시간으로 사용할지는 프로그램에서 정해야 한다. [날짜와 시간 명세](https://toml.io/en/v1.0.0#offset-date-time)

## 배열

대괄호(`[]`) 안에 값을 쉼표로 구분한다. 여러 줄 작성과 마지막 항목 뒤의 쉼표도 허용된다.

```toml
ports = [8080, 8081]
tags = ["toml", "config"]
matrix = [[1, 2], [3, 4]]
mixed = [1, "two", true]

languages = [
    "ko",  # 한국어
    "en",  # 영어
]
```

서로 다른 자료형을 섞는 것도 문법상 가능하지만, 실제 설정 항목이 어떤 자료형을 허용하는지는 사용하는 프로그램에 따라 다르다. [배열 명세](https://toml.io/en/v1.1.0#array)

## 테이블과 중첩 구조

### 테이블

테이블은 관련 키와 값을 묶는 구조다. `[이름]`을 선언하면 다음 테이블 헤더 또는 파일 끝까지의 키가 해당 테이블에 속한다.

```toml
app_name = "sample"

[server]
host = "127.0.0.1"
port = 8080

[logging]
level = "info"
```

`app_name`은 최상위 키이고, `host`와 `port`는 `server` 테이블의 키다. 빈 줄이나 들여쓰기로 테이블이 끝나지는 않으므로 최상위 설정은 첫 테이블 헤더 앞에 작성한다. [테이블 명세](https://toml.io/en/v1.1.0#table)

### 중첩 테이블과 점으로 구분한 키

점(`.`)으로 하위 구조를 표현한다.

```toml
[server.ssl]
enabled = true
port = 8443
```

위 구조는 다음처럼 점으로 구분한 키로도 표현할 수 있다. 두 예제 중 한 가지 방식만 사용한다.

```toml
server.ssl.enabled = true
server.ssl.port = 8443
```

두 경우 모두 `server` 아래 `ssl` 테이블이 만들어진다. 반면 `"server.ssl.port" = 8443`처럼 키 전체를 따옴표로 감싸면 점까지 포함한 하나의 키가 된다.

### 인라인 테이블

짧은 설정 묶음은 중괄호(`{}`) 안에 작성할 수 있다.

```toml
author = { name = "Pioh", role = "developer" }
window = { width = 1280, height = 720 }
```

인라인 테이블은 중괄호 안에서 정의를 끝내야 한다. 위 예제 뒤에 `author.email = "pioh@example.com"`을 덧붙이는 방식으로 확장할 수 없다. TOML 1.0과의 호환성을 유지하려면 한 줄로 쓰고 마지막 쉼표를 생략한다. [인라인 테이블 명세](https://toml.io/en/v1.0.0#inline-table)

## 테이블 배열

객체 여러 개를 목록으로 만들 때는 이중 대괄호(`[[이름]]`)를 사용한다. 같은 헤더를 쓸 때마다 새 항목이 추가된다.

```toml
[[menus]]
name = "홈"
path = "/"

[[menus]]
name = "글 목록"
path = "/posts/"
```

JSON으로 표현하면 다음 구조다. [테이블 배열 명세](https://toml.io/en/v1.1.0#array-of-tables)

```json
{
  "menus": [
    { "name": "홈", "path": "/" },
    { "name": "글 목록", "path": "/posts/" }
  ]
}
```

## 설정 파일 사용 예제

### config.toml 작성

다음은 가상의 앱 설정이다. 키 이름과 의미는 예제를 위해 정한 것으로, 실제 프로그램에서는 해당 프로그램이 지원하는 설정 항목을 사용해야 한다.

```toml
app_name = "TOML Demo"
debug = false
tags = ["blog", "example"]

[server]
host = "127.0.0.1"
port = 8080

[logging]
level = "info"

[[menus]]
name = "홈"
path = "/"

[[menus]]
name = "글 목록"
path = "/posts/"
```

파일을 저장하는 것만으로 앱에 설정이 적용되지는 않는다. 프로그램에서 파일을 읽고 필요한 값에 접근하는 과정이 필요하다.

### Python에서 읽기

Python 3.11 이상에서는 표준 라이브러리 `tomllib`을 사용할 수 있다. `load()`에는 바이너리 읽기 모드(`rb`)로 연 파일을 전달한다. [Python tomllib 문서](https://docs.python.org/3.11/library/tomllib.html)

아래 코드를 `main.py`로 저장하고, 같은 폴더에 앞의 `config.toml`을 둔다.

```python
from pathlib import Path
import tomllib

config_path = Path(__file__).with_name("config.toml")

with config_path.open("rb") as file:
    config = tomllib.load(file)

print(config["app_name"])
print(config["server"]["port"])
print(config["menus"][0]["name"])
print(config.get("timeout", 30))  # 키가 없으면 기본값 사용
```

해당 폴더에서 실행한다.

```bash
python main.py
```

출력 결과는 다음과 같다.

```text
TOML Demo
8080
홈
30
```

테이블은 `dict`, 배열은 `list`로 변환된다. 문자열을 직접 읽을 때는 `tomllib.loads()`를 사용한다. 문법 오류가 있으면 `tomllib.TOMLDecodeError`가 발생하며, `tomllib` 자체는 TOML 파일 쓰기를 지원하지 않는다. [tomllib 함수와 자료형 변환](https://docs.python.org/3.11/library/tomllib.html)

## 자주 실수하는 부분

| 잘못된 작성 또는 오해 | 올바른 사용법 |
|:---|:---|
| `name = sample` | 문자열 값은 `name = "sample"`처럼 따옴표로 감싼다. |
| `enabled = True` | 불리언은 소문자 `true`, `false`를 사용한다. |
| `value = null` 또는 `value =` | TOML에는 null 자료형이 없다. 필요한 경우 키를 생략하고 프로그램에서 기본값을 처리한다. |
| 동일한 범위에서 `port`를 두 번 정의 | 기존 값을 덮어쓰는 것이 아니라 오류가 된다. |
| `[server]`를 두 번 선언 | 일반 테이블은 중복 선언할 수 없다. 목록이 필요하면 `[[servers]]`를 사용한다. |
| `path = "C:\workspace\sample"` | `path = 'C:\workspace\sample'`처럼 리터럴 문자열을 사용한다. |
| 빈 줄 다음 키는 최상위에 속한다고 생각 | 다음 헤더가 나올 때까지 현재 테이블에 속한다. |

키 생략과 빈 문자열(`""`)은 다르다. 기본값 적용 여부 등 실제 동작은 설정을 읽는 프로그램에서 결정한다. [키와 값 명세](https://toml.io/en/v1.0.0#keyvalue-pair)

## TOML 1.1에서 달라진 점

TOML 1.1에서는 인라인 테이블의 여러 줄 작성과 마지막 쉼표, 문자열의 `\e` 및 `\xHH` 이스케이프, 시간의 초 생략 등이 추가되었다. 파서마다 지원 버전이 다를 수 있으므로, 해당 문법을 사용하기 전에 사용하는 도구의 지원 범위를 확인한다. [TOML 변경 이력](https://github.com/toml-lang/toml/blob/main/CHANGELOG.md)

## 출처 및 같이 보기

- [TOML 1.0 공식 명세](https://toml.io/en/v1.0.0)
- [TOML 1.1 공식 명세](https://toml.io/en/v1.1.0)
- [Python 3.11 tomllib 문서](https://docs.python.org/3.11/library/tomllib.html)
- [TOML 버전별 변경 이력](https://github.com/toml-lang/toml/blob/main/CHANGELOG.md)
