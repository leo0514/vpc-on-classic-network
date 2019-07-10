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

# VPN mit Ihrem VPC verwenden
{: #--using-vpn-with-your-vpc}
[comment]: # (Verlinktes Hilfethema)

Der VPN-Service von {{site.data.keyword.cloud}} ermöglicht Ihnen, private Netze auf sichere Weise zu verbinden. Sie können VPN verwenden, um von einem Standort zu einem anderen einen IPsec-Tunnel zwischen Ihrer VPC-Instanz und Ihrem lokalen privaten Netz oder einer anderen VPC-Instanz einzurichten.

Für das aktuelle Release von {{site.data.keyword.cloud}} VPC wird nur die richtlinienbasierte Weiterleitung unterstützt.

## Features
{: #vpn-features}

* IKEv1 und IKEv2
* Authentifizierungsalgorithmen: `md5`, `sha1`, `sha256`
* Verschlüsselungsalgorithmen: `3des`, `aes128`, `aes256`
* Diffie-Hellman-Gruppen (DH-Gruppen): 2, 5, 14
* IKE-Vereinbarungsmodus: main
* IPSec-Transformationsprotokoll: ESP
* IPSec-Kapselungsmodus: tunnel
* Absolute vorwärts gerichtete Sicherheit (PFS, Perfect Forward Secrecy)
* Erkennung inaktiver Peers
* Weiterleitung: Richtlinienbasiert
* Authentifizierungsmodus: Vorab verteilter Schlüssel
* HA-Unterstützung nur im Aktiv/Standby-Modus

## Verfügbare Anwendungsprogrammierschnittstellen (APIs)
{: #apis-available}

Der folgende Abschnitt liefert detaillierte Angaben zu den APIs, die Sie für VPN in Ihrer IBM Cloud VPC-Umgebung verwenden können. Weitere Details können Sie der Seite [VPC-REST-APIs](https://{DomainName}/apidocs/vpc-on-classic#list-all-ike-policies) entnehmen.

### VPN-Gateways und VPN-Verbindungen
{: #vpn-gateways-and-vpn-connections}

| Beschreibung | API |
|----------------------------|-------------|
| Erstellt ein VPN-Gateway | POST /vpn_gateways |
| Ruft VPN-Gateways ab | GET /vpn_gateways |
| Ruft ein VPN-Gateway ab | GET /vpn_gateways/{id} |
| Löscht ein VPN-Gateway | DELETE /vpn_gateways/{id} |
| Aktualisiert ein VPN-Gateway | PATCH /vpn_gateways/{id} |
| Erstellt eine neue VPN-Verbindung | POST /vpn_gateways/{vpn_gateway_id}/connections |
| Ruft VPN-Verbindungen ab | GET /vpn_gateways/{vpn_gateway_id}/connections |
| Ruft eine VPN-Verbindung ab | GET /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Löscht eine VPN-Verbindung | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Aktualisiert eine VPN-Verbindung | PATCH /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Ruft alle lokalen CIDRs für eine VPN-Verbindung ab | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs |
| Löscht eine lokale CIDR aus einer VPN-Verbindung | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Prüft, ob eine bestimmte lokale CIDR in einer VPN-Verbindung vorhanden ist | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Legt eine lokale CIDR für eine VPN-Verbindung fest | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Ruft alle Peer-CIDRs für eine VPN-Verbindung ab | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs |
| Löscht eine Peer-CIDR aus einer VPN-Verbindung | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Prüft, ob eine bestimmte Peer-CIDR in einer VPN-Verbindung vorhanden ist | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Legt eine Peer-CIDR für eine VPN-Verbindung fest | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |

### IKE-Richtlinien
{: #ike-policies}

| Beschreibung | API |
|-----------------------------|--------------|
| Ruft alle IKE-Richtlinien ab | GET /ike_policies |
| Erstellt eine IKE-Richtlinie | POST /ike_policies |
| Löscht eine IKE-Richtlinie | DELETE /ike_policies/{id} |
| Ruft eine IKE-Richtlinie ab | GET /ike_policies/{id} |
| Aktualisiert eine IKE-Richtlinie | PATCH /ike_policies/{id} |
| Ruft alle Verbindungen ab, die die angegebene IKE-Richtlinie verwenden | GET /ike_policies/{id}/connections |

### IPsec-Richtlinien
{: #ipsec-policies}

| Beschreibung | API |
|---------------------|-------------|
| Ruft alle IPsec-Richtlinien ab | GET /ipsec_policies |
| Erstellt eine IPsec-Richtlinie | POST /ipsec_policies |
| Löscht eine IPsec-Richtlinie | DELETE /ipsec_policies/{id} |
| Ruft eine IPsec-Richtlinie ab | GET /ipsec_policies/{id} |
| Aktualisiert eine IPsec-Richtlinie | PATCH /ipsec_policies/{id} |
| Ruft alle Verbindungen ab, die die angegebene IPsec-Richtlinie verwenden | GET /ipsec_policies/{id}/connections |

## VPN-Beispiel
{: #vpn-example}

Im folgenden Beispiel sind Sie in der Lage, zwei VPCs über VPN miteinander zu verbinden. Das bedeutet, dass Sie Teilnetze in zwei separaten VPCs so verbinden können, als wären sie ein einzelnes Netz. Die IP-Adressen der Teilnetze dürfen sich nicht überschneiden.
Das Szenario sieht wie folgt aus, wobei in jeder VPC-Instanz einige virtuelle Maschinen (VMs) hinzugefügt wurden:

![VPN für IBM VPC](images/vpc-vpn.svg "VPN für IBM VPC")

### Beispielschritte
{: #vpn-example-steps}

Bei den nachfolgenden Beispielschritten werden die vorausgesetzten Schritte zum Erstellen von VPCs unter Verwendung der IBM Cloud-API oder -CLI übersprungen. Weitere Informationen finden Sie in der [Einführung](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) und in [VPC-Einrichtung mit APIs](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

Wahlweise können Sie über die Benutzerschnittstelle (UI) ein VPN-Gateway erstellen. Die entsprechenden Schritte sind im [Dokument für das Lernprogramm für die Konsole](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn) beschrieben.

#### Schritt 1. VPN-Gateway in Ihrem VPC-Teilnetz erstellen
{: #step-1-create-a-vpn-gateway-in-your-vpc-subnet}

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways?version=2019-05-31&generation=1" \
    -d '{
            "name": "vpn-gateway-1",
            "subnet": {"id": $subnet1}
        }'
```
{: codeblock}

Beispielausgabe:
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

Stellen Sie sicher, dass die Angaben für die folgenden Felder für die nachfolgenden Schritte gespeichert werden.
* `id`: Dies ist die VPN-Gateway-ID. Sie wird hier als `$gwid1` bezeichnet.
* `address`: Dies ist die öffentliche IP-Adresse des VPN-Gateways. Sie wird hier als `$gwaddress1` bezeichnet.

Für das Gateway wird der Status 'Anstehend' (`pending`) angezeigt, solange das VPN-Gateway erstellt wird. Der Status wechselt zu 'Verfügbar' (`available`), sobald der Erstellungsvorgang abgeschlossen ist. Die Erstellung kann einige Zeit in Anspruch nehmen.
{: note}


Sie können den Gateway-Status mit dem folgenden Befehl überprüfen:

```bash
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/vpn_gateways/$gwid1?version=2019-05-31&generation=1"
```
{: codeblock}

#### Schritt 2. Zweites VPN-Gateway auf einer anderen VPC erstellen
{: #step-2-create-a-second-vpn-gateway-on-a-different-vpc}

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-gateway-2",
                "subnet": {"id": $subnet2}
            }'
```
{: codeblock}

Beispielausgabe:
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

Stellen Sie sicher, dass Sie die Angaben für die folgenden Felder für die nachfolgenden Schritte speichern.
* `id`: Dies ist die VPN-Gateway-ID. Sie wird hier als `$gwid2` bezeichnet.
* `address`: Dies ist die öffentliche IP-Adresse des VPN-Gateways. Sie wird hier als `$gwaddress2` bezeichnet.


#### Schritt 3. VPN-Verbindung vom ersten VPN-Gateway zum zweiten VPN-Gateway erstellen
{: #step-3-create-a-vpc-connection-from-the-first-vpn-gateway-to-the-second-vpn-gateway}

Wenn Sie die Verbindung erstellen, setzen Sie `local_cidrs` in das Teilnetz auf **VPC one** und `peer_cidrs` in das Teilnetz auf **VPC two** fest.

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

Beispielausgabe:
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

#### Schritt 4. VPN-Verbindung vom zweiten VPN-Gateway zum ersten VPN-Gateway erstellen
{: #step-4-create-a-vpn-connection-from-the-second-vpn-gateway-to-the-first-vpn-gateway}

Wenn Sie die Verbindung erstellen, setzen Sie `local_cidrs` in das Teilnetz auf **VPC two** und `peer_cidrs` in das Teilnetz auf **VPC one** fest.

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

Beispielausgabe:
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

#### Schritt 5. Konnektivität überprüfen
{: #step-5-verify-connectivity}

Sobald die VPN-Verbindung hergestellt ist, können Sie Ihre Instanzen in Teilnetz 2 von Teilnetz 1 aus erreichen und umgekehrt.

Sie können den Status der VPN-Verbindung wie folgt prüfen:
```bash
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-05-31&generation=1"
```
{: codeblock}

Beispielausgabe:
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

## Beschränkungen bzw. Kontingente
{: #see-vpn-quotas}

Informationen zu Beschränkungen bzw. Kontingenten für VPNs finden Sie im Abschnitt [VPC-Kontingente](/docs/vpc-on-classic?topic=vpc-on-classic-quotas#vpn-quotas).

## Automatische Vereinbarung von Richtlinien
{: #policy-auto-negotiation}

Die Verwendung von IKE- und IPsec-Richtlinien zum Konfigurieren einer VPN-Verbindung ist optional. Wenn keine Richtlinie ausgewählt ist, werden automatisch Standardvorschläge für ein Verfahren ausgewählt, das als _automatische Vereinbarung_ bezeichnet wird. 

Die automatische IBM Cloud-Vereinbarung verwendet **IKEv2** und daher muss die On-Premises-Einheit auch **IKEv2** verwenden. Verwenden Sie eine angepasste IKE-Richtlinie, wenn Ihre On-Premises-Einheit **IKEv2** nicht unterstützt.{: note}

### Automatische IKE-Vereinbarung (Phase 1)
{: #ike-auto-negotiation-phase-1}

Die folgenden Optionen für die Verschlüsselung, die Authentifizierung und die Diffie-Hellman-Gruppe (DH-Gruppe) können in beliebiger Kombination verwendet werden:

|    | Verschlüsselung | Authentifizierung | DH-Gruppe |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | 2  |
| 2  | aes256 | sha256 | 5  |
| 3  | 3des   | md5    | 14 |

### Automatische IPsec-Vereinbarung (Phase 2)
{: #ipsec-auto-negotiation-phase-2}

Die folgenden Optionen für die Verschlüsselung und die Authentifizierung können in beliebiger Kombination verwendet werden:

|    | Verschlüsselung | Authentifizierung | DH-Gruppe |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | inaktiviert  |
| 2  | aes256 | sha256 | inaktiviert  |
| 3  | 3des   | md5    | inaktiviert  |

## Häufig gestellte Fragen (FAQs)
{: #vpn-faq}

**Kann ich beim Erstellen eines VPN-Gateways gleichzeitig auch VPN-Verbindungen erstellen?**

Wenn Sie die Anwendungsprogrammierschnittstelle (API) oder die Befehlszeilenschnittstelle (CLI) verwenden, müssen VPN-Verbindungen erstellt werden, nachdem das VPN-Gateway erstellt worden ist. Bei Verwendung der IBM Cloud-Konsole können Sie das Gateway und eine Verbindung zur gleichen Zeit erstellen.

**Was passiert mit den Verbindungen, wenn ich ein VPN-Gateway lösche, an das VPN-Verbindungen angehängt sind?**

Die VPN-Verbindungen werden zusammen mit dem VPN-Gateway gelöscht.

**Werden die IKE- oder IPsec-Richtlinien gelöscht, wenn ich ein VPN-Gateway oder eine VPN-Verbindung lösche?**

Nein, IKE- und IPsec-Richtlinien können für mehrere Verbindungen gelten.

**Was passiert mit einem VPN-Gateway, wenn ich versuche, das Teilnetz zu löschen, in dem sich das Gateway befindet?**

Das Teilnetz kann nicht gelöscht werden, solange noch Instanzen vorhanden sind. Dies gilt auch für das VPN-Gateway.

**Gibt es IKE- und IPsec-Standardrichtlinien?**

Wenn Sie eine VPN-Verbindung erstellen, ohne auf eine Richtlinien-ID (IKE oder IPsec) zu verweisen, wird die automatische Vereinbarung verwendet.

**Warum muss ich bei der Bereitstellung des VPN-Gateways ein Teilnetz auswählen?**

Das Teilnetz verbindet das VPN-Gateway mit anderen Ressourcen in Ihrem VPC. Als bewährtes Verfahren wird empfohlen, ein dediziertes Teilnetz für das VPN-Gateway zu erstellen, ohne dass in diesem Teilnetz weitere VPC-Instanzen vorhanden sind, um sicherzustellen, dass im Teilnetz genügend freie private IPs vorhanden sind. Ein VPN-Gateway benötigt 8 private IP-Adressen, um HA- und Rolling-Upgrades zu empfangen.

**In welcher Zone befindet sich das VPN-Gateway?**

Das VPN-Gateway befindet sich in der Zone mit dem Teilnetz, das Sie während der Bereitstellung ausgewählt haben. Denken Sie daran, dass das VPN-Gateway nur den VPC-Instanzen in derselben Zone und dasselbe VPC bedient. Deshalb können VPC-Instanzen in anderen Zonen das VPN-Gateway nicht nutzen, um mit einem privaten On-Premises-Netz zu kommunizieren. Für die Zonenfehlertoleranz müssen Sie ein VPN-Gateway pro Zone bereitstellen.

**Was muss ich tun, wenn ich ACLs für das Teilnetz verwende, das für die Bereitstellung des VPN-Gateways verwendet wird?**

Stellen Sie sicher, dass die folgenden ACL-Regeln vorhanden sind, um den Managementdatenverkehr und den Datenverkehr im VPN-Tunnel zu ermöglichen:

* **Eingehende Regeln**
    - Protokoll-TCP-Quellenport 9091 zulassen
    - Protokoll-TCP-Quellenport 10514 zulassen
    - Protokoll TCP-Quellenport 443 zulassen
    - Protokoll-TCP-Quellenport 80 zulassen
    - Protokoll-TCP-Quellenport 53 zulassen
    - Protokoll-UDP-Quellenport 53 zulassen
    - Protokoll-ALL-Quellen-IP ist öffentliche VPN-Peer-Gateway-IP zulassen
    - Protokoll TCP-Zielport 443 zulassen
    - Protokoll TCP-Zielport 56500 zulassen
    - Datenverkehr zwischen Instanzen in VPC und Ihrem privaten On-Premise-Netz zulassen
    - ICMP-Datenverkehr zulassen

* **Abgehende Regeln**
   - Gesamten Datenverkehr zulassen

**Was muss ich tun, wenn ich Zugriffssteuerungslisten (ACLs) oder Sicherheitsgruppen in den Teilnetzen verwende, die mit dem privaten On-Premises-Netz kommunizieren müssen?**

Sie müssen sicherstellen, dass die ACL-Regeln oder Sicherheitsgruppenregeln vorhanden sind, um den Datenverkehr zwischen Instanzen in Ihrem VPC und Ihrem privaten On-Premises-Netz zu ermöglichen.

**Unterstützt VPN für VPC HA-Konfigurationen für Hochverfügbarkeit?**

Ja, die HA-Konfiguration wird in einer Aktiv/Standby-Konfiguration unterstützt.

**Ist eine Unterstützung von SSL-VPN geplant?**

Nein, nur IPsec Site-to-Site wird unterstützt.

**Gibt es bei Site-to-Site VPNaaS Obergrenzen für den Durchsatz?**

Ein Durchsatz von 650 MB/s wird unterstützt.

**Wird die PSK- und zertifikatbasierte IKE-Authentifizierung für VPNaaS unterstützt?**

Es wird nur die PSK-Authentifizierung unterstützt.

**Kann VPN für VPC als VPN-Gateway für IBM Cloud Infrastructure Classic verwendet werden?**

Nein, um ein VPN-Gateway in Ihrer IBM Cloud Infrastructure Classic-Umgebung verwenden zu können, müssen Sie das [IPsec-VPN](https://cloud.ibm.com/catalog/infrastructure/ipsec-vpn) verwenden.
