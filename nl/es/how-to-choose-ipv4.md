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


# Selección de rangos de IP para la VPC
{: #choosing-ip-ranges-for-your-vpc}

Utilice la notación CIDR, como por ejemplo:

* `<IPv4 address>/number` (ejemplo de dirección de VPC: 10.10.0.0/16).

Es posible que desee reservar los últimos 16 bits (65.536 direcciones) del IPv4 como 0 para poder utilizarlos en varias direcciones IP de subred del mismo VPC de {{site.data.keyword.cloud}} (ejemplo de dirección IP de subred: 10.10.1.0/24).

La notación CIDR se define en [RFC 1518](https://tools.ietf.org/html/rfc1518) y [RFC 1519](https://tools.ietf.org/html/rfc1519).
{: note}

Si utiliza un rango de IP fuera de los rangos definidos en [RFC 1918](https://tools.ietf.org/html/rfc1918) (`10.0.0.0/8`, `172.16.0.0/12` o en `192.168.0.0/16`) para una subred, es posible que las instancias conectadas a esa subred no puedan acceder a partes de la Internet pública.

Recuerde que, si es nuevo en lo que respecta a la notación CIDR, cuanto menor sea el número situado tras la barra inclinada, **más** direcciones IP asignará, puesto que dicho número representa el número de 1 bits iniciales en la máscara de prefijo de la subred.

En la tabla siguiente se muestra el número de direcciones disponibles en una subred, basado en el tamaño de bloque CIDR especificado:

| Tamaño de bloque CIDR | Direcciones disponibles |
| --------------- | ------------------- |
|      /22        |        1019         |
|      /23        |         507         |
|      /24        |         251         |
|      /25        |         123         |
|      /26        |          59         |
|      /27        |          27         |
|      /28        |          11         |

Si necesita más información, encontrará artículos en línea relacionados con el _Direccionamiento entre dominios sin clases_ (CIDR).
