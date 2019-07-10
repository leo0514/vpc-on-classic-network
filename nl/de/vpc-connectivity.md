---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-14"

keywords: capabilities, use cases, subnets, VPN, connections, reserved, IP, IPv4, floating

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Informationen zum Netzbetrieb für VPC
{: #about-networking-for-vpc}

Im vorliegenden Dokument werden Konzepte behandelt, die Anwendungsfälle und die Funktionalität innerhalb von {{site.data.keyword.cloud}} VPC für Teilnetze, variable IP-Adressen und VPN-Verbindungen beschreiben.

![IBM VPC - Konnektivität und Sicherheit](images/vpc-connectivity-and-security.svg "IBM VPC - Konnektivität und Sicherheit")

Die Abbildung veranschaulicht Folgendes:

* Teilnetze können über ein optionales öffentliches Gateway (PGW, Public Gateway) eine Verbindung zum öffentlichen Internet herstellen.
* Jeder virtuellen Serverinstanz (VSI, Virtual Server Instance) kann eine variable IP-Adresse zugewiesen werden, damit sie , damit sie vom Internet aus erreichbar ist oder damit sie das Internet erreichen kann.
* Teilnetze in IBM Cloud VPC bieten private Konnektivität und können über eine private Verbindung miteinander kommunizieren. Sie müssen keine Routen einrichten.
* Weitere Informationen enthalten die [Informationen zu VPC Infrastructure](/docs/vpc-on-classic?topic=vpc-on-classic-about).

## Terminologie
{: #terminology}

Dieses [Glossar](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary) enthält Definitionen und Informationen zu den Begriffen, die in diesem Dokument für IBM Cloud VPC verwendet werden.

## Merkmale von Teilnetzen in der VPC
{: #characteristics-of-subnets-in-the vpc}

Ein Teilnetz besteht aus einem angegebenen IP-Adressbereich (CIDR-Block). Teilnetze sind auf eine einzelne Zone beschränkt und können sich nicht über mehrere Zonen oder Regionen erstrecken. Ein Teilnetz kann jedoch die Gesamtheit der Zonenabstraktionen in ihrer virtuellen privaten Cloud umfassen. Teilnetze in derselben IBM Cloud-VPC sind miteinander verbunden.

### Zonen
{: #zones}

Eine Zone ist eine Abstraktion, die zur Unterstützung durch eine verbesserte Fehlertoleranz und eine verringerte Latenzzeit konzipiert wurde. Eine Zone stellt die folgenden Eigenschaften sicher:

 * Jede Zone ist eine unabhängige Fehlerdomäne und es ist äußerst unwahrscheinlich, dass zwei Zonen in einer Region gleichzeitig ausfallen.
 * Für den Datenverkehr zwischen den Zonen in einer Region trifft eine Latenzzeit von weniger als 2 Millisekunden zu.

### Reservierte IP-Adressen
{: #reserved-ip-addresses}

Bestimmte IP-Adressen sind bei Verwendung der virtuellen privaten Cloud für die Verwendung durch IBM reserviert. Nachfolgend sind die reservierten Adressen aufgeführt (wobei bei den angegebenen IP-Adressen vorausgesetzt wird, dass der CIDR-Bereich des Teilnetzes 10.10.10.10.0/24 lautet):

  * Erste Adresse im CIDR-Bereich (10.10.10.0): Netzadresse
  * Zweite Adresse im CIDR-Bereich (10.10.10.1): Gateway-Adresse
  * Dritte Adresse im CIDR-Bereich (10.10.10.2): durch IBM reserviert
  * Vierte Adresse im CIDR-Bereich (10.10.10.3): zur späteren Nutzung durch IBM reserviert
  * Letzte Adresse im CIDR-Bereich (10.10.10.255): Netzbroadcastadresse

### Öffentliches Gateway für die externe Konnektivität eines Teilnetzes verwenden
{: #use-a-public-gateway-for-external-connectivity-of-a-subnet}

Ein **öffentliches Gateway** ermöglicht einem Teilnetz (und allen an dieses Teilnetz angebundenen virtuellen Serverinstanzen) die Verbindungsherstellung zum Internet. Beachten Sie, dass Teilnetze standardmäßig privat sind. Sie können jedoch optional ein öffentliches Gateway (PGW) erstellen und an dieses ein Teilnetz anhängen. Nachdem ein Teilnetz an das öffentliche Gateway angehängt worden ist, können alle virtuelle Serverinstanzen in diesem Teilnetz eine Verbindung mit dem Internet herstellen.

Das öffentliche Gateway verwendet die _Netzadressumsetzung (NAT, Network Address Translation) vom Typ 'Viele-zu-eins'_, was bedeutet, dass mehrere Tausend virtuelle Serverinstanzen mit privaten Adressen genau eine öffentliche IP-Adresse verwenden, um mit dem öffentlichen Internet zu kommunizieren.Das öffentliche Gateway ermöglicht dem Internet nicht, eine Verbindung zu diesen Instanzen zu starten. Verwenden Sie die Anwendungsprogrammierschnittstelle (API), um Teilnetze an Ihr öffentliches Gateway anzuhängen oder von diesem abzuhängen.

In der folgenden Abbildung wird der Geltungsbereich von Gateway-Services zusammengefasst.

![Gateway-Services](images/scope-of-gateway-services.png)

### Variable IP-Adresse für die externe Konnektivität einer virtuellen Servierinstanz (VSI) verwenden
{: #use-a-floating-ip-address-for-external-connectivity-of-a-vsi}

**Variable IP-Adressen** sind IP-Adressen, die vom System bereitgestellt werden und vom öffentlichen Internet aus erreichbar sind.

Sie können eine variable IP-Adresse aus dem Pool der von IBM bereitgestellten verfügbaren variablen IP-Adressen reservieren und diese reservierte IP-Adresse jeder beliebigen Instanz in derselben VPC zuordnen, und zwar mithilfe der virtuellen Netzschnittstellenkarte (vNIC, Virtual Network Interface Card) für diese Instanz. Jede variable IP-Adresse kann verschiedenen Typen von virtuellen Serverinstanzen (VSIs) zugeordnet werden, z. B. einer Lastausgleichsfunktion (Load Balancer) oder einem VPN-Gateway.

Ihre variable IP-Adresse kann nicht mehreren Schnittstellen zugeordnet werden.Sie müssen die Schnittstelle auf der virtuellen Serverinstanz (VSI) angeben, die dieser einzelnen variablen IP-Adresse zugeordnet werden soll. Diese Schnittstelle verfügt ebenfalls über eine private IP-Adresse. Das Back-End-System führt Operationen für die _Netzadressumsetzung (NAT, Network Address Translation) vom Typ 'Eins-zu-eins'_ zwischen der variablen IP-Adresse und der privaten IP-Adresse dieser Schnittstelle aus. Eine variable IP-Adresse wird nur dann freigegeben, wenn Sie die Freigabeaktion explizit anfordern.

**Hinweise:**
* **Durch die Zuordnung einer variablen IP-Adresse zu einer virtuellen Serverinstanz (VSI) wird die VSI aus der zuvor erwähnten Netzadressumsetzung (NAT) vom Typ 'Viele-zu-eins' des öffentlichen Gateways entfernt.**
* **Gegenwärtig werden nur IPv4-Adressen als variable IP-Adressen unterstützt.**
* **Sie können Ihre eigene öffentliche IP-Adresse nicht als variable IP-Adresse verwenden. **

Weitere Informationen zu NAT-Operationen können Sie [im zugehörigen RFC-Dokument im Internet ![Symbol für externen Link](../../icons/launch-glyph.svg "Symbol für externen Link")](http://www.faqs.org/rfcs/rfc1631.html){: new_window} nachlesen.

### VPN für sichere externe Konnektivität verwenden
{: #use-a-vpn-for-secure-external-connectivity}

Der VPN-Service ermöglicht Benutzern, sich sicher aus dem Internet mit ihrer IBM Cloud VPC-Instanz zu verbinden. Schrittweise Anweisungen dazu können Sie im [Handbuch zur IBM Console UI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console) nachlesen.

**VPN-Funktionalität**
  * CRUD-Fähigkeit (Create, Read, Update, and Delete - Erstellen, Lesen, Aktualisieren und Löschen) des VPN-Service (IPsec-VPN für Site-to-Site-Konnektivität) für die Datenübertragung.
  * Fähigkeit, einen VPN-Service zu der IBM Cloud VPC-Instanz oder dem virtuellen Netz eines Kunden zuzuordnen.
  * Fähigkeit, die lokalen Teilnetze des Kunden entweder durch statische Definition oder durch dynamisches Routing zu identifizieren.
  * Unterstützung von sicheren Verschlüsselungscodes wie SHA256, AES, 3DES oder IKEv2.
  * Zuverlässigkeit des integrierten Service.
  * Überwachung des Status der VPN-Verbindung.
  * Unterstützung des Initiator- wie auch des Respondermodus, was bedeutet, dass der Datenverkehr von beiden Seiten des Tunnels eingeleitet werden kann.
