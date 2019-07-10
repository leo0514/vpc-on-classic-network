---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-07"

keywords: vpc, address prefix, region, subnet, zone, reserved, IP, ranges, deleting, creating, CIDR

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

# Connaissance des plages d'adresses IP, des préfixes d'adresse, des régions et des sous-réseaux
{: #working-with-ip-address-ranges-address-prefixes-regions-and-subnets}
[comment]: # (rubrique d'aide associée)

Ce document traite des relations entre les régions, les préfixes d'adresse et les sous-réseaux du VPC {{site.data.keyword.cloud}} :

* Les VPC sont déployés et liés à une région.
* Dans cette région, les VPC peuvent couvrir plusieurs zones.
* Les préfixes d'adresse permettent la communication entre les zones que couvre un VPC. Chaque VPC reçoit un préfixe d'adresse par défaut pour chaque zone qu'il couvre.
* Les sous-réseaux sont créés dans le cadre d'un préfixe d'adresse, ce qui signifie qu'un sous-réseau doit être entièrement contenu dans un seul préfixe d'adresse existant.
* Si vous utilisez une plage d'adresses IP en dehors des plages définies par [RFC 1918](https://tools.ietf.org/html/rfc1918) (`10.0.0.0/8`, `172.16.0.0/12` ou `192.168.0.0/16`) pour un sous-réseau, il est possible que les instances associées à ce sous-réseau soient incapables de joindre une partie de l'Internet public.

## VPC et régions IBM Cloud
{: #ibm-cloud-vpc-and-regions}

Un VPC {{site.data.keyword.cloud}} est déployé dans une région, mais il peut couvrir plusieurs zones. Chaque région contient plusieurs zones, qui représentent des domaines d'erreur indépendants.

## VPC IBM Cloud et préfixes d'adresse
{: #ibm-cloud-vpc-and-address-prefixes}

Les préfixes d'adresse permettent la communication entre les instances du VPC {{site.data.keyword.cloud_notm}} dans différentes zones. Ils fournissent des informations de routage, dont le _routeur implicite_ a besoin pour envoyer des données à la zone dans laquelle l'instance de destination est située. Chaque sous-réseau doit être contenu par un préfixe d'adresse. Le VPC {{site.data.keyword.cloud_notm}} ne prend pas en charge le concept de sous-réseaux "locaux" ou "inaccessibles".

Le _routeur implicite_ est la connectivité réseau inhérente entre tous les sous-réseaux créés dans un VPC.
{: note}

Chaque VPC {{site.data.keyword.cloud_notm}} peut avoir jusqu'à cinq préfixes d'adresse pour chaque zone. Pour vous aider à démarrer, le VPC {{site.data.keyword.cloud_notm}} définit un préfixe d'adresse par défaut pour chaque zone (voir le tableau ci-dessous), cependant, il est recommandé de concevoir un plan d'adressage de VPC avant d'en déployer un.

### Préfixes d'adresse par défaut du VPC
{: #default-vpc-address-prefixes}

Lorsqu'un VPC est créé, les préfixes d'adresse par défaut sont attribués comme suit, en fonction de la région et de la zone.

Les [VPC Classic Access](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#classic-access-default-address-prefixes) ont un ensemble différent de préfixes d'adresse par défaut.
{: important}

Zone         | Préfixe d'adresse
---------------|---------------
`us-south-1`   | `10.240.0.0/18`
`us-south-2`   | `10.240.64.0/18`
`us-south-3`   | `10.240.128.0/18`
`eu-de-1`      | `10.243.0.0/18`
`eu-de-2`      | `10.243.64.0/18`
`eu-de-3`      | `10.243.128.0/18`
`jp-tok-1`     | `10.244.0.0/18`
`jp-tok-2`     | `10.244.64.0/18`
`jp-tok-3`     | `10.244.128.0/18`

Différents préfixes par défaut seront assignés aux nouvelles zones ou régions.

### Préfixes d'adresse et interface utilisateur de la console IBM Cloud
{: #address-prefixes-and-the-ibm-cloud-console-ui}

Lorsque vous créez un VPC à l'aide de l'interface utilisateur de la console IBM Cloud, le système sélectionne automatiquement votre préfixe d'adresse et vous oblige à créer un sous-réseau avec ce préfixe par défaut. Si ce schéma d'adresse ne correspond pas à vos besoins, vous pouvez personnaliser les préfixes d'adresse après avoir créé le VPC. Vous pouvez ensuite créer des sous-réseaux dans vos préfixes d'adresses personnalisés et supprimer le sous-réseau que vous avez créé avec le préfixe par défaut.

Cette solution de contournement est nécessaire pour utiliser BYOIP via l'interface utilisateur de la console IBM Cloud.
{:note}

## VPC et sous-réseaux IBM Cloud
{: #ibm-cloud-vpc-and-subnets}

Vous pouvez diviser un {{site.data.keyword.cloud_notm}} VPC en sous-réseaux. Tous les sous-réseaux compris dans un VPC {{site.data.keyword.cloud_notm}} peuvent se joindre par le biais d'un routage L3 privé via un routeur implicite. Il est inutile de configurer des routeurs ou des routes.

Informations utiles sur les sous-réseaux dans le VPC :

* Un sous-réseau consiste en une plage d'adresses IP que vous spécifiez.
* Un sous-réseau est lié à une zone unique. Il ne peut pas couvrir plusieurs zones ou régions.
* Un sous-réseau peut couvrir la totalité de la zone dans une instance de VPC {{site.data.keyword.cloud_notm}}.
* Vous devez créer votre VPC avant de créer vos sous-réseaux au sein de ce VPC.
* La prise en charge d'IPv6 n'est pas disponible.
* Vous pouvez associer ou dissocier une VSI sur un sous-réseau. (Vous devez ajouter une carte vNIC et sélectionner une bande passante.)
* Chaque sous-réseau doit être contenu dans un préfixe d'adresse qui appartient à la zone dans laquelle ce sous-réseau est lié.

Vous pouvez apporter votre propre plage d'adresses IPv4 publiques (BYOIP) à votre compte de VPC {{site.data.keyword.cloud_notm}}. Lorsque vous utilisez BYOIP, {{site.data.keyword.cloud_notm}} doit configurer ces adresses IPv4 sur des ressources {{site.data.keyword.cloud_notm}} qui enverront les paquets à destination et en provenance des adresses fournies. Par conséquent, si vous utilisez votre propre plage d'adresses IPv4 sur {{site.data.keyword.cloud_notm}}, ces adresses IP peuvent être exposées au personnel de support IBM et à des tiers dans le cadre de votre utilisation de ce service.
{:important}

![Présentation du VPC IBM Cloud](images/vpc-experience.svg "Présentation du VPC IBM Cloud")

### Utilisation de préfixes d'adresse pour les sous-réseaux
{: #using-address-prefixes-for-subnets}

Chaque sous-réseau doit se trouver dans un préfixe d'adresse.
 * Pour votre nouveau sous-réseau, vous pouvez sélectionner la plage d'adresses IP parmi les préfixes d'adresse existants.
 * Si les préfixes d'adresse de la zone ne conviennent pas, vous pouvez modifier l'un des préfixes d'adresse existants ou en ajouter un nouveau pour la zone (à l'aide de l'API, de l'interface CLI ou de l'interface utilisateur).

### Adresses IP disponibles
{: #available-ip-addresses}

Les adresses IP disponibles définies dans **RFC 1918** sont les suivantes :

 * 10.0.0.0 – 10.255.255.255
 * 172.16.0.0 – 172.31.255.255
 * 192.168.0.0 – 192.168.255.255

Si vous utilisez une plage d'adresses IP qui se trouve en dehors des plages autorisées pour un sous-réseau (indiquées dans les sections précédentes), il est possible que les instances associées à ce sous-réseau soient incapables de joindre une partie de l'Internet public.

### Informations additionnelles sur la création d'un sous-réseau
{: #more-about-creating-a-subnet}

Il existe deux façons de spécifier un sous-réseau pour votre VPC :
  * Vous pouvez créer un sous-réseau en indiquant la taille du sous-réseau dont vous avez besoin, tel que le nombre d'adresses prises en charge (par exemple, 1024).
  * Vous pouvez créer un sous-réseau en fournissant une plage CIDR (telle que 10.0.0.8/29)

Par exemple, pour indiquer un bloc 1024 à l'aide de CIDR, si vous spécifiez une plage CIDR plutôt qu'une taille de sous-réseau, le bloc IPv4 `192.168.100.0/22` représente les 1024 adresses IPv4 de `192.168.100.0` à `192.168.103.255`.
{:tip}

