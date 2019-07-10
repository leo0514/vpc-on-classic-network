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

# Esercitazione introduttiva
{: #getting-started}

Per iniziare a utilizzare la rete per {{site.data.keyword.cloud}} Virtual Private Cloud:

1. Crea un VPC (Virtual Private Cloud).
2. Crea una o più sottoreti nel VPC (Virtual Private Cloud) in una o più zone.
3. Crea un gateway pubblico (PGW) su una sottorete se vuoi che le risorse nella tua sottorete abbiano accesso a internet o viceversa.

## Prerequisiti

 * **Autorizzazioni utente**: assicurati che il tuo utente disponga delle autorizzazioni necessarie per creare e gestire le risorse nel tuo VPC. Per un elenco di autorizzazioni necessarie, consulta [Concessione delle autorizzazioni necessarie agli utenti VPC](/docs/vpc-on-classic?topic=vpc-on-classic-managing-user-permissions-for-vpc-resources).

## Utilizza l'IU, la CLI o l'API REST per eseguire il provisioning delle risorse di rete VPC

Il provisioning e la gestione delle risorse di rete VPC possono essere eseguiti tramite l'IU, la CLI o l'API REST.

* Per l'accesso tramite l'interfaccia utente, accedi alla [Console IBM Cloud ![Icona link esterno](../../icons/launch-glyph.svg "Icona link esterno")]( https://{DomainName}/vpc){: new_window}. Segui la [Guida dell'interfaccia utente](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console) se hai bisogno di aiuto.
* Per utilizzare l'interfaccia della riga di comando, attieniti all'esempio [Hello World](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli).
* Per gli utenti più esperti, puoi richiamare direttamente le [API REST](https://{DomainName}/apidocs/vpc-on-classic). Segui l'esercitazione [codice di esempio](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis) per iniziare ad utilizzare le API REST.

## Passi successivi

Dopo che è stato eseguito il provisioning della tua rete VPC (Virtual Private Cloud), esplora altri argomenti.

* [Creazione e gestione delle istanze del server virtuale](/docs/vpc-on-classic?topic=vpc-on-classic-creating-and-managing-virtual-server-instances)
* [Utilizzo dei gruppi di sicurezza](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-cli)
* [Utilizzo degli ACL di rete](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls)
* [Utilizzo della VPN](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-vpn-with-your-vpc)
* [Utilizzo dei programmi di bilanciamento del carico](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-load-balancers-in-ibm-cloud-vpc)
