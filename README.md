# EXEMPLO DE CONFIGURAÇÃO DE MPLS L2VC E L3VPN
ESSE CASE É DE UM EXEMPLO SIMILAR A UM IMPLEMENTADO EM ROTEADORES FISICOS.

---
### - CENÁRIO PROPOSTO
![diagrama](https://github.com/Lucasolizz/MPLS/assets/90730521/ad296f03-a2dd-4022-ba4c-2df9b0068382)

---
### - ENDEREÇAMENTO
![Screenshot from 2024-04-20 15-54-51](https://github.com/Lucasolizz/MPLS/assets/90730521/7c2d2442-9d6f-4362-bc77-7c845b9becbb)

---

- CONFIGURAÇÃO DO PE 1

```
sytem-view

sysname PE1

interface loopback 0
ip address 10.210.0.1 32
commit
quit

mpls
mpls ldp
quit
mpls te
mpls ldp lsr-id 10.210.0.1
commit
mpls ldp remote-peer PE2
remote-ip 10.210.0.2
commit
quit
mpls ldp remote-peer PE3
remote-ip 10.210.0.3
commit
quit



ospf 1 router-id 10.210.0.1
opaque-capability enable
area 0.0.0.0
network 10.0.0.0 0.255.255.255
mpls te
quit
quit
commit

interface 25g0/5/20
ip address  10.200.0.1 30
descripion CONNECTION-INTERFACE PE_2
ospf enable 1 area 0.0.0.0
ospf network-type p2p
ospf cost 10
mtu 9216
mpls
mpls ldp
mpls te
mpls mtu 9200
commit
quit

interface 25g0/5/22
ip address  10.200.0.5 30
description CONNECTION-ITERFACE_PE3
ospf enable 1 area 0.0.0.0
ospf network-type p2p
ospf cost 10
mtu 9216
mpls
mpls ldp
mpls te
mpls mtu 9200
commit
quit
commit
quit

ip vpn-instance vpna
ipv4-family
router-distinguisher 111:1
vpn-target 100:1 both
commit
quit

ip vpn-instance vpnb
ipv4-family
router-distinguisher 111:2
vpn-target 200:1 both
commit
quit

route-policy PERMIT-ALL permit node 999
quit
commit

bgp 65055
router-id 10.210.0.1
peer 10.210.0.2 as-number 65055
peer 10.210.0.2 route-policy PERMIT-ALL import
peer 10.210.0.2 route-policy PERMIT-ALL export
peer 10.210.0.2 connect-interface loopback 0
peer 10.210.0.2 description PE-2
peer 10.210.0.3 as-number 65055
peer 10.210.0.3 route-policy PERMIT-ALL import
peer 10.210.0.3 route-policy PERMIT-ALL export
peer 10.210.0.3 connect-interface loopback 0
peer 10.210.0.3 description PE-3

ipv4-family vpnv4
peer 10.210.0.2 enable
peer 10.210.0.3 enable
peer 10.210.0.2 route-policy PERMIT-ALL import
peer 10.210.0.2 route-policy PERMIT-ALL export
peer 10.210.0.3 route-policy PERMIT-ALL import
peer 10.210.0.3 route-policy PERMIT-ALL export
quit

ipv4-family vpn-instance vpna
import-route direct
import-route static
peer 172.30.10.2 as-number 65000
peer 172.30.10.2 description CE-VPNA
peer 172.30.10.2 route-policy PERMIT-ALL import
peer 172.30.10.2 route-policy PERMIT-ALL export
quit
commit

ipv4-family vpn-instance vpnb
import-route direct
peer 172.30.10.6 as-number 65000
peer 172.30.10.6 description CE-VPNB 
peer 172.30.10.6 route-policy PERMIT-ALL import
peer 172.30.10.6 route-policy PERMIT-ALL export

interface g0/0/0
description CE-VPNA
ip binding vpn-instance vpna
ip address 172.30.10.1 30
commit
quit

interface g0/0/1
description CE-VPNB
ip binding vpn-instance vpnb
ip address 172.30.10.5 30
commit
quit

interface g0/0/2.3030
vlan-type dot1q 3030
description TRANSPORTE-MPLS
mpls l2vc 10.210.0.2 3030 control-word
commit
quit

run save
y

```

- CONFIGURAÇÃO DO PE 2


```
system-view

sysname PE2

interface loopback 0
ip address 10.210.0.2 32
commit
quit

mpls
mpls ldp
quit
mpls te
mpls ldp lsr-id 10.210.0.2
commit
mpls ldp remote-peer PE1
remote-ip 10.210.0.1
commit
quit
mpls ldp remote-peer PE3
remote-ip 10.210.0.3
commit
quit



ospf 1 router-id 10.210.0.2
opaque-capability enable
area 0.0.0.0
network 10.0.0.0 0.255.255.255
mpls te
quit
quit
commit

interface 25g0/5/20
ip address  10.200.0.2 30
descripion CONNECTION-INTERFACE PE_1
ospf enable 1 area 0.0.0.0
ospf network-type p2p
ospf cost 10
mtu 9216
mpls
mpls ldp
mpls te
mpls mtu 9200
commit
quit

interface 25g0/5/24
ip address  10.200.0.9 30
description CONNECTION-ITERFACE_PE3
ospf enable 1 area 0.0.0.0
ospf network-type p2p
ospf cost 10
mtu 9216
mpls
mpls ldp
mpls te
mpls mtu 9200
commit
quit


ip vpn-instance vpna
ipv4-family
router-distinguisher 222:1
vpn-target 100:1 both
commit
quit
quit

route-policy PERMIT-ALL permit node 999
quit
commit

bgp 65055
router-id 10.210.0.2
peer 10.210.0.1 as-number 65055
peer 10.210.0.1 route-policy PERMIT-ALL import
peer 10.210.0.1 route-policy PERMIT-ALL export
peer 10.210.0.1 connect-interface loopback 0
peer 10.210.0.1 description PE-1
peer 10.210.0.3 as-number 65055
peer 10.210.0.3 route-policy PERMIT-ALL import
peer 10.210.0.3 route-policy PERMIT-ALL export
peer 10.210.0.3 connect-interface loopback 0
peer 10.210.0.3 description PE-3

ipv4-family vpnv4
peer 10.210.0.1 enable
peer 10.210.0.3 enable
peer 10.210.0.1 route-policy PERMIT-ALL import
peer 10.210.0.1 route-policy PERMIT-ALL export
peer 10.210.0.3 route-policy PERMIT-ALL import
peer 10.210.0.3 route-policy PERMIT-ALL export
quit

ipv4-family vpn-instance vpna
import-route direct
peer 172.30.20.2 as-number 65000
peer 172.30.20.2 description CE-VPNA
peer 172.30.20.2 route-policy PERMIT-ALL import
peer 172.30.20.2 route-policy PERMIT-ALL export
quit
commit
quit

interface g0/0/0
description CE-VPNA
ip binding vpn-instance vpna
ip address 172.30.20.1 30
commit
quit

interface g0/0/1.3030
vlan-type dot1q 3030
description TRANSPORTE-MPLS
mpls l2vc 10.210.0.1 3030 control-word
commit
quit

run save
y

```
- CONFIGURAÇÃO DO PE 3

```
system-view

sysname PE3

interface loopback 0
ip address 10.210.0.3 32
commit
quit

mpls
mpls ldp
quit
mpls te
mpls ldp lsr-id 10.210.0.3
commit
mpls ldp remote-peer PE2
remote-ip 10.210.0.2
commit
quit
mpls ldp remote-peer PE1
remote-ip 10.210.0.1
commit
quit



ospf 1 router-id 10.210.0.3
opaque-capability enable
area 0.0.0.0
network 10.0.0.0 0.255.255.255
mpls te
quit
quit
commit

interface 25g0/5/22
ip address  10.200.0.6 30
descripion CONNECTION-INTERFACE PE_1
ospf enable 1 area 0.0.0.0
ospf network-type p2p
ospf cost 10
mtu 9216
mpls
mpls ldp
mpls te
mpls mtu 9200
commit
quit

interface 25g0/5/24
ip address  10.200.0.10 30
description CONNECTION-ITERFACE_PE2
ospf enable 1 area 0.0.0.0
ospf network-type p2p
ospf cost 10
mtu 9216
mpls
mpls ldp
mpls te
mpls mtu 9200
commit
quit
commit
quit

ip vpn-instance vpna
ipv4-family
router-distinguisher 333:1
vpn-target 100:1 both
commit
quit

ip vpn-instance vpnb
ipv4-family
router-distinguisher 333:2
vpn-target 200:1 both
commit
quit

route-policy PERMIT-ALL permit node 999
quit
commit

bgp 65055
router-id 10.210.0.3
peer 10.210.0.2 as-number 65055
peer 10.210.0.2 route-policy PERMIT-ALL import
peer 10.210.0.2 route-policy PERMIT-ALL export
peer 10.210.0.2 connect-interface loopback 0
peer 10.210.0.2 description PE-2
peer 10.210.0.1 as-number 65055
peer 10.210.0.1 route-policy PERMIT-ALL import
peer 10.210.0.1 route-policy PERMIT-ALL export
peer 10.210.0.1 connect-interface loopback 0
peer 10.210.0.1 description PE-1

ipv4-family vpnv4
peer 10.210.0.2 enable
peer 10.210.0.1 enable
peer 10.210.0.2 route-policy PERMIT-ALL import
peer 10.210.0.2 route-policy PERMIT-ALL export
peer 10.210.0.1 route-policy PERMIT-ALL import
peer 10.210.0.1 route-policy PERMIT-ALL export
quit

ipv4-family vpn-instance vpna
import-route direct
import-route static
peer 172.30.30.6 as-number 65055
peer 172.30.30.6 description CE-VPNA
peer 172.30.30.6 route-policy PERMIT-ALL import
peer 172.30.30.6 route-policy PERMIT-ALL export
quit

ipv4-family vpn-instance vpnb
import-route direct
peer 172.30.30.2 as-number 65000
peer 172.30.30.2 description CE-VPNB 
peer 172.30.30.2 route-policy PERMIT-ALL import
peer 172.30.30.2 route-policy PERMIT-ALL export
commit
quit


interface g0/0/0
description CE-VPNB
ip binding vpn-instance vpnb
ip address 172.30.30.1 30
commit
quit

interface g0/0/1
description CE-VPNA
ip binding vpn-instance vpna
ip address 172.30.30.5 30
commit
quit

run save
y

```
---
- EXEMPLO DE CONFIGURAÇÃO DO CE-L3 

```
system-view

sysname CE

interface g0/0/0
description WAN-MPLS
ip address 172.30.10.2 30
commit
quit

interface g0/0/1
description LAN
ip address 192.168.1.1 24
quit
commit

route-policy PERMIT-ALL permit node 999
quit
commit

bgp 65000
peer 172.30.10.1 as-number 65055
peer 172.30.10.1 description GATEWAY
peer 172.30.10.1 route-policy PERMIT-ALL import
peer 172.30.10.1 route-policy PERMIT-ALL export

ipv4-family unicast
network 192.168.1.0 24
quit
quit
commit

```
---

- EXEMPLO DE CONFIGURAÇÃO DE CE-L2VC (ROUTER)

> CE 1



```
system-view

sysname CE1

interface g0/0/0.3030
vlan-type dot1q 3030
description Point-to-Point
ip address 192.168.30.1 30

interface g0/0/1.100
vlan-type dot1q 100
description REDE 100
ip address 192.168.100.1 24

interface g0/0/1.200
vlan-type dot1q 200
description REDE 200
ip address 192.168.200.1 24
commit
quit

ip route-static 192.168.110.0 24 192.168.30.2 description ROUTE-TO-NET-110
ip route-static 192.168.210.0 24 192.168.30.2 description ROUTE-TO-NET-210

commit
run save
y
```

> CE 2

```
system-view

sysname CE2

interface g0/0/0.3030
vlan-type dot1q 3030
description Pont-to-Point
ip address 192.168.30.2 30

interface g0/0/1.110
vlan-type dot1q 110
description REDE 110
ip address 192.168.110.1 24

interface g0/0/1.210
vlan-type dot1q 210
description REDE 210
ip address 192.168.210.1 24
commit
quit

ip route-static 192.168.100.0 24 192.168.30.1 description ROUTE-TO-NET 100
ip route-static 192.168.200.0 24 192.168.30.1 description ROUTE-TO-NET 200

commit
run save
y

```
---
- EXEMPLO DE CONFIGURAÇÃO DE CE-L2VC (SWITCH)


> CE Switch

```
system-view

sysname CE-Switch

vlan 3030

interface g0/0/0
portswitch
description Point-to-Point
port link-type trunk
port trunk allow-pass vlan 3030
undo port trunk allow-pass vlan 1
quit

interface g0/0/1
portswitch
description Point-to-Point_in-Access
port link-type access
port default-vlan 3030
quit
save
y

```
> CRIAR ROTA ESTÁTICA PARA UMA VPN-INSTANCE

```
ip route-static vpn-instance [nome da vpn] [CIDR] [gateway]
```

COMANDOS DE VISUALIZAÇÃO
========================

> PING COM VPN-INSTANCE

```
ping -vpn-instance [nome da vpn] [ip de destino]
```
 > EXIBIR TABELA DE ROTEAMENTO DE UMA VPN-INSTANCE
```
display ip routing-table vpn-instance [nome da vpn]
```

> VERIFICAR ADJASCÊNCIA OSPF


```
display ospf peer brief
```

> VERIFICAR ADJASCÊNCIA LDP


```
display mpls ldp session
```
> VERIFICAR SESSÃO BGP


```
display bgp peer
```
> VERIFICAR SESSÃO BGP DAS VPNS

será mostrado tanto a sessão do vpnv4, entre os PEs, como com as vpn-instances

```
display bgp vpnv4 all peer
```

> VERFICAR TUNNEL L3-VPN DE UMA VPN INSTANCE


```
display ip vpn-instance [nome da vpn] tunnel-info
```

> VERIFICAR TUNNEL L2VC


```
display mpls l2vc [id do virtual circuit]
```
---
