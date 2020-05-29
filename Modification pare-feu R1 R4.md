# Ajustement des politiques de pare-feu

## Sur R1 :
- Permettre à R1 de pouvoir réaliser ses tests de connectivité et de résolution de nom
```
! ajout d'une nouvelle zone-pair de la self-zone vers l'internet
! réutilisation de la service-policy déjà utilisée pour la zone-pair de l'internet vers la self-zone
configure terminal
 zone-pair security self-internet source self destination internet
   service-policy type inspect to-self-policy
   end
wr
```
- Correction des logs altérants la console de R1 (DHCP)
```
*May 28 19:46:10.318: %FW-6-DROP_PKT: Dropping udp session 0.0.0.0:68 255.255.255.255:67 
on zone-pair internet-self class class-default due to  DROP action found in policy-map with ip ident 38949
```
```
! modification de l'access-list de l'access-group DHCP
configure terminal
 ip access-list extended DHCP
! déjà fait  no permit udp any any eq 68
! déjà fait  no deny udp any any
  permit udp any eq 67 any eq 68
  permit udp any eq 68 any eq 67
  end
wr
```

## Sur R4
- Permettre à R4 de pouvoir réaliser ses tests de connectivité et de résolution de nom
```
! ajout d'une nouvelle zone-pair de la self-zone vers l'internet
! réutilisation de la service-policy déjà utilisée pour la zone-pair de l'internet vers la self-zone
configure terminal
 zone-pair security self-internet source self destination internet
   service-policy type inspect to-self-policy
   end
wr
```
- Correction des logs altérants la console de R1 (DNS et DHCP)
```
*May 28 19:46:10.318: %FW-6-DROP_PKT: Dropping udp session 0.0.0.0:68 255.255.255.255:67 
on zone-pair internet-self class class-default due to  DROP action found in policy-map with ip ident 38949
```
```
*May 28 19:48:12.605: %FW-6-DROP_PKT: Dropping udp session 1.1.1.1:53 192.168.122.162:51761 
on zone-pair internet-self class class-default due to  DROP action found in policy-map with ip ident 18527
```
```
! modification de l'access-list de l'access-group DHCP
configure terminal
 ip access-list extended DHCP
! déjà fait  no permit udp any any eq 68
! déjà fait  no deny udp any any
  permit udp any eq 67 any eq 68
  permit udp any eq 68 any eq 67
  end
wr
```
```
! modification de l'access-list de l'access-group DNS
configure terminal
 ip access-list extended DNS
  permit udp any any eq 53
  permit udp any eq 53 any
  deny udp any any
  end
wr
```
