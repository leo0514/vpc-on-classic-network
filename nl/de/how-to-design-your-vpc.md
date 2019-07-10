---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: VPC, subnet, address prefixes, design, addressing

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Adressierungsplan für eine VPC entwerfen 
{: #vpc-addressing-plan-design}

Der erste Schritt bei der Gestaltung Ihrer VPC sollte der Entwurf Ihres Adressierungsplans sein. Ein ordnungsgemäß ausgeführter Adressierungsplan hat zwei Ziele:

* Er erfüllt die Kommunikationsanforderungen von VPC-Instanzen.
* Er ist für zukünftiges Wachstum flexibel. 

Dieses Dokument enthält ein Beispiel für die Gestaltung des Adressierungsplans für eine dreistufige Webanwendung, in der jede Schicht von mehreren Zonen unterstützt wird.

Obwohl jede {{site.data.keyword.cloud}}-VPC für eine bestimmte Region bereitgestellt wird, kann eine VPC alle Zonen in dieser Region umfassen. Die {{site.data.keyword.cloud_notm}}-VPC definiert ein Standardadressenpräfix für jede Zone. Diese Adresspräfixe ermöglichen die Kommunikation zwischen {{site.data.keyword.cloud_notm}}-VPC-Instanzen in verschiedenen Zonen, indem die zonenbasierte Routing-Informationen bereitgestellt werden, die der [implizite Router](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary#implicit-router) benötigt.

Es sind dieselben Konstruktionsschritte erforderlich, unabhängig davon, ob die Anwendung vollständig in der Cloud enthalten ist oder ob Teile der Anwendung an einem anderen Standort ausgeführt werden.
{: tip}

## Überlegungen zu Entwurf und Annahmen
{: #design-considerations-and-assumptions}

Bei der Konzeption des Adressierungsplans für eine Anwendung ist die primäre Überlegung, die CIDR-Blöcke für die Erstellung von Teilnetzen innerhalb einer einzelnen Zone so zusammenhängend wie möglich zu gestalten. Dadurch lassen sie sich in einem einzigen Adresspräfix zusammenfassen und lässt Raum für künftiges Wachstum.

Eine weitere Überlegung ist die Anzahl der verfügbaren Adressen, die ein Teilnetz für die horizontale Skalierung benötigt. Die folgende Tabelle listet die Anzahl der verfügbaren Adressen in einem Teilnetz auf, basierend auf der angegebenen CIDR-Blockgröße:

| CIDR-Blockgröße | Verfügbare Adressen |
| --------------- | ------------------- |
|      /22        |        1019         |
|      /23        |         507         |
|      /24        |         251         |
|      /25        |         123         |
|      /26        |          59         |
|      /27        |          27         |
|      /28        |          11         |

Basierend auf diesen beiden Überlegungen, folgen hier die Annahmen, die für dieses Beispiel gemacht werden:

* Für dieses Beispiel werden CIDR-Bereiche aus dem 172.16.0.0/12-Block von RFC 1918-Adressen für alle Teilnetze verwendet.
* Wir gehen davon aus, dass die Darstellungsschicht der Anwendung eine dünne Schicht über einer REST-API ist. Daher wirkt sich die horizontale Skalierung auf die mittlere Schicht stärker aus als auf die Darstellungsschicht.

## Teilnetzgröße jeder Schicht bestimmen
{: #determine-each-tier-s-subnet-size}

Der nächste Schritt besteht darin, die Teilnetzgröße der einzelnen Schichten (in Bezug auf die verfügbaren Adressen) zu bestimmen. Jede Schicht der Anwendung verfügt über eine Präsenz in jeder Zone, so dass jede Zone drei Teilnetze benötigt. 

Im Folgenden finden Sie die Überlegungen, die wir bei der Planung der Teilnetzgröße der einzelnen Schichten verwenden:

* Die Datenbankschicht (das Back-End) ist die am wenigsten wahrscheinliche Schicht, die eine dynamische Skalierung erfordert, sodass diese Teilnetze die kleinsten sein können. Das heißt, diese Teilnetze können die geringste Anzahl an verfügbaren Adressen enthalten. 
    * _In diesem Beispiel wird ein CIDR-Block `/27` verwendet, der 27 Adressen in dieser Schicht zulässt._
* Die Mittelschicht ist die wahrscheinlichste, die eine dynamische Skalierung erfordert, so dass diese Teilnetze die größten sind. Das heißt, sie müssen die größte Anzahl an verfügbaren Adressen enthalten. 
    * _In diesem Beispiel wird ein CIDR-Block `/25` verwendet, der 123 Adressen in dieser Schicht zulässt._
* Die Front-End-Schicht passt dazwischen. Es werden nicht so viele Adressen benötigt wie für die Mittelschicht, aber es werden mehr Adressen benötigt als in der Datenbankschicht. 
    * _In diesem Beispiel wird ein CIDR-Block `/26` verwendet, der 59 Adressen in dieser Schicht zulässt._

## Teilnetze kombinieren und Adresspräfixe auswählen
{: #combining-the-subnets-and-selecting-the-address-prefixes}

Um ein akzeptables Adresspräfix für jede Zone auszuwählen, benötigen Sie eine Teilnetzgröße, die alle drei Teilnetze in jeder Schicht unterbringen kann und dennoch Raum für horizontale Skalierung und zukünftige Erweiterung lässt. 

Das Adressenpräfix `/24` ist das kleinste Präfix, in das diese drei Teilnetze kombiniert werden können (27 + 123 + 59). Als bewährtes Verfahren wird empfohlen, die nächsthöhere Teilnetzgröße zu wählen und nicht die kleinste. Die Zuordnung der nächsthöheren Teilnetzgröße (`/23`) ermöglicht eine horizontale Skalierung über die zuvor angegebenen Grenzwerte hinaus, da sie jeder Schicht neue Teilnetze hinzufügen kann, und zwar aus dem gleichen Adresspräfix.

Nachdem die korrekte Teilnetzgröße festgelegt wurde, können die eigentlichen Adresspräfixe zugeordnet werden, jeweils eine für jede Zone:

|  Zone  | Adresspräfix  |
| ------ | --------------- |
| Zone 1 | `172.16.0.0/23` |
| Zone 2 | `172.16.2.0/23` |
| Zone 3 | `172.16.4.0/23` |

Und von dieser Basis aus können die 3 Teilnetze innerhalb jeder Zone zugeordnet werden:

|  Zone  | Kategorie  |    Teilnetz-CIDR    |
| ------ | -------- | ----------------- |
| Zone 1 |  Mittel  |  `172.16.0.0/25`  |
| Zone 1 |  Front   |  `172.16.1.0/26`  |
| Zone 1 | Datenbank | `172.16.1.128/27` |
| Zone 2 |  Mittel  |  `172.16.2.0/25`  |
| Zone 2 |  Front   |  `172.16.3.0/26`  |
| Zone 2 | Datenbank | `172.16.3.128/27` |
| Zone 3 |  Mittel  |  `172.16.4.0/25`  |
| Zone 3 |  Front   |  `172.16.5.0/26`  |
| Zone 3 | Datenbank | `172.16.5.128/27` |

## Hinweise zur Erweiterung einer vorhandenen Infrastruktur
{: #considerations-for-extending-an-existing-infrastructure}

Wenn Sie eine VPC planen, die eine vorhandene Infrastruktur erweitert, können Sie die vorherigen Schritte ausführen, unabhängig davon, ob es sich bei dieser Infrastruktur um Ihre On-Premises-Infrastruktur, um eine andere VPC oder um eine andere Cloud handelt. Denken Sie daran, dass Sie vorhandene Adressbereiche nicht wiederverwenden dürfen. Dadurch, dass Sie vermeiden, Adressen wiederzuverwenden, können Sie den Nutzen der {{site.data.keyword.cloud_notm}}-VPC-Funktionen maximieren.
