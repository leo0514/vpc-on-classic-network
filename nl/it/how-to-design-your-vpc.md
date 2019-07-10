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


# Progettazione di un piano di indirizzamento per un VPC 
{: #vpc-addressing-plan-design}

Il primo passo nella progettazione del tuo VPC dovrebbe essere quello di progettare il tuo piano di indirizzamento. Un piano di indirizzamento correttamente eseguito ha due obiettivi:

* Soddisfa i requisiti di comunicazione delle istanze VPC
* Mantiene la flessibilità per lo sviluppo futuro. 

Questo documento fornisce un esempio di progettazione del piano di indirizzamento per un'applicazione web a tre livelli, in cui ogni livello è supportato da più zone.

Sebbene ogni {{site.data.keyword.cloud}} VPC venga distribuito in una specifica regione, il VPC può estendersi su tutte le zone all'interno di quella regione. {{site.data.keyword.cloud_notm}} VPC definisce un prefisso di indirizzo predefinito per ogni zona. Questi prefissi di indirizzo consentono la comunicazione tra le istanze {{site.data.keyword.cloud_notm}} VPC in zone diverse, fornendo le informazioni di instradamento correlate alla zona richieste dal [router implicito](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary#implicit-router).

Sono coinvolte le stesse procedure di progettazione, indipendentemente dal fatto che l'applicazione sia contenuta completamente nel cloud o se parti dell'applicazione siano in esecuzione in un'altra ubicazione.
{: tip}

## Considerazioni e ipotesi di progettazione
{: #design-considerations-and-assumptions}

Quando si progetta il piano di indirizzamento per un'applicazione, la considerazione primaria è quella di mantenere i blocchi CIDR utilizzati per creare sottoreti all'interno di una singola zona il più possibile contigui. In tal modo, il progettista consente loro di essere riepilogati in un unico prefisso di indirizzo, lasciando spazio per lo sviluppo futuro.

Un'altra considerazione è il numero di indirizzi disponibili di cui una sottorete può avere bisogno per l'adattamento orizzontale. La seguente tabella elenca il numero di indirizzi disponibili in una sottorete, in base alla dimensione del blocco CIDR specificata:

| Dimensione blocco CIDR | Indirizzi disponibili |
| --------------- | ------------------- |
|      /22        |        1019         |
|      /23        |         507         |
|      /24        |         251         |
|      /25        |         123         |
|      /26        |          59         |
|      /27        |          27         |
|      /28        |          11         |

Sulla base di queste due considerazioni, ecco le ipotesi che facciamo per questo esempio:

* Per questo esempio, gli intervalli CIDR dal blocco 172.16.0.0/12 degli indirizzi RFC 1918 verranno utilizzati per tutte le sottoreti.
* Supponiamo che il livello di presentazione dell'applicazione sia uno strato sottile sopra un'API REST. Pertanto, l'adattamento orizzontale influisce sul livello intermedio più di quanto influisca sul livello di presentazione.

## Determina la dimensione della sottorete di ogni livello
{: #determine-each-tier-s-subnet-size}

Il passo successivo consiste nel determinare la dimensione della sottorete di ogni livello (in termini di indirizzi disponibili). Ogni livello dell'applicazione ha una presenza in ciascuna zona, quindi ogni zona richiederà tre sottoreti.

Queste sono le considerazioni che utilizzeremo durante la pianificazione della dimensione della sottorete di ogni livello:

* Il livello di database (il backend) è il meno probabile che abbia bisogno del ridimensionamento dinamico, quindi queste sottoreti possono essere le più piccole. Vale a dire, queste sottoreti possono contenere il minor numero di indirizzi disponibili. 
    * _Questo esempio utilizza un blocco CIDR `/27`, che consente 27 indirizzi in questo livello._
* Il livello intermedio è il più probabile che abbia bisogno del ridimensionamento dinamico, quindi queste sottoreti saranno le più grandi. Vale a dire, devono contenere il maggior numero di indirizzi disponibili. 
    * _Questo esempio utilizza un blocco CIDR `/25`, che consente 123 indirizzi in questo livello._
* Il livello di frontend si colloca nel mezzo; non avrà bisogno di tanti indirizzi quanto il livello intermedio, ma ha bisogno di più indirizzi rispetto al livello di database. 
    * _Questo esempio utilizza un blocco CIDR `/26`, che consente 59 indirizzi in questo livello._

## Combinazione delle sottoreti e selezione dei prefissi di indirizzo
{: #combining-the-subnets-and-selecting-the-address-prefixes}

Per selezionare un prefisso di indirizzo accettabile per ciascuna zona, avrai bisogno di una dimensione di sottorete abbastanza grande per ospitare tutte e tre le sottoreti in ogni livello e lasciare comunque spazio per l'adattamento orizzontale e l'espansione futura. 

Un prefisso di indirizzo `/24` è il prefisso più piccolo in cui queste tre sottoreti possono essere combinate (27 + 123 + 59). Come procedura ottimale, ti consigliamo di selezionare la dimensione della sottorete successiva più grande, non la più piccola. L'assegnazione della dimensione della sottorete successiva più grande (`/23`) consente l'adattamento orizzontale oltre i limiti indicati in precedenza, poiché consente l'aggiunta di nuove sottoreti a ciascun livello, dall'interno dello stesso prefisso di indirizzo.

Ora che abbiamo determinato la dimensione corretta della sottorete, è possibile assegnare i prefissi di indirizzo effettivi, uno per ogni zona:

|  Zona  | Prefisso indirizzo  |
| ------ | --------------- |
| Zona 1 | `172.16.0.0/23` |
| Zona 2 | `172.16.2.0/23` |
| Zona 3 | `172.16.4.0/23` |

E da questa base, è possibile assegnare le 3 sottoreti all'interno di ciascuna zona:

|  Zona  |   Livello   |   CIDR sottorete   |
| ------ | -------- | ----------------- |
| Zona 1 |  Intermedio  |  `172.16.0.0/25`  |
| Zona 1 |  Frontend   |  `172.16.1.0/26`  |
| Zona 1 | Database | `172.16.1.128/27` |
| Zona 2 |  Intermedio  |  `172.16.2.0/25`  |
| Zona 2 |  Frontend   |  `172.16.3.0/26`  |
| Zona 2 | Database | `172.16.3.128/27` |
| Zona 3 |  Intermedio  |  `172.16.4.0/25`  |
| Zona 3 |  Frontend   |  `172.16.5.0/26`  |
| Zona 3 | Database | `172.16.5.128/27` |

## Considerazioni sull'estensione di un'infrastruttura esistente
{: #considerations-for-extending-an-existing-infrastructure}

Quando pianifichi un VPC che estende un'infrastruttura esistente, puoi seguire i passi precedenti, indipendentemente dal fatto che tale infrastruttura sia la tua infrastruttura in loco, un altro VPC o anche un altro cloud. Tieni presente che non devi riutilizzare gli intervalli di indirizzi esistenti. Evitando il riutilizzo degli indirizzi, puoi massimizzare la tua capacità di utilizzo delle funzioni di {{site.data.keyword.cloud_notm}} VPC.
