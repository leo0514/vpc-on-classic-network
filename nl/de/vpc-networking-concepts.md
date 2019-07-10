---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-29"

keywords: VRF, router, hypervisor, address prefixes, classic access, implicit router, packet flows, NAT, data flows

subcollection: vpc-on-classic-network


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}

# VPC unter genauerer Betrachtung
{: #vpc-behind-the-curtain}

Diese Seite vermittelt einen detaillierteren konzeptionellen Einblick darauf, was in Bezug auf den VPC-Netzbetrieb 'hinter den Kulissen' abläuft. Bei den Lesern wird ein gewisser Hintergrund im Bereich des Netzbetriebs erwartet.

## Isolation von Netzen
{: #network-isolation}

Die Isolation des VPC-Netzes findet auf drei Ebenen statt:

* **Hypervisor**: Die virtuellen Serverinstanzen bzw. VSIs (Virtual Server Instances) sind durch den Hypervisor selbst isoliert. Eine VSI kann andere VSIs, die von demselben Hypervisor per Hosting bereitgestellt werden, nicht direkt erreichen, wenn sich die VSIs nicht alle in derselben VPC befinden.

* **Netz**: Die Isolation auf Netzebene erfolgt durch die Verwendung **virtueller Netz-IDs** bzw. VNIs (Virtual Network Identifiers). Diese IDs gelten für die lokale Zone. Diese VNIs werden allen Datenpaketen hinzugefügt, die eine beliebige Zone der VPC betreten: entweder vom Hypervisor aus, wenn sie von einem VSI gesendet werden, oder von der Cloud aus, wenn sie von der impliziten Routing-Funktion gesendet werden.

Bei einem Paket, das eine Zone verlässt, wird die VNI entfernt. Wenn das Paket seine Zielzone erreicht und über die implizite Routing-Funktion eingeht, fügt der implizite Router immer die richtige VNI für diese Zone hinzu.
{: note}

* **Router**: Die _implizite Routerfunktion_ stellt die Isolation für jede VPC sicher, indem sie eine **virtuelle Routingfunktion** (VRF) und ein VPN mit MPLS (Multi-Protocol Label Switching) im Cloud-Backbone bereitstellt. Die virtuelle Routingfunktion (VRF) einer jeden VPC besitzt eine eindeutige Kennung und diese Isolation ermöglicht jeder VPC den Zugriff auf ihre eigene Kopie des IPv4-Adressraums. Das VPN mit MPLS lässt die Einbindung aller Randbereiche (Edges) der Cloud zu: klassische Infrastruktur, Direct Link und VPC.

## Adresspräfixe
{: #address-prefixes}

Adresspräfixe sind die Übersichtsinformationen, die von der impliziten Routingfunktion einer VPC verwendet werden, um eine _Ziel-VSI_ zu lokalisieren, und zwar unabhängig von der Verfügbarkeitszone, in der sich diese Ziel-VSI befindet. Die Hauptfunktion von Adresspräfixen besteht in der Optimierung des Routings über das MPLS-VPN unter gleichzeitiger Vermeidung pathologischer Routingfälle. Alle in einer VPC erstellten Teilnetze müssen in einem Adresspräfix enthalten sein, sodass alle VSIs in einer VPC von allen anderen VSIs in der VPC erreicht werden können.

## Datenflüsse von Datenpaketen und der implizite Router
{: #data-packet-flows-and-the-implicit-router}

In einer VPC treten sechs unterschiedliche Typen von Datenflüssen für die VSI-Datenpakete auf. Dabei handelt es sich um die folgenden Datenflüsse, die hier nach steigender Komplexität genannt werden:

* Innerhalb desselben Teilnetzes und innerhalb desselben Hosts (derselbe Hypervisor)
* Innerhalb desselben Teilnetzes und auf verschiedenen Hosts
* Zwischen verschiedenen Teilnetzen und innerhalb derselben Zone
* Zwischen verschiedenen Teilnetzen und in verschiedenen Zonen
* Extra-VPC-Service (für IaaS- oder CSE-Zugang)
* Extra-VPC-Internet (für Internetzugang)

Datenflüsse **innerhalb desselben Teilnetzes und innerhalb desselben Hosts**: Diese Datenflüsse weisen die geringste Komplexität auf. Die Übertragung von Paketen erfolgt zwischen den VSIs auf dem Hypervisor, wobei keine Pakete den Hypervisor verlassen.

Datenflüsse **innerhalb desselben Teilnetzes und auf verschiedenen Hosts**: Bei diesen Datenflüssen verlassen Pakete den Hypervisor. Daher wird jedes Paket zur Sicherstellung der Datenisolierung zuerst mit der ordnungsgemäßen virtualisierten Netz-ID bzw. VNI (Virtualized Network Identifier) versehen und dann an den Zielhypervisor gesendet, der die Ziel-VSI hostet. Der Zielhypervisor entfernt die VNI und leitet das Datenpaket an die Ziel-VSI weiter.

Datenflüsse **zwischen verschiedenen Teilnetzen und innerhalb derselben Zone**: Hierbei werden Pakete verwendet, die die implizite Routerfunktion der VPC nutzen, die alle in der VPC erstellten Teilnetze miteinander verbindet. Sie leitet das Datenpaket an den richtigen Zielhypervisor weiter. Wenn der Zielhypervisor ein anderer als der Quellenhypervisor ist, wird das Datenpaket mit der ordnungsgemäßen virtualisierten Netz-ID bzw. VNI (Virtualized Network Identifier) versehen und dann an den Zielhypervisor gesendet. Dort wird die VNI entfernt und das Datenpaket an die Ziel-VSI weitergeleitet. (Diese letzten Schritte sind die gleichen wie im zuvor beschriebenen Typ von Datenfluss.)

Datenflüsse **zwischen verschiedenen Teilnetzen und in verschiedenen Zonen**: Bei solchen Datenflüssen entfernt die implizite Routerfunktion das Paket über das MPLS-VPN der VPCs zwecks Transit über den Cloud-Backbone. In der Zielzone versieht die implizite Routerfunktion das Datenpaket mit der entsprechenden VNI. Dann wird das Paket an den Zielhypervisor weitergeleitet, wo die VNI (wie zuvor beschrieben) wiederum entfernt wird, damit das Datenpaket an die Ziel-VSI weitergeleitet werden kann.

Datenflüsse an **Services jenseits der VPC**: Pakete, die für IaaS-Services oder CSE-Services (Services von IBM Cloud Service Endpoint) bestimmt sind, verwenden die implizite Routerfunktion der VPC sowie die Funktion der Netzadressumsetzung (NAT, Network Address Translation). Die Umsetzungsfunktion ersetzt die VSI-Adresse durch eine IPv4-Adresse, welche die VPC gegenüber dem angeforderten IaaS- oder CSE-Service identifiziert.

Datenflüsse an das **Internet jenseits der VPC**: Pakete, die für das Internet bestimmt sind, weisen den höchsten Komplexitätsgrad auf. Zusätzlich zu der Verwendung der impliziten Routerfunktion der VPC stützt sich jeder dieser Flüsse auch noch auf eine der beiden Netzadressumsetzungsfunktionen des impliziten Routers:

  * Eine Funktion für die explizite Eins-zu-viele-Netzadressumsetzung über ein öffentliches Gateway, das alle mit ihm verbundenen Teilnetze bedient.
  * Eine Eins-zu-eins-Netzadressumsetzung, die einzelnen VSIs zugewiesen ist.

Nach der NAT-Umsetzung leitet der implizite Router diese an das Internet bestimmten Pakete über den Cloud-Backbone an das Internet weiter.

## Klassischer Zugriff
{: #classic-access}

Das Feature des [**klassischen Zugriffs**](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc) für VPC wird durch die Wiederverwendung der ID für VRF (Virtual Routing and Forwarding) aus dem Konto der klassischen Infrastruktur von {{site.data.keyword.cloud}} als VRF-ID für die VPC erzielt. Diese Implementierung ermöglicht der impliziten Routerfunktion der VPC, sich demselben MPLS-VPN anzuschließen, das auch vom Konto der klassischen Infrastruktur verwendet wird. Daher hat die VPC Zugriff auf klassische Ressourcen sowie auf alles, was über bestehende Direct Link-Verbindungen erreicht werden kann.
