---

copyright:
  years: 2019

lastupdated: "2019-05-20"

keywords: security groups, traffic, firewall, stateful, filtering

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Utilisation de groupes de sécurité
{: #using-security-groups}
[comment]: # (rubrique d'aide associée)

Les groupes de sécurité constituent un moyen pratique d'appliquer des règles qui établissent un filtrage pour chaque interface réseau d'une instance de serveur virtuel (VSI), en fonction de l'adresse IP. Lorsque vous créez une ressource Groupe de sécurité, vous devez la mettre à jour pour créer les masques de trafic réseau souhaités.

Pour obtenir une comparaison des caractéristiques des groupes de sécurité et des listes de contrôle d'accès, consultez le [tableau de comparaison](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-compare-security-groups-and-access-control-lists).

Par défaut, un groupe de sécurité refuse tout le trafic. Lorsque des règles sont ajoutées à un groupe de sécurité, il définit le trafic que ce groupe autorise.

Les règles sont _stateful_ (avec état), ce qui signifie que le trafic inversé en réponse au trafic autorisé est automatiquement autorisé. Ainsi, par exemple, une règle autorisant le trafic TCP entrant sur le port 80 permet également de répliquer le trafic TCP sortant sur le port 80 vers l'hôte d'origine, sans qu'une règle supplémentaire soit nécessaire.

Les groupes de sécurité sont configurés pour un seul VPC. Cette portée implique qu'un groupe de sécurité peut être connecté _uniquement_ aux interfaces réseau des VSI au sein du même VPC.

Lorsqu'une VSI est créée sans qu'aucun groupe de sécurité ne soit spécifié, l'interface réseau principale de la VSI est placée dans le groupe de sécurité _par défaut_ du VPC de cette VSI. Cette édition du VPC {{site.data.keyword.cloud}} comporte un groupe de sécurité défini par défaut qui autorise un certain trafic. Pour plus d'informations, voir [Mise à jour du groupe de sécurité par défaut](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-updating-the-default-security-group).

Vous pouvez configurer des groupes de sécurité à l'aide de l'API REST, de l'interface de ligne de commande ou de l'interface utilisateur :

* [Configuration de groupes de sécurité à l'aide de l'API](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-apis)
* [Configuration de groupes de sécurité à l'aide de l'interface de ligne de commande](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-cli)
* [Configuration de groupes de sécurité à l'aide de l'interface utilisateur](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#configuring-the-security-group-for-the-instance)
