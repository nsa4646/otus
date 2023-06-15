### IS-IS на базе Underlay N9k
### Цели
1. Распределить адресное пространство на Underlay сети.
2. Настроить IP связанность между всеми устройствами NXOS.
3. Настроить IS-IS для на устройствах NXOS.
3. Настроить аутентификацию на интерфесах между is-is соседями.
4. Проверить соседство и топологию между is-is соседями.
5. Проверить связность между PC1, PC2, PC3.
### Реализовать схему


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
feature isis
interface Ethernet1/1
  description Leaf01 e1/1
  no switchport
  ip address 172.16.0.11/31
  no isis hello-padding always
  isis network point-to-point
  isis authentication key-chain otus
  ip router isis 1
  no shutdown
interface Ethernet1/2
  description Leaf02 e1/1
  no switchport
  ip address 172.16.0.15/31
  no isis hello-padding always
  isis network point-to-point
  isis authentication key-chain otus
  ip router isis 1
  no shutdown
interface Ethernet1/3
  description Leaf03 e1/1
  no switchport
  ip address 172.16.0.19/31
  no isis hello-padding always
  isis network point-to-point
  isis authentication key-chain otus
  ip router isis 1
  no shutdown
router isis 1
  net 49.0001.0100.0000.0004.00
  is-type level-1
  log-adjacency-changes
``` 
#### Spine02
```
feature isis
interface Ethernet1/1
  description Leaf01 e1/2
  no switchport
  ip address 172.16.0.13/31
  no isis hello-padding always
  isis network point-to-point
  isis authentication key-chain otus
  ip router isis 1
  no shutdown
interface Ethernet1/2
  description Leaf02 e1/2
  no switchport
  ip address 172.16.0.17/31
  no isis hello-padding always
  isis network point-to-point
  isis authentication key-chain otus
  ip router isis 1
  no shutdown
interface Ethernet1/3
  description Leaf03 e1/2
  no switchport
  ip address 172.16.0.21/31
  no isis hello-padding always
  isis network point-to-point
  isis authentication key-chain otus
  ip router isis 1
  no shutdown
router isis 1
  net 49.0001.0100.0000.0005.00
  is-type level-1
  log-adjacency-changes
```
#### Leaf01
```
feature ospf
interface Ethernet1/1
  description Spine01 e1/1
  no switchport
  ip address 172.16.0.10/31
  no isis hello-padding always
  isis network point-to-point
  isis authentication key-chain otus
  ip router isis 1
  no shutdown
interface Ethernet1/2
  description Spine02 e1/1
  no switchport
  ip address 172.16.0.12/31
  no isis hello-padding always
  isis network point-to-point
  isis authentication key-chain otus
  ip router isis 1
  no shutdown
interface Ethernet1/3
  no switchport
  ip address 10.1.10.1/24
  ip router isis 1
  no shutdown
router isis 1
  net 49.0001.0100.0000.0001.00
  is-type level-1
  log-adjacency-changes
```
#### Leaf02
```
feature ospf
interface Ethernet1/1
  description Spine01 e1/2
  no switchport
  ip address 172.16.0.14/31
  no isis hello-padding always
  isis network point-to-point
  isis authentication key-chain otus
  ip router isis 1
  no shutdown
interface Ethernet1/2
  description Spine01 e1/2
  no switchport
  ip address 172.16.0.16/31
  no isis hello-padding always
  isis network point-to-point
  isis authentication key-chain otus
  ip router isis 1
  no shutdown
interface Ethernet1/3
  no switchport
  ip address 10.2.20.1/24
  ip router isis 1
  no shutdown
router isis 1
  net 49.0001.0100.0000.0002.00
  is-type level-1
  log-adjacency-changes
```
#### Leaf03
```
feature ospf
interface Ethernet1/1
  description Spine01 e1/3
  no switchport
  ip address 172.16.0.18/31
  no isis hello-padding always
  isis network point-to-point
  isis authentication key-chain otus
  ip router isis 1
  no shutdown
interface Ethernet1/2
  description Spine02 e1/3
  no switchport
  ip address 172.16.0.20/31
  no isis hello-padding always
  isis network point-to-point
  isis authentication key-chain otus
  ip router isis 1
  no shutdown
interface Ethernet1/3
  no switchport
  ip address 10.3.30.1/24
  ip router isis 1
  no shutdown
router isis 1
  net 49.0001.0100.0000.0003.00
  is-type level-1
  log-adjacency-changes

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