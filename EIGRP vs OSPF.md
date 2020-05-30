# Pare-feu

Notre projet est basé sur une infrastructure homogène Cisco, nous avons donc configuré un pare-feu ZBF Cisco sur le routeur R1 (réseau principal) et sur le routeur R4 (site distant R4). Toutefois, nous avons quand même configuré un pare-feu Fortinet sur le routeur R5 (site distant R5) afin d'appréhender le principe de gestion unifiée des menaces (UTM), même si nous avons déployé uniquement un pare-feu.

Le pare-feu est un composant fondamental de la sécurité des réseaux. Chaque noeud d'interconnexion entre notre réseau interne et l'extérieur (vers l'internet), doit inclure une fonctionnalité de pare-feu.  Par défaut, une fois les fonctionnalités du pare-feu activées, aucun trafic ne pourra entrer ni sortir de notre réseau, principe du moindre privilège. Suivant les politiques de sécurité retenues, le pare-feu va filtrer tout ou partie du trafic le traversant, en ne laissant passer que les types de paquets explicitement mentionés dans sa configuration (exceptions).

Dans notre cas, ces noeuds se situent aux emplacements de R1 (site principal), R4 (site distant) et R5 (site distant). On active les fonctionnalités de pare-feu Cisco sur les routeurs R1 et R4. R5 est un pare-feu Fortinet incluant des fonctionnalités de routage et de NAT. Nous souhaitions en effet réutiliser un pare-feu Fortinet, acteur majeur du marché au même titre que Cisco.

| **security zone** | **interface** | **zone-pair** | **Niveau de Confiance** |
| :-| :- | :- | :-: |
| internet | g0/1 | internet-dmz, internet-self | 0% |
| lan | g0/2, g0/3 | lan-internet, lan-dmz | 100% |
| dmz | g0/0 | - | à risque |
| self-zone | all | self-internet, internet-self | Firewall ZBF R1 |

| **zone-pair** | **source**| **destination** | **policy-map / action** | **class-map** |
| :- | :- | :- | :- | :- |
| lan-internet | lan | internet | lan-internet-policy / inspect | internet-trafic-class, default-class |
| lan-dmz | lan | dmz | lan-dmz-policy / inspect | lan-dmz-class, default-class |
| internet-dmz | internet | dmz | internet-dmz-policy / inspect | internet-dmz-class, default-class |
| internet-self | internet | self | to-self-policy / inspect | remote-access-class, icmp-class, dhcp-class, dns-class, ntp-class, class-default |
| self-internet | self | internet | to-self-policy / inspect | remote-access-class, icmp-class, dhcp-class, dns-class, ntp-class, class-default |



| **class-map / action** | **match class-map** |
| :- | :- |
| internet-trafic-class / inspect| dns, http, https, icmp |
| lan-dmz-class / inspect | http, https |
| internet-dmz-class / inspect | http, https |

 class type inspect remote-access-class
  pass
 class type inspect icmp-class
  inspect
 class type inspect dhcp-class
  pass
 class type inspect dns-class
  pass
 class type inspect ntp-class
  pass
 class class-default
  drop log

class-map type inspect match-any internet-trafic-class
 match protocol dns
 match protocol http
 match protocol https
 match protocol icmp
 match protocol ssh

| lan-dmz | lan | dmz | lan-dmz-policy | internet-trafic-class/inspect |

zone security lan
zone security internet
zone security dmz
zone-pair security lan-internet source lan destination internet
 service-policy type inspect lan-internet-policy
zone-pair security lan-dmz source lan destination dmz
 service-policy type inspect lan-dmz-policy
zone-pair security internet-dmz source internet destination dmz
 service-policy type inspect internet-dmz-policy
zone-pair security internet-self source internet destination self
 service-policy type inspect to-self-policy
zone-pair security self-internet source self destination internet
 service-policy type inspect to-self-policy


Cisco ZBF
Zone-based policy firewall
Le modèle de configuration Zone-based policy firewall (ZPF or ZBF or ZFW) a été introduit en 2006 avec l’IOS 12.4(6)T.

Avec ZBF, les interfaces sont assignées à une des zones sur lesquelles une règle d’inspection du trafic (inspection policy) est appliquée. Elle vérifie le trafic qui transite entre les zones.

Une règle par défaut bloque tout trafic tant qu’une règle explicite ne contredit pas ce comportement.

ZBF supporte toutes les fonctionnalités Stateful Packet Inspection (SPI), filtrage des URLs et contre-mesure des DoS.

Principes ZBF
Une zone doit être configurée (créée) avant qu’une interface puisse en faire partie. Une interface ne peut être assignée qu’à une seule zone.

Tout le trafic vers ou venant d’une interface donnée est bloqué quand elle est assignée à une zone sauf pour le trafic entre interfaces d’une même zone et pour le trafic du routeur lui-même (Self zone).

Une politique de sécurité (zone-pair) peut contrôler le trafic entre deux zones en faisant référence à un en ensemble de règles (policy-map).

Un policy-map prend des actions et fait référence à des critères de filtrage (class-maps).

Quand du trafic passe d’une zone à une autre (zone-pair), un policy-map est appliqué.

Pour chaque class-map (critère de filtrage) du policy-map, une action est prise : pass, inspect ou drop de manière séquentielle.

Il est conseillé de travailler dans un éditeur de texte avant d’appliquer les règles du firewall.

Trois actions sur les class-maps
Inspect
Met en place un pare-feu à état (équivalent à la commande ip inspect) SPI.

Capable de suivre les protocoles comme ICMP ou FTP (avec de multiples connexions data et session)

Pass
Équivalent à l’action permit d’un ACL.

Ne suit pas l’état des connexions ou des sessions.

Nécessite une règle correspondante pour du trafic de retour.

Drop
Équivalent à l’action deny d’un ACL.

Une option log est possible pour journaliser les paquets rejetés.

Règles de filtrage : policy-maps
Les actions Inspect, Pass et Drop ne peuvent être appliquées qu’entre des interfaces appartenant à des zones distinctes.

La Self zone, la zone du routeur/parefeu comme source ou destination est une exception à ce refus implicite de tout. Tout le trafic vers n’importe quelle interface du routeur est autorisé jusqu’au moment où il est implicitement refusé.

Les interfaces qui ne participent pas à ZBF fonctionnent comme des ports classiques et peuvent utiliser une configuration SPI/CBAC.











# NAT

Notre bloc IPv4 10.0.0.0 /8 (Plage d'adresses privées de la Classe A, défini par le RFC 1918) n'est pas routable vers l'internet. 
Nous devons utiliser le NAT "Network Address Translation" 
R1, R4 et R5 gèrent aussi les fonctionnalités de NAT pour leurs réseaux respectifs. (Traduction d'adresses IPv4 privées en adresses IPv4 publiques ou privées, privées dans notre cas en raison du cas particulier de notre environnement GNS3). R1, R4 et R5 ne sont pas directement reliés à l'internet (domaine IPv4 publique), un réseau en IPv4 privé 
La liste d'accès du NAT contribue indirectement à la politique de sécurité en autorisant tout ou partie de notre bloc d'adresses privées à joindre l'internet ou à être contacté par l'internet.







L'infrastructure nécessite l'utilisation de Firewalls (pare-feu) en extrimité de réseau, afin de filtrer les trafics entrant et sortant. 


Ces derniers n'assurent pas la sécurité au sein même du réseau. Il est crucial de déterminer les flux autorisés nécessaires.

Le bloc principal suit architecture Core-Distribution-Access, dont le noeud d'interconnexion vers les réseaux extérieurs/étrangers (Zone de confiance 0%) se situe au niveau du routeur R1.
Nous utilisons une infrastrucuture homogène Cisco par conséquent nous activons les fonctionalités firewall du pare-feu Cisco sur le routeur cisco R1.





Quelques éléments essentiels sont à retenir concernant les pare-feux

Dans un système d’information, les politiques de filtrage et de contrôle du trafic sont placées sur un matériel ou un logiciel intermédiaire communément appelé pare-feu (firewall).
Cet élément du réseau a pour fonction d’examiner et filtrer le trafic qui le traverse.
On peut le considérer comme une fonctionnalité d’un réseau sécurisé : la fonctionnalité pare-feu
L’idée qui prévaut à ce type de fonctionnalité est le contrôle des flux du réseau TCP/IP.
Le pare-feu limite le taux de paquets et de connexions actives. Il reconnaît les flux applicatifs.
Se placer au milieu du routage TCP/IP, il fait office de routeur
Il agit au minimum au niveau de la couche 4 (L4) mais il peut inspecter du trafic L7 (Web Application Firewall)
Il ne faut pas le confondre avec le routeur NAT
2. Objectifs d’un pare-feu
Il a pour objectifs de répondre aux menaces et attaques suivantes, de manière non-exhaustive :

Usurpation d’identité
La manipulation d’informations
Les attaques de déni de service (DoS/DDoS)
Les attaques par code malicieux
La fuite d’information
Les accès non-autorisé (en vue d’élévation de privilège)
Les attaques de reconnaissance, d’homme du milieu, l’exploitation de TCP/IP
3. Ce que le pare-feu ne fait pas
Le pare-feu est central dans une architecture sécurisée mais :

Il ne protège pas des menaces internes.
Il n’applique pas tout seul les politiques de sécurité et leur surveillance.
Il n’établit pas la connectivité par défaut.
Le filtrage peut intervenir à tous les niveaux TCP/IP de manière très fine.
4. Fonctionnement
Il a pour principale tâche de contrôler le trafic entre différentes zones de confiance, en filtrant les flux de données qui y transitent.
Généralement, les zones de confiance incluent l’Internet (une zone dont la confiance est nulle) et au moins un réseau interne (une zone dont la confiance est plus importante).
Le but est de fournir une connectivité contrôlée et maîtrisée entre des zones de différents niveaux de confiance, grâce à l’application de la politique de sécurité et d’un modèle de connexion basé sur le principe du moindre privilège.
Un pare-feu fait souvent office de routeur et permet ainsi d’isoler le réseau en plusieurs zones de sécurité appelées zones démilitarisées ou DMZ. Ces zones sont séparées suivant le niveau de confiance qu’on leur porte.
5. Zone de confiance sur un pare-feu
Un organisation du réseau en zones composées d’interfaces permet d’abstraire les règles de filtrages.


Zones de confiance
À titre d’exemple, les politiques de filtrage des applications pourrait être facilement mis en oeuvre quelque soit le protocole de transport IPv4 ou IPv6.

Aussi les politiques de sécurité appliquée sur les pare-feux sont alors plus lisibles, plus faciles à auditer et à gérer.

6. Niveau de confiance
Le niveau de confiance est la certitude que les utilisateurs vont respecter les politiques de sécurité de l’organisation.
Ces politiques de sécurité sont édictées dans un document écrit de manière générale. Ces recommandations touchent tous les éléments de sécurité de l’organisation et sont traduites particulièrement sur les pare-feu en différentes règles de filtrage.
On notera que le pare-feu n’examine que le trafic qui le traverse et ne protège en rien des attaques internes, notamment sur le LAN.
7. Politiques de filtrage
Selon les besoins, on placera les politiques de filtrage à différents endroits du réseau, au minimum sur chaque hôte contrôlé (pare-feu local) et en bordure du réseau administré sur le pare-feu. Ces emplacements peuvent être distribué dans la topologie selon sa complexité.
Pour éviter qu’il ne devienne un point unique de rupture, on s’efforcera d’assurer la redondance des pare-feu. On placera plusieurs pare-feu dans l’architecture du réseau à des fins de contrôle au plus proche d’une zone ou pour répartir la charge.
8. Filtrage
La configuration d’un pare-feu consiste la plupart du temps en un ensemble de règles qui déterminent une action de rejet ou d’autorisation du trafic qui passe les interfaces du pare-feu en fonction de certains critères tels que :
l’origine et la destination du trafic,
des informations d’un protocole de couche 3 (IPv4, IPv6, ARP, etc.),
des informations d’un protocole de couche 4 (ICMP, TCP, UDP, ESP, AH, etc.)
et/ou des informations d’un protocole applicatif (HTTP, SMTP, DNS, etc.).


https://cisco.goffinet.org/ccnp/filtrage/lab-pare-feu-cisco-ios-zbf-zone-based-firewall/











À compléter (Stéphane ?)



