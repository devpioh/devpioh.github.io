---
title: "맥북에서 Homebrew와 Conda 업데이트 방법"
date: 2026-09-16
last_modified_at: 2026-09-16

toc: true
toc_sticky: true

categories:
    - etc
tags:
    - [mac, homebrew, brew, conda]
---

## 개요

맥북에서 개발 환경을 구성할 때 Homebrew와 Conda를 자주 사용한다. 두 도구 모두 패키지를 설치하고 업데이트할 수 있지만 관리하는 대상과 업데이트 방식은 서로 다르다.

Homebrew는 `update`와 `upgrade`의 역할이 구분되어 있고, Conda는 Conda 자체와 각 가상 환경의 패키지를 따로 업데이트한다. 이 글에서는 두 도구의 버전을 확인하고 안전하게 업데이트하는 방법을 정리한다.

> Homebrew는 macOS에 설치한 명령줄 도구와 애플리케이션을 관리하고, Conda는 프로젝트별 Python과 라이브러리를 가상 환경 단위로 관리한다.

## Homebrew의 update와 upgrade

Homebrew에서 `update`와 `upgrade`는 이름이 비슷하지만 갱신하는 대상이 다르다.

| 명령어 | 갱신 대상 |
| :--- | :--- |
| `brew update` | Homebrew와 formula, cask의 최신 정보 |
| `brew upgrade` | Homebrew로 설치한 실제 패키지 |

`brew update`만 실행하면 설치 가능한 버전 정보는 최신 상태가 되지만, 이미 설치된 패키지의 버전은 바뀌지 않는다. 설치된 패키지까지 올리려면 이어서 `brew upgrade`를 실행해야 한다.

## Homebrew 업데이트 방법

먼저 현재 설치된 Homebrew의 버전을 확인한다.

```bash
brew --version
```

Homebrew와 패키지 정보를 최신 상태로 갱신한다.

```bash
brew update
```

업데이트할 수 있는 패키지를 확인한다.

```bash
brew outdated
```

목록을 확인한 다음 설치된 모든 패키지를 업데이트한다.

```bash
brew upgrade
```

특정 패키지만 업데이트하려면 이름을 함께 입력한다.

```bash
brew upgrade [패키지 이름]
```

예를 들어 `wget`만 업데이트한다.

```bash
brew upgrade wget
```

Cask로 설치한 애플리케이션을 명확하게 지정하려면 `--cask` 옵션을 사용한다.

```bash
brew upgrade --cask [애플리케이션 이름]
```

### 이전 버전과 캐시 정리

업데이트가 끝난 뒤 오래된 패키지 버전과 내려받은 파일을 정리할 수 있다.

```bash
brew cleanup
```

삭제될 항목만 미리 확인하려면 `--dry-run` 옵션을 사용한다.

```bash
brew cleanup --dry-run
```

Homebrew는 패키지를 업그레이드할 때 이전 버전을 자동으로 정리하고, 일정 주기마다 전체 정리도 수행한다. 디스크 공간을 확인하거나 남아 있는 파일을 직접 정리할 때만 별도로 실행해도 된다.

업데이트 후 경고나 문제가 발생하면 현재 설치 상태를 검사한다.

```bash
brew doctor
```

## Conda 업데이트 방법

Conda는 패키지 관리 도구인 Conda 자체와 가상 환경에 설치된 패키지를 구분해서 업데이트한다. 한 환경의 패키지를 업데이트해도 다른 환경에는 영향을 주지 않는다.

먼저 Conda 버전과 생성된 가상 환경을 확인한다. 현재 활성화된 환경은 환경 목록에서 별표(`*`)로 표시된다.

```bash
conda --version
conda info --envs
```

### Conda 자체 업데이트

Conda 자체는 일반적으로 `base` 환경에 설치되어 있다. 현재 활성화된 환경과 관계없이 `base`를 지정하여 업데이트한다.

```bash
conda update --name base conda
```

Conda가 변경할 패키지와 버전을 계산한 뒤 설치 계획을 출력한다. 내용을 확인하고 계속 진행하려면 `y`를 입력한다.

### 가상 환경의 패키지 업데이트

업데이트하려는 가상 환경을 활성화한다.

```bash
conda activate [환경 이름]
```

특정 패키지만 업데이트하려면 패키지 이름을 입력한다.

```bash
conda update [패키지 이름]
```

예를 들어 현재 환경의 `numpy`를 업데이트한다.

```bash
conda update numpy
```

현재 환경에 설치된 모든 패키지를 한 번에 업데이트할 수도 있다.

```bash
conda update --all
```

환경을 활성화하지 않고 이름을 직접 지정하려면 `--name` 옵션을 사용한다.

```bash
conda update --name [환경 이름] --all
```

Conda는 환경 안의 다른 패키지와 호환되는 범위에서 요청한 패키지의 버전을 선택한다. 따라서 `--all`을 사용해도 모든 패키지가 각각의 가장 최신 버전으로 바뀌는 것은 아니다.

### environment.yml 기준으로 업데이트

프로젝트에서 `environment.yml` 파일을 사용한다면 파일에 정의된 내용에 맞춰 환경을 업데이트할 수 있다.

```bash
conda env update --name [환경 이름] --file environment.yml --prune
```

`--prune`은 파일에서 제거되어 더 이상 필요하지 않은 패키지를 가상 환경에서도 삭제한다. 기존 패키지를 유지해야 한다면 이 옵션을 생략한다.

## 업데이트할 때 주의할 점

- Homebrew 명령어는 일반 사용자 권한으로 실행하고 `sudo`를 붙이지 않는다.
- `brew upgrade`는 업데이트 가능한 여러 패키지를 함께 변경할 수 있다. 필요한 패키지만 올리려면 이름을 지정한다.
- Conda 명령어를 실행하기 전에 `conda info --envs`로 현재 활성화된 환경을 확인한다.
- `conda update --all`은 의존성 해결 과정에서 여러 패키지를 업데이트하거나 버전을 조정할 수 있으므로 설치 계획을 확인한다.
- 버전이 고정된 프로젝트는 `environment.yml`이나 별도의 환경 파일을 먼저 확인한다.

## 명령어 정리

Homebrew의 전체 업데이트 과정은 다음과 같다.

```bash
brew update
brew outdated
brew upgrade
brew cleanup
```

Conda 자체와 특정 가상 환경의 전체 패키지를 업데이트한다.

```bash
conda update --name base conda
conda update --name [환경 이름] --all
```

## 출처 및 같이 보기

- [Homebrew FAQ - How do I update my local packages?](https://docs.brew.sh/FAQ#how-do-i-update-my-local-packages) : Homebrew와 설치된 패키지를 업데이트하는 기본 순서
- [Homebrew Manpage](https://docs.brew.sh/Manpage) : `update`, `upgrade`, `cleanup` 명령어와 옵션 설명
- [Conda update](https://docs.conda.io/projects/conda/en/stable/commands/update.html) : Conda 패키지 업데이트 명령어와 옵션 설명
- [Conda 환경 관리](https://docs.conda.io/projects/conda/en/stable/user-guide/tasks/manage-environments.html#updating-an-environment) : `environment.yml`을 이용한 가상 환경 업데이트 방법
