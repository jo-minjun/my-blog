---
title: "네트워크 기초 지식"
author: ["조민준"]
date: 2022-08-17T17:10:48+09:00
draft: false
categories: ["Network/Security"]
tags: ["Network"]
ShowToc: true
---

## 1. MAC, IP, Port 번호의 식별

### MAC, IP, Port는 각 다음과 같은 레이어의 식별자이다.

```shell
* DoD로 구분한 Layer *

user mode
============================================
- Application

kernel mode
============================================
- Transport: PORT
- Network: IP

H/W
============================================
- Access: MAC
```

Transport

- Port가 식별자가 된다.

Network

- Host에 대해서 IP가 식별자가 된다.
  - Host: 네트워크에 연결된 컴퓨터, NIC 하나에 IP 주소를 여러 개 바인딩 할 수 있다.  
    → Host에 IP가 여러개 존재한다.

Access

- Network Interface Card에 대해서 MAC이 식별자가 된다.
  - 하드웨어 식별자지만, MAC 변경이 가능하다.
  - Network Interface Card: 노트북은 유선/무선 랜카드가 2개
    → NIC가 2개

## 2. Host, Switch, Network의 관계

**Host**는 Network에 연결된 컴퓨터이다.

Network는 다음과 같이 두 가지로 나뉜다.

- Network 이용 주체
  → **End-Point**가 된다. (Peer, Server, Client 등)
- Network 자체
  → 이 Computer를 **Switch**라 한다. (Firewall, Router 등)

**Network**는 **Router(L3 Switch)와 DNS의 집합체**이다.

```shell
(스위칭 비용 증가)

user mode
============================================
L7
--------------------------------------------
L6
--------------------------------------------
L5
--------------------------------------------

kernel mode
============================================
L4
- TCP
--------------------------------------------
L3 (Router)
- IP
--------------------------------------------

H/W
============================================
L2
--------------------------------------------
L1
--------------------------------------------

(스위칭 비용 감소)
```

## 3. IPv4 주소 체계

### IP 주소

Host에 대한 식별자

IP 주소는 다음과 같이 두가지 표현 방법이 있다.

- IPv4: 32bit
- IPv6: 128bit

IPv4의 32bit는 다음과 같이 이루어져 있다.

- 32bit는 8bit씩 나누어 표기한다.
  - 121.123.223.10
- 크게 두 부분으로 나뉜다.
  - Net ID, Host ID
  ```shell
  Net ID   Host ID
  ----------- --
  121.123.223.10
  ```
  - 이때 Net ID의 길이를 구분하기 위해 서브넷 마스크를 사용한다.

### 서브넷 마스크

서브넷 마스크도 32bit로, 8bit씩 나누어 표기한다.

서브넷 마스크와 IP 주소를 and 연산하면 Net ID를 확인할 수 있다.

하지만 일반적으로 다음과 같이 Net ID의 길이를 함께 표기한다.

- 121.123.223.10/24

→ 121.123.223.0은 Net ID, Host는 10이다.

- 이 표기하는 것을 CIDR라 한다.

## 4. Port 번호의 이해

Port는 관점에 따라 여러 의미를 가진다.

- Process 식별자
- Service 식별자
- Interface 번호

여기서는 개발자 관점에서 Process 식별자를 알아본다.

```shell
user mode
============================================
- Process

kernel mode
============================================
- TCP
- IP

- Driver
H/W
============================================
- NIC
```

- kernel mode는 user mode가 접근이 가능하게 하기 위해 file이라는 인터페이스를 제공한다.
- 하지만 이를 프로토콜 관점에서 추상화하면 socket이 된다.
- 이때 socket에 attach되는 정보 중 하나가 Port 번호이다.

Port는 다음과 같은 특징이 있다.

- socket에 attach되는 정보이다.
- 16bit 정보이다.

→ 2^16

→ Port 번호의 범위는 0 ~ 61535 이다. (하지만 0과 61535는 사용할 수 없다.)

- 패킷이 하위 레이어에서 상위 레이어로 올라갈 때 Port를 이용해서 프로세스를 식별한다.

## 5. Switch, Switching

**Switching은 경로 또는 인터페이스를 선택**하는 것이다.

이 때 **선택지가 나오는 곳을 Switch**라 한다.

Network는 라우터와 DNS의 집합이다.

- 라우터는 L3 스위치이다.
- 라우터는 **라우팅 테이블**을 근거로 최적의 경로를 찾아낸다.

## 6. 네트워크 데이터 단위

```shell
user mode
============================================
- Application (Socket 수준)

kernel mode
============================================
- TCP
- IP

H/W
============================================
-
```

- 위에서 설명한 user와 kernel 모드 사이의 file(socket)은 **Stream**이다.
  - 기본적으로 file은 사용자가 계속 데이터를 입력하면 계속해서 데이터가 커진다.
- TCP에서 다루는 데이터 단위: **Segment**
- IP에서 다루는 데이터 단위: **Packet**
- IP 아래 단계에서 다루는 데이터 단위: **Frame**

이 데이터 단위의 흐름은 다음과 같다.

- Stream이 Segment로 넘어갈 때 일정한 길이로 데이터를 분해한다.
  - 이때 **Segment의 최대 크기를 Maximum Segment Size(MSS)**라 한다.
- MSS는 Packet의 최대 크기으로 결정하게 되는데, **Packet의 최대 크기를 (Maximum Transport Unit)MTU**라 한다.
- 이 Packet을 Frame 데이터로 캡슐화하여 전달한다.

## 7. 네트워크 인터페이스 선택 원리와 기준

```shell
user mode
============================================
- HTTP (L7)
-
- SSL

kernel mode
============================================
- TCP, UDP (L4)
- IP (L3)

H/W
============================================
- Ethernet
-
```

아래와 같은 상황에서 인터페이스는 어떻게 선택될까?

- 사용자가 브라우저를 켰다. (Socket이 열리고, TCP와 IP가 바인딩 되어야 한다.)
- KT 유선 인터넷과 SKT 무선 인터넷을 두 개 연결했다. (일반적으로 IP 주소는 2개가 된다.)

→ 메트릭(쉽게 말해 비용) 값으로 결정한다.

→ 따라서 위 경우에는 KT 유선과 SKT 무선 중 메트랙 값이 적은 쪽으로 바인딩 된다.

## 8. 웹 서비스를 만드신 분에 대해

### 웹 탄생 배경

1. 영국 물리학 연구원 **팀 버너스 리**
2. 연구원은 논문을 많이 읽는다.
3. 논문에는 항상 참고문헌이 많았다.
4. 하지만 당시 링크 개념이 존재하지 않았고 문서는 모두 Text 파일이었다.

→ 문서(Text) + Link 를 이용해서 **HTML**이라는 문서 형식을 만들었다.

→ **HTML**의 인터넷 전달 방법을 위해서 **HTTP**라는 프로토콜을 만들었다.

→ 여러 문서들이 계속해서 연결되니, Web이 형성되었다.

→ 이는 웹 서비스가 되었고, 팀 버너스 리가 창안했다.

## 9. 초창기 웹 서비스의 구조

**웹 클라이언트 (브라우저)**는 인터넷을 통해서 **웹 서버**와 연결된다.

이때 연결은 HTTP라는 TCP/IP 기반으로 된다.

- HTTP는 Stateless하다.

웹 클라이언트의 IP 주소가 있고 웹 서버에도 IP 주소가 있다.

→ 이 주소 URL을 알고 리소스(HTML 문서)에 대한 **요청**을 하면 연결이 되고, **응답**을 준다.

→ 응답을 받은 클라이언트는 HTML **구문 분석**을 하고, 내용을 **렌더링** 한다.

즉, 이 당시 브라우저는 원격 문서 뷰어 역할(단방향 작용)을 했다.

## 10. 웹 서비스 3대 요소

위 구조에는 문제가 있었다.

1. **문서의 내용과는 별개로 UI를 개선하고 싶었다.**

   → HTML에 기능을 추가하니 유지보수가 불편하다.

   → CSS와 이미지가 나왔다.

   → 요청하면 HTML + CSS + IMAGE(서버에 저장되어 있다.)가 순서대로 응답된다.

2. **문서를 변경해야 했다.**

   → 문서를 변경하기 위해 처리를 담당하는 서버가 생겼다.

   → 단방향 작용이 양방향 상호 작용이 되면서 상태를 처리해야 했다.

   → DB를 이용해서 처리하게 되었다.

3. **문서가 변경되고, 처리가 되니 문서가 복잡해졌다.**

   → 기능에 따라 동적인 움직임을 주는 것이 필요해졌다.

   → 브라우저에서 렌더링 후에 **연산**하는 기능을 가지게 되었다. (Javascript)

즉, 웹 서비스 3대 요소는 다음과 같다.

- HTML 구문 분석
- 렌더링
- 연산

## 11. LAN vs WAN

- 주의) 이 내용은 명확한 구분은 아니다.

```shell
user mode
============================================
- HTTP
-
- SSL

kernel mode (Logical == virtual)
============================================
- TCP, UDP
- IP(IP 주소) --> [Internet] == [virtual network] -> *WAN

H/W (Physical)
============================================
- Ethernet(MAC 주소) --> *LAN
-
```

LAN과 WAN은 흔히 범위의 차이로 구분한다.

하지만 논리/물리적 구성요소로 구분하는 방법은 아래와 같다.

- 시스템은 S/W와 H/W로 구분된다.
- S/W(kernel mode 이상)은 IP(Internet Protocol) 를 이용해서 통신한다.
  - Internet은 virtual network이다.
    → WAN
- H/W는 MAC 주소를 이용한다. (이는 물리적인 주소이다.)
  → 하드웨어로 설명되는 네트워크
  → LAN

## 12. 패킷의 생성 원리와 캡슐화

```shell
user mode
============================================
- HTTP
-
- SSL

kernel mode
============================================
- TCP, UDP
- IP

H/W
============================================
- Ethernet
-
```

1. user mode 영역은 socket에 데이터를 IO한다.
   - socket은 file을 추상화한 것이다. 때문에 데이터를 계속해서 쓰기할 수 있다.
   - 이때 데이터의 단위는 Stream이다.
2. user mode의 Stream이 Kernel mode에서 일정 단위로 나누어진다.
   - 이때 데이터의 단위는 Segment이다.
   - 이 Segment가 한 번 캡슐화되어 Packet이 된다.
     - Packet의 최대 크기: MTU(Maximum Transport Unit)
       - 일반적으로 1500이다.
     - Packet의 구조: Header, Payload
     - Header에는 IP(L3)와 TCP(L4) 데이터가 있다.
       - 이때 크기는 일반적으로 각 20씩으로 총 40이다.
     - MTU 크기 - Header 크기 = 1460이다.
       - 이 크기가 MSS(Maximum Segment Size) 이다.
       - Stream을 1460 크기로 나눈 것이다.
3. H/W영역에서 Packet이 한 번 더 캡슐화된다.
   - Frame이 된다.

## 13. L2 스위치

L2 스위치는 MAC 주소(48bit)로 스위칭시킨다.

```shell
                (multilayer switch)
 NIC  L2 Access   L2 Distribution    L2 Access    NIC
         |  (Up-link)        (Up-link)
         |---------------#---------------|
PC1------|               |               |---------PC3
         |               |               |
         |               |               |
         |               |               |---------PC4
PC2------|               |               |
         |               |               |
                         |
                         | (Up-link)
                         |
                     (gateway)
                         @
                       라우터
                         |
                       (방화벽)
```

- L2 Access: End-Point가 네트워크에서 가장 처음 만나는 스위치
- L2 Distribution: L2 Access와 L3 라우터를 연결해주는 스위치
- Up-link: 상위 계층 스위치로 연결되는 케이블

## 14. IP Header

![0.png](/images/notes/networkbasic/0.png)

위에서 언급한 것처럼 IP의 헤더는 20바이트이다. (+ @ Opitonal)

최상단 우측에 Total Length는 16비트인데, 이것은 패킷의 최대 크기를 나타낸다.

- 따라서 패킷의 최대 크기는 2^16정도인 65536이다.

Identification ~ Fragment Offset은 단편화에 관련된 부분이다.

- 단편화는 큰 패킷을 작은 패킷으로 나눈 것을 말한다.
- MTU가 1400인 곳에 1500짜리 패킷을 보내는 경우 단편화가 일어난다.

TTL은 패킷이 라우터 하나를 지날때마다 1씩 감소하고 0이 되면 패킷이 사라진다.

- 일반적으로 값은 256이다. (2^8)

Protocol은 상위 계층 프로토콜이다.

- 이 값을 보고 데이터가 TCP인지 UDP 인지 다른 값인지 확인할 수 있다.

Header checksum은 전송간에 오류가 있는지 확인한다.

## 15. Proxy 구조와 원리

Proxy는 대리자 역할을 한다.

<Proxy 미적용>

```shell
										    HTTPS TCP/IP
PC1 (1.1.1.1)--------------Internet--------------------SERVER(9.9.9.9)
```

<Proxy 적용>

```shell
PC1 (1.1.1.1)                                          SERVER(9.9.9.9)
  |                                                        |
  |                                                        |
Internet                                                   |
  |                                                        |
  | (Proxy)                                                |
PC2 (2.2.2.2)---------------- Internet---------------------|
```

<PC2의 역할>

```shell
user mode    |          PC2
============================================
- HTTP       |   Proxy 역할을 하는
-            |       Process
- SSL        |       (Stream)
             |   socket1  socket2
kernel mode  |
============================================
- TCP, UDP   |
- IP         |
             |
H/W          |
============================================
- Ethernet   |
-            |
```

1. socket1은 외부에서 접속하길 대기하고 있다.
2. PC1이 접근
3. 정보가 들어오면 socket2를 이용해서 9.9.9.9에 접근

## 16. Proxy의 활용

### 1. 우회

Proxy를 사용하면 SERVER 입장에서 PC2의 아이피를 확인한다.

그러나 PC2는 PC1의 모든 통신을 감청할 수 있다.

### 2. 분석

웹 통신에 SSL을 적용하면 패킷 레벨에서 데이터가 암호화되어 있다.

때문에 와이어 샤크등 프로그램에서 복호화된 데이터를 확인할 수 없다.

이때 프록시를 아래와 같이 사용할 수 있다.

- Proxy를 127.0.0.1:8080으로 건다. (내 PC)
- HTTP 요청을 보내면 8080번 포트의 소켓으로 데이터가 지나간다.
- 이 평문 데이터를 Stream 레벨에서 확인한다.

## 17. TCP 송신/수신 원리

1. Stream을 Segment로 나눈다.
   - 이때 TCP Buffer(Window Size)에 데이터를 저장하고, 일정 크기가 되면 Segment로 나눈다.
2. Segment를 Packet으로 캡슐화한다.
3. Packet을 Frame으로 캡슐화한다.
4. Frame을 전달한다.
   - 전체를 보내는 것은 아니다. (n개 만큼 보낸다.)
   - 일반적으로 Frame은 바뀔 수 있다. (Packet은 그대로 이지만)
5. Frame을 전달받고 Segment로 만든다.
6. n개의 Segment를 받으면 TCP Buffer에 저장하고 ackn+1을 송신쪽에 전달한다.
   - ack에는 Window Size가 포함되어 있다.
7. 송신쪽은 ackn+1을 받고 Segment를 n+1번부터 다시 보낸다.
   - 이 과정때문에 속도 지연이 발생한다. (UDP 보다 느리다.)
   - 만약 ack로 받은 수신 측 Window Size가 작으면 Segment를 보내지 않는다.

## 18. TCP 연결에 대해

TCP는 연결지향 프로토콜이다.

우선 아래 TCP 헤더를 보자

![1.png](/images/notes/networkbasic/1.png)

- 출발지/목적지 Port 번호가 가장위에 위치한다.
- Sequence Number: 32bit → 4GB (2^32), Segment의 순서
- TCP Flags: Ack, Sync
  - 3-way handshake에 사용

3-way handshake

- Seq번호와 MSS를 교환하는 행위
- 혼잡 제어 정책 교환

## 19. Unicast, Broadcast, Multicast

```shell
 NIC  L2 Access   L2 Distribution    L2 Access    NIC
         |  (Up-link)        (Up-link)
         |---------------#---------------|
PC1------|               |               |---------PC3
         |               |               |
         |               |               |
         |               |               |---------PC4
PC2------|               |               |
         |               |               |
                         |
                         | (Up-link)
                         |
                         @ 라우터 (gateway)
                       (방화벽)
```

Unicast

- L2 스위치 내부에서 연결이 끝나는 것 (라우터 이전)
- 한번에 한 지점에게만 신호를 보낸다.

Broadcast

- 어떤 지점에서 다수의 지점에 신호를 보내는 것
- 네트워크 효율을 떨어뜨린다.
- 2진수 IP의 끝자리가 모두 1이다. (210.153.0.255)

Multicast

- Broadcast와 유사하나 관심이 없는 지점은 신호를 보내지 않음
- Group을 등록해서 Group에 전달함

## 20. IP의 종류

**Global**

- 인터넷(public, global network)에서 라우터가 라우팅 시켜주는 IP이다.

**Private**

- 작은 소규모 사설 인터넷을 구축할 때 사용한다.
- 공유기에서도 자주 사용한다.
  - 공유기는 하나의 Global IP를 Private IP에 공유해주는 역할을 한다.
- 4개의 클래스로 나뉜다.
  - A:
    - network id: 8bit
    - host id: 24bit
    - 10.xxx.xxx.xxx
  - B
    - network id: 12bit
    - host id: 20bit
    - 172.16.xxx.xxx
  - C
    - network id: 16bit
    - host id: 16bit
    - 192.168.xxx.xxx
  - D
    - multicast를 할 때 사용한다.

**Loopback**

- 127.0.0.1
- 호스트 자신을 의미한다.
- 패킷이 만들어지지만 L2로 가지는 않는다.

**Broadcast**

- 다수의 지점에 신호를 보낼 때 사용한다.

## 21. DNS

Domain Name

- 숫자로된 IP를 사람이 보기 편하도록 해준다.
- naver.com
- google.com

DNS

- IP와 Domain Name을 연결하는 테이블 역할
- DNS는 계층적 구조로 되어 있다. (분산형 DB 구조)
  - 가장 상위 DNS인 root DNS는 전세계에 13대가 있다.

Domain Name으로 IP를 찾는 과정

1. 컴퓨터 캐시에서 검색
2. host file에서 검색
3. DNS에서 검색
4. root DNS에서 .com, co.kr 등을 관리하는 DNS 목록을 검색
5. DNS 목록에서 검색
6. …

## 22. TCP/IP 통신과 MAC 주소

TCP/IP 통신을 할 때의 MAC 주소의 변화

1. 패킷이 프레임으로 캡슐화된다.
   - 프레임 헤더에도 시작/목적 지점이 있다.
2. L2 구간(라우터)을 지나면 프레임 헤더가 새로 교체된다.
   - L2구간에 새로 접근할 때마다 프레임 헤더의 시작/목적 지점도 계속해서 변경된다.
   - MAC 주소만 고려하고, IP 주소는 고려하지 않는다.

## 23. MTU와 Packet 단편화

![2.png](/images/notes/networkbasic/2.png)

- 위 그림에서 2번째 줄(Identification, Fragment Offset)이 단편화 관련 부분이다.
- MTU는 1500이 기본값인 경우가 많다.
- MTU - 20(IP 헤더) - 20(TCP 헤더) = MSS이다.

아래와 같은 경우에는 단편화가 어떻게 이루어 질까?

```shell
MTU:1500     MTU:1500       MTU:1400       MTU:1500  MTU:1500
PC1 ---|---#-----R1-------------R2-------------R3-----SERVER
```

- 위 경우 R1 → R2에서 단편화가 이루어져야 한다.
  1. 1500짜리 패킷을 어느 지점에서 자른다. A와 B가 생긴다.
  2. 헤더와 A를 붙이고, 헤더와 B를 붙여서 2개의 패킷을 만든다.
     - 이때 두 패킷 헤더의 Idenfication이 같은 값이 된다.
     - Fragment Offset 값은 A는 0, B는 A의 길이만큼이 된다.
  3. 수신하는 쪽에서 단편화를 조립한다. (SERVER)

## 24. 서브넷팅

ISP(Internet Service Provider)에게 어떤 회사가 100개의 사설 IP를 요청하면 어떻게 될까?

- C레벨(Net ID: 24, Host ID: 8)인 private IP를 할당한다.
- 그런데 C레벨 private IP를 할당하면 2^8 - 100 = 146개의 IP를 낭비하게 되는 것이다.
- 이런 경우 서브넷팅을 사용할 수 있다.

서브넷팅은 Net ID에 몇 개의 비트를 더 할당하는 것이다.

- 192.168.0.1/25라면 마지막 4번째 자리의 1의 가장 앞부분 비트를 Net ID로 할당 시킨다.
  - (192.168.0.0)0000001
  - (192.168.0.1)0000001
- 하지만 Host ID가 0인 경우는 아무것도 가리키지 않고, 2진수에서 모두 1인 경우는 broadcast IP 이므로 서브넷 하나마다 2개의 IP를 손실보게 된다.
  - 예를 들어 이 경우라면 192.168.0.1/25
  - 192.168.0.00000000, 192.168.0.01111111
  - 192.168.0.10000000, 192.168.0.11111111
  - 위 4가지 IP를 손실보게 된다.

## Reference

- [https://www.youtube.com/watch?v=k1gyh9BlOT8&list=PLXvgR_grOs1BFH-TuqFsfHqbh-gpMbFoy](https://www.youtube.com/watch?v=k1gyh9BlOT8&list=PLXvgR_grOs1BFH-TuqFsfHqbh-gpMbFoy)
