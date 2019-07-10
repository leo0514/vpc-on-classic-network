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

# Dietro le quinte del VPC
{: #vpc-behind-the-curtain}

Questa pagina presenta una figura concettuale più dettagliata di cosa succede "dietro le quinte" rispetto alla rete VPC. Si prevede che i lettori abbiano del background di rete.

## Isolamento della rete
{: #network-isolation}

L'isolamento della rete VPC si verifica su tre livelli:

* **Hypervisor**: le VSI (istanze del server virtuale) vengono isolate dallo stesso hypervisor. Una VSI non può direttamente raggiungere le altre VSI ospitate dallo stesso hypervisor se non sono nello stesso VPC.

* **Rete**: l'isolamento si verifica al livello di rete tramite l'utilizzo dei VNI (**virtual network identifier**). L'ambito di questi identificativi è limitato alla zona locale. Questi VNI vengono aggiunti a tutti i pacchetti di dati che entrano in qualsiasi zona del VPC: entrando dall'hypervisor, quando inviati da una VSI, o entrando nella zona dal cloud, quando inviati dalla funzione di instradamento implicito.

A un pacchetto che lascia una zona viene rimosso il VNI. Quando il pacchetto raggiunge la sua zona di destinazione, entrando tramite la funzione di instradamento implicito, il router implicito aggiunge sempre il VNI corretto per quella zona.
{: note}

* **Router**: la _funzione router implicito_ fornisce l'isolamento per ogni VPC fornendo una VRF (**virtual routing function**) con una VPN con MPLS (multi-protocol label switching) nel backbone cloud. Ogni VRF del VPC ha un identificativo univoco e questo isolamento consente ad ogni VPC di avere accesso alla propria copia dello spazio di indirizzo IPv4. La VPN MPLS consente la federazione di tutti gli edge del cloud: infrastruttura classica, Direct Link e VPC.

## Prefissi di indirizzo
{: #address-prefixes}

I prefissi di indirizzo sono le informazioni di riepilogo utilizzate dalla funzione di instradamento implicito di un VPC per individuare una _VSI di destinazione_, indipendentemente dalla zona di disponibilità in cui è ubicata la VSI di destinazione. La funzione primaria dei prefissi di indirizzo è di ottimizzare l'instradamento sulla VPN MPLS, evitando i casi di instradamento patologici. Tutte le sottoreti create in un VPC devono essere contenute in un prefisso di indirizzo, in modo che tutte le VSI in un VPC siano raggiungibili da tutte le altre VSI nel VPC.

## Flussi di pacchetti di dati e router implicito
{: #data-packet-flows-and-the-implicit-router}

Si verificano sei diversi tipi di flussi di pacchetti di dati VSI in un VPC. In ordine crescente di complessità, questi sono i flussi:

* Intra-subnet, intra-host (stesso l'hypervisor)
* Intra-subnet, inter-host
* Inter-subnet, intra-zone
* Inter-subnet, inter-zone
* Extra-VPC service (per l'accesso IaaS o CSE)
* Extra-VPC Internet (per l'accesso Internet)

Flussi di dati **Intra-subnet, intra-host**: sono i più semplici. I pacchetti fluiscono tra le VSI sull'hypervisor e nessun pacchetto lascia l'hypervisor.

Flussi di dati **Intra-subnet, inter-host**: questi flussi comprendono i pacchetti che escono dall'hypervisor. Ogni pacchetto viene contrassegnato con il VNI (virtualized network identifier) appropriato per garantire l'isolamento dei dati e viene quindi inviato all'hypervisor di destinazione, che ospita la VSI di destinazione. L'hypervisor di destinazione rimuove il VNI e inoltra il pacchetto di dati alla VSI di destinazione.

Flussi di dati **Inter-subnet, intra-zone**: questi flussi comprendono i pacchetti che utilizzano la funzione router implicito del VPC, che connette tutte le sottoreti create nel VPC. Instrada il pacchetto di dati all'hypervisor di destinazione corretto. Se l'hypervisor di destinazione è diverso da quello di origine, il pacchetto di dati viene contrassegnato con il VNI appropriato e inviato all'hypervisor di destinazione. Qui, il VNI viene rimosso e il pacchetto di dati viene instradato alla VSI di destinazione. (Questi ultimi passi sono gli stessi di quelli descritti nel tipo precedente di flusso di dati).

Flussi di dati **Inter-subnet, inter-zone**: per questi flussi, la funzione router implicito rimuove il VNI e inoltra il pacchetto nella VPN MPLS del VPC per il transito nel backbone cloud. Nella zona di destinazione, la funzione router implicito contrassegna il pacchetto di dati con il VNI appropriato. Il pacchetto viene quindi inoltrato all'hypervisor di destinazione, dove il VNI viene nuovamente rimosso (come descritto in precedenza) in modo che il pacchetto di dati possa essere inoltrato alla VSI di destinazione.

Flussi di dati **Extra-vpc service**: i pacchetti destinati ai servizi IaaS o IBM CSE (Cloud Service Endpoint) utilizzano la funzione router implicito del VPC e utilizzano anche una funzione NAT (network address translation). La funzione di traduzione sostituisce l'indirizzo della VSI con uno IPv4, che identifica il VPC per il servizio CSE o IaaS che sta venendo richiesto.

Flussi di dati **Extra-vpc Internet**: i pacchetti destinati a internet sono i più complessi. Oltre a utilizzare la funzione router implicito del VPC, ognuno di questi flussi si basa anche su una delle due funzioni NAT (network address translation) del router implicito:

  * un NAT one-to-many esplicito tramite una funzione gateway pubblico che serve tutte le sottoreti connesse ad esso.
  * NAT one-to-one assegnato a singole VSI.

Dopo la traduzione NAT, il router implicito inoltra questi pacchetti destinati a internet tramite il backbone cloud.

## Accesso classico
{: #classic-access}

La funzione [**Accesso classico**](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc) per il VPC viene eseguita riutilizzando l'identificativo VRF dall'account dell'infrastruttura classica {{site.data.keyword.cloud}} come l'identificativo VRF del VPC. Questa implementazione consente alla funzione router implicito del VPC di unirsi alla stessa VPN MPLS utilizzata dall'account dell'infrastruttura classica. Pertanto, il VPC ha accesso alle risorse classiche e a tutto ciò che è raggiungibile tramite le connessioni Direct Link.
