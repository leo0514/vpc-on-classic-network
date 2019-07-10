---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-29"

keywords: security, ACLs, security groups, traffic, subnet, instance, VSI, firewall, encryption

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# Sicurezza nel tuo IBM Cloud VPC
{: #security-in-your-ibm-cloud-vpc}

Puoi mantenere protetti il tuo VPC e i tuoi carichi di lavoro controllando il traffico di rete utilizzando i gruppi di sicurezza (SG), gli ACL (Access Control List) di rete o entrambi i tipi di controllo. Ricorda:

* I gruppi di sicurezza controllano il traffico a livello di singola istanza (VSI).
* Gli ACL (Access Control List) controllano il traffico a livello di singola sottorete.

## Panoramica della sicurezza
{: #security-overview}

* Il traffico a/da una sottorete può essere controllato dagli elenchi di controllo degli accessi (ACL).
* I gruppi di sicurezza (SG) possono controllare il traffico al livello della VSI
* Configura un gateway pubblico per l'accesso della sottorete a internet, protetto dagli ACL
* Configura un IP mobile per l'accesso della VSI a internet, protetto dagli SG

![Connettività e sicurezza di IBM VPC](images/vpc-connectivity-and-security.svg "Connettività e sicurezza di IBM VPC")

## Definizioni
{: #definitions}

Questo [glossario](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary) fornisce le definizioni e le descrizioni degli ACL e degli SG, e le azioni che puoi effettuare con essi. La sezione che segue descrive le funzionalità di base degli ACL e dei gruppi di sicurezza e i modi in cui VPC supporta la crittografia end-to-end.

### ACL (Access Control List)
{: #access-control-list}

Un **ACL (Access Control List)** può gestire (ossia, può consentire o negare) il traffico in entrata e in uscita per una sottorete. Un ACL è senza stato, il che significa che devono essere specificate delle regole in entrata e in uscita separatamente e in modo esplicito. Ogni ACL è composto di regole, basate su un *IP di origine*, una *porta di origine*, un *IP destinazione*, una *porta di destinazione* e un *protocollo*.

In {{site.data.keyword.cloud}} VPC, ogni sottorete viene creata con un ACL predefinito, che consente il traffico in entrata e in uscita, ma i clienti possono creare degli ACL personalizzati. È sempre collegato solo un ACL a una sottorete, ma un ACL può essere collegato a più sottoreti. Per ulteriori informazioni su come utilizzare gli ACL, consulta la nostra [Guida ACL](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls).

### Gruppo di sicurezza
{: #security-group}

Un **gruppo di sicurezza** si comporta come un firewall virtuale che controlla il traffico per uno o più server (VSI). Un gruppo di sicurezza è una raccolta di regole che specificano se consentire il traffico per una VSI associata.

Quando un cliente crea una VSI, può associare uno o più gruppi di sicurezza a tale VSI. Fornendo le autorizzazioni corrette, i clienti possono modificare le regole del gruppo di sicurezza utilizzando l'API, la CLI o la console IBM.

Per ulteriori informazioni su come creare una VSI che utilizza i gruppi di sicurezza e altre informazioni sulle funzionalità dei gruppi di sicurezza fai riferimento alla nostra [guida per i gruppi di sicurezza](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-using-security-groups).

### Crittografia end-to-end
{: #end-to-end-encryption}

Sebbene IBM Cloud VPC non fornisca la crittografia end-to-end, la consente. Ad esempio, se hai un endpoint sicuro su un server virtuale (ad esempio, un server HTTPS sulla porta 443), puoi collegare un IP mobile a tale server, dopodiché la tua connessione è crittografata end-to-end dal client al server sulla porta 443.  Nulla nel percorso forza una descrizione.

Tieni presente tuttavia, che se utilizzi un protocollo non sicuro come ad esempio HTTP sulla porta 80, i tuoi dati end-to-end sono in chiaro.
