---
title: 컨테이너- Network Namespace
date: 2023-07-18
categories: [container, network]
tags: [container, namespace, linux]     # TAG names should always be lowercase
---

- umount로 했던 다른 네임스페이스들과는 달리 ip로 시작하는 명령어로 조작, 관리

- ip link

  ```shell
  $ ip link
  ```

## 1. host와 container를 연결하는 가상 인터페이스

- veth(Virtual Ethernet Device)
	
    - 리눅스의 가상 이더넷 인터페이스
    
    - 한개의 쌍으로 만들어지고 네트워크 네임스페이스들을 연결해주거나 물리 디바이스와 다른 네트워크 네임스페이스를 연결하는 용도로 사용
    
    - 활성화하지 않은 상태로 생성
    
    ```shell
    #{veth0_name}, {veth1_name}을 치환하여 사용
    $ sudo ip link add {veth0_name} type veth peer name {veth1_name}
    ```
    ```shell
    #위 명령어를 실행하면 {veth0_name}, {veth1_name}이름을 가진 것이 나타남
    $ sudo ip -br link
    ```
    
## 2. network namespace

```shell
#네임스페이스 생성
$ ip netns add {network_namespace}

$ ip netns list
{network_namespace}

#네임스페이스 삭제
#-all option은 모든 네트워크 네임스페이스를 제거
$ ip [-all] netns delete {nework_namespace}

#네임스페이스가 추가되거나 삭제되면 알리기
$ ip netns monitor
```

- 이를 사용하면, /var/run/netns/ 하위에 네트워크 네임스페이스가 생성됨

- 해당 파일 디스크립터를 열면 해당 네임스페이스가 작동함(setns system call을 이용 바꿀 수 있음) 

- 각 네임스페잇별 설정은 /etc/netns 하위에 저장하면 됨

  - /etc 하위에 있는 다른 설정을 /etc/netns/{ns_name} 하위에 해당 파일을 생성하고 쓰면 됨
  
  ### 1) 가상 인터페이스간 연결

     ```shell
     #veth에 주소 부여
     $ ip a add {ip_addr/subnet_mask} {dev|group} {veth_name}

     #veth를 network namesapce에 할당
     $ ip link set {veth_name} netns {netns_name}

     #namespace에서 명령 실행
     $ ip netns exec {netns_name} {command}
     ```

     - 이러한 방식으로 주소를 할당하고 namespace와 연결하고 ping을 보내도 되질 않는데, 이는 veth가 활성화 되지 않기 때문
     ```shell
     # 여기에선 dev를 할당
     $ ip link set {dev | group} {veth_name} up
     ```
  ### 2) 브리지 네트워크
  
     ```shell
      # bridge 생성
      $ ip link add {bridge_name} type bridge
      $ ip link set {bridge_name} up

      # bridge에 veth 연결
      $ ip link set {bridge_veth} master {bridge}
      $ ip link set dev {bridge_veth} up
     ```

     - bridge 네트워크시 container의 veth 디바이스 외에도 loopback 디바이스도 활성화 해야함

     - ping시 컨테이너간 데이터를 주고 받지 못하는 경우가 있음

       - 이는 리눅스 방화벽 때문이므로 iptables설정을 바꿔주면 해결

       ```shell
       #확인
       $iptables -L | grep FORWARD
       Chain FORWARD(policy DROP)
       #변경
       $iptables -policy FORWARD ACCEPT
       ```

  - host에서 container로 보낼 때도 가지 못하는 경우가 있음
    - 이는 routing table에 bridge 정보가 없기 때문
    ```shell
    # route table 확인
    $ route

    # bridge 관련 정보 추가
    # +는 255까지, -는 0까지
    $ ip addr add 10.201.0.1/24 brd + dev br0
    ```

      
  ### 3) NAT구성

    - 컨테이너의 인터넷 연결을 위해 필요

    - default 테이블을 추가해줘야 하는데, 별도 라우팅 규칙이 없을 때, 라우팅 처리

  ```shell
   $ ip netns exec {netns} ip route add default via {IPv4 address}
   # IP FORWARD 기능 활성화
   $ sysctl -w net.ipv4.ip_forward=1
   # 혹은
   $ echo "1" > /proc/sys/net/ipv4/ip_forward
   # NAT 규칙 추가
   $ iptables -t nat -A POSTROUTING -s {IPv4 addr(외부망과 연결 되는 곳)} -j MASQUERADE
   # DNS
   $ mkdir -p /etc/netns/{netns}/
   $ echo 'nameserver 8.8.8.8' > /etc/netns/{netns}/resolv.conf

   ```
   
   - NAT 규칙 추가
   - -t
   	: 테이블선택 옵션
   	: Filter, NAT, Mangle 3가지 존재
   - -A
   	: 체인에 정책을 차례대로 추가하는 옵션
    : 먼저 입력된 옵션일 수록 먼저 실행되므로 순서가 중요
    : POSTROUTING은 NAT 테이블 체인 중 하나로 라우팅 이후를 의미
   - -s
   	: 출발지 주소를 정해줌
    : route table에 10.201.10.0이면 이거 넣으면됨
   - -j
   	: 매칭 옵션에 맞는 패킷일 경우 어떤 정책으로 실행할지 결정
    : MASQUERADE( NAPT:Network Address Port Translation )
   
   [참고자료1](https://www.44bits.io/ko/post/container-network-2-ip-command-and-network-namespace)
   
   [참고자료2](https://wariua.github.io/man-pages-ko/ip-netns%288%29/)
   
   [참고자료3](https://blog.naver.com/croshine/50100828808)