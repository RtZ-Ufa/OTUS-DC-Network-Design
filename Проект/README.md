#  Проектная работа
## Проектирование сетевой фабрики на основе VxLAN EVPN
### Цель: Создание масштабируемой, отказоустойчивой и гибкой сети в рамках ЦОД

Проект посвящен разработке эффективной архитектуры сетевой фабрики для центров обработки данных (ЦОД) с использованием технологии виртуализации сетей VxLAN (Virtual Extensible LAN) совместно с механизмом EVPN (Ethernet Virtual Private Network). Выбор данной комбинации обусловлен необходимостью обеспечить масштабируемость, высокую производительность и гибкость современных корпоративных инфраструктур.
Обоснование выбора:
+ **Масштабируемость**: Технология VxLAN позволяет строить плоские L2-домены практически неограниченного масштаба благодаря использованию 24-разрядных идентификаторов сегментов (VNI), поддерживая тысячи виртуальных локальных сетей одновременно.
+ **Эффективность управления трафиком**: Механизм EVPN предоставляет улучшенные средства контроля над многоадресной передачей и автоматизацию распределения MAC/IP-адресов среди коммутаторов, сокращая нагрузку на управляющие плоскости и уменьшая время распространения изменений конфигурации.    Повышение отказоустойчивости: Благодаря использованию MPLS-подобных методов коммутации и динамического перенаправления потоков достигается высокая степень устойчивости к сбоям и быстрым изменениям топологии сети.

Особенности организации сетей ЦОД:
+ проект учитывает особенности современного ЦОД, такие как необходимость поддержки множества виртуализированных сервисов, высокая плотность подключенных узлов и требовательность приложений к минимизации задержек и потере пакетов;
+ предусматривается применение многозональных подходов и схем резервирования ключевых элементов сети, обеспечение качества обслуживания (QoS) для критически важных бизнес-процессов;
+ особое внимание уделено вопросам безопасности и изоляции трафика различных клиентов и отделов внутри единой физической инфраструктуры.

Данный проект нацелен на создание высокопроизводительной, легко управляемой и надежной платформы для удовлетворения потребностей предприятий различного уровня сложности и объемов нагрузки.

## Практическая часть
#### Схема ЦОД
1. Схема сети

![Схема сети](network4.PNG)

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
|leaf-1    |L01        |172.16.201.1/32 |              |
|leaf-1    |L02        |172.17.201.1/32|              |
|leaf-1    |Vlan10   |192.168.10.254/24|              |
|leaf-1    |Vlan11   |192.168.11.254/24|              |
|leaf-1    |Vlan100   |100.100.100.100/24|              |
|leaf-1    |Vlan101  |101.101.101.101/24|              |
|leaf-2    |Eth1     |172.18.1.3/31  |-S- spine-1    |
|leaf-2    |Eth2     |172.18.2.3/31  |-S- spine-2    |
|leaf-2    |L01        |172.16.202.1/32 |              |
|leaf-2    |L02        |172.17.202.1/32|              |
|leaf-3    |Eth1     |172.18.1.5/31  |-S- spine-1    |
|leaf-3    |Eth2     |172.18.2.5/31  |-S- spine-2    |
|leaf-3    |L01        |172.16.203.1/32 |              |
|leaf-3    |L02        |172.17.203.1/32|              |
|srv    |Vlan10        |192.168.10.10/24|              |
|srv    |Vlan11        |192.168.11.11/24|              |
|ext    |Vlan100        |100.100.100.99/24|            |
|ext    |Vlan101        |101.101.101.100/24|           |
|ext    |Loopback1      |5.5.5.5/32|           |
|client-1   |Eth        |192.168.10.1/24|              |
|client-2   |Eth        |101.101.101.1/24|              |
|client-3   |Eth        |192.168.10.2/24|              |
|client-4   |Eth        |101.101.11.1/24|              |


Для коммутаторов spine выбрана AS 65000, для коммутаторов leaf - соответственно по их номерам 65001, 65002, 65003.

3. Настройки оборудования приведены в соотвествующих текстовых файлах в этом каталоге, настройки клиентов приведены ниже:

**client-1**

```
client-1> show ip

NAME        : client-1[1]
IP/MASK     : 192.168.10.1/24
GATEWAY     : 192.168.10.254
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
IP/MASK     : 101.101.101.1/24
GATEWAY     : 101.101.101.101
DNS         :
MAC         : 00:50:79:66:68:09
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500
```

**client-3**

```
client-3> show ip

NAME        : client-3[1]
IP/MASK     : 192.168.10.2/24
GATEWAY     : 192.168.10.254
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
IP/MASK     : 192.168.11.1/24
GATEWAY     : 192.168.11.254
DNS         :
MAC         : 00:50:79:66:68:07
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500
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
```

**Коммутатор spine-2**

```
spine-2#show ip route vrf all

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
```

**Коммутатор ext**

```
ext#show ip route vrf all

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

Gateway of last resort:
 S        0.0.0.0/0 is directly connected, Null0

 C        5.5.5.5/32 is directly connected, Loopback1
 C        100.100.100.0/24 is directly connected, Vlan100
 C        101.101.101.0/24 is directly connected, Vlan101
 B E      192.168.10.0/24 [200/0] via 100.100.100.100, Vlan100
 B E      192.168.11.0/24 [200/0] via 100.100.100.100, Vlan100
```

**Коммутатор leaf-1**

```
leaf-1#show ip route vrf all

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

 C        10.50.50.0/31 is directly connected, Vlan50
 B E      172.16.101.1/32 [200/0] via 172.18.1.0, Ethernet1
 B E      172.16.102.1/32 [200/0] via 172.18.2.0, Ethernet2
 C        172.16.201.1/32 is directly connected, Loopback1
 B E      172.16.202.1/32 [200/0] via 172.18.1.0, Ethernet1
                                  via 172.18.2.0, Ethernet2
 B E      172.16.203.1/32 [200/0] via 172.18.1.0, Ethernet1
                                  via 172.18.2.0, Ethernet2
 B E      172.17.101.1/32 [200/0] via 172.18.1.0, Ethernet1
 B E      172.17.102.1/32 [200/0] via 172.18.2.0, Ethernet2
 C        172.17.201.1/32 is directly connected, Loopback2
 B E      172.17.202.1/32 [200/0] via 172.18.1.0, Ethernet1
                                  via 172.18.2.0, Ethernet2
 B E      172.17.203.1/32 [200/0] via 172.18.1.0, Ethernet1
                                  via 172.18.2.0, Ethernet2
 C        172.18.1.0/31 is directly connected, Ethernet1
 C        172.18.2.0/31 is directly connected, Ethernet2


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

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via 100.100.100.99, Vlan100

 B E      5.5.5.5/32 [200/0] via 100.100.100.99, Vlan100
 C        100.100.100.0/24 is directly connected, Vlan100
 B E      101.101.101.0/24 [200/0] via 100.100.100.99, Vlan100
 C        192.168.10.0/24 is directly connected, Vlan10
 C        192.168.11.0/24 is directly connected, Vlan11


VRF: OTUS2
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

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via 101.101.101.100, Vlan101

 B E      5.5.5.5/32 [200/0] via 101.101.101.100, Vlan101
 B E      100.100.100.0/24 [200/0] via 101.101.101.100, Vlan101
 C        101.101.101.0/24 is directly connected, Vlan101
leaf-1
```

**Коммутатор leaf-2**

```
leaf-2#show ip route vrf all

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

 C        10.50.50.0/31 is directly connected, Vlan50
 B E      172.16.101.1/32 [200/0] via 172.18.1.2, Ethernet1
 B E      172.16.102.1/32 [200/0] via 172.18.2.2, Ethernet2
 B E      172.16.201.1/32 [200/0] via 172.18.1.2, Ethernet1
                                  via 172.18.2.2, Ethernet2
 C        172.16.202.1/32 is directly connected, Loopback1
 B E      172.16.203.1/32 [200/0] via 172.18.1.2, Ethernet1
                                  via 172.18.2.2, Ethernet2
 B E      172.17.101.1/32 [200/0] via 172.18.1.2, Ethernet1
 B E      172.17.102.1/32 [200/0] via 172.18.2.2, Ethernet2
 B E      172.17.201.1/32 [200/0] via 172.18.1.2, Ethernet1
                                  via 172.18.2.2, Ethernet2
 C        172.17.202.1/32 is directly connected, Loopback2
 B E      172.17.203.1/32 [200/0] via 172.18.1.2, Ethernet1
                                  via 172.18.2.2, Ethernet2
 C        172.18.1.2/31 is directly connected, Ethernet1
 C        172.18.2.2/31 is directly connected, Ethernet2


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

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via VTEP 172.17.201.1 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1

 B E      5.5.5.5/32 [200/0] via VTEP 172.17.201.1 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      100.100.100.0/24 [200/0] via VTEP 172.17.201.1 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      101.101.101.0/24 [200/0] via VTEP 172.17.201.1 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      192.168.10.1/32 [200/0] via VTEP 172.17.201.1 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      192.168.10.10/32 [200/0] via VTEP 172.17.201.1 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      192.168.10.0/24 [200/0] via VTEP 172.17.201.1 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      192.168.11.11/32 [200/0] via VTEP 172.17.201.1 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1
 B E      192.168.11.0/24 [200/0] via VTEP 172.17.201.1 VNI 20000 router-mac 50:00:00:d5:5d:c0 local-interface Vxlan1

leaf-2#
```

**Коммутатор leaf-3**

```
leaf-3#show ip route vrf all

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
                                  via 172.18.2.4, Ethernet2
 B E      172.16.202.1/32 [200/0] via 172.18.1.4, Ethernet1
                                  via 172.18.2.4, Ethernet2
 C        172.16.203.1/32 is directly connected, Loopback1
 B E      172.17.101.1/32 [200/0] via 172.18.1.4, Ethernet1
 B E      172.17.102.1/32 [200/0] via 172.18.2.4, Ethernet2
 B E      172.17.201.1/32 [200/0] via 172.18.1.4, Ethernet1
                                  via 172.18.2.4, Ethernet2
 B E      172.17.202.1/32 [200/0] via 172.18.1.4, Ethernet1
                                  via 172.18.2.4, Ethernet2
 C        172.17.203.1/32 is directly connected, Loopback2
 C        172.18.1.4/31 is directly connected, Ethernet1
 C        172.18.2.4/31 is directly connected, Ethernet2

leaf-3#
```

5. Проверка дополнительных настроек BGP и EVPN, наличия маршрутов, MLAG'ов

**Коммутатор spine-1**

```
spine-1#show ip bgp summary
BGP summary information for VRF default
Router identifier 172.16.101.1, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  172.16.201.1 4 65001           1530      1501    0    0 01:01:54 Estab   2      2
  172.16.202.1 4 65002           2393      2356    0    0 01:35:16 Estab   2      2
  172.16.203.1 4 65003           2396      2389    0    0 01:35:15 Estab   2      2
  172.18.1.1   4 65001           1478      1463    0    0 01:02:00 Estab   4      2
  172.18.1.3   4 65002           1458      1459    0    0 01:01:58 Estab   6      2
  172.18.1.5   4 65003           1456      1461    0    0 01:01:57 Estab   4      2

spine-1#show bgp evpn
BGP routing table information for VRF default
Router identifier 172.16.101.1, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:10010 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65001 i
 * >      RD: 65001:10011 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65001 i
 * >      RD: 65001:10101 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65001 i
 * >      RD: 65002:10010 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65002 i
 * >      RD: 65002:10011 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65002 i
 * >      RD: 65002:10101 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65002 i
 * >      RD: 65003:10010 mac-ip 0000.0000.0002
                                 172.16.203.1          -       100     0       65003 i
 * >      RD: 65003:10011 mac-ip 0000.0000.0002
                                 172.16.203.1          -       100     0       65003 i
 * >      RD: 65001:10010 mac-ip 0050.7966.6806
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65002:10010 mac-ip 0050.7966.6806
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65001:10010 mac-ip 0050.7966.6806 192.168.10.1
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65003:10011 mac-ip 0050.7966.6807
                                 172.17.203.1          -       100     0       65003 i
 * >      RD: 65003:10010 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65003 i
 * >      RD: 65001:10101 mac-ip 0050.7966.6809
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65002:10101 mac-ip 0050.7966.6809
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65001:10101 mac-ip 0050.7966.6809 101.101.101.1
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10101 mac-ip 5000.00ae.f703
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65002:10101 mac-ip 5000.00ae.f703
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65001:10101 mac-ip 5000.00ae.f703 101.101.101.100
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10010 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10011 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65002:10010 mac-ip 5000.00af.d3f6
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65002:10011 mac-ip 5000.00af.d3f6
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65001:10010 mac-ip 5000.00af.d3f6 192.168.10.10
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10011 mac-ip 5000.00af.d3f6 192.168.11.11
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10010 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10011 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10101 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65002:10010 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65002:10011 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65002:10101 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65003:10010 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65003 i
 * >      RD: 65003:10011 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65003 i
 * >      RD: 65001:10010 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10011 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10101 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65002:10010 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65002:10011 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65002:10101 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65003:10010 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65003 i
 * >      RD: 65003:10011 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65003 i
 * >      RD: 65001:101 ip-prefix 0.0.0.0/0
                                 172.17.201.1          -       100     0       65001 65050 ?
 * >      RD: 65001:20000 ip-prefix 0.0.0.0/0
                                 172.17.201.1          -       100     0       65001 65050 ?
 * >      RD: 65001:101 ip-prefix 5.5.5.5/32
                                 172.17.201.1          -       100     0       65001 65050 i
 * >      RD: 65001:20000 ip-prefix 5.5.5.5/32
                                 172.17.201.1          -       100     0       65001 65050 i
 * >      RD: 65001:101 ip-prefix 100.100.100.0/24
                                 172.17.201.1          -       100     0       65001 65050 i
 * >      RD: 65001:20000 ip-prefix 100.100.100.0/24
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:101 ip-prefix 101.101.101.0/24
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:20000 ip-prefix 101.101.101.0/24
                                 172.17.201.1          -       100     0       65001 65050 i
 * >      RD: 65001:20000 ip-prefix 192.168.10.0/24
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:20000 ip-prefix 192.168.11.0/24
                                 172.17.201.1          -       100     0       65001 i
spine-1#
```

**Коммутатор spine-2**

```
spine-2#show ip bgp summary
BGP summary information for VRF default
Router identifier 172.16.102.1, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  172.16.201.1 4 65001           2375      2331    0    0 01:35:19 Estab   2      2
  172.16.202.1 4 65002           2332      2365    0    0 01:35:25 Estab   2      2
  172.16.203.1 4 65003           2336      2408    0    0 01:35:24 Estab   2      2
  172.18.2.1   4 65001           1457      1473    0    0 01:02:02 Estab   8      2
  172.18.2.3   4 65002           1466      1466    0    0 01:02:05 Estab   6      2
  172.18.2.5   4 65003           1463      1467    0    0 01:02:06 Estab   8      2

spine-2#show bgp evpn
BGP routing table information for VRF default
Router identifier 172.16.102.1, local AS number 65000
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 65001:10010 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65001 i
 * >      RD: 65001:10011 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65001 i
 * >      RD: 65001:10101 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65001 i
 * >      RD: 65002:10010 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65002 i
 * >      RD: 65002:10011 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65002 i
 * >      RD: 65002:10101 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65002 i
 * >      RD: 65003:10010 mac-ip 0000.0000.0002
                                 172.16.203.1          -       100     0       65003 i
 * >      RD: 65003:10011 mac-ip 0000.0000.0002
                                 172.16.203.1          -       100     0       65003 i
 * >      RD: 65001:10010 mac-ip 0050.7966.6806
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65002:10010 mac-ip 0050.7966.6806
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65001:10010 mac-ip 0050.7966.6806 192.168.10.1
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65003:10011 mac-ip 0050.7966.6807
                                 172.17.203.1          -       100     0       65003 i
 * >      RD: 65003:10010 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65003 i
 * >      RD: 65001:10101 mac-ip 0050.7966.6809
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65002:10101 mac-ip 0050.7966.6809
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65001:10101 mac-ip 0050.7966.6809 101.101.101.1
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10101 mac-ip 5000.00ae.f703
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65002:10101 mac-ip 5000.00ae.f703
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65001:10101 mac-ip 5000.00ae.f703 101.101.101.100
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10010 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10011 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65002:10010 mac-ip 5000.00af.d3f6
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65002:10011 mac-ip 5000.00af.d3f6
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65001:10010 mac-ip 5000.00af.d3f6 192.168.10.10
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10011 mac-ip 5000.00af.d3f6 192.168.11.11
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10010 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10011 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10101 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65002:10010 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65002:10011 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65002:10101 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65003:10010 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65003 i
 * >      RD: 65003:10011 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65003 i
 * >      RD: 65001:10010 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10011 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:10101 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65002:10010 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65002:10011 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65002:10101 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65002 i
 * >      RD: 65003:10010 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65003 i
 * >      RD: 65003:10011 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65003 i
 * >      RD: 65001:101 ip-prefix 0.0.0.0/0
                                 172.17.201.1          -       100     0       65001 65050 ?
 * >      RD: 65001:20000 ip-prefix 0.0.0.0/0
                                 172.17.201.1          -       100     0       65001 65050 ?
 * >      RD: 65001:101 ip-prefix 5.5.5.5/32
                                 172.17.201.1          -       100     0       65001 65050 i
 * >      RD: 65001:20000 ip-prefix 5.5.5.5/32
                                 172.17.201.1          -       100     0       65001 65050 i
 * >      RD: 65001:101 ip-prefix 100.100.100.0/24
                                 172.17.201.1          -       100     0       65001 65050 i
 * >      RD: 65001:20000 ip-prefix 100.100.100.0/24
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:101 ip-prefix 101.101.101.0/24
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:20000 ip-prefix 101.101.101.0/24
                                 172.17.201.1          -       100     0       65001 65050 i
 * >      RD: 65001:20000 ip-prefix 192.168.10.0/24
                                 172.17.201.1          -       100     0       65001 i
 * >      RD: 65001:20000 ip-prefix 192.168.11.0/24
                                 172.17.201.1          -       100     0       65001 i
spine-2#
```

**Коммутатор ext**

```
ext#show ip bgp summary
BGP summary information for VRF default
Router identifier 5.5.5.5, local AS number 65050
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  100.100.100.100  4  65001           4078      3476    0    0 01:02:08 Estab   3      3
  101.101.101.101  4  65001           4395      3767    0    0 01:02:09 Estab   1      1
ext#
```
**Коммутатор leaf-1**

```
leaf-1#
leaf-1#show ip bgp summary
BGP summary information for VRF default
Router identifier 172.16.201.1, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  172.16.101.1 4 65000           2327      2408    0    0 01:02:18 Estab   6      6
  172.16.102.1 4 65000           2337      2381    0    0 01:35:35 Estab   6      6
  172.18.1.0   4 65000           2260      2271    0    0 01:02:25 Estab   6      6
  172.18.2.0   4 65000           2268      2251    0    0 01:02:18 Estab   6      6

leaf-1#
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
 * >      RD: 65001:10101 mac-ip 0000.0000.0001
                                 172.16.201.1          -       -       0       i
 * >Ec    RD: 65002:10010 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10010 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10011 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10101 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10101 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65003:10010 mac-ip 0000.0000.0002
                                 172.16.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10010 mac-ip 0000.0000.0002
                                 172.16.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65003:10011 mac-ip 0000.0000.0002
                                 172.16.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10011 mac-ip 0000.0000.0002
                                 172.16.203.1          -       100     0       65000 65003 i
 * >      RD: 65001:10010 mac-ip 0050.7966.6806
                                 -                     -       -       0       i
 * >Ec    RD: 65002:10010 mac-ip 0050.7966.6806
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10010 mac-ip 0050.7966.6806
                                 172.17.202.1          -       100     0       65000 65002 i
 * >      RD: 65001:10010 mac-ip 0050.7966.6806 192.168.10.1
                                 -                     -       -       0       i
 * >Ec    RD: 65003:10011 mac-ip 0050.7966.6807
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10011 mac-ip 0050.7966.6807
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65003:10010 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10010 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65000 65003 i
 * >      RD: 65001:10101 mac-ip 0050.7966.6809
                                 -                     -       -       0       i
 * >Ec    RD: 65002:10101 mac-ip 0050.7966.6809
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10101 mac-ip 0050.7966.6809
                                 172.17.202.1          -       100     0       65000 65002 i
 * >      RD: 65001:10101 mac-ip 0050.7966.6809 101.101.101.1
                                 -                     -       -       0       i
 * >      RD: 65001:10101 mac-ip 5000.00ae.f703
                                 -                     -       -       0       i
 * >Ec    RD: 65002:10101 mac-ip 5000.00ae.f703
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10101 mac-ip 5000.00ae.f703
                                 172.17.202.1          -       100     0       65000 65002 i
 * >      RD: 65001:10101 mac-ip 5000.00ae.f703 101.101.101.100
                                 -                     -       -       0       i
 * >      RD: 65001:10010 mac-ip 5000.00af.d3f6
                                 -                     -       -       0       i
 * >      RD: 65001:10011 mac-ip 5000.00af.d3f6
                                 -                     -       -       0       i
 * >Ec    RD: 65002:10010 mac-ip 5000.00af.d3f6
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10010 mac-ip 5000.00af.d3f6
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10011 mac-ip 5000.00af.d3f6
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 mac-ip 5000.00af.d3f6
                                 172.17.202.1          -       100     0       65000 65002 i
 * >      RD: 65001:10010 mac-ip 5000.00af.d3f6 192.168.10.10
                                 -                     -       -       0       i
 * >      RD: 65001:10011 mac-ip 5000.00af.d3f6 192.168.11.11
                                 -                     -       -       0       i
 * >      RD: 65001:10010 imet 172.16.201.1
                                 -                     -       -       0       i
 * >      RD: 65001:10011 imet 172.16.201.1
                                 -                     -       -       0       i
 * >      RD: 65001:10101 imet 172.16.201.1
                                 -                     -       -       0       i
 * >Ec    RD: 65002:10010 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10010 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10011 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10101 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10101 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65003:10010 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10010 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65003:10011 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10011 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >      RD: 65001:10010 imet 172.17.201.1
                                 -                     -       -       0       i
 * >      RD: 65001:10011 imet 172.17.201.1
                                 -                     -       -       0       i
 * >      RD: 65001:10101 imet 172.17.201.1
                                 -                     -       -       0       i
 * >Ec    RD: 65002:10010 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10010 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10011 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10101 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10101 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65003:10010 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10010 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65003:10011 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10011 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >      RD: 65001:101 ip-prefix 0.0.0.0/0
                                 -                     -       100     0       65050 ?
 * >      RD: 65001:20000 ip-prefix 0.0.0.0/0
                                 -                     -       100     0       65050 ?
 * >      RD: 65001:101 ip-prefix 5.5.5.5/32
                                 -                     -       100     0       65050 i
 * >      RD: 65001:20000 ip-prefix 5.5.5.5/32
                                 -                     -       100     0       65050 i
 * >      RD: 65001:101 ip-prefix 100.100.100.0/24
                                 -                     -       100     0       65050 i
 * >      RD: 65001:20000 ip-prefix 100.100.100.0/24
                                 -                     -       -       0       i
 *        RD: 65001:20000 ip-prefix 100.100.100.0/24
                                 -                     -       100     0       65050 i
 * >      RD: 65001:101 ip-prefix 101.101.101.0/24
                                 -                     -       -       0       i
 *        RD: 65001:101 ip-prefix 101.101.101.0/24
                                 -                     -       100     0       65050 i
 * >      RD: 65001:20000 ip-prefix 101.101.101.0/24
                                 -                     -       100     0       65050 i
 * >      RD: 65001:20000 ip-prefix 192.168.10.0/24
                                 -                     -       -       0       i
 * >      RD: 65001:20000 ip-prefix 192.168.11.0/24
                                 -                     -       -       0       i

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


leaf-1#show port-channel 10
Port Channel Port-Channel10:
  Active Ports: Ethernet7 PeerEthernet7

leaf-1#show port-channel 50
Port Channel Port-Channel50:
  Active Ports: Ethernet3 Ethernet4
leaf-1#
```

**Коммутатор leaf-2**

```
leaf-2#show ip bgp summary
BGP summary information for VRF default
Router identifier 172.16.202.1, local AS number 65002
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  172.16.101.1 4 65000           2369      2406    0    0 01:35:50 Estab   6      6
  172.16.102.1 4 65000           2376      2342    0    0 01:35:51 Estab   6      6
  172.18.1.2   4 65000           2263      2272    0    0 01:02:31 Estab   6      6
  172.18.2.2   4 65000           2265      2269    0    0 01:02:31 Estab   6      6

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
 * >Ec    RD: 65001:10101 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10101 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65000 65001 i
 * >      RD: 65002:10010 mac-ip 0000.0000.0002
                                 172.16.202.1          -       -       0       i
 * >      RD: 65002:10011 mac-ip 0000.0000.0002
                                 172.16.202.1          -       -       0       i
 * >      RD: 65002:10101 mac-ip 0000.0000.0002
                                 172.16.202.1          -       -       0       i
 * >Ec    RD: 65003:10010 mac-ip 0000.0000.0002
                                 172.16.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10010 mac-ip 0000.0000.0002
                                 172.16.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65003:10011 mac-ip 0000.0000.0002
                                 172.16.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10011 mac-ip 0000.0000.0002
                                 172.16.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6806
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6806
                                 172.17.201.1          -       100     0       65000 65001 i
 * >      RD: 65002:10010 mac-ip 0050.7966.6806
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6806 192.168.10.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6806 192.168.10.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65003:10011 mac-ip 0050.7966.6807
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10011 mac-ip 0050.7966.6807
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65003:10010 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10010 mac-ip 0050.7966.6808
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:10101 mac-ip 0050.7966.6809
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10101 mac-ip 0050.7966.6809
                                 172.17.201.1          -       100     0       65000 65001 i
 * >      RD: 65002:10101 mac-ip 0050.7966.6809
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10101 mac-ip 0050.7966.6809 101.101.101.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10101 mac-ip 0050.7966.6809 101.101.101.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10101 mac-ip 5000.00ae.f703
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10101 mac-ip 5000.00ae.f703
                                 172.17.201.1          -       100     0       65000 65001 i
 * >      RD: 65002:10101 mac-ip 5000.00ae.f703
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10101 mac-ip 5000.00ae.f703 101.101.101.100
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10101 mac-ip 5000.00ae.f703 101.101.101.100
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10010 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10011 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10011 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65000 65001 i
 * >      RD: 65002:10010 mac-ip 5000.00af.d3f6
                                 -                     -       -       0       i
 * >      RD: 65002:10011 mac-ip 5000.00af.d3f6
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10010 mac-ip 5000.00af.d3f6 192.168.10.10
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 mac-ip 5000.00af.d3f6 192.168.10.10
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10011 mac-ip 5000.00af.d3f6 192.168.11.11
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10011 mac-ip 5000.00af.d3f6 192.168.11.11
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10010 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10011 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10011 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10101 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10101 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >      RD: 65002:10010 imet 172.16.202.1
                                 -                     -       -       0       i
 * >      RD: 65002:10011 imet 172.16.202.1
                                 -                     -       -       0       i
 * >      RD: 65002:10101 imet 172.16.202.1
                                 -                     -       -       0       i
 * >Ec    RD: 65003:10010 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10010 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65003:10011 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10011 imet 172.16.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:10010 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10011 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10011 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10101 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10101 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >      RD: 65002:10010 imet 172.17.202.1
                                 -                     -       -       0       i
 * >      RD: 65002:10011 imet 172.17.202.1
                                 -                     -       -       0       i
 * >      RD: 65002:10101 imet 172.17.202.1
                                 -                     -       -       0       i
 * >Ec    RD: 65003:10010 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10010 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65003:10011 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 *  ec    RD: 65003:10011 imet 172.17.203.1
                                 172.17.203.1          -       100     0       65000 65003 i
 * >Ec    RD: 65001:101 ip-prefix 0.0.0.0/0
                                 172.17.201.1          -       100     0       65000 65001 65050 ?
 *  ec    RD: 65001:101 ip-prefix 0.0.0.0/0
                                 172.17.201.1          -       100     0       65000 65001 65050 ?
 * >Ec    RD: 65001:20000 ip-prefix 0.0.0.0/0
                                 172.17.201.1          -       100     0       65000 65001 65050 ?
 *  ec    RD: 65001:20000 ip-prefix 0.0.0.0/0
                                 172.17.201.1          -       100     0       65000 65001 65050 ?
 * >Ec    RD: 65001:101 ip-prefix 5.5.5.5/32
                                 172.17.201.1          -       100     0       65000 65001 65050 i
 *  ec    RD: 65001:101 ip-prefix 5.5.5.5/32
                                 172.17.201.1          -       100     0       65000 65001 65050 i
 * >Ec    RD: 65001:20000 ip-prefix 5.5.5.5/32
                                 172.17.201.1          -       100     0       65000 65001 65050 i
 *  ec    RD: 65001:20000 ip-prefix 5.5.5.5/32
                                 172.17.201.1          -       100     0       65000 65001 65050 i
 * >Ec    RD: 65001:101 ip-prefix 100.100.100.0/24
                                 172.17.201.1          -       100     0       65000 65001 65050 i
 *  ec    RD: 65001:101 ip-prefix 100.100.100.0/24
                                 172.17.201.1          -       100     0       65000 65001 65050 i
 * >Ec    RD: 65001:20000 ip-prefix 100.100.100.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:20000 ip-prefix 100.100.100.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:101 ip-prefix 101.101.101.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:101 ip-prefix 101.101.101.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:20000 ip-prefix 101.101.101.0/24
                                 172.17.201.1          -       100     0       65000 65001 65050 i
 *  ec    RD: 65001:20000 ip-prefix 101.101.101.0/24
                                 172.17.201.1          -       100     0       65000 65001 65050 i
 * >Ec    RD: 65001:20000 ip-prefix 192.168.10.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:20000 ip-prefix 192.168.10.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:20000 ip-prefix 192.168.11.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:20000 ip-prefix 192.168.11.0/24
                                 172.17.201.1          -       100     0       65000 65001 i

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

leaf-2#show port-channel 10
Port Channel Port-Channel10:
  Active Ports: Ethernet7 PeerEthernet7

leaf-2#show port-channel 50
Port Channel Port-Channel50:
  Active Ports: Ethernet3 Ethernet4
leaf-2#
```

**Коммутатор leaf-3**

```
leaf-3#show ip bgp summary
BGP summary information for VRF default
Router identifier 172.16.203.1, local AS number 65003
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  172.16.101.1 4 65000           2406      2413    0    0 01:35:58 Estab   6      6
  172.16.102.1 4 65000           2421      2349    0    0 01:35:58 Estab   6      6
  172.18.1.4   4 65000           2274      2272    0    0 01:02:39 Estab   6      6
  172.18.2.4   4 65000           2270      2266    0    0 01:02:40 Estab   6      6

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
 * >Ec    RD: 65001:10101 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10101 mac-ip 0000.0000.0001
                                 172.16.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65002:10010 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10010 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10011 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10101 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10101 mac-ip 0000.0000.0002
                                 172.16.202.1          -       100     0       65000 65002 i
 * >      RD: 65003:10010 mac-ip 0000.0000.0002
                                 172.16.203.1          -       -       0       i
 * >      RD: 65003:10011 mac-ip 0000.0000.0002
                                 172.16.203.1          -       -       0       i
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6806
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6806
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65002:10010 mac-ip 0050.7966.6806
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10010 mac-ip 0050.7966.6806
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65001:10010 mac-ip 0050.7966.6806 192.168.10.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 mac-ip 0050.7966.6806 192.168.10.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >      RD: 65003:10011 mac-ip 0050.7966.6807
                                 -                     -       -       0       i
 * >      RD: 65003:10010 mac-ip 0050.7966.6808
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10101 mac-ip 0050.7966.6809
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10101 mac-ip 0050.7966.6809
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65002:10101 mac-ip 0050.7966.6809
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10101 mac-ip 0050.7966.6809
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65001:10101 mac-ip 0050.7966.6809 101.101.101.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10101 mac-ip 0050.7966.6809 101.101.101.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10101 mac-ip 5000.00ae.f703
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10101 mac-ip 5000.00ae.f703
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65002:10101 mac-ip 5000.00ae.f703
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10101 mac-ip 5000.00ae.f703
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65001:10101 mac-ip 5000.00ae.f703 101.101.101.100
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10101 mac-ip 5000.00ae.f703 101.101.101.100
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10010 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10011 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10011 mac-ip 5000.00af.d3f6
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65002:10010 mac-ip 5000.00af.d3f6
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10010 mac-ip 5000.00af.d3f6
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10011 mac-ip 5000.00af.d3f6
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 mac-ip 5000.00af.d3f6
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65001:10010 mac-ip 5000.00af.d3f6 192.168.10.10
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 mac-ip 5000.00af.d3f6 192.168.10.10
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10011 mac-ip 5000.00af.d3f6 192.168.11.11
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10011 mac-ip 5000.00af.d3f6 192.168.11.11
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10010 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10011 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10011 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10101 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10101 imet 172.16.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65002:10010 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10010 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10011 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10101 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10101 imet 172.16.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >      RD: 65003:10010 imet 172.16.203.1
                                 -                     -       -       0       i
 * >      RD: 65003:10011 imet 172.16.203.1
                                 -                     -       -       0       i
 * >Ec    RD: 65001:10010 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10010 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10011 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10011 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:10101 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:10101 imet 172.17.201.1
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65002:10010 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10010 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10011 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10011 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >Ec    RD: 65002:10101 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 *  ec    RD: 65002:10101 imet 172.17.202.1
                                 172.17.202.1          -       100     0       65000 65002 i
 * >      RD: 65003:10010 imet 172.17.203.1
                                 -                     -       -       0       i
 * >      RD: 65003:10011 imet 172.17.203.1
                                 -                     -       -       0       i
 * >Ec    RD: 65001:101 ip-prefix 0.0.0.0/0
                                 172.17.201.1          -       100     0       65000 65001 65050 ?
 *  ec    RD: 65001:101 ip-prefix 0.0.0.0/0
                                 172.17.201.1          -       100     0       65000 65001 65050 ?
 * >Ec    RD: 65001:20000 ip-prefix 0.0.0.0/0
                                 172.17.201.1          -       100     0       65000 65001 65050 ?
 *  ec    RD: 65001:20000 ip-prefix 0.0.0.0/0
                                 172.17.201.1          -       100     0       65000 65001 65050 ?
 * >Ec    RD: 65001:101 ip-prefix 5.5.5.5/32
                                 172.17.201.1          -       100     0       65000 65001 65050 i
 *  ec    RD: 65001:101 ip-prefix 5.5.5.5/32
                                 172.17.201.1          -       100     0       65000 65001 65050 i
 * >Ec    RD: 65001:20000 ip-prefix 5.5.5.5/32
                                 172.17.201.1          -       100     0       65000 65001 65050 i
 *  ec    RD: 65001:20000 ip-prefix 5.5.5.5/32
                                 172.17.201.1          -       100     0       65000 65001 65050 i
 * >Ec    RD: 65001:101 ip-prefix 100.100.100.0/24
                                 172.17.201.1          -       100     0       65000 65001 65050 i
 *  ec    RD: 65001:101 ip-prefix 100.100.100.0/24
                                 172.17.201.1          -       100     0       65000 65001 65050 i
 * >Ec    RD: 65001:20000 ip-prefix 100.100.100.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:20000 ip-prefix 100.100.100.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:101 ip-prefix 101.101.101.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:101 ip-prefix 101.101.101.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:20000 ip-prefix 101.101.101.0/24
                                 172.17.201.1          -       100     0       65000 65001 65050 i
 *  ec    RD: 65001:20000 ip-prefix 101.101.101.0/24
                                 172.17.201.1          -       100     0       65000 65001 65050 i
 * >Ec    RD: 65001:20000 ip-prefix 192.168.10.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:20000 ip-prefix 192.168.10.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 * >Ec    RD: 65001:20000 ip-prefix 192.168.11.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
 *  ec    RD: 65001:20000 ip-prefix 192.168.11.0/24
                                 172.17.201.1          -       100     0       65000 65001 i
leaf-3#
```


7. Проверка связности между клиентскими устройствами утилитой **ping**.

**client-1**

```
client-1> ping 192.168.10.2

84 bytes from 192.168.10.2 icmp_seq=1 ttl=64 time=388.442 ms
84 bytes from 192.168.10.2 icmp_seq=2 ttl=64 time=44.840 ms
84 bytes from 192.168.10.2 icmp_seq=3 ttl=64 time=43.758 ms
84 bytes from 192.168.10.2 icmp_seq=4 ttl=64 time=32.877 ms
84 bytes from 192.168.10.2 icmp_seq=5 ttl=64 time=120.265 ms

client-1> ping 192.168.11.1

84 bytes from 192.168.11.1 icmp_seq=1 ttl=63 time=317.791 ms
84 bytes from 192.168.11.1 icmp_seq=2 ttl=63 time=61.564 ms
84 bytes from 192.168.11.1 icmp_seq=3 ttl=63 time=47.321 ms
84 bytes from 192.168.11.1 icmp_seq=4 ttl=63 time=39.922 ms
84 bytes from 192.168.11.1 icmp_seq=5 ttl=63 time=47.904 ms

client-1> ping 192.168.10.10

84 bytes from 192.168.10.10 icmp_seq=1 ttl=64 time=106.006 ms
84 bytes from 192.168.10.10 icmp_seq=2 ttl=64 time=37.521 ms
84 bytes from 192.168.10.10 icmp_seq=3 ttl=64 time=20.514 ms
84 bytes from 192.168.10.10 icmp_seq=4 ttl=64 time=19.356 ms
84 bytes from 192.168.10.10 icmp_seq=5 ttl=64 time=18.753 ms

client-1> ping 192.168.11.11

84 bytes from 192.168.11.11 icmp_seq=1 ttl=64 time=95.707 ms
84 bytes from 192.168.11.11 icmp_seq=2 ttl=64 time=24.498 ms
84 bytes from 192.168.11.11 icmp_seq=3 ttl=64 time=28.905 ms
84 bytes from 192.168.11.11 icmp_seq=4 ttl=64 time=42.718 ms
84 bytes from 192.168.11.11 icmp_seq=5 ttl=64 time=25.076 ms

client-1> ping 101.101.101.1

84 bytes from 101.101.101.1 icmp_seq=1 ttl=61 time=319.602 ms
84 bytes from 101.101.101.1 icmp_seq=2 ttl=61 time=210.943 ms
84 bytes from 101.101.101.1 icmp_seq=3 ttl=61 time=65.325 ms
84 bytes from 101.101.101.1 icmp_seq=4 ttl=61 time=78.160 ms
84 bytes from 101.101.101.1 icmp_seq=5 ttl=61 time=67.629 ms

client-1> ping 5.5.5.5

84 bytes from 5.5.5.5 icmp_seq=1 ttl=63 time=30.463 ms
84 bytes from 5.5.5.5 icmp_seq=2 ttl=63 time=118.851 ms
84 bytes from 5.5.5.5 icmp_seq=3 ttl=63 time=23.894 ms
84 bytes from 5.5.5.5 icmp_seq=4 ttl=63 time=28.943 ms
84 bytes from 5.5.5.5 icmp_seq=5 ttl=63 time=21.839 ms

```

**client-2**

```
lient-2> ping 192.168.10.2

84 bytes from 192.168.10.2 icmp_seq=1 ttl=62 time=318.347 ms
84 bytes from 192.168.10.2 icmp_seq=2 ttl=62 time=138.225 ms
84 bytes from 192.168.10.2 icmp_seq=3 ttl=62 time=134.960 ms
84 bytes from 192.168.10.2 icmp_seq=4 ttl=62 time=165.742 ms
84 bytes from 192.168.10.2 icmp_seq=5 ttl=62 time=181.071 ms

client-2> ping 192.168.11.1

84 bytes from 192.168.11.1 icmp_seq=1 ttl=62 time=278.964 ms
84 bytes from 192.168.11.1 icmp_seq=2 ttl=62 time=103.900 ms
84 bytes from 192.168.11.1 icmp_seq=3 ttl=62 time=97.676 ms
84 bytes from 192.168.11.1 icmp_seq=4 ttl=62 time=129.918 ms
84 bytes from 192.168.11.1 icmp_seq=5 ttl=62 time=105.770 ms
ping 192.168.10.10
client-2> ping 192.168.10.10

84 bytes from 192.168.10.10 icmp_seq=1 ttl=62 time=97.967 ms
84 bytes from 192.168.10.10 icmp_seq=2 ttl=62 time=92.838 ms
84 bytes from 192.168.10.10 icmp_seq=3 ttl=62 time=132.653 ms
84 bytes from 192.168.10.10 icmp_seq=4 ttl=62 time=86.372 ms
84 bytes from 192.168.10.10 icmp_seq=5 ttl=62 time=138.247 ms

client-2> ping 192.168.11.11

84 bytes from 192.168.11.11 icmp_seq=1 ttl=62 time=99.131 ms
84 bytes from 192.168.11.11 icmp_seq=2 ttl=62 time=82.082 ms
84 bytes from 192.168.11.11 icmp_seq=3 ttl=62 time=88.682 ms
84 bytes from 192.168.11.11 icmp_seq=4 ttl=62 time=102.215 ms
84 bytes from 192.168.11.11 icmp_seq=5 ttl=62 time=70.492 ms

client-2> ping 5.5.5.5

84 bytes from 5.5.5.5 icmp_seq=1 ttl=64 time=47.056 ms
84 bytes from 5.5.5.5 icmp_seq=2 ttl=64 time=44.897 ms
84 bytes from 5.5.5.5 icmp_seq=3 ttl=64 time=35.440 ms
84 bytes from 5.5.5.5 icmp_seq=4 ttl=64 time=35.668 ms
84 bytes from 5.5.5.5 icmp_seq=5 ttl=64 time=90.157 ms
```

**client-3**

```
client-3> ping 192.168.10.1

84 bytes from 192.168.10.1 icmp_seq=1 ttl=64 time=44.742 ms
84 bytes from 192.168.10.1 icmp_seq=2 ttl=64 time=32.655 ms
84 bytes from 192.168.10.1 icmp_seq=3 ttl=64 time=39.804 ms
84 bytes from 192.168.10.1 icmp_seq=4 ttl=64 time=40.469 ms
84 bytes from 192.168.10.1 icmp_seq=5 ttl=64 time=43.768 ms

client-3> ping 192.168.11.1

84 bytes from 192.168.11.1 icmp_seq=1 ttl=63 time=84.595 ms
84 bytes from 192.168.11.1 icmp_seq=2 ttl=63 time=61.758 ms
84 bytes from 192.168.11.1 icmp_seq=3 ttl=63 time=74.567 ms
84 bytes from 192.168.11.1 icmp_seq=4 ttl=63 time=192.917 ms
84 bytes from 192.168.11.1 icmp_seq=5 ttl=63 time=77.765 ms

client-3> ping 192.168.10.10

84 bytes from 192.168.10.10 icmp_seq=1 ttl=64 time=54.927 ms
84 bytes from 192.168.10.10 icmp_seq=2 ttl=64 time=116.644 ms
84 bytes from 192.168.10.10 icmp_seq=3 ttl=64 time=41.858 ms
84 bytes from 192.168.10.10 icmp_seq=4 ttl=64 time=53.906 ms
84 bytes from 192.168.10.10 icmp_seq=5 ttl=64 time=61.810 ms

client-3> ping 192.168.11.11

84 bytes from 192.168.11.11 icmp_seq=1 ttl=64 time=55.511 ms
84 bytes from 192.168.11.11 icmp_seq=2 ttl=64 time=43.924 ms
84 bytes from 192.168.11.11 icmp_seq=3 ttl=64 time=46.339 ms
84 bytes from 192.168.11.11 icmp_seq=4 ttl=64 time=45.592 ms
84 bytes from 192.168.11.11 icmp_seq=5 ttl=64 time=162.343 ms
```

**client-4**

```
client-4> ping 192.168.10.1

84 bytes from 192.168.10.1 icmp_seq=1 ttl=63 time=69.558 ms
84 bytes from 192.168.10.1 icmp_seq=2 ttl=63 time=50.862 ms
84 bytes from 192.168.10.1 icmp_seq=3 ttl=63 time=39.300 ms
84 bytes from 192.168.10.1 icmp_seq=4 ttl=63 time=47.733 ms
84 bytes from 192.168.10.1 icmp_seq=5 ttl=63 time=41.803 ms

client-4> ping 192.168.10.10

84 bytes from 192.168.10.10 icmp_seq=1 ttl=64 time=82.617 ms
84 bytes from 192.168.10.10 icmp_seq=2 ttl=64 time=74.239 ms
84 bytes from 192.168.10.10 icmp_seq=3 ttl=64 time=50.125 ms
84 bytes from 192.168.10.10 icmp_seq=4 ttl=64 time=51.404 ms
84 bytes from 192.168.10.10 icmp_seq=5 ttl=64 time=44.686 ms

client-4> ping 192.168.11.11

84 bytes from 192.168.11.11 icmp_seq=1 ttl=64 time=67.708 ms
84 bytes from 192.168.11.11 icmp_seq=2 ttl=64 time=45.223 ms
84 bytes from 192.168.11.11 icmp_seq=3 ttl=64 time=46.963 ms
84 bytes from 192.168.11.11 icmp_seq=4 ttl=64 time=49.455 ms
84 bytes from 192.168.11.11 icmp_seq=5 ttl=64 time=46.103 ms

client-4> ping 101.101.101.1

84 bytes from 101.101.101.1 icmp_seq=1 ttl=61 time=182.338 ms
84 bytes from 101.101.101.1 icmp_seq=2 ttl=61 time=94.718 ms
84 bytes from 101.101.101.1 icmp_seq=3 ttl=61 time=105.471 ms
84 bytes from 101.101.101.1 icmp_seq=4 ttl=61 time=92.539 ms
84 bytes from 101.101.101.1 icmp_seq=5 ttl=61 time=86.847 ms

client-4> ping 5.5.5.5

84 bytes from 5.5.5.5 icmp_seq=1 ttl=63 time=47.910 ms
84 bytes from 5.5.5.5 icmp_seq=2 ttl=63 time=48.374 ms
84 bytes from 5.5.5.5 icmp_seq=3 ttl=63 time=49.341 ms
84 bytes from 5.5.5.5 icmp_seq=4 ttl=63 time=50.939 ms
84 bytes from 5.5.5.5 icmp_seq=5 ttl=63 time=71.259 ms
```

**srv**

```
srv(config)#ping 192.168.10.1
PING 192.168.10.1 (192.168.10.1) 72(100) bytes of data.
80 bytes from 192.168.10.1: icmp_seq=1 ttl=64 time=50.8 ms
80 bytes from 192.168.10.1: icmp_seq=2 ttl=64 time=44.4 ms
80 bytes from 192.168.10.1: icmp_seq=3 ttl=64 time=46.3 ms
80 bytes from 192.168.10.1: icmp_seq=4 ttl=64 time=46.4 ms
80 bytes from 192.168.10.1: icmp_seq=5 ttl=64 time=46.0 ms

--- 192.168.10.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 47ms
rtt min/avg/max/mdev = 44.466/46.825/50.831/2.131 ms, pipe 5, ipg/ewma 11.956/48.790 ms
srv(config)#
srv(config)#ping 192.168.11.1
PING 192.168.11.1 (192.168.11.1) 72(100) bytes of data.
80 bytes from 192.168.11.1: icmp_seq=1 ttl=64 time=754 ms
80 bytes from 192.168.11.1: icmp_seq=2 ttl=64 time=746 ms
80 bytes from 192.168.11.1: icmp_seq=3 ttl=64 time=745 ms
80 bytes from 192.168.11.1: icmp_seq=4 ttl=64 time=798 ms
80 bytes from 192.168.11.1: icmp_seq=5 ttl=64 time=796 ms

--- 192.168.11.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 49ms
rtt min/avg/max/mdev = 745.842/768.380/798.676/24.063 ms, pipe 5, ipg/ewma 12.324/763.146 ms
srv(config)#
srv(config)#ping 101.101.101.1
PING 101.101.101.1 (101.101.101.1) 72(100) bytes of data.
80 bytes from 101.101.101.1: icmp_seq=1 ttl=61 time=178 ms
80 bytes from 101.101.101.1: icmp_seq=2 ttl=61 time=190 ms
80 bytes from 101.101.101.1: icmp_seq=3 ttl=61 time=199 ms
80 bytes from 101.101.101.1: icmp_seq=4 ttl=61 time=214 ms
80 bytes from 101.101.101.1: icmp_seq=5 ttl=61 time=222 ms

--- 101.101.101.1 ping statistics ---
5 packets transmitted, 5 received, +3 duplicates, 0% packet loss, time 47ms
rtt min/avg/max/mdev = 178.070/207.193/234.752/17.568 ms, pipe 5, ipg/ewma 11.913/200.306 ms
srv(config)#
srv(config)#ping 5.5.5.5
PING 5.5.5.5 (5.5.5.5) 72(100) bytes of data.
80 bytes from 5.5.5.5: icmp_seq=1 ttl=63 time=55.4 ms
80 bytes from 5.5.5.5: icmp_seq=2 ttl=63 time=64.2 ms
80 bytes from 5.5.5.5: icmp_seq=3 ttl=63 time=102 ms
80 bytes from 5.5.5.5: icmp_seq=4 ttl=63 time=172 ms
80 bytes from 5.5.5.5: icmp_seq=5 ttl=63 time=176 ms

--- 5.5.5.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 51ms
rtt min/avg/max/mdev = 55.475/114.303/176.907/51.724 ms, pipe 5, ipg/ewma 12.969/88.674 ms
srv(config)#
```


