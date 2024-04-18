---
title: 컨테이너 - linux namespace
date: 2023-07-12
categories: [container, namespace]
tags: [container, namespace, linux]     # TAG names should always be lowercase
---

## 1. Description
- 전역 시스템 자원을 네임스페이스 안의 process들에게 독립적인 공간이 있는 것처럼 만드는 것
- 같은 네임스페이스에서는 변경사항이 보이지만, 그 외 process들에게는 보이지 않음
- 자식 process는 부모 process의 네임스페이스를 상속받음.


## 2. Type

|namespace|description|
|---------|-----------|
|User|process 각각의 UID, GID정보를 격리|
|IPC|IPC를 격리하여 외부 process의 접근이나 제어를 막음|
|Time|시간을 격리|
|cgroup|process는 /proc/self/cgroup에 가상화된 crgroup 마운트를 갖음|

### 1) PID 네임스페이스
- process의 id를 격리하는 네임스페이스
- PID 네임스페이스를 격리하면, init process인 1로 시작함
- 분리된 새로운 네임스페이스는 1로 시작하지만, default 네임스페이스의 경우에는 다른 값으로 봄
- init process의 경우 kernel에서 주는 것을 각 process에 할당하는(?) 역할을 수행하는데, 격리된 1번 process의 경우에도 init process에서 온 것을 하위 process에 주는 역할을 수행함.

### 2) Network 네임스페이스
- 각종 네트워크 자원을 격리
- ip 명령어로 조작할 수 있음
- unshare 명령어가 아닌 ip 명령어로 실행함


[추가설명](https://velog.io/@seongho9276/%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-Network-namespace)
    
### 3) UTS 네임스페이스
- 이름을 분리하는 네임스페이스
- 호스트 네임을 변경시 사용
``` shell
$ unshare --uts={path} {hostname}
```
### 4) 마운트 네임스페이스
- file system을 마운트시 root와 분리해줌
```shell
linux$ sudo unshare --mount /bin/sh
#if memory device => none
root# mount -t tmpfs {device} {mount_path} 
```
### 5) cgroup

### 6) IPC

### 7) User

## 3. /proc/{pid}/ns/
- 각 process마다 '/proc/{pid}/ns/'가 존재하고, 이를 통해 파일 핸들을 반환한다.
- pid에 $$를 입력하면 현재 process의 pid를 넣어줌.
- 각 namespace에 해당하는 i-node 값을 가지고 있음.
- i-node값이 같으면, 같은 네임스페이스
![proc+ns](https://velog.velcdn.com/images/seongho9276/post/e5e8dc68-2742-4e06-aa4d-5b0e1254e252/image.PNG)


## 4. /proc/sys/user/
- 각 유형의 네임스페이스들이 총 몇개까지 만들 수 있는지 보여줌
- privileged process가 각 파일의 값을 변경 할 수 있음.
- 제한의 사용자별로 다름



## 5. 네임스페이스의 수명
- 해당 네임스페이스의 마지막 process가 종료되거나, 해당 네임스페이스를 떠날 때 자동으로 해체됨
- 다른 사유로 process가 없더라도 존재하도록 할 수 있음.
