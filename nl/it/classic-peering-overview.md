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

# Peering classico per VPC
{: #setting-up-classic-peering-for-vpc}

{{site.data.keyword.cloud}} VPC è in grado di eseguire il peering con le tue risorse IBM Cloud classiche. I tuoi account devono soddisfare alcuni requisiti in modo da poter eseguire il peer.

Per ottenere delle istruzioni dettagliate su come creare un VPC con il peering classico abilitato, consulta il nostro [documento Infrastruttura VPC](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#setting-up-access-to-your-classic-infrastructure-from-vpc) che illustra questa procedura con ulteriori dettagli.

## Requisiti
{: #classic-peering-requirements}

1. Il tuo VPC deve essere predisposto per il "peering classico" al momento della creazione, non lo può essere in un momento successivo.

2. L'account collegato a cui stai eseguendo il peering del tuo VPC deve essere abilitato per VRF. Per ulteriori informazioni sul VRF, vedi [questo documento esplicativo](/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud).

3. Devi anche aggiungere un instradamento al tuo VPC abilitato per l'accesso classico da ogni istanza che utilizzi sul lato dell'infrastruttura classica.

## Limitazioni
{: #classic-peering-restrictions}

Per il peering classico è possibile abilitare solo 1 VPC per ogni regione.
