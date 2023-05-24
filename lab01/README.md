### Схема CLOS
### Цели
1. Собрать топологию CLOS.
2. Распределить адресное пространство для Underlay сети.

### Реализовать схему
![Топология](https://github.com/nsa4646/otus/blob/main/lab01/raw/images/Топология.png "Топология")

### Таблица адресов
| Device        | Interface | IP Address   | Mask |
| ------------- |:----------| :------------| :----|
| Spine01       | Ethernet1 | 169.254.0.2  | /30  |
|               | Ethernet2 | 169.254.0.10 | /30  |
|               | Ethernet3 | 169.254.0.18 | /30  |
| Spine02       | Ethernet1 | 169.254.0.6  | /30  |
|               | Ethernet2 | 169.254.0.14 | /30  |
|               | Ethernet3 | 169.254.0.22 | /30  |
| Leaf01        | Ethernet1 | 169.254.0.1  | /30  |
|               | Ethernet2 | 169.254.0.5  | /30  |
|               | vlan10    | 10.1.10.1    | /24  |
| Leaf02        | Ethernet1 | 169.254.0.9  | /30  |
|               | Ethernet2 | 169.254.0.13 | /30  |
|               | vlan20    | 10.2.20.1    | /24  |
| Leaf03        | Ethernet1 | 169.254.0.17 | /30  |
|               | Ethernet2 | 169.254.0.21 | /30  |
|               | vlan30    | 10.3.30.1    | /24  |
| PC1           | Ethernet0 | 10.1.10.2    | /24  |
| PC2           | Ethernet0 | 10.2.20.2    | /24  |
| PC3           | Ethernet0 | 10.3.30.2    | /24  |

### Конфигурация устройств
#### Spine01
interface Ethernet1  
   no switchport  
   ip address 169.254.0.2/30  
interface Ethernet2  
   no switchport  
   ip address 169.254.0.10/30  
interface Ethernet3  
   no switchport  
   ip address 169.254.0.18/30  
interface Loopback1  
   ip address 10.0.1.1/32  
#### Spine02
interface Ethernet1  
   no switchport  
   ip address 169.254.0.6/30  
interface Ethernet2  
   no switchport  
   ip address 169.254.0.14/30  
interface Ethernet3  
   no switchport  
   ip address 169.254.0.22/30  
interface Loopback1  
   ip address 10.0.2.1/32 
#### Leaf01
vlan 10  
interface Ethernet1  
   no switchport  
   ip address 169.254.0.1/30  
interface Ethernet2  
   no switchport  
   ip address 169.254.0.5/30  
interface Ethernet3  
   switchport access vlan 10  
   spanning-tree portfast  
interface Loopback1  
   ip address 10.1.1.1/32  
interface Vlan10  
   ip address 10.1.10.1/24  
#### Leaf02
vlan 20  
interface Ethernet1  
   no switchport  
   ip address 169.254.0.9/30  
interface Ethernet2  
   no switchport  
   ip address 169.254.0.13/30  
interface Ethernet3  
   switchport access vlan 20  
   spanning-tree portfast  
interface Loopback1  
   ip address 10.1.2.1/32 
interface Vlan20  
   ip address 10.2.20.1/24  
#### Leaf03
vlan 30
!
interface Ethernet1  
   no switchport  
   ip address 169.254.0.17/30  
interface Ethernet2  
   no switchport  
   ip address 169.254.0.21/30  
interface Ethernet3  
   switchport access vlan 30  
   spanning-tree portfast  
interface Ethernet4  
   switchport access vlan 30  
   spanning-tree portfast  
interface Vlan30  
   ip address 10.3.30.1/24  