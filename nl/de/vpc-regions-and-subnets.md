---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-07"

keywords: vpc, address prefix, region, subnet, zone, reserved, IP, ranges, deleting, creating, CIDR

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

# IP-Adressbereiche verstehen, Adresspräfixe, Regionen und Teilnetze verstehen
{: #working-with-ip-address-ranges-address-prefixes-regions-and-subnets}
[comment]: # (Verlinktes Hilfethema)

In diesem Dokument werden die Beziehungen zwischen Regionen, Adresspräfixen und Teilnetzen für {{site.data.keyword.cloud}}-VPC besprochen:

* VPCs werden bereitgestellt und an eine Region gebunden.
* Innerhalb dieser einen Region können VPCs mehrere Zonen umfassen.
* Adresspräfixe ermöglichen die Kommunikation zwischen den Zonen, die eine VPC umfasst. Jede VPC erhält für jede Zone, die sie umfasst, ein abgekürztes Standard-Adressenpräfix.
* Teilnetze werden im Rahmen eines Adresspräfixes erstellt, d. h., ein Teilnetz muss vollständig in einem einzigen vorhandenen Adressenpräfix enthalten sein.
* Wenn Sie einen IP-Bereich außerhalb der Bereiche verwenden, die durch [RFC 1918](https://tools.ietf.org/html/rfc1918) (`10.0.0.0/8`, `172.16.0.0/12` oder `192.168.0.0/16`) für ein Teilnetz definiert sind, können die an diesem Teilnetz angeschlossenen Instanzen keine Teile des öffentlichen Internets erreichen.

## IBM Cloud-VPC und Regionen
{: #ibm-cloud-vpc-and-regions}

Eine {{site.data.keyword.cloud}}-VPC wird in einer Region bereitgestellt, kann sich aber über mehrere Zonen erstrecken. Jede Region enthält mehrere Zonen, die jeweils unabhängige Fehlerdomänen darstellen.

## IBM Cloud-VPC und Adresspräfixe
{: #ibm-cloud-vpc-and-address-prefixes}

Adresspräfixe ermöglichen die Kommunikation zwischen {{site.data.keyword.cloud_notm}}-VPC-Instanzen in verschiedenen Zonen. Sie stellen Routing-Informationen bereit, die der _implizite Router_ benötigt, um Daten an die Zone zu senden, in der sich die Zielinstanz befindet. Jedes Teilnetz muss durch ein Adresspräfix enthalten sein. {{site.data.keyword.cloud_notm}}-VPC unterstützt das Konzept von "lokalen" oder "nicht erreichbaren" Subnetzen nicht. 

Der _implizite Router_ ist die inhärente Netzkonnektivität zwischen allen Teilnetzen, die innerhalb einer VPC erstellt wurden.
{: note}

Jede {{site.data.keyword.cloud_notm}}-VPC kann bis zu fünf Adresspräfixe für jede Zone haben. Um Ihnen den Einstieg zu erleichtern, definiert die {{site.data.keyword.cloud_notm}}-VPC ein Standardadressenpräfix für jede Zone (siehe die folgende Tabelle). Als bewährtes Verfahren sollten Sie jedoch einen VPC-Adressierungsplan entwerfen, bevor Sie ihn bereitstellen.

### VPC-Standardadressenpräfixe
{: #default-vpc-address-prefixes}

Wenn eine neue VPC erstellt wird, werden die Standardadresspräfixe auf der Grundlage der Region und der Zone wie folgt zugewiesen.

[Classic Access
VPCs](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#classic-access-default-address-prefixes) haben einen anderen Satz von Standardadressenpräfixen.
{: important}

Zone         | Adresspräfix
---------------|---------------
`us-south-1`   | `10.240.0.0/18`
`us-south-2`   | `10.240.64.0/18`
`us-south-3`   | `10.240.128.0/18`
`eu-de-1`      | `10.243.0.0/18`
`eu-de-2`      | `10.243.64.0/18`
`eu-de-3`      | `10.243.128.0/18`
`jp-tok-1`     | `10.244.0.0/18`
`jp-tok-2`     | `10.244.64.0/18`
`jp-tok-3`     | `10.244.128.0/18`

Den neuen Zonen oder Regionen werden unterschiedliche Standardpräfixe zugeordnet.

### Adresspräfixe und die Benutzerschnittstelle (UI) der IBM Cloud-Konsole
{: #address-prefixes-and-the-ibm-cloud-console-ui}

Wenn Sie eine VPC mit der Benutzerschnittstelle (UI) der IBM Cloud-Konsole erstellen, wählt das System automatisch Ihr Adresspräfix aus, innerhalb dessen Sie ein Teilnetz erstellen müssen. Falls dieses Adressschema nicht Ihren Anforderungen entspricht, können Sie die Adresspräfixe nach dem Erstellen der VPC anpassen. Dann können Sie in Ihren benutzerdefinierten Adresspräfixen Teilnetze erstellen und das mit dem Standardpräfix erstellte Teilnetz löschen.

Diese Ausweichlösung ist erforderlich, um BYOIP (Bring Your Own IP) über die Benutzerschnittstelle (UI) der IBM Cloud-Konsole nutzen zu können.
{:note}

## IBM Cloud-VPC und Teilnetze
{: #ibm-cloud-vpc-and-subnets}

Eine {{site.data.keyword.cloud_notm}}-VPC kann in Teilnetze unterteilt werden. Alle Teilnetze in einer {{site.data.keyword.cloud_notm}}-VPC können sich gegenseitig über privates L3-Routing mit einem impliziten Router erreichen.Sie müssen keine Router oder Routen einrichten.

Hier einige praktische Fakten zu Teilnetzen in VPC:

* Ein Teilnetz besteht aus einem IP-Adressbereich, den Sie angeben.
* Ein Teilnetz ist an eine einzelne Zone gebunden und kann sich nicht über mehrere Zonen oder Regionen erstrecken.
* Ein Teilnetz kann die gesamte Zone in einer {{site.data.keyword.cloud_notm}}-VPC umfassen.
* Sie müssen zuerst Ihre virtuelle private Cloud (VPC) erstellen, bevor Sie Ihr(e) Teilnetz(e) in dieser VPC erstellen können.
* Es ist keine Unterstützung für IPv6 verfügbar.
* Sie können eine VSI einem Teilnetz zuordnen oder ihre Zuordnung aufheben. (Dazu müssen Sie zuerst eine vNIC hinzufügen und eine Bandbreite auswählen.)
* Jedes Teilnetz muss in einem Adresspräfix enthalten sein, das zu der Zone gehört, in die das Teilnetz eingebunden ist.

Sie können Ihren eigenen öffentlichen IPv4-Adressbereich (BYOIP) in Ihr {{site.data.keyword.cloud_notm}}-VPC-Konto einbringen. Wenn Sie BYOIP (Bring Your Own IP) verwenden, muss {{site.data.keyword.cloud_notm}} diese IPv4-Adressen auf {{site.data.keyword.cloud_notm}}-Ressourcen konfigurieren, die ihrerseits Pakete an und von den bereitgestellten Adressen senden. Die Verwendung Ihres bereitgestellten IPv4-Bereichs in {{site.data.keyword.cloud_notm}} hat zur Folge, dass diese IP-Adressen möglicherweise gegenüber Mitarbeitern von IBM Support sowie gegenüber Dritten als Teil der Nutzung dieses Service offengelegt werden.
{:important}

![Überblick über IBM Cloud-VPC](images/vpc-experience.svg "Überblick über IBM Cloud-VPC")

### Verwendung von Adresspräfixen für Teilnetze
{: #using-address-prefixes-for-subnets}

Jedes Teilnetz muss innerhalb eines Adresspräfixes bestehen.
 * Für Ihr neues Teilnetz können Sie Ihren IP-Adressbereich aus den vorhandenen Adresspräfixen auswählen.
 * Wenn die Adresspräfixe der Zone nicht geeignet sind, kann eines der vorhandenen Adresspräfixe bearbeitet oder ein neues Adresspräfix für die Zone (über die API, die CLI oder die Benutzerschnittstelle) hinzugefügt werden.

### Verfügbare IP-Adressen
{: #available-ip-addresses}

Die folgenden IP-Adressen sind verfügbar, wie in **RFC 1918** definiert:

 * 10.0.0.0 – 10.255.255.255
 * 172.16.0.0 – 172.31.255.255
 * 192.168.0.0 – 192.168.255.255

Wenn Sie einen IP-Bereich verwenden, der sich außerhalb der zulässigen Bereiche für ein Teilnetz befindet (in vorherigen Abschnitten angegeben), können die an diesem Teilnetz angeschlossenen Instanzen keine Teile des öffentlichen Internets erreichen.

### Weitere Informationen zur Erstellung eines Teilnetzes
{: #more-about-creating-a-subnet}

Sie können ein Teilnetz für Ihre VPC auf zwei Arten angeben:
  * Sie können ein Teilnetz erstellen, indem Sie die Größe des benötigten Teilnetzes angeben, beispielsweise durch die Anzahl der unterstützten Adressen (z. B. 1024).
  * Sie können ein Teilnetz erstellen, indem Sie einen CIDR-Bereich angeben (z. B. 10.0.0.8/29).

Das folgende Beispiel verdeutlicht, wie Sie einen Block von 1024 mithilfe von CIDR angeben: Wenn Sie anstatt einer Teilnetzgröße einen CIDR-Bereich angeben, stellt der IPv4-Block `192.168.100.0/22` die 1024 IPv4-Adressen von `192.168.100.0` bis `192.168.103.255` dar.
{:tip}

