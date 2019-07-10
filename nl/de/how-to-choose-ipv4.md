---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: IPv4, ranges, subnets, CIDR, 1918

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


# IP-Bereiche für Ihre VPC auswählen
{: #choosing-ip-ranges-for-your-vpc}

Verwenden Sie die CIDR-Notation wie folgt:

* `<IPv4 address>/number` (Beispiel der VPC-Adresse: 10.10.0.0/16).

Sie sollten die letzten 16 Bit (65.536 Adressen) der IPv4-IP als Nullwerte reservieren, damit Sie sie für unterschiedliche Teilnetz-IP-Adressen in derselben {{site.data.keyword.cloud}}-VPC verwenden können (Beispiel einer Teilnetz-ID: 10.10.1.0/24).

Die CIDR-Notation ist in [RFC 1518](https://tools.ietf.org/html/rfc1518) und [RFC 1519](https://tools.ietf.org/html/rfc1519) definiert.
{: note}

Wenn Sie einen IP-Bereich außerhalb der Bereiche verwenden, die durch [RFC 1918](https://tools.ietf.org/html/rfc1918) (`10.0.0.0/8`, `172.16.0.0/12` oder `192.168.0.0/16`) für ein Teilnetz definiert sind, können die an diesem Teilnetz angeschlossenen Instanzen keine Teile des öffentlichen Internets erreichen.

Für den Fall, dass Sie mit der CIDR-Notation noch nicht vertraut sind, sollten Sie Folgendes berücksichtigen: Je kleiner die Zahl nach dem Schrägstrich ist, umso **mehr** IP-Adressen werden zugewiesen, da die Zahl nach dem Schrägstrich die Anzahl der führenden 1-Bits in der Präfixmaske des Subnetzes darstellt.

Die folgende Tabelle listet die Anzahl der verfügbaren Adressen in einem Teilnetz auf, basierend auf der angegebenen CIDR-Blockgröße:

| CIDR-Blockgröße | Verfügbare Adressen |
| --------------- | ------------------- |
|      /22        |        1019         |
|      /23        |         507         |
|      /24        |         251         |
|      /25        |         123         |
|      /26        |          59         |
|      /27        |          27         |
|      /28        |          11         |

Falls Sie weitere Informationen benötigen, können Sie online eine Reihe hervorragender Artikel zum Thema _Classless Inter-Domain Routing (CIDR)_ finden.
