---


copyright:
  years: 2018, 2019
lastupdated: "2019-06-12"

keywords: load balancer, public, listener, back-end, front-end, pool, round-robin, weighted, connections, methods, policies, APIs, access, ports

subcollection: vpc-on-classic-network

---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:note: .note}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Utilizzo dei programmi di bilanciamento del carico per VPC
{: #--using-load-balancers-in-ibm-cloud-vpc}

Il servizio {{site.data.keyword.cloud}} **Load Balancer for VPC** distribuisce il traffico tra più istanze del server all'interno della stessa regione del tuo VPC.

## Programma di bilanciamento del carico pubblico
{: #public-load-balancer}

Alla tua istanza del servizio del programma di bilanciamento del carico viene assegnato un nome di dominio completo (FQDN) accessibile pubblicamente, che devi utilizzare per accedere alle tue applicazioni ospitate dietro l'IBM Cloud Load Balancer for VPC. Questo nome di dominio può essere registrato con uno o più indirizzi IP pubblici.

Nel tempo, questi indirizzi IP pubblici e il loro numero può venire modificato in base alle attività di manutenzione e ridimensionamento. Le istanze del servizio di backend (VSI) che ospitano la tua applicazione devono essere in esecuzione nella stessa regione e nello stesso VPC.

## Programma di bilanciamento del carico privato
{: #private-load-balancer}

Il programma di bilanciamento del carico privato è accessibile solo ai client interni sulle tue sottoreti private, all'interno della stessa regione e dello stesso VPC. Il programma di bilanciamento del carico privato accetta il traffico solo dagli spazi di indirizzo [RFC1918 ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")](https://tools.ietf.org/html/rfc1918){: new_window}.

In modo simile al programma di bilanciamento del carico pubblico, all'istanza del servizio del programma di bilanciamento del carico privato viene assegnato un nome di dominio completo (FQDN). Tuttavia, questo nome dominio viene registrato con uno o più indirizzi IP privati.

Le operazioni di IBM Cloud possono modificare il numero e il valore dei tuoi indirizzi IP privati assegnati nel tempo, in base alle attività di manutenzione e ridimensionamento. Le istanze del servizio di backend (VSI) che ospitano la tua applicazione devono essere in esecuzione nella stessa regione e nello stesso VPC.

Consulta la seguente tabella per un confronto riepilogativo delle funzioni:

| Funzione | Programma di bilanciamento del carico pubblico | Programma di bilanciamento del carico privato |
|--------|-------|-------|
| Accessibile su internet? |  Sì, con FQDN, internet | No, solo i client interni sulla stessa regione e lo stesso VPC |
| Accetta tutto il traffico? | Sì | No, solo RFC 1918 |
| Numero di IP privati | si modifica nel tempo | si modifica nel tempo |
| Come viene registrato il nome dominio? | indirizzi IP pubblici | indirizzi IP privati |

## Listener di frontend e pool di backend
{: #front-end-listeners-and-back-end-pools}

Puoi definire fino a dieci (10) listener di frontend (porte dell'applicazione) e associarli ai rispettivi pool di backend nei server dell'applicazione di backend. Il nome di dominio completo (FQDN) assegnato alla tua istanza del servizio del programma di bilanciamento del carico e le porte del listener di frontend, vengono esposti a internet pubblicamente. Le richieste utente in entrata vengono ricevute su queste porte.

I protocolli del listener di frontend supportati sono:
* HTTP
* HTTPS
* TCP

I protocolli del pool di backend supportati sono:
* HTTP
* TCP

Il traffico HTTPS in entrata viene terminato nel programma di bilanciamento del carico, per consentire la comunicazione HTTP in testo semplice con il server di backend.

Puoi collegare fino a cinquanta (50) istanze del server a un pool di backend. Ogni istanza del server collegata deve avere una porta configurata. La porta può essere la stessa o meno della porta del listener di frontend.

L'intervallo di porte da 56500 a 56520 è riservato per motivi di gestione; queste porte non possono essere utilizzate come porte del listener di frontend.
{: note}

## Bilanciamento del carico di livello 7
{: #layer-7-load-balancing}

I programmi di bilanciamento del carico pubblici e privati supportano entrambi il bilanciamento del carico di livello 7. Il traffico di dati viene distribuito in base alle politiche e alle regole configurate. Una _politica_ definisce l'azione, ossia la modalità di distribuzione del traffico, quando la richiesta in entrata corrisponde alle regole associate alla politica.

### Politica di livello 7 
{: #layer-7-policy}

Una politica di livello 7 è associata a un listener e solo a un listener HTTP o HTTPS. Ogni politica può avere una serie di regole. La politica viene applicata **solo** quando viene trovata una corrispondenza per tutte le sue regole.

Puoi collegare più di una politica a un listener. In generale, una politica con la priorità più bassa viene valutata per prima. La priorità deve essere univoca per una determinata politica.

Se la richiesta in entrata non corrisponde a alcuna regola della politica, la richiesta viene reindirizzata al pool predefinito del listener, se configurato.

Le azioni supportate per una politica di livello 7 sono le seguenti:

* **Rifiuto:** la richiesta viene negata con una risposta 403.
* **Reindirizzamento:** la richiesta viene reindirizzata a un URL e a un codice di risposta configurati.
* **Inoltro:** la richiesta viene inviata a uno specifico pool di backend.

Le politiche impostate su `reject` vengono valutate per prime, indipendentemente dalla loro priorità.

Successivamente, vengono valutate le politiche impostate su `redirect`.

Infine, vengono valutate le politiche impostate su `forward`.

All'interno di ciascuna categoria di azione, le politiche vengono valutate in ordine crescente di priorità (dalla più bassa alla più alta).

### Proprietà della politica di livello 7
{: #layer-7-policy-properties}

Proprietà  | Descrizione
------------- | -------------
Nome | Il nome della politica. Il nome deve essere univoco all'interno del listener.
Azione | L'azione da intraprendere quando tutte le regole della politica corrispondono. I valori accettabili sono `reject`, `redirect` e `forward`. Una politica con l'azione `reject` viene sempre valutata per prima, a prescindere dalla sua priorità. Le politiche con le azioni `redirect` vengono valutate successivamente, seguite dalle politiche con l'azione `redirect`.
Priorità | Le politiche vengono valutate in base all'ordine crescente di priorità.
URL | L'URL a cui viene reindirizzata la richiesta, se l'azione è impostata su `redirect`.
Codice di stato HTTP | Il codice di stato della risposta restituita dal programma di bilanciamento del carico quando l'azione è impostata su `redirect`. I valori accettabili sono: 301, 302, 303, 307 o 308.
Destinazione | Il pool di backend delle istanze del server a cui viene inoltrata la richiesta, se l'azione è impostata su `forward`.

### Regole di livello 7 
{: #layer-7-rules}

Una regola di livello 7 definisce come deve essere trovata una corrispondenza per una richiesta. Sono supportati tre tipi:

Tipo      |  Descrizione
----------| -----------------------
`hostname` | La richiesta corrisponde al nome host specificato (ad esempio, `api.my_company.com`).
`header`    | La richiesta corrisponde ad un campo di intestazione HTTP (ad esempio, `Cookie`).
`path`      | La richiesta corrisponde al percorso nell'URL (ad esempio, `/index.html`).

Per la corrispondenza di una richiesta, è necessario definire una `condition` in una regola. Sono supportate tre condizioni:

Condizione |  Tipo di valutazione
----------------|---------------------
`contains`        |  Verifica se il campo estratto contiene la stringa fornita.
`equals`        |  Verifica se il campo estratto è identico alla stringa fornita.
`matches_regex`           |  Trova una corrispondenza tra il campo estratto e l'espressione regolare fornita.

## Proprietà della regola di livello 7
{: #layer-7-rule-properties}

Proprietà  | Descrizione
------------- | -------------
Tipo      | Specifica il tipo di regola. I valori accettabili sono `hostname`, `header` o `path`.
Condizione | Specifica la condizione con cui viene valutata una regola. La condizione può essere: `contains`, `equals` o `matches_regex`.
Campo | Specifica il nome del campo di intestazione HTTP. Questo campo è applicabile solo al tipo di regola `header`. Ad esempio, per trovare una corrispondenza con un cookie nell'intestazione HTTP, il campo può essere impostato su `Cookie`.
Valore |  Il valore di cui trovare una corrispondenza.

## Metodi di bilanciamento del carico
{: #load-balancing-methods}

Sono disponibili i seguenti tre metodi di bilanciamento del carico per distribuire il traffico tra i server dell'applicazione di backend:

* **Round-robin:** Round-robin è il metodo di bilanciamento del carico predefinito. Con questo metodo, il programma di bilanciamento del carico
inoltra le connessioni client in entrata in modalità round robin ai server di backend. Di conseguenza, tutti
i server di backend ricevono all'incirca un numero uguale di connessioni client.

* **Round-robin ponderato:** con questo metodo, il programma di bilanciamento del carico inoltra
le connessioni client in entrata ai server di backend in base al peso assegnato a tali server. Ad ogni server viene assegnato un peso predefinito
di 50, che può essere personalizzato con qualsiasi valore compreso tra 0 e 100.

Ad esempio, se tre applicazioni server A, B e C, hanno i pesi personalizzati con 60, 60 e 30 rispettivamente, i server A e B ricevono lo stesso numero di connessioni, mentre il server C ne riceve la metà.

* **Numero minimo di connessioni:** con questo metodo, l'istanza del server che inoltra il numero minimo di connessioni a un ora specificata riceve la successiva connessione client.

**Ulteriori caratteristiche di questi metodi:**

* Reimpostare un peso del server su '0' significa che nessuna nuova connessione viene inoltrata a tale server, ma tutto il traffico esistente continuerà a fluire. L'utilizzo di un peso di '0' può aiutare a terminare un server correttamente e a rimuoverlo dalla rotazione del servizio.
* I valori del peso del server sono applicabili solo con il metodo round-robin ponderato. Vengono ignorati con i metodi di bilanciamento del carico round-robin e numero minimo di connessioni.

## Adattamento orizzontale
{: #horizontal-scaling}

Il programma di bilanciamento del carico regola automaticamente la propria capacità in base al carico. Quando si verifica questa regolazione, potresti vedere una modifica nel numero di indirizzi IP associati al nome DNS del programma di bilanciamento del carico.

## Controlli di integrità
{: #health-checks}

Le definizioni del controllo di integrità sono obbligatorie per i pool di backend.

Il programma di bilanciamento del carico effettua controlli di integrità periodici per monitorare l'integrità delle porte di backend e inoltra il traffico client ad esse di conseguenza. Se una porta del server di backend specificata viene trovata non integra, non viene più inoltrata alcuna nuova connessione ad essa. Il programma di bilanciamento del carico continua a monitorare l'integrità delle porte non integre e riprende ad utilizzarle se diventano nuovamente integre dopo aver superato due tentativi di controllo di integrità consecutivi.

I controlli di integrità per le porte HTTP e TCP vengono eseguiti nel seguente modo:

* **HTTP:** viene inviata una richiesta `HTTP GET` per un URL pre-specificato alla porta del server di backend. La porta del server viene contrassegnata come integra se viene ricevuta una risposta `200 OK`. Il percorso di integrità `GET` predefinito è "/" tramite l'IU e può essere personalizzato.

* **TCP:** il programma di bilanciamento del carico tenta di aprire una connessione TCP con il server di backend su una porta TCP specificata. La porta del server viene contrassegnata come integra se il tentativo di connessione ha esito positivo e la connessione viene chiusa.

L'intervallo del controllo di integrità è 5 secondi, il timeout predefinito per una richiesta del controllo di integrità è 2 secondi e il numero di nuovi tentativi predefinito è 2.
{: note}

## Offload SSL e autorizzazioni necessarie
{: #ssl-offloading-and-required-authorizations}

Per tutte le connessioni HTTPS in entrata, il servizio del programma di bilanciamento del carico termina la connessione SSL e stabilisce una comunicazione HTTP in testo semplice con l'istanza del server di backend. Con questa tecnica, gli handshake SSL con uso intensivo di CPU e le attività di crittografia/decrittografia sono stati spostati dalle istanze del server di backend, consentendo loro di utilizzare i propri cicli di CPU per elaborare il traffico dell'applicazione.

Un certificato SSL è obbligatorio al programma di bilanciamento del carico per eseguire le attività di offload SSL. Puoi gestire i certificati SSL tramite il [IBM Certificate Manager ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window}.

Per consentire a un programma di bilanciamento del carico di accedere ai tuoi certificati SSL, devi abilitare l'**autorizzazione da-servizio-a-servizio**, che concede alla tua istanza del servizio del programma di bilanciamento del carico l'accesso alla tua istanza Certificate Manager. Puoi gestire una simile autorizzazione tramite questa documentazione [Concessione dell'accesso tra i servizi ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")](/docs/iam?topic=iam-serviceauth#create-auth){: new_window}. Assicurati di scegliere **Infrastruttura VPC** come servizio di origine, **Load Balancer for VPC** come tipo di risorsa, **Certificate Manager** come servizio di destinazione e assegna il ruolo di accesso al servizio **Scrittore**.

Se viene rimossa l'autorizzazione richiesta, si potrebbero verificare degli errori con il tuo programma di bilanciamento del carico.
{: note}

## Identity and access management (IAM)
{: #identity-and-access-management-iam}

Puoi configurare le politiche di accesso per un'istanza **Load Balancer for VPC**. Per gestire le tue politiche di accesso utente, visita [Gestione dell'accesso alle risorse ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")](/docs/services/iam?topic=iam-iammanidaccser#resourceaccess){: new_window} per ulteriori informazioni su Identity and access management.

### Configurazione delle politiche di accesso al gruppo di risorse per gli utenti
{: #configuring-resource-group-access-policies-for-users}

Per creare un programma di bilanciamento del carico, dovrai accedere a un gruppo di risorse. L'utente che sta creando un programma di bilanciamento del carico deve avere l'accesso appropriato al gruppo di risorse fornito o al gruppo di risorse predefinito, se non ne viene fornito alcuno.

1. Passa a **Gestisci > Account > Utenti**. Visualizzerai un elenco di utenti con accesso al tuo account IBM Cloud.
2. Seleziona il nome dell'utente a cui vuoi assegnare una politica di accesso. Se l'utente non viene visualizzato, fai clic su **Invita utenti** per aggiungerlo al tuo account IBM Cloud.
3. Seleziona **Assegna accesso**.
4. Seleziona **Assegna l'accesso in un gruppo di risorse**.
5. Dall'elenco a discesa **Gruppo di risorse**, seleziona il gruppo di risorse desiderato.
6. Dall'elenco a discesa **Assegna accesso a un gruppo di risorse**, seleziona l'accesso desiderato.
7. Dall'elenco a discesa **Servizi**, seleziona i servizi desiderati.
9. Seleziona **Assegna** per assegnare la politica di accesso al gruppo di risorse all'utente.

### Configurazione delle politiche di accesso alla risorsa per gli utenti
{: #configuring-resource-access-policies-for-users}

| Ruolo di accesso alla piattaforma | Azione del programma di bilanciamento del carico |
|-------------|-----|
| Amministratore | Creare/Visualizzare/Modificare/Eliminare il programma di bilanciamento del carico |
| Editor | Creare/Visualizzare/Modificare/Eliminare il programma di bilanciamento del carico |
| Visualizzatore | Visualizzare il programma di bilanciamento del carico |

1. Passa a **Gestisci > Account > Utenti**. Visualizzerai l'elenco di utenti con accesso al tuo account IBM Cloud.
2. Seleziona il nome dell'utente a cui vuoi assegnare una politica di accesso. Se l'utente non viene visualizzato, fai clic su **Invita utenti** per aggiungerlo al tuo account IBM Cloud.
3. Seleziona **Assegna accesso**.
4. Seleziona **Assegna l'accesso alle risorse**.
5. Dall'elenco a discesa **Servizi**, seleziona **Infrastruttura VPC**.
6. Dall'elenco a discesa **Tipo di risorsa**, seleziona **Load Balancer for VPC**.
7. Dall'elenco a discesa **Load Balancer ID**, seleziona un ID dell'istanza del programma di bilanciamento del carico o utilizza il valore predefinito **All load balancers**.
8. Assegna un ruolo di accesso alla piattaforma all'utente.
9. Seleziona **Assegna** per assegnare la politica di accesso all'utente.

## Integrazione di Activity Tracker
{: #activity-tracker-integration}

Il servizio del programma di bilanciamento del carico è integrato con **IBM Cloud Activity Tracker with LogDNA**, che registra gli eventi, in modo conforme allo standard CADF, attivati dalle attività avviate dall'utente che modificano lo stato del servizio nel cloud.

Per un elenco dettagliato delle azioni registrate come eventi di controllo sulle istanze del servizio del programma di bilanciamento del carico, fai riferimento a [Eventi di Activity Tracker with LogDNA](/docs/vpc-on-classic?topic=vpc-on-classic-at-events#events-load-balancers).

Tutti gli eventi di controllo vengono registrati in "IBM Cloud Activity Tracker with LogDNA" nella regione `us-south`. Non importa in quale regione viene eseguito il provisioning del servizio del programma di bilanciamento del carico.
{:note}

Per visualizzare gli eventi, devi eseguire il provisioning di un'istanza "IBM Cloud Activity Tracker with LogDNA" nella regione `us-south` con il tuo account. Gli utenti nel tuo account devono disporre di una politica IAM che conceda il ruolo di accesso alla piattaforma **Visualizzatore** e il ruolo di accesso al servizio **Lettore** sull'istanza "IBM Cloud Activity Tracker with LogDNA".

Ulteriori informazioni sulla concessione dell'accesso sono disponibili all'indirizzo [Concessione delle autorizzazioni per visualizzare gli eventi. ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")](https://cloud.ibm.com/docs/services/cloud-activity-tracker/how-to/grant_permissions.html#grant_permissions){: new_window}

Gli utenti di IBM Cloud possono monitorare le operazioni a livello di account eseguite sul servizio del programma di bilanciamento del carico.
{: tip}

Utilizza questa procedura per eseguire il provisioning di un'istanza "IBM Cloud Activity Tracker with LogDNA" nel tuo account:

1. Accedi alla console IBM Cloud. [Accedi alla console IBM Cloud. ![Icona link esterno](../icons/launch-glyph.svg "Icona link esterno")](https://cloud.ibm.com/){: new_window}
2. Fai clic sull'icona ![Icona a tre linee](../../icons/icon_hamburger.svg) in alto a sinistra. Da lì, seleziona **Osservabilità > Activity Tracker**.
3. In alto a destra, fai clic su **Crea istanza**.
4. Definisci un nome servizio.
5. Seleziona `us-south` come regione e scegli il gruppo di risorse
6. Se hai un account a pagamento, scegli un piano diverso da `lite`.
7. Fai clic su **Crea**.

Questo è il messaggio del programma di traccia dell'attività di esempio per un'operazione **Create Listener**:
```bash
{
    "logSourceCRN": "crn:v1:bluemix:public:is:eu-gb:a/<ACCOUNT_ID>::load-balancer:4633518f-8aac-48a1-a694-d15ee6bd70e3",
    "meta": {
        "serviceProviderName": "is-load-balancer",
        "serviceProviderProjectId": "48a7a7b7-6642-4aa1-8af9-c1be4ef82050",
        "serviceProviderRegion": "ng",
        "userAccountIds": [
            <ACCOUNT_ID>
        ]
    },
    "payload": {
        "action": "is.load-balancer.load-balancer.listener.create",
        "eventTime": "2019-05-30T18:42:48.96+0000",
        "eventType": "activity",
        "id": "e4ee1906d01a35efe8bd8303ce0a734e",
        "initiator": {
            "credential": {
                "type": "token"
            },
            "host": {
                "address": <CLIENT_IP>,
                "agent": "python-requests/2.19.1"
            },
            "id": <USER-ID>,
            "name": <USER_ID>,
            "project_id": <ACCOUNT_ID>,
            "typeURI": "service/security/account/user"
        },
        "message": "is.load-balancer: create listener 4633518f-8aac-48a1-a694-d15ee6bd70e3 success",
        "observer": {
            "id": "activity-tracker.ng.bluemix.net",
            "name": "ActivityTracker",
            "typeURI": "security/edge/activity-tracker"
        },
        "outcome": "success",
        "reason": {
            "reasonCode": 201,
      "reasonType": "https://www.iana.org/assignments/http-status-codes/http-status-codes.xml"
        },
        "requestData": "{\"headers\":{\"RayID\":\"4df2d9911a3ac2bd\"},\"extraData\":{\"resourceName\":\"4633518f-8aac-48a1-a694-d15ee6bd70e3\"}}",
        "requestPath": "/v1/load_balancers/4633518f-8aac-48a1-a694-d15ee6bd70e3/listeners",
        "severity": "normal",
        "target": {
            "host": {
                "address": <API_END_POINT>
            },
            "id": "crn:v1:bluemix:public:is:eu-gb:a/<ACCOUNT_ID>::load-balancer:4633518f-8aac-48a1-a694-d15ee6bd70e3",
            "name": "4633518f-8aac-48a1-a694-d15ee6bd70e3",
            "typeURI": "/v1/load_balancers/4633518f-8aac-48a1-a694-d15ee6bd70e3/listeners"
        },
        "typeURI": "http://schemas.dmtf.org/cloud/audit/1.0/event"
    },
    "saveServiceCopy": true
}
```

## API disponibili
{: #lbaas-apis-available}

Per effettuare le chiamate API, devi utilizzare una sorta di client REST. Ad esempio, puoi utilizzare il comando `curl` per richiamare tutti i programmi di bilanciamento del carico esistenti:

```
curl -X GET "$rias_endpoint/v1/load_balancers?version=2019-05-31&generation=1" -H "Authorization: $iam_token"
```
{: pre}

La seguente sezione ti fornisce i dettagli sulle API che puoi utilizzare per i programmi di bilanciamento del carico nel tuo ambiente VPC. Per la specifica completa, vedi la [Guida di riferimento API VPC on Classic](https://{DomainName}/apidocs/vpc-on-classic#list-all-load-balancers).

| Descrizione | API |
|-------------|-----|
| Crea ed esegue il provisioning di un programma di bilanciamento del carico | POST /load_balancers |
| Richiama tutti i programmi di bilanciamento del carico | GET /load_balancers |
| Richiama un programma di bilanciamento del carico | GET /load_balancers/{id} |
| Elimina un programma di bilanciamento del carico | DELETE /load_balancers/{id} |
| Aggiorna un programma di bilanciamento del carico | PATCH /load_balancers/{id} |
| Crea un listener | POST /load_balancers/{id}/listeners |
| Richiama tutti i listener del programma di bilanciamento del carico | GET /load_balancers/{id}/listeners |
| Richiama un listener | GET /load_balancers/{id}/listeners/{listener_id} |
| Elimina un listener | DELETE /load_balancers/{id}/listeners/{listener_id} |
| Aggiorna un listener | PATCH /load_balancers/{id}/listeners/{listener_id} |
| Crea un pool | POST /load_balancers/{id}/pools |
| Richiama tutti i pool del programma di bilanciamento del carico | GET /load_balancers/{id}/pools |
| Richiama un pool | GET /load_balancers/{id}/pools/{pool_id} |
| Elimina un pool | DELETE /load_balancers/{id}/pools/{pool_id} |
| Aggiorna un pool | PATCH /load_balancers/{id}/pools/{pool_id} |
| Crea un membro | POST /load_balancers/{id}/pools/{pool_id}/members |
| Richiama tutti i membri del pool | GET /load_balancers/{id}/pools/{pool_id}/members |
| Richiama un membro |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Elimina un membro dal pool | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Aggiorna un membro | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Aggiorna i membri del pool | PUT /load_balancers/{id}/pools/{pool_id}/members |
| Richiama le statistiche di un programma di bilanciamento del carico | GET /load_balancers/{id}/statistics |
| Richiama tutte le politiche del listener |  GET /load_balancers/{id}/listeners/{listener_id}/policies
| Crea una politica per il listener | POST /load_balancers/{id}/listeners/{listener_id}/policies
| Elimina una politica dal listener | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Richiama una politica del listener | GET /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Aggiorna una politica del listener | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Richiama tutte le regole associate a una politica | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| Crea una regola per la politica | POST /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| Elimina una regola dalla politica | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| Richiama una regola dalla politica | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| Aggiorna una regola della politica | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}

## Esempio di programma di bilanciamento del carico
{: #load-balancer-example}

Nel seguente esempio, utilizzerai l'API per creare un programma di bilanciamento del carico davanti a 2 istanze del server VPC (`192.168.100.5` e `192.168.100.6`) eseguendo un'applicazione web in ascolto sulla porta `80`. Il programma di bilanciamento del carico ha un listener di frontend che consente l'accesso all'applicazione web in modo sicuro tramite HTTPS. Successivamente, puoi utilizzare l'API per ottenere i dettagli dell'istanza del programma di bilanciamento del carico dopo averla creata e per eliminarla.

### Istruzioni di esempio
{: #lbaas-example-steps}

Le seguenti istruzioni di esempio saltano i passi preliminari dell'utilizzo dell'[IU IBM Cloud](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console), della [CLI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli) o dell'[API VPC on Classic](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis) per eseguire il provisioning di un VPC, di sottoreti e di istanze.

Le istruzioni di esempio del programma di bilanciamento del carico possono anche essere eseguite utilizzando la [CLI](/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference).
{: note}

**Passo 1. Crea un programma di bilanciamento del carico con il listener, il pool e le istanze del server collegato (membri del pool)**

```bash
curl -H "Authorization: $iam_token" -X POST
"$rias_endpoint/v1/load_balancers?version=2019-05-31&generation=1" \
    -d '{
        "name": "example-balancer",
        "is_public": true,
        "listeners": [
            {
                "certificate_instance": {
                    "crn": "crn:v1:bluemix:public:cloudcerts:us-south:a/123456:b8877ea4-b8eg-467e-912a-da1eb7f031cg:certificate:43219c4c97d013fb2a95b21dddde1234"
                },
                "port": 443,
                "protocol": "https",
                "default_pool": {
                    "name": "example-pool"
                }
            }
        ],
        "pools": [
            {
                "algorithm": "round_robin",
                "health_monitor": {
                    "delay": 5,
                    "max_retries": 2,
                    "timeout": 2,
                    "type": "http",
                    "url_path": "/"
                },
                "name": "example-pool",
                "protocol": "http",
                "session_persistence": {
                    "cookie_name": "string",
                    "type": "source_ip"
                },
                "members": [
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.5"
                        },
                        "weight": 50
                    },
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.6"
                        },
                        "weight": 50
                    }
                ]
            }
        ],
        "subnets": [
            {
                "id": "7ec87131-1c7e-4990-b4f0-a26f2e61f98e"
            }
        ]
        }'
```
{: codeblock}

Output di esempio:
```
{
    "created_at": "2018-07-12T23:17:07.5985381Z",
    "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "hostname": "ac34687d.lb.appdomain.cloud",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "is_public": true,
    "listeners": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
        }
    ],
    "name": "example-balancer",
    "operating_status": "offline",
    "pools": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
            "name": "example-pool"
        }
    ],
    "provisioning_status": "create_pending",
    "resource_group": {
        "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
    },
    "subnets": [
        {
            "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "name": "example-subnet"
        }
    ]
}
```
{: screen}

Salva l'ID del programma di bilanciamento del carico da utilizzare nei prossimi passi, ad esempio, salvalo nella variabile `lbid`.

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

**Passo 2. Ottieni un programma di bilanciamento del carico**

```
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

Concedi un po' di tempo al provisioning. Quando il programma di bilanciamento del carico è pronto, lo stato sarà impostato su `online` e `active`, come vedrai nel seguente output di esempio:

Output di esempio:

```bash
{
  "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "name": "example-balancer",
  "created_at": "2018-07-13T22:22:24.489Z",
  "hostname": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud",
  "is_public": true,
  "listeners": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
    }
  ],
  "operating_status": "online",
  "pools": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
      "name": "example-pool"
    }
  ],
  "private_ips": [
    {
      "address": "192.168.10.5"
    },
    {
      "address": "192.168.10.6"
    }
  ],
  "provisioning_status": "active",
  "public_ips": [
    {
        "address": "169.11.111.115"
    },
    {
        "address": "169.11.111.116"
    }
  ],
  "resource_group": {
    "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
  },
  "subnets": [
    {
      "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "name": "example-subnet"
    }
  ]
}
```
{: screen}

**Passo 3. Elimina un programma di bilanciamento del carico**

```bash
curl -H "Authorization: $iam_token" -X DELETE "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

## Esempi di Livello 7: crea politiche e regole
{: #layer-7-examples-create-policy-and-rules}

I seguenti due esempi forniscono i passi che mostrano come vengono create le politiche e le regole e come vengono associate a un listener.

### Esempio 1: crea un listener HTTPS con politiche e regole. Le politiche vengono create con l'azione `Redirect`

```
bash
curl -H "Authorization: $iam_token" -X POST
"$rias_endpoint/v1/load_balancers/$lbId/listeners" \
    -d '{
            "certificate_instance": {
                "crn": "crn:v1:bluemix:public:cloudcerts:us-south:a/1111111111111111111111111111:22222222-3333-4444-5555-666666666666:certificate:77777777777777777777777777777777"
            },
            "connection_limit": 2000,
            "port": 443,
            "protocol": "https",
            "policies": [
                {
                    "name": "hostname_header",
                    "action": "redirect",
                    "priority": 1,
                    "target": {
                        "url": "https://www.examples.com/",
                        "http_status_code": 307
                    },
                    "rules": [
                        {
                            "condition": "contains",
                            "type": "header",
                            "field": "aheader",
                            "value": "avalue"
                        },
                        {
                            "condition": "equals",
                            "type": "hostname",
                            "value": "abc.com"
                        }
                    ]
                },
                {
                    "name": "header_cookie",
                    "action": "redirect",
                    "priority": 5,
                    "target": {
                        "url": "https://www.mycookies.com/",
                        "http_status_code": 302
                    },
                    "rules": [
                        {
                            "condition": "contains",
                            "type": "header",
                            "field": "aheader",
                            "value": "avalue"
                        },
                        {
                            "condition": "equals",
                            "type": "header",
                            "field": "cookie",
                            "value": "flavor=oatmeal"
                        }
                    ]
                },
                {
                    "name": "path_hostname",
                    "action": "redirect",
                    "priority": 10,
                    "target": {
                        "url": "https://www.myexamples.com/",
                        "http_status_code": 301
                    },
                    "rules": [
                        {
                            "condition": "contains",
                            "type": "hostname",
                            "value": "abc"
                        },
                        {
                            "condition": "equals",
                            "value": "/test",
                            "type": "path"
                          }
                    ]
                }
            ]
        }'
```
{: codeblock}

### Esempio 2: crea politiche con l'azione di inoltro (`Forward`) ai pool e associale a un listener esistente
```
bash
curl -H "Authorization: $iam_token" -X POST
"$rias_endpoint/v1/load_balancers/$lbId/listeners/$listenerId/policies" \
    -d '{
            "policies": [
                {
                    "action": "forward",
                    "priority": 1,
                    "target": {
                        "id": "7df616da-4dd6-43d3-881d-801ae29e29fe"
                    },
                    "rules": [
                        {
                            "condition": "equals",
                            "type": "header",
                            "field": "cookie",
                            "value": "flavor=oatmeal"
                        }
                    ]
                },
                {
                    "action": "forward",
                    "priority": 5,
                    "target": {
                        "id": "8061c411-0d50-4c79-b475-102666796434"
                    },
                    "rules": [
                        {
                            "condition": "contains",
                            "type": "header",
                            "field": "aheader",
                            "value": "avalue"
                        }
                    ]
                },
                {
                    "action": "forward",
                    "priority": 10,
                    "target": {
                        "id": "62914e09-3928-4d89-b7f7-1bb7a6d7fe85"
                    },
                    "rules": [
                        {
                            "condition": "matches_regex",
                            "type": "hostname",
                            "value": "abc[a-z]*.com"
                        }
                    ]
                },
                {
                    "action": "forward",
                    "priority": 6,
                    "target": {
                        "id": "62914e09-3928-4d89-b7f7-1bb7a6d7fe85"
                    },
                    "rules": [
                        {
                            "condition": "equals",
                            "type": "path",
                            "value": "/test/testtest"
                        }
                    ]
                }
            ]
        }'
```
{: codeblock}

## Domande frequenti
{: #load-balancer-faqs}

Questa sezione contiene le risposte ad alcune domande frequenti sul servizio **Load Balancer for VPC**.

### Posso utilizzare un nome DNS diverso per il mio programma di bilanciamento del carico?
{: #can-i-use-a-different-dns-name-for-my-load-balancer}

Il nome DNS assegnato automaticamente per il programma di bilanciamento del carico non è personalizzabile. Tuttavia, puoi aggiungere un record del nome canonico (CNAME) che faccia puntare il tuo nome DNS preferito al nome DNS del programma di bilanciamento del carico assegnato automaticamente. Ad esempio, il tuo programma di bilanciamento del carico in `us-south` ha l'ID `dd754295-e9e0-4c9d-bf6c-58fbc59e5727`, il nome DNS assegnato automaticamente al programma di bilanciamento del carico è `dd754295-us-south.lb.appdomain.cloud`. Il tuo nome DNS preferito è `www.myapp.com`. Puoi aggiungere un record CNAME (tramite il provider DNS che utilizzi per gestire `myapp.com`) che faccia puntare `www.myapp.com` al nome DNS del programma di bilanciamento del carico `dd754295-us-south.lb.appdomain.cloud`.

### Qual è il numero massimo di listener di frontend che posso definire con il mio programma di bilanciamento del carico?
{: #what-s-the-maximum-number-of-front-end-listeners-i-can-define-with-my-load-balancer}

10.

### Qual è il numero massimo di istanze del server che posso collegare al mio pool di backend?
{: #what-s-the-maximum-number-of-server-instances-i-can-attach-to-my-back-end-pool}

50.

### Il programma di bilanciamento del carico è scalabile orizzontalmente?
{: #is-the-load-balancer-horizontally-scalable}

Sì. Il programma di bilanciamento del carico modifica automaticamente la propria capacità in base al carico. Quando viene eseguito l'adattamento orizzontale, il numero di indirizzi IP associati al nome DNS del programma di bilanciamento del carico sarà modificato.

### Cosa devo fare se sto utilizzando degli ACL o dei gruppi di sicurezza sulle sottoreti utilizzate per distribuire il programma di bilanciamento del carico?
{: #what-should-i-do-if-i-am-using-acls-or-security-groups-on-the-subnets-that-are-used-to-deploy-the-load-balancer}

Dovrai assicurarti che siano in vigore le regole del gruppo di sicurezza o dell'ACL appropriate per consentire il traffico in entrata per le porte di gestione e del listener configurate (porte da 56500 a 56520). Deve essere consentito anche il traffico tra il programma di bilanciamento del carico e le istanze di backend.

### Perché sto ricevendo un messaggio di errore: `certificate instance not found`?

* Il CRN dell'istanza del certificato potrebbe non essere valido.
* Potresti non aver concesso l'**autorizzazione da-servizio-a-servizio**. Consulta la sezione **Offload SSL** di questo documento.

### Perché ricevo il codice `401 Unauthorized Error`?

Controlla le seguenti politiche di accesso per il tuo utente:
* La politica di accesso per il tipo di risorse del programma di bilanciamento del carico
* La politica di accesso per il gruppo di risorse
* Se vengono utilizzati i listener `HTTPS`, controlla anche l'autorizzazione da-servizio-a-servizio per l'istanza Gestore Certificato.

### Perché il mio programma di bilanciamento del carico è nello stato `maintenance_pending`?

Il programma di bilanciamento del carico sarà nello stato `maintenance_pending` durante molte attività di manutenzione come ad esempio:
* Attività di adattamento orizzontale
* Attività di ripristino
* Aggiornamento progressivo per risolvere le vulnerabilità e applicare le patch di sicurezza

### Perché devo scegliere più sottoreti durante il provisioning?
{: #why-do-I-need-to-choose-multiple-subnets-during-provisioning}

**Load Balancer for VPC** è pronto per MZR (Multi Zone Region). Le applicazioni del programma di bilanciamento del carico sono distribuite alle sottoreti che hai selezionato. Si consiglia vivamente di scegliere le sottoreti da zone differenti, in modo che venga fornita l'alta disponibilità e la ridondanza.

### Perché l'integrità del membro di backend nel mio pool è sconosciuta (`unknown`)?

* Il pool non è associato ad alcun listener
* Potrebbero essere state apportate delle modifiche alla configurazione del pool o dei suoi listener associati

### Quale versione TLS è supportata con l'offload SSL?
{: #which-tls-version-is-supported-with-ssl-offload}

**Load Balancer for VPC** supporta TLS 1.2 con terminazione SSL.

Il seguente elenco mostra le cifrature supportate (elencate in ordine di precedenza):
* ECDHE-RSA-AES256-GCM-SHA384
* ECDHE-RSA-AES256-SHA384
* AES256-GCM-SHA384
* AES256-SHA256
* ECDHE-RSA-AES128-GCM-SHA256
* ECDHE-RSA-AES128-SHA256
* AES128-GCM-SHA256
* AES128-SHA256

### Quali sono le impostazioni predefinite e i valori consentiti per i parametri del controllo di integrità?
{: #what-are-the-default-settings-and-allowed-values-for-health-check-parameters}

Le impostazioni predefinite e i valori consentiti sono elencati di seguito:
* Intervallo controllo di integrità: il valore predefinito è 5 secondi, l'intervallo è compreso tra 2 e 60 secondi.
* Timeout risposta controllo di integrità: il valore predefinito è 2 secondi, l'intervallo è compreso tra 1 e 59 secondi.
* Numero massimo di nuovi tentativi: il valore predefinito è 2 nuovi tentativi e l'intervallo è compreso tra 1 e 10 nuovi tentativi.

Il valore del timeout della risposta del controllo di integrità deve essere sempre inferiore al valore dell'intervallo del controllo di integrità.
{:note}

### Gli indirizzi IP del programma di bilanciamento del carico sono fissi?
{: #are-the-load-balancer-ip-addresses-fixed}

Non è garantito che gli indirizzi IP del programma di bilanciamento del carico siano fissi, a causa dell'elasticità integrata del servizio. Durante l'adattamento orizzontale, vedrai delle modifiche negli IP disponibili associati al FQDN del tuo programma di bilanciamento del carico.

Utilizza FQDN, invece degli indirizzi IP memorizzati nella cache.
{:note}

### Il programma di bilanciamento del carico supporta il passaggio al livello 7?
{: #does-the-load-balancer-support-layer-7-switching}

Sì.

### Perché la creazione o l'aggiornamento del listener HTTPS mi comunica che il mio certificato non è valido?
{: #why-does-https-listener-creation-or-update-tell-me-that-my-certificate-is-invalid}

Controlla queste possibilità:

* Il CRN del certificato fornito potrebbe non essere valido.
* L'istanza del certificato fornita nel Gestore certificato potrebbe non avere una chiave privata associata.
