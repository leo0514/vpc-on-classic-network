---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: IPv4, ranges, subnets, CIDR, 1918

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


# Sélection des plages d'adresses IP pour votre VPC
{: #choosing-ip-ranges-for-your-vpc}

Utilisez la notation CIDR telle que :

* `<IPv4 address>/number` (exemple d'adresse VPC : 10.10.0.0/16).

Réservez les 16 derniers bits (65 536 adresses) de l'IPv4 en tant que zéros (0) afin de pouvoir les utiliser pour différentes adresses IP de sous-réseau au sein du même VPC {{site.data.keyword.cloud}}. Exemple d'adresse IP de sous-réseau : 10.10.1.0/24.

La notation CIDR est définie dans [RFC 1518](https://tools.ietf.org/html/rfc1518) et [RFC 1519](https://tools.ietf.org/html/rfc1519).
{: note}

Si vous utilisez une plage d'adresses IP en dehors des plages définies par [RFC 1918](https://tools.ietf.org/html/rfc1918) (`10.0.0.0/8`, `172.16.0.0/12` ou `192.168.0.0/16`) pour un sous-réseau, il est possible que les instances associées à ce sous-réseau soient incapables de joindre une partie de l'Internet public.

Au cas où vous ne seriez pas familiarisé avec la notation CIDR, n'oubliez pas que plus le nombre après la barre oblique est petit, **plus** vous allouez d'adresses IP, car le nombre situé après la barre oblique représente le nombre de bits 1 de tête dans le masque de préfixe du sous-réseau.

Le tableau suivant répertorie le nombre d'adresses disponibles dans un sous-réseau pour les différentes tailles de bloc CIDR :

| Taille de bloc CIDR | Adresses disponibles |
| --------------- | ------------------- |
|      /22        |        1019         |
|      /23        |         507         |
|      /24        |         251         |
|      /25        |         123         |
|      /26        |          59         |
|      /27        |          27         |
|      /28        |          11         |

Pour consulter plus d'informations, vous trouverez d'excellents articles en ligne sur _Classless Inter-Domain Routing_ (CIDR).
