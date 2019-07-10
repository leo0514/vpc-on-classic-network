---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, StrongSwan, connection, secure, Linux, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{: note: .note}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Création d'une connexion sécurisée avec un homologue StrongSwan distant
{: #creating-a-secure-connection-with-a-remote-strongswan-peer}

Ce document est basé sur Strongswan, version Linux StrongSwan U5.3.5/K4.4.0-133-generic.

Les exemples qui suivent ignorent les étapes prérequises de l'utilisation de l'API ou de l'interface de ligne de commande {{site.data.keyword.cloud}} pour créer des clouds privés virtuels. Pour plus d'informations, voir [Initiation](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) et [Configuration de VPC avec des API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Exemple d'étapes
{: #strongswan-example-steps}

La topologie de connexion à l'homologue StrongSwan distant est similaire à la création d'une connexion VPN entre deux VPC. Cependant, l'un des côtés de la connexion est remplacé par l'unité StrongSwan.

![entrer la description de l'image ici](./images/vpc-vpn-sw-figure.png)

### Pour créer une connexion sécurisée avec un homologue StrongSwan distant
{: #to-create-a-secure-connection-with-a-remote-strongswan-peer}

Accédez à **/etc** et créez votre nouveau fichier de configuration de tunnel personnalisé avec un nom similaire à **ipsec.abc.conf**. Modifiez **/etc/ipsec.conf** pour inclure **ipsec.abc.conf** en ajoutant cette ligne :

    include /etc/ipsec.abc.conf

Lorsqu'un homologue VPN reçoit une demande de connexion d'un homologue VPN distant, il utilise les paramètres IPsec Phase 1 pour établir une connexion sécurisée et authentifier cet homologue VPN. Ensuite, si la stratégie de sécurité autorise la connexion, l'unité StrongSwan établit le tunnel à l'aide des paramètres IPsec Phase 2 et applique la stratégie de sécurité IPsec. Les services de gestion de clés, d'authentification et de sécurité sont négociés de manière dynamique via le protocole IKE.

**Pour prendre en charge ces fonctions, les étapes de configuration générales suivantes doivent être effectuées par l'unité StrongSwan :**

* Définissez les paramètres de phase 1 requis par StrongSwan pour authentifier l'homologue distant et établir une connexion sécurisée.

* Définissez les paramètres de phase 2 nécessaires à l'unité StrongSwan pour créer un tunnel VPN avec l'homologue distant.
Pour vous connecter à la fonctionnalité VPN d'IBM Cloud VPC, nous vous recommandons la configuration suivante :

1. Sélectionnez `IKEv2` dans l'authentification
2. Activez `DH-group 2` dans la proposition de phase 1
3. Définissez `lifetime = 36000` dans la proposition de phase 1
4. Désactivez PFS dans la proposition de phase 2
5. Définissez `lifetime = 10800` dans la proposition de phase 2
6. Saisissez les informations de l'homologue et du sous-réseau dans la proposition de phase 2

```
    vim /etc/ipsec.abc.conf
    conn all
           type=tunnel
           auto=route
           #aggressive=no
           esp=aes256-sha256!
           ike=aes128-sha1-modp1024!
           left=169.45.74.119
           leftsubnet=10.160.26.64/26
           rightsubnet=192.168.17.0/28
           right=169.61.181.116
           leftauth=psk
           rightauth=psk
           leftid="169.45.74.119"
           keyexchange=ikev2
           rightid="169.61.181.116"
           lifetime=10800s
           ikelifetime=36000s
           dpddelay=30s
           dpdaction=restart
           dpdtimeout=120s
```
{: screen}

Définissez la clé prépartagée dans `/etc/ipsec.secrets`

```
vim ipsec.secrets
# This file holds shared secrets or RSA private keys for authentication.

169.45.74.119 169.61.181.116 : PSK "******"

```
{: screen}

Une fois l'exécution du fichier de configuration terminée, redémarrez l'unité StrongSwan.

```
 ipsec restart
```
{: screen}

### Pour créer une connexion sécurisée avec l'instance locale d'IBM Cloud VPC
{: #strongswan-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

* Pour créer une connexion sécurisée, créez une connexion VPN au sein de votre VPC, similaire à l'exemple VPC 2.

* Créez une passerelle VPN sur votre sous-réseau VPC avec une connexion VPN entre le VPC et l'unité StrongSwan, en définissant `local_cidrs` sur la valeur de sous-réseau sur le VPC et `peer_cidrs` sur la valeur de sous-réseau sur l'unité StrongSwan.

REMARQUE : Le statut de la passerelle apparaît en tant qu'`en attente` lors de la création de la passerelle VPN et devient `disponible` une fois la création terminée. La création peut prendre du temps.
{: note}

![vpc-vpn-sw-connection](./images/vpc-vpn-sw-connection.png)

### Vérification du statut d'une connexion sécurisée
{: #strongswan-check-the-status-for-a-secure-connection}

Vous pouvez vérifier le statut de votre connexion via la console IBM Cloud. En outre, vous pouvez tenter de créer un fichier `ping` d'un site à l'autre à l'aide des VSI.

![vpc-vpn-sw-status.png](./images/vpc-vpn-sw-status.png)
