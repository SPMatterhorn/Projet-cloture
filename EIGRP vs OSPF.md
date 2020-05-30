# Choix du protocole de routage

https://openclassrooms.com/fr/courses/2557196-administrez-une-architecture-reseau-avec-cisco/5135481-choisissez-entre-ospf-et-eigrp-pour-votre-topologie

## EIGRP : Enhanced Interior Gateway Routing Protocol

- Protocole de routage à vecteur de distance, propriétaire Cisco
- Peut être utilisé par les autres constructeurs, mais seul Cisco peut le modifier
- Protocole hybride qui peut réutiliser certaines fonctionnalités d'un protocole de routage à état de lien
- Résout les problèmes de boucle
- Route vers le plus court chemin + une route de secours 
- Utilise la métrique en prenant en compte la bande passante et les délais, et nom les sauts (contrairement à RIP) => Hybride
- Load balanching (répartition de charge), Bandes passantes égales ou inégales (deux câbles avec des capacités différentes)
- Distance Administrative : 90, plus la distance est faible, plus le protocole est fiable, le protocole de routage avec la plus faible distance administrative est préféré pour alimenter les tables de routage.


## OSPF
- Protocole de routage à état de lien, issu d'une RFC de l'IETF

|**EIGRP|OSPF**|
|:-:|:-:|
|Protocole de routage Hybride|Protocole de routage à état de lien |
|Priopriétaire Cisco, Standard récent|Standard IETF|
|Distance Administrative: 90|Distance Administrative: 110|
|Route secondaire de secours|Pas de route secondaire|
|Algorithme DUAL|Algorithme Dijkstra|
|Système autonome|Division du réseau en aires|



Eigrp est plus fiable qu' Ospf



## Couche Core (Tripod) 
|Périphérique|Interface|Liaison<br>|Adresse IPv4 statique|Adresse<br>IPv6 Link-local|Adresse IPv6 publique|Adresse IPv6 privée|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|R1|G0/2|R1-R2|10.1.1.1|fe80::dd:1|2001:470:c814:1001::1|fd00:470:c814:1001::1|
|R1|G0/3|R1-R3|10.1.2.1|fe80::dd:1|2001:470:c814:1002::1|fd00:470:c814:1002::1|
|R2|G0/1|R2-R1|10.1.1.2|fe80::dd:2|2001:470:c814:1001::2|fd00:470:c814:1001::2|
|R2|G0/3|R2-R3|10.1.3.2|fe80::dd:2|2001:470:c814:1003::2|fd00:470:c814:1003::2|
|R3|G0/1|R3-R1|10.1.2.2|fe80::dd:3|2001:470:c814:1002::2|fd00:470:c814:1002::2|
|R3|G0/2|R3-R2|10.1.3.1|fe80::dd:3|2001:470:c814:1003::1|fd00:470:c814:1003::1|

OSPF

EIGRP

C’est un standard de l’IETF, il est donc présent sur la totalité des routeurs que vous rencontrerez.

C’est devenu un standard depuis quelques années, mais il n’est pas encore présent sur tous les routeurs du marché.

Distance administrative 110.

Distance administrative 90.

À état de lien

Hybride

Load-balancing sur câble de même bande passante.

Load-balancing sur câble de bande passante différente.

Il ne possède pas de route secondaire.

Il possède une route secondaire.

Il peut diviser le réseau en aire pour limiter l’utilisation de la bande passante.

Notion de système autonome

Il sélectionne un routeur désigné.

Il envoie les messages à tous les routeurs.

Métrique par le coût (la bande passante).

Métrique par la bande passante et le délai (plus d’autres facteurs en option).

Aucune notion de saut.

Pas plus de 224 sauts.

Algorithme de Dijkstra

Algorithme DUAL

Un peu long à configurer au départ (c’est un avis personnel)

Assez facile à configurer.
