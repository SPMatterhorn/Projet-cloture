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

# Routeur R4

## Adressage IPv4
