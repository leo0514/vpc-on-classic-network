---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, StrongSwan, connection, secure, Linux, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{: note: .note}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Creazione di una connessione sicura con un peer StrongSwan remoto
{: #creating-a-secure-connection-with-a-remote-strongswan-peer}

Questo documento si basa su Strongswan, versione Linux StrongSwan U5.3.5/K4.4.0-133-generic.

Le seguenti istruzioni di esempio saltano i passi preliminari dell'utilizzo della CLI o dell'API {{site.data.keyword.cloud}} per creare i VPC (Virtual Private Cloud). Per ulteriori informazioni, consulta [Introduzione](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) e [Configurazione di VPC con le API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Istruzioni di esempio
{: #strongswan-example-steps}

La topologia per la connessione al peer StrongSwan remoto è simile alla creazione di una connessione VPN tra due VPC. Tuttavia, un lato della connessione viene sostituito dall'unità StrongSwan.

![immetti la descrizione dell'immagine qui](./images/vpc-vpn-sw-figure.png)

### Per creare una connessione sicura con un peer StrongSwan remoto
{: #to-create-a-secure-connection-with-a-remote-strongswan-peer}

Vai a **/etc** e crea il tuo nuovo file di configurazione del tunnel personalizzato con un nome simile a **ipsec.abc.conf**. Modifica **/etc/ipsec.conf** in modo che includa **ipsec.abc.conf** aggiungendo questa riga:

    include /etc/ipsec.abc.conf

Quando un peer VPN riceve una richiesta di connessione da un peer VPN remoto, utilizza i parametri IPsec Phase 1 per stabilire una connessione sicura e autenticare tale peer VPN. Successivamente, se la politica di sicurezza consente la connessione, l'unità StrongSwan stabilisce il tunnel utilizzando i parametri IPsec Phase 2 e applica la politica di sicurezza IPsec. I servizi di sicurezza, autenticazione e di gestione della chiave vengono negoziati in modo dinamico tramite il protocollo IKE.

**Per supportare queste funzioni, devono essere eseguiti i seguenti passi di configurazione generale dall'unità StrongSwan:**

* Definisci i parametri Phase 1 necessari a StrongSwan per autenticare il peer remoto e stabilire una connessione sicura.

* Definisci i parametri Phase 2 necessari a StrongSwan per creare un tunnel VPN con il peer remoto.
Per la connessione alla funzionalità VPN di IBM Cloud VPC, consigliamo la seguente configurazione:

1. Scegli `IKEv2` nell'autenticazione;
2. Abilita `DH-group 2` nella proposta Phase 1
3. Imposta `lifetime = 36000` nella proposta Phase 1
4. Disabilita PFS nella proposta Phase 2
5. Imposta `lifetime = 10800` nella proposta Phase 2
6. Immetti le informazioni del tuo peer e della tua sottorete nella proposta Phase 2

```
    vim /etc/ipsec.abc.conf
    conn all
           type=tunnel
           auto=route
           #aggressive=no
           esp=aes256-sha256!
           ike=aes128-sha1-modp1024!
           left=169.45.74.119
           leftsubnet=10.160.26.64/26
           rightsubnet=192.168.17.0/28
           right=169.61.181.116
           leftauth=psk
           rightauth=psk
           leftid="169.45.74.119"
           keyexchange=ikev2
           rightid="169.61.181.116"
           lifetime=10800s
           ikelifetime=36000s
           dpddelay=30s
           dpdaction=restart
           dpdtimeout=120s
```
{: screen}

Imposta la chiave precondivisa con `/etc/ipsec.secrets`

```
vim ipsec.secrets
# This file holds shared secrets or RSA private keys for authentication.

169.45.74.119 169.61.181.116 : PSK "******"

```
{: screen}

Dopo che il file di configurazione ha terminato l'esecuzione, riavvia l'unità StrongSwan.

```
 ipsec restart
```
{: screen}

### Per creare una connessione sicura con l'IBM Cloud VPC locale
{: #strongswan-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

* Per creare una connessione sicura, creerai la connessione VPN all'interno del tuo VPC, che è simile all'esempio dei 2 VPC.

* Crea un gateway VPN sulla sottorete VPC, insieme a una connessione VPN tra il VPC e StrongSwan, impostando `local_cidrs` sul valore della sottorete nel VPC e `peer_cidrs` sul valore della sottorete in StrongSwan.

Lo stato del gateway viene visualizzato come `pending` mentre viene creato il gateway VPN e diventa `available` una volta completata la creazione. La creazione può richiedere qualche minuto.
{: note}

![vpc-vpn-sw-connection](./images/vpc-vpn-sw-connection.png)

### Controlla lo stato per una connessione sicura
{: #strongswan-check-the-status-for-a-secure-connection}

Puoi controllare lo stato della tua connessione tramite la console IBM Cloud. Inoltre, puoi provare ad eseguire il `ping` tra i siti utilizzando le VSI.

![vpc-vpn-sw-status.png](./images/vpc-vpn-sw-status.png)
