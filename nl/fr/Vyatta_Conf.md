---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, Vyatta, connection, secure, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Création d'une connexion sécurisée avec un homologue Vyatta distant
{: #creating-a-secure-connection-with-a-remote-vyatta-peer}

Ce document est basé sur la version de Vyatta : AT&T vRouter 5600 1801d.

Les exemples qui suivent ignorent les étapes prérequises de l'utilisation de l'API ou de l'interface de ligne de commande {{site.data.keyword.cloud}} pour créer des clouds privés virtuels. Pour plus d'informations, voir [Initiation](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) et [Configuration de VPC avec des API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Exemple d'étapes
{: #vyatta-example-steps}

La topologie de connexion à l'homologue Vyatta distant est similaire à la création d'une connexion VPN entre deux VPC {{site.data.keyword.cloud_notm}}. Cependant, un côté est remplacé par l'unité Vyatta.

![entrer la description de l'image ici](images/vpc-vpn-vy-figure.png)

### Pour créer une connexion sécurisée avec l'homologue Vyatta distant
{: #vyatta-to-create-a-secure-connection-with-the-remote-vyatta-peer}

Lorsqu'un homologue VPN reçoit une demande de connexion d'un homologue VPN distant, il utilise les paramètres IPsec Phase 1 pour établir une connexion sécurisée et authentifier cet homologue VPN. Ensuite, si la stratégie de sécurité autorise la connexion, l'unité Vyatta établit le tunnel à l'aide des paramètres IPsec Phase 2 et applique la stratégie de sécurité IPsec. Les services de gestion de clés, d'authentification et de sécurité sont négociés de manière dynamique via le protocole IKE.

Vous pouvez accéder à la ligne de commande Vyatta pour configurer le tunnel IPsec ou télécharger un fichier `*.vcli` pour charger votre configuration.

**Pour prendre en charge ces fonctions, les étapes de configuration générales suivantes doivent être effectuées par l'unité Vyatta :**

* Définissez les paramètres de phase 1 requis par l'unité Vyatta pour authentifier l'homologue distant et établir une connexion sécurisée.

* Définissez les paramètres de phase 2 nécessaires à l'unité Vyatta pour créer un tunnel VPN avec l'homologue distant.

Pour vous connecter à la fonctionnalité VPN d'IBM Cloud VPC, nous vous recommandons la configuration suivante :

1. Sélectionnez `IKEv2` dans l'authentification
2. Activez `DH-group 2` dans la proposition de phase 1
3. Définissez `lifetime = 36000` dans la proposition de phase 1
4. Désactivez PFS dans la proposition de phase 2
5. Définissez `lifetime = 10800` dans la proposition de phase 2
6. Entrez les informations de vos homologues et des sous-réseaux dans la phase 2

```
vim vyatta_temp/create_vpn.vcli
#!/bin/vcli -f
configure

set security vpn ipsec ike-group 169.61.161.151_test_ike
set security vpn ipsec ike-group 169.61.161.151_test_ike dead-peer-detection timeout 120
set security vpn ipsec ike-group 169.61.161.151_test_ike lifetime 36000
set security vpn ipsec ike-group 169.61.161.151_test_ike ike-version 2

set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 dh-group 2
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 encryption aes256
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 hash sha2_256
set security vpn ipsec esp-group 169.61.161.151_test_ipsec compression disable
set security vpn ipsec esp-group 169.61.161.151_test_ipsec lifetime 10800
set security vpn ipsec esp-group 169.61.161.151_test_ipsec mode tunnel
set security vpn ipsec esp-group 169.61.161.151_test_ipsec pfs disable


set security vpn ipsec esp-group 169.61.161.151_test_ipsec proposal 1 encryption aes256
set security vpn ipsec esp-group 169.61.161.151_test_ipsec proposal 1 hash sha2_256
set security vpn ipsec site-to-site peer 169.61.161.151 authentication mode pre-shared-secret
set security vpn ipsec site-to-site peer 169.61.161.151 authentication pre-shared-secret ******
set security vpn ipsec site-to-site peer 169.61.161.151 ike-group 169.61.161.151_test_ike
set security vpn ipsec site-to-site peer 169.61.161.151 default-esp-group 169.61.161.151_test_ipsec
set security vpn ipsec site-to-site peer 169.61.161.151 description "automation test"
set security vpn ipsec site-to-site peer 169.61.161.151 local-address 169.45.74.119
set security vpn ipsec site-to-site peer 169.61.161.151 connection-type initiate


set security vpn ipsec site-to-site peer 169.61.161.151 tunnel 1 local prefix 192.168.200.0/24
set security vpn ipsec site-to-site peer 169.61.161.151 tunnel 1 remote prefix 192.168.17.0/28

commit
```
{: screen}

Ensuite, vous pouvez télécharger ce fichier `*.vcli` sur le Vyatta à l'aide de SCP, pour appliquer la configuration.

`scp example.vcli <vyatta_username>@<vyatta_ip>`

### Pour créer une connexion sécurisée avec l'instance locale d'IBM Cloud VPC
{: #vyatta-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

 Pour créer une connexion sécurisée, créez une connexion VPN au sein de votre VPC, similaire à l'exemple VPC 2.

* Créez une passerelle VPN sur votre sous-réseau VPC avec une connexion VPN entre le VPC et l'unité Vyatta, en définissant `local_cidrs` à la valeur de sous-réseau sur le VPC et `peer_cidrs` à la valeur de sous-réseau sur l'unité Vyatta.

REMARQUE : Le statut de la passerelle apparaît en tant qu'`en attente` lors de la création de la passerelle VPN et devient `disponible` une fois la création terminée. La création peut prendre du temps.
{: note}

![entrer la description de l'image ici](images/vpc-vpn-vy-connection.png)

### Vérification du statut de la connexion sécurisée
{: #vyatta-check-the-status-of-the-secure-connection}

Vous pouvez vérifier le statut de votre connexion à l'aide de la console {{site.data.keyword.cloud_notm}}. En outre, vous pouvez tenter de créer un fichier `ping` d'un site à l'autre à l'aide des VSI.

![entrer la description de l'image ici](images/vpc-vpn-vy-status.png)
