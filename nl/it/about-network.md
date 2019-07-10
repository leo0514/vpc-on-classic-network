---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: secure, region, zone, subnet, terminology, public gateway, floating IP, NAT

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

# Informazioni sulla rete per VPC
{: #about-networking-for-vpc}

Un Virtual Private Cloud (VPC) è una rete virtuale collegata al tuo account cliente. Ti fornisce la sicurezza cloud, con la capacità di eseguire il ridimensionamento in modo dinamico, fornendo un controllo dettagliato sulla tua infrastruttura virtuale e sulla tua segmentazione del traffico di rete.

Questo documento esamina alcuni concetti di rete man mano che vengono applicati all'interno di {{site.data.keyword.cloud}} VPC. I casi di utilizzo e le caratteristiche descritti includono:

* regioni
* zone
* indirizzi IP riservati
* gateway pubblici
* interfacce vNIC
* sottoreti
* indirizzi IP mobili
* connessioni VPN

## Panoramica
{: #subnets-overview}

Per l'accesso a Internet dal tuo VPC sono disponibili tre opzioni:
* Utilizzare un gateway pubblico per il traffico Internet dall'intera sottorete.
* Utilizzare un IP mobile per il traffico Internet da e verso un'istanza.
* Utilizzare una VPN per proteggere la connettività esterna.

Ricorda:
* Alcuni intervalli CIDR di indirizzi della sottorete sono riservati a IBM.
* Devi creare il tuo VPC prima di creare le sottoreti all'interno di tale VPC.
* Il supporto IPv6 non è disponibile.

Facoltativamente, puoi creare un VPC con accesso classico per la connessione alla tua infrastruttura classica IBM Cloud.
{:note}

Come viene mostrato nella seguente figura:
* Le sottoreti nel tuo VPC possono connettersi all'Internet pubblico tramite un gateway pubblico (PGW) facoltativo.
* Puoi assegnare un indirizzo IP mobile (FIP) a qualsiasi istanza per raggiungerla da Internet o viceversa, indipendentemente dal fatto che la sottorete sia collegata a un gateway pubblico.
* Le sottoreti all'interno di IBM Cloud VPC offrono la connettività privata; possono comunicare tra loro con un link privato, tramite il router implicito. La configurazione degli instradamenti non è necessaria.

![Connettività e sicurezza di IBM VPC](images/vpc-connectivity-and-security.svg "Connettività e sicurezza di IBM VPC")

## Terminologia
{: #network-terminology}

Questo [glossario](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary#vpc-glossary) contiene le definizioni e le informazioni sui termini utilizzati in questo documento di IBM Cloud VPC. Quando utilizzi il tuo VPC, dovrai avere familiarità con i concetti di base di _regione_ e _zona_ in quanto vengono applicati alla tua distribuzione.

### Regioni
{: #subnet-regions}

Una regione è un'astrazione correlata all'area geografica in cui viene distribuito un VPC. Ogni regione contiene più zone, che rappresentano dei domini di errore indipendenti. Un IBM Cloud VPC può essere suddiviso su più zone all'interno della sua regione assegnata.

### Zone
{: #subnet-zones}

Una zona è un'astrazione che fa riferimento al data center fisico che ospita le risorse di calcolo, rete e archiviazione, nonché il raffreddamento e l'alimentazione correlati, che fornisce i servizi e le applicazioni. L'isolamento delle zone migliora la tolleranza di errore generale del sistema, riduce la latenza ed impedisce la creazione di un singolo punto di errore condiviso. Una zona garantisce le seguenti proprietà:

 * Ogni zona è un dominio di errore indipendente ed è estremamente improbabile che due zone in una regione abbiamo contemporaneamente un malfunzionamento.
 * Il traffico tra le zone in una regione avrà meno di 2ms di latenza.

## Caratteristiche delle sottoreti nel VPC
{: #characteristics-of-subnets}

Una sottorete è formata da un intervallo di indirizzi IP specificati (blocco CIDR). Le sottoreti sono associate a una sola zona e non possono essere suddivise su più zone o regioni. Tuttavia, una sottorete può estendersi sulla totalità delle astrazioni della zona all'interno del proprio Virtual Private Cloud. Le sottoreti nello stesso IBM Cloud VPC sono connesse tra loro.

### Indirizzi IP riservati
{: #reserved-ip-addresses}

Alcuni indirizzi IP sono riservati per l'utilizzo da parte di IBM quando utilizza il Virtual Private Cloud. Di seguito vengono riportati gli indirizzi riservati (questi indirizzi IP presumono che l'intervallo CIDR della sottorete sia 10.10.10.0/24):

  * Primo indirizzo nell'intervallo CIDR (10.10.10.0): indirizzo della rete
  * Secondo indirizzo nell'intervallo CIDR (10.10.10.1): indirizzo del gateway
  * Terzo indirizzo nell'intervallo CIDR (10.10.10.2): riservato da IBM
  * Quarto indirizzo nell'intervallo CIDR (10.10.10.3): riservato da IBM per un utilizzo futuro
  * Ultimo indirizzo nell'intervallo CIDR (10.10.10.255): indirizzo broadcast di rete

### Utilizza un gateway pubblico per la connettività esterna di una sottorete
{: #use-a-public-gateway}

Un **gateway pubblico (PGW)** consente a una sottorete (con tutte le istanze collegate alla sottorete) di connettersi a Internet. Tieni presente che le sottoreti sono private per impostazione predefinita; tuttavia, facoltativamente, puoi creare un PGW e collegargli una sottorete. Dopo aver collegato una sottorete al PGW, tutte le istanze in tale sottorete possono connettersi a Internet.

Il PGW utilizza la _NAT many-to-1_, il che significa che migliaia di istanze con indirizzi privati utilizzeranno 1 indirizzo IP pubblico per comunicare con l'Internet pubblico. Il PGW non consente a Internet di avviare una connessione con tali istanze. Utilizza l'API per collegare e scollegare le sottoreti a/da il tuo PGW.

La seguente figura riepiloga l'attuale campo di applicazione dei servizi gateway.

| SNAT | DNAT | ACL | VPN |
| ---- | ---- | --- | --- |
| Le istanze possono avere accesso solo in uscita a Internet | Consenti la connettività in entrata da Internet a un IP privato | Fornisci accesso in entrata limitato da Internet a istanze o sottoreti | La VPN Site-to-Site gestisce clienti di qualsiasi dimensione e singole o più ubicazioni |
| Intere sottoreti condividono lo stesso endpoint pubblico in uscita | Fornisce accesso limitato a un singolo server privato | Limita l'accesso in entrata da Internet, in base a servizio, protocollo o porta | L'elevata velocità effettiva (fino a 10 Gbps) offre ai clienti la possibilità di trasferire file di dati di grandi dimensioni in modo sicuro e rapido |
| Protegge le istanze; non è possibile avviare l'accesso alle istanze attraverso l'endpoint pubblico | Il servizio DNAT può essere ampliato o ridotto in base ai requisiti | Gli ACL senza stato consentono il controllo granulare del traffico | Crea connessioni sicure con la crittografia standard del settore |

Un gateway pubblico viene creato in un VPC, ma non fa nulla finché non viene collegato a una sottorete. Puoi creare solo un gateway pubblico per zona, il che significa, ad esempio, che potresti avere tre (3) gateway pubblici per ogni VPC in un ambiente con 3 zone.
{:note}

## Limitazioni delle sottoreti
{: #limitations-of-subnets}

Per un elenco completo di limitazioni note e funzioni non attualmente supportate, fai riferimento al documento [Limitazioni note](/docs/vpc-on-classic?topic=vpc-on-classic-known-limitations).

### Restrizioni all'eliminazione di una sottorete
{: #restrictions-on-deleting-a-subnet}

Non puoi eliminare una sottorete se al suo interno sono presenti delle risorse in uso (come una VSI (Virtual Server Instance) o IP mobili); è necessario eliminare prima le risorse.

### Limitazioni sull'aggiornamento di una sottorete esistente
{: #limitations-on-updating-an-existing-subnet}

* Non puoi ridimensionare una sottorete esistente. Ad esempio, 10.10.16.0/24 non può essere ridimensionata in 10.10.16.0/20.
* Non puoi spostare una sottorete esistente. Ad esempio, 10.10.10.0/24 non può essere spostata su 10.10.11.0/24.

## Connettività esterna
{: #external-connectivity}

La connettività esterna può essere ottenuta tramite un indirizzo IP mobile collegato a un'istanza, un gateway esterno collegato a una sottorete o mediante un tunnel VPN.

### Utilizza un indirizzo IP mobile per la connettività esterna di un'istanza 
{: #use-floating-ip}

Gli **indirizzi IP mobili** sono indirizzi IP forniti dal sistema e raggiungibili dall'Internet pubblico.

Puoi riservare un indirizzo IP mobile dal pool di indirizzi IP mobili disponibili forniti da IBM e puoi associarlo o annullarne l'associazione a/da ogni istanza nello stesso Virtual Private Cloud, tramite la vNIC di tale istanza. Ogni indirizzo IP mobile può essere associato a vari tipi di istanze del server virtuale (VSI), ad esempio, un programma di bilanciamento del carico o un gateway VPN.

Il tuo indirizzo IP mobile non può essere associato a più interfacce.Devi specificare l'interfaccia sulla VSI che sarà associata a quell'indirizzo IP mobile individuale. Tale interfaccia avrà anche un indirizzo IP privato. Il sistema di backend esegue le operazioni _NAT 1-to-1_ tra l'IP mobile e l'IP privato di tale interfaccia. Un IP mobile viene rilasciato solo quando richiedi specificatamente l'azione di rilascio.

**Note:**
* **L'associazione di un indirizzo IP mobile a una VSI, rimuove la VSI dalla NAT many-to-1 del PGW menzionata in precedenza.**
* **Al momento, l'IP mobile supporta solo indirizzi IPv4.**
* **Non puoi utilizzare il tuo indirizzo IP pubblico come IP mobile.**

Per ulteriori informazioni sulle operazioni NAT, fai riferimento al [documento RFC internet correlato ![Icona link esterno](../../icons/launch-glyph.svg "Icona link esterno")](http://www.faqs.org/rfcs/rfc1631.html){: new_window}.

### Utilizza la VPN per proteggere la connettività esterna
{: #use-vpn}

Il servizio VPN (Virtual Private Network) è disponibile per consentire agli utenti di connettersi al proprio IBM Cloud VPC da Internet in modo sicuro.

**Funzionalità della VPN**
  * Capacità di eseguire le operazioni CRUD (creare, leggere, aggiornare ed eliminare) per il servizio VPN (VPN IPSEC site-to-site) per gestire il trasferimento di dati.
  * Capacità di associare il servizio VPN all'IBM Cloud VPC o alla rete virtuale di un cliente.
  * Capacità di identificare le sottoreti in loco del cliente tramite la definizione statica o l'instradamento dinamico.
  * Supporto per proteggere le cifrature come ad esempio SHA256, AES, 3DES, IKEv2.
  * Affidabilità del servizio integrata.
  * Monitoraggio continuo dell'integrità della connessione VPN.
  * Supporto per le modalità dell'iniziatore e del risponditore; ossia, il traffico può essere avviato da entrambi i lati del tunnel.

## Ulteriori informazioni
{: #subnets-learn-more}
   * [Sicurezza di IBM VPC](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-security-in-your-ibm-cloud-vpc)
   * [Indirizzi, regioni e sottoreti di IBM VPC](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-working-with-ip-address-ranges-address-prefixes-regions-and-subnets)
