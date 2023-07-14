### VxLAN. EVPN L3 на базе Arista
### Цели
1. Распределить адресное пространство на Underlay сети.
2. Настроить IP связанность между всеми устройствами Arista.
3. Настроить BGP peering между Leaf и Spine в AF l3vpn evpn на устройствах Arista (Vlan-Based).
4. Настроить каждого клиента в своем VNI.
5. Проверить связность между PC1, PC2, PC3, PC4, PC5.
6. Настроить симметричную настройку IRB для связности клиентов через разные vtep.
### Реализовать схему
![VxLAN. EVPN L3](Topology.PNG)

### Таблица адресов
| Device        | Interface | IP Address   | Mask |
| ------------- |:----------| :------------| :----|
| Spine01       | Ethernet1 | 172.16.0.11  | /31  |
|               | Ethernet2 | 172.16.0.15  | /31  |
| Leaf01        | Ethernet1 | 172.16.0.10  | /31  |
|               | Ethernet3 | vlan10       |      |
|               | Ethernet4 | vlan20       |      |
|               | Ethernet5 | vlan30       |      |
|               | Loopback0 | 10.10.10.1   | /32  |
|               | Loopback1 | 10.10.11.1   | /32  |
| Leaf02        | Ethernet1 | 172.16.0.14  | /31  |
|               | Ethernet3 | vlan10       |      |
|               | Ethernet4 | vlan20       |      |
|               | Loopback0 | 10.10.10.2   | /32  |
|               | Loopback1 | 10.10.11.2   | /32  |
| PC1 PROD_WEB_1| Ethernet0 | 10.1.10.2    | /24  |
| PC2 PROD_DB_1 | Ethernet0 | 10.2.20.2    | /24  |
| PC3 PROD_LB_1 | Ethernet0 | 10.3.30.2    | /24  |
| PC4 PROD_WEB_2| Ethernet0 | 10.1.10.3    | /24  |
| PC5 PROD_DB_2 | Ethernet0 | 10.2.20.3    | /24  |

### Конфигурация устройств
#### Spine01
```
ip routing
mpls ip
service routing protocols model multi-agent
interface Ethernet1
   description Leaf01
   no switchport
   ip address 172.16.0.11/31
interface Ethernet2
   description Leaf02
   no switchport
   ip address 172.16.0.15/31
interface Loopback0
   ip address 10.10.10.4/32
ip prefix-list Spine01_Local seq 5 permit 10.10.10.4/32
route-map FROM_Spine01 permit 10
   match ip address prefix-list Spine01_Local
router bgp 65001
   router-id 10.10.10.4
   bgp listen range 10.10.10.0/24 peer-group LEAF_OVERLAY remote-as 65001
   bgp listen range 172.16.0.0/27 peer-group LEAF_UNDERLAY remote-as 65001
   neighbor LEAF_OVERLAY peer group
   neighbor LEAF_OVERLAY update-source Loopback0
   neighbor LEAF_OVERLAY route-reflector-client
   neighbor LEAF_OVERLAY send-community
   neighbor LEAF_UNDERLAY peer group
   neighbor LEAF_UNDERLAY next-hop-self
   neighbor LEAF_UNDERLAY route-reflector-client
   redistribute connected route-map FROM_Spine01
   address-family evpn
      neighbor LEAF_OVERLAY activate
   address-family ipv4
      no neighbor LEAF_OVERLAY activate
```
#### Leaf01
```
ip routing
mpls ip
service routing protocols model multi-agent
vlan 10,20,30
vrf instance PROD
interface Ethernet1
   description Spine01
   no switchport
   ip address 172.16.0.10/31
interface Ethernet3
   description PROD_WEB_1
   switchport access vlan 10
   spanning-tree portfast
interface Ethernet4
   description PROD_DB_1
   switchport access vlan 20
   spanning-tree portfast
interface Ethernet5
   description PROD_LB_1
   switchport access vlan 30
   spanning-tree portfast
interface Loopback0
   ip address 10.10.10.1/32
interface Loopback1
   ip address 10.10.11.1/32
interface Vlan10
   vrf PROD
   ip address virtual 10.1.10.1/24
interface Vlan20
   vrf PROD
   ip address virtual 10.2.20.1/24
interface Vlan30
   vrf PROD
   ip address virtual 10.3.30.1/24
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vlan 30 vni 10030
   vxlan vrf PROD vni 65001
ip virtual-router mac-address 00:00:22:22:33:33
ip routing
ip routing vrf PROD
ip prefix-list Leaf01_Local
   seq 5 permit 10.10.10.1/32
   seq 10 permit 10.10.11.1/32
route-map FROM_Leaf01 permit 10
   match ip address prefix-list Leaf01_Local
router bgp 65001
   router-id 10.10.10.1
   neighbor SPINE_OVERLAY peer group
   neighbor SPINE_OVERLAY remote-as 65001
   neighbor SPINE_OVERLAY update-source Loopback0
   neighbor SPINE_OVERLAY send-community
   neighbor Spine_UNDERLAY peer group
   neighbor Spine_UNDERLAY remote-as 65001
   neighbor 10.10.10.4 peer group SPINE_OVERLAY
   neighbor 172.16.0.11 peer group Spine_UNDERLAY
   redistribute connected route-map FROM_Leaf01
   vlan 10
      rd 10.10.10.1:10010
      route-target both 1:10010
      redistribute learned
   vlan 20
      rd 10.10.10.1:10020
      route-target both 1:10020
      redistribute learned
   vlan 30
      rd 10.10.10.1:10030
      route-target both 1:10030
      redistribute learned
   address-family evpn
      neighbor SPINE_OVERLAY activate
   address-family ipv4
      no neighbor SPINE_OVERLAY activate
   vrf PROD
      rd 10.10.10.1:65001
      route-target import evpn 1:65001
      route-target export evpn 1:65001
```
#### Leaf02
```
ip routing
mpls ip
service routing protocols model multi-agent
vlan 10,20
vrf instance PROD
interface Ethernet1
   description Spine01
   no switchport
   ip address 172.16.0.14/31
interface Ethernet3
   description PROD_WEB_2
   switchport access vlan 10
   spanning-tree portfast
interface Ethernet4
   description PROD_DB_2
   switchport access vlan 20
   spanning-tree portfast
interface Loopback0
   ip address 10.10.10.2/32
interface Loopback1
   ip address 10.10.11.2/32
interface Vlan10
   vrf PROD
   ip address virtual 10.1.10.1/24
interface Vlan20
   vrf PROD
   ip address virtual 10.2.20.1/24
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 10010
   vxlan vlan 20 vni 10020
   vxlan vrf PROD vni 65001
ip virtual-router mac-address 00:00:22:22:33:33
ip routing
ip routing vrf PROD
ip prefix-list Leaf02_Local seq 5 permit 10.10.10.2/32
ip prefix-list Leaf02_Local seq 10 permit 10.10.11.2/32
route-map FROM_Leaf02 permit 10
   match ip address prefix-list Leaf02_Local
router bgp 65001
   router-id 10.10.10.2
   neighbor SPINE_OVERLAY peer group
   neighbor SPINE_OVERLAY remote-as 65001
   neighbor SPINE_OVERLAY update-source Loopback0
   neighbor SPINE_OVERLAY send-community
   neighbor Spine_UNDERLAY peer group
   neighbor Spine_UNDERLAY remote-as 65001
   neighbor 10.10.10.4 peer group SPINE_OVERLAY
   neighbor 172.16.0.15 peer group Spine_UNDERLAY
   redistribute connected route-map FROM_Leaf02
   vlan 10
      rd 10.10.10.2:10010
      route-target both 1:10010
      redistribute learned
   vlan 20
      rd 10.10.10.2:10020
      route-target both 1:10020
      redistribute learned
   address-family evpn
      neighbor SPINE_OVERLAY activate
   address-family ipv4
      no neighbor SPINE_OVERLAY activate
   vrf PROD
      rd 10.10.10.2:65001
      route-target import evpn 1:65001
      route-target export evpn 1:65001
```
#### PC1 PROD_WEB_1
```
IP/MASK: 10.1.10.2/24  
GATEWAY: 10.1.10.1
```  
#### PC2 PROD_DB_1
```
IP/MASK: 10.2.20.2/24  
GATEWAY: 10.2.20.1
```  
#### PC3 PROD_LB_1
```
IP/MASK: 10.3.30.2/24  
GATEWAY: 10.3.30.1  
```
#### PC4 PROD_WEB_2
```
IP/MASK: 10.1.10.3/24  
GATEWAY: 10.1.10.1  
```
#### PC5 PROD_DB_2
```
IP/MASK: 10.2.20.3/24  
GATEWAY: 10.2.20.1 
``` 

### Вывод BGP, evpn соседства и топологии между устрйоствами
#### Spine01
```
Spine01#sh ip bgp summary
  Neighbor    V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  172.16.0.10 4 65001            144       147    0    0 01:59:40 Estab   2      2
  172.16.0.14 4 65001             16        17    0    0 00:09:41 Estab   2      2
```
```
Spine01#sh bgp evpn summary
  Neighbor   V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.10.10.1 4 65001            148       159    0    0 01:48:17 Estab   5      5
  10.10.10.2 4 65001             21        21    0    0 00:10:09 Estab   4      4
```
```
Spine01#sh bgp evpn
          Network                Next Hop              Metric  LocPref Weight  Path
 * >     RD: 10.10.10.2:10010 mac-ip 0050.7966.6807
                                 10.10.11.2            -       100     0       i
 * >     RD: 10.10.10.2:10010 mac-ip 0050.7966.6807 10.1.10.3
                                 10.10.11.2            -       100     0       i
 * >     RD: 10.10.10.1:10030 mac-ip 0050.7966.6808
                                 10.10.11.1            -       100     0       i
 * >     RD: 10.10.10.1:10030 mac-ip 0050.7966.6808 10.3.30.2
                                 10.10.11.1            -       100     0       i
 * >     RD: 10.10.10.1:10010 imet 10.10.11.1
                                 10.10.11.1            -       100     0       i
 * >     RD: 10.10.10.1:10020 imet 10.10.11.1
                                 10.10.11.1            -       100     0       i
 * >     RD: 10.10.10.1:10030 imet 10.10.11.1
                                 10.10.11.1            -       100     0       i
 * >     RD: 10.10.10.2:10010 imet 10.10.11.2
                                 10.10.11.2            -       100     0       i
 * >     RD: 10.10.10.2:10020 imet 10.10.11.2
                                 10.10.11.2            -       100     0       i
```
#### Leaf01
```
Leaf01#sh bgp evpn summary
  Neighbor   V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.10.10.4 4 65001            167       163    0    0 01:49:49 Estab   2      2
```
```
Leaf01#sh bgp evpn
          Network                Next Hop              Metric  LocPref Weight  Path
 * >     RD: 10.10.10.2:10010 imet 10.10.11.2
                                 10.10.11.2            -       100     0       i Or-ID: 10.10.10.2 C-LST: 10.10.10.4
 * >     RD: 10.10.10.2:10020 imet 10.10.11.2
                                 10.10.11.2            -       100     0       i Or-ID: 10.10.10.2 C-LST: 10.10.10.4
 * >     RD: 10.10.10.1:10010 imet 10.10.11.1
                                 -                     -       -       0       i
 * >     RD: 10.10.10.1:10020 imet 10.10.11.1
                                 -                     -       -       0       i
 * >     RD: 10.10.10.1:10030 imet 10.10.11.1
                                 -                     -       -       0       i
```
```
Leaf01#sh ip route vrf PROD
 B I      10.1.10.3/32 [200/0] via VTEP 10.10.11.2 VNI 65001 router-mac 50:0b:00:d7:ac:d7 local-interface Vxlan1
 C        10.1.10.0/24 is directly connected, Vlan10
 B I      10.2.20.3/32 [200/0] via VTEP 10.10.11.2 VNI 65001 router-mac 50:0b:00:d7:ac:d7 local-interface Vxlan1
 C        10.2.20.0/24 is directly connected, Vlan20
 C        10.3.30.0/24 is directly connected, Vlan30
```
#### Leaf02
```
Leaf02#sh bgp evpn summary
  Neighbor   V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.10.10.4 4 65001             31        31    0    0 00:15:02 Estab   9      9
```
```
Leaf02#sh bgp evpn
          Network                Next Hop              Metric  LocPref Weight  Path
 * >     RD: 10.10.10.1:10020 mac-ip 0050.7966.6802
                                 10.10.11.1            -       100     0       i Or-ID: 10.10.10.1 C-LST: 10.10.10.4
 * >     RD: 10.10.10.1:10020 mac-ip 0050.7966.6802 10.2.20.2
                                 10.10.11.1            -       100     0       i Or-ID: 10.10.10.1 C-LST: 10.10.10.4
 * >     RD: 10.10.10.2:10020 mac-ip 0050.7966.6805
                                 -                     -       -       0       i
 * >     RD: 10.10.10.2:10020 mac-ip 0050.7966.6805 10.2.20.3
                                 -                     -       -       0       i
 * >     RD: 10.10.10.1:10010 mac-ip 0050.7966.6806
                                 10.10.11.1            -       100     0       i Or-ID: 10.10.10.1 C-LST: 10.10.10.4
 * >     RD: 10.10.10.1:10010 mac-ip 0050.7966.6806 10.1.10.2
                                 10.10.11.1            -       100     0       i Or-ID: 10.10.10.1 C-LST: 10.10.10.4
 * >     RD: 10.10.10.2:10010 mac-ip 0050.7966.6807
                                 -                     -       -       0       i
 * >     RD: 10.10.10.2:10010 mac-ip 0050.7966.6807 10.1.10.3
                                 -                     -       -       0       i
 * >     RD: 10.10.10.1:10030 mac-ip 0050.7966.6808
                                 10.10.11.1            -       100     0       i Or-ID: 10.10.10.1 C-LST: 10.10.10.4
 * >     RD: 10.10.10.1:10030 mac-ip 0050.7966.6808 10.3.30.2
                                 10.10.11.1            -       100     0       i Or-ID: 10.10.10.1 C-LST: 10.10.10.4
 * >     RD: 10.10.10.1:10010 imet 10.10.11.1
                                 10.10.11.1            -       100     0       i Or-ID: 10.10.10.1 C-LST: 10.10.10.4
 * >     RD: 10.10.10.1:10020 imet 10.10.11.1
                                 10.10.11.1            -       100     0       i Or-ID: 10.10.10.1 C-LST: 10.10.10.4
 * >     RD: 10.10.10.1:10030 imet 10.10.11.1
                                 10.10.11.1            -       100     0       i Or-ID: 10.10.10.1 C-LST: 10.10.10.4
 * >     RD: 10.10.10.2:10010 imet 10.10.11.2
                                 -                     -       -       0       i
 * >     RD: 10.10.10.2:10020 imet 10.10.11.2
                                 -                     -       -       0       i
```
```
Leaf02#sh ip route vrf PROD
 B I      10.1.10.2/32 [200/0] via VTEP 10.10.11.1 VNI 65001 router-mac 50:0b:00:57:9d:f0 local-interface Vxlan1
 C        10.1.10.0/24 is directly connected, Vlan10
 B I      10.2.20.2/32 [200/0] via VTEP 10.10.11.1 VNI 65001 router-mac 50:0b:00:57:9d:f0 local-interface Vxlan1
 C        10.2.20.0/24 is directly connected, Vlan20
 B I      10.3.30.2/32 [200/0] via VTEP 10.10.11.1 VNI 65001 router-mac 50:0b:00:57:9d:f0 local-interface Vxlan1
```