---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-29"

keywords: peering, classic, infrastructure, VRF, resources

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Appairage classique pour VPC
{: #setting-up-classic-peering-for-vpc}

Un VPC {{site.data.keyword.cloud}} peut effectuer un appairage avec vos ressources IBM Cloud classiques. Vos comptes doivent répondre à certaines conditions requises pour effectuer l'appairage.

Pour obtenir des instructions pas à pas sur la création d'un VPC avec l'appairage classique activé, consultez notre [document Infrastructure VPC](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#setting-up-access-to-your-classic-infrastructure-from-vpc) qui couvre cette procédure de façon plus détaillée.

## Configuration requise
{: #classic-peering-requirements}

1. Votre VPC doit être conçu pour un "appairage classique" lors de sa création, car cette opération ne peut pas être effectuée ultérieurement.

2. Le compte lié avec lequel vous effectuez l'appairage de votre VPC doit être activé pour VRF. Pour plus d'informations sur VRF, veuillez consulter [ce document explicatif](/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud).

3. Vous devez également ajouter un itinéraire de retour à votre VPC Classic dans chaque instance que vous utilisez du côté de l'infrastructure Classic.

## Restrictions
{: #classic-peering-restrictions}

Un seul VPC par région peut être activé pour l'appairage classique.
