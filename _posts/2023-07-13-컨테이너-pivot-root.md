---
title: 컨테이너 - pivot root
date: 2023-07-13
categories: [container, pivot root]
tags: [container, pivot root, linux]     # TAG names should always be lowercase
---

## 1. Synopsis
```C
int pivot_root(const char* new_root, const char* put_old);
```
```shell
$ pivot_root {new_root} {put_old}
```

## 2. Descrption
- 호출한 마운트 네임스페이스에서 root 마운트를 put_old로 옮기고, new_root를 새 루트 마운트로 변경.
- pivot_root는 새로운 root filesystem이 될 위치에서 사용해야함
- put_old 경로에 기존 root filesystem의 파일이 있음
