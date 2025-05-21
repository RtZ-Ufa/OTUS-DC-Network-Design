#  Домашнее задание №4
# Underlay. BGP
## по теме №8 "Построение Underlay сети(eBGP)"
### Цель: Настроить BGP для Underlay сети
### Задачи:
+ настроить BGP в Underlay сети, для IP связанности между всеми сетевыми устройствами;
+ зафиксировать в документации - план работы, адресное пространство, схему сети, конфигурацию устройств;
+ проверить IP-связанность между устройствами в BGP домене.
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

Для коммутаторов spine выбрана AS 65000, для коммутаторов leaf - соответственно по их номерам 650001, 65002, 65003. При добавлениии новых leaf нумерация будет продолжаться.
3. Настройки BGP приложены в файлах leaf-x.txt и spine-x.txt (базовые настройки приведены в ДЗ №1). Далее выборочно приведены настройки оборудования:

**Коммутатор spine-1**

```
spine-1#show run
...
hostname spine-1
!
spanning-tree mode mstp
!
interface Ethernet1
   description -L- leaf-1
   no switchport
   ip address 172.18.1.0/31
   isis enable 10
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 bMtaY5EaFQ/hSDpSm56UHg==
!
interface Ethernet2
   description -L- leaf-2
   no switchport
   ip address 172.18.1.2/31
   isis enable 10
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 bMtaY5EaFQ/hSDpSm56UHg==
!
interface Ethernet3
   description -L- leaf-3
   no switchport
   ip address 172.18.1.4/31
   isis enable 10
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 bMtaY5EaFQ/hSDpSm56UHg==
!
...
!
interface Loopback1
   ip address 172.16.101.1/32
   isis enable 10
!
interface Loopback2
   ip address 172.17.101.1/32
   isis enable 10
!
interface Management1
!
ip routing
!
router isis 10
   net 49.0001.0010.0100.1001.00
   is-type level-1
   log-adjacency-changes
   authentication mode md5
   authentication key 7 bMtaY5EaFQ/hSDpSm56UHg==
   !
   address-family ipv4 unicast
      bfd all-interfaces
!
end
spine-1#
```

**Коммутатор spine-2**

```
spine-2#show run
...
hostname spine-2
!
spanning-tree mode mstp
!
interface Ethernet1
   description -L- leaf-1
   no switchport
   ip address 172.18.2.0/31
   isis enable 10
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 bMtaY5EaFQ/hSDpSm56UHg==
!
interface Ethernet2
   description -L- leaf-2
   no switchport
   ip address 172.18.2.2/31
   isis enable 10
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 bMtaY5EaFQ/hSDpSm56UHg==
!
interface Ethernet3
   description -L- leaf-3
   no switchport
   ip address 172.18.2.4/31
   isis enable 10
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 bMtaY5EaFQ/hSDpSm56UHg==
!
...
!
interface Loopback1
   ip address 172.16.102.1/32
   isis enable 10
!
interface Loopback2
   ip address 172.17.102.1/32
   isis enable 10
!
interface Management1
!
ip routing
!
router isis 10
   net 49.0001.0010.0100.1002.00
   is-type level-1
   log-adjacency-changes
   authentication mode md5
   authentication key 7 bMtaY5EaFQ/hSDpSm56UHg==
   !
   address-family ipv4 unicast
      bfd all-interfaces
!
end
spine-2#
```

**Коммутатор leaf-1**

```
leaf-1#
leaf-1#show run
...
hostname leaf-1
!
spanning-tree mode mstp
!
interface Ethernet1
   description -S- spine-1
   no switchport
   ip address 172.18.1.1/31
   isis enable 10
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 bMtaY5EaFQ/hSDpSm56UHg==
!
interface Ethernet2
   description -S- spine-2
   no switchport
   ip address 172.18.2.1/31
   isis enable 10
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 bMtaY5EaFQ/hSDpSm56UHg==
!
...
!
interface Loopback1
   ip address 172.16.201.1/32
   isis enable 10
!
interface Loopback2
   ip address 172.17.201.1/32
   isis enable 10
!
interface Management1
!
ip routing
!
router isis 10
   net 49.0001.0010.0100.2001.00
   is-type level-1
   log-adjacency-changes
   authentication mode md5
   authentication key 7 bMtaY5EaFQ/hSDpSm56UHg==
   !
   address-family ipv4 unicast
      bfd all-interfaces
!
end
leaf-1#
```

**Коммутатор leaf-2**

```
leaf-2#show run
...
hostname leaf-2
!
spanning-tree mode mstp
!
interface Ethernet1
   description -S- spine-1
   no switchport
   ip address 172.18.1.3/31
   isis enable 10
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 bMtaY5EaFQ/hSDpSm56UHg==
!
interface Ethernet2
   description -S- spine-2
   no switchport
   ip address 172.18.2.3/31
   isis enable 10
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 bMtaY5EaFQ/hSDpSm56UHg==
!
...
!
interface Loopback1
   ip address 172.16.202.1/32
   isis enable 10
!
interface Loopback2
   ip address 172.17.202.1/32
   isis enable 10
!
interface Management1
!
ip routing
!
router isis 10
   net 49.0001.0010.0100.2002.00
   is-type level-1
   log-adjacency-changes
   authentication mode md5
   authentication key 7 bMtaY5EaFQ/hSDpSm56UHg==
   !
   address-family ipv4 unicast
      bfd all-interfaces
!
end
leaf-2#
```

**Коммутатор leaf-3**

```
leaf-3#show run
...
hostname leaf-3
!
spanning-tree mode mstp
!
interface Ethernet1
   description -S- spine-1
   no switchport
   ip address 172.18.1.5/31
   isis enable 10
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 bMtaY5EaFQ/hSDpSm56UHg==
!
interface Ethernet2
   description -S- spine-2
   no switchport
   ip address 172.18.2.5/31
   isis enable 10
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 bMtaY5EaFQ/hSDpSm56UHg==
!
...
!
interface Loopback1
   ip address 172.16.203.1/32
   isis enable 10
!
interface Loopback2
   ip address 172.17.203.1/32
   isis enable 10
!
interface Management1
!
ip routing
!
router isis 10
   net 49.0001.0010.0100.2003.00
   is-type level-1
   log-adjacency-changes
   authentication mode md5
   authentication key 7 bMtaY5EaFQ/hSDpSm56UHg==
   !
   address-family ipv4 unicast
      bfd all-interfaces
!
end
leaf-3#
```

4. Таблицы маршрутизации на коммутаторах:

**Коммутатор spine-1**

```
spine-1#show ip route

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
 I L1     172.16.102.1/32 [115/30] via 172.18.1.1, Ethernet1
                                   via 172.18.1.3, Ethernet2
                                   via 172.18.1.5, Ethernet3
 I L1     172.16.201.1/32 [115/20] via 172.18.1.1, Ethernet1
 I L1     172.16.202.1/32 [115/20] via 172.18.1.3, Ethernet2
 I L1     172.16.203.1/32 [115/20] via 172.18.1.5, Ethernet3
 C        172.17.101.1/32 is directly connected, Loopback2
 I L1     172.17.102.1/32 [115/30] via 172.18.1.1, Ethernet1
                                   via 172.18.1.3, Ethernet2
                                   via 172.18.1.5, Ethernet3
 I L1     172.17.201.1/32 [115/20] via 172.18.1.1, Ethernet1
 I L1     172.17.202.1/32 [115/20] via 172.18.1.3, Ethernet2
 I L1     172.17.203.1/32 [115/20] via 172.18.1.5, Ethernet3
 C        172.18.1.0/31 is directly connected, Ethernet1
 C        172.18.1.2/31 is directly connected, Ethernet2
 C        172.18.1.4/31 is directly connected, Ethernet3
 I L1     172.18.2.0/31 [115/20] via 172.18.1.1, Ethernet1
 I L1     172.18.2.2/31 [115/20] via 172.18.1.3, Ethernet2
 I L1     172.18.2.4/31 [115/20] via 172.18.1.5, Ethernet3

spine-1#
```

**Коммутатор spine-2**

```
spine-2#show ip route

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

 I L1     172.16.101.1/32 [115/30] via 172.18.2.1, Ethernet1
                                   via 172.18.2.3, Ethernet2
                                   via 172.18.2.5, Ethernet3
 C        172.16.102.1/32 is directly connected, Loopback1
 I L1     172.16.201.1/32 [115/20] via 172.18.2.1, Ethernet1
 I L1     172.16.202.1/32 [115/20] via 172.18.2.3, Ethernet2
 I L1     172.16.203.1/32 [115/20] via 172.18.2.5, Ethernet3
 I L1     172.17.101.1/32 [115/30] via 172.18.2.1, Ethernet1
                                   via 172.18.2.3, Ethernet2
                                   via 172.18.2.5, Ethernet3
 C        172.17.102.1/32 is directly connected, Loopback2
 I L1     172.17.201.1/32 [115/20] via 172.18.2.1, Ethernet1
 I L1     172.17.202.1/32 [115/20] via 172.18.2.3, Ethernet2
 I L1     172.17.203.1/32 [115/20] via 172.18.2.5, Ethernet3
 I L1     172.18.1.0/31 [115/20] via 172.18.2.1, Ethernet1
 I L1     172.18.1.2/31 [115/20] via 172.18.2.3, Ethernet2
 I L1     172.18.1.4/31 [115/20] via 172.18.2.5, Ethernet3
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

 I L1     172.16.101.1/32 [115/20] via 172.18.1.0, Ethernet1
 I L1     172.16.102.1/32 [115/20] via 172.18.2.0, Ethernet2
 C        172.16.201.1/32 is directly connected, Loopback1
 I L1     172.16.202.1/32 [115/30] via 172.18.1.0, Ethernet1
                                   via 172.18.2.0, Ethernet2
 I L1     172.16.203.1/32 [115/30] via 172.18.1.0, Ethernet1
                                   via 172.18.2.0, Ethernet2
 I L1     172.17.101.1/32 [115/20] via 172.18.1.0, Ethernet1
 I L1     172.17.102.1/32 [115/20] via 172.18.2.0, Ethernet2
 C        172.17.201.1/32 is directly connected, Loopback2
 I L1     172.17.202.1/32 [115/30] via 172.18.1.0, Ethernet1
                                   via 172.18.2.0, Ethernet2
 I L1     172.17.203.1/32 [115/30] via 172.18.1.0, Ethernet1
                                   via 172.18.2.0, Ethernet2
 C        172.18.1.0/31 is directly connected, Ethernet1
 I L1     172.18.1.2/31 [115/20] via 172.18.1.0, Ethernet1
 I L1     172.18.1.4/31 [115/20] via 172.18.1.0, Ethernet1
 C        172.18.2.0/31 is directly connected, Ethernet2
 I L1     172.18.2.2/31 [115/20] via 172.18.2.0, Ethernet2
 I L1     172.18.2.4/31 [115/20] via 172.18.2.0, Ethernet2
leaf-1#
```

**Коммутатор leaf-2**

```
leaf-2#show ip route

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

 I L1     172.16.101.1/32 [115/20] via 172.18.1.2, Ethernet1
 I L1     172.16.102.1/32 [115/20] via 172.18.2.2, Ethernet2
 I L1     172.16.201.1/32 [115/30] via 172.18.1.2, Ethernet1
                                   via 172.18.2.2, Ethernet2
 C        172.16.202.1/32 is directly connected, Loopback1
 I L1     172.16.203.1/32 [115/30] via 172.18.1.2, Ethernet1
                                   via 172.18.2.2, Ethernet2
 I L1     172.17.101.1/32 [115/20] via 172.18.1.2, Ethernet1
 I L1     172.17.102.1/32 [115/20] via 172.18.2.2, Ethernet2
 I L1     172.17.201.1/32 [115/30] via 172.18.1.2, Ethernet1
                                   via 172.18.2.2, Ethernet2
 C        172.17.202.1/32 is directly connected, Loopback2
 I L1     172.17.203.1/32 [115/30] via 172.18.1.2, Ethernet1
                                   via 172.18.2.2, Ethernet2
 I L1     172.18.1.0/31 [115/20] via 172.18.1.2, Ethernet1
 C        172.18.1.2/31 is directly connected, Ethernet1
 I L1     172.18.1.4/31 [115/20] via 172.18.1.2, Ethernet1
 I L1     172.18.2.0/31 [115/20] via 172.18.2.2, Ethernet2
 C        172.18.2.2/31 is directly connected, Ethernet2
 I L1     172.18.2.4/31 [115/20] via 172.18.2.2, Ethernet2
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

 I L1     172.16.101.1/32 [115/20] via 172.18.1.4, Ethernet1
 I L1     172.16.102.1/32 [115/20] via 172.18.2.4, Ethernet2
 I L1     172.16.201.1/32 [115/30] via 172.18.1.4, Ethernet1
                                   via 172.18.2.4, Ethernet2
 I L1     172.16.202.1/32 [115/30] via 172.18.1.4, Ethernet1
                                   via 172.18.2.4, Ethernet2
 C        172.16.203.1/32 is directly connected, Loopback1
 I L1     172.17.101.1/32 [115/20] via 172.18.1.4, Ethernet1
 I L1     172.17.102.1/32 [115/20] via 172.18.2.4, Ethernet2
 I L1     172.17.201.1/32 [115/30] via 172.18.1.4, Ethernet1
                                   via 172.18.2.4, Ethernet2
 I L1     172.17.202.1/32 [115/30] via 172.18.1.4, Ethernet1
                                   via 172.18.2.4, Ethernet2
 C        172.17.203.1/32 is directly connected, Loopback2
 I L1     172.18.1.0/31 [115/20] via 172.18.1.4, Ethernet1
 I L1     172.18.1.2/31 [115/20] via 172.18.1.4, Ethernet1
 C        172.18.1.4/31 is directly connected, Ethernet1
 I L1     172.18.2.0/31 [115/20] via 172.18.2.4, Ethernet2
 I L1     172.18.2.2/31 [115/20] via 172.18.2.4, Ethernet2
 C        172.18.2.4/31 is directly connected, Ethernet2
leaf-3#
```

5. Таблицы соседства в ISIS:

**Коммутатор spine-1**

```
spine-1#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
10        default  leaf-1           L1   Ethernet1          P2P               UP    26          0F
10        default  leaf-2           L1   Ethernet2          P2P               UP    28          0F
10        default  leaf-3           L1   Ethernet3          P2P               UP    26          0F
spine-1# 
```

**Коммутатор spine-2**

```
spine-2#sh isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
10        default  leaf-1           L1   Ethernet1          P2P               UP    27          10
10        default  leaf-2           L1   Ethernet2          P2P               UP    23          10
10        default  leaf-3           L1   Ethernet3          P2P               UP    26          10
spine-2#
```

**Коммутатор leaf-1**

```
leaf-1#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
10        default  spine-1          L1   Ethernet1          P2P               UP    27          0F
10        default  spine-2          L1   Ethernet2          P2P               UP    25          0F
leaf-1#
```

**Коммутатор leaf-2**

```
leaf-2#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
10        default  spine-1          L1   Ethernet1          P2P               UP    27          10
10        default  spine-2          L1   Ethernet2          P2P               UP    26          10
leaf-2#
```

**Коммутатор leaf-3**

```
leaf-3#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
10        default  spine-1          L1   Ethernet1          P2P               UP    29          11
10        default  spine-2          L1   Ethernet2          P2P               UP    28          11
leaf-3#
```

6. IS-IS Level 1 Link State Database

**Коммутатор spine-1**

```
spine-1#show isis database

IS-IS Instance: 10 VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    spine-1.00-00                19   6384   642    179 L1 <>
    spine-2.00-00                19  16930   606    179 L1 <>
    leaf-1.00-00                 17  30980   849    154 L1 <>
    leaf-2.00-00                 17  61916   528    154 L1 <>
    leaf-3.00-00                 16  41578   769    154 L1 <>
spine-1#
```

**Коммутатор spine-2**

```
spine-2#show isis database

IS-IS Instance: 10 VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    spine-1.00-00                19   6384   586    179 L1 <>
    spine-2.00-00                19  16930   549    179 L1 <>
    leaf-1.00-00                 17  30980   792    154 L1 <>
    leaf-2.00-00                 18  61959  1172    154 L1 <>
    leaf-3.00-00                 16  41578   712    154 L1 <>
spine-2#
```

**Коммутатор leaf-1**

```
leaf-1#
leaf-1#show isis database

IS-IS Instance: 10 VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    spine-1.00-00                19   6384   554    179 L1 <>
    spine-2.00-00                19  16930   517    179 L1 <>
    leaf-1.00-00                 17  30980   760    154 L1 <>
    leaf-2.00-00                 18  61959  1140    154 L1 <>
    leaf-3.00-00                 16  41578   680    154 L1 <>
leaf-1#
```

**Коммутатор leaf-2**

```
leaf-2#show isis database

IS-IS Instance: 10 VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    spine-1.00-00                19   6384   530    179 L1 <>
    spine-2.00-00                20  19213  1198    179 L1 <>
    leaf-1.00-00                 17  30980   737    154 L1 <>
    leaf-2.00-00                 18  61959  1116    154 L1 <>
    leaf-3.00-00                 16  41578   657    154 L1 <>
leaf-2#
```

**Коммутатор leaf-3**

```
leaf-3#show isis database

IS-IS Instance: 10 VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    spine-1.00-00                20  53436  1166    179 L1 <>
    spine-2.00-00                20  19213  1155    179 L1 <>
    leaf-1.00-00                 17  30980   694    154 L1 <>
    leaf-2.00-00                 18  61959  1074    154 L1 <>
    leaf-3.00-00                 16  41578   614    154 L1 <>
leaf-3#
```

7. Проверка связности между устройствами в ISIS домене проводилась утилитой **ping**. Проверлась связность от текущего коммутатора до интерфейсов Loopback1 и Loopback2 других коммутаторов.

**Коммутатор spine-1**

```
spine-1#ping 172.16.102.1
PING 172.16.102.1 (172.16.102.1) 72(100) bytes of data.
80 bytes from 172.16.102.1: icmp_seq=1 ttl=63 time=46.2 ms
80 bytes from 172.16.102.1: icmp_seq=2 ttl=63 time=50.2 ms
80 bytes from 172.16.102.1: icmp_seq=3 ttl=63 time=46.5 ms
80 bytes from 172.16.102.1: icmp_seq=4 ttl=63 time=37.5 ms
80 bytes from 172.16.102.1: icmp_seq=5 ttl=63 time=16.3 ms

--- 172.16.102.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 87ms
rtt min/avg/max/mdev = 16.373/39.408/50.262/12.255 ms, pipe 4, ipg/ewma 21.865/41.956 ms
spine-1#
spine-1#ping 172.17.102.1
PING 172.17.102.1 (172.17.102.1) 72(100) bytes of data.
80 bytes from 172.17.102.1: icmp_seq=1 ttl=63 time=25.2 ms
80 bytes from 172.17.102.1: icmp_seq=2 ttl=63 time=19.6 ms
80 bytes from 172.17.102.1: icmp_seq=3 ttl=63 time=20.3 ms
80 bytes from 172.17.102.1: icmp_seq=4 ttl=63 time=15.9 ms
80 bytes from 172.17.102.1: icmp_seq=5 ttl=63 time=17.6 ms

--- 172.17.102.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 88ms
rtt min/avg/max/mdev = 15.953/19.772/25.239/3.139 ms, pipe 2, ipg/ewma 22.181/22.341 ms
spine-1#
spine-1#ping 172.16.201.1
PING 172.16.201.1 (172.16.201.1) 72(100) bytes of data.
80 bytes from 172.16.201.1: icmp_seq=1 ttl=64 time=9.29 ms
80 bytes from 172.16.201.1: icmp_seq=2 ttl=64 time=7.98 ms
80 bytes from 172.16.201.1: icmp_seq=3 ttl=64 time=10.0 ms
80 bytes from 172.16.201.1: icmp_seq=4 ttl=64 time=9.47 ms
80 bytes from 172.16.201.1: icmp_seq=5 ttl=64 time=15.1 ms

--- 172.16.201.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 46ms
rtt min/avg/max/mdev = 7.980/10.379/15.111/2.463 ms, ipg/ewma 11.604/10.004 ms
spine-1#
spine-1#ping 172.17.201.1
PING 172.17.201.1 (172.17.201.1) 72(100) bytes of data.
80 bytes from 172.17.201.1: icmp_seq=1 ttl=64 time=9.99 ms
80 bytes from 172.17.201.1: icmp_seq=2 ttl=64 time=8.51 ms
80 bytes from 172.17.201.1: icmp_seq=3 ttl=64 time=12.2 ms
80 bytes from 172.17.201.1: icmp_seq=4 ttl=64 time=16.5 ms
80 bytes from 172.17.201.1: icmp_seq=5 ttl=64 time=8.71 ms

--- 172.17.201.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 57ms
rtt min/avg/max/mdev = 8.514/11.212/16.539/2.986 ms, pipe 2, ipg/ewma 14.358/10.646 ms
spine-1#
spine-1#ping 172.16.202.1
PING 172.16.202.1 (172.16.202.1) 72(100) bytes of data.
80 bytes from 172.16.202.1: icmp_seq=1 ttl=64 time=12.3 ms
80 bytes from 172.16.202.1: icmp_seq=2 ttl=64 time=22.9 ms
80 bytes from 172.16.202.1: icmp_seq=3 ttl=64 time=27.0 ms
80 bytes from 172.16.202.1: icmp_seq=4 ttl=64 time=16.2 ms
80 bytes from 172.16.202.1: icmp_seq=5 ttl=64 time=9.83 ms

--- 172.16.202.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 72ms
rtt min/avg/max/mdev = 9.838/17.701/27.026/6.431 ms, pipe 2, ipg/ewma 18.157/14.782 ms
spine-1#
spine-1#ping 172.17.202.1
PING 172.17.202.1 (172.17.202.1) 72(100) bytes of data.
80 bytes from 172.17.202.1: icmp_seq=1 ttl=64 time=15.0 ms
80 bytes from 172.17.202.1: icmp_seq=2 ttl=64 time=7.92 ms
80 bytes from 172.17.202.1: icmp_seq=3 ttl=64 time=8.70 ms
80 bytes from 172.17.202.1: icmp_seq=4 ttl=64 time=8.12 ms
80 bytes from 172.17.202.1: icmp_seq=5 ttl=64 time=10.2 ms

--- 172.17.202.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 61ms
rtt min/avg/max/mdev = 7.928/10.015/15.063/2.654 ms, ipg/ewma 15.274/12.497 ms
spine-1#
spine-1#ping 172.16.203.1
PING 172.16.203.1 (172.16.203.1) 72(100) bytes of data.
80 bytes from 172.16.203.1: icmp_seq=1 ttl=64 time=12.4 ms
80 bytes from 172.16.203.1: icmp_seq=2 ttl=64 time=16.1 ms
80 bytes from 172.16.203.1: icmp_seq=3 ttl=64 time=16.7 ms
80 bytes from 172.16.203.1: icmp_seq=4 ttl=64 time=8.77 ms
80 bytes from 172.16.203.1: icmp_seq=5 ttl=64 time=10.7 ms

--- 172.16.203.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 67ms
rtt min/avg/max/mdev = 8.770/12.978/16.784/3.085 ms, pipe 2, ipg/ewma 16.935/12.575 ms
spine-1#
spine-1#ping 172.17.203.1
PING 172.17.203.1 (172.17.203.1) 72(100) bytes of data.
80 bytes from 172.17.203.1: icmp_seq=1 ttl=64 time=11.0 ms
80 bytes from 172.17.203.1: icmp_seq=2 ttl=64 time=9.54 ms
80 bytes from 172.17.203.1: icmp_seq=3 ttl=64 time=13.7 ms
80 bytes from 172.17.203.1: icmp_seq=4 ttl=64 time=16.0 ms
80 bytes from 172.17.203.1: icmp_seq=5 ttl=64 time=11.7 ms

--- 172.17.203.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 61ms
rtt min/avg/max/mdev = 9.548/12.431/16.033/2.256 ms, pipe 2, ipg/ewma 15.295/11.821 ms
spine-1#
```

**Коммутатор spine-2**

```
spine-2#ping 172.16.101.1
PING 172.16.101.1 (172.16.101.1) 72(100) bytes of data.
80 bytes from 172.16.101.1: icmp_seq=1 ttl=63 time=40.8 ms
80 bytes from 172.16.101.1: icmp_seq=2 ttl=63 time=37.7 ms
80 bytes from 172.16.101.1: icmp_seq=3 ttl=63 time=42.2 ms
80 bytes from 172.16.101.1: icmp_seq=4 ttl=63 time=47.8 ms
80 bytes from 172.16.101.1: icmp_seq=5 ttl=63 time=21.0 ms

--- 172.16.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 85ms
rtt min/avg/max/mdev = 21.027/37.933/47.805/9.063 ms, pipe 4, ipg/ewma 21.265/39.009 ms
spine-2#
spine-2#ping 172.17.101.1
PING 172.17.101.1 (172.17.101.1) 72(100) bytes of data.
80 bytes from 172.17.101.1: icmp_seq=1 ttl=63 time=63.8 ms
80 bytes from 172.17.101.1: icmp_seq=2 ttl=63 time=58.5 ms
80 bytes from 172.17.101.1: icmp_seq=3 ttl=63 time=68.9 ms
80 bytes from 172.17.101.1: icmp_seq=4 ttl=63 time=64.2 ms
80 bytes from 172.17.101.1: icmp_seq=5 ttl=63 time=62.7 ms

--- 172.17.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 49ms
rtt min/avg/max/mdev = 58.566/63.676/68.956/3.335 ms, pipe 5, ipg/ewma 12.486/63.800 ms
spine-2#
spine-2#ping 172.16.201.1
PING 172.16.201.1 (172.16.201.1) 72(100) bytes of data.
80 bytes from 172.16.201.1: icmp_seq=1 ttl=64 time=14.6 ms
80 bytes from 172.16.201.1: icmp_seq=2 ttl=64 time=15.5 ms
80 bytes from 172.16.201.1: icmp_seq=3 ttl=64 time=14.8 ms
80 bytes from 172.16.201.1: icmp_seq=4 ttl=64 time=10.4 ms
80 bytes from 172.16.201.1: icmp_seq=5 ttl=64 time=15.6 ms

--- 172.16.201.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 65ms
rtt min/avg/max/mdev = 10.405/14.208/15.645/1.945 ms, pipe 2, ipg/ewma 16.445/14.384 ms
spine-2#
spine-2#ping 172.17.201.1
PING 172.17.201.1 (172.17.201.1) 72(100) bytes of data.
80 bytes from 172.17.201.1: icmp_seq=1 ttl=64 time=9.76 ms
80 bytes from 172.17.201.1: icmp_seq=2 ttl=64 time=15.8 ms
80 bytes from 172.17.201.1: icmp_seq=3 ttl=64 time=11.9 ms
80 bytes from 172.17.201.1: icmp_seq=4 ttl=64 time=14.1 ms
80 bytes from 172.17.201.1: icmp_seq=5 ttl=64 time=9.14 ms

--- 172.17.201.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 59ms
rtt min/avg/max/mdev = 9.143/12.156/15.827/2.545 ms, pipe 2, ipg/ewma 14.946/10.876 ms
spine-2#
spine-2#ping 172.16.202.1
PING 172.16.202.1 (172.16.202.1) 72(100) bytes of data.
80 bytes from 172.16.202.1: icmp_seq=1 ttl=64 time=11.3 ms
80 bytes from 172.16.202.1: icmp_seq=2 ttl=64 time=19.5 ms
80 bytes from 172.16.202.1: icmp_seq=3 ttl=64 time=14.7 ms
80 bytes from 172.16.202.1: icmp_seq=4 ttl=64 time=11.3 ms
80 bytes from 172.16.202.1: icmp_seq=5 ttl=64 time=7.53 ms

--- 172.16.202.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 64ms
rtt min/avg/max/mdev = 7.535/12.909/19.514/4.016 ms, pipe 2, ipg/ewma 16.004/11.902 ms
spine-2#
spine-2#ping 172.17.202.1
PING 172.17.202.1 (172.17.202.1) 72(100) bytes of data.
80 bytes from 172.17.202.1: icmp_seq=1 ttl=64 time=12.4 ms
80 bytes from 172.17.202.1: icmp_seq=2 ttl=64 time=13.1 ms
80 bytes from 172.17.202.1: icmp_seq=3 ttl=64 time=9.37 ms
80 bytes from 172.17.202.1: icmp_seq=4 ttl=64 time=8.13 ms
80 bytes from 172.17.202.1: icmp_seq=5 ttl=64 time=10.2 ms

--- 172.17.202.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 59ms
rtt min/avg/max/mdev = 8.131/10.661/13.163/1.882 ms, pipe 2, ipg/ewma 14.762/11.452 ms
spine-2#
spine-2#ping 172.16.203.1
PING 172.16.203.1 (172.16.203.1) 72(100) bytes of data.
80 bytes from 172.16.203.1: icmp_seq=1 ttl=64 time=11.5 ms
80 bytes from 172.16.203.1: icmp_seq=2 ttl=64 time=15.0 ms
80 bytes from 172.16.203.1: icmp_seq=3 ttl=64 time=20.4 ms
80 bytes from 172.16.203.1: icmp_seq=4 ttl=64 time=15.1 ms
80 bytes from 172.16.203.1: icmp_seq=5 ttl=64 time=23.5 ms

--- 172.16.203.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 59ms
rtt min/avg/max/mdev = 11.575/17.167/23.509/4.259 ms, pipe 2, ipg/ewma 14.792/14.608 ms
spine-2#
spine-2#ping 172.17.203.1
PING 172.17.203.1 (172.17.203.1) 72(100) bytes of data.
80 bytes from 172.17.203.1: icmp_seq=1 ttl=64 time=12.1 ms
80 bytes from 172.17.203.1: icmp_seq=2 ttl=64 time=10.4 ms
80 bytes from 172.17.203.1: icmp_seq=3 ttl=64 time=11.1 ms
80 bytes from 172.17.203.1: icmp_seq=4 ttl=64 time=7.78 ms
80 bytes from 172.17.203.1: icmp_seq=5 ttl=64 time=8.23 ms

--- 172.17.203.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 60ms
rtt min/avg/max/mdev = 7.780/9.943/12.121/1.678 ms, ipg/ewma 15.071/10.925 ms
spine-2#
```

**Коммутатор leaf-1**

```
leaf-1#ping 172.16.101.1
PING 172.16.101.1 (172.16.101.1) 72(100) bytes of data.
80 bytes from 172.16.101.1: icmp_seq=1 ttl=64 time=12.4 ms
80 bytes from 172.16.101.1: icmp_seq=2 ttl=64 time=11.7 ms
80 bytes from 172.16.101.1: icmp_seq=3 ttl=64 time=12.2 ms
80 bytes from 172.16.101.1: icmp_seq=4 ttl=64 time=10.4 ms
80 bytes from 172.16.101.1: icmp_seq=5 ttl=64 time=8.38 ms

--- 172.16.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 68ms
rtt min/avg/max/mdev = 8.386/11.028/12.402/1.492 ms, ipg/ewma 17.203/11.608 ms
leaf-1#
leaf-1#ping 172.17.101.1
PING 172.17.101.1 (172.17.101.1) 72(100) bytes of data.
80 bytes from 172.17.101.1: icmp_seq=1 ttl=64 time=9.65 ms
80 bytes from 172.17.101.1: icmp_seq=2 ttl=64 time=8.13 ms
80 bytes from 172.17.101.1: icmp_seq=3 ttl=64 time=7.92 ms
80 bytes from 172.17.101.1: icmp_seq=4 ttl=64 time=7.96 ms
80 bytes from 172.17.101.1: icmp_seq=5 ttl=64 time=8.32 ms

--- 172.17.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 50ms
rtt min/avg/max/mdev = 7.925/8.401/9.651/0.643 ms, ipg/ewma 12.665/9.009 ms
leaf-1#
leaf-1#ping 172.16.102.1
PING 172.16.102.1 (172.16.102.1) 72(100) bytes of data.
80 bytes from 172.16.102.1: icmp_seq=1 ttl=64 time=11.7 ms
80 bytes from 172.16.102.1: icmp_seq=2 ttl=64 time=11.8 ms
80 bytes from 172.16.102.1: icmp_seq=3 ttl=64 time=14.9 ms
80 bytes from 172.16.102.1: icmp_seq=4 ttl=64 time=14.0 ms
80 bytes from 172.16.102.1: icmp_seq=5 ttl=64 time=13.0 ms

--- 172.16.102.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 57ms
rtt min/avg/max/mdev = 11.743/13.127/14.902/1.223 ms, pipe 2, ipg/ewma 14.419/12.473 ms
leaf-1#
leaf-1#ping 172.17.102.1
PING 172.17.102.1 (172.17.102.1) 72(100) bytes of data.
80 bytes from 172.17.102.1: icmp_seq=1 ttl=64 time=8.83 ms
80 bytes from 172.17.102.1: icmp_seq=2 ttl=64 time=7.73 ms
80 bytes from 172.17.102.1: icmp_seq=3 ttl=64 time=13.6 ms
80 bytes from 172.17.102.1: icmp_seq=4 ttl=64 time=28.0 ms
80 bytes from 172.17.102.1: icmp_seq=5 ttl=64 time=17.0 ms

--- 172.17.102.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 53ms
rtt min/avg/max/mdev = 7.739/15.073/28.041/7.305 ms, pipe 2, ipg/ewma 13.410/12.338 ms
leaf-1#
leaf-1#ping 172.16.202.1
PING 172.16.202.1 (172.16.202.1) 72(100) bytes of data.
80 bytes from 172.16.202.1: icmp_seq=1 ttl=63 time=30.3 ms
80 bytes from 172.16.202.1: icmp_seq=2 ttl=63 time=25.6 ms
80 bytes from 172.16.202.1: icmp_seq=3 ttl=63 time=33.0 ms
80 bytes from 172.16.202.1: icmp_seq=4 ttl=63 time=29.1 ms
80 bytes from 172.16.202.1: icmp_seq=5 ttl=63 time=17.1 ms

--- 172.16.202.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 89ms
rtt min/avg/max/mdev = 17.133/27.055/33.034/5.509 ms, pipe 3, ipg/ewma 22.447/28.439 ms
leaf-1#
leaf-1#ping 172.17.202.1
PING 172.17.202.1 (172.17.202.1) 72(100) bytes of data.
80 bytes from 172.17.202.1: icmp_seq=1 ttl=63 time=24.0 ms
80 bytes from 172.17.202.1: icmp_seq=2 ttl=63 time=27.8 ms
80 bytes from 172.17.202.1: icmp_seq=3 ttl=63 time=36.4 ms
80 bytes from 172.17.202.1: icmp_seq=4 ttl=63 time=27.9 ms
80 bytes from 172.17.202.1: icmp_seq=5 ttl=63 time=22.7 ms

--- 172.17.202.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 78ms
rtt min/avg/max/mdev = 22.792/27.821/36.417/4.756 ms, pipe 3, ipg/ewma 19.558/25.847 ms
leaf-1#
leaf-1#ping 172.16.203.1
PING 172.16.203.1 (172.16.203.1) 72(100) bytes of data.
80 bytes from 172.16.203.1: icmp_seq=1 ttl=63 time=23.5 ms
80 bytes from 172.16.203.1: icmp_seq=2 ttl=63 time=21.9 ms
80 bytes from 172.16.203.1: icmp_seq=3 ttl=63 time=16.5 ms
80 bytes from 172.16.203.1: icmp_seq=4 ttl=63 time=15.9 ms
80 bytes from 172.16.203.1: icmp_seq=5 ttl=63 time=20.4 ms

--- 172.16.203.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 85ms
rtt min/avg/max/mdev = 15.930/19.681/23.577/2.989 ms, pipe 2, ipg/ewma 21.296/21.536 ms
leaf-1#
leaf-1#ping 172.17.203.1
PING 172.17.203.1 (172.17.203.1) 72(100) bytes of data.
80 bytes from 172.17.203.1: icmp_seq=1 ttl=63 time=25.7 ms
80 bytes from 172.17.203.1: icmp_seq=2 ttl=63 time=20.9 ms
80 bytes from 172.17.203.1: icmp_seq=3 ttl=63 time=18.3 ms
80 bytes from 172.17.203.1: icmp_seq=4 ttl=63 time=27.0 ms
80 bytes from 172.17.203.1: icmp_seq=5 ttl=63 time=28.8 ms

--- 172.17.203.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 89ms
rtt min/avg/max/mdev = 18.364/24.195/28.872/3.928 ms, pipe 2, ipg/ewma 22.334/25.156 ms
leaf-1#
```

**Коммутатор leaf-2**

```
leaf-2#ping 172.16.101.1
PING 172.16.101.1 (172.16.101.1) 72(100) bytes of data.
80 bytes from 172.16.101.1: icmp_seq=1 ttl=64 time=12.6 ms
80 bytes from 172.16.101.1: icmp_seq=2 ttl=64 time=12.4 ms
80 bytes from 172.16.101.1: icmp_seq=3 ttl=64 time=9.92 ms
80 bytes from 172.16.101.1: icmp_seq=4 ttl=64 time=10.2 ms
80 bytes from 172.16.101.1: icmp_seq=5 ttl=64 time=10.4 ms

--- 172.16.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 59ms
rtt min/avg/max/mdev = 9.923/11.152/12.693/1.184 ms, ipg/ewma 14.958/11.859 msleaf-2#
leaf-2#ping 172.17.101.1
PING 172.17.101.1 (172.17.101.1) 72(100) bytes of data.
80 bytes from 172.17.101.1: icmp_seq=1 ttl=64 time=9.35 ms
80 bytes from 172.17.101.1: icmp_seq=2 ttl=64 time=13.8 ms
80 bytes from 172.17.101.1: icmp_seq=3 ttl=64 time=19.1 ms
80 bytes from 172.17.101.1: icmp_seq=4 ttl=64 time=12.5 ms
80 bytes from 172.17.101.1: icmp_seq=5 ttl=64 time=10.9 ms

--- 172.17.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 61ms
rtt min/avg/max/mdev = 9.350/13.160/19.117/3.341 ms, pipe 2, ipg/ewma 15.320/11.209 ms
leaf-2#
leaf-2#ping 172.16.102.1
PING 172.16.102.1 (172.16.102.1) 72(100) bytes of data.
80 bytes from 172.16.102.1: icmp_seq=1 ttl=64 time=10.9 ms
80 bytes from 172.16.102.1: icmp_seq=2 ttl=64 time=8.12 ms
80 bytes from 172.16.102.1: icmp_seq=3 ttl=64 time=9.30 ms
80 bytes from 172.16.102.1: icmp_seq=4 ttl=64 time=9.23 ms
80 bytes from 172.16.102.1: icmp_seq=5 ttl=64 time=9.56 ms

--- 172.16.102.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 57ms
rtt min/avg/max/mdev = 8.125/9.440/10.973/0.915 ms, ipg/ewma 14.488/10.208 ms
leaf-2#
leaf-2#ping 172.17.102.1
PING 172.17.102.1 (172.17.102.1) 72(100) bytes of data.
80 bytes from 172.17.102.1: icmp_seq=1 ttl=64 time=9.30 ms
80 bytes from 172.17.102.1: icmp_seq=2 ttl=64 time=9.45 ms
80 bytes from 172.17.102.1: icmp_seq=3 ttl=64 time=9.18 ms
80 bytes from 172.17.102.1: icmp_seq=4 ttl=64 time=8.03 ms
80 bytes from 172.17.102.1: icmp_seq=5 ttl=64 time=15.5 ms

--- 172.17.102.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 46ms
rtt min/avg/max/mdev = 8.032/10.314/15.595/2.690 ms, ipg/ewma 11.560/9.951 ms
leaf-2#
leaf-2#ping 172.16.201.1
PING 172.16.201.1 (172.16.201.1) 72(100) bytes of data.
80 bytes from 172.16.201.1: icmp_seq=1 ttl=63 time=22.9 ms
80 bytes from 172.16.201.1: icmp_seq=2 ttl=63 time=40.1 ms
80 bytes from 172.16.201.1: icmp_seq=3 ttl=63 time=41.1 ms
80 bytes from 172.16.201.1: icmp_seq=4 ttl=63 time=25.9 ms
80 bytes from 172.16.201.1: icmp_seq=5 ttl=63 time=26.5 ms

--- 172.16.201.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 92ms
rtt min/avg/max/mdev = 22.942/31.351/41.116/7.676 ms, pipe 2, ipg/ewma 23.101/26.909 ms
leaf-2#
leaf-2#ping 172.17.201.1
PING 172.17.201.1 (172.17.201.1) 72(100) bytes of data.
80 bytes from 172.17.201.1: icmp_seq=1 ttl=63 time=26.3 ms
80 bytes from 172.17.201.1: icmp_seq=2 ttl=63 time=31.5 ms
80 bytes from 172.17.201.1: icmp_seq=3 ttl=63 time=25.6 ms
80 bytes from 172.17.201.1: icmp_seq=4 ttl=63 time=38.4 ms
80 bytes from 172.17.201.1: icmp_seq=5 ttl=63 time=22.5 ms

--- 172.17.201.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 97ms
rtt min/avg/max/mdev = 22.507/28.902/38.496/5.602 ms, pipe 2, ipg/ewma 24.443/27.582 ms
leaf-2#
leaf-2#ping 172.16.203.1
PING 172.16.203.1 (172.16.203.1) 72(100) bytes of data.
80 bytes from 172.16.203.1: icmp_seq=1 ttl=63 time=19.9 ms
80 bytes from 172.16.203.1: icmp_seq=2 ttl=63 time=17.0 ms
80 bytes from 172.16.203.1: icmp_seq=3 ttl=63 time=21.1 ms
80 bytes from 172.16.203.1: icmp_seq=4 ttl=63 time=16.9 ms
80 bytes from 172.16.203.1: icmp_seq=5 ttl=63 time=17.1 ms

--- 172.16.203.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 79ms
rtt min/avg/max/mdev = 16.918/18.455/21.157/1.760 ms, pipe 2, ipg/ewma 19.788/19.151 ms
leaf-2#
leaf-2#ping 172.17.203.1
PING 172.17.203.1 (172.17.203.1) 72(100) bytes of data.
80 bytes from 172.17.203.1: icmp_seq=1 ttl=63 time=22.3 ms
80 bytes from 172.17.203.1: icmp_seq=2 ttl=63 time=19.1 ms
80 bytes from 172.17.203.1: icmp_seq=3 ttl=63 time=27.7 ms
80 bytes from 172.17.203.1: icmp_seq=4 ttl=63 time=37.1 ms
80 bytes from 172.17.203.1: icmp_seq=5 ttl=63 time=22.3 ms

--- 172.17.203.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 86ms
rtt min/avg/max/mdev = 19.133/25.761/37.150/6.334 ms, pipe 2, ipg/ewma 21.593/24.236 ms
leaf-2#
```

**Коммутатор leaf-3**

```
leaf-3#ping 172.16.101.1
PING 172.16.101.1 (172.16.101.1) 72(100) bytes of data.
80 bytes from 172.16.101.1: icmp_seq=1 ttl=64 time=12.8 ms
80 bytes from 172.16.101.1: icmp_seq=2 ttl=64 time=15.0 ms
80 bytes from 172.16.101.1: icmp_seq=3 ttl=64 time=23.0 ms
80 bytes from 172.16.101.1: icmp_seq=4 ttl=64 time=35.3 ms
80 bytes from 172.16.101.1: icmp_seq=5 ttl=64 time=25.4 ms

--- 172.16.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 76ms
rtt min/avg/max/mdev = 12.810/22.332/35.382/8.053 ms, pipe 2, ipg/ewma 19.017/18.017 ms
leaf-3#
leaf-3#ping 172.17.101.1
PING 172.17.101.1 (172.17.101.1) 72(100) bytes of data.
80 bytes from 172.17.101.1: icmp_seq=1 ttl=64 time=13.0 ms
80 bytes from 172.17.101.1: icmp_seq=2 ttl=64 time=8.33 ms
80 bytes from 172.17.101.1: icmp_seq=3 ttl=64 time=7.80 ms
80 bytes from 172.17.101.1: icmp_seq=4 ttl=64 time=10.0 ms
80 bytes from 172.17.101.1: icmp_seq=5 ttl=64 time=7.78 ms

--- 172.17.101.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 55ms
rtt min/avg/max/mdev = 7.785/9.406/13.045/2.001 ms, ipg/ewma 13.944/11.165 ms
leaf-3#
leaf-3#ping 172.16.102.1
PING 172.16.102.1 (172.16.102.1) 72(100) bytes of data.
80 bytes from 172.16.102.1: icmp_seq=1 ttl=64 time=15.4 ms
80 bytes from 172.16.102.1: icmp_seq=2 ttl=64 time=16.3 ms
80 bytes from 172.16.102.1: icmp_seq=3 ttl=64 time=9.13 ms
80 bytes from 172.16.102.1: icmp_seq=4 ttl=64 time=10.1 ms
80 bytes from 172.16.102.1: icmp_seq=5 ttl=64 time=8.65 ms

--- 172.16.102.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 57ms
rtt min/avg/max/mdev = 8.659/11.955/16.341/3.271 ms, pipe 2, ipg/ewma 14.279/13.502 ms
leaf-3#
leaf-3#ping 172.17.102.1
PING 172.17.102.1 (172.17.102.1) 72(100) bytes of data.
80 bytes from 172.17.102.1: icmp_seq=1 ttl=64 time=12.4 ms
80 bytes from 172.17.102.1: icmp_seq=2 ttl=64 time=9.78 ms
80 bytes from 172.17.102.1: icmp_seq=3 ttl=64 time=10.5 ms
80 bytes from 172.17.102.1: icmp_seq=4 ttl=64 time=10.1 ms
80 bytes from 172.17.102.1: icmp_seq=5 ttl=64 time=8.74 ms

--- 172.17.102.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 60ms
rtt min/avg/max/mdev = 8.746/10.341/12.457/1.216 ms, ipg/ewma 15.211/11.337 ms
leaf-3#
leaf-3#ping 172.16.201.1
PING 172.16.201.1 (172.16.201.1) 72(100) bytes of data.
80 bytes from 172.16.201.1: icmp_seq=1 ttl=63 time=21.0 ms
80 bytes from 172.16.201.1: icmp_seq=2 ttl=63 time=17.9 ms
80 bytes from 172.16.201.1: icmp_seq=3 ttl=63 time=17.3 ms
80 bytes from 172.16.201.1: icmp_seq=4 ttl=63 time=23.4 ms
80 bytes from 172.16.201.1: icmp_seq=5 ttl=63 time=17.7 ms

--- 172.16.201.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 82ms
rtt min/avg/max/mdev = 17.306/19.516/23.472/2.383 ms, pipe 2, ipg/ewma 20.558/20.294 ms
leaf-3#
leaf-3#ping 172.17.201.1
PING 172.17.201.1 (172.17.201.1) 72(100) bytes of data.
80 bytes from 172.17.201.1: icmp_seq=1 ttl=63 time=30.5 ms
80 bytes from 172.17.201.1: icmp_seq=2 ttl=63 time=27.0 ms
80 bytes from 172.17.201.1: icmp_seq=3 ttl=63 time=25.5 ms
80 bytes from 172.17.201.1: icmp_seq=4 ttl=63 time=20.3 ms
80 bytes from 172.17.201.1: icmp_seq=5 ttl=63 time=19.9 ms

--- 172.17.201.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 81ms
rtt min/avg/max/mdev = 19.916/24.684/30.570/4.077 ms, pipe 3, ipg/ewma 20.339/27.343 ms
leaf-3#
leaf-3#ping 172.16.202.1
PING 172.16.202.1 (172.16.202.1) 72(100) bytes of data.
80 bytes from 172.16.202.1: icmp_seq=1 ttl=63 time=80.0 ms
80 bytes from 172.16.202.1: icmp_seq=2 ttl=63 time=90.7 ms
80 bytes from 172.16.202.1: icmp_seq=3 ttl=63 time=90.1 ms
80 bytes from 172.16.202.1: icmp_seq=4 ttl=63 time=87.2 ms
80 bytes from 172.16.202.1: icmp_seq=5 ttl=63 time=79.2 ms

--- 172.16.202.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 45ms
rtt min/avg/max/mdev = 79.229/85.476/90.730/4.927 ms, pipe 5, ipg/ewma 11.345/82.587 ms
leaf-3#
leaf-3#ping 172.17.202.1
PING 172.17.202.1 (172.17.202.1) 72(100) bytes of data.
80 bytes from 172.17.202.1: icmp_seq=1 ttl=63 time=134 ms
80 bytes from 172.17.202.1: icmp_seq=2 ttl=63 time=129 ms
80 bytes from 172.17.202.1: icmp_seq=3 ttl=63 time=124 ms
80 bytes from 172.17.202.1: icmp_seq=4 ttl=63 time=118 ms
80 bytes from 172.17.202.1: icmp_seq=5 ttl=63 time=111 ms

--- 172.17.202.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 53ms
rtt min/avg/max/mdev = 111.708/123.704/134.572/8.006 ms, pipe 5, ipg/ewma 13.457/128.545 ms
leaf-3#
```
