# mpls-l2vc-l3vpn
EXEMPLO DE CONFIGURAÇÃO DE MPLS L2VC E L3VPN

NESSE CASE DE UMA REDE SIMILAR A UMA IMPLEMENTADA


OS ROTEADORES USADOS NO EXEMPLO SÃO INSPIRADOS NO NE8000 M4 DA HUAWEI


Para todos os exemplos o asn do PE será 65055 e o asn do CE será 65000

O diagrama de rede esta no arquivo https://github.com/Lucasolizz/mpls-l2vc-l3vpn/blob/main/diagrama.png



CONFIGURAÇÃO DO PE 1
=====================

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

CONFIGURAÇÃO DO PE 2
====================

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
CONFIGURAÇÃO DO PE 3
====================

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

CONFIGURAÇÃO DO CE
