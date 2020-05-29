# Routeur R1

## Adressage IPv4
```
R1#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         10.191.0.1      YES NVRAM  down                  down
GigabitEthernet0/1         192.168.122.239 YES DHCP   up                    up
GigabitEthernet0/2         10.1.1.1        YES NVRAM  up                    up
GigabitEthernet0/3         10.1.2.1        YES NVRAM  up                    up
GigabitEthernet0/4         unassigned      YES NVRAM  administratively down down
GigabitEthernet0/5         unassigned      YES NVRAM  administratively down down
GigabitEthernet0/6         unassigned      YES NVRAM  administratively down down
GigabitEthernet0/7         11.12.13.108    YES DHCP   up                    up
NVI0                       unassigned      YES unset  up                    up
R1#
```

## Adressage IPv6 Global Unicast site principal (interface G0/1 de R1)
- Adresses des réseaux maîtrisés, routé jusqu'à votre routeur externe : `2001:470:c814:1000::/52`
- L'adresse IPv6 externe de votre routeur : `fe80::cafe:1`
- L'adresse IPv6 de la passerelle vers l'Internet : `fe80::e53:21ff:fe38:5800`

## Adressage IPv6
```
R1#show ipv6 int brie
GigabitEthernet0/0     [down/down]
    FE80::1
GigabitEthernet0/1     [up/up]
    FE80::CAFE:1
GigabitEthernet0/2     [up/up]
    FE80::DD:1
    2001:470:C814:1001::1
GigabitEthernet0/3     [up/up]
    FE80::DD:1
    2001:470:C814:1002::1
GigabitEthernet0/4     [administratively down/down]
    unassigned
GigabitEthernet0/5     [administratively down/down]
    unassigned
GigabitEthernet0/6     [administratively down/down]
    unassigned
GigabitEthernet0/7     [up/up]
    unassigned
NVI0                   [up/up]
    unassigned
R1#
```

# Routeur R4 (Site distant LanR4)

## Adressage IPv4
```
R4#show ip int brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         unassigned      YES NVRAM  administratively down down
GigabitEthernet0/1         192.168.122.162 YES DHCP   up                    up
GigabitEthernet0/2         10.104.1.1      YES NVRAM  up                    up
GigabitEthernet0/3         unassigned      YES NVRAM  down                  down
GigabitEthernet0/4         unassigned      YES NVRAM  administratively down down
GigabitEthernet0/5         unassigned      YES NVRAM  administratively down down
GigabitEthernet0/6         unassigned      YES NVRAM  administratively down down
GigabitEthernet0/7         unassigned      YES NVRAM  administratively down down
NVI0                       unassigned      YES unset  up                    up
R4#
```
## Configuration IPv6 site distant (lanR4)

### Adressage IPv6 Global Unicast site distant (interface G0/1 de R4)
- Adresses des réseaux maîtrisés, routé jusqu'à votre routeur externe : `2001:470:c814:5000::/52`
- L'adresse IPv6 externe de votre routeur : `fe80::cafe:5`
- L'adresse IPv6 de la passerelle vers l'Internet : `fe80::e53:21ff:fe38:5800`

### Configuration du routage IPv4
```
R4#configure terminal
R4(config)#router eigrp 1
R4(config-router)#passive-interface GigabitEthernet0/0
R4(config-router)#eigrp router-id 4.4.4.4
R4(config-router)#network 10.104.1.0 0.0.0.255
R4(config-router)#redistribute static
R4(config-router)#exit
R4(config)#exit
R4#wr
```
```
! Ajout de l'adresse IPv6 externe du routeur, type link-local
R4#configure terminal
R4(config)#interface G0/1
R4(config-if)#ipv6 address fe80::cafe:5 link-local
R4(config-if)#exit
R4(config)#exit
R4#wr
! Bloc d'adresses IPv6 2000:470:c814:5004::0/64 pour lanR4
R4#configure terminal
R4(config)#interface G0/2
R4(config-if)#ipv6 address 2001:470:c814:5004::2/64
R4(config-if)#ipv6 enable
R4(config-if)#exit
R4(config)#interface G0/0
R4(config-if)#ipv6 enable
R4(config-if)#exit
R4(config)#exit
R4#wr
```
### Configuration du routage IPv6 / Protocole propriétaire Cisco : eigrp
```
! Activation du routage IPv6
R4#configure terminal
R4(config)#ipv6 unicast-routing
R4(config)#exit
R4#wr
! Configuration du routage IPv6
R4#configure terminal
R4(config)#ipv6 router eigrp 1
R4(config-rtr)#eigrp router-id 4.4.4.4
R4(config-rtr)#passive-interface GigabitEthernet0/0
R4(config-rtr)#redistribute static
R4(config-rtr)#exit
R4(config)#interface GigabitEthernet0/0
R4(config-if)#ipv6 eigrp 1
R4(config-if)#exit
R4(config)#interface GigabitEthernet0/2
R4(config-if)#ipv6 eigrp 1
R4(config-if)#exit
! Route IPv6 statique de G0/1 vers l'adresse IPv6 de la passerelle vers l'internet
R4(config)#ipv6 route 2000::/3 GigabitEthernet0/1 FE80::E53:21FF:FE38:5800
R4(config)#exit
R4#wr
```
### Diagnostics
- Table de routage IPv6 de R4
```
R4#show ipv6 route
IPv6 Routing Table - default - 4 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       RL - RPL, O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1
       OE2 - OSPF ext 2, ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2
       la - LISP alt, lr - LISP site-registrations, ld - LISP dyn-eid
       lA - LISP away, a - Application
S   2000::/3 [1/0]
     via FE80::E53:21FF:FE38:5800, GigabitEthernet0/1
C   2001:470:C814:5004::/64 [0/0]
     via GigabitEthernet0/2, directly connected
L   2001:470:C814:5004::2/128 [0/0]
     via GigabitEthernet0/2, receive
L   FF00::/8 [0/0]
     via Null0, receive
R4#
```
- Ping depuis R4 vers google, domaine internet public (test connectivité IP et DNS)
```
R4#ping 8.8.8.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/3/4 ms
R4#
```
- Ping depuis R4 vers google.com, domaine internet public (test connectivité IP et DNS)
```
R4#ping google.com
Translating "google.com"...domain server (1.1.1.1) [OK]

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2A00:1450:400E:80D::200E, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/12/13 ms
R4#
```
- Ping depuis R4 vers test.tf
```
R4#ping ipv6 test.tf
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:41D0:305:1000::1D8A, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 7/8/13 ms
R4#
```
- Ping depuis la station de travail PC9 du lanR4 vers google, NAT PAT overload opérationnel sur R4
```
root@PC9:~# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=55 time=4.03 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=55 time=3.43 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=55 time=3.30 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=55 time=3.12 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=55 time=3.13 ms
^C
--- 8.8.8.8 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 3.129/3.405/4.032/0.339 ms
root@PC9:~#
```

### Configuration du routage IPv4
```
!
! Configuration du routage IPv4 / Protocole propriétaire Cisco : eigrp
!
configure terminal
 router eigrp 1
  passive-interface GigabitEthernet0/0
  eigrp router-id 4.4.4.4
  network 10.104.1.0 0.0.0.255
  redistribute static
  exit
! Ajout de l'adresse IPv6 externe du routeur, type link-local
 interface G0/1
  ipv6 address fe80::cafe:5 link-local
  exit
! Bloc d'adresses IPv6 2000:470:c814:5004::0/64 pour lanR4
 interface G0/2
  ipv6 address 2001:470:c814:5004::2/64
  ipv6 enable
  exit
 interface G0/0
  ipv6 enable
  exit
 exit
wr
```
### Configuration du routage IPv6 / Protocole propriétaire Cisco : eigrp
```
!
! Configuration du routage IPv6 / Protocole propriétaire Cisco : eigrp
!
! Activation du routage IPv6
configure terminal
 ipv6 unicast-routing
! Configuration du routage IPv6
 ipv6 router eigrp 1
  eigrp router-id 4.4.4.4
  passive-interface GigabitEthernet0/0
  redistribute static
  exit
 interface GigabitEthernet0/0
  ipv6 eigrp 1
  exit
 interface GigabitEthernet0/2
  ipv6 eigrp 1
  exit
! Route IPv6 statique de G0/1 vers l'adresse IPv6 de la passerelle vers l'internet
 ipv6 route 2000::/3 GigabitEthernet0/1 FE80::E53:21FF:FE38:5800
 exit
wr
