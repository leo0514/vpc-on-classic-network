---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-29"

keywords: VRF, router, hypervisor, address prefixes, classic access, implicit router, packet flows, NAT, data flows

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

# Coulisses de VPC
{: #vpc-behind-the-curtain}

Cette page présente une image conceptuelle plus détaillée des coulisses de la mise en réseau VPC. Les lecteurs sont supposés avoir certaines connaissances relatives à la mise en réseau.

## Isolement du réseau
{: #network-isolation}

L'isolement du réseau VPC a lieu à trois niveaux :

* **Hyperviseur** : Les instances de serveur virtuel (VSI) sont isolées par l'hyperviseur lui-même. Une VSI ne peut pas atteindre directement d'autres VSI hébergées par le même hyperviseur, si elles ne se trouvent pas dans le même VPC.

* **Réseau** : L'isolement se produit au niveau du réseau via l'utilisation d'**identificateurs de réseau virtuel** (VNI). Ces identificateurs sont étendus à la zone locale. Ces VNI sont ajoutés à tous les paquets de données entrant dans n'importe quelle zone du VPC, que ce soit depuis l'hyperviseur (lorsqu'ils sont envoyés par un VSI) ou depuis le cloud (lorsqu'ils sont envoyés par la fonction de routage implicite).

Un paquet quittant une zone a le VNI retiré. Lorsque le paquet atteint sa zone de destination, en entrant par la fonction de routage implicite, le routeur implicite ajoute toujours le VNI approprié pour cette zone.
{: note}

* **Routeur** : La _fonction de routeur implicite_ permet d'isoler chaque VPC en fournissant une **fonction de routage virtuel** (VRF) et un VPN avec MPLS (commutation multiprotocole par étiquette) dans le réseau principal du cloud. Chaque VRF du VPC comporte un identificateur unique et cet isolement permet à chaque VPC d'avoir accès à sa propre copie de l'espace d'adresses IPv4. Le VPN MPLS permet de fédérer toutes les structures périphériques du cloud : Infrastructure classique, Direct Link et VPC.

## Préfixes d'adresse
{: #address-prefixes}

Les préfixes d'adresse sont les informations récapitulatives utilisées par la fonction de routage implicite d'un VPC pour localiser une _VSI cible_, quelle que soit la zone de disponibilité dans laquelle se trouve cette dernière. La fonction principale des préfixes d'adresse vise à optimiser le routage via le VPN MPLS, tout en évitant les cas de routage pathologique. Tous les sous-réseaux créés dans un VPC doivent se trouver dans un préfixe d'adresse, afin que toutes les VSI d'un VPC soient accessibles à partir de toutes les autres VSI du VPC.

## Flux de paquet de données et routeur implicite
{: #data-packet-flows-and-the-implicit-router}

Six types de flux de paquet de données VSI différents sont présents dans un VPC. Ils sont répertoriés ici, par ordre croissant de complexité :

* Intra-sous-réseau, intra-hôte (même hyperviseur)
* Intra-sous-réseau, inter-hôte
* Inter-sous-réseau, intra-zone
* Inter-sous-réseau, inter-zone
* Service extra-VPC (pour l'accès CSE ou IaaS)
* Internet extra-VPC (pour l'accès Internet)

Flux de données **intra-sous-réseau, intra-hôte** : ce sont les plus simples. Les paquets passent entre les VSI sur l'hyperviseur, et aucun paquet ne quitte l'hyperviseur.

Flux de données **intra-sous-réseau, inter-hôte** : ces flux impliquent que des paquets quittent l'hyperviseur. Chaque paquet est balisé avec le VNI (identificateur de réseau virtuel) approprié pour garantir l'isolement des données, puis est envoyé à l'hyperviseur cible qui héberge la VSI cible. L'hyperviseur cible retire le VNI et achemine le paquet de données vers la VSI cible.

Flux de données **inter-sous-réseau, intra-zone** : ces flux impliquent des paquets utilisant la fonction de routeur implicite du VPC, qui connecte tous les sous-réseaux créés dans le VPC. Le paquet de données est acheminé vers l'hyperviseur cible approprié. Si l'hyperviseur cible est différent de l'hyperviseur source, le paquet de données est balisé avec le VNI approprié et envoyé à l'hyperviseur cible. Le VNI est alors retiré et le paquet de données est acheminé vers la VSI cible. (Ces dernières étapes sont les mêmes que celles décrites dans le type de flux de données précédent.)

Flux de données **inter-sous-réseau, inter-zone** : pour ces flux, la fonction de routeur implicite supprime le VNI et achemine le paquet dans le VPN MPLS du VPC pour qu'il transite par le réseau principal du cloud. Dans la zone cible, la fonction de routeur implicite balise le paquet de données avec le VNI approprié. Le paquet est ensuite acheminé vers l'hyperviseur cible, où le VNI est à nouveau retiré (comme décrit précédemment) afin que le paquet de données puisse être acheminé vers la VSI cible.

Flux de données **service extra-VPC** : les paquets destinés aux services IaaS ou CSE (noeud final de service IBM Cloud) utilisent la fonction de routeur implicite du VPC, ainsi qu'une fonction de conversion d'adresses réseau (NAT). La fonction de conversion remplace l'adresse de la VSI par une adresse IPv4, qui identifie le VPC sur le service IaaS ou CSE demandé.

Flux de données **Internet extra-VPC** : Les paquets destinés à Internet sont les plus complexes. Outre l'utilisation de la fonction de routeur implicite du VPC, chacun de ces flux s'appuie également sur l'une des deux fonctions de conversion d'adresses réseau (NAT) du routeur implicite :

  * une fonction explicite NAT multi-destinataire via une passerelle publique qui sert tous les sous-réseaux qui lui sont connectés.
  * une fonction NAT individuelle affectée à des VSI individuelles.

Après la conversion NAT, le routeur implicite achemine ces paquets destinés à Internet vers Internet, à l'aide du réseau principal du cloud.

## Accès classique
{: #classic-access}

La fonctionnalité [**Accès classique**](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc) pour le VPC s'exécute en réutilisant l'identificateur VRF du compte d'infrastructure classique {{site.data.keyword.cloud}} comme identificateur VRF pour le VPC. Cette implémentation permet à la fonction de routeur implicite du VPC de rejoindre le VPN MPLS qui est utilisé par le compte d'infrastructure classique. Par conséquent, le VPC a accès aux ressources classiques et à tout autre élément accessible par le biais de connexions Direct Link existantes.
