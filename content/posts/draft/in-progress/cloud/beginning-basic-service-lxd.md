---
title: "[OpenStack기반 Private Cloud 구축기] 2. 기본 서비스 구성 - LXD"
date: 2020-05-24T13:05:10+09:00
comments: true
draft: true
lightgallery: true
toc: 
  auto: false
images:
categories:
  - Cloud
  - OpenStack
series:
  - OpenStack기반 Private Cloud 구축기
tags:
  - linux
  - cloud
  - openstack
---

## 시작하면서...
이전 글에서 정리를 하였듯 OpenStack의 각 서비스는 데이터의 저장 및 전송 등을 위해 필요한 서비스들이 필요하다.  
이번편에서는 해당 서비스들을 Controller Node에 설치를 하겠다.  

## LXD
일반적으로 모든 서비스를 그냥 설치를 해도 되지만 개인적으로 격리를 시켜 운용하는것을 더 선호하여 LXD를 사용해 각 서비스들을 격리하도록 하겠다.  
<br />

### LXD 설치
먼저 LXD를 사용하기 위하여 설치해야 한다.  
만약 기존에 LXD가 있다면 그대로 사용을 해도 괜찮다.  

Ubuntu Server 18.04.4 기준으로는 3.0.3이 설치가 되어 있었고, apt상 3.0.3이 최신이였다.  
```bash
$ sudo lxd version 
  3.0.3
$ sudo lxc version
  If this is your first time running LXD on this machine, you should also run: lxd init
  To start your first container, try: lxc launch ubuntu:18.04

  Client version: 3.0.3
  Server version: 3.0.3
$ sudo apt update
$ apt show lxd
  Package: lxd
  Version: 3.0.3-0ubuntu1~18.04.1
  ...

```
<br />
최신 버전을 사용할 겸, snap을 더 선호하여 기존의 lxd를 제거 하고 snap으로 설치를 진행한다.  
```bash
$ sudo apt remove --purge lxd lxd-client
$ sudo snap install lxd
  lxd 4.1 from Canonical✓ installed
$ sudo lxd version 
  4.1
$ sudo lxc version
  If this is your first time running LXD on this machine, you should also run: lxd init
  To start your first instance, try: lxc launch ubuntu:18.04

  Client version: 4.1
  Server version: 4.1
```
<br />

### LXD 초기화
LXD의 설치가 완료 되었으면 컨테이너를 생성해보자.   
```bash
$ lxc launch ubuntu:18.04 test
  Creating test
  Error: Failed instance creation: No storage pool found. Please create a new storage pool
```
LXD의 초기 설정을 잡아 준 적이 없기 때문에 위와 같이 storage pool이 없어 생성이 안된다고 애러가 발생할 것이다.  
<br />
`lxd init`명령어를 통해 초기 설정을 잡아 주자.  
```bash
$ sudo lxd init
  Would you like to use LXD clustering? (yes/no) [default=no]: 
  Do you want to configure a new storage pool? (yes/no) [default=yes]: 
  Name of the new storage pool [default=default]:
  Name of the storage backend to use (ceph, btrfs, dir, lvm, zfs) [default=zfs]: zfs
  Create a new ZFS pool? (yes/no) [default=yes]: 
  Would you like to use an existing block device? (yes/no) [default=no]: 
  Size in GB of the new loop device (1GB minimum) [default=18GB]: 10GB
  Would you like to connect to a MAAS server? (yes/no) [default=no]: 
  Would you like to create a new local network bridge? (yes/no) [default=yes]: 
  What should the new bridge be called? [default=lxdbr0]:
  What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
  What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
  Would you like LXD to be available over the network? (yes/no) [default=no]: 
  Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
  Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: 
```
`lxd init`명령어는 LXD의 기본 구성을 설정해 주는 명령어로, storage와 network 설정을 도와준다.  
<br />
해당 명령어를 실행하면 다음과 같이 storage와 network가 생성 되는것을 확인 할 수 있다.  
```bash
$ lxc storage list
+---------+-------------+--------+--------------------------------------------+---------+
|  NAME   | DESCRIPTION | DRIVER |                   SOURCE                   | USED BY |
+---------+-------------+--------+--------------------------------------------+---------+
| default |             | zfs    | /var/snap/lxd/common/lxd/disks/default.img | 1       |
+---------+-------------+--------+--------------------------------------------+---------+

$ lxc network list
+--------+----------+---------+-------------+---------+
|  NAME  |   TYPE   | MANAGED | DESCRIPTION | USED BY |
+--------+----------+---------+-------------+---------+
| enp2s0 | physical | NO      |             | 0       |
+--------+----------+---------+-------------+---------+
| ...                                                 |
+--------+----------+---------+-------------+---------+
| lxdbr0 | bridge   | YES     |             | 0       |
+--------+----------+---------+-------------+---------+

$ ip -c a show dev lxdbr0
5: lxdbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 5a:aa:dd:83:3f:c0 brd ff:ff:ff:ff:ff:ff
    inet 10.8.53.1/24 scope global lxdbr0
       valid_lft forever preferred_lft forever
    inet6 fd42:c6a8:fd07:f50c::1/64 scope global
       valid_lft forever preferred_lft forever
    inet6 fe80::58aa:ddff:fe83:3fc0/64 scope link
       valid_lft forever preferred_lft forever
```
<br />
이제 다시 컨테이너를 생성해 보자.  
```bash
$ lxc launch ubuntu:18.04 test
  Creating test
  Starting test

$ lxc list 
+------+---------+-------------------+-----------------------------------------------+-----------+-----------+
| NAME |  STATE  |       IPV4        |                     IPV6                      |   TYPE    | SNAPSHOTS |
+------+---------+-------------------+-----------------------------------------------+-----------+-----------+
| test | RUNNING | 10.8.53.53 (eth0) | fd42:c6a8:fd07:f50c:216:3eff:fe7c:75c4 (eth0) | CONTAINER | 0         |
+------+---------+-------------------+-----------------------------------------------+-----------+-----------+
```
정상적으로 생성이 잘 되었음을 확인 할 수 있다.   
<br />
확인을 하였으니 test로 만든 컨테이너를 삭제하겠다.  
```bash
$ lxc stop test
$ lxc delete test
```
<br />

## LXD 설정
위에서 `lxd init`으로 설정을 하였지만, 원하는 구성으로 설정 하는데는 한계가 있다.  
이전 글에서 Node0을 Controller Node로 사용하려 하였으니 Node0 기준으로 설정을 다시 잡아 보도록 하겠다.
<br />

## Storage 설정
Storage의 구성으로 보면 SSD(500GB), HDD(300GB)가 있으며, 해당 Storage에서 각각 380GB, 320GB를 데이터 전용으로 파티션을 잡아둔 상태이다.  
```bash
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
...
/dev/sda2        92G  3.2G   84G   4% /
/dev/sda3       354G  394M  354G   1% /data/ssd
/dev/sdb1       298G  337M  298G   1% /data/hd
...
```
{{< admonition type=question title="df 명령어에서 용량 차이가 나는 이유" open=false >}}
여기서 용량의 차이가 나타나는데 이것은 어떤 단위를 기준으로 계산하여 보여줄 것인가에 차이이다.  
  - SI단위인 10진법으로 표기 (10^n) [1000B = 1kB] 
  - 컴퓨터의 단위인 2진법으로 표기 (2^n) [1024B = 1KB]

디스크 용량은 10진법으로 표기가 되어 판매가 되며, 실제 컴퓨터에서 사용하는 용량과 차이가 생기게 된다.  
만일 df 명령어에서 10진법 기준으로 확인하고자 하면 `df -H`로 확인이 가능하다.  
위의 명령어는 각각 다음과 같이 사용이 가능하다.  
  - `df -h` = `df --human-readable` 
  - `df -H` = `df --si`
{{</ admonition >}}

이 파티션을 사용하여 3개의 Storage Pool을 생성을 할 계획이다.    
LXD의 Storage는 ZFS 또는 btrfs을 권장하고 있으므로 ZFS를 사용하도록 하겠다.  
<br />

### ZFS Utils 설치
ZFS를 구성하려면 `zpool` Command를 사용하여 구성하게 되는데 `zfsutils`가 필요하다.  
snap에는 해당 패키지가 검색이 안되서 apt로 설치를 진행한다.  
```bash
$ sudo apt install zfsutils-linux
```
<br />
설치 후 zpool을 확인하면 LXD 초기화시 생성된 pool 하나가 확인될 것이다.  
```bash
$ sudo zpool list
NAME      SIZE  ALLOC   FREE  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
default  9.25G   346M  8.91G         -     0%     3%  1.00x  ONLINE  -
```

### ZFS Pool 구성
이제 3개의 pool을 구성할 것이다.  
먼저 `/dev/sda2`의 경우 `/`로 mount가 되어 있으므로 해당 pool을 관리할 포인트로 `/data/root` dir를 만들고 시작하겠다.  
```bash
$ sudo mkdir -p /data/root
```
<br />
이미 각 디스크가 마운트 되어 있는 상태이고, 루트 파티션도 사용할 예정 이므로 손쉽게 File기반으로 ZFS를 구성하도록 하겠다.   
`truncate` 명령어를 통하여 일단 50G를 할당하여 File을 생성하도록 하겠다.
```bash
$ sudo truncate -s 50G /data/root/lxd-zfs-disk.img
$ sudo truncate -s 50G /data/ssd/lxd-zfs-disk.img
$ sudo truncate -s 50G /data/hd/lxd-zfs-disk.img
```
<br />
확인을 해보면 50G로 할당된 File이 생성된것을 확인 할 수 있다.  
```bash
$ sudo ls -lh /data/root /data/ssd /data/hd
/data/hd:
total 0
-rw-r--r-- 1 root root 50G May 24 17:00 lxd-zfs-disk.img

/data/root:
total 0
-rw-r--r-- 1 root root 50G May 24 17:00 lxd-zfs-disk.img

/data/ssd:
total 0
-rw-r--r-- 1 root root 50G May 24 17:00 lxd-zfs-disk.img
```
<br />
생성된 File로 ZFS Pool을 생성해 보겠다.
```bash
$ sudo zpool create lxd-root /data/root/lzd-zfs-disk.img
$ sudo zpool create lxd-fast /data/ssd/lzd-zfs-disk.img
$ sudo zpool create lxd-standard /data/hd/lzd-zfs-disk.img
```
<br />
생성이 완료되었으면 정상적으로 pool list에서 확인이 가능할 것이다.
```bash
$ sudo zpool list
NAME           SIZE  ALLOC   FREE  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
default       9.25G   344M  8.91G         -     0%     3%  1.00x  ONLINE  -
lxd-fast      49.8G   106K  49.7G         -     0%     0%  1.00x  ONLINE  -
lxd-root      49.8G   108K  49.7G         -     0%     0%  1.00x  ONLINE  -
lxd-standard  49.8G   108K  49.7G         -     0%     0%  1.00x  ONLINE  -
```
<br />

### ZFS Pool - LXD 연결
생성된 Pool을 LXD에서 사용을 하려면 Storage로 등록해야 한다.  
```bash
$ sudo lxc storage create lxd-root zfs source=lxd-root
$ sudo lxc storage create lxd-fast zfs source=lxd-fast
$ sudo lxc storage create lxd-stadard zfs source=lxd-standard
```
등록이 되면 list에 정상적으로 등록이 되었는지 확인을 한다.  
```bash
$ sudo lxc storage list
+--------------+-------------+--------+--------------------------------------------+---------+
|     NAME     | DESCRIPTION | DRIVER |                   SOURCE                   | USED BY |
+--------------+-------------+--------+--------------------------------------------+---------+
| default      |             | zfs    | /var/snap/lxd/common/lxd/disks/default.img | 2       |
+--------------+-------------+--------+--------------------------------------------+---------+
| lxd-fast     |             | zfs    | lxd-fast                                   | 0       |
+--------------+-------------+--------+--------------------------------------------+---------+
| lxd-root     |             | zfs    | lxd-root                                   | 0       |
+--------------+-------------+--------+--------------------------------------------+---------+
| lxd-standard |             | zfs    | lxd-standard                               | 0       |
+--------------+-------------+--------+--------------------------------------------+---------+
```
<br />

### ZFS Pool 용량 조절
Pool을 사용하다 보면 용량이 부족해질수가 있다.  
그럴때 용량을 확장해야 하는데 File기반이므로 File의 용량을 조절하여 Pool의 용량을 조절 할 수 있다.
```bash
```

{{< admonition type=tip title="참고 자료" open=false >}}
- [LXD Getting Started - command line | Linux containers](https://linuxcontainers.org/lxd/getting-started-cli/)
- [LXD Documentation | Linux containers](https://linuxcontainers.org/lxd/docs/master/)
- [Using LXD with a file-based ZFS pool on Ubuntu Wily | Ubuntu Blog](https://ubuntu.com/blog/using-lxd-with-a-file-based-zfs-pool-on-ubuntu-wily)
{{< /admonition >}}