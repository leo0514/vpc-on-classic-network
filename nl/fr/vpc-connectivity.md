---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-14"

keywords: capabilities, use cases, subnets, VPN, connections, reserved, IP, IPv4, floating

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# A propos de la mise en réseau pour VPC
{: #about-networking-for-vpc}

Vous trouverez dans ce document des concepts qui décrivent les cas d'utilisation et les fonctionnalités du VPC {{site.data.keyword.cloud}} pour les sous-réseaux, les adresses IP flottantes et les connexions VPN.

![Connectivité et sécurité dans le VPC IBM](images/vpc-connectivity-and-security.svg "Connectivité et sécurité dans le VPC IBM")

Comme le montre la figure :

* Les sous-réseaux peuvent se connecter à l'Internet public via une passerelle publique facultative.
* Vous pouvez attribuer une adresse IP flottante (FIP) à n'importe quelle VSI pour l'atteindre à partir d'Internet ou inversement.
* Les sous-réseaux dans IBM Cloud VPC offrent une connectivité privée ; ils peuvent communiquer entre eux sur une liaison privée. Il est inutile de configurer des routes.
* Pour plus d'informations, consultez notre rubrique [A propos de l'infrastructure du VPC](/docs/vpc-on-classic?topic=vpc-on-classic-about).

## Terminologie
{: #terminology}

Ce [glossaire](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary) contient des définitions et des informations sur les termes utilisés dans le présent document pour IBM Cloud VPC.

## Caractéristiques des sous-réseaux dans le VPC
{: #characteristics-of-subnets-in-the vpc}

Un sous-réseau est constitué d'une plage d'adresses IP spécifiée (bloc CIDR). Les sous-réseaux sont liés à une seule zone et ils ne peuvent pas couvrir plusieurs zones ou régions. Toutefois, un sous-réseau peut couvrir l'intégralité des abstractions de zone dans leur cloud privé virtuel. Les sous-réseaux du même IBM Cloud VPC sont connectés les uns aux autres.

### Zones
{: #zones}

Une zone est une abstraction conçue pour aider grâce à une tolérance aux pannes améliorée et des temps d'attente réduits. Une zone garantit les propriétés suivantes :

 * Chaque zone est un domaine de panne indépendant et il est extrêmement improbable que deux zones d'une région tombent en panne simultanément
 * Le trafic entre les zones d'une région a un temps d'attente inférieur à 2 ms

### Adresses IP réservées
{: #reserved-ip-addresses}

Certaines adresses IP sont réservées à l'utilisation par IBM lors de l'exploitation du cloud privé virtuel. Voici les adresses réservées (les adresses IP données supposent que la plage CIDR du sous-réseau est 10.10.10.0/24) :

  * Première adresse dans la plage CIDR (10.10.10.0) : adresse réseau
  * Deuxième adresse dans la plage CIDR (10.10.10.1) : adresse de passerelle
  * Troisième adresse de la plage CIDR (10.10.10.2) : réservée par IBM
  * Quatrième adresse de la gamme CIDR (10.10.10.3) : réservée par IBM pour une utilisation future
  * Dernière adresse dans la plage CIDR (10.10.10.255) : adresse de diffusion réseau

### Utilisation d'une passerelle publique pour la connectivité externe d'un sous-réseau
{: #use-a-public-gateway-for-external-connectivity-of-a-subnet}

Une **passerelle publique** permet à un sous-réseau (dont toutes les VSI sont liées au sous-réseau) de se connecter à Internet. Notez que les sous-réseaux sont privés par défaut. Toutefois, vous pouvez éventuellement créer une passerelle publique et associer un sous-réseau à cette dernière. Une fois qu'un sous-réseau est connecté à la passerelle publique, toutes les VSI de ce sous-réseau peuvent se connecter à Internet.

La passerelle publique utilise la conversion _NAT plusieurs-à-un_, ce qui signifie que des milliers de VSI dotées d'adresses privées utilisent une adresse IP publique pour communiquer avec l'Internet public. La passerelle publique ne permet pas à Internet d'établir une connexion avec ces instances. Utilisez l'API pour connecter et déconnecter des sous-réseaux vers et depuis votre passerelle publique.

La figure suivante résume la portée des services de passerelle.

![services passerelle](images/scope-of-gateway-services.png)

### Utilisation d'une adresse IP flottante pour la connectivité externe d'une VSI
{: #use-a-floating-ip-address-for-external-connectivity-of-a-vsi}

Les **adresses IP flottantes** sont les adresses IP fournies par le système et accessibles depuis l'internet public.

Vous pouvez réserver une adresse IP flottante dans le pool d'adresses IP flottantes disponibles fournies par IBM et vous pouvez l'associer ou la dissocier à n'importe quelle instance du même cloud privé virtuel au moyen de la carte vNIC correspondant à cette instance. Toute adresse IP flottante peut être associée à différents types d'instances de serveur virtuel (VSI), par exemple un équilibreur de charge ou une passerelle VPN.

Votre adresse IP flottante ne peut pas être associée à plusieurs interfaces.Vous devez spécifier l'interface sur la VSI qui sera associée à cette IP flottante individuelle. Cette interface possède également une adresse IP privée. Le système de back end effectue des opérations _NAT 1-à-1_ entre l'adresse IP flottante et l'adresse IP privée de cette interface. Une adresse IP flottante est libérée uniquement lorsque vous demandez spécifiquement l'action de libération.

**Remarques :**
* **L'association d'une adresse IP flottante à une VSI supprime la VSI de la conversion NAT plusieurs-à-un de la passerelle publique mentionnée précédemment.**
* **Actuellement, les adresses IP flottantes ne prennent en charge que les adresses IPv4.**
* **Vous ne pouvez pas utiliser votre propre adresse IP publique comme adresse IP flottante.**

Pour plus d'informations sur les opérations NAT, reportez-vous au [document RFC Internet associé ![Icône de lien externe](../../icons/launch-glyph.svg "Icône de lien externe")](http://www.faqs.org/rfcs/rfc1631.html){: new_window}.

### Utilisation d'un VPN pour la connectivité externe sécurisée
{: #use-a-vpn-for-secure-external-connectivity}

Le service de réseau privé virtuel (VPN) est disponible pour permettre aux utilisateurs de se connecter en toute sécurité à leur IBM Cloud VPC à partir d'Internet. Pour des instructions détaillées, veuillez consulter notre [Guide de l'interface utilisateur de la console IBM](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console).

**Fonctionnalités de VPN**
  * Possibilité d'effectuer une opération CRUD (Create, Read, Update and Delete - Créer, Lire, Mettre à jour et Supprimer) sur un service VPN (VPN IPSEC de site à site) pour gérer le transfert de données.
  * Possibilité d'associer un service VPN à IBM Cloud VPC ou au réseau virtuel d'un client.
  * Possibilité d'identifier les sous-réseaux sur site du client, soit par définition statique, soit par routage dynamique.
  * Prise en charge de chiffrements sécurisés tels que SHA256, AES, 3DES, IKEv2
  * Fiabilité du service intégré.
  * Surveillance de la santé de la connexion VPN.
  * Prise en charge des modes initiateur et répondeur ; c'est-à-dire que le trafic peut être lancé de chaque côté du tunnel.
