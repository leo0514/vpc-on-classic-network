---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-07"

keywords: vpc, address prefix, region, subnet, zone, reserved, IP, ranges, deleting, creating, CIDR

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:download: .download}

# Informazioni sugli intervalli di indirizzi IP, prefissi di indirizzo, regioni e sottoreti
{: #working-with-ip-address-ranges-address-prefixes-regions-and-subnets}
[comment]: # (argomento della guida collegato)

Questo documento descrive le relazioni tra le regioni, i prefissi di indirizzo e le sottoreti per {{site.data.keyword.cloud}} VPC:

* I VPC vengono distribuiti e associati a una regione.
* All'interno di quella sola regione, i VPC possono estendersi su più zone.
* I prefissi di indirizzo consentono la comunicazione tra le zone in cui si estende un VPC. Ogni VPC ottiene un prefisso di indirizzo predefinito di scelta rapida per ciascuna zona in cui si estende.
* Le sottoreti vengono create nell'ambito di un prefisso di indirizzo, il che significa che una sottorete deve essere contenuta completamente all'interno di un unico prefisso di indirizzo esistente.
* Se utilizzi un intervallo IP al di fuori degli intervalli definiti da [RFC 1918](https://tools.ietf.org/html/rfc1918) (`10.0.0.0/8`, `172.16.0.0/12` o `192.168.0.0/16`) per una sottorete, le istanze collegate a tale sottorete potrebbero non essere in grado di raggiungere parti dell'Internet pubblico.

## IBM Cloud VPC e regioni
{: #ibm-cloud-vpc-and-regions}

Un {{site.data.keyword.cloud}} VPC viene distribuito in una regione, ma può estendersi su più zone. Ogni regione contiene più zone, che rappresentano dei domini di errore indipendenti.

## IBM Cloud VPC e prefissi di indirizzo
{: #ibm-cloud-vpc-and-address-prefixes}

I prefissi di indirizzo consentono la comunicazione tra le istanze {{site.data.keyword.cloud_notm}} VPC in zone diverse. Forniscono le informazioni di instradamento, richieste dal _router implicito_, per l'invio di dati alla zona in cui si trova l'istanza di destinazione. Ogni sottorete deve essere contenuta all'interno di un prefisso di indirizzo. {{site.data.keyword.cloud_notm}} VPC non supporta il concetto di sottoreti "locali" o "non raggiungibili".

Il _router implicito_ è la connettività di rete intrinseca tra tutte le sottoreti create all'interno di un VPC.
{: note}

Ogni {{site.data.keyword.cloud_notm}} VPC può avere fino a cinque prefissi di indirizzo per ogni zona. Per aiutarti a iniziare, {{site.data.keyword.cloud_notm}} VPC definisce un prefisso di indirizzo predefinito per ciascuna zona (vedi la tabella che segue), tuttavia, come procedura ottimale, dovresti progettare un piano di indirizzamento per VPC prima di distribuirne uno.

### Prefissi di indirizzo predefiniti del VPC
{: #default-vpc-address-prefixes}

Quando viene creato un nuovo VPC, i prefissi di indirizzo predefiniti vengono assegnati come segue, in base alla regione e alla zona.

[I VPC con accesso
classico](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#classic-access-default-address-prefixes) hanno una diversa serie di prefissi di indirizzo predefiniti.
{: important}

Zona         | Prefisso indirizzo
---------------|---------------
`us-south-1`   | `10.240.0.0/18`
`us-south-2`   | `10.240.64.0/18`
`us-south-3`   | `10.240.128.0/18`
`eu-de-1`      | `10.243.0.0/18`
`eu-de-2`      | `10.243.64.0/18`
`eu-de-3`      | `10.243.128.0/18`
`jp-tok-1`     | `10.244.0.0/18`
`jp-tok-2`     | `10.244.64.0/18`
`jp-tok-3`     | `10.244.128.0/18`

Diversi prefissi predefiniti saranno assegnati a nuove zone o regioni.

### Prefissi dell'indirizzo e IU della console IBM Cloud
{: #address-prefixes-and-the-ibm-cloud-console-ui}

Quando crei un VPC utilizzando l'IU della console IBM Cloud, il sistema seleziona il tuo prefisso di indirizzo automaticamente e ti richiede di creare una sottorete all'interno di tale prefisso predefinito. Se questo schema di indirizzi non si adatta ai tuoi requisiti, puoi personalizzare i prefissi dell'indirizzo dopo aver creato il VPC. Successivamente, puoi creare delle sottoreti nei tuoi prefissi dell'indirizzo personalizzati ed eliminare la sottorete che hai creato con il prefisso predefinito.

Questa soluzione temporanea è necessaria per utilizzare BYOIP tramite l'IU della console IBM Cloud.
{:note}

## IBM Cloud VPC e sottoreti
{: #ibm-cloud-vpc-and-subnets}

Puoi dividere un {{site.data.keyword.cloud_notm}} VPC in sottoreti. Tutte le sottoreti in un {{site.data.keyword.cloud_notm}} VPC possono raggiungersi a vicenda attraverso l'instradamento L3 privato da parte di un router implicito. Non devi configurare alcun instradamento o router.

Informazioni utili sulle sottoreti nel VPC:

* Una sottorete è formata da un intervallo di indirizzi IP da te specificato.
* Una sottorete è associata a una sola zona, che non può essere suddivisa in più zone e regioni.
* Una sottorete può estendersi su tutta la zona in un {{site.data.keyword.cloud_notm}} VPC.
* Devi creare il tuo VPC prima di creare le tue sottoreti all'interno di tale VPC.
* Il supporto IPv6 non è disponibile.
* Puoi associare o annullare l'associazione di una VSI a/da una sottorete. (Devi aggiungere un vNIC e selezionare una larghezza di banda).
* Ogni sottorete deve essere contenuta all'interno di un prefisso di indirizzo che appartiene alla zona a cui è associata tale sottorete.

Puoi portare il tuo intervallo di indirizzi IPv4 pubblici (BYOIP) nel tuo account {{site.data.keyword.cloud_notm}} VPC. Quando utilizzi BYOIP, {{site.data.keyword.cloud_notm}} deve configurare tali indirizzi IPv4 sulle risorse {{site.data.keyword.cloud_notm}}, che invieranno i pacchetti da e verso gli indirizzi forniti. Pertanto, in seguito all'utilizzo del tuo intervallo IPv4 fornito su {{site.data.keyword.cloud_notm}}, questi indirizzi IP possono essere esposti al personale di supporto di IBM e terze parti come parte del tuo utilizzo di questo servizio.
{:important}

![Panoramica di IBM Cloud VPC](images/vpc-experience.svg "Panoramica di IBM Cloud VPC")

### Utilizzo dei prefissi di indirizzo per le sottoreti
{: #using-address-prefixes-for-subnets}

Ogni sottorete deve essere all'interno di un prefisso di indirizzo.
 * Per la tua nuova sottorete, puoi selezionare il tuo intervallo di indirizzi IP dai prefissi dell'indirizzo esistenti.
 * Se i prefissi di indirizzo della zona non sono adatti, è possibile modificare uno dei prefissi di indirizzo esistenti oppure è possibile aggiungere un nuovo prefisso di indirizzo per la zona (utilizzando l'API, la CLI o l'IU).

### Indirizzi IP disponibili
{: #available-ip-addresses}

Questi sono gli indirizzi IP disponibili, come definito in **RFC 1918**:

 * 10.0.0.0 – 10.255.255.255
 * 172.16.0.0 – 172.31.255.255
 * 192.168.0.0 – 192.168.255.255

Se utilizzi un intervallo IP al di fuori degli intervalli consentiti per una sottorete (indicati nelle sezioni precedenti), le istanze collegate a tale sottorete potrebbero non essere in grado di raggiungere parti dell'Internet pubblico.

### Ulteriori informazioni sulla creazione di una sottorete
{: #more-about-creating-a-subnet}

Puoi specificare una sottorete per il tuo VPC in due modi:
  * Puoi creare una sottorete fornendo la dimensione della sottorete di cui hai bisogno, come ad esempio il numero di indirizzi supportati (ad esempio, 1024)
  * Puoi creare una sottorete fornendo un intervallo CIDR (ad esempio 10.0.0.8/29)

Esempio di come specificare un blocco di 1024 utilizzando CIDR: se stai specificando un intervallo CIDR invece di una dimensione della sottorete, il blocco IPv4 `192.168.100.0/22` rappresenta i 1024 indirizzi IPv4 da `192.168.100.0` a `192.168.103.255`.
{:tip}

