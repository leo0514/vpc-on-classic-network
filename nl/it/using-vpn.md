---



copyright:
  years: 2017,2018, 2019
lastupdated: "2019-06-04"

keywords: VPN, network, encryption, authentication, algorithm, IKE, IPsec, policies, gateway, auto-negotiation

subcollection: vpc-on-classic-network


---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Utilizzo della VPN con il tuo VPC
{: #--using-vpn-with-your-vpc}
[comment]: # (argomento della guida collegato)

Il servizio VPN di {{site.data.keyword.cloud}} VPC ti consente di connettere le reti private in modalità sicura. Puoi utilizzare la VPN per configurare un tunnel site-to-site IPsec tra il tuo VPC e la tua rete privata in loco o un altro VPC.

Per la release corrente di {{site.data.keyword.cloud}} VPC, è supportato solo l'instradamento basato sulla politica.

## Funzioni
{: #vpn-features}

* IKEv1 e IKEv2
* Algoritmi di autenticazione: `md5`, `sha1`, `sha256`
* Algoritmi di crittografia: `3des`, `aes128`, `aes256`
* Gruppi DH (Diffie-Hellman): 2, 5, 14
* Modalità di negoziazione IKE: main
* Protocollo di trasformazione IPSec: ESP
* Modalità di incapsulamento IPSec: tunnel
* PFS (Perfect Forward Secrecy)
* Rilevamento peer non attivo
* Instradamento: basato sulla politica
* Modalità di autenticazione: chiave precondivisa
* Supporto HA solo in modalità Attivo/Standby

## API disponibili
{: #apis-available}

La seguente sezione ti fornisce i dettagli sulle API che puoi utilizzare per la VPN nel tuo ambiente IBM Cloud VPC. Per ulteriori dettagli, consulta la pagina [API REST VPC](https://{DomainName}/apidocs/vpc-on-classic#list-all-ike-policies).

### Gateway e connessioni VPN
{: #vpn-gateways-and-vpn-connections}

| Descrizione | API |
|----------------------------|-------------|
| Crea un gateway VPN | POST /vpn_gateways |
| Richiama i gateway VPN | GET /vpn_gateways |
| Richiama un gateway VPN | GET /vpn_gateways/{id} |
| Elimina un gateway VPN | DELETE /vpn_gateways/{id} |
| Aggiorna un gateway VPN | PATCH /vpn_gateways/{id} |
| Crea una nuova connessione VPN | POST /vpn_gateways/{vpn_gateway_id}/connections |
| Richiama le connessioni VPN | GET /vpn_gateways/{vpn_gateway_id}/connections |
| Richiama una connessione VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Elimina una connessione VPN | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Aggiorna una connessione VPN | PATCH /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Richiama tutti i CIDR locali per una connessione VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs |
| Elimina un CIDR locale da una connessione VPN | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Verifica se esiste un CIDR locale specifico su una connessione VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Imposta un CIDR locale su una connessione VPN | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Richiama tutti i CIDR peer per una connessione VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs |
| Elimina un CIDR peer da una connessione VPN | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Verifica se esiste un CIDR peer specifico su una connessione VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Imposta un CIDR peer su una connessione VPN | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |

### Politiche IKE
{: #ike-policies}

| Descrizione | API |
|-----------------------------|--------------|
| Richiama tutte le politiche IKE | GET /ike_policies |
| Crea una politica IKE | POST /ike_policies |
| Elimina una politica IKE | DELETE /ike_policies/{id} |
| Richiama una politica IKE | GET /ike_policies/{id} |
| Aggiorna una politica IKE | PATCH /ike_policies/{id} |
| Richiama tutte le connessioni che utilizzano la politica IKE specificata | GET /ike_policies/{id}/connections |

### Politiche IPsec
{: #ipsec-policies}

| Descrizione | API |
|---------------------|-------------|
| Richiama tutte le politiche IPSec | GET /ipsec_policies |
| Crea una politica IPSec | POST /ipsec_policies |
| Elimina una politica IPSec | DELETE /ipsec_policies/{id} |
| Richiama una politica IPSec | GET /ipsec_policies/{id} |
| Aggiorna una politica IPSec | PATCH /ipsec_policies/{id} |
| Richiama tutte le connessioni che utilizzano la politica IPsec specificata | GET /ipsec_policies/{id}/connections |

## Esempio VPN
{: #vpn-example}

Nel seguente esempio, potrai connettere due VPC tra loro utilizzando la VPN, il che significa che puoi
connettere le sottoreti in due VPC separati come se fossero una singola rete. Gli indirizzi IP delle sottoreti non devono sovrapporsi.
Ecco come appare lo scenario (con alcune VM aggiunte in ogni VPC):

![VPN per IBM VPC](images/vpc-vpn.svg "VPN per IBM VPC")

### Istruzioni di esempio
{: #vpn-example-steps}

Le seguenti istruzioni di esempio saltano i passi preliminari dell'utilizzo della CLI o dell'API IBM Cloud per creare i VPC. Per ulteriori informazioni, consulta [Introduzione](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) e [Configurazione di VPC con le API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

Facoltativamente, puoi creare un gateway VPN utilizzando l'IU. Le istruzioni possono essere trovate nel [documento dell'esercitazione della console](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn).

#### Passo 1. Crea un gateway VPN nella tua sottorete VPC
{: #step-1-create-a-vpn-gateway-in-your-vpc-subnet}

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways?version=2019-05-31&generation=1" \
    -d '{
            "name": "vpn-gateway-1",
            "subnet": {"id": $subnet1}
        }'
```
{: codeblock}

Output di esempio:
```
{
    "id": "7fd72524-6e2d-49a6-b975-0071efccd89a",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:7fd72524-6e2d-49a6-b975-0071efccd89a",
    "name": "vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a",
    "created_at": "2018-07-06T19:19:28.694388Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.167"
    },
    "subnet": {
        "id": "f45ee0be-cf3f-41ca-a279-23139110aa58",
        "name": "subnet-1",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f45ee0be-cf3f-41ca-a279-23139110aa58"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

Assicurati di salvare i seguenti campi per i prossimi passi.
* `id`. Questo è l'ID del gateway VPN e vi sarà fatto riferimento come a `$gwid1`.
* `address`. Questo è l'indirizzo IP pubblico del gateway VPN e vi sarà fatto riferimento come a `$gwaddress1`.

Lo stato del gateway viene visualizzato come `pending` mentre viene creato il gateway VPN e diventa `available` una volta completata la creazione. La creazione può richiedere qualche minuto.
{: note}


Puoi controllare lo stato del gateway con il seguente comando:

```bash
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/vpn_gateways/$gwid1?version=2019-05-31&generation=1"
```
{: codeblock}

#### Passo 2. Crea un secondo gateway VPN su un VPC differente
{: #step-2-create-a-second-vpn-gateway-on-a-different-vpc}

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-gateway-2",
                "subnet": {"id": $subnet2}
            }'
```
{: codeblock}

Output di esempio:
```
{
    "id": "f72559a3-2fac-4958-b937-54474e6a8a8d",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:f72559a3-2fac-4958-b937-54474e6a8a8d",
    "name": "vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d",
    "created_at": "2018-07-06T19:33:23.789675Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.150"
    },
    "subnet": {
        "id": "f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4",
        "name": "subnet-2",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

Assicurati di salvare i seguenti campi per i prossimi passi.
* `id`. Questo è l'ID del gateway VPN e vi sarà fatto riferimento come a `$gwid2`.
* `address`. Questo è l'indirizzo IP pubblico del gateway VPN e vi sarà fatto riferimento come a `$gwaddress2`.


#### Passo 3. Crea una connessione VPN dal primo al secondo gateway VPN
{: #step-3-create-a-vpc-connection-from-the-first-vpn-gateway-to-the-second-vpn-gateway}

Quando crei la connessione, imposta `local_cidrs` sulla sottorete nel **VPC uno** e `peer_cidrs` sulla sottorete nel **VPC due**.

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-2",
                "peer_address": $gwaddress2,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

Output di esempio:
```
{
    "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "name": "vpn-connection-to-vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.0.0/24"
    ],
    "peer_address": "169.61.161.150",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
                "interval": 30,
                "timeout": 120
    },
    "created_at": "2018-07-06T19:50:49.252072Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### Passo 4. Crea una connessione VPN dal secondo al primo gateway VPN
{: #step-4-create-a-vpn-connection-from-the-second-vpn-gateway-to-the-first-vpn-gateway}

Quando crei la connessione, imposta `local_cidrs` sulla sottorete nel **VPC due** e `peer_cidrs` sulla sottorete nel **VPC uno**.

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways/$gwid2/connections?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-1",
                "peer_address": $gwaddress1,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

Output di esempio:
```
{
    "id": "1d4dbacq-673d-2qed-hf68-858961739gf0",
    "name": "vpn-connection-to-vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d/connections/1d4dbacq-673d-2qed-hf68-858961739gf0",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_address": "169.61.161.167",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
                "interval": 30,
                "timeout": 120
    },
    "created_at": "2018-07-06T19:54:14.961597Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### Passo 5. Verifica della connettività
{: #step-5-verify-connectivity}

Dopo aver stabilito la connessione VPN, potrai raggiungere le tue istanze sulla sottorete due dalla sottorete uno e viceversa.

Puoi controllare lo stato della connessione VPN nel seguente modo:
```bash
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-05-31&generation=1"
```
{: codeblock}

Output di esempio:
```
{
    "first": {
        "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections?limit=10"
    },
    "limit": 10,
    "connections": [
        {
            "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "name": "vpn-connection-to-vpn-gateway-2",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "local_cidrs": [
                "192.168.100.0/24"
            ],
            "peer_cidrs": [
                "192.168.0.0/24"
            ],
            "peer_address": "169.61.161.150",
            "admin_state_up": true,
            "psk": "VPNDemoPassword",
            "dead_peer_detection": {
                "action": "none",
                "interval": 30,
                "timeout": 120
            },
            "created_at": "2018-07-06T19:50:49.252072Z",
            "route_mode": "policy",
            "authentication_mode": "psk",
            "status": "up"
        }
    ]
}
```
{: screen}

## Quote
{: #see-vpn-quotas}

Consulta il nostro argomento [Quote VPC](/docs/vpc-on-classic?topic=vpc-on-classic-quotas#vpn-quotas) per le quote VPN.

## Negoziazione automatica della politica
{: #policy-auto-negotiation}

L'utilizzo delle politiche IKE e IPsec per configurare una connessione VPN è facoltativo. Quando non viene selezionata alcuna politica, vengono automaticamente scelte le proposte predefinite per un processo a cui viene fatto riferimento come _negoziazione automatica_. 

La negoziazione automatica di IBM Cloud utilizza **IKEv2** e, pertanto, anche il dispositivo in loco deve utilizzare **IKEv2**. Utilizza una politica IKE personalizzata se il tuo dispositivo in loco non supporta **IKEv2**.
{: note}

### Negoziazione automatica IKE (fase 1)
{: #ike-auto-negotiation-phase-1}

È possibile utilizzare le seguenti opzioni di crittografia, autenticazione e gruppo Diffie-Hellman in qualsiasi combinazione:

|    | Crittografia | Autenticazione | Gruppo DH |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | 2  |
| 2  | aes256 | sha256 | 5  |
| 3  | 3des   | md5    | 14 |

### Negoziazione automatica IPsec (fase 2)
{: #ipsec-auto-negotiation-phase-2}

È possibile utilizzare le seguenti opzioni di crittografia e autenticazione in qualsiasi combinazione:

|    | Crittografia | Autenticazione | Gruppo DH |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | disabilitato  |
| 2  | aes256 | sha256 | disabilitato  |
| 3  | 3des   | md5    | disabilitato  |

## Domande frequenti (FAQ)
{: #vpn-faq}

**Quando creo un gateway VPN, posso creare le connessioni VPN nello stesso momento?**

Se utilizzi l'API o la CLI, le connessioni VPN devono essere creata dopo aver creato il gateway VPN. Nella console IBM Cloud, puoi creare il gateway e una connessione contemporaneamente.

**Se elimino un gateway VPN con delle connessioni VPN collegate, cosa succede alle connessioni?**

Le connessioni VPN vengono eliminate insieme al gateway VPN.

**Le politiche IKE o IPSec saranno eliminate se elimino una connessione o un gateway VPN?**

No, le politiche IKE e IPSec possono essere applicate a più connessioni.

**Cosa succede a un gateway VPN se provo ad eliminare la sottorete su cui risiede il gateway?**

La sottorete non può essere eliminata se è presente una qualsiasi istanza, incluso il gateway VPN.

**Esistono delle politiche IKE e IPsec predefinite?**

Quando crei una connessione VPN senza riferimento a un ID politica (IKE o IPsec), viene utilizzata la negoziazione automatica.

**Perché devo scegliere una sottorete durante il provisioning di un gateway VPN?**

La sottorete connette il gateway VPN ad altre risorse nel tuo VPC. La procedura ottimale consigliata è quella di creare
una sottorete dedicata per il gateway VPN, senza altre istanze VPC su questa sottorete, per garantire che
nella sottorete ci siano abbastanza IP privati liberi. Un gateway VPN ha bisogno di 8 indirizzi IP privati per supportare l'HA e gli aggiornamenti progressivi.

**In quale zona risiede il gateway VPN?**

Il gateway VPN risiede nella zona con la sottorete che hai scelto durante il provisioning. Ricorda che il gateway VPN gestisce solo le istanze VPC presenti nella stessa zona e nello stesso VPC. Pertanto, le istanze VPC in altre zone non possono sfruttare il gateway VPN
per comunicare con una rete privata in loco. Per la tolleranza di errore della zona, devi distribuire un gateway VPN per ogni zona.

**Cosa devo fare se utilizzo gli ACL nella sottorete usata per distribuire il gateway VPN?**

Assicurati che siano in vigore le seguenti regole ACL per consentire il traffico di gestione e il traffico del tunnel VPN:

* **Regole in entrata**
    - Consenti protocollo porta di origine TCP 9091
    - Consenti protocollo porta di origine TCP 10514
    - Consenti protocollo porta di origine TCP 443
    - Consenti protocollo porta di origine TCP 80
    - Consenti protocollo porta di origine TCP 53
    - Consenti protocollo porta di origine UDP 53
    - Consenti protocollo TUTTO l'IP di origine è l'IP pubblico del gateway peer VPN
    - Consenti protocollo porta di destinazione TCP 443
    - Consenti protocollo porta di destinazione TCP 56500
    - Consenti il traffico tra le istanze in VPC e la tua rete privata in loco
    - Consenti il traffico ICMP

* **Regole in uscita**
   - Consenti tutto il traffico

**Cosa devo fare se utilizzo gli ACL o i gruppi di sicurezza nelle sottoreti che devono comunicare con la rete privata in loco?**

Dovrai assicurarti che le regole ACL o le regole del gruppo di sicurezza siano in vigore per consentire il traffico tra le istanze nel tuo VPC e nella tua rete privata in loco.

**VPN for VPC supporta le configurazioni HA?**

Sì, supporta HA in una configurazione Attiva/Standby.

**Esistono dei piani per supportare la VPN SSL?**

No, è supportato solo il site-to-site IPsec.

**Ci sono dei limiti sulla velocità effettiva per VPNaaS site-to-site?**

Supportiamo fino a 650 Mbps di velocità effettiva.

**L'autenticazione PSK e IKE basata sul certificato è supportata per VPNaaS?**

È supportata solo l'autenticazione PSK.

**Puoi utilizzare VPN for VPC come gateway VPN per la tua infrastruttura classica IBM Cloud?**

No, per utilizzare il gateway VPN nel tuo ambiente classico dell'infrastruttura IBM Cloud, devi utilizzare la [VPN IPsec](https://cloud.ibm.com/catalog/infrastructure/ipsec-vpn).
