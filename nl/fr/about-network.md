---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: secure, region, zone, subnet, terminology, public gateway, floating IP, NAT

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:download: .download}

# A propos de la mise en réseau pour VPC
{: #about-networking-for-vpc}

Un cloud privé virtuel (VPC) est un réseau virtuel lié à votre compte client. Il vous offre une sécurité cloud, avec la possibilité d'évoluer de manière dynamique, en fournissant un contrôle fin de votre infrastructure virtuelle et de la segmentation de votre trafic réseau.

Le présent document décrit certains concepts de mise en réseau lorsqu'ils sont appliqués dans un VPC {{site.data.keyword.cloud}}. Les caractéristiques et les cas d'utilisation
décrits sont les suivants :

* régions
* zones
* adresses IP réservées
* passerelles publiques
* interfaces vNIC
* sous-réseaux
* adresses IP flottantes
* connexions VPN

## Présentation
{: #subnets-overview}

Trois options sont disponibles pour accéder à Internet à partir de votre VPC :
* Utiliser une passerelle publique pour le trafic Internet depuis l'ensemble du sous-réseau.
* Utiliser une adresse IP flottante pour le trafic Internet en provenance et à destination d'une instance.
* Utiliser un VPN pour une connectivité externe sécurisée.

Rappel :
* Certaines plages CIDR d'adresses de sous-réseau sont réservées par IBM.
* Vous devez créer votre VPC avant de créer des sous-réseaux au sein de ce VPC.
* La prise en charge d'IPv6 n'est pas disponible.

Vous pouvez également créer un VPC Classic Access pour vous connecter à votre infrastructure IBM Cloud Classic.
{:note}

Comme le montre la figure suivante :
* Les sous-réseaux dans votre VPC peuvent se connecter à Internet via une passerelle publique facultative.
* Vous pouvez assigner une adresse IP flottante (ou FIP) à n'importe quelle instance pour l'atteindre depuis Internet, ou vice-versa, indépendamment du fait que le sous-réseau soit associé à une passerelle publique.
* Les sous-réseaux dans le VPC IBM Cloud offrent une connectivité privée ; ils peuvent communiquer via une liaison privée, par le biais du routeur implicite. La configuration de routes n'est pas nécessaire.

![Connectivité et sécurité du VPC IBM](images/vpc-connectivity-and-security.svg "Connectivité et sécurité du VPC IBM")

## Terminologie
{: #network-terminology}

Ce [glossaire](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary#vpc-glossary) contient des définitions et des informations sur les termes utilisés dans le présent document pour le VPC IBM Cloud. Lorsque vous utilisez votre VPC, vous devez être familiarisé avec les concepts de base _région_ et _zone_, car ils s'appliquent à votre déploiement.

### Régions
{: #subnet-regions}

Une région est une abstraction liée à la zone géographique dans laquelle un VPC est déployé. Chaque région contient plusieurs zones, qui représentent des domaines d'erreur indépendants. Un VPC IBM Cloud peut s'étendre sur plusieurs zones dans la région qui lui a été affectée.

### Zones
{: #subnet-zones}

Une zone est une abstraction qui fait référence au centre de données physique qui héberge les ressources de calcul, de réseau et de stockage, ainsi que le refroidissement et l'alimentation associés, fournissant des services et des applications. L'isolement des zones améliore la tolérance aux pannes globale du système, diminue la latence et évite la création d'un seul point de défaillance commun. Une zone garantit les propriétés suivantes :

 * Chaque zone est un domaine de panne indépendant et il est extrêmement improbable que deux zones d'une région tombent en panne simultanément.
 * Le trafic entre les zones d'une région a une latence inférieure à 2 ms.

## Caractéristiques des sous-réseaux dans le VPC
{: #characteristics-of-subnets}

Un sous-réseau est constitué d'une plage d'adresses IP spécifiée (bloc CIDR). Les sous-réseaux sont liés à une seule zone et ils ne peuvent pas couvrir plusieurs zones ou régions. Toutefois, un sous-réseau peut couvrir l'intégralité des abstractions de zone dans leur cloud privé virtuel. Les sous-réseaux du même IBM Cloud VPC sont connectés les uns aux autres.

### Adresses IP réservées
{: #reserved-ip-addresses}

Certaines adresses IP sont réservées à l'utilisation par IBM lors de l'exploitation du cloud privé virtuel. Voici les adresses réservées (ces adresses IP supposent que la plage CIDR du sous-réseau est 10.10.10.0/24) :

  * Première adresse dans la plage CIDR (10.10.10.0) : adresse réseau
  * Deuxième adresse dans la plage CIDR (10.10.10.1) : adresse de passerelle
  * Troisième adresse de la plage CIDR (10.10.10.2) : réservée par IBM
  * Quatrième adresse de la gamme CIDR (10.10.10.3) : réservée par IBM pour une utilisation future
  * Dernière adresse dans la plage CIDR (10.10.10.255) : adresse de diffusion réseau

### Utilisation d'une passerelle publique pour la connectivité externe d'un sous-réseau
{: #use-a-public-gateway}

Une **passerelle publique** permet à un sous-réseau (avec toutes les instances liées au sous-réseau) de se connecter à Internet. Notez que les sous-réseaux sont privés par défaut. Toutefois, vous pouvez créer une passerelle publique et associer un sous-réseau à cette dernière si vous le souhaitez. Une fois qu'un sous-réseau est connecté à la passerelle publique, toutes les instances de ce sous-réseau peuvent se connecter à Internet.

La passerelle publique utilise la conversion _NAT plusieurs-à-1_, ce qui signifie que plusieurs milliers d'instances dotées d'adresses privées utilisent une adresse IP publique pour communiquer avec l'internet public. La passerelle publique ne permet pas à Internet d'établir une connexion avec ces instances. Utilisez l'API pour connecter et déconnecter des sous-réseaux vers et depuis votre passerelle publique.

La figure suivante résume la portée actuelle des services de passerelle.

| SNAT | DNAT | ACL | VPN |
| ---- | ---- | --- | --- |
| Les instances peuvent avoir un accès uniquement sortant à Internet | Autorisez la connectivité entrante depuis Internet vers une adresse IP privée | Fournissez un accès entrant restreint depuis Internet vers les instances ou les sous-réseaux | Le VPN de site à site traite les clients de toute taille, et les emplacements uniques ou multiples |
| Les sous-réseaux entiers partagent le même noeud final public sortant | Fournit un accès limité à un serveur privé unique | Limitez l'accès entrant depuis Internet en fonction du service, du protocole ou du port | Le haut débit (jusqu'à 10 Gbits/s) permet aux clients de transférer des fichiers de données volumineux rapidement et en toute sécurité |
| Protège les instances. L'accès aux instances via le noeud final public n'est pas possible | Le service DNAT peut être augmenté ou réduit en fonction des besoins | Les listes de contrôle d'accès sans état permettent un contrôle granulaire du trafic | Créez des connexions sécurisées avec un chiffrement conforme aux normes de l'industrie  |

Une passerelle publique est créée dans un VPC, mais elle ne fait rien tant qu'elle n'est pas connectée à un sous-réseau. Vous ne pouvez créer qu'une passerelle publique par zone, ce qui signifie, par exemple, que vous pouvez avoir trois passerelles publiques par VPC dans un environnement avec 3 zones.
{:note}

## Limitations des sous-réseaux
{: #limitations-of-subnets}

Pour obtenir une liste complète des limitations connues et des fonctions qui ne sont pas encore prises en charges, veuillez vous reporter au document [Limitations connues](/docs/vpc-on-classic?topic=vpc-on-classic-known-limitations).

### Restrictions relatives à la suppression d'un sous-réseau
{: #restrictions-on-deleting-a-subnet}

Vous ne pouvez pas supprimer un sous-réseau si des ressources (telles qu'une instance de serveur virtuel (VSI) ou des adresses IP flottantes) sont utilisées dans ce sous-réseau. Les ressources doivent d'abord être supprimées.

### Limitations de la mise à jour d'un sous-réseau existant 
{: #limitations-on-updating-an-existing-subnet}

* Vous ne pouvez pas redimensionner un sous-réseau existant. Par exemple, 10.10.16.0/24 ne peut pas être redimensionné à 10.10.16.0/20.
* Vous ne pouvez pas déplacer un sous-réseau existant. Par exemple, 10.10.10.0/24 ne peut pas être déplacé à 10.10.11.0/24.

## Connectivité externe
{: #external-connectivity}

La connectivité externe peut être assurée au moyen d'une adresse IP flottante connectée à une instance, par une passerelle externe connectée à un sous-réseau, ou par un tunnel VPN.

### Utilisation d'une adresse IP flottante pour la connectivité externe d'une instance 
{: #use-floating-ip}

Les **adresses IP flottantes** sont des adresses IP fournies par le système et accessibles depuis l'Internet public.

Vous pouvez réserver une adresse IP flottante dans le pool d'adresses IP flottantes disponibles fournies par IBM et vous pouvez l'associer ou la dissocier à n'importe quelle instance du même cloud privé virtuel au moyen de la carte vNIC correspondant à cette instance. Toute adresse IP flottante peut être associée à différents types d'instances de serveur virtuel (VSI), par exemple un équilibreur de charge ou une passerelle VPN.

Votre adresse IP flottante ne peut pas être associée à plusieurs interfaces.Vous devez spécifier l'interface sur la VSI qui sera associée à cette adresse IP flottante individuelle. Cette interface possède également une adresse IP privée. Le système de back end effectue des opérations _NAT 1-à-1_ entre l'adresse IP flottante et l'adresse IP privée de cette interface. Une adresse IP flottante est libérée uniquement lorsque vous demandez spécifiquement l'action de libération.

**Remarques :**
* **L'association d'une adresse IP flottante à une VSI supprime la VSI de la conversion NAT plusieurs-à-un de la passerelle publique mentionnée précédemment.**
* **Actuellement, les adresses IP flottantes ne prennent en charge que les adresses IPv4.**
* **Vous ne pouvez pas utiliser votre propre adresse IP publique comme adresse IP flottante.**

Pour plus d'informations sur les opérations NAT, reportez-vous au [document RFC Internet associé ![Icône de lien externe](../../icons/launch-glyph.svg "Icône de lien externe")](http://www.faqs.org/rfcs/rfc1631.html){: new_window}.

### Utilisation d'un VPN pour une connectivité externe sécurisée
{: #use-vpn}

Le service VPN (Virtual Private Network) permet aux utilisateurs de se connecter au VPC IBM Cloud depuis Internet en toute sécurité.

**Fonctionnalités de VPN**
  * Possibilité d'effectuer une opération CRUD (Create, Read, Update and Delete - Créer, Lire, Mettre à jour et Supprimer) sur un service VPN (VPN IPSEC de site à site) pour gérer le transfert de données.
  * Possibilité d'associer un service VPN à IBM Cloud VPC ou au réseau virtuel d'un client.
  * Possibilité d'identifier les sous-réseaux sur site du client, soit par définition statique, soit par routage dynamique.
  * Prise en charge des chiffrements sécurisés tels que SHA256, AES, 3DES, IKEv2.
  * Fiabilité du service intégré.
  * Surveillance continue de la santé de la connexion VPN.
  * Prise en charge des modes initiateur et répondeur ; c'est-à-dire que le trafic peut être lancé de chaque côté du tunnel.

## En savoir plus
{: #subnets-learn-more}
   * [Sécurité d'un VPC IBM](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-security-in-your-ibm-cloud-vpc)
   * [Adresses, régions et sous-réseaux d'un VPC IBM](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-working-with-ip-address-ranges-address-prefixes-regions-and-subnets)
