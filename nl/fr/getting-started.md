---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-06-06"

keywords: provisioning, resources, permissions

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:important: .important}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Tutoriel d'initiation
{: #getting-started}

Pour vous initier à la mise en réseau dans {{site.data.keyword.cloud}} Virtual Private Cloud :

1. Créez un cloud privé virtuel.
2. Créez un ou plusieurs sous-réseaux dans le cloud privé virtuel dans une ou plusieurs zones.
3. Créez une passerelle publique sur un sous-réseau si vous souhaitez que les ressources de votre sous-réseau aient accès à Internet ou inversement.

## Prérequis

 * **Droits utilisateur** : Assurez-vous que votre utilisateur dispose des autorisations suffisantes pour créer et gérer des ressources dans votre VPC. Pour obtenir la liste des droits requis, voir [Granting permissions needed for VPC users](/docs/vpc-on-classic?topic=vpc-on-classic-managing-user-permissions-for-vpc-resources).

## Utilisation de l'interface utilisateur, de l'interface de ligne de commande ou de l'API REST pour la mise à disposition des ressources réseau du VPC

La mise à disposition et la gestion des ressources réseau du VPC peut s'effectuer via l'interface utilisateur, l'interface de ligne de commande ou l'API REST.

* Pour un accès via l'interface utilisateur, connectez-vous à la [console IBM Cloud ![Icône de lien externe](../../icons/launch-glyph.svg "Icône de lien externe")]( https://{DomainName}/vpc){: new_window}. Suivez le [guide de l'interface utilisateur](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console) si vous avez besoin d'aide.
* Pour utiliser l'interface de ligne de commande, suivez l'exemple [Hello World](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli).
* Les utilisateurs plus expérimentés peuvent appeler directement les [API REST](https://{DomainName}/apidocs/vpc-on-classic). Suivez le tutoriel [Exemple de code](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis) pour vous initier aux API REST.

## Etapes suivantes

Une fois votre mise en réseau de cloud privé virtuel à disposition, explorez d'autres fonctions.

* [Création et gestion d'instances de serveur virtuel](/docs/vpc-on-classic?topic=vpc-on-classic-creating-and-managing-virtual-server-instances)
* [Utilisation de groupes de sécurité](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-cli)
* [Utilisation de listes de contrôle d'accès au réseau](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls)
* [Utilisation du VPN](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-vpn-with-your-vpc)
* [Utilisation des équilibreurs de charge](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-load-balancers-in-ibm-cloud-vpc)
