---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, Cisco, ASAv, connection, secure, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc} 
{:new_window: target="_blank"} 
{:DomainName: data-hd-keyref="DomainName"} 
{:note: .note} 
{:important: .important} 
{:deprecated: .deprecated} 
{:generic: data-hd-programlang="generic"}

# Creazione di una connessione sicura con un peer Cisco ASAv remoto
{: #creating-a-secure-connection-with-a-remote-cisco-asav-peer}

Questo documento si basa su Cisco ASAv, Cisco Adaptive Security Appliance Software versione 9.10(1).

Le seguenti istruzioni di esempio saltano i passi preliminari dell'utilizzo della CLI o dell'API {{site.data.keyword.cloud}} per creare i VPC (Virtual Private Cloud). Per ulteriori informazioni, consulta [Introduzione](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) e [Configurazione di VPC con le API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Istruzioni di esempio
{: #cisco-example-steps}

La topologia per la connessione al peer Cisco ASAv remoto è simile alla creazione di una connessione VPN tra due {{site.data.keyword.cloud}} Virtual Private Cloud. Tuttavia, un lato viene sostituito dall'unità Cisco ASAv.

![immetti la descrizione dell'immagine qui](./images/vpc-vpn-asav-figure.png)

### Per creare una connessione sicura con il peer Cisco ASAv remoto
{: #to-create-a-secure-connection-with-the-remote-cisco-asav-peer}

Il primo passo nella configurazione del tuo Cisco ASA per poterlo utilizzare con la VPN di IBM VPC è
di assicurarti che le seguenti condizioni prerequisite siano state impostate:

* Cisco ASAv è online e funzionante con una licenza appropriata
* È abilitata una password per Cisco ASAv
* Ci sia almeno un'interfaccia interna funzionale verificata e configurata
* Ci sia almeno un'interfaccia esterna funzionale verificata e configurata

Quando un'unità Cisco ASAv riceve una richiesta di connessione da un peer VPN remoto, utilizza i parametri IPsec Phase 1 per stabilire una connessione sicura e autenticare tale peer VPN. Successivamente, se la politica di sicurezza consente la connessione, Cisco ASAv stabilisce il tunnel utilizzando i parametri IPsec Phase 1 e applica la politica di sicurezza IPsec. I servizi di sicurezza, autenticazione e di gestione della chiave vengono negoziati in modo dinamico tramite il protocollo IKE.

**Per supportare queste funzioni, devono essere eseguiti i seguenti passi di configurazione generale dall'unità Cisco ASAv:**

* Definisci i parametri Phase 1 necessari all'unità Cisco ASAv per autenticare il peer remoto e stabilire una connessione sicura.
* Definisci i parametri Phase 2 necessari all'unità Cisco ASAv per creare un tunnel VPN con il peer remoto.

Crea un oggetto della proposta IKE (Internet Key Exchange) versione 2. Gli oggetti della proposta IKEv2
contengono i parametri necessari per la creazione delle proposte IKEv2 durante la definizione delle politiche VPN site-to-site
e di accesso remoto. IKE è un protocollo di gestione delle chiavi che facilita la gestione delle comunicazioni
basate su IPsec. Viene utilizzato per autenticare i peer IPsec, negoziare e distribuire le chiavi di crittografia
IPsec e stabilire automaticamente le associazioni di sicurezza (SA) IPsec. 

```
group-policy GroupPolicy_161.156.80.10 internal
group-policy GroupPolicy_161.156.80.10 attributes
 vpn-tunnel-protocol ikev1 ikev2 
tunnel-group 161.156.80.10 type ipsec-l2l
tunnel-group 161.156.80.10 general-attributes
 default-group-policy GroupPolicy_161.156.80.10
tunnel-group 161.156.80.10 ipsec-attributes
 ikev1 pre-shared-key <key value>
 ikev2 remote-authentication pre-shared-key <key value>
 ikev2 local-authentication pre-shared-key <key value>
```

Crea una configurazione della politica IKEv2 per la connessione IPsec. Il blocco della politica IKEv2 imposta i
parametri per lo scambio IKE. In questo blocco, sono impostati i seguenti parametri:
* Algoritmo di crittografia - impostato su AES-256 per questo esempio
* Algoritmo di integrità - impostato su SHA256 per questo esempio
* Gruppo Diffie-Hellman - IPsec utilizza l'algoritmo Diffie-Hellman per generare la chiave di crittografia iniziale
tra i peer. In questo esempio è impostato su group 14
* PRF (Pseudo-Random Function) - IKEv2 richiede un metodo separato utilizzato come algoritmo
con cui derivare il materiale chiave e le operazioni di hash necessarie per la crittografia
del tunnel IKEv2. Vi viene fatto riferimento come alla funzione pseudocasuale ed è impostata su SHA
* Ciclo di vita SA - imposta il ciclo di vita delle associazioni di sicurezza (dopo il quale
si verificherà una riconnessione). Impostato su 36.000 secondi.
* Tipo di operazione - mantieni il valore predefinito, bi-directional. (Non è esplicito nella visualizzazione "show running".)

Come mostrato nel seguente esempio di codice, questa politica di esempio utilizza AES-256 per crittografare il canale sicuro. L'algoritmo di hash SHA512
viene utilizzato per convalidare l'identità del peer remoto, il group 14 di Diffie-Hellman
viene utilizzato per la generazione della chiave. Group 14 utilizza blocchi di crittografia a 2048 bit. Infine,
viene impostato un ciclo di vita per l'associazione di sicurezza di 36.000 secondi.

```
crypto ikev2 policy 100
encryption aes-256
integrity sha-1
group 14
prf sha
lifetime seconds 36000
```

* Definisci l'elenco degli accessi e la mappa di crittografia per la VPN:

```
access-list outside_cryptomap_1 extended permit ip object NETWORK_OBJ_192.168.236.0_24 object vpc 
crypto map outside_map 1 match address outside_cryptomap_1
crypto map outside_map 1 set peer 161.156.80.10 
crypto map outside_map 1 set ikev1 transform-set ESP-AES-128-SHA ESP-AES-128-MD5 ESP-AES-192-SHA ESP-AES-192-MD5 ESP-AES-256-SHA ESP-AES-256-MD5 ESP-3DES-SHA ESP-3DES-MD5 ESP-DES-SHA ESP-DES-MD5
crypto map outside_map 1 set ikev2 ipsec-proposal AES256 AES192 AES 3DES DES
crypto map outside_map interface outside
nat (any,outside) source static NETWORK_OBJ_192.168.236.0_24 NETWORK_OBJ_192.168.236.0_24 destination static vpc vpc no-proxy-arp route-lookup
```

## Per creare una connessione sicura con l'IBM Cloud VPC locale
{: #to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

Per creare una connessione sicura, creerai la connessione VPN all'interno del tuo VPC, che è simile all'esempio dei 2 VPC.

* Crea un gateway VPN sulla tua sottorete VPC insieme a una connessione VPN tra il VPC e Cisco ASAv, impostando `local_cidrs` come valore della sottorete nel VPC e `peer_cidrs` come valore della sottorete nel Cisco ASAv.

Lo stato del gateway viene visualizzato come `pending` mentre viene creato il gateway VPN e diventa `available` una volta completata la creazione. La creazione può richiedere qualche minuto. 
{:note}


![immetti la descrizione dell'immagine qui](./images/vpc-vpn-asav-connection.png)

### Controlla lo stato della connessione sicura
{: #cisco-check-the-status-of-the-secure-connection}

Puoi controllare lo stato della tua connessione tramite la console IBM Cloud. Inoltre, puoi provare ad eseguire il `ping` tra i siti utilizzando le VSI.

![immetti la descrizione dell'immagine qui](./images/vpc-vpn-asav-status.png)
