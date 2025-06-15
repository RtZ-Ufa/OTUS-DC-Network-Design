#  Домашнее задание №8
# VxLAN. Routing
## по теме №15 "VxLAN. Оптимизация таблиц маршрутизации" 
### Цель: Реализовать передачу суммарных префиксов через EVPN route-type 5
### Задачи:
+ подключить двух "клиентов" в разных VRF в рамках одной фабрики;
+ маршрутизацию между клиентами через внешнее устройство (граничный роутер\фаерволл\etc);
+ зафиксировать в документации - план работы, адресное пространство, схему сети, конфигурацию устройств.
## Практическая часть
#### Схема ЦОД
1. Схема сети

![Схема сети](network2.PNG)

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
|leaf-1    |Eth4     |               |to_leaf-2 |
|leaf-1    |Eth5     |               |to_leaf-2 |
|leaf-1    |Eth6     |               |to_srv-1 |
|leaf-1    |L01        |172.16.201.1/32 |              |
|leaf-1    |L02        |172.17.201.1/32|              |
|leaf-2    |Eth1     |172.18.1.3/31  |-S- spine-1    |
|leaf-2    |Eth2     |172.18.2.3/31  |-S- spine-2    |
|leaf-2    |Eth4     |               |to_leaf-1 |
|leaf-2    |Eth5     |               |to_leaf-1 |
|leaf-2    |Eth6     |               |to_srv-1 |
|leaf-2    |L01        |172.16.202.1/32 |              |
|leaf-2    |L02        |172.17.202.1/32|              |
|leaf-3    |Eth1     |172.18.1.5/31  |-S- spine-1    |
|leaf-3    |Eth2     |172.18.2.5/31  |-S- spine-2    |
|leaf-3    |Eth6     |               |to_client-3  |
|leaf-3    |Eth7     |               |to_client-4  |
|leaf-3    |L01        |172.16.203.1/32 |              |
|leaf-3    |L02        |172.17.203.1/32|              |
|srv   |Vlan10        |92.168.0.2/24|              |
|srv   |Vlan11        |192.168.1.2/24|              |
|client-3   |Eth        |192.168.0.3/24|              |
|client-4   |Eth        |192.168.1.3/24|              |

Для коммутаторов spine выбрана AS 65000, для коммутаторов leaf - соответственно по их номерам 65001, 65002, 65003.

3. Настройки оборудования приведены в соотвествующих текстовых файлах в этом каталоге (настройки коммутаторов spine остались неизменными с позапрошлой лабы), настройки клиентов приведены ниже:

**client-3**

```
client-3> show ip

NAME        : client-3[1]
IP/MASK     : 192.168.0.3/24
GATEWAY     : 192.168.0.254
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
IP/MASK     : 192.168.1.3/24
GATEWAY     : 192.168.1.254
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

 B E      192.168.0.2/32 [200/0] via VTEP 172.17.202.1 VNI 20000 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 B E      192.168.0.3/32 [200/0] via VTEP 172.17.203.1 VNI 20000 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 C        192.168.0.0/24 is directly connected, Vlan10
 B E      192.168.1.3/32 [200/0] via VTEP 172.17.203.1 VNI 20000 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 C        192.168.1.0/24 is directly connected, Vlan11

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

 B E      192.168.0.3/32 [200/0] via VTEP 172.17.203.1 VNI 20000 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 C        192.168.0.0/24 is directly connected, Vlan10
 B E      192.168.1.3/32 [200/0] via VTEP 172.17.203.1 VNI 20000 router-mac 50:00:00:15:f4:e8 local-interface Vxlan1
 C        192.168.1.0/24 is directly connected, Vlan11
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

 B E      192.168.0.2/32 [200/0] via VTEP 172.17.202.1 VNI 20000 router-mac 50:00:00:03:37:66 local-interface Vxlan1
 C        192.168.0.0/24 is directly connected, Vlan10
 C        192.168.1.0/24 is directly connected, Vlan11

leaf-3#
```

5. Проверка дополнительных настроек BGP и EVPN, наличия маршрутов, MLAG'ов, настройки интерфейса Vxlan1

**Коммутатор leaf-1**

```
leaf-1#show mlag
MLAG Configuration:
domain-id                          :              mlag50
local-interface                    :              Vlan50
peer-address                       :          10.50.50.1
peer-link                          :      Port-Channel50
peer-config                        :        inconsistent

MLAG Status:
state                              :              Active
negotiation status                 :           Connected
peer-link status                   :                  Up
local-int status                   :                  Up
system-id                          :   52:00:00:03:37:66
dual-primary detection             :            Disabled
dual-primary interface errdisabled :               False

MLAG Ports:
Disabled                           :                   0
Configured                         :                   0
Inactive                           :                   0
Active-partial                     :                   0
Active-full                        :                   1


leaf-1#show mlag interfaces detail
                                        local/remote
 mlag         state   local   remote    oper    config    last change   changes
------ ------------- ------- -------- ------- ---------- -------------- -------
    1   active-full    Po10     Po10   up/up   ena/ena    0:52:47 ago         5
leaf-1#
leaf-1#show port-channel 10 detailed
Port Channel Port-Channel10 (Fallback State: Unconfigured):
Minimum links: unconfigured
Minimum speed: unconfigured
Current weight/Max weight: 1/16
  Active Ports:
      Port               Time Became Active      Protocol      Mode      Weight
    ------------------ ----------------------- ------------- ----------- ------
      Ethernet6          17:34:36                LACP          Active      1
      PeerEthernet6      17:34:37                LACP          Active      0

leaf-1#show bgp evpn
BGP routing table information for VRF default
Router identifier 172.16.201.1, local AS number 65001
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:10010 mac-ip 0000.0000.0001
                                 172.16.201.1          -       -       0       i
 * >      RD: 65001:10011 mac-ip 0000.0000.0001
                                 172.16.201.1          -       -       0       i
 * >Ec    RD: 65001:10010 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65001:10010 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65001:10011 mac-ip 0000.0000.0002
                                 172.16.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10011 mac-ip 0000.0000.0002
                                 172.16.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65002:10011 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6808 192.168.0.3
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6808 192.168.0.3
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:10011 mac-ip 0050.7966.6809
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10011 mac-ip 0050.7966.6809
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:10011 mac-ip 0050.7966.6809 192.168.1.3
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10011 mac-ip 0050.7966.6809 192.168.1.3
                                 172.17.203.1          -       100     0       65000 65003 i
 * >      RD: 65001:10010 mac-ip 5000.00af.d3f6
                                 -                     -       -       0       i
 * >      RD: 65001:10011 mac-ip 5000.00af.d3f6
                                 -                     -       -       0       i
 * >Ec    RD: 65002:10011 mac-ip 5000.00af.d3f6
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 mac-ip 5000.00af.d3f6
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65001:10010 mac-ip 5000.00af.d3f6 192.168.0.2
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65001:10010 mac-ip 5000.00af.d3f6 192.168.0.2
                                 172.17.202.1          -       100     0       65000 65002 i
 * >      RD: 65001:10010 imet 172.16.201.1
                                 -                     -       -       0       i
 * >      RD: 65001:10011 imet 172.16.201.1
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10010 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65001:10010 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10011 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65001:10010 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10010 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:10011 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10011 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >      RD: 65001:10010 imet 172.17.201.1
                                 -                     -       -       0       i
 * >      RD: 65001:10011 imet 172.17.201.1
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10010 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65001:10010 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10011 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65001:10010 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10010 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:10011 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10011 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >      RD: 65001:20000 ip-prefix 192.168.0.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 65002:20000 ip-prefix 192.168.0.0/24
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:20000 ip-prefix 192.168.0.0/24
                                 172.17.202.1          -       100     0       65000 65002 i
 * >      RD: 65001:20000 ip-prefix 192.168.1.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 65002:20000 ip-prefix 192.168.1.0/24
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:20000 ip-prefix 192.168.1.0/24
                                 172.17.202.1          -       100     0       65000 65002 i

leaf-1#show vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  10  0000.0000.0002  STATIC    Vx1  172.16.202.1
  10  0050.7966.6808  EVPN      Vx1  172.17.203.1     1       0:03:59 ago
  11  0000.0000.0002  STATIC    Vx1  172.16.202.1
  11  0050.7966.6809  EVPN      Vx1  172.17.203.1     1       0:04:07 ago
4094  5000.0003.3766  EVPN      Vx1  172.17.202.1     2       0:52:48 ago
4094  5000.0015.f4e8  EVPN      Vx1  172.17.203.1     1       0:04:08 ago
Total Remote Mac Addresses for this criterion: 6

leaf-1#show interface vxlan1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback2 and is active with 172.17.201.1
  Listening on UDP port 4789
  Virtual VTEP source interface is 'Loopback1'
  Virtual VTEP IP address is 172.16.201.1
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is
    [10, 10010]       [11, 10011]
  Dynamic VLAN to VNI mapping for 'evpn' is
    [4094, 20000]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is
   [OTUS, 20000]
  Headend replication flood vtep list is:
    10 172.16.202.1    172.17.202.1    172.16.203.1    172.17.203.1
    11 172.16.202.1    172.17.202.1    172.16.203.1    172.17.203.1
  MLAG Shared Router MAC is 0000.0000.0000

leaf-1#show vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI         VLAN       Source       Interface            802.1Q Tag
----------- ---------- ------------ -------------------- ----------
10010       10         static       Ethernet6            untagged
                                    Port-Channel10       10
                                    Vxlan1               10
10011       11         static       Port-Channel10       11
                                    Vxlan1               11

VNI to dynamic VLAN Mapping for Vxlan1
VNI         VLAN       VRF        Source
----------- ---------- ---------- ------------
20000       4094       OTUS       evpn

```

**Коммутатор leaf-2**

```
leaf-2#show mlag
MLAG Configuration:
domain-id                          :              mlag50
local-interface                    :              Vlan50
peer-address                       :          10.50.50.0
peer-link                          :      Port-Channel50
peer-config                        :        inconsistent

MLAG Status:
state                              :              Active
negotiation status                 :           Connected
peer-link status                   :                  Up
local-int status                   :                  Up
system-id                          :   52:00:00:03:37:66
dual-primary detection             :            Disabled
dual-primary interface errdisabled :               False

MLAG Ports:
Disabled                           :                   0
Configured                         :                   0
Inactive                           :                   0
Active-partial                     :                   0
Active-full                        :                   1

leaf-2#show mlag interfaces detail
                                        local/remote
 mlag         state   local   remote    oper    config    last change   changes
------ ------------- ------- -------- ------- ---------- -------------- -------
    1   active-full    Po10     Po10   up/up   ena/ena    0:52:55 ago         5

leaf-2#show port-channel 10 detailed
Port Channel Port-Channel10 (Fallback State: Unconfigured):
Minimum links: unconfigured
Minimum speed: unconfigured
Current weight/Max weight: 1/16
  Active Ports:
      Port               Time Became Active      Protocol      Mode      Weight
    ------------------ ----------------------- ------------- ----------- ------
      Ethernet6          17:34:37                LACP          Active      1
      PeerEthernet6      17:34:36                LACP          Active      0

leaf-2#show bgp evpn
BGP routing table information for VRF default
Router identifier 172.16.202.1, local AS number 65002
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65001:10010 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10011 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10011 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65000 65001 i
 * >      RD: 65001:10010 mac-ip 0000.0000.0002
                                 172.16.202.1          -       -       0       i
 * >Ec    RD: 65001:10011 mac-ip 0000.0000.0002
                                 172.16.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10011 mac-ip 0000.0000.0002
                                 172.16.203.1          -       100     0       65000 65003 i
 * >      RD: 65002:10011 mac-ip 0000.0000.0002
                                 172.16.202.1          -       -       0       i
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6808 192.168.0.3
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6808 192.168.0.3
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:10011 mac-ip 0050.7966.6809
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10011 mac-ip 0050.7966.6809
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:10011 mac-ip 0050.7966.6809 192.168.1.3
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10011 mac-ip 0050.7966.6809 192.168.1.3
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:10010 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65000 65001 i
 *        RD: 65001:10010 mac-ip 5000.00af.d3f6
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10011 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10011 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65000 65001 i
 * >      RD: 65002:10011 mac-ip 5000.00af.d3f6
                                 -                     -       -       0       i
 * >      RD: 65001:10010 mac-ip 5000.00af.d3f6 192.168.0.2
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10010 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10011 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10011 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >      RD: 65001:10010 imet 172.16.202.1
                                 -                     -       -       0       i
 * >      RD: 65002:10011 imet 172.16.202.1
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10010 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10010 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:10011 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10011 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:10010 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10011 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10011 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >      RD: 65001:10010 imet 172.17.202.1
                                 -                     -       -       0       i
 * >      RD: 65002:10011 imet 172.17.202.1
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10010 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10010 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:10011 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65001:10011 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:20000 ip-prefix 192.168.0.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:20000 ip-prefix 192.168.0.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 * >      RD: 65002:20000 ip-prefix 192.168.0.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 65001:20000 ip-prefix 192.168.1.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:20000 ip-prefix 192.168.1.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 * >      RD: 65002:20000 ip-prefix 192.168.1.0/24
                                 -                     -       -       0       i
leaf-2#show vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  10  0000.0000.0001  STATIC    Vx1  172.16.201.1
  10  0050.7966.6808  EVPN      Vx1  172.17.203.1     1       0:04:08 ago
  11  0000.0000.0001  STATIC    Vx1  172.16.201.1
  11  0050.7966.6809  EVPN      Vx1  172.17.203.1     1       0:04:16 ago
4094  5000.0015.f4e8  EVPN      Vx1  172.17.203.1     1       0:04:17 ago
Total Remote Mac Addresses for this criterion: 5

leaf-2#show interface vxlan1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback2 and is active with 172.17.202.1
  Listening on UDP port 4789
  Virtual VTEP source interface is 'Loopback1'
  Virtual VTEP IP address is 172.16.202.1
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is
    [10, 10010]       [11, 10011]
  Dynamic VLAN to VNI mapping for 'evpn' is
    [4094, 20000]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is
   [OTUS, 20000]
  Headend replication flood vtep list is:
    10 172.16.201.1    172.16.203.1    172.17.203.1    172.17.201.1
    11 172.16.201.1    172.16.203.1    172.17.203.1    172.17.201.1
  MLAG Shared Router MAC is 0000.0000.0000

leaf-2#show vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI         VLAN       Source       Interface            802.1Q Tag
----------- ---------- ------------ -------------------- ----------
10010       10         static       Port-Channel10       10
                                    Vxlan1               10
10011       11         static       Ethernet6            untagged
                                    Port-Channel10       11
                                    Vxlan1               11

VNI to dynamic VLAN Mapping for Vxlan1
VNI         VLAN       VRF        Source
----------- ---------- ---------- ------------
20000       4094       OTUS       evpn

```

**Коммутатор leaf-3**

```
leaf-3#show bgp evpn
BGP routing table information for VRF default
Router identifier 172.16.203.1, local AS number 65003
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 65001:10010 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10011 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10011 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10010 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65001:10010 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 *        RD: 65001:10010 mac-ip 0000.0000.0002
                                 172.16.203.1          -       -       0       i
 * >      RD: 65001:10011 mac-ip 0000.0000.0002
                                 172.16.203.1          -       -       0       i
 * >Ec    RD: 65002:10011 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 * >      RD: 65001:10010 mac-ip 0050.7966.6808
                                 -                     -       -       0       i
 * >      RD: 65001:10010 mac-ip 0050.7966.6808 192.168.0.3
                                 -                     -       -       0       i
 * >      RD: 65001:10011 mac-ip 0050.7966.6809
                                 -                     -       -       0       i
 * >      RD: 65001:10011 mac-ip 0050.7966.6809 192.168.1.3
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10010 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10011 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10011 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65002:10011 mac-ip 5000.00af.d3f6
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 mac-ip 5000.00af.d3f6
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65001:10010 mac-ip 5000.00af.d3f6 192.168.0.2
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65001:10010 mac-ip 5000.00af.d3f6 192.168.0.2
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65001:10010 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10011 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10011 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10010 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65001:10010 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10011 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >      RD: 65001:10010 imet 172.16.203.1
                                 -                     -       -       0       i
 * >      RD: 65001:10011 imet 172.16.203.1
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10010 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10011 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10011 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10010 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65001:10010 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10011 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >      RD: 65001:10010 imet 172.17.203.1
                                 -                     -       -       0       i
 * >      RD: 65001:10011 imet 172.17.203.1
                                 -                     -       -       0       i
 * >Ec    RD: 65001:20000 ip-prefix 192.168.0.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:20000 ip-prefix 192.168.0.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65002:20000 ip-prefix 192.168.0.0/24
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:20000 ip-prefix 192.168.0.0/24
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65001:20000 ip-prefix 192.168.1.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:20000 ip-prefix 192.168.1.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65002:20000 ip-prefix 192.168.1.0/24
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:20000 ip-prefix 192.168.1.0/24
                                 172.17.202.1          -       100     0       65000 65002 i

leaf-3#show vxlan address-table
          Vxlan Mac Address Table
----------------------------------------------------------------------

VLAN  Mac Address     Type      Prt  VTEP             Moves   Last Move
----  -----------     ----      ---  ----             -----   ---------
  10  0000.0000.0001  STATIC    Vx1  172.16.201.1
  10  5000.00af.d3f6  EVPN      Vx1  172.17.201.1     1       0:04:17 ago
  11  0000.0000.0001  STATIC    Vx1  172.16.201.1
  11  5000.00af.d3f6  EVPN      Vx1  172.17.201.1     2       0:04:25 ago
4094  5000.0003.3766  EVPN      Vx1  172.17.202.1     1       0:53:06 ago
4094  5000.00d5.5dc0  EVPN      Vx1  172.17.201.1     1       0:53:08 ago
Total Remote Mac Addresses for this criterion: 6

leaf-3#show interface vxlan1
Vxlan1 is up, line protocol is up (connected)
  Hardware is Vxlan
  Source interface is Loopback2 and is active with 172.17.203.1
  Listening on UDP port 4789
  Virtual VTEP source interface is 'Loopback1'
  Virtual VTEP IP address is 172.16.203.1
  Replication/Flood Mode is headend with Flood List Source: EVPN
  Remote MAC learning via EVPN
  VNI mapping to VLANs
  Static VLAN to VNI mapping is
    [10, 10010]       [11, 10011]
  Dynamic VLAN to VNI mapping for 'evpn' is
    [4094, 20000]
  Note: All Dynamic VLANs used by VCS are internal VLANs.
        Use 'show vxlan vni' for details.
  Static VRF to VNI mapping is
   [OTUS, 20000]
  Headend replication flood vtep list is:
    10 172.16.202.1    172.17.202.1    172.16.201.1    172.17.201.1
    11 172.16.202.1    172.17.202.1    172.16.201.1    172.17.201.1
  Shared Router MAC is 0000.0000.0000

leaf-3#show vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI         VLAN       Source       Interface       802.1Q Tag
----------- ---------- ------------ --------------- ----------
10010       10         static       Ethernet6       untagged
                                    Vxlan1          10
10011       11         static       Ethernet7       untagged
                                    Vxlan1          11

VNI to dynamic VLAN Mapping for Vxlan1
VNI         VLAN       VRF        Source
----------- ---------- ---------- ------------
20000       4094       OTUS       evpn

```



7. Проверка связности между клиентскими устройствами утилитой **ping**.

**srv**

```
srv#ping 192.168.0.3
PING 192.168.0.3 (192.168.0.3) 72(100) bytes of data.
80 bytes from 192.168.0.3: icmp_seq=1 ttl=64 time=92.1 ms
80 bytes from 192.168.0.3: icmp_seq=2 ttl=64 time=85.8 ms
80 bytes from 192.168.0.3: icmp_seq=3 ttl=64 time=83.8 ms
80 bytes from 192.168.0.3: icmp_seq=4 ttl=64 time=81.1 ms
80 bytes from 192.168.0.3: icmp_seq=5 ttl=64 time=77.6 ms

--- 192.168.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 52ms
rtt min/avg/max/mdev = 77.683/84.127/92.122/4.847 ms, pipe 5, ipg/ewma 13.101/87.799 ms
srv#

srv#ping 192.168.1.3
PING 192.168.1.3 (192.168.1.3) 72(100) bytes of data.
80 bytes from 192.168.1.3: icmp_seq=1 ttl=64 time=44.8 ms
80 bytes from 192.168.1.3: icmp_seq=2 ttl=64 time=51.3 ms
80 bytes from 192.168.1.3: icmp_seq=3 ttl=64 time=54.6 ms
80 bytes from 192.168.1.3: icmp_seq=4 ttl=64 time=62.9 ms
80 bytes from 192.168.1.3: icmp_seq=5 ttl=64 time=81.8 ms

--- 192.168.1.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 91ms
rtt min/avg/max/mdev = 44.891/59.136/81.849/12.768 ms, pipe 4, ipg/ewma 22.895/52.961 ms
```

**client-3**

```

client-3> ping 192.168.0.2

84 bytes from 192.168.0.2 icmp_seq=1 ttl=64 time=49.733 ms
84 bytes from 192.168.0.2 icmp_seq=2 ttl=64 time=75.222 ms
84 bytes from 192.168.0.2 icmp_seq=3 ttl=64 time=46.064 ms
84 bytes from 192.168.0.2 icmp_seq=4 ttl=64 time=39.158 ms
84 bytes from 192.168.0.2 icmp_seq=5 ttl=64 time=46.079 ms

client-3> ping 192.168.1.2

192.168.1.2 icmp_seq=1 timeout
84 bytes from 192.168.1.2 icmp_seq=2 ttl=64 time=92.218 ms
84 bytes from 192.168.1.2 icmp_seq=3 ttl=64 time=92.264 ms
84 bytes from 192.168.1.2 icmp_seq=4 ttl=64 time=94.178 ms
84 bytes from 192.168.1.2 icmp_seq=5 ttl=64 time=94.797 ms
84 bytes from 192.168.1.2 icmp_seq=5 ttl=64 time=93.587 ms


client-3> ping 192.168.1.3

84 bytes from 192.168.1.3 icmp_seq=1 ttl=63 time=15.532 ms
84 bytes from 192.168.1.3 icmp_seq=2 ttl=63 time=16.716 ms
84 bytes from 192.168.1.3 icmp_seq=3 ttl=63 time=16.618 ms
84 bytes from 192.168.1.3 icmp_seq=4 ttl=63 time=15.257 ms
84 bytes from 192.168.1.3 icmp_seq=5 ttl=63 time=18.782 ms

client-3>
```

**client-4**

```
client-4> ping 192.168.1.2

84 bytes from 192.168.1.2 icmp_seq=1 ttl=64 time=55.722 ms
84 bytes from 192.168.1.2 icmp_seq=2 ttl=64 time=52.953 ms
84 bytes from 192.168.1.2 icmp_seq=3 ttl=64 time=59.413 ms
84 bytes from 192.168.1.2 icmp_seq=4 ttl=64 time=100.454 ms
84 bytes from 192.168.1.2 icmp_seq=5 ttl=64 time=42.188 ms


client-4> ping 192.168.0.3

84 bytes from 192.168.0.3 icmp_seq=1 ttl=63 time=39.631 ms
84 bytes from 192.168.0.3 icmp_seq=2 ttl=63 time=18.465 ms
84 bytes from 192.168.0.3 icmp_seq=3 ttl=63 time=26.007 ms
84 bytes from 192.168.0.3 icmp_seq=4 ttl=63 time=17.723 ms
84 bytes from 192.168.0.3 icmp_seq=5 ttl=63 time=24.755 ms


client-4> ping 192.168.0.2

84 bytes from 192.168.0.2 icmp_seq=1 ttl=64 time=773.491 ms
84 bytes from 192.168.0.2 icmp_seq=2 ttl=64 time=111.783 ms
84 bytes from 192.168.0.2 icmp_seq=3 ttl=64 time=51.146 ms
84 bytes from 192.168.0.2 icmp_seq=4 ttl=64 time=46.040 ms
84 bytes from 192.168.0.2 icmp_seq=5 ttl=64 time=45.165 ms
```


