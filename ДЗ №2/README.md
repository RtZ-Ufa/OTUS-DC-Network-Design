#  Домашнее задание №2
# Underlay. OSPF
## по теме №5 "Построение Underlay сети(OSPF)"
### Цель: Настроить OSPF для Underlay сети
### Задачи:
+ настроить OSPF в Underlay сети, для IP связанности между всеми сетевыми устройствами;
+ зафиксировать в документации - план работы, адресное пространство, схему сети, конфигурацию устройств;
+ проверить IP-связанность между устройствами в OSPF домене.
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

3. Настройки OSPF приложены в файлах leaf-x.txt и spine-x.txt. Далее выборочно приведены настройки оборудования:

**Коммутатор spine-1**

```
spine-1#sh run
_вывод_ _опущен_
!
hostname spine-1
!
spanning-tree mode mstp
!
interface Ethernet1
   description -L- leaf-1
   no switchport
   ip address 172.18.1.0/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet2
   description -L- leaf-2
   no switchport
   ip address 172.18.1.2/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet3
   description -L- leaf-3
   no switchport
   ip address 172.18.1.4/31
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
_вывод опущен_
!
interface Loopback1
   ip address 172.16.101.1/32
   ip ospf area 0.0.0.0
!
interface Loopback2
   ip address 172.17.101.1/32
   ip ospf area 0.0.0.0
!
interface Management1
!
ip routing
!
router ospf 10
   passive-interface default
   no passive-interface Ethernet1
   no passive-interface Ethernet2
   no passive-interface Ethernet3
   max-lsa 12000
!
end
spine-1#
spine-1#
```

**Коммутатор spine-2**

```
hostname spine-2

interface Ethernet1
   description -L- leaf-1
   no switchport
   ip address 172.18.2.0/31

interface Ethernet2
   description -L- leaf-2
   no switchport
   ip address 172.18.2.2/31

interface Ethernet3
   description -L- leaf-3
   no switchport
   ip address 172.18.2.4/31

interface Loopback1
   ip address 172.16.102.1/32

interface Loopback2
   ip address 172.17.102.1/32

ip routing
```

**Коммутатор leaf-1**

```
hostname leaf-1

interface Ethernet1
   description -S- spine-1
   no switchport
   ip address 172.18.1.1/31

interface Ethernet2
   description -S- spine-2
   no switchport
   ip address 172.18.2.1/31

interface Loopback1
   ip address 172.16.201.1/32

interface Loopback2
   ip address 172.17.201.1/32

ip routing
```

**Коммутатор leaf-2**

```
hostname leaf-2

interface Ethernet1
   description -S- spine-1
   no switchport
   ip address 172.18.1.3/31

interface Ethernet2
   description -S- spine-2
   no switchport
   ip address 172.18.2.3/31

interface Loopback1
   ip address 172.16.202.1/32

interface Loopback2
   ip address 172.17.202.1/32

ip routing
```

**Коммутатор leaf-3**

```
hostname leaf-3

interface Ethernet1
   description -S- spine-1
   no switchport
   ip address 172.18.1.5/31

interface Ethernet2
   description -S- spine-2
   no switchport
   ip address 172.18.2.5/31

interface Loopback1
   ip address 172.16.203.1/32

interface Loopback2
   ip address 172.17.203.1/32

ip routing
```
