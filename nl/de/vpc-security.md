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

# Sicherheit in Ihrer IBM Cloud-VPC
{: #security-in-your-ibm-cloud-vpc}

Sie können Ihre VPC und Workloads schützen, indem Sie den Netzverkehr mithilfe von Sicherheitsgruppen (SGs) steuern, oder Listen für die Netzzugriffssteuerung (ACLs, Access Control Lists) verwenden, oder beide Arten von Steuerung verwenden. Berücksichtigen Sie Folgendes:

* Sicherheitsgruppen steuern den Datenverkehr pro Instanz (VSI).
* Die Zugriffssteuerungslisten steuern den Datenverkehr pro Teilnetzbasis.

## Sicherheitsübersicht
{: #security-overview}

* Der Datenverkehr von und zu einem Teilnetz kann durch Zugriffssteuerungslisten (ACLs) gesteuert werden.
* Sicherheitsgruppen (SGs) können den Datenverkehr auf VSI-Ebene steuern.
* Richten Sie ein öffentliches Gateway für den Teilnetzzugriff auf das Internet ein, der durch ACLs geschützt ist.
* Richten Sie einen variable IP-Adresse für VSI-Zugriff auf das Internet ein, der durch ACLs geschützt ist.

![IBM VPC - Konnektivität und Sicherheit](images/vpc-connectivity-and-security.svg "IBM VPC - Konnektivität und Sicherheit")

## Definitionen
{: #definitions}

Dieses [Glossar](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary) liefert Definitionen und Beschreibungen von ACLs und SGs sowie der Aktionen, die Sie mit ihnen ausführen können. Im folgenden Abschnitt wird die Basisfunktionalität von Zugriffssteuerungslisten (ACLs) und Sicherheitsgruppen (SGs) sowie die Unterstützung der durchgängigen Verschlüsselung (End-to-End-Verschlüsselung) durch VPC beschrieben.

### Zugriffssteuerungsliste (ACL, Access Control List)
{: #access-control-list}

Eine **Zugriffssteuerungsliste (ACL)** kann eingehenden und abgehenden Datenverkehr für ein Teilnetz verwalten, d. h. zulassen oder zurückweisen. Eine Zugriffssteuerungsliste ist statusunabhängig, was bedeutet, dass die Regeln für ein- und abgehende Daten gesondert und explizit angegeben werden müssen. Jede Zugriffssteuerungsliste besteht aus Regeln, die auf einer *Quellen-IP*, einem *Quellenport*, einer *Ziel-IP*, einem *Zielport* und einem *Protokoll* basieren.

In {{site.data.keyword.cloud}} VPC wird jedes Teilnetz mit einer Standard-ACL erstellt, die ein- und abgehenden Datenverkehr zulässt. Kunden können jedoch angepasste ACLs erstellen. Zu jedem Zeitpunkt ist immer nur jeweils eine ACL an das Teilnetz angehängt. Eine ACL kann jedoch an mehrere Teilnetze angehängt sein. Weitere Informationen zur Verwendung von Zugriffssteuerungslisten finden Sie im [Leitfaden für Zugriffssteuerungslisten (ACLs)](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls).

### Sicherheitsgruppe
{: #security-group}

Eine **Sicherheitsgruppe** fungiert als virtuelle Firewall, die den Datenverkehr für einen oder mehrere Server (VSIs) steuert. Eine Sicherheitsgruppe ist eine Sammlung von Regeln, die angeben, ob Datenverkehr für eine zugeordnete virtuelle Serverinstanz (VSI) zugelassen werden soll. 

Wenn ein Kunde eine VSI erstellt, kann er dieser VSI eine oder mehrere Sicherheitsgruppen zuordnen. Sofern sie über die richtigen Berechtigungen verfügen, können Kunden Änderungen an den Regeln für Sicherheitsgruppen über die IBM Konsole, die Befehlszeilenschnittstelle (CLI) oder die Anwendungsprogrammierschnittstelle (API) vornehmen.

Weitere Informationen zum Erstellen einer virtuellen Serverinstanz (VSI), die Sicherheitsgruppen verwendet, und weitere Informationen zu der Funktionalität von Sicherheitsgruppen enthält der [Leitfaden für Sicherheitsgruppen](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-using-security-groups).

### Durchgängige Verschlüsselung (End-to-End-Verschlüsselung)
{: #end-to-end-encryption}

IBM Cloud VPC stellt zwar keine End-to-End-Verschlüsselung bereit, lässt sie jedoch zu. Wenn Sie beispielsweise über einen sicheren Endpunkt auf einem virtuellen Server verfügen (z. B. einem HTTPS-Server an Port 443), so können Sie an diesen Server eine variable IP-Adresse anhängen. Dadurch ist Ihre Verbindung vom Client bis zum Server an Port 443 durchgängig verschlüsselt.  Nichts im Pfad erzwingt eine Entschlüsselung.

Beachten Sie jedoch, dass Ihre Daten bei Verwendung eines unsicheren Protokolls wie HTTP an Port 80 durchgängig unverschlüsselt sind.
