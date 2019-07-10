---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-29"

keywords: security, ACLs, security groups, traffic, subnet, instance, VSI, firewall, encryption

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# Sécurité de votre instance de VPC IBM Cloud
{: #security-in-your-ibm-cloud-vpc}

Vous pouvez sécuriser votre VPC et vos charges de travail en contrôlant le trafic réseau à l'aide de groupes de sécurité et/ou de listes de contrôle d'accès au réseau (ACL). Rappel :

* Les groupes de sécurité contrôlent le trafic au niveau de chaque instance (VSI).
* Les listes de contrôle d'accès contrôlent le trafic au niveau de chaque sous-réseau.

## Présentation de la sécurité
{: #security-overview}

* Le trafic vers et depuis un sous-réseau peut être contrôlé par des listes de contrôle d'accès (ACL).
* Les groupes de sécurité peuvent contrôler le trafic au niveau de la VSI.
* Configurez une passerelle publique pour l'accès du sous-réseau à Internet, protégé par des listes de contrôle d'accès.
* Configurez une adresse IP flottante pour l'accès de la VSI à Internet, protégé par des groupes de sécurité.

![Connectivité et sécurité dans IBM VPC](images/vpc-connectivity-and-security.svg "Connectivité et sécurité dans IBM VPC")

## Définitions
{: #definitions}

Ce [glossaire](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary) fournit des définitions et des descriptions des ACL et des SG, ainsi que des actions que vous pouvez effectuer avec ces outils. La section qui suit décrit les fonctionnalités de base des listes ACL et des groupes de sécurité, ainsi que la manière dont VPC prend en charge le chiffrement de bout en bout.

### Liste de contrôle d'accès
{: #access-control-list}

Une ** liste de contrôle d'accès (ACL)** peut gérer (c'est-à-dire autoriser ou interdire) le trafic entrant et sortant d'un sous-réseau. Une ACL est sans état, ce qui signifie que les règles entrantes et sortantes doivent être spécifiées séparément et explicitement. Chaque ACL est composée de règles, basées sur une *adresse IP source*, un *port source*, une *adresse IP de destination*, un *port de destination* et un *protocole*.

Dans {{site.data.keyword.cloud}} VPC, chaque sous-réseau est créé avec une ACL par défaut, qui autorise le trafic entrant et sortant, mais les clients peuvent créer des ACL personnalisées. Une ACL est connectée à un sous-réseau à la fois, mais une ACL peut être connectée à plusieurs sous-réseaux. Pour plus d'informations sur l'utilisation des ACL, consultez notre [guide sur les ACL](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls).

### Groupe de sécurité
{: #security-group}

Un **groupe de sécurité** agit comme un pare-feu virtuel contrôlant le trafic d'un ou de plusieurs serveurs (VSI). Un groupe de sécurité est un ensemble de règles qui spécifient si le trafic doit être autorisé pour une VSI associée.

Lorsqu'un client crée une VSI, il peut associer un ou plusieurs groupes de sécurité à cette VSI. Avec les autorisations appropriées, les clients peuvent modifier les règles de groupe de sécurité à l'aide de la console IBM, de la CLI ou de l'API.

Pour plus d'informations sur la création d'une VSI utilisant des groupes de sécurité et sur les fonctionnalités des groupes de sécurité, reportez-vous à notre [guide sur les groupes de sécurité](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-using-security-groups).

### Chiffrement de bout en bout
{: #end-to-end-encryption}

Bien qu'IBM Cloud VPC ne fournisse pas de chiffrement de bout en bout, il le permet. Par exemple, s'il existe un noeud final sécurisé sur un serveur virtuel (par exemple, un serveur HTTPS sur le port 443), vous pouvez associer une adresse IP flottante à ce serveur pour chiffrer votre connexion de bout en bout, du client au serveur sur le port 443.  Rien dans le chemin ne force le déchiffrement.

Notez cependant que si vous utilisez un protocole non sécurisé tel que HTTP sur le port 80, vos données sont en clair de bout en bout.
