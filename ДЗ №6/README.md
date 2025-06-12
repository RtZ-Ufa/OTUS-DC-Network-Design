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

5. Проверка дополнительных настроек BGP и EVPN

**Коммутатор spine-1**

```
spine-1#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 172.16.101.1, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:10010 mac-ip 0050.7966.6806
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10010 mac-ip 0050.7966.6807
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65001:10010 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65003 i
 * >      RD: 65001:10010 mac-ip 0050.7966.6809
                                 172.17.203.1          -       100     0       65003 i

spine-1#show bgp evpn summary
BGP summary information for VRF default
Router identifier 172.16.101.1, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  172.16.201.1 4 65001            509       509    0    0 00:21:10 Estab   2      2
  172.16.202.1 4 65002            502       501    0    0 00:20:52 Estab   2      2
  172.16.203.1 4 65003            500       497    0   19 00:20:41 Estab   3      3

spine-1#show ip bgp summary
BGP summary information for VRF default
Router identifier 172.16.101.1, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  172.16.201.1 4 65001            509       509    0    0 00:21:10 Estab   2      2
  172.16.202.1 4 65002            502       501    0    0 00:20:52 Estab   2      2
  172.16.203.1 4 65003            501       497    0    0 00:20:41 Estab   2      2
  172.18.1.1   4 65001            501       504    0    0 00:21:12 Estab   6      2
  172.18.1.3   4 65002            492       496    0    0 00:20:53 Estab   6      2
  172.18.1.5   4 65003            491       492    0    0 00:20:43 Estab   8      2
spine-1#
```

**Коммутатор spine-2**

```
spine-2#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 172.16.102.1, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:10010 mac-ip 0050.7966.6806
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10010 mac-ip 0050.7966.6807
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65001:10010 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65003 i
 * >      RD: 65001:10010 mac-ip 0050.7966.6809
                                 172.17.203.1          -       100     0       65003 i

spine-2#show bgp evpn summary
BGP summary information for VRF default
Router identifier 172.16.102.1, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  172.16.201.1 4 65001            534       539    0    0 00:22:15 Estab   2      2
  172.16.202.1 4 65002            532       527    0    0 00:21:58 Estab   2      2
  172.16.203.1 4 65003            529       521    0    0 00:21:45 Estab   3      3
spine-2#
spine-2#show ip bgp summary
BGP summary information for VRF default
Router identifier 172.16.102.1, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  172.16.201.1 4 65001            534       539    0    0 00:22:16 Estab   2      2
  172.16.202.1 4 65002            532       527    0    0 00:21:58 Estab   2      2
  172.16.203.1 4 65003            529       521    0    0 00:21:45 Estab   2      2
  172.18.2.1   4 65001            532       529    0    0 00:22:19 Estab   6      2
  172.18.2.3   4 65002            522       520    0    0 00:22:01 Estab   6      2
  172.18.2.5   4 65003            521       517    0    0 00:21:51 Estab   4      2
spine-2#
```

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
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6807
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6807
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6809
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6809
                                 172.17.203.1          -       100     0       65000 65003 i

leaf-1#show vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  10  0050.7966.6807  EVPN      Vx1  172.17.202.1     1       0:02:33 ago
  10  0050.7966.6808  EVPN      Vx1  172.17.203.1     1       0:02:14 ago
  10  0050.7966.6809  EVPN      Vx1  172.17.203.1     1       0:01:59 ago
Total Remote Mac Addresses for this criterion: 3


leaf-1#show bgp evpn summary
BGP summary information for VRF default
Router identifier 172.16.201.1, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  172.16.101.1 4 65000            539       539    0    0 00:22:26 Estab   5      5
  172.16.102.1 4 65000            542       538    0    0 00:22:24 Estab   5      5
leaf-1#
leaf-1#show ip bgp summary
BGP summary information for VRF default
Router identifier 172.16.201.1, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  172.16.101.1 4 65000            539       539    0    0 00:22:26 Estab   6      6
  172.16.102.1 4 65000            542       538    0    0 00:22:24 Estab   6      6
  172.18.1.0   4 65000            533       530    0    0 00:22:28 Estab   6      6
  172.18.2.0   4 65000            532       535    0    0 00:22:27 Estab   6      6

leaf-1#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP               Tunnel Type(s)
------------------ --------------
172.17.202.1       flood, unicast
172.17.203.1       flood, unicast

Total number of remote VTEPS:  2
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
 * >      RD: 65001:10010 mac-ip 0050.7966.6807
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6809
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6809
                                 172.17.203.1          -       100     0       65000 65003 i

leaf-2#show vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  10  0050.7966.6806  EVPN      Vx1  172.17.201.1     1       0:03:00 ago
  10  0050.7966.6808  EVPN      Vx1  172.17.203.1     1       0:02:42 ago
  10  0050.7966.6809  EVPN      Vx1  172.17.203.1     1       0:02:27 ago
Total Remote Mac Addresses for this criterion: 3

leaf-2#show bgp evpn summary
BGP summary information for VRF default
Router identifier 172.16.202.1, local AS number 65002
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  172.16.101.1 4 65000            542       541    0    0 00:22:35 Estab   5      5
  172.16.102.1 4 65000            540       546    0    0 00:22:34 Estab   5      5

leaf-2#show ip bgp summary
BGP summary information for VRF default
Router identifier 172.16.202.1, local AS number 65002
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  172.16.101.1 4 65000            542       541    0    0 00:22:35 Estab   6      6
  172.16.102.1 4 65000            540       546    0    0 00:22:34 Estab   6      6
  172.18.1.2   4 65000            534       532    0    0 00:22:37 Estab   6      6
  172.18.2.2   4 65000            535       536    0    0 00:22:36 Estab   6      6

leaf-2#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP               Tunnel Type(s)
------------------ --------------
172.17.201.1       flood, unicast
172.17.203.1       flood, unicast

Total number of remote VTEPS:  2
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
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6807
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6807
                                 172.17.202.1          -       100     0       65000 65002 i
 * >      RD: 65001:10010 mac-ip 0050.7966.6808
                                 -                     -       -       0       i
 * >      RD: 65001:10010 mac-ip 0050.7966.6809
                                 -                     -       -       0       i

leaf-3#show vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  10  0050.7966.6806  EVPN      Vx1  172.17.201.1     1       0:03:10 ago
  10  0050.7966.6807  EVPN      Vx1  172.17.202.1     1       0:03:10 ago
Total Remote Mac Addresses for this criterion: 2

leaf-3#show bgp evpn summary
BGP summary information for VRF default
Router identifier 172.16.203.1, local AS number 65003
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  172.16.101.1 4 65000            541       545    0    0 00:22:34 Estab   4      4
  172.16.102.1 4 65000            538       547    0    0 00:22:31 Estab   4      4

leaf-3#show ip bgp summary
BGP summary information for VRF default
Router identifier 172.16.203.1, local AS number 65003
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  172.16.101.1 4 65000            542       545    0    0 00:22:35 Estab   6      6
  172.16.102.1 4 65000            539       547    0    0 00:22:31 Estab   6      6
  172.18.1.4   4 65000            535       535    0    0 00:22:37 Estab   6      6
  172.18.2.4   4 65000            535       539    0    0 00:22:37 Estab   6      6

leaf-3#show vxlan vtep
Remote VTEPS for Vxlan1:

VTEP               Tunnel Type(s)
------------------ --------------
172.17.201.1       flood, unicast
172.17.202.1       flood, unicast

Total number of remote VTEPS:  2
leaf-3#
```



7. Проверка связности между клиентскими устройствами утилитой **ping**.

**client-1**

```
client-1> ping 192.168.0.2

84 bytes from 192.168.0.2 icmp_seq=1 ttl=64 time=32.128 ms
84 bytes from 192.168.0.2 icmp_seq=2 ttl=64 time=31.835 ms
84 bytes from 192.168.0.2 icmp_seq=3 ttl=64 time=32.857 ms
84 bytes from 192.168.0.2 icmp_seq=4 ttl=64 time=42.356 ms
84 bytes from 192.168.0.2 icmp_seq=5 ttl=64 time=30.261 ms

client-1> ping 192.168.0.3

84 bytes from 192.168.0.3 icmp_seq=1 ttl=64 time=40.211 ms
84 bytes from 192.168.0.3 icmp_seq=2 ttl=64 time=43.224 ms
84 bytes from 192.168.0.3 icmp_seq=3 ttl=64 time=143.691 ms
84 bytes from 192.168.0.3 icmp_seq=4 ttl=64 time=36.844 ms
84 bytes from 192.168.0.3 icmp_seq=5 ttl=64 time=35.865 ms

client-1> ping 192.168.0.4

84 bytes from 192.168.0.4 icmp_seq=1 ttl=64 time=44.362 ms
84 bytes from 192.168.0.4 icmp_seq=2 ttl=64 time=30.546 ms
84 bytes from 192.168.0.4 icmp_seq=3 ttl=64 time=37.396 ms
84 bytes from 192.168.0.4 icmp_seq=4 ttl=64 time=33.088 ms
84 bytes from 192.168.0.4 icmp_seq=5 ttl=64 time=40.510 ms
```

**client-2**

```
client-2> ping 192.168.0.1

84 bytes from 192.168.0.1 icmp_seq=1 ttl=64 time=35.314 ms
84 bytes from 192.168.0.1 icmp_seq=2 ttl=64 time=37.119 ms
84 bytes from 192.168.0.1 icmp_seq=3 ttl=64 time=36.233 ms
84 bytes from 192.168.0.1 icmp_seq=4 ttl=64 time=37.552 ms
84 bytes from 192.168.0.1 icmp_seq=5 ttl=64 time=37.323 ms

client-2> ping 192.168.0.4

84 bytes from 192.168.0.4 icmp_seq=1 ttl=64 time=38.056 ms
84 bytes from 192.168.0.4 icmp_seq=2 ttl=64 time=50.732 ms
84 bytes from 192.168.0.4 icmp_seq=3 ttl=64 time=42.052 ms
84 bytes from 192.168.0.4 icmp_seq=4 ttl=64 time=37.738 ms
84 bytes from 192.168.0.4 icmp_seq=5 ttl=64 time=35.355 ms

client-2> ping 192.168.0.3

84 bytes from 192.168.0.3 icmp_seq=1 ttl=64 time=33.578 ms
84 bytes from 192.168.0.3 icmp_seq=2 ttl=64 time=41.683 ms
84 bytes from 192.168.0.3 icmp_seq=3 ttl=64 time=48.181 ms
84 bytes from 192.168.0.3 icmp_seq=4 ttl=64 time=36.934 ms
84 bytes from 192.168.0.3 icmp_seq=5 ttl=64 time=33.034 ms
```


**client-3**

```
client-3> ping 192.168.0.1

84 bytes from 192.168.0.1 icmp_seq=1 ttl=64 time=40.824 ms
84 bytes from 192.168.0.1 icmp_seq=2 ttl=64 time=35.768 ms
84 bytes from 192.168.0.1 icmp_seq=3 ttl=64 time=47.706 ms
84 bytes from 192.168.0.1 icmp_seq=4 ttl=64 time=36.131 ms
84 bytes from 192.168.0.1 icmp_seq=5 ttl=64 time=36.016 ms

client-3> ping 192.168.0.2

84 bytes from 192.168.0.2 icmp_seq=1 ttl=64 time=39.184 ms
84 bytes from 192.168.0.2 icmp_seq=2 ttl=64 time=34.451 ms
84 bytes from 192.168.0.2 icmp_seq=3 ttl=64 time=38.735 ms
84 bytes from 192.168.0.2 icmp_seq=4 ttl=64 time=33.742 ms
84 bytes from 192.168.0.2 icmp_seq=5 ttl=64 time=38.760 ms

client-3> ping 192.168.0.4

84 bytes from 192.168.0.4 icmp_seq=1 ttl=64 time=11.959 ms
84 bytes from 192.168.0.4 icmp_seq=2 ttl=64 time=9.268 ms
84 bytes from 192.168.0.4 icmp_seq=3 ttl=64 time=10.094 ms
84 bytes from 192.168.0.4 icmp_seq=4 ttl=64 time=12.839 ms
84 bytes from 192.168.0.4 icmp_seq=5 ttl=64 time=10.230 ms
```

**client-4**

```
client-4> ping 192.168.0.3

84 bytes from 192.168.0.3 icmp_seq=1 ttl=64 time=158.707 ms
84 bytes from 192.168.0.3 icmp_seq=2 ttl=64 time=11.091 ms
84 bytes from 192.168.0.3 icmp_seq=3 ttl=64 time=13.027 ms
84 bytes from 192.168.0.3 icmp_seq=4 ttl=64 time=13.996 ms
84 bytes from 192.168.0.3 icmp_seq=5 ttl=64 time=10.643 ms

client-4> ping 192.168.0.2

84 bytes from 192.168.0.2 icmp_seq=1 ttl=64 time=109.527 ms
84 bytes from 192.168.0.2 icmp_seq=2 ttl=64 time=38.904 ms
84 bytes from 192.168.0.2 icmp_seq=3 ttl=64 time=38.122 ms
84 bytes from 192.168.0.2 icmp_seq=4 ttl=64 time=38.435 ms
84 bytes from 192.168.0.2 icmp_seq=5 ttl=64 time=37.562 ms

client-4> ping 192.168.0.1

84 bytes from 192.168.0.1 icmp_seq=1 ttl=64 time=90.992 ms
84 bytes from 192.168.0.1 icmp_seq=2 ttl=64 time=54.431 ms
84 bytes from 192.168.0.1 icmp_seq=3 ttl=64 time=34.148 ms
84 bytes from 192.168.0.1 icmp_seq=4 ttl=64 time=38.755 ms
84 bytes from 192.168.0.1 icmp_seq=5 ttl=64 time=36.173 ms
```


