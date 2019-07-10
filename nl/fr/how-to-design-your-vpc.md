---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: VPC, subnet, address prefixes, design, addressing

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


# Conception d'un plan d'adressage pour un VPC 
{: #vpc-addressing-plan-design}

La première étape dans la conception de votre VPC consiste à concevoir votre plan d'adressage. Un plan d'adressage correctement exécuté a deux objectifs :

* Il répond aux exigences de communication des instances du VPC.
* Il maintient la flexibilité pour la croissance future. 

Ce document donne un exemple de conception d'un plan d'adressage pour une application Web à trois niveaux, dans laquelle chaque niveau est pris en charge par plusieurs zones.

Bien que chaque VPC {{site.data.keyword.cloud}} soit déployé dans une région spécifique, le VPC peut couvrir toutes les zones dans cette région. Le VPC {{site.data.keyword.cloud_notm}} définit un préfixe d'adresse par défaut pour chaque zone. Ces préfixes d'adresse permettent la communication entre les instances du VPC {{site.data.keyword.cloud_notm}} dans différentes zones, en fournissant les informations de routage relatives aux zones dont le [routeur implicite](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary#implicit-router) a besoin.

Les étapes de conception requises sont les mêmes, que l'application s'exécute entièrement dans le cloud ou partiellement.
{: tip}

## Considérations et hypothèses de conception
{: #design-considerations-and-assumptions}

Lors de la conception du plan d'adressage d'une application, il est important de garder les blocs CIDR utilisés pour créer des sous-réseaux dans une seule zone aussi contiguë que possible. Le concepteur permet ainsi de les regrouper en un seul préfixe d'adresse, ce qui laisse de la place pour une croissance future.

Il est également important de prendre en compte le nombre d'adresses disponibles dont un sous-réseau peut avoir besoin pour la mise à l'échelle horizontale. Le tableau suivant répertorie le nombre d'adresses disponibles dans un sous-réseau en fonction de la taille de bloc CIDR spécifiée :

| Taille de bloc CIDR | Adresses disponibles |
| --------------- | ------------------- |
|      /22        |        1019         |
|      /23        |         507         |
|      /24        |         251         |
|      /25        |         123         |
|      /26        |          59         |
|      /27        |          27         |
|      /28        |          11         |

Sur la base de ces deux considérations, voici les hypothèses que nous formulons pour cet exemple :

* Pour cet exemple, les plages CIDR du bloc 172.16.0.0.0/12 des adresses RFC 1918 seront utilisées pour tous les sous-réseaux.
* Nous supposons que la couche de présentation de l'application est une plaque fine au-dessus d'une API REST. Par conséquent, la mise à l'échelle horizontale affecte davantage le niveau intermédiaire que le niveau de présentation.

## Détermination de la taille du sous-réseau de chaque niveau
{: #determine-each-tier-s-subnet-size}

L'étape suivante consiste à déterminer la taille du sous-réseau de chaque niveau (en termes d'adresses disponibles). Chaque niveau de l'application a une présence dans chaque zone, et chaque zone nécessite donc trois sous-réseaux.

Voici les considérations que nous utilisons lors de la planification de la taille du sous-réseau de chaque niveau :

* Le niveau de base de données (l'arrière-plan) est le moins susceptible d'avoir besoin d'une mise à l'échelle dynamique, de sorte que ces sous-réseaux peuvent être les plus petits. C'est-à-dire que ces sous-réseaux peuvent contenir le plus petit nombre d'adresses disponibles. 
    * _Cet exemple utilise un bloc CIDR `/27`, qui permet d'avoir 27 adresses dans ce niveau._
* Le niveau intermédiaire est le plus susceptible d'avoir besoin d'une mise à l'échelle dynamique, de sorte que ces sous-réseaux seront les plus grands. C'est-à-dire qu'ils doivent contenir le plus grand nombre d'adresses disponibles. 
    * _Cet exemple utilise un bloc CIDR `/25`, qui permet d'avoir 123 adresses dans ce niveau._
* Le niveau frontal se situe entre les deux ; il n'aura pas besoin d'autant d'adresses que le niveau intermédiaire, mais il a besoin de plus d'adresses que le niveau de base de données. 
    * _Cet exemple utilise un bloc CIDR `/26`, qui permet d'avoir 59 adresses dans ce niveau._

## Combinaison des sous-réseaux et sélection des préfixes d'adresse
{: #combining-the-subnets-and-selecting-the-address-prefixes}

Pour sélectionner un préfixe d'adresse acceptable pour chaque zone, vous aurez besoin d'un sous-réseau suffisamment grand pour accueillir les trois sous-réseaux de chaque niveau, tout en laissant de la place pour la mise à l'échelle horizontale et l'expansion future. 

Le préfixe d'adresse `/24` est le plus petit préfixe dans lequel ces trois sous-réseaux peuvent être combinés (27 + 123 + 59). En règle générale, il est recommandé de sélectionner la taille du sous-réseau immédiatement supérieure, et non la plus petite. L'attribution de la taille de sous-réseau supérieure suivante (`/23`) permet une mise à l'échelle horizontale au-delà des limites données précédemment, car elle permet d'ajouter de nouveaux sous-réseaux à chaque couche, à partir du même préfixe d'adresse.

Maintenant que nous avons déterminé la taille correcte du sous-réseau, les préfixes d'adresse réels peuvent être assignés, un pour chaque zone :

|  Zone  | Préfixe d'adresse|
| ------ | --------------- |
| Zone 1 | `172.16.0.0/23` |
| Zone 2 | `172.16.2.0/23` |
| Zone 3 | `172.16.4.0/23` |

Et sur cette base, les 3 sous-réseaux de chaque zone peuvent être attribués :

|  Zone  |  Niveau  |   CIDR de sous-réseau    |
| ------ | -------- | ----------------- |
| Zone 1 | Intermédiaire |  `172.16.0.0/25`  |
| Zone 1 |  Frontal  |  `172.16.1.0/26`  |
| Zone 1 | Base de données | `172.16.1.128/27` |
| Zone 2 | Intermédiaire |  `172.16.2.0/25`  |
| Zone 2 |  Frontal  |  `172.16.3.0/26`  |
| Zone 2 | Base de données | `172.16.3.128/27` |
| Zone 3 | Intermédiaire |  `172.16.4.0/25`  |
| Zone 3 |  Frontal  |  `172.16.5.0/26`  |
| Zone 3 | Base de données | `172.16.5.128/27` |

## Considérations pour l'extension d'une infrastructure existante
{: #considerations-for-extending-an-existing-infrastructure}

Lorsque vous planifiez un VPC qui étend une infrastructure existante, vous pouvez suivre les étapes précédentes, que ce soit pour votre infrastructure sur site, pour un autre VPC ou même pour un autre cloud. Gardez à l'esprit que vous ne devez pas réutiliser les plages d'adresses existantes. En évitant la réutilisation des adresses, vous pourrez tirer un maximum de profit des fonctionnalités du VPC {{site.data.keyword.cloud_notm}}.
