---
title: "[OpenStack기반 Private Cloud 구축기] 1. 어쩌다 OpenStack"
date: 2020-05-17T19:07:56+09:00
draft: true
toc: true
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

## 어쩌다 보니?
어쩌다 보니 기존에 간단한 FTP와 토이 프로젝트용 서버로 간간히 굴리고 있던 컴퓨터와 추가로 2대의 컴퓨터 자원이 남게 되면서 이전부터 한번 해 보고 싶었던 Private Cloud를 구축이 생각났다.  
컴퓨터들이 다 오래 되다 보니 사양도 높지 않고 성능은 안나오겠지만 한번 구축 해보면서 공부도 할 겸, 구축하고 운용 해보면서 이리저리 살펴보는것에 의미를 두며(?) 시작 해보도록 하겠다.  


## OpenStack?
`OpenStack`은 1대 이상의 컴퓨터의 가상화된 리소스 풀을 가지고 Iaas(Infrastructure As A Service)를 구축 할 수 있게 도외주는 **`오픈소스 클라우드 컴퓨팅 프로젝트`**이다.  
클라우드 컴퓨팅 플랫폼에 맞게 여러가지 서비스가 있는데 주요하게 사용되는 `컴퓨팅(Nova)`, `스토리지(Swift)`, `네트워킹(Neutron)` 그 외에도 각종 서비스가 존재한다.  
기본적으로 모든 서비스의 인증은 `Identity 서비스(Keystone)`를 통하여 이루어 지며 각 개별 서비스는 API를 통하여 상호 작용한다.  
OpenStack의 각 서비스는 [`여기서`](https://www.openstack.org/software/project-navigator/openstack-components#openstack-services)확인이 가능하며 필요에 따라 설치하여 구성할 수 있다.  


## OepnStack 서비스 구성
OpenStack의 [Install Guide](https://docs.openstack.org/install-guide/overview.html#example-architecture)를 먼저 참고하면, 기본적으로 2개의 node로 구성하는 예제로 설명이 되어 있다.  
- OpenStack의 각 서비스에 필요한 인증, 이미지, 리소스 관리, 대시보드 등 management를 위한 서비스들을 설치하기 위한 controller node
- KVM과 같은 hypervisor를 실행하여 가상화 환경을 만들고 vm instance를 생성하게될 compute node

\
꼭 이대로 구성 할 필요는 없으며 1대에 전부 설치해서 구성해도 되고 여러대에 각 서비스를 원하는 대로 구성을 하면 된다.  
만약 `단일 Node`에 기본적인 구성만 하고자 한다면 [DevStack](https://docs.openstack.org/devstack/latest/)을 이용하면 아주 손쉽게 설치가 가능하다.  

\
위의 가이드를 따라서 하나씩 살펴보면, 기본적으로 아래의 항목이 설치가 되어 있어야 한다.  
- NTP
  - 각 노드간의 시간을 동기화 해주기 위하여 필요하다.
- SQL, Memcached
  - 각 서비스들의 정보를 저장하기 위하여 필요하며, 토큰을 캐싱하기 위하여 필요하다.
- Message Queue
  - 각 서비스 간의 작업과 상태 정보에 대한 교환 및 조정 등을 위해 사용된다.

\
그리고 다음의 컴포넌트 구성이 OpenStack의 기본 구성이 된다.  
- Identity (Keystone)
  - 인증, 인가, 서비스 카탈로그 관리(각 서비스에 대한 정보)를 한 곳에서 통합하기 위한 서비스
- Image (Glance)
  - 인스턴스를 띄우려면 가상 머신 이미지가 필요하다. 해당 이미지를 관리하기 위한 서비스
- Placement (Placement)
  - 리소스 사용량을 추적 관리 하기 위한 서비스
- Compute (Nova)
  - 인스턴스를 생성, 종료 등 컴퓨팅 자원을 관리하기 위한 서비스 
  - 관리를 위한 서비스들과 컴퓨팅을 위한 서비스로 나뉘어져 있다
- Networking (Neutron)
  - 네트워크를 생성하고 인터페이스에 대한 연결을 관리하기 위한 서비스
  - 가상 네트워크 환경을 구축하고 그 위에서 각 서비스간 연결, 내부 네트워크와 외부 네트워크에 대한 연결을 관리 해준다고 보면 된다.
- 대시보드 (Horizon)
  - 구성된 OpenStack 자원과 서비스들을 관리 할 수 있는 웹 인터페이스이다.
  - 없어도 되지만(?) 직접 CLI로 작업할게 아니라면 무조건 설치하자.

\
그 외에 보통 추가로 구성하게 되는 컴포넌트는 다음과 같다.  
- 블록 스토리지 (Cinder)
  - 인스턴스에 스토리지를 제공하기 위한 서비스.
  - 관리를 위한 api, 스케줄러 서비스와 볼륨 서비스로 구성된다. 
- 오브젝트 스토리지 (Swift)
  - 오브젝트 스토리지를 제공하기 위한 서비스.
  - 관리를 위한 서비스들과 스토리지를 위한 서비스들로 구성된다.

\
일단 기본적인 위의 구성을 올려보고 나서 여유가 되면 DNS, Load Balancer, Key Management, Orchestration, Container(+Orchestration)등 서비스를 확장해 보도록 하겠다.  

## Node 구성
일단 나의 경우는 앞에서 적었듯 3대의 컴퓨터가 준비 되어있고 사양은 다음과 같다.  
- Node0 
  - CPU: 4Core 2.8Ghz 
  - Mem: 12GB
  - Storage: 500GB(SSD), 300GB(HDD)
  - NIC: 3
- Node1
  - CPU: (4/8)Core 3.4Ghz
  - Mem: 16GB
  - Storage: 500GB(Nvme), 256GB(SSD), 3TB(HDD), 1TB(HDD)x2
  - NIC: 2
- Node2
  - CPU: 4Core 1.66Ghz
  - Mem: 4GB
  - Storage: 1TB(HDD), 2TB(HDD)
  - NIC: 2

모든 노드는 `Ubuntu Server 18.04 LTS`를 사용하고, 전부 `5~10이 넘은` 컴퓨터들이라 사양이 그렇게 좋지는 않다.  

\
각 노드에는 다음과 같이 서비스를 구성해 보도록 할 계획이다.  
- Node0
  - NTP, SQL, Memcached, Message Queue
  - Keystone, Placement, Horizon
  - Nova(Management), Neutron(Management)
- Node1
  - Nova, Neutron
  - Cinder, Swift
- Node2
  - Swift

\
성능 순서대로 `Compute Node`, `Controller Node`로 나누었으며, 남은 Node를 `Storage Node`로 일단 정리하였다.  
이번 글에서는 대략적인 OpenStack 구성과 어떻게 구성할 것인지를 보았고, 다음 글부터는 차근차근 서비스 하나씩 올려 보도록 하겠다.  

\
만약 따라서 해보실 분이 있다면, 주변에 굴러다니는 컴퓨터 혹은 가상화 환경 위에서도 충분히 가능하니 시도 해보는것도 추천한다.  
반드시 위와 같은 구성이 아니더라도 각 서비스를 적절하게 구성하기만 하면 된다.  


{{< admonition type=tip title="참고 자료" open=false >}}
- [OpenStack Components | OpenStack](https://www.openstack.org/software/project-navigator/openstack-components)  
- [OpenStack Install Guide | OpenStack](https://docs.openstack.org/install-guide/index.html)
- [오픈스택(OpenStack) 이란? | Popit](https://www.popit.kr/%EC%98%A4%ED%94%88%EC%8A%A4%ED%83%9D-openstack-%EC%9D%B4%EB%9E%80/)
- [OepnStack | RedHat](https://www.redhat.com/ko/topics/openstack)
{{< /admonition >}}
