---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, FortiGate, connection, secure

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Creazione di una connessione sicura con un peer FortiGate remoto
{: #creating-a-secure-connection-with-a-remote-fortigate-peer}

Questo documento si basa su FortiGate 300C, versione firmware	v5.2.13,build762 (GA).

Le seguenti istruzioni di esempio saltano i passi preliminari dell'utilizzo della CLI o dell'API {{site.data.keyword.cloud}} per creare i VPC (Virtual Private Cloud). Per ulteriori informazioni, consulta [Introduzione](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) e [Configurazione di VPC con le API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Istruzioni di esempio
{: #fortigate-example-steps}

La topologia per la connessione al peer FortiGate remoto è simile alla creazione di una connessione VPN tra due VPC. Tuttavia, un lato viene sostituito dall'unità FortiGate 300C.

![immetti la descrizione dell'immagine qui](./images/vpc-vpn-fg-figure.png)

### Per creare una connessione sicura con il peer Fortigate remoto
{: #to-create-a-secure-connection-with-the-remote-fortigate-peer}

Vai a **VPN \> IPsec \> Tunnels** e crea un nuovo tunnel personalizzato o modificane uno esistente.

![vpc-vpn-fg-start](./images/vpc-vpn-fg-start.JPG)

Quando un'unità FortiGate riceve una richiesta di connessione da un peer VPN remoto, utilizza i parametri IPsec Phase 1 per stabilire una connessione sicura e autenticare tale peer VPN. Successivamente, se la politica di sicurezza consente la connessione, l'unità FortiGate stabilisce il tunnel utilizzando i parametri IPsec Phase e applica la politica di sicurezza IPsec. I servizi di sicurezza, autenticazione e di gestione della chiave vengono negoziati in modo dinamico tramite il protocollo IKE.

**Per supportare queste funzioni, devono essere eseguiti i seguenti passi di configurazione generale dall'unità FortiGate:**

* Definisci i parametri Phase 1 necessari all'unità FortiGate per autenticare il peer remoto e stabilire una connessione sicura.

* Definisci i parametri Phase 2 necessari all'unità FortiGate per creare un tunnel VPN con il peer remoto.

* Crea le politiche di sicurezza per controllare i servizi e la direzione del traffico consentiti tra gli indirizzi di destinazione e di origine IP.

Per impostazione predefinita, sono stati impostati alcuni parametri Phase 1 e Phase 2 tipici.

### Configurazione della VPN per IBM Cloud VPC
{: #fortigate-configuring-for-the-ibm-cloud-vpc-vpn}

Per la connessione alla funzionalità VPN di IBM Cloud VPC, consigliamo la seguente configurazione:

1. Scegli IKEv2 nell'autenticazione;
2. Abilita `DH-group 2` nella proposta Phase 1
3. Imposta `lifetime = 36000` nella proposta Phase 1
4. Disabilita PFS nella proposta Phase 2
5. Imposta `lifetime = 10800` nella proposta Phase 2
6. Immetti le informazioni della tua sottorete nella Phase 2
7. I parametri rimanenti conservano i propri valori predefiniti.

![immetti la descrizione dell'immagine qui](./images/vpc-vpn-fg-network.JPG)

### Parametri di rete
{: #fortigate-network-parameters}

![immetti la descrizione dell'immagine qui](./images/vpc-vpn-fg-authentication.JPG)

### Parametri di autenticazione
{: #fortigate-authentication-parameters}

![immetti la descrizione dell'immagine qui](./images/vpc-vpn-fg-phase1.JPG)

### Parametri Phase 1
{: #fortigate-phase-1-parameters}

![immetti la descrizione dell'immagine qui](./images/vpc-vpn-fg-phase2.JPG)

### Parametri Phase 2
{: #fortigate-phase-2-parameters}

## Per creare una connessione sicura con l'IBM Cloud VPC locale
{: #fortigate-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

Per creare una connessione sicura, creerai la connessione VPN all'interno del tuo VPC, che è simile all'esempio dei 2 VPC.

* Crea un gateway VPN sulla tua sottorete VPC insieme a una connessione VPN tra il VPC e l'unità FortiGate, impostando `local_cidrs` come valore della sottorete nel VPC e `peer_cidrs` come valore della sottorete nel FortiGate.

**Nota:** lo stato del gateway viene visualizzato come `pending` mentre viene creato il gateway VPN e diventa `available` una volta completata la creazione. La creazione può richiedere qualche minuto.

![immetti la descrizione dell'immagine qui](images/vpc-vpn-fg-connection.png)

### Controlla lo stato della connessione sicura
{: #fortigate-check-the-status-of-the-secure-connection}

Puoi controllare lo stato della tua connessione tramite la console IBM Cloud. Inoltre, puoi provare ad eseguire il `ping` tra i siti utilizzando le VSI.

![immetti la descrizione dell'immagine qui](images/vpc-vpn-fg-status.JPG)
