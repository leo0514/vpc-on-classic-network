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

# Lastausgleichsfunktionen für VPC verwenden
{: #--using-load-balancers-in-ibm-cloud-vpc}

Der {{site.data.keyword.cloud}}-Service **Load Balancer for VPC** verteilt den Datenverkehr auf mehrere Serverinstanzen in derselben Region Ihres VPC. 

## Öffentliche Lastausgleichsfunktion
{: #public-load-balancer}

Ihrer Instanz des Lastausgleichsfunktionsservice ist ein öffentlich zugänglicher, vollständig qualifizierter Domänenname (FQDN, Fully Qualified Domain Name) zugewiesen, den Sie für den Zugriff auf Ihre Anwendungen verwenden müssen, die hinter IBM Cloud Load Balancer for VPC gehostet werden. Dieser Domänenname kann mit einer oder mit mehreren öffentlichen IP-Adressen registriert sein.

Im Rahmen von Wartungs- und Skalierungsaktivitäten können sich diese öffentlichen IP-Adressen und die Anzahl der öffentlichen IP-Adressen im Laufe der Zeit ändern. Die Back-End-Serverinstanzen (VSIs), auf denen Ihre Anwendung per Hosting bereitgestellt wird, müssen in derselben Region und unter derselben VPC ausgeführt werden.

## Private Lastausgleichsfunktion
{: #private-load-balancer}

Auf die private Lastausgleichsfunktion können nur interne Clients in Ihren privaten Teilnetzen zugreifen, die sich in derselben Region und VPC befinden. Die private Lastausgleichsfunktion akzeptiert nur Datenverkehr aus [RFC1918-Adressräumen ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")](https://tools.ietf.org/html/rfc1918){: new_window}.

Wie bei einer öffentlichen Lastausgleichsfunktion ist auch der Instanz Ihres privaten Lastausgleichsfunktionsservice ein vollständig qualifizierter Domänenname (FQDN) zugewiesen. Dieser Domänenname ist jedoch für eine oder mehrere private IP-Adressen registriert.

Durch IBM Cloud-Operationen werden Anzahl und Wert der zugewiesenen privaten IP-Adressen im Laufe der Zeit möglicherweise geändert, wenn Wartungs- und Skalierungsaktivitäten stattfinden. Die Back-End-Serverinstanzen (VSIs), auf denen Ihre Anwendung per Hosting bereitgestellt wird, müssen in derselben Region und unter derselben VPC ausgeführt werden.

In der folgenden Tabelle ist ein Vergleich der Features zusammengefasst:

| Feature | Öffentliche Lastausgleichsfunktion | Private Lastausgleichsfunktion |
|--------|-------|-------|
| Zugriff über das Internet möglich? |  Ja, mit FQDN, Internet | Nein, nur interne Clients in derselben Region und VPC |
| Gesamten Datenverkehr akzeptieren? | Ja | Nein, nur RFC 1918 |
| Anzahl privater IU-Adressen | Variiert im Laufe der Zeit | Variiert im Laufe der Zeit |
| Wie wird ein Domänenname registriert? | Öffentliche IP-Adressen | Private IP-Adressen |

## Front-End-Listener und Back-End-Pools
{: #front-end-listeners-and-back-end-pools}

Sie können bis zu 10 Front-End-Listener (Anwendungsports) definieren und diese den jeweiligen Back-End-Pools auf den Back-End-Anwendungsservern zuordnen. Der vollständig qualifizierte Domänenname (FQDN), der Ihrer Instanz des Lastausgleichsfunktionsservice zugewiesen ist, und die Front-End-Listener-Ports sind für das öffentliche Internet verfügbar. An diesen Ports werden eingehende Benutzeranforderungen empfangen.

Die folgenden Front-End-Listener-Protokolle werden unterstützt:
* HTTP
* HTTPS
* TCP

Die folgenden Back-End-Pool-Protokolle werden unterstützt:
* HTTP
* TCP

Eingehender HTTPS-Datenverkehr wird an der Lastausgleichsfunktion beendet, um die HTTP-Kommunikation in unverschlüsseltem Text (Klartext) mit dem Back-End-Server zu ermöglichen.

An einen Back-End-Pool können bis zu 50 Serverinstanzen angeschlossen werden. Für jede angeschlossene Serverinstanz muss ein Port konfiguriert sein. Der Port kann mit dem Front-End-Listener-Port übereinstimmen, muss dies aber nicht.

Der Portbereich von 56500 bis 56520 ist für Verwaltungszwecke reserviert. Diese Ports können nicht für Front-End-Listener-Ports verwendet werden.
{: note}

## Layer-7-Lastausgleich
{: #layer-7-load-balancing}

Sowohl öffentliche als auch private Lastausgleichsfunktionen unterstützen den Layer-7-Lastausgleich. Der Datenverkehr wird auf der Basis von konfigurierten Richtlinien und Regeln verteilt. Eine _Richtlinie_ definiert die Aktion, d. h. wie der Datenverkehr verteilt wird, wenn die eingehende Anforderung mit den Regeln übereinstimmt, die der Richtlinie zugeordnet sind.

### Layer-7-Richtlinie
{: #layer-7-policy}

Eine Layer-7-Richtlinie ist einem Listener - und nur einem HTTP- oder HTTPS-Listener - zugeordnet. Jede Richtlinie kann einen Richtliniensatz haben. Die Richtlinie wird **nur** angewendet, wenn alle zugehörigen Regeln abgeglichen werden. 

Sie können einem Listener mehr als eine Richtlinie zuordnen. Im Allgemeinen wird zuerst eine Richtlinie mit der niedrigsten Priorität ausgewertet. Die Priorität muss für eine bestimmte Richtlinie eindeutig sein.

Wenn die eingehende Anforderung keiner der Richtlinienregeln entspricht, wird die Anforderung an den Standardpool des Listeners weitergeleitet, wenn der Standardpool konfiguriert ist.

Dies sich die unterstützten Aktionen für eine Layer-7-Richtlinie:

* **Reject:** Die Anforderung wird mit der Antwort 403 verweigert.
* **Redirect:** Die Anforderung wird an einen konfigurierten URL- und Antwortcode weitergeleitet.
* **Forward:** Die Anforderung wird an einen bestimmten Back-End-Pool gesendet.

Richtlinien, für die `Ablehnen` festgelegt ist, werden zuerst ausgewertet, unabhängig von ihrer Priorität. 

Anschließend werden Richtlinien ausgewertet, für die `redirect` festgelegt ist. 

Schließlich werden die Richtlinien ausgewertet, die auf `forward` festgelegt sind.

Innerhalb jeder Aktionskategorie werden die Richtlinien in aufsteigender Prioritätsfolge (niedrigste bis höchste Priorität) ausgewertet. 

### Layer-7-Richtlinieneigenschaften
{: #layer-7-policy-properties}

Eigenschaft  | Beschreibung
------------- | -------------
Name | Der Name der Richtlinie. Der Name muss innerhalb des Listeners eindeutig sein.
Aktion | Die Aktion, die ausgeführt werden soll, wenn alle Richtlinienregeln übereinstimmen. Die zulässigen Werte sind `reject`, `redirect` und `forward`. Eine Richtlinie mit der Aktion `Zurückweisen` wird immer zuerst ausgewertet, unabhängig von ihrer Priorität. Richtlinien mit `redirect`-Aktionen werden als Nächstes ausgewertet, gefolgt von Richtlinien mit der Aktion `redirect`.
Priorität | Richtlinien werden basierend auf der aufsteigenden Prioritätsfolge ausgewertet. 
URL | Die URL, an die der Datenverkehr weitergeleitet wird, wenn die Aktion `redirect` lautet.
HTTP-Statuscode | Statuscode der Antwort, die von der Lastausgleichsfunktion zurückgegeben wird, wenn die Aktion auf `redirect` festgelegt ist. Die zulässigen Werte sind: 301, 302, 303, 307 oder 308.
Ziel| Der Back-End-Pool von Serverinstanzen, an die die Anforderung weitergeleitet wird, wenn die Aktion auf `forward` festgelegt ist.

### Layer-7-Regeln
{: #layer-7-rules}

Eine Layer-7-Regel definiert, wie eine Anforderung abgeglichen werden soll. Drei Typen werden unterstützt:

Typ      |  Beschreibung
----------| -----------------------
`hostname` | Die Anforderung stimmt mit dem angegebenen Hostnamen überein (z. B. `api.my_company.com`).
`header`    | Die Anforderung stimmt mit einem HTTP-Headerfeld überein (zum Beispiel `Cookie`).
`path`      | Die Anforderung stimmt mit dem Pfad in der URL überein (zum Beispiel `/index.html`).

Um eine Anforderung abzugleichen, muss `condition` in einer Regel definiert werden. Es werden drei Bedingungen unterstützt:

Bedingung |  Art der Auswertung
----------------|---------------------
`contains`        | Überprüfen Sie, ob das extrahierte Feld die angegebene Zeichenfolge enthält.
`equals`        | Überprüfen, ob das extrahierte Feld mit der bereitgestellten Zeichenfolge identisch ist.
`matches_regex`           | Gleichen Sie das extrahierte Feld mit dem angegebenen regulären Ausdruck ab.

## Layer-7-Regeleigenschaften
{: #layer-7-rule-properties}

Eigenschaft  | Beschreibung
------------- | -------------
Typ | Gibt den Typ der Regel an. Die zulässigen Werte sind `hostname`, `header` oder `path`.
Bedingung | Gibt die Bedingung an, mit der eine Regel ausgewertet wird. Die Bedingung kann sein: `contains`, `equals` oder `matches_regex`.
Feld | Gibt den Namen des HTTP-Headerfelds an. Dieses Feld ist nur für den Regeltyp `header` gültig. Wenn Sie beispielsweise ein Cookie im HTTP-Header abgleichen möchten, kann das Feld auf `Cookie` festgelegt werden.
Wert | Der Wert, der abgeglichen werden soll.

## Lastausgleichsmethoden
{: #load-balancing-methods}

Für die Verteilung des Datenverkehrs zwischen Back-End-Anwendungsservern stehen die folgenden drei Lastausgleichsmethoden zur Verfügung:

* **Umlauf:** Der Umlauf ist die Standardmethode für den Lastausgleich. Bei dieser Methode leitet die Lastausgleichsfunktion eingehende Clientverbindungen im Umlaufverfahren an die Back-End-Server weiter. Als Ergebnis erhalten alle Back-End-Server in etwa die gleiche Anzahl von Clientverbindungen.

* **Gewichteter Umlauf:** Bei dieser Methode leitet die Lastausgleichsfunktion eingehende Clientverbindungen proportional zu der diesen Servern zugewiesenen Gewichtung an die Back-End-Server weiter. Jedem Server ist eine Standardgewichtung von 50 zugewiesen. Diese Gewichtung kann durch die Festlegung eines beliebigen Werts zwischen 0 und 100 angepasst werden.

Wenn zum Beispiel für drei Anwendungsserver namens A, B und C eine angepasste Gewichtung von 60, 60 und 30 festgelegt wurde, so erhalten die Server A und B jeweils die gleiche Anzahl von Verbindungen, während Server C die Hälfte dieser Verbindungsanzahl erhält.

* **Wenigste Verbindungen:** Bei dieser Methode erhält diejenige Serverinstanz, die zu einem gegebenen Zeitpunkt die geringste Anzahl von Verbindungen bereitstellt, die nächste Clientverbindung.

**Weitere Merkmale dieser Methoden: **

* Das Zurücksetzen einer Servergewichtung auf den Wert '0' bedeutet, dass keine neuen Verbindungen an diesen Server weitergeleitet werden. Der vorhandene Datenverkehr wird jedoch weiterhin fortgesetzt. Die Verwendung einer Gewichtung von '0' kann helfen, einen Server ordnungsgemäß herunterzufahren und ihn aus der Servicerotation zu entfernen.
* Die Servergewichtungswerte gelten nur für die Methode des gewichteten Umlaufs. Sie werden bei den Lastausgleichsmethoden des Umlaufs und der geringsten Anzahl an Verbindungen ignoriert.

## Horizontale Skalierung
{: #horizontal-scaling}

Die Lastausgleichsfunktion passt ihre Kapazität automatisch an die Workload an. Ist diese Anpassung der Fall, kann es zu einer Änderung bei der Anzahl der IP-Adressen kommen, die dem DNS-Namen der Lastausgleichsfunktion zugeordnet sind. 

## Statusprüfungen
{: #health-checks}

Statusprüfungsdefinitionen sind für Back-End-Pools obligatorisch.

Die Lastausgleichsfunktion führt in regelmäßigen Abständen Statusprüfungen durch, um den Zustand der Back-End-Ports zu überwachen. Danach leitet sie den Clientdatenverkehr entsprechend an die Ports weiter. Wenn sich ein bestimmter Back-End-Server-Port als fehlerhaft herausstellt, werden keine neuen Verbindungen an ihn weitergeleitet. Die Lastausgleichsfunktion überwacht den Zustand fehlerhafter Ports auch weiterhin und nimmt ihre Verwendung wieder auf, wenn sie sich wieder in einwandfreiem Zustand befinden. Dazu müssen sie zwei aufeinanderfolgende Statusprüfungen erfolgreich durchlaufen haben.

Die Statusprüfungen für HTTP- und TCP-Ports werden wie folgt durchgeführt:

* **HTTP:** An den Back-End-Server-Port wird für eine vordefinierte URL eine `HTTP GET`-Anforderung gesendet. Bei Erhalt einer Antwort `200 OK` wird der Server-Port als in einwandfreiem Zustand befindlich markiert. Der Standardpfad für den `GET`-Status lautet '/' über die Benutzerschnittstelle (UI). Er kann jedoch angepasst werden.

* **TCP:** Die Lastausgleichsfunktion versucht, an einem angegebenen TCP-Port eine TCP-Verbindung mit dem Back-End-Server herzustellen. Der Server-Port wird als in einwandfreiem Zustand befindlich markiert, wenn der Verbindungsversuch erfolgreich ist, woraufhin die Verbindung beendet wird.

Das Intervall für die Statusprüfung beträgt standardmäßig 5 Sekunden und als Zeitlimit für eine Zustandsprüfungsanforderung ist standardmäßig ein Wert von 2 Sekunden definiert. Die Standardanzahl der Wiederholungsversuche beläuft sich auf 2.
{: note}

## SSL-Auslagerung und erforderliche Autorisierungen
{: #ssl-offloading-and-required-authorizations}

Die Lastausgleichsfunktion beendet für alle eingehenden HTTPS-Verbindungen die SSL-Verbindung und stellt mit der Back-End-Serverinstanz eine HTTP-Kommunikation in unverschlüsseltem Text (Klartext) her. Bei diesem Verfahren werden CPU-intensive SSL-Handshakes und Verschlüsselungs- oder Entschlüsselungstasks weg von den Back-End-Serverinstanzen verlagert, sodass diese ihre gesamten CPU-Zyklen für die Verarbeitung von Anwendungsverkehr aufwenden können.

Damit die Lastausgleichsfunktion SSL-Auslagerungstasks ausführen kann, ist ein SSL-Zertifikat erforderlich. Die Verwaltung der SSL-Zertifikate kann über [IBM Certificate Manager ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window} vorgenommen werden.

Damit eine Lastausgleichsfunktion auf Ihr SSL-Zertifikat zugreifen kann, müssen Sie die **Service-zu-Service-Autorisierung** aktivieren. Dadurch erhält Ihre Instanz des Lastausgleichsfunktionsservice Zugriff auf Ihre Certificate Manager-Instanzen. Informationen zur Verwaltung einer solchen Autorisierung finden Sie in der Dokumentation [Zugriff zwischen Services erteilen![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")](/docs/iam?topic=iam-serviceauth#create-auth){: new_window}. Stellen Sie sicher, dass Sie **VPC-Infrastruktur** als Quellenservice, **Lastausgleichsfunktion für VPC** als Ressourcentyp und **Certificate Manager** als Zielservice auswählen. Weisen Sie **Writer (Schreibberechtigter)** als Servicezugriffsrolle zu. 

Wenn die erforderliche Autorisierung entfernt wird, können Fehler für Ihre Lastausgleichsfunktion auftreten.
{: note}

## Identity and Access Management (IAM) für das Identitäts- und Zugriffsmanagement
{: #identity-and-access-management-iam}

Sie können Zugriffsrichtlinien für eine Instanz der **Lastausgleichsfunktion für VPC** konfigurieren. Wenn Sie Informationen zur Verwaltung von Benutzerzugriffsrichtlinien benötigen, finden Sie in [Zugriff auf Ressourcen verwalten ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")](/docs/services/iam?topic=iam-iammanidaccser#resourceaccess){: new_window} weitere Details zum Identitäts- und Zugriffsmanagement.

### Zugriffsrichtlinien für Ressourcengruppen für Benutzer konfigurieren
{: #configuring-resource-group-access-policies-for-users}

Zum Erstellen einer Lastausgleichsfunktion benötigen Sie Zugriff auf eine Ressourcengruppe. Der Benutzer, der einen Lastenausgleich erstellt, muss über ordnungsgemäßen Zugriff auf die bereitgestellte Ressourcengruppe oder aber auf die Standardressourcengruppe verfügen, falls keine bereitgestellt wird.

1. Navigieren Sie zu **Verwalten > Konto > Benutzer**. Es wird eine Liste der Benutzer mit Zugriff auf Ihr IBM Cloud-Konto angezeigt.
2. Wählen Sie den Namen des Benutzers aus, dem Sie eine Zugriffsrichtlinie zuweisen wollen. Falls der gewünschte Benutzer nicht angezeigt wird, klicken Sie auf **Benutzer einladen**, um den Benutzer zu Ihrem IBM Cloud-Konto hinzuzufügen.
3. Wählen Sie **Zugriff zuweisen** aus.
4. Wählen Sie **Zugriff in einer Ressourcengruppe zuweisen** aus.
5. Wählen Sie in der Dropdown-Liste **Ressourcengruppe** die gewünschte Ressourcengruppe aus.
6. Wählen Sie in der Dropdown-Liste **Zugriff für eine Ressourcengruppe zuweisen** den gewünschten Zugriff aus.
7. Wählen Sie in der Dropdown-Liste **Services** die gewünschten Services aus.
9. Wählen Sie **Zuweisen** aus, um dem Benutzer die Zugriffsrichtlinie für die Ressourcengruppe zuzuweisen.

### Zugriffsrichtlinien für Ressourcen für Benutzer konfigurieren
{: #configuring-resource-access-policies-for-users}

| Plattformzugriffsrolle | Aktion für die Lastausgleichsfunktion |
|-------------|-----|
| Administrator | Erstellen/Anzeigen/Bearbeiten/Löschen der Lastausgleichsfunktion |
| Editor (Bearbeiter) | Erstellen/Anzeigen/Bearbeiten/Löschen der Lastausgleichsfunktion |
| Viewer (Anzeigeberechtigter) | Anzeigen der Lastausgleichsfunktion |

1. Navigieren Sie zu **Verwalten > Konto > Benutzer**. Es wird eine Liste der Benutzer mit Zugriff auf Ihr IBM Cloud-Konto angezeigt.
2. Wählen Sie den Namen des Benutzers aus, dem Sie eine Zugriffsrichtlinie zuweisen wollen. Falls der gewünschte Benutzer nicht angezeigt wird, klicken Sie auf **Benutzer einladen**, um den Benutzer zu Ihrem IBM Cloud-Konto hinzuzufügen.
3. Wählen Sie **Zugriff zuweisen** aus.
4. Wählen Sie **Zugriff auf Ressourcen zuweisen** aus.
5. Wählen Sie in der Dropdown-Liste **Services** den Eintrag **VPC-Infrastruktur** aus.
6. Wählen Sie in der Dropdown-Liste **Ressourcentyp** den Eintrag **Lastausgleichsfunktion für VPC** aus.
7. Wählen Sie in der Dropdown-Liste **ID der Lastausgleichsfunktion** die ID für eine Lastausgleichsfunktionsinstanz aus oder verwenden Sie den Standardwert **Alle Lastausgleichsfunktionen**.
8. Weisen Sie dem Benutzer einen Plattformzugriff zu.
9. Wählen Sie **Zuweisen** aus, um dem Benutzer die Zugriffsrichtlinie zuzuweisen.

## Activity Tracker-Integration
{: #activity-tracker-integration}

Der Lastausgleichsfunktionsservice ist in **IBM Cloud Activity Tracker with LogDNA** integriert. Dieser Service zeichnet in Übereinstimmung mit dem CADF-Standard Ereignisse auf, die durch Aktivitäten ausgelöst werden, die von Benutzern initiiert werden und den Status von Services in der Cloud ändern.

Eine detaillierte Liste der Aktionen, die als Prüfereignisse in den Serviceinstanzen der Lastausgleichsfunktion aufgezeichnet werden, finden Sie im Abschnitt ["Activity Tracker with LogDNA"-Ereignissen](/docs/vpc-on-classic?topic=vpc-on-classic-at-events#events-load-balancers).

Alle Auditereignisse werden in "IBM Cloud Activity Tracker with LogDNA" in der Region `us-south` aufgezeichnet. Dabei spielt es keine Rolle, in welcher Region der Lastausgleichsfunktionsservice bereitgestellt wird.
{:note}

Sie müssen zum Anzeigen von Ereignissen eine Instanz von "IBM Cloud Activity Tracker with LogDNA" in der Region `us-south` unter Ihrem Konto bereitstellen. Benutzer in Ihrem Konto müssen über eine IAM-Richtlinie verfügen, die die Zugriffsrolle **Anzeigeberechtigter** und die Servicezugriffsrolle **Leseberechtigter** für den Service "IBM Cloud Activity Tracker with LogDNA" erteilt.

Weitere Informationen zum Erteilen von Zugriffsberechtigungen finden Sie in [Berechtigungen zum Anzeigen von Kontoereignissen erteilen ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")](https://cloud.ibm.com/docs/services/cloud-activity-tracker/how-to/grant_permissions.html#grant_permissions){: new_window}.

Benutzer des IBM Cloud-Kontos können Operationen auf Kontoebene überwachen, die für den Lastausgleichsfunktionsservice ausgeführt werden.
{: tip}

Führen Sie die folgenden Schritte aus, um eine Instanz von "IBM Cloud Activity Tracker with LogDNA" unter Ihrem Konto bereitzustellen:

1. Melden Sie sich an der IBM Cloud-Konsole an. [Melden Sie sich an der IBM Cloud-Konsole an. ![Symbol für externen Link](../icons/launch-glyph.svg "Symbol für externen Link")](https://cloud.ibm.com/){: new_window}
2. Klicken Sie oben links auf das ![Menüsymbol](../../icons/icon_hamburger.svg). Wählen Sie dort **Beobachtbarkeit > Activity Tracker** aus.
3. Klicken Sie rechts oben auf **Instanz erstellen**.
4. Definieren Sie einen Servicenamen.
5. Wählen Sie `us-south` als Region aus und wählen Sie die Ressourcengruppe aus.
6. Wählen Sie einen anderen Plan als `lite` aus, wenn Sie ein kostenpflichtiges Konto haben.
7. Klicken Sie auf **Erstellen**.

Im folgenden Beispiel ist eine Activity Tracker-Nachricht für die Operation **Listener erstellen** dargestellt:
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

## Verfügbare Anwendungsprogrammierschnittstellen (APIs)
{: #lbaas-apis-available}

Zum Durchführen von API-Aufrufen müssen Sie eine Form des REST-Clients verwenden. Sie können zum Beispiel den Befehl `curl` verwenden, um alle vorhandenen Lastausgleichsfunktionen abzurufen:

```
curl -X GET "$rias_endpoint/v1/load_balancers?version=2019-05-31&generation=1" -H "Authorization: $iam_token"
```
{: pre}

Der folgende Abschnitt liefert detaillierte Angaben zu den APIs, die Sie für Lastausgleichsfunktionen in Ihrer VPC-Umgebung verwenden können. Die vollständigen Spezifikationen enthält die Veröffentlichung [VPC on Classic API reference](https://{DomainName}/apidocs/vpc-on-classic#list-all-load-balancers).

| Beschreibung | API |
|-------------|-----|
| Erstellt eine Lastausgleichsfunktion und stellt sie bereit | POST /load_balancers |
| Ruft alle Lastausgleichsfunktionen ab | GET /load_balancers |
| Ruft eine Lastausgleichsfunktion ab | GET /load_balancers/{id} |
| Löscht eine Lastausgleichsfunktion | DELETE /load_balancers/{id} |
| Aktualisiert eine Lastausgleichsfunktion | PATCH /load_balancers/{id} |
| Erstellt einen Listener | POST /load_balancers/{id}/listeners |
| Ruft alle Listener der Lastausgleichsfunktion ab | GET /load_balancers/{id}/listeners |
| Ruft einen Listener ab | GET /load_balancers/{id}/listeners/{listener_id} |
| Löscht einen Listener | DELETE /load_balancers/{id}/listeners/{listener_id} |
| Aktualisiert einen Listener | PATCH /load_balancers/{id}/listeners/{listener_id} |
| Erstellt einen Pool | POST /load_balancers/{id}/pools |
| Ruft alle Pools der Lastausgleichsfunktion ab | GET /load_balancers/{id}/pools |
| Ruft eine Pool ab | GET /load_balancers/{id}/pools/{pool_id} |
| Löscht einen Pool | DELETE /load_balancers/{id}/pools/{pool_id} |
| Aktualisiert einen Pool | PATCH /load_balancers/{id}/pools/{pool_id} |
| Erstellt ein Mitglied (Member) | POST /load_balancers/{id}/pools/{pool_id}/members |
| Ruft alle Mitglieder des Pools ab | GET /load_balancers/{id}/pools/{pool_id}/members |
| Ruft ein Mitglied ab |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Löscht ein Mitglied aus dem Pool | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Aktualisiert ein Mitglied | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Aktualisiert Mitglieder des Pools | PUT /load_balancers/{id}/pools/{pool_id}/members |
| Ruft Statistikdaten einer Lastausgleichsfunktion ab | GET /load_balancers/{id}/statistics |
| Ruft alle Richtlinien des Listeners ab |  GET /load_balancers/{id}/listeners/{listener_id}/policies
| Erstellt eine Richtlinie für den Listener | POST /load_balancers/{id}/listeners/{listener_id}/policies
| Löscht eine Richtlinie aus dem Listener | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Ruft eine Richtlinie des Listeners ab | GET /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Aktualisiert eine Richtlinie des Listeners | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Ruft alle Regeln ab, die einer Richtlinie zugeordnet sind | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| Erstellt eine Regel für die Richtlinie | POST /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| Löscht eine Regel aus der Richtlinie | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| Ruft eine Regel aus der Richtline ab | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| Aktualisiert eine Regel der Richtlinie | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}

## Beispiel einer Lastausgleichsfunktion
{: #load-balancer-example}

Im folgenden Beispiel verwenden Sie die API, um eine Lastausgleichsfunktion vor zwei VPC-Serverinstanzen (`192.168.100.5` und `192.168.100.6`) zu erstellen, die eine Webanwendung ausführen, die an Port `80` empfangsbereit ist. Die Lastausgleichsfunktion verfügt über einen Front-End-Listener, der den sicheren Zugriff auf die Webanwendung mittels HTTPS ermöglicht. Nach der Erstellung der Lastausgleichsinstanz können Sie über die API Details zu der Lastausgleichsinstanz abrufen und die Instanz der Lastausgleichsfunktion löschen.

### Beispielschritte
{: #lbaas-example-steps}

Bei den nachfolgenden Beispielschritten werden die vorausgesetzten Schritte zum Bereitstellen einer VPC, Teilnetzen und Instanzen über die [IBM Cloud-UI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console), [CLI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli) oder [VPC on Classic-API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis) übersprungen.

Die Beispielschritte für die Lastausgleichsfunktion können auch über die [CLI](/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference) ausgeführt werden.
{: note}

**Schritt 1. Lastausgleichsfunktion mit Listener-, Pool- und angehängten Serverinstanzen (Poolmitgliedern) erstellen**

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

Beispielausgabe:
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

Speichern Sie die ID der Lastausgleichsfunktion zur Verwendung in den nachfolgenden Schritten, zum Beispiel in der Variablen `lbid`.

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

**Schritt 2. Lastausgleichsfunktion abrufen**

```
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

Kalkulieren Sie etwas Zeit für die Bereitstellung ein. Wenn die Lastausgleichsfunktion bereit ist, wird sie in den Status `online` und `active` versetzt, wie in der folgenden Beispielausgabe dargestellt:

Beispielausgabe:

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

**Schritt 3. Lastausgleichsfunktion löschen**

```bash
curl -H "Authorization: $iam_token" -X DELETE "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

## Layer-7-Beispiele: Richtlinien und Regeln erstellen
{: #layer-7-examples-create-policy-and-rules}

Die folgenden beiden Beispiele enthalten Schritte, die erklären, wie Richtlinien und Regeln erstellt und mit einem Listener verknüpft werden.

### Beispiel 1: HTTPS-Listener mit Richtlinien und Regeln erstellen. Die Richtlinien werden mit der Aktion `Redirect` erstellt.

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

### Beispiel 2: Richtlinien mit der Aktion `Forward` für Pools erstellen und mit einem vorhandenen Listener verknüpfen
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

## Häufig gestellte Fragen (FAQs)
{: #load-balancer-faqs}

Dieser Abschnitt enthält Antworten auf häufig gestellte Fragen (FAQs) zum Service der **Lastausgleichsfunktion für VPC**.

### Kann ich einen anderen DNS-Namen für meine Lastausgleichsfunktion verwenden?
{: #can-i-use-a-different-dns-name-for-my-load-balancer}

Der DNS-Name für die Lastausgleichsfunktion wird automatisch zugeordnet und kann nicht angepasst werden. Sie können jedoch einen CNAME-Datensatz für den kanonischen Namen hinzufügen, der mit dem bevorzugten DNS-Namen auf den automatisch zugewiesenen DNS-Namen der Lastausgleichsfunktion verweist. Beispiel: Ihre Lastausgleichsfunktion in `us-south` hat die ID `dd754295-e9e0-4c9d-bf6c-58fbc59e5727`, daher lautet der DNS-Name der automatisch zugeordneten Lastausgleichsfunktion `dd754295-us-south.lb. appdomain.cloud`. Nehmen Sie an, dass Ihr bevorzugter DNS-Name `www.myapp.com` wäre. Sie könnten dann (über den DNS-Provider, den Sie zum Verwalten von `myapp.com` verwenden) einen CNAME-Datensatz hinzufügen, mit dem Sie `www.myapp.com` auf den DNS-Namen `dd754295-us-south.lb.appdomain.cloud` der Lastausgleichsfunktion verweisen. 

### Wie viele Front-End-Listener kann ich mit meiner Lastausgleichsfunktion maximal definieren?
{: #what-s-the-maximum-number-of-front-end-listeners-i-can-define-with-my-load-balancer}

10.

### Wie viele Serverinstanzen kann ich maximal meinem Back-End-Pool zuordnen?
{: #what-s-the-maximum-number-of-server-instances-i-can-attach-to-my-back-end-pool}

50.

### Ist die Lastausgleichsfunktion horizontal skalierbar?
{: #is-the-load-balancer-horizontally-scalable}

Ja. Die Lastausgleichsfunktion passt ihre Kapazität automatisch an die Workload an. Wenn eine horizontale Skalierung stattfindet, ändert sich die Anzahl der IP-Adressen, die dem DNS-Namen der Lastausgleichsfunktion zugeordnet sind.

### Was soll ich tun, wenn ich ACLs oder Sicherheitsgruppen für die Teilnetze verwende, die zum Bereitstellen der Lastausgleichsfunktion verwendet werden?
{: #what-should-i-do-if-i-am-using-acls-or-security-groups-on-the-subnets-that-are-used-to-deploy-the-load-balancer}

Sie müssen sicherstellen, dass die richtigen ACL- oder Sicherheitsgruppenregeln vorhanden sind, damit eingehender Datenverkehr für konfigurierte Listener-Ports und Management-Ports (Ports 56500-56520) zugelassen wird. Zwischen der Lastausgleichsfunktion und Back-End-Instanzen sollte der Datenverkehr ebenfalls zulässig sein.

### Warum erhalte ich eine Meldung mit dem Inhalt, dass die Zertifikatsinstanz nicht gefunden wurde (`certificate instance not found`)?

* Möglicherweise ist der CRN der Zertifikatsinstanz ungültig.
* Möglicherweise haben Sie keine **Service-zu-Service-Autorisierung** erteilt. Lesen Sie hierzu den Abschnitt **SSL-Auslagerung** im vorliegenden Dokument.

### Warum wird ein Fehlercode vom Typ `401 – Nicht autorisiert` zurückgegeben?

Überprüfen Sie die folgenden Zugriffsrichtlinien für Ihren Benutzer:
* Die Zugriffsrichtlinie für den Ressourcentyp der Lastausgleichsfunktion. 
* Die Zugriffsrichtlinie für die Ressourcengruppe. 
* Ob `HTTPS`-Listener verwendet werden. Überprüfen Sie darüber hinaus die Service-zu-Service-Autorisierung für die Certificate Manager-Instanz.

### Warum hat meine Lastausgleichsfunktion den Status `maintenance_pending`?

Die Lastausgleichsfunktion weist den Status `maintenance_pending` bei diversen Verwaltungsaktivitäten auf, wie zum Beispiel den folgenden:
* Horizontale Skalierungsaktivitäten
* Wiederherstellungsaktivitäten
* Rollierendes Upgrade, bei dem Sicherheitslücken angesprochen und Sicherheitspatches angewendet werden

### Warum muss ich bei der Bereitstellung mehrere Teilnetze auswählen?
{: #why-do-I-need-to-choose-multiple-subnets-during-provisioning}

**Load Balancer for VPC** ist MZR-fähig (MZR = Multi-Zone-Region, Region mit mehreren Zonen). Lastausgleichsappliances werden in den von Ihnen ausgewählten Teilnetzen bereitgestellt. Es wird dringend empfohlen, Teilnetze aus verschiedenen Zonen auszuwählen, um eine höhere Verfügbarkeit und Redundanz zu erzielen.

### Warum lautet der Back-End-Mitgliedsstatus im Pool `unknown`?

* Dem Pool sind keine Listener zugeordnet.
* Möglicherweise wurden Konfigurationsänderungen am Pool oder den zugeordneten Listenern vorgenommen.

### Welche TLS-Version wird mit der SSL-Auslagerung unterstützt?
{: #which-tls-version-is-supported-with-ssl-offload}

**Load Balancer for VPC** unterstützt TLS 1.2 mit SSL-Terminierung.

In der folgenden Liste sind die unterstützten Verschlüsselungen aufgeführt (in der Reihenfolge ihrer Priorität):
* ECDHE-RSA-AES256-GCM-SHA384
* ECDHE-RSA-AES256-SHA384
* AES256-GCM-SHA384
* AES256-SHA256
* ECDHE-RSA-AES128-GCM-SHA256
* ECDHE-RSA-AES128-SHA256
* AES128-GCM-SHA256
* AES128-SHA256

### Wie lauten die Standardeinstellungen und die zulässigen Werte für Statusprüfungsparameter?
{: #what-are-the-default-settings-and-allowed-values-for-health-check-parameters}

Die Standardeinstellungen und die zulässigen Werte sind nachfolgend aufgeführt:
* Statusprüfungsintervall: Die Standardeinstellung ist 5 Sekunden, der zulässige Bereich ist 2 bis 60 Sekunden.
* Antwortzeitlimit für die Statusprüfung: Die Standardeinstellung ist 2 Sekunden, der zulässige Bereich ist 1 bis 59 Sekunden.
* Maximale Anzahl der Wiederholungsversuche: Die Standardeinstellung ist 2 Wiederholungsversuche, der zulässige Bereich ist 1 bis 10 Wiederholungen.

Der Wert für das Antwortzeitlimit der Statusprüfung muss stets niedriger sein als der Wert für das Statusprüfungsintervall.
{:note}

### Sind die IP-Adressen der Lastausgleichsfunktion festgelegt?
{: #are-the-load-balancer-ip-addresses-fixed}

Aufgrund der integrierten Elastizität des Service ist nicht gewährleistet, dass die IP-Adressen der Lastausgleichsfunktion festgelegt sind. Während der horizontalen Skalierung können sich die verfügbaren IP-Adressen, die dem FQDN Ihrer Lastausgleichsfunktion zugeordnet sind, ändern.

Verwenden Sie den FQDN anstelle der im Cache gespeicherten IP-Adressen.
{:note}

### Unterstützt die Lastausgleichsfunktion Layer-7-Switching? 
{: #does-the-load-balancer-support-layer-7-switching}

Ja.

### Warum wird bei der HTTPS-Listener-Erstellung oder -Aktualisierung die Nachricht ausgegeben, dass ein Zertifikat ungültig ist?
{: #why-does-https-listener-creation-or-update-tell-me-that-my-certificate-is-invalid}

Überprüfen Sie die folgenden Möglichkeiten:

* Möglicherweise ist der angegebene Zertifikats-CRN ungültig.
* Möglicherweise ist der in Certificate Manager angegebenen Zertifikatsinstanz kein privater Schlüssel zugeordnet.
