---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, Cisco, ASAv, connection, secure, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc} 
{:new_window: target="_blank"} 
{:DomainName: data-hd-keyref="DomainName"} 
{:note: .note} 
{:important: .important} 
{:deprecated: .deprecated} 
{:generic: data-hd-programlang="generic"}

# Création d'une connexion sécurisée avec un homologue Cisco ASAv distant
{: #creating-a-secure-connection-with-a-remote-cisco-asav-peer}

Ce document est basé sur Cisco ASAv, Cisco Adaptive Security Appliance Software Version 9.10(1).

Les exemples qui suivent ignorent les étapes prérequises de l'utilisation de l'API ou de l'interface de ligne de commande {{site.data.keyword.cloud}} pour créer des clouds privés virtuels. Pour plus d'informations, voir [Initiation](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) et [Configuration de VPC avec des API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Exemple d'étapes
{: #cisco-example-steps}

La topologie de connexion à l'homologue Cisco ASAv distant est similaire à la création d'une connexion VPN entre deux clouds privés virtuels {{site.data.keyword.cloud}}. Cependant, un côté est remplacé par l'unité Cisco ASAv.

![entrer la description de l'image ici](./images/vpc-vpn-asav-figure.png)

### Pour créer une connexion sécurisée avec l'homologue Cisco ASAv distant
{: #to-create-a-secure-connection-with-the-remote-cisco-asav-peer}

La première étape de la configuration de votre unité Cisco ASAv pour une utilisation avec le réseau privé virtuel IBM VPC consiste à vérifier que les conditions préalables suivantes ont été définies :

* Cisco ASAv est en ligne et fonctionnel avec une licence appropriée
* Un mot de passe pour Cisco ASAv est activé
* Il existe au moins une interface interne fonctionnelle configurée et vérifiée
* Il existe au moins une interface externe fonctionnelle configurée et vérifiée

Lorsqu'une unité Cisco ASAv reçoit une demande de connexion d'un homologue VPN distant, elle utilise les paramètres IPsec Phase 1 pour établir une connexion sécurisée et authentifier cet homologue VPN. Ensuite, si la stratégie de sécurité autorise la connexion, l'unité Cisco ASAv établit le tunnel à l'aide des paramètres IPsec Phase 1 et applique la stratégie de sécurité IPsec. Les services de gestion de clés, d'authentification et de sécurité sont négociés de manière dynamique via le protocole IKE.

**Pour prendre en charge ces fonctions, les étapes de configuration générales suivantes doivent être effectuées par l'unité Cisco ASAv :**

* Définissez les paramètres de phase 1 requis par l'unité Cisco ASAv pour authentifier l'homologue distant et établir une connexion sécurisée.
* Définissez les paramètres de phase 2 nécessaires à l'unité Cisco ASAv pour créer un tunnel VPN avec l'homologue distant.

Créez un objet de proposition Internet Key Exchange (IKE) version 2. Les objets de proposition IKEv2 contiennent les paramètres requis pour créer des propositions IKEv2 lors de la définition de stratégies d'accès distant et de VPN de site à site. IKE est un protocole de gestion de clés qui facilite la gestion des communications basées sur IPsec. Il est utilisé pour authentifier les homologues IPsec, négocier et distribuer les clés de chiffrement IPsec et établir automatiquement des associations de sécurité (SA) IPsec. 

```
group-policy GroupPolicy_161.156.80.10 internal
group-policy GroupPolicy_161.156.80.10 attributes
 vpn-tunnel-protocol ikev1 ikev2 
tunnel-group 161.156.80.10 type ipsec-l2l
tunnel-group 161.156.80.10 general-attributes
 default-group-policy GroupPolicy_161.156.80.10
tunnel-group 161.156.80.10 ipsec-attributes
 ikev1 pre-shared-key <key value>
 ikev2 remote-authentication pre-shared-key <key value>
 ikev2 local-authentication pre-shared-key <key value>
```

Créez une configuration de stratégie IKEv2 pour la connexion IPsec. Le bloc de stratégie IKEv2 définit les paramètres de l'échange IKE. Dans ce bloc, les paramètres suivants sont définis :
* Algorithme de chiffrement - défini sur AES-256 pour cet exemple
* Algorithme d'intégrité - défini sur SHA256 pour cet exemple
* Groupe Diffie-Hellman - IPsec utilise l'algorithme Diffie-Hellman pour générer la clé de chiffrement initiale entre les homologues. Dans cet exemple, il est défini sur le groupe 14
* Fonction pseudo-aléatoire (PRF) - IKEv2 nécessite une méthode distincte utilisée comme algorithme pour dériver le matériel de clé et les opérations de hachage requis pour le chiffrement du tunnel IKEv2. Cette fonction est appelée pseudo-aléatoire et est définie sur SHA
* SA Lifetime - Permet de définir la durée de vie des associations de sécurité (après quoi une reconnexion a lieu). Défini sur 36 000 secondes.
* Type d'opération - Conservez-le comme valeur par défaut, bidirectionnel. (Il n'est pas explicite dans l'affichage "show running".)

Comme illustré dans l'exemple de code suivant, cet exemple de stratégie utilise AES-256 pour chiffrer le canal sécurisé. L'algorithme de hachage SHA512 est utilisé pour valider l'identité de l'homologue distant et le groupe Diffie Hellman 14 est utilisé pour la génération de clé. Le groupe 14 utilise des blocs de chiffrement de 2048 bits. Enfin, la durée de vie de l'association de sécurité est définie sur 36 000 secondes.

```
crypto ikev2 policy 100
encryption aes-256
integrity sha-1
group 14
prf sha
lifetime seconds 36000
```

* Définissez la liste d'accès et la carte de chiffrement pour le VPN :

```
access-list outside_cryptomap_1 extended permit ip object NETWORK_OBJ_192.168.236.0_24 object vpc 
crypto map outside_map 1 match address outside_cryptomap_1
crypto map outside_map 1 set peer 161.156.80.10 
crypto map outside_map 1 set ikev1 transform-set ESP-AES-128-SHA ESP-AES-128-MD5 ESP-AES-192-SHA ESP-AES-192-MD5 ESP-AES-256-SHA ESP-AES-256-MD5 ESP-3DES-SHA ESP-3DES-MD5 ESP-DES-SHA ESP-DES-MD5
crypto map outside_map 1 set ikev2 ipsec-proposal AES256 AES192 AES 3DES DES
crypto map outside_map interface outside
nat (any,outside) source static NETWORK_OBJ_192.168.236.0_24 NETWORK_OBJ_192.168.236.0_24 destination static vpc vpc no-proxy-arp route-lookup
```

## Pour créer une connexion sécurisée avec l'instance locale d'IBM Cloud VPC
{: #to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

Pour créer une connexion sécurisée, créez une connexion VPN au sein de votre VPC, similaire à l'exemple VPC 2.

* Créez une passerelle VPN sur votre sous-réseau VPC avec une connexion VPN entre le VPC et l'unité Cisco ASAv, en définissant `local_cidrs` à la valeur de sous-réseau sur le VPC et `peer_cidrs` à la valeur de sous-réseau sur l'unité Cisco ASAv.

REMARQUE : Le statut de la passerelle apparaît en tant qu'`en attente` lors de la création de la passerelle VPN et devient `disponible` une fois la création terminée. La création peut prendre du temps. 
{:note}


![entrer la description de l'image ici](./images/vpc-vpn-asav-connection.png)

### Vérification du statut de la connexion sécurisée
{: #cisco-check-the-status-of-the-secure-connection}

Vous pouvez vérifier le statut de votre connexion via la console IBM Cloud. En outre, vous pouvez tenter de créer un fichier `ping` d'un site à l'autre à l'aide des VSI.

![entrer la description de l'image ici](./images/vpc-vpn-asav-status.png)
