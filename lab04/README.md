### BGP на базе Underlay N9k
### Цели
1. Распределить адресное пространство на Underlay сети.
2. Настроить IP связанность между всеми устройствами NXOS.
3. Настроить BGP для на устройствах NXOS.
3. Настроить аутентификацию на интерфесах между BGP соседями.
4. Проверить соседство и топологию между BGP соседями.
5. Проверить связность между PC1, PC2, PC3.
### Реализовать схему
![BGP](BGP.PNG)

### Таблица адресов
| Device        | Interface | IP Address   | Mask |
| ------------- |:----------| :------------| :----|
| Spine01       | Ethernet1 | 172.16.0.11  | /31  |
|               | Ethernet2 | 172.16.0.15  | /31  |
|               | Ethernet3 | 172.16.0.19  | /31  |
| Spine02       | Ethernet1 | 172.16.0.13  | /31  |
|               | Ethernet2 | 172.16.0.17  | /31  |
|               | Ethernet3 | 172.16.0.21  | /31  |
| Leaf01        | Ethernet1 | 172.16.0.10  | /31  |
|               | Ethernet2 | 172.16.0.12  | /31  |
|               | Ethernet3 | 10.1.10.1    | /24  |
| Leaf02        | Ethernet1 | 172.16.0.14  | /31  |
|               | Ethernet2 | 172.16.0.16  | /31  |
|               | Ethernet3 | 10.2.20.1    | /24  |
| Leaf03        | Ethernet1 | 172.16.0.18  | /31  |
|               | Ethernet2 | 172.16.0.20  | /31  |
|               | Ethernet3 | 10.3.30.1    | /24  |
| PC1           | Ethernet0 | 10.1.10.2    | /24  |
| PC2           | Ethernet0 | 10.2.20.2    | /24  |
| PC3           | Ethernet0 | 10.3.30.2    | /24  |
| PC4           | Ethernet0 | 10.3.30.3    | /24  |

### Конфигурация устройств
#### Spine01
```
feature bgp
interface Ethernet1/1
  description Leaf01 e1/1
  no switchport
  ip address 172.16.0.11/31
  no shutdown
interface Ethernet1/2
  description Leaf02 e1/1
  no switchport
  ip address 172.16.0.15/31
  no shutdown
interface Ethernet1/3
  description Leaf03 e1/1
  no switchport
  ip address 172.16.0.19/31
  no shutdown
router bgp 65004
  router-id 10.10.10.4
  bestpath as-path multipath-relax
  log-neighbor-changes
  template peer Leafs
    password 3 4258f34a25410d21
    address-family ipv4 unicast
  neighbor 172.16.0.10
    inherit peer Leafs
    remote-as 65001
    description Leaf01
  neighbor 172.16.0.14
    inherit peer Leafs
    remote-as 65002
    description Leaf02
  neighbor 172.16.0.18
    inherit peer Leafs
    remote-as 65003
    description Leaf03
``` 
#### Spine02
```
feature bgp
interface Ethernet1/1
  description Leaf01 e1/2
  no switchport
  ip address 172.16.0.13/31
  no shutdown
interface Ethernet1/2
  description Leaf02 e1/2
  no switchport
  ip address 172.16.0.17/31
  no shutdown
interface Ethernet1/3
  description Leaf03 e1/2
  no switchport
  ip address 172.16.0.21/31
  no shutdown
router bgp 65005
  router-id 10.10.10.4
  bestpath as-path multipath-relax
  log-neighbor-changes
  template peer Leafs
    password 3 4258f34a25410d21
    address-family ipv4 unicast
  neighbor 172.16.0.12
    inherit peer Leafs
    remote-as 65001
    description Leaf01
  neighbor 172.16.0.16
    inherit peer Leafs
    remote-as 65002
    description Leaf02
  neighbor 172.16.0.20
    inherit peer Leafs
    remote-as 65003
    description Leaf03
```
#### Leaf01
```
feature bgp
interface Ethernet1/1
  description Spine01 e1/1
  no switchport
  ip address 172.16.0.10/31
  no shutdown
interface Ethernet1/2
  description Spine02 e1/1
  no switchport
  ip address 172.16.0.12/31
  no shutdown
interface Ethernet1/3
  no switchport
  ip address 10.1.10.1/24
  no shutdown
ip prefix-list PL_Local_Leaf01 seq 5 permit 10.1.10.0/24
route-map FROM_Leaf01 permit 10
  match ip address prefix-list PL_Local_Leaf01
router bgp 65001
  router-id 10.10.10.1
  bestpath as-path multipath-relax
  log-neighbor-changes
  template peer Spines
    password 3 4258f34a25410d21
    address-family ipv4 unicast
      route-map FROM_Leaf01 out
  neighbor 172.16.0.11
    inherit peer Spines
    remote-as 65004
    description Spine01
  neighbor 172.16.0.13
    inherit peer Spines
    remote-as 65005
    description Spine02
```
#### Leaf02
```
feature bgp
interface Ethernet1/1
  description Spine01 e1/2
  no switchport
  ip address 172.16.0.14/31
  no shutdown
interface Ethernet1/2
  description Spine01 e1/2
  no switchport
  ip address 172.16.0.16/31
  no shutdown
interface Ethernet1/3
  no switchport
  ip address 10.2.20.1/24
  no shutdown
ip prefix-list PL_Local_Leaf02 seq 5 permit 10.2.20.0/24
route-map FROM_Leaf02 permit 10
  match ip address prefix-list PL_Local_Leaf02
router bgp 65002
  router-id 10.10.10.2
  bestpath as-path multipath-relax
  log-neighbor-changes
  template peer Spines
    password 3 4258f34a25410d21
    address-family ipv4 unicast
      route-map FROM_Leaf02 out
  neighbor 172.16.0.15
    inherit peer Spines
    remote-as 65004
    description Spine01
  neighbor 172.16.0.17
    inherit peer Spines
    remote-as 65005
    description Spine02
```
#### Leaf03
```
feature bgp
interface Ethernet1/1
  description Spine01 e1/3
  no switchport
  ip address 172.16.0.18/31
  no shutdown
interface Ethernet1/2
  description Spine02 e1/3
  no switchport
  ip address 172.16.0.20/31
  no shutdown
interface Ethernet1/3
  no switchport
  ip address 10.3.30.1/24
  no shutdown
ip prefix-list PL_Local_Leaf03 seq 5 permit 10.3.30.0/24
route-map FROM_Leaf03 permit 10
  match ip address prefix-list PL_Local_Leaf03
router bgp 65003
  router-id 10.10.10.3
  bestpath as-path multipath-relax
  log-neighbor-changes
  template peer Spines
    password 3 4258f34a25410d21
    address-family ipv4 unicast
      route-map FROM_Leaf03 out
  neighbor 172.16.0.19
    inherit peer Spines
    remote-as 65004
    description Spine01
  neighbor 172.16.0.21
    inherit peer Spines
    remote-as 65005
    description Spine02
``` 
#### PC1
```
IP/MASK: 10.1.10.2/24  
GATEWAY: 10.1.10.1
```  
#### PC2
```
IP/MASK: 10.2.20.2/24  
GATEWAY: 10.2.20.1
```  
#### PC3
```
IP/MASK: 10.3.30.2/24  
GATEWAY: 10.3.30.1  
```
#### PC4
```
IP/MASK: 10.3.30.3/24  
GATEWAY: 10.3.30.1 
``` 