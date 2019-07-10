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


# Scelta degli intervalli di IP per il tuo VPC
{: #choosing-ip-ranges-for-your-vpc}

Utilizza un'annotazione CIDR come ad esempio:

* `<IPv4 address>/number` (esempio di indirizzo VPC: 10.10.0.0/16).

Vorrai riservare gli ultimi 16 bit (65.536 indirizzi) dell'IPv4 come degli 0 in modo da poterli utilizzare per vari indirizzi IP della sottorete all'interno dello stesso VPC {{site.data.keyword.cloud}} (esempio indirizzo IP sottorete: 10.10.1.0/24).

La notazione CIDR è definita in [RFC 1518](https://tools.ietf.org/html/rfc1518) e [RFC 1519](https://tools.ietf.org/html/rfc1519).
{: note}

Se utilizzi un intervallo IP al di fuori degli intervalli definiti da [RFC 1918](https://tools.ietf.org/html/rfc1918) (`10.0.0.0/8`, `172.16.0.0/12` o `192.168.0.0/16`) per una sottorete, le istanze collegate a tale sottorete potrebbero non essere in grado di raggiungere parti dell'Internet pubblico.

Ricorda, nel caso tu non abbia esperienza con l'annotazione CIDR, più piccolo è il numero dopo la barra e **più** indirizzi IP stai assegnando, perché il numero dopo la barra rappresenta il numero di bit iniziali 1 nella maschera del prefisso della sottorete.

La seguente tabella elenca il numero di indirizzi disponibili in una sottorete, in base alla dimensione del blocco CIDR specificata:

| Dimensione blocco CIDR | Indirizzi disponibili |
| --------------- | ------------------- |
|      /22        |        1019         |
|      /23        |         507         |
|      /24        |         251         |
|      /25        |         123         |
|      /26        |          59         |
|      /27        |          27         |
|      /28        |          11         |

Se hai bisogno di ulteriori informazioni, possono essere trovati online molti eccellenti articoli riguardanti CIDR (_Classless Inter-Domain Routing_).
