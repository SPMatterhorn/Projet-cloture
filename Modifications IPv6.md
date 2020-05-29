# Modification de l'adressage IPv6, Topologie Principale
- Passerelles virtuelles IPv6 VLANs, passerelle doit être identique pour chaque VLAN et différente des link local de DS1 et DS2 (voir tableur pour le choix retenu)
- Ajout des IPv6 publiques dans la topologie principale (couche Core et couche distribution => R1, R2, R3, DS1 et DS2 
- Adaptation hsrpv6 (standby, preempt... pour chaque vlan et pour chaque DS, soit 8 blocs à modifier)

```
! Configuration supplémentaire HSRP DS1 IPv6
interface vlan 10
 standby 16 priority 150
 standby 16 preempt
 standby version 2
 standby 16 ipv6 fe80::d3:1
!
interface vlan 20
 standby version 2
 standby 26 ipv6 fe80::d3:1
!
interface vlan 30
 standby 36 priority 150
 standby 36 preempt
 standby version 2
 standby 36 ipv6 fe80::d3:1
!
interface vlan 40
 standby version 2
 standby 46 ipv6 fe80::d3:1
end
wr
```
```
! Configuration supplémentaire HSRP DS2 IPv6
interface vlan 10
 standby version 2
 standby 16 ipv6 fe80::d3:1
!
interface vlan 20
 standby 26 priority 150
 standby 26 preempt
 standby version 2
 standby 26 ipv6 fe80::d3:1
!
interface vlan 30
 standby version 2
 standby 36 ipv6 fe80::d3:1
!
interface vlan 40
 standby 46 priority 150
 standby 46 preempt
 standby version 2
 standby 46 ipv6 fe80::d3:1
end
wr
```
- Modification Configuration des Machines Virtuelles Ubuntu
```
systemctl disable systemd-resolved
systemctl stop systemd-resolved
rm -f /etc/resolv.conf
echo "nameserver 1.1.1.1" >> /etc/resolv.conf
chattr +i /etc/resolv.conf
```
