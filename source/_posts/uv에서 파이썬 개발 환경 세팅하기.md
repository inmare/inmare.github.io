---
title: uv에서 파이썬 개발 환경 세팅하기
date: 2025-07-28T01:31:30
categories: [Python]
tags: [Python, uv]
---

- [설치 방법](#설치-방법)
- [사용 방법](#사용-방법)
  - [초기화](#초기화)
  - [uv에 사용할 python 버전 지정](#uv에-사용할-python-버전-지정)
  - [라이브러리 추가/삭제](#라이브러리-추가삭제)
  - [uv로 실행](#uv로-실행)
- [기타 명령어](#기타-명령어)
  - [requirements.txt 생성](#requirementstxt-생성)
  - [중간에 다른 버전 Python으로 바꾸기](#중간에-다른-버전-python으로-바꾸기)
- [vscode에서 실행하는 법](#vscode에서-실행하는-법)
- [참고](#참고)

# 설치 방법

pip로 설치

```bash
pip install uv
```

# 사용 방법

## 초기화

```bash
uv init
```

## uv에 사용할 python 버전 지정

```bash
uv python install 3.13
```

## 라이브러리 추가/삭제

```bash
uv add ruff
uv remove ruff
```

## uv로 실행

```bash
# commands에 실행할 명령어를 넣음
uv run commands
```

# 기타 명령어

## requirements.txt 생성

```bash
uv export -o requirements.txt
```

## 중간에 다른 버전 Python으로 바꾸기

pyproject.toml에서 버전 변경

```toml
requires-python = ">=3.12"
```

그 후 `.python-version` 파일 수정 명령어 실행

```bash
uv python pin 3.12
```

# vscode에서 실행하는 법

uv는 파일을 실행할 때 자동으로 가상환경을 만드므로, 명령어를 통해 실행한 다음 vscode에서 해당 가상환경을 지정하면 된다.

# 참고

- https://docs.astral.sh/uv/getting-started/installation/#standalone-installer
- https://rudaks.tistory.com/entry/python%EC%9D%98-uv-%EC%82%AC%EC%9A%A9%EB%B2%95
- https://pydevtools.com/handbook/how-to/how-to-change-the-python-version-of-a-uv-project/
