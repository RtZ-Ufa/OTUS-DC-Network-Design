#  Домашнее задание №5
# VxLAN. L2 VNI
## по теме №11 "VxLAN. EVPN L2" 
### Цель: Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами
### Задачи:
+ настроить BGP peering между Leaf и Spine в AF l2vpn evpn;
+ зафиксировать в документации - план работы, адресное пространство, схему сети, конфигурацию устройств.
## Практическая часть
#### Схема ЦОД
1. Схема сети

![Схема сети](network.PNG)

2. Адресное пространство для Underlay
+ Lo1 - 172.16.XNN.0/16, где X нечётный - spine, X чётный - leaf; N - номер коммутатора 
+ Lo2 - 172.17.XNN.0/16, где X нечётный - spine, X чётный - leaf; N - номер коммутатора
+ P2P links - 172.18.NNN.0/15, где N - номер коммутатора spine

Таблица адресов
  
| Hostname | Interface |  	IP/MASK    |	Description |
|----------|-----------|---------------|--------------|
|spine-1   |Eth1     |172.18.1.0/31  |-L- leaf-1    |
|spine-1   |Eth2     |172.18.1.2/31  |-L- leaf-2    |
|spine-1   |Eth3     |172.18.1.4/31  |-L- leaf-3    |
|spine-1   |L01        |172.16.101.1/32  |              |
|spine-1   |L02        |172.17.101.1/32|              |
|spine-2   |Eth1     |172.18.2.0/31  |-L- leaf-1    |
|spine-2   |Eth2     |172.18.2.2/31  |-L- leaf-2    |
|spine-2   |Eth3     |172.18.2.4/31  |-L- leaf-3    |
|spine-2   |L01        |172.16.102.1/32  |              |
|spine-2   |L02        |172.17.102.1/32|              |
|leaf-1    |Eth1     |172.18.1.1/31  |-L- spine-1    |
|leaf-1    |Eth2     |172.18.2.1/31  |-L- spine-2    |
|leaf-1    |L01        |172.16.201.1/32 |              |
|leaf-1    |L02        |172.17.201.1/32|              |
|leaf-2    |Eth1     |172.18.1.3/31  |-L- spine-1    |
|leaf-2    |Eth2     |172.18.2.3/31  |-L- spine-2    |
|leaf-2    |L01        |172.16.202.1/32 |              |
|leaf-2    |L02        |172.17.202.1/32|              |
|leaf-3    |Eth1     |172.18.1.5/31  |-L- spine-1    |
|leaf-3    |Eth2     |172.18.2.5/31  |-L- spine-2    |
|leaf-3    |L01        |172.16.203.1/32 |              |
|leaf-3    |L02        |172.17.203.1/32|              |
|client-1   |Eth        |192.168.0.1/24|              |
|client-2   |Eth        |192.168.0.2/24|              |
|client-3   |Eth        |192.168.0.3/24|              |
|client-4   |Eth        |192.168.0.4/24|              |

Для коммутаторов spine выбрана AS 65000, для коммутаторов leaf - соответственно по их номерам 650001, 65002, 65003.

3. Настройки оборудования приведены в соотвествующих текстовых файлах в этом каталоге:

4. Таблицы маршрутизации на коммутаторах:

**Коммутатор spine-1**

```
spine-1#show  ip route

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 C        172.16.101.1/32 is directly connected, Loopback1
 B E      172.16.201.1/32 [200/0] via 172.18.1.1, Ethernet1
 B E      172.16.202.1/32 [200/0] via 172.18.1.3, Ethernet2
 B E      172.16.203.1/32 [200/0] via 172.18.1.5, Ethernet3
 C        172.17.101.1/32 is directly connected, Loopback2
 B E      172.17.201.1/32 [200/0] via 172.18.1.1, Ethernet1
 B E      172.17.202.1/32 [200/0] via 172.18.1.3, Ethernet2
 B E      172.17.203.1/32 [200/0] via 172.18.1.5, Ethernet3
 C        172.18.1.0/31 is directly connected, Ethernet1
 C        172.18.1.2/31 is directly connected, Ethernet2
 C        172.18.1.4/31 is directly connected, Ethernet3

spine-1#
```

**Коммутатор spine-2**

```
spine-2#sh ip route

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 C        172.16.102.1/32 is directly connected, Loopback1
 B E      172.16.201.1/32 [200/0] via 172.18.2.1, Ethernet1
 B E      172.16.202.1/32 [200/0] via 172.18.2.3, Ethernet2
 B E      172.16.203.1/32 [200/0] via 172.18.2.5, Ethernet3
 C        172.17.102.1/32 is directly connected, Loopback2
 B E      172.17.201.1/32 [200/0] via 172.18.2.1, Ethernet1
 B E      172.17.202.1/32 [200/0] via 172.18.2.3, Ethernet2
 B E      172.17.203.1/32 [200/0] via 172.18.2.5, Ethernet3
 C        172.18.2.0/31 is directly connected, Ethernet1
 C        172.18.2.2/31 is directly connected, Ethernet2
 C        172.18.2.4/31 is directly connected, Ethernet3

spine-2#
```

**Коммутатор leaf-1**

```
leaf-1#show ip route

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 B E      172.16.101.1/32 [200/0] via 172.18.1.0, Ethernet1
 B E      172.16.102.1/32 [200/0] via 172.18.2.0, Ethernet2
 C        172.16.201.1/32 is directly connected, Loopback1
 B E      172.16.202.1/32 [200/0] via 172.18.1.0, Ethernet1
 B E      172.16.203.1/32 [200/0] via 172.18.1.0, Ethernet1
 B E      172.17.101.1/32 [200/0] via 172.18.1.0, Ethernet1
 B E      172.17.102.1/32 [200/0] via 172.18.2.0, Ethernet2
 C        172.17.201.1/32 is directly connected, Loopback2
 B E      172.17.202.1/32 [200/0] via 172.18.1.0, Ethernet1
 B E      172.17.203.1/32 [200/0] via 172.18.1.0, Ethernet1
 C        172.18.1.0/31 is directly connected, Ethernet1
 C        172.18.2.0/31 is directly connected, Ethernet2

leaf-1#
```

**Коммутатор leaf-2**

```
leaf-2#sh ip route

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 B E      172.16.101.1/32 [200/0] via 172.18.1.2, Ethernet1
 B E      172.16.102.1/32 [200/0] via 172.18.2.2, Ethernet2
 B E      172.16.201.1/32 [200/0] via 172.18.1.2, Ethernet1
 C        172.16.202.1/32 is directly connected, Loopback1
 B E      172.16.203.1/32 [200/0] via 172.18.1.2, Ethernet1
 B E      172.17.101.1/32 [200/0] via 172.18.1.2, Ethernet1
 B E      172.17.102.1/32 [200/0] via 172.18.2.2, Ethernet2
 B E      172.17.201.1/32 [200/0] via 172.18.1.2, Ethernet1
 C        172.17.202.1/32 is directly connected, Loopback2
 B E      172.17.203.1/32 [200/0] via 172.18.1.2, Ethernet1
 C        172.18.1.2/31 is directly connected, Ethernet1
 C        172.18.2.2/31 is directly connected, Ethernet2

leaf-2#
```

**Коммутатор leaf-3**

```
leaf-3#show ip route

VRF: default
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 B E      172.16.101.1/32 [200/0] via 172.18.1.4, Ethernet1
 B E      172.16.102.1/32 [200/0] via 172.18.2.4, Ethernet2
 B E      172.16.201.1/32 [200/0] via 172.18.1.4, Ethernet1
 B E      172.16.202.1/32 [200/0] via 172.18.1.4, Ethernet1
 C        172.16.203.1/32 is directly connected, Loopback1
 B E      172.17.101.1/32 [200/0] via 172.18.1.4, Ethernet1
 B E      172.17.102.1/32 [200/0] via 172.18.2.4, Ethernet2
 B E      172.17.201.1/32 [200/0] via 172.18.1.4, Ethernet1
 B E      172.17.202.1/32 [200/0] via 172.18.1.4, Ethernet1
 C        172.17.203.1/32 is directly connected, Loopback2
 C        172.18.1.4/31 is directly connected, Ethernet1
 C        172.18.2.4/31 is directly connected, Ethernet2

leaf-3#
```

5. Проверка связности между устройствами в ISIS домене проводилась утилитой **ping**. Проверлась связность от текущего коммутатора до интерфейсов Loopback1 и Loopback2 других коммутаторов.

**Коммутатор spine-1**

```
spine-1#
spine-1#ping 172.16.201.1 source lo1
PING 172.16.201.1 (172.16.201.1) from 172.16.101.1 : 72(100) bytes of data.
80 bytes from 172.16.201.1: icmp_seq=1 ttl=64 time=10.3 ms
80 bytes from 172.16.201.1: icmp_seq=2 ttl=64 time=7.76 ms
80 bytes from 172.16.201.1: icmp_seq=3 ttl=64 time=7.27 ms
80 bytes from 172.16.201.1: icmp_seq=4 ttl=64 time=9.89 ms
80 bytes from 172.16.201.1: icmp_seq=5 ttl=64 time=10.0 ms

--- 172.16.201.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 54ms
rtt min/avg/max/mdev = 7.272/9.064/10.388/1.284 ms, ipg/ewma 13.622/9.768 ms
spine-1#
spine-1#ping 172.17.201.1 source lo1
PING 172.17.201.1 (172.17.201.1) from 172.16.101.1 : 72(100) bytes of data.
80 bytes from 172.17.201.1: icmp_seq=1 ttl=64 time=9.69 ms
80 bytes from 172.17.201.1: icmp_seq=2 ttl=64 time=7.46 ms
80 bytes from 172.17.201.1: icmp_seq=3 ttl=64 time=12.2 ms
80 bytes from 172.17.201.1: icmp_seq=4 ttl=64 time=12.8 ms
80 bytes from 172.17.201.1: icmp_seq=5 ttl=64 time=8.62 ms

--- 172.17.201.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 47ms
rtt min/avg/max/mdev = 7.460/10.170/12.835/2.069 ms, pipe 2, ipg/ewma 11.791/9.960 ms
spine-1#
spine-1#
spine-1#
spine-1#ping 172.16.202.1 source lo1
PING 172.16.202.1 (172.16.202.1) from 172.16.101.1 : 72(100) bytes of data.
80 bytes from 172.16.202.1: icmp_seq=1 ttl=64 time=10.3 ms
80 bytes from 172.16.202.1: icmp_seq=2 ttl=64 time=10.3 ms
80 bytes from 172.16.202.1: icmp_seq=3 ttl=64 time=8.63 ms
80 bytes from 172.16.202.1: icmp_seq=4 ttl=64 time=8.11 ms
80 bytes from 172.16.202.1: icmp_seq=5 ttl=64 time=7.51 ms

--- 172.16.202.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 56ms
rtt min/avg/max/mdev = 7.517/8.991/10.376/1.165 ms, ipg/ewma 14.214/9.599 ms
spine-1#
spine-1#ping 172.17.202.1 source lo1
PING 172.17.202.1 (172.17.202.1) from 172.16.101.1 : 72(100) bytes of data.
80 bytes from 172.17.202.1: icmp_seq=1 ttl=64 time=8.86 ms
80 bytes from 172.17.202.1: icmp_seq=2 ttl=64 time=13.6 ms
80 bytes from 172.17.202.1: icmp_seq=3 ttl=64 time=13.3 ms
80 bytes from 172.17.202.1: icmp_seq=4 ttl=64 time=18.3 ms
80 bytes from 172.17.202.1: icmp_seq=5 ttl=64 time=16.8 ms

--- 172.17.202.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 55ms
rtt min/avg/max/mdev = 8.865/14.225/18.394/3.298 ms, pipe 2, ipg/ewma 13.885/11.737 ms
spine-1#
spine-1#
spine-1#
spine-1#ping 172.16.203.1 source lo1
PING 172.16.203.1 (172.16.203.1) from 172.16.101.1 : 72(100) bytes of data.
80 bytes from 172.16.203.1: icmp_seq=1 ttl=64 time=8.86 ms
80 bytes from 172.16.203.1: icmp_seq=2 ttl=64 time=12.1 ms
80 bytes from 172.16.203.1: icmp_seq=3 ttl=64 time=8.89 ms
80 bytes from 172.16.203.1: icmp_seq=4 ttl=64 time=18.4 ms
80 bytes from 172.16.203.1: icmp_seq=5 ttl=64 time=14.2 ms

--- 172.16.203.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 45ms
rtt min/avg/max/mdev = 8.867/12.524/18.445/3.598 ms, pipe 2, ipg/ewma 11.370/10.866 ms
spine-1#
spine-1#ping 172.17.203.1 source lo1
PING 172.17.203.1 (172.17.203.1) from 172.16.101.1 : 72(100) bytes of data.
80 bytes from 172.17.203.1: icmp_seq=1 ttl=64 time=21.6 ms
80 bytes from 172.17.203.1: icmp_seq=2 ttl=64 time=11.2 ms
80 bytes from 172.17.203.1: icmp_seq=3 ttl=64 time=9.52 ms
80 bytes from 172.17.203.1: icmp_seq=4 ttl=64 time=7.71 ms
80 bytes from 172.17.203.1: icmp_seq=5 ttl=64 time=10.7 ms

--- 172.17.203.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 73ms
rtt min/avg/max/mdev = 7.716/12.181/21.645/4.887 ms, pipe 2, ipg/ewma 18.465/16.732 ms
spine-1#
```

**Коммутатор spine-2**

```
spine-2#
spine-2#ping 172.16.201.1 source lo1
PING 172.16.201.1 (172.16.201.1) from 172.16.102.1 : 72(100) bytes of data.
80 bytes from 172.16.201.1: icmp_seq=1 ttl=64 time=9.82 ms
80 bytes from 172.16.201.1: icmp_seq=2 ttl=64 time=9.03 ms
80 bytes from 172.16.201.1: icmp_seq=3 ttl=64 time=7.92 ms
80 bytes from 172.16.201.1: icmp_seq=4 ttl=64 time=12.4 ms
80 bytes from 172.16.201.1: icmp_seq=5 ttl=64 time=17.6 ms

--- 172.16.201.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 48ms
rtt min/avg/max/mdev = 7.921/11.361/17.630/3.465 ms, pipe 2, ipg/ewma 12.115/10.832 ms
spine-2#
spine-2#ping 172.17.201.1 source lo1
PING 172.17.201.1 (172.17.201.1) from 172.16.102.1 : 72(100) bytes of data.
80 bytes from 172.17.201.1: icmp_seq=1 ttl=64 time=8.06 ms
80 bytes from 172.17.201.1: icmp_seq=2 ttl=64 time=6.91 ms
80 bytes from 172.17.201.1: icmp_seq=3 ttl=64 time=6.95 ms
80 bytes from 172.17.201.1: icmp_seq=4 ttl=64 time=6.56 ms
80 bytes from 172.17.201.1: icmp_seq=5 ttl=64 time=8.54 ms

--- 172.17.201.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 46ms
rtt min/avg/max/mdev = 6.563/7.408/8.547/0.764 ms, ipg/ewma 11.731/7.758 ms
spine-2#
spine-2#
spine-2#
spine-2#ping 172.16.202.1 source lo1
PING 172.16.202.1 (172.16.202.1) from 172.16.102.1 : 72(100) bytes of data.
80 bytes from 172.16.202.1: icmp_seq=1 ttl=64 time=13.3 ms
80 bytes from 172.16.202.1: icmp_seq=2 ttl=64 time=12.8 ms
80 bytes from 172.16.202.1: icmp_seq=3 ttl=64 time=7.63 ms
80 bytes from 172.16.202.1: icmp_seq=4 ttl=64 time=7.66 ms
80 bytes from 172.16.202.1: icmp_seq=5 ttl=64 time=7.55 ms

--- 172.16.202.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 60ms
rtt min/avg/max/mdev = 7.554/9.808/13.323/2.685 ms, ipg/ewma 15.180/11.400 ms
spine-2#
spine-2#ping 172.17.202.1 source lo1
PING 172.17.202.1 (172.17.202.1) from 172.16.102.1 : 72(100) bytes of data.
80 bytes from 172.17.202.1: icmp_seq=1 ttl=64 time=13.9 ms
80 bytes from 172.17.202.1: icmp_seq=2 ttl=64 time=15.3 ms
80 bytes from 172.17.202.1: icmp_seq=3 ttl=64 time=14.7 ms
80 bytes from 172.17.202.1: icmp_seq=4 ttl=64 time=15.5 ms
80 bytes from 172.17.202.1: icmp_seq=5 ttl=64 time=21.8 ms

--- 172.17.202.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 56ms
rtt min/avg/max/mdev = 13.926/16.294/21.838/2.833 ms, pipe 2, ipg/ewma 14.191/15.296 ms
spine-2#
spine-2#
spine-2#
spine-2#ping 172.16.203.1 source lo1
PING 172.16.203.1 (172.16.203.1) from 172.16.102.1 : 72(100) bytes of data.
80 bytes from 172.16.203.1: icmp_seq=1 ttl=64 time=9.56 ms
80 bytes from 172.16.203.1: icmp_seq=2 ttl=64 time=7.85 ms
80 bytes from 172.16.203.1: icmp_seq=3 ttl=64 time=7.34 ms
80 bytes from 172.16.203.1: icmp_seq=4 ttl=64 time=10.0 ms
80 bytes from 172.16.203.1: icmp_seq=5 ttl=64 time=8.28 ms

--- 172.16.203.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 50ms
rtt min/avg/max/mdev = 7.344/8.610/10.014/1.021 ms, ipg/ewma 12.707/9.096 ms
spine-2#
spine-2#ping 172.17.203.1 source lo1
PING 172.17.203.1 (172.17.203.1) from 172.16.102.1 : 72(100) bytes of data.
80 bytes from 172.17.203.1: icmp_seq=1 ttl=64 time=7.94 ms
80 bytes from 172.17.203.1: icmp_seq=2 ttl=64 time=8.73 ms
80 bytes from 172.17.203.1: icmp_seq=3 ttl=64 time=8.74 ms
80 bytes from 172.17.203.1: icmp_seq=4 ttl=64 time=8.95 ms
80 bytes from 172.17.203.1: icmp_seq=5 ttl=64 time=13.6 ms

--- 172.17.203.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 41ms
rtt min/avg/max/mdev = 7.943/9.611/13.685/2.069 ms, ipg/ewma 10.303/8.914 ms
spine-2#
spine-2#
```


**Коммутатор leaf-1**

```
leaf-1#
leaf-1#ping 172.16.101.1 source lo1
PING 172.16.101.1 (172.16.101.1) from 172.16.201.1 : 72(100) bytes of data.
80 bytes from 172.16.101.1: icmp_seq=1 ttl=64 time=11.2 ms
80 bytes from 172.16.101.1: icmp_seq=2 ttl=64 time=7.01 ms
80 bytes from 172.16.101.1: icmp_seq=3 ttl=64 time=6.51 ms
80 bytes from 172.16.101.1: icmp_seq=4 ttl=64 time=6.42 ms
80 bytes from 172.16.101.1: icmp_seq=5 ttl=64 time=6.96 ms

--- 172.16.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 53ms
rtt min/avg/max/mdev = 6.429/7.638/11.260/1.826 ms, ipg/ewma 13.292/9.386 ms
leaf-1#
leaf-1#ping 172.17.101.1 source lo1
PING 172.17.101.1 (172.17.101.1) from 172.16.201.1 : 72(100) bytes of data.
80 bytes from 172.17.101.1: icmp_seq=1 ttl=64 time=8.55 ms
80 bytes from 172.17.101.1: icmp_seq=2 ttl=64 time=9.51 ms
80 bytes from 172.17.101.1: icmp_seq=3 ttl=64 time=8.55 ms
80 bytes from 172.17.101.1: icmp_seq=4 ttl=64 time=7.74 ms
80 bytes from 172.17.101.1: icmp_seq=5 ttl=64 time=8.32 ms

--- 172.17.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 44ms
rtt min/avg/max/mdev = 7.747/8.538/9.517/0.576 ms, ipg/ewma 11.140/8.516 ms
leaf-1#
leaf-1#
leaf-1#
leaf-1#ping 172.16.102.1 source lo1
PING 172.16.102.1 (172.16.102.1) from 172.16.201.1 : 72(100) bytes of data.
80 bytes from 172.16.102.1: icmp_seq=1 ttl=64 time=8.65 ms
80 bytes from 172.16.102.1: icmp_seq=2 ttl=64 time=11.9 ms
80 bytes from 172.16.102.1: icmp_seq=3 ttl=64 time=17.7 ms
80 bytes from 172.16.102.1: icmp_seq=4 ttl=64 time=14.4 ms
80 bytes from 172.16.102.1: icmp_seq=5 ttl=64 time=17.0 ms

--- 172.16.102.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 43ms
rtt min/avg/max/mdev = 8.653/13.978/17.751/3.368 ms, pipe 2, ipg/ewma 10.936/11.488 ms
leaf-1#
leaf-1#ping 172.17.102.1 source lo1
PING 172.17.102.1 (172.17.102.1) from 172.16.201.1 : 72(100) bytes of data.
80 bytes from 172.17.102.1: icmp_seq=1 ttl=64 time=9.05 ms
80 bytes from 172.17.102.1: icmp_seq=2 ttl=64 time=13.3 ms
80 bytes from 172.17.102.1: icmp_seq=3 ttl=64 time=9.59 ms
80 bytes from 172.17.102.1: icmp_seq=4 ttl=64 time=10.2 ms
80 bytes from 172.17.102.1: icmp_seq=5 ttl=64 time=11.8 ms

--- 172.17.102.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 52ms
rtt min/avg/max/mdev = 9.055/10.821/13.378/1.584 ms, ipg/ewma 13.169/9.946 ms
leaf-1#
leaf-1#
leaf-1#
leaf-1#ping 172.16.202.1 source lo1
PING 172.16.202.1 (172.16.202.1) from 172.16.201.1 : 72(100) bytes of data.
80 bytes from 172.16.202.1: icmp_seq=1 ttl=63 time=16.8 ms
80 bytes from 172.16.202.1: icmp_seq=2 ttl=63 time=29.1 ms
80 bytes from 172.16.202.1: icmp_seq=3 ttl=63 time=26.0 ms
80 bytes from 172.16.202.1: icmp_seq=4 ttl=63 time=15.8 ms
80 bytes from 172.16.202.1: icmp_seq=5 ttl=63 time=20.1 ms

--- 172.16.202.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 82ms
rtt min/avg/max/mdev = 15.843/21.629/29.164/5.189 ms, pipe 2, ipg/ewma 20.571/19.086 ms
leaf-1#
leaf-1#ping 172.17.202.1 source lo1
PING 172.17.202.1 (172.17.202.1) from 172.16.201.1 : 72(100) bytes of data.
80 bytes from 172.17.202.1: icmp_seq=1 ttl=63 time=17.5 ms
80 bytes from 172.17.202.1: icmp_seq=2 ttl=63 time=15.4 ms
80 bytes from 172.17.202.1: icmp_seq=3 ttl=63 time=24.4 ms
80 bytes from 172.17.202.1: icmp_seq=4 ttl=63 time=15.9 ms
80 bytes from 172.17.202.1: icmp_seq=5 ttl=63 time=25.1 ms

--- 172.17.202.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 70ms
rtt min/avg/max/mdev = 15.456/19.718/25.143/4.207 ms, pipe 2, ipg/ewma 17.727/18.825 ms
leaf-1#
leaf-1#
leaf-1#
leaf-1#ping 172.16.203.1 source lo1
PING 172.16.203.1 (172.16.203.1) from 172.16.201.1 : 72(100) bytes of data.
80 bytes from 172.16.203.1: icmp_seq=1 ttl=63 time=24.0 ms
80 bytes from 172.16.203.1: icmp_seq=2 ttl=63 time=58.2 ms
80 bytes from 172.16.203.1: icmp_seq=3 ttl=63 time=48.8 ms
80 bytes from 172.16.203.1: icmp_seq=4 ttl=63 time=31.0 ms
80 bytes from 172.16.203.1: icmp_seq=5 ttl=63 time=16.6 ms

--- 172.16.203.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 96ms
rtt min/avg/max/mdev = 16.657/35.776/58.270/15.515 ms, pipe 3, ipg/ewma 24.030/29.120 ms
leaf-1#
leaf-1#ping 172.17.203.1 source lo1
PING 172.17.203.1 (172.17.203.1) from 172.16.201.1 : 72(100) bytes of data.
80 bytes from 172.17.203.1: icmp_seq=1 ttl=63 time=19.8 ms
80 bytes from 172.17.203.1: icmp_seq=2 ttl=63 time=18.1 ms
80 bytes from 172.17.203.1: icmp_seq=3 ttl=63 time=17.4 ms
80 bytes from 172.17.203.1: icmp_seq=4 ttl=63 time=20.4 ms
80 bytes from 172.17.203.1: icmp_seq=5 ttl=63 time=19.8 ms

--- 172.17.203.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 76ms
rtt min/avg/max/mdev = 17.451/19.142/20.444/1.156 ms, pipe 2, ipg/ewma 19.048/19.526 ms
leaf-1#
leaf-1#
```

**Коммутатор leaf-2**

```
leaf-2#
leaf-2#ping 172.16.101.1 source lo1
PING 172.16.101.1 (172.16.101.1) from 172.16.202.1 : 72(100) bytes of data.
80 bytes from 172.16.101.1: icmp_seq=1 ttl=64 time=9.05 ms
80 bytes from 172.16.101.1: icmp_seq=2 ttl=64 time=8.49 ms
80 bytes from 172.16.101.1: icmp_seq=3 ttl=64 time=9.54 ms
80 bytes from 172.16.101.1: icmp_seq=4 ttl=64 time=7.02 ms
80 bytes from 172.16.101.1: icmp_seq=5 ttl=64 time=10.5 ms

--- 172.16.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 46ms
rtt min/avg/max/mdev = 7.029/8.938/10.574/1.174 ms, ipg/ewma 11.631/9.020 ms
leaf-2#
leaf-2#ping 172.17.101.1 source lo1
PING 172.17.101.1 (172.17.101.1) from 172.16.202.1 : 72(100) bytes of data.
80 bytes from 172.17.101.1: icmp_seq=1 ttl=64 time=8.55 ms
80 bytes from 172.17.101.1: icmp_seq=2 ttl=64 time=6.86 ms
80 bytes from 172.17.101.1: icmp_seq=3 ttl=64 time=13.3 ms
80 bytes from 172.17.101.1: icmp_seq=4 ttl=64 time=13.6 ms
80 bytes from 172.17.101.1: icmp_seq=5 ttl=64 time=7.69 ms

--- 172.17.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 52ms
rtt min/avg/max/mdev = 6.867/10.012/13.638/2.876 ms, pipe 2, ipg/ewma 13.220/9.316 ms
leaf-2#
leaf-2#
leaf-2#
leaf-2#ping 172.16.102.1 source lo1
PING 172.16.102.1 (172.16.102.1) from 172.16.202.1 : 72(100) bytes of data.
80 bytes from 172.16.102.1: icmp_seq=1 ttl=64 time=16.3 ms
80 bytes from 172.16.102.1: icmp_seq=2 ttl=64 time=13.3 ms
80 bytes from 172.16.102.1: icmp_seq=3 ttl=64 time=7.41 ms
80 bytes from 172.16.102.1: icmp_seq=4 ttl=64 time=7.06 ms
80 bytes from 172.16.102.1: icmp_seq=5 ttl=64 time=11.2 ms

--- 172.16.102.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 66ms
rtt min/avg/max/mdev = 7.069/11.097/16.388/3.551 ms, ipg/ewma 16.670/13.613 ms
leaf-2#
leaf-2#ping 172.17.102.1 source lo1
PING 172.17.102.1 (172.17.102.1) from 172.16.202.1 : 72(100) bytes of data.
80 bytes from 172.17.102.1: icmp_seq=1 ttl=64 time=7.85 ms
80 bytes from 172.17.102.1: icmp_seq=2 ttl=64 time=8.58 ms
80 bytes from 172.17.102.1: icmp_seq=3 ttl=64 time=14.8 ms
80 bytes from 172.17.102.1: icmp_seq=4 ttl=64 time=12.5 ms
80 bytes from 172.17.102.1: icmp_seq=5 ttl=64 time=12.2 ms

--- 172.17.102.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 41ms
rtt min/avg/max/mdev = 7.854/11.203/14.844/2.611 ms, pipe 2, ipg/ewma 10.271/9.639 ms
leaf-2#
leaf-2#
leaf-2#
leaf-2#ping 172.16.201.1 source lo1
PING 172.16.201.1 (172.16.201.1) from 172.16.202.1 : 72(100) bytes of data.
80 bytes from 172.16.201.1: icmp_seq=1 ttl=63 time=17.1 ms
80 bytes from 172.16.201.1: icmp_seq=2 ttl=63 time=15.9 ms
80 bytes from 172.16.201.1: icmp_seq=3 ttl=63 time=14.5 ms
80 bytes from 172.16.201.1: icmp_seq=4 ttl=63 time=15.6 ms
80 bytes from 172.16.201.1: icmp_seq=5 ttl=63 time=15.3 ms

--- 172.16.201.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 74ms
rtt min/avg/max/mdev = 14.599/15.732/17.138/0.840 ms, pipe 2, ipg/ewma 18.597/16.405 ms
leaf-2#
leaf-2#ping 172.17.201.1 source lo1
PING 172.17.201.1 (172.17.201.1) from 172.16.202.1 : 72(100) bytes of data.
80 bytes from 172.17.201.1: icmp_seq=1 ttl=63 time=25.6 ms
80 bytes from 172.17.201.1: icmp_seq=2 ttl=63 time=87.0 ms
80 bytes from 172.17.201.1: icmp_seq=3 ttl=63 time=82.7 ms
80 bytes from 172.17.201.1: icmp_seq=4 ttl=63 time=75.2 ms

--- 172.17.201.1 ping statistics ---
5 packets transmitted, 4 received, 20% packet loss, time 78ms
rtt min/avg/max/mdev = 25.686/67.661/87.045/24.604 ms, pipe 4, ipg/ewma 19.583/43.985 ms
leaf-2#
leaf-2#
leaf-2#
leaf-2#ping 172.16.203.1 source lo1
PING 172.16.203.1 (172.16.203.1) from 172.16.202.1 : 72(100) bytes of data.
80 bytes from 172.16.203.1: icmp_seq=1 ttl=63 time=38.1 ms
80 bytes from 172.16.203.1: icmp_seq=2 ttl=63 time=40.9 ms
80 bytes from 172.16.203.1: icmp_seq=3 ttl=63 time=38.3 ms
80 bytes from 172.16.203.1: icmp_seq=4 ttl=63 time=37.5 ms
80 bytes from 172.16.203.1: icmp_seq=5 ttl=63 time=16.4 ms

--- 172.16.203.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 77ms
rtt min/avg/max/mdev = 16.440/34.306/40.991/9.012 ms, pipe 4, ipg/ewma 19.267/35.646 ms
leaf-2#
leaf-2#ping 172.17.203.1 source lo1
PING 172.17.203.1 (172.17.203.1) from 172.16.202.1 : 72(100) bytes of data.
80 bytes from 172.17.203.1: icmp_seq=1 ttl=63 time=18.4 ms
80 bytes from 172.17.203.1: icmp_seq=2 ttl=63 time=19.1 ms
80 bytes from 172.17.203.1: icmp_seq=3 ttl=63 time=23.3 ms
80 bytes from 172.17.203.1: icmp_seq=4 ttl=63 time=16.0 ms
80 bytes from 172.17.203.1: icmp_seq=5 ttl=63 time=14.0 ms

--- 172.17.203.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 76ms
rtt min/avg/max/mdev = 14.004/18.180/23.358/3.159 ms, pipe 2, ipg/ewma 19.169/18.131 ms
leaf-2#
leaf-2#
```

**Коммутатор leaf-3**

```
leaf-3#ping 172.16.101.1 source lo1
PING 172.16.101.1 (172.16.101.1) from 172.16.203.1 : 72(100) bytes of data.
80 bytes from 172.16.101.1: icmp_seq=1 ttl=64 time=8.89 ms
80 bytes from 172.16.101.1: icmp_seq=2 ttl=64 time=14.5 ms
80 bytes from 172.16.101.1: icmp_seq=3 ttl=64 time=8.39 ms
80 bytes from 172.16.101.1: icmp_seq=4 ttl=64 time=12.9 ms
80 bytes from 172.16.101.1: icmp_seq=5 ttl=64 time=10.7 ms

--- 172.16.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 49ms
rtt min/avg/max/mdev = 8.399/11.114/14.515/2.343 ms, pipe 2, ipg/ewma 12.483/10.001 ms
leaf-3#
leaf-3#ping 172.17.101.1 source lo1
PING 172.17.101.1 (172.17.101.1) from 172.16.203.1 : 72(100) bytes of data.
80 bytes from 172.17.101.1: icmp_seq=1 ttl=64 time=8.27 ms
80 bytes from 172.17.101.1: icmp_seq=2 ttl=64 time=7.04 ms
80 bytes from 172.17.101.1: icmp_seq=3 ttl=64 time=8.34 ms
80 bytes from 172.17.101.1: icmp_seq=4 ttl=64 time=9.03 ms
80 bytes from 172.17.101.1: icmp_seq=5 ttl=64 time=8.81 ms

--- 172.17.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 40ms
rtt min/avg/max/mdev = 7.046/8.302/9.036/0.689 ms, ipg/ewma 10.238/8.327 ms
leaf-3#
leaf-3#
leaf-3#
leaf-3#ping 172.16.102.1 source lo1
PING 172.16.102.1 (172.16.102.1) from 172.16.203.1 : 72(100) bytes of data.
80 bytes from 172.16.102.1: icmp_seq=1 ttl=64 time=8.71 ms
80 bytes from 172.16.102.1: icmp_seq=2 ttl=64 time=8.99 ms
80 bytes from 172.16.102.1: icmp_seq=3 ttl=64 time=9.48 ms
80 bytes from 172.16.102.1: icmp_seq=4 ttl=64 time=11.8 ms
80 bytes from 172.16.102.1: icmp_seq=5 ttl=64 time=15.3 ms

--- 172.16.102.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 47ms
rtt min/avg/max/mdev = 8.713/10.875/15.318/2.486 ms, pipe 2, ipg/ewma 11.754/9.981 ms
leaf-3#
leaf-3#ping 172.17.102.1 source lo1
PING 172.17.102.1 (172.17.102.1) from 172.16.203.1 : 72(100) bytes of data.
80 bytes from 172.17.102.1: icmp_seq=1 ttl=64 time=9.10 ms
80 bytes from 172.17.102.1: icmp_seq=2 ttl=64 time=11.3 ms
80 bytes from 172.17.102.1: icmp_seq=3 ttl=64 time=7.73 ms
80 bytes from 172.17.102.1: icmp_seq=4 ttl=64 time=7.77 ms
80 bytes from 172.17.102.1: icmp_seq=5 ttl=64 time=10.9 ms

--- 172.17.102.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 47ms
rtt min/avg/max/mdev = 7.734/9.378/11.315/1.526 ms, ipg/ewma 11.988/9.245 ms
leaf-3#
leaf-3#
leaf-3#
leaf-3#ping 172.16.201.1 source lo1
PING 172.16.201.1 (172.16.201.1) from 172.16.203.1 : 72(100) bytes of data.
80 bytes from 172.16.201.1: icmp_seq=1 ttl=63 time=27.5 ms
80 bytes from 172.16.201.1: icmp_seq=2 ttl=63 time=18.2 ms
80 bytes from 172.16.201.1: icmp_seq=3 ttl=63 time=16.9 ms
80 bytes from 172.16.201.1: icmp_seq=4 ttl=63 time=16.6 ms
80 bytes from 172.16.201.1: icmp_seq=5 ttl=63 time=17.1 ms

--- 172.16.201.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 95ms
rtt min/avg/max/mdev = 16.645/19.304/27.541/4.158 ms, pipe 2, ipg/ewma 23.893/23.256 ms
leaf-3#
leaf-3#ping 172.17.201.1 source lo1
PING 172.17.201.1 (172.17.201.1) from 172.16.203.1 : 72(100) bytes of data.
80 bytes from 172.17.201.1: icmp_seq=1 ttl=63 time=17.2 ms
80 bytes from 172.17.201.1: icmp_seq=2 ttl=63 time=16.5 ms
80 bytes from 172.17.201.1: icmp_seq=3 ttl=63 time=41.5 ms
80 bytes from 172.17.201.1: icmp_seq=4 ttl=63 time=38.7 ms

--- 172.17.201.1 ping statistics ---
5 packets transmitted, 4 received, 20% packet loss, time 69ms
rtt min/avg/max/mdev = 16.557/28.519/41.554/11.666 ms, pipe 3, ipg/ewma 17.251/22.522 ms
leaf-3#
leaf-3#
leaf-3#
leaf-3#ping 172.16.202.1 source lo1
PING 172.16.202.1 (172.16.202.1) from 172.16.203.1 : 72(100) bytes of data.
80 bytes from 172.16.202.1: icmp_seq=1 ttl=63 time=16.1 ms
80 bytes from 172.16.202.1: icmp_seq=2 ttl=63 time=15.8 ms
80 bytes from 172.16.202.1: icmp_seq=3 ttl=63 time=31.3 ms
80 bytes from 172.16.202.1: icmp_seq=4 ttl=63 time=20.9 ms
80 bytes from 172.16.202.1: icmp_seq=5 ttl=63 time=20.2 ms

--- 172.16.202.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 66ms
rtt min/avg/max/mdev = 15.871/20.926/31.311/5.591 ms, pipe 2, ipg/ewma 16.632/18.641 ms
leaf-3#
leaf-3#ping 172.17.202.1 source lo1
PING 172.17.202.1 (172.17.202.1) from 172.16.203.1 : 72(100) bytes of data.
80 bytes from 172.17.202.1: icmp_seq=1 ttl=63 time=15.7 ms
80 bytes from 172.17.202.1: icmp_seq=2 ttl=63 time=18.2 ms
80 bytes from 172.17.202.1: icmp_seq=3 ttl=63 time=20.3 ms
80 bytes from 172.17.202.1: icmp_seq=4 ttl=63 time=21.8 ms
80 bytes from 172.17.202.1: icmp_seq=5 ttl=63 time=14.3 ms

--- 172.17.202.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 67ms
rtt min/avg/max/mdev = 14.396/18.107/21.852/2.775 ms, pipe 2, ipg/ewma 16.994/16.881 ms
leaf-3#
```
