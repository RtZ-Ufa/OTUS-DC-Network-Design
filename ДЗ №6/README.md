#  Домашнее задание №6
# VxLAN. L3 VNI
## по теме №12 "VxLAN. EVPN L2" 
### Цель: Настроить маршрутизацию в рамках Overlay между клиентами
### Задачи:
+ настроить каждого клиента в своем VNI;
+ настроить маршрутизацию между клиентами;
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
|spine-1   |L01        |172.16.101.1/32  |           |
|spine-1   |L02        |172.17.101.1/32|             |
|spine-2   |Eth1     |172.18.2.0/31  |-L- leaf-1    |
|spine-2   |Eth2     |172.18.2.2/31  |-L- leaf-2    |
|spine-2   |Eth3     |172.18.2.4/31  |-L- leaf-3    |
|spine-2   |L01        |172.16.102.1/32  |            |
|spine-2   |L02        |172.17.102.1/32|              |
|leaf-1    |Eth1     |172.18.1.1/31  |-S- spine-1    |
|leaf-1    |Eth2     |172.18.2.1/31  |-S- spine-2    |
|leaf-1    |Eth7     |               |to_client-1 |
|leaf-1    |L01        |172.16.201.1/32 |              |
|leaf-1    |L02        |172.17.201.1/32|              |
|leaf-2    |Eth1     |172.18.1.3/31  |-S- spine-1    |
|leaf-2    |Eth2     |172.18.2.3/31  |-S- spine-2    |
|leaf-2    |Eth7     |               |to_client-2  |
|leaf-2    |L01        |172.16.202.1/32 |              |
|leaf-2    |L02        |172.17.202.1/32|              |
|leaf-3    |Eth1     |172.18.1.5/31  |-S- spine-1    |
|leaf-3    |Eth2     |172.18.2.5/31  |-S- spine-2    |
|leaf-3    |Eth7     |               |to_client-3  |
|leaf-3    |Eth8     |               |to_client-4  |
|leaf-3    |L01        |172.16.203.1/32 |              |
|leaf-3    |L02        |172.17.203.1/32|              |
|client-1   |Eth        |192.168.0.1/24|              |
|client-2   |Eth        |192.168.1.1/24|              |
|client-3   |Eth        |192.168.2.1/24|              |
|client-4   |Eth        |192.168.2.2/24|              |

Для коммутаторов spine выбрана AS 65000, для коммутаторов leaf - соответственно по их номерам 650001, 65002, 65003.

3. Настройки оборудования приведены в соотвествующих текстовых файлах в этом каталоге (настройки коммутаторов spine остались неизменными с прошлой лабы), настройки клиентов приведены ниже:

**client-1**

```
client-1> show ip

NAME        : client-1[1]
IP/MASK     : 192.168.0.1/24
GATEWAY     : 192.168.0.254
DNS         :
MAC         : 00:50:79:66:68:06
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500
```

**client-2**

```
client-2> show ip

NAME        : client-2[1]
IP/MASK     : 192.168.1.1/24
GATEWAY     : 192.168.1.254
DNS         :
MAC         : 00:50:79:66:68:07
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500
```

**client-3**

```
client-3> show ip

NAME        : client-3[1]
IP/MASK     : 192.168.2.1/24
GATEWAY     : 192.168.2.254
DNS         :
MAC         : 00:50:79:66:68:08
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500
```

**client-4**

```
client-4> show ip

NAME        : client-4[1]
IP/MASK     : 192.168.2.2/24
GATEWAY     : 192.168.2.254
DNS         :
MAC         : 00:50:79:66:68:09
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500
```

4. Таблицы маршрутизации на коммутаторах:

**Коммутатор leaf-1**

```
leaf-1#show ip route vrf OTUS

VRF: OTUS
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

 C        192.168.0.0/24 is directly connected, Vlan10
 B E      192.168.1.1/32 [200/0] via VTEP 172.17.202.1 VNI 20000 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 B E      192.168.2.1/32 [200/0] via VTEP 172.17.203.1 VNI 20000 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 B E      192.168.2.2/32 [200/0] via VTEP 172.17.203.1 VNI 20000 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1

leaf-1#
```

**Коммутатор leaf-2**

```
leaf-2#show ip route vrf OTUS

VRF: OTUS
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

 B E      192.168.0.1/32 [200/0] via VTEP 172.17.201.1 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 C        192.168.1.0/24 is directly connected, Vlan11
 B E      192.168.2.1/32 [200/0] via VTEP 172.17.203.1 VNI 20000 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 B E      192.168.2.2/32 [200/0] via VTEP 172.17.203.1 VNI 20000 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1

leaf-2#
```

**Коммутатор leaf-3**

```
leaf-3#show ip route vrf OTUS

VRF: OTUS
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

 B E      192.168.0.1/32 [200/0] via VTEP 172.17.201.1 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      192.168.1.1/32 [200/0] via VTEP 172.17.202.1 VNI 20000 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 C        192.168.2.0/24 is directly connected, Vlan12

leaf-3#
```

5. Проверка дополнительных настроек BGP и EVPN, наличия маршрутов, MAC-адресов в табоицах коммутации, настройки интерфейса Vxlan1

**Коммутатор leaf-1**

```
leaf-1#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 172.16.201.1, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:10010 mac-ip 0050.7966.6806
                                 -                     -       -       0       i
 * >      RD: 65001:10010 mac-ip 0050.7966.6806 192.168.0.1
                                 -                     -       -       0       i
 * >Ec    RD: 65002:10011 mac-ip 0050.7966.6807
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 mac-ip 0050.7966.6807
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10011 mac-ip 0050.7966.6807 192.168.1.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 mac-ip 0050.7966.6807 192.168.1.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65003:10012 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10012 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65003:10012 mac-ip 0050.7966.6808 192.168.2.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10012 mac-ip 0050.7966.6808 192.168.2.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65003:10012 mac-ip 0050.7966.6809
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10012 mac-ip 0050.7966.6809
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65003:10012 mac-ip 0050.7966.6809 192.168.2.2
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10012 mac-ip 0050.7966.6809 192.168.2.2
                                 172.17.203.1          -       100     0       65000 65003 i

leaf-1#show arp vrf OTUS
Address         Age (sec)  Hardware Addr   Interface
192.168.0.1       0:01:17  0050.7966.6806  Vlan10, Ethernet6

leaf-1#show interface vxlan1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback2 and is active with 172.17.201.1
  Listening on UDP port 4789
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is
    [10, 10010]
  Dynamic VLAN to VNI mapping for 'evpn' is
    [4093, 20000]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is
   [OTUS, 20000]
  Shared Router MAC is 0000.0000.0000

leaf-1#show vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI         VLAN       Source       Interface       802.1Q Tag
----------- ---------- ------------ --------------- ----------
10010       10         static       Ethernet6       untagged
                                    Vxlan1          10

VNI to dynamic VLAN Mapping for Vxlan1
VNI         VLAN       VRF        Source
----------- ---------- ---------- ------------
20000       4093       OTUS       evpn

leaf-1#
```

**Коммутатор leaf-2**

```
leaf-2#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 172.16.202.1, local AS number 65002
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6806
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6806
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6806 192.168.0.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6806 192.168.0.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >      RD: 65002:10011 mac-ip 0050.7966.6807
                                 -                     -       -       0       i
 * >      RD: 65002:10011 mac-ip 0050.7966.6807 192.168.1.1
                                 -                     -       -       0       i
 * >Ec    RD: 65003:10012 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10012 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65003:10012 mac-ip 0050.7966.6808 192.168.2.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10012 mac-ip 0050.7966.6808 192.168.2.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65003:10012 mac-ip 0050.7966.6809
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10012 mac-ip 0050.7966.6809
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65003:10012 mac-ip 0050.7966.6809 192.168.2.2
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10012 mac-ip 0050.7966.6809 192.168.2.2
                                 172.17.203.1          -       100     0       65000 65003 i

leaf-2#show arp vrf OTUS
Address         Age (sec)  Hardware Addr   Interface
192.168.1.1       0:02:02  0050.7966.6807  Vlan11, Ethernet6

leaf-2#show interface vxlan1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback2 and is active with 172.17.202.1
  Listening on UDP port 4789
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is
    [11, 10011]
  Dynamic VLAN to VNI mapping for 'evpn' is
    [4094, 20000]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is
   [OTUS, 20000]
  Shared Router MAC is 0000.0000.0000

leaf-2#show vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI         VLAN       Source       Interface       802.1Q Tag
----------- ---------- ------------ --------------- ----------
10011       11         static       Ethernet6       untagged
                                    Vxlan1          11

VNI to dynamic VLAN Mapping for Vxlan1
VNI         VLAN       VRF        Source
----------- ---------- ---------- ------------
20000       4094       OTUS       evpn

leaf-2#
```

**Коммутатор leaf-3**

```
leaf-3#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 172.16.203.1, local AS number 65003
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6806
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6806
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6806 192.168.0.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6806 192.168.0.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65002:10011 mac-ip 0050.7966.6807
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 mac-ip 0050.7966.6807
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10011 mac-ip 0050.7966.6807 192.168.1.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 mac-ip 0050.7966.6807 192.168.1.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >      RD: 65003:10012 mac-ip 0050.7966.6808
                                 -                     -       -       0       i
 * >      RD: 65003:10012 mac-ip 0050.7966.6808 192.168.2.1
                                 -                     -       -       0       i
 * >      RD: 65003:10012 mac-ip 0050.7966.6809
                                 -                     -       -       0       i
 * >      RD: 65003:10012 mac-ip 0050.7966.6809 192.168.2.2
                                 -                     -       -       0       i

leaf-3#show arp vrf OTUS
Address         Age (sec)  Hardware Addr   Interface
192.168.2.1       0:02:37  0050.7966.6808  Vlan12, Ethernet6
192.168.2.2       0:03:58  0050.7966.6809  Vlan12, Ethernet7

leaf-3#show interface vxlan1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback2 and is active with 172.17.203.1
  Listening on UDP port 4789
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is
    [12, 10012]
  Dynamic VLAN to VNI mapping for 'evpn' is
    [4094, 20000]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is
   [OTUS, 20000]
  Shared Router MAC is 0000.0000.0000

leaf-3#show vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI         VLAN       Source       Interface       802.1Q Tag
----------- ---------- ------------ --------------- ----------
10012       12         static       Ethernet6       untagged
                                    Ethernet7       untagged
                                    Vxlan1          12

VNI to dynamic VLAN Mapping for Vxlan1
VNI         VLAN       VRF        Source
----------- ---------- ---------- ------------
20000       4094       OTUS       evpn

leaf-3#
```



7. Проверка связности между клиентскими устройствами утилитой **ping**.

**client-1**

```
client-1> ping 192.168.1.1

84 bytes from 192.168.1.1 icmp_seq=1 ttl=62 time=55.315 ms
84 bytes from 192.168.1.1 icmp_seq=2 ttl=62 time=39.293 ms
84 bytes from 192.168.1.1 icmp_seq=3 ttl=62 time=122.908 ms
84 bytes from 192.168.1.1 icmp_seq=4 ttl=62 time=40.806 ms
84 bytes from 192.168.1.1 icmp_seq=5 ttl=62 time=36.830 ms

client-1> ping 192.168.2.1

84 bytes from 192.168.2.1 icmp_seq=1 ttl=62 time=47.306 ms
84 bytes from 192.168.2.1 icmp_seq=2 ttl=62 time=40.859 ms
84 bytes from 192.168.2.1 icmp_seq=3 ttl=62 time=93.983 ms
84 bytes from 192.168.2.1 icmp_seq=4 ttl=62 time=42.563 ms
84 bytes from 192.168.2.1 icmp_seq=5 ttl=62 time=46.297 ms


client-1> ping 192.168.2.2

84 bytes from 192.168.2.2 icmp_seq=1 ttl=62 time=51.619 ms
84 bytes from 192.168.2.2 icmp_seq=2 ttl=62 time=144.941 ms
84 bytes from 192.168.2.2 icmp_seq=3 ttl=62 time=37.954 ms
84 bytes from 192.168.2.2 icmp_seq=4 ttl=62 time=35.641 ms
84 bytes from 192.168.2.2 icmp_seq=5 ttl=62 time=91.041 ms
```

**client-2**

```
client-2> ping 192.168.0.1

84 bytes from 192.168.0.1 icmp_seq=1 ttl=62 time=53.743 ms
84 bytes from 192.168.0.1 icmp_seq=2 ttl=62 time=69.883 ms
84 bytes from 192.168.0.1 icmp_seq=3 ttl=62 time=37.007 ms
84 bytes from 192.168.0.1 icmp_seq=4 ttl=62 time=47.500 ms
84 bytes from 192.168.0.1 icmp_seq=5 ttl=62 time=56.960 ms

client-2> ping 192.168.2.1

84 bytes from 192.168.2.1 icmp_seq=1 ttl=62 time=80.804 ms
84 bytes from 192.168.2.1 icmp_seq=2 ttl=62 time=39.275 ms
84 bytes from 192.168.2.1 icmp_seq=3 ttl=62 time=52.834 ms
84 bytes from 192.168.2.1 icmp_seq=4 ttl=62 time=37.782 ms
84 bytes from 192.168.2.1 icmp_seq=5 ttl=62 time=53.516 ms

client-2> ping 192.168.2.2

84 bytes from 192.168.2.2 icmp_seq=1 ttl=62 time=74.848 ms
84 bytes from 192.168.2.2 icmp_seq=2 ttl=62 time=45.711 ms
84 bytes from 192.168.2.2 icmp_seq=3 ttl=62 time=55.852 ms
84 bytes from 192.168.2.2 icmp_seq=4 ttl=62 time=51.112 ms
84 bytes from 192.168.2.2 icmp_seq=5 ttl=62 time=34.437 ms
```


**client-3**

```
client-3> ping 192.168.0.1

84 bytes from 192.168.0.1 icmp_seq=1 ttl=62 time=72.284 ms
84 bytes from 192.168.0.1 icmp_seq=2 ttl=62 time=37.385 ms
84 bytes from 192.168.0.1 icmp_seq=3 ttl=62 time=122.281 ms
84 bytes from 192.168.0.1 icmp_seq=4 ttl=62 time=50.107 ms
84 bytes from 192.168.0.1 icmp_seq=5 ttl=62 time=37.853 ms

client-3> ping 192.168.1.1

84 bytes from 192.168.1.1 icmp_seq=1 ttl=62 time=54.964 ms
84 bytes from 192.168.1.1 icmp_seq=2 ttl=62 time=39.246 ms
84 bytes from 192.168.1.1 icmp_seq=3 ttl=62 time=66.034 ms
84 bytes from 192.168.1.1 icmp_seq=4 ttl=62 time=50.492 ms
84 bytes from 192.168.1.1 icmp_seq=5 ttl=62 time=45.874 ms

client-3> ping 192.168.2.2

84 bytes from 192.168.2.2 icmp_seq=1 ttl=64 time=12.426 ms
84 bytes from 192.168.2.2 icmp_seq=2 ttl=64 time=18.157 ms
84 bytes from 192.168.2.2 icmp_seq=3 ttl=64 time=10.544 ms
84 bytes from 192.168.2.2 icmp_seq=4 ttl=64 time=11.400 ms
84 bytes from 192.168.2.2 icmp_seq=5 ttl=64 time=11.915 ms
```

**client-4**

```
client-4> ping 192.168.2.1

84 bytes from 192.168.2.1 icmp_seq=1 ttl=64 time=20.714 ms
84 bytes from 192.168.2.1 icmp_seq=2 ttl=64 time=13.156 ms
84 bytes from 192.168.2.1 icmp_seq=3 ttl=64 time=18.181 ms
84 bytes from 192.168.2.1 icmp_seq=4 ttl=64 time=17.286 ms
84 bytes from 192.168.2.1 icmp_seq=5 ttl=64 time=14.403 ms

client-4> ping 192.168.1.1

84 bytes from 192.168.1.1 icmp_seq=1 ttl=62 time=214.269 ms
84 bytes from 192.168.1.1 icmp_seq=2 ttl=62 time=40.156 ms
84 bytes from 192.168.1.1 icmp_seq=3 ttl=62 time=45.366 ms
84 bytes from 192.168.1.1 icmp_seq=4 ttl=62 time=41.702 ms
84 bytes from 192.168.1.1 icmp_seq=5 ttl=62 time=48.994 ms

client-4> ping 192.168.0.1

84 bytes from 192.168.0.1 icmp_seq=1 ttl=62 time=41.788 ms
84 bytes from 192.168.0.1 icmp_seq=2 ttl=62 time=43.619 ms
84 bytes from 192.168.0.1 icmp_seq=3 ttl=62 time=43.351 ms
84 bytes from 192.168.0.1 icmp_seq=4 ttl=62 time=159.673 ms
84 bytes from 192.168.0.1 icmp_seq=5 ttl=62 time=41.420 ms
```


