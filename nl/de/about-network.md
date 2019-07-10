---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: secure, region, zone, subnet, terminology, public gateway, floating IP, NAT

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:download: .download}

# Informationen zum Netzbetrieb für VPC
{: #about-networking-for-vpc}

Eine virtuelle private Cloud (VPC, Virtual Private Cloud) ist ein virtuelles Netz, das an Ihr Kundenkonto gebunden ist. Es gibt Ihnen Cloudsicherheit und ermöglicht anhand der bereitgestellten feinkörnigen Steuerung der virtuellen Infrastruktur und der Segmentierung Ihres Datenverkehrs eine dynamische Skalierung.

In diesem Dokument werden einige Konzept des Netzbetriebs behandelt, wie sie in {{site.data.keyword.cloud}} VPC Anwendung gefunden haben. Es werden unter anderem die folgenden Anwendungsfälle und Merkmale beschrieben:

* Regionen
* Zonen
* Reservierte IP-Adressen
* Öffentliche Gateways
* vNIC-Schnittstellen
* Teilnetze
* Variable IP-Adressen
* VPN-Verbindungen

## Übersicht
{: #subnets-overview}

Für den Zugriff auf das Internet von Ihrem VPC stehen drei Optionen zur Verfügung:
* Verwenden Sie ein öffentliches Gateway für den Internetdatenverkehr aus dem gesamten Teilnetz.
* Verwenden Sie eine variable IP-Adresse für den Internetdatenverkehr von und zu einer Instanz.
* Verwenden Sie ein VPN für sichere externe Konnektivität.

Berücksichtigen Sie Folgendes:
* Manche Teilnetzadressbereiche für CIDR sind durch IBM reserviert.
* Sie müssen Ihre VPC erstellen, bevor Sie Teilnetze in dieser VPC erstellen können.
* Es ist keine Unterstützung für IPv6 verfügbar.

Optional können Sie eine VPC mit klassischem Zugriff für die Verbindungsherstellung zur klassischen Infrastruktur Ihrer IBM Cloud erstellen.
{:note}

Die nachstehende Abbildung veranschaulicht Folgendes:
* Teilnetze in Ihrer virtuellen privaten Cloud (VPC) können über ein optionales öffentliches Gateway (PGW, Public Gateway) eine Verbindung zum öffentlichen Internet herstellen.
* Sie können einer beliebigen Instanz eine variable IP-Adresse zuordnen, damit sie vom Internet aus erreichbar ist (oder umgekehrt), unabhängig davon, ob das Teilnetz an ein öffentliches Gateway angeschlossen ist.
* Teilnetze innerhalb der VPC von IBM Cloud bieten private Konnektivität und können durch den impliziten Router über eine private Verbindung miteinander kommunizieren. Eine Einrichtung von Routen ist nicht erforderlich.

![IBM VPC - Konnektivität und Sicherheit](images/vpc-connectivity-and-security.svg "IBM VPC - Konnektivität und Sicherheit")

## Terminologie
{: #network-terminology}

Dieses [Glossar](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary#vpc-glossary) enthält Definitionen und Informationen zu den Begriffen, die in diesem Dokument für IBM Cloud VPC verwendet werden. Wenn Sie mit Ihrer virtuellen privaten Cloud (VPC) arbeiten, müssen Sie mit den Grundkonzepten der Begriffe _Region_ und _Zone_ vertraut sein, da diese sich auf Ihre Bereitstellung beziehen.

### Regionen
{: #subnet-regions}

Eine Region ist eine Abstraktion, die sich auf den geografischen Bereich bezieht, in dem ein VPC bereitgestellt wird. Jede Region enthält mehrere Zonen, die jeweils unabhängige Fehlerdomänen darstellen. Eine IBM Cloud-VPC kann sich in der ihr zugewiesenen Region über mehrere Zonen erstrecken.

### Zonen
{: #subnet-zones}

Eine Zone ist eine Abstraktion, die sich auf das physische Rechenzentrum bezieht, das die Rechen-, Netz- und Speicherressourcen sowie die damit verbundene Kühlung und Stromversorgung hostet und die Services und Anwendungen bereitstellt. Die Isolierung von Zonen verbessert die allgemeine Fehlertoleranz des Systems, verringert die Latenz und vermeidet die Erstellung eines einzelnen gemeinsamen Fehlerpunkts. Eine Zone stellt die folgenden Eigenschaften sicher:

 * Jede Zone ist eine unabhängige Fehlerdomäne und es ist äußerst unwahrscheinlich, dass zwei Zonen in einer Region gleichzeitig ausfallen.
 * Für den Datenverkehr zwischen den Zonen in einer Region trifft eine Latenzzeit von weniger als 2 Millisekunden zu.

## Merkmale von Teilnetzen in der VPC
{: #characteristics-of-subnets}

Ein Teilnetz besteht aus einem angegebenen IP-Adressbereich (CIDR-Block). Teilnetze sind auf eine einzelne Zone beschränkt und können sich nicht über mehrere Zonen oder Regionen erstrecken. Ein Teilnetz kann jedoch die Gesamtheit der Zonenabstraktionen in ihrer virtuellen privaten Cloud umfassen. Teilnetze in derselben IBM Cloud-VPC sind miteinander verbunden.

### Reservierte IP-Adressen
{: #reserved-ip-addresses}

Bestimmte IP-Adressen sind bei Verwendung der virtuellen privaten Cloud für die Verwendung durch IBM reserviert. Vorausgesetzt, dass der CIDR-Bereich des Subnetzes 10.10.10.0/24 ist, lauten diese reservierten Adressen wie folgt:

  * Erste Adresse im CIDR-Bereich (10.10.10.0): Netzadresse
  * Zweite Adresse im CIDR-Bereich (10.10.10.1): Gateway-Adresse
  * Dritte Adresse im CIDR-Bereich (10.10.10.2): durch IBM reserviert
  * Vierte Adresse im CIDR-Bereich (10.10.10.3): zur späteren Nutzung durch IBM reserviert
  * Letzte Adresse im CIDR-Bereich (10.10.10.255): Netzbroadcastadresse

### Öffentliches Gateway für die externe Konnektivität eines Teilnetzes verwenden
{: #use-a-public-gateway}

Ein **öffentliches Gateway** ermöglicht einem Teilnetz (und allen an dieses Teilnetz angebundenen Instanzen) die Verbindungsherstellung zum Internet. Beachten Sie, dass Teilnetze standardmäßig privat sind. Sie können jedoch optional ein öffentliches Gateway (PGW) erstellen und an dieses ein Teilnetz anhängen. Nachdem ein Teilnetz an das öffentliche Gateway angehängt worden ist, können alle Instanzen in diesem Teilnetz eine Verbindung mit dem Internet herstellen.

Das öffentliche Gateway verwendet die _Netzadressumsetzung (NAT, Network Address Translation) vom Typ 'Viele-zu-eins'_, was bedeutet, dass mehrere Tausend Instanzen mit privaten Adressen genau eine öffentliche IP-Adresse verwenden, um mit dem öffentlichen Internet zu kommunizieren. Das öffentliche Gateway ermöglicht dem Internet nicht, eine Verbindung zu diesen Instanzen zu starten. Verwenden Sie die Anwendungsprogrammierschnittstelle (API), um Teilnetze an Ihr öffentliches Gateway anzuhängen oder von diesem abzuhängen.

Die folgende Abbildung stellt eine Zusammenfassung des derzeitigen Geltungsbereichs von Gateway-Services dar.

| SNAT | DNAT | ACL | VPN |
| ---- | ---- | --- | --- |
| Instanzen können nur ausgehenden Zugriff auf das Internet haben | Eingehende Konnektivität aus dem Internet zu einer privaten IP-Adresse zulassen | Eingeschränkten eingehenden Zugriff vom Internet auf Instanzen oder Teilnetze bereitstellen | Site-to-Site-VPN verarbeitet Kunden beliebiger Größe und einzelne oder mehrere Standorte |
| Ganze Teilnetze nutzen denselben abgehenden öffentlichen Endpunkt | Bietet eingeschränkten Zugriff auf einen einzelnen privaten Server | Eingehenden Zugriff über das Internet basierend auf Service, Protokoll oder Port beschränken | Hoher Durchsatz (bis zu 10 Gb/s) bietet Kunden die Möglichkeit, große Dateien sicher und schnell zu übertragen |
| Schützt Instanzen; Zugriff auf Instanzen kann nicht über den öffentlichen Endpunkt initiiert werden | DNAT-Service kann auf der Basis von Anforderungen auf- oder abskaliert werden | Statusunabhängige ACLs ermöglichen eine differenzierte Steuerung des Datenverkehrs | Sichere Verbindungen mit Verschlüsselung nach Branchenstandard erstellen |

Ein öffentliches Gateway, das in einer VPC-Instanz erstellt wird, erfüllt so lange keinerlei Funktion, bis es an ein Teilnetz angeschlossen worden ist. Pro Zone kann nur ein öffentliches Gateway erstellt werden. Das bedeutet, dass in einer Umgebung mit drei Zonen drei (3) öffentliche Gateways pro VPC vorhanden sein können.
{:note}

## Einschränkungen der Teilnetze
{: #limitations-of-subnets}

Eine vollständige Liste der bekannten Einschränkungen und der Features, die derzeit nicht unterstützt, enthält das Dokument [Bekannte Einschränkungen](/docs/vpc-on-classic?topic=vpc-on-classic-known-limitations).

### Einschränkungen beim Löschen eines Teilnetzes
{: #restrictions-on-deleting-a-subnet}

Ein Teilnetz kann nicht gelöscht werden, solange es in diesem Teilnetz Ressourcen (wie z. B. eine virtuelle Serverinstanz bzw. VSI oder eine variable IP-Adresse) gibt, die verwendet werden. Diese Ressourcen müssen zuerst gelöscht werden.

### Einschränkungen beim Aktualisieren eines vorhandenen Teilnetzes
{: #limitations-on-updating-an-existing-subnet}

* Sie können die Größe eines vorhandenen Teilnetzes nicht ändern. Beispiel: "10.10.16.0/24" kann nicht zu "to 10.10.16.0/20" geändert werden.
* Sie können ein vorhandenes Teilnetz nicht verschieben. Beispiel: "10.10.10.0/24" kann nicht zu "to 10.10.11.0/24" verschoben werden.

## Externe Konnektivität
{: #external-connectivity}

Die externe Verbindung kann durch eine an eine Instanz angehängte variable IP-Adresse, durch ein externes Gateway, das an ein Teilnetz angeschlossen ist, oder durch einen VPN-Tunnel erreicht werden.

### Variable IP-Adresse für die externe Konnektivität einer Instanz verwenden
{: #use-floating-ip}

**Variable IP-Adressen** sind IP-Adressen, die vom System bereitgestellt werden und vom öffentlichen Internet aus erreichbar sind.

Sie können eine variable IP-Adresse aus dem Pool der von IBM bereitgestellten verfügbaren variablen IP-Adressen reservieren und diese reservierte IP-Adresse jeder beliebigen Instanz in derselben VPC zuordnen, und zwar mithilfe der virtuellen Netzschnittstellenkarte (vNIC, Virtual Network Interface Card) für diese Instanz. Jede variable IP-Adresse kann verschiedenen Typen von virtuellen Serverinstanzen (VSIs) zugeordnet werden, z. B. einer Lastausgleichsfunktion (Load Balancer) oder einem VPN-Gateway.

Ihre variable IP-Adresse kann nicht mehreren Schnittstellen zugeordnet werden.Sie müssen die Schnittstelle auf der virtuellen Serverinstanz (VSI) angeben, die dieser einzelnen variablen IP-Adresse zugeordnet werden soll. Diese Schnittstelle verfügt ebenfalls über eine private IP-Adresse. Das Back-End-System führt Operationen für die _Netzadressumsetzung (NAT, Network Address Translation) vom Typ 'Eins-zu-eins'_ zwischen der variablen IP-Adresse und der privaten IP-Adresse dieser Schnittstelle aus. Eine variable IP-Adresse wird nur dann freigegeben, wenn Sie die Freigabeaktion explizit anfordern.

**Hinweise:**
* **Durch die Zuordnung einer variablen IP-Adresse zu einer virtuellen Serverinstanz (VSI) wird die VSI aus der zuvor erwähnten Netzadressumsetzung (NAT) vom Typ 'Viele-zu-eins' des öffentlichen Gateways entfernt.**
* **Gegenwärtig werden nur IPv4-Adressen als variable IP-Adressen unterstützt.**
* **Sie können Ihre eigene öffentliche IP-Adresse nicht als variable IP-Adresse verwenden.**

Weitere Informationen zu NAT-Operationen können Sie [im zugehörigen RFC-Dokument im Internet ![Symbol für externen Link](../../icons/launch-glyph.svg "Symbol für externen Link")](http://www.faqs.org/rfcs/rfc1631.html){: new_window} nachlesen.

### VPN für sichere externe Konnektivität verwenden
{: #use-vpn}

Der VPN-Service ermöglicht Benutzern, sich sicher aus dem Internet mit ihrer IBM Cloud VPC-Instanz zu verbinden. 

**VPN-Funktionalität**
  * CRUD-Fähigkeit (Create, Read, Update, and Delete - Erstellen, Lesen, Aktualisieren und Löschen) des VPN-Service (IPsec-VPN für Site-to-Site-Konnektivität) für die Datenübertragung.
  * Fähigkeit, einen VPN-Service zu der IBM Cloud VPC-Instanz oder dem virtuellen Netz eines Kunden zuzuordnen.
  * Fähigkeit, die lokalen Teilnetze des Kunden entweder durch statische Definition oder durch dynamisches Routing zu identifizieren.
  * Unterstützung von sicheren Verschlüsselungscodes wie SHA256, AES, 3DES oder IKEv2.
  * Zuverlässigkeit des integrierten Service.
  * Fortlaufende Überwachung des Status der VPN-Verbindung.
  * Unterstützung des Initiator- wie auch des Respondermodus, was bedeutet, dass der Datenverkehr von beiden Seiten des Tunnels eingeleitet werden kann.

## Weitere Informationen
{: #subnets-learn-more}
   * [Sicherheit bei IBM VPC](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-security-in-your-ibm-cloud-vpc)
   * [IBM VPC-Adressen, Regionen und Teilnetze](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-working-with-ip-address-ranges-address-prefixes-regions-and-subnets)
