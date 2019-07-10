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

# Entendendo intervalos de endereços IP, prefixos de endereço, regiões e sub-redes
{: #working-with-ip-address-ranges-address-prefixes-regions-and-subnets}
[comment]: # (tópico da ajuda vinculado)

Este documento discute os relacionamentos entre regiões, prefixos de endereço e sub-redes para o {{site.data.keyword.cloud}} VPC:

* Os VPCs são implementados e ligados a uma região.
* Dentro dessa região, os VPCs podem abranger várias zonas.
* Os prefixos de endereço permitem a comunicação entre as zonas que um VPC abrange. Cada VPC obtém um prefixo de endereço padrão de atalho para cada zona que ele abrange.
* As sub-redes são criadas dentro do escopo de um prefixo de endereço, o que significa que uma sub-rede deve estar completamente contida dentro de um único prefixo de endereço existente.
* Se você usar um intervalo de IPs fora dos intervalos definidos pelo [RFC 1918](https://tools.ietf.org/html/rfc1918) (`10.0.0.0/8`, `172.16.0.0/12` ou `192.168.0.0/16`) para uma sub-rede, as instâncias anexadas a essa sub-rede poderão ser incapazes de acessar partes da Internet pública.

## VPC do IBM Cloud e regiões
{: #ibm-cloud-vpc-and-regions}

Um {{site.data.keyword.cloud}} VPC é implementado em uma região, mas pode abranger múltiplas zonas. Cada região contém múltiplas zonas, que representam domínios de falha independentes.

## IBM Cloud VPC e Prefixos de endereço
{: #ibm-cloud-vpc-and-address-prefixes}

Os prefixos de endereço ativam a comunicação entre as instâncias do {{site.data.keyword.cloud_notm}} VPC em zonas diferentes. Eles fornecem informações de roteamento, as quais o _roteador implícito_ precisa, para enviar dados para a zona na qual a instância de destino está localizada. Cada sub-rede deve ser contida por um prefixo de endereço. O {{site.data.keyword.cloud_notm}} VPC não suporta o conceito de sub-redes "locais" ou "inacessíveis".

O _Roteador implícito_ é a conectividade de rede inerente entre todas as sub-redes criadas dentro de um VPC.
{: note}

Cada {{site.data.keyword.cloud_notm}} VPC pode ter até cinco prefixos de endereço para cada zona. Para ajudá-lo a começar, o {{site.data.keyword.cloud_notm}} VPC define um prefixo de endereço padrão para cada zona (consulte a tabela a seguir), no entanto, como uma boa prática, é necessário projetar um plano de endereçamento de VPC antes de implementar um.

### Prefixos de endereço padrão do VPC
{: #default-vpc-address-prefixes}

Quando um novo VPC é criado, os prefixos de endereço padrão são designados da forma a seguir, com base na região e na zona.

Os [VPCs de acesso clássico](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#classic-access-default-address-prefixes) têm um conjunto diferente de prefixos de endereço padrão.
{: important}

Zona         | Prefixo do endereço
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

Diferentes prefixos padrão serão designados para novas zonas ou regiões.

### Prefixos de endereço e a IU do console do IBM Cloud
{: #address-prefixes-and-the-ibm-cloud-console-ui}

Quando você cria uma VPC usando a IU do console do IBM Cloud, o sistema seleciona seu prefixo de endereço automaticamente e requer que você crie uma sub-rede nesse prefixo padrão. Se esse esquema de endereço não se adequar aos seus requisitos, será possível customizar os prefixos de endereço depois de criar a VPC. Então, será possível criar sub-redes em seus prefixos de endereço customizados e excluir a sub-rede criada com o prefixo padrão.

Essa solução alternativa é necessária para usar BYOIP por meio da IU do IBM Cloud Console.
{:note}

## VPC do IBM Cloud e sub-redes
{: #ibm-cloud-vpc-and-subnets}

É possível dividir um {{site.data.keyword.cloud_notm}} VPC em sub-redes. Todas as sub-redes em um {{site.data.keyword.cloud_notm}} VPC podem alcançar uma a outra, por meio do roteamento L3 privado por um roteador implícito. Não é necessário configurar nenhum roteador ou rota.

Fatos úteis sobre as sub-redes na VPC:

* Uma sub-rede consiste em um intervalo de endereços IP que você especifica.
* Uma sub-rede está ligada a uma única zona, então ela não pode abranger múltiplas zonas ou regiões.
* Uma sub-rede pode abranger a totalidade da zona em um {{site.data.keyword.cloud_notm}} VPC.
* Deve-se criar sua VPC antes de criar suas sub-redes dentro dessa VPC.
* O suporte ao IPv6 não está disponível.
* É possível associar ou desassociar uma VSI a uma sub-rede. (Isso requer que você inclua uma vNIC e selecione uma largura da banda.)
* Cada sub-rede deve estar contida dentro de um prefixo de endereço que pertença à zona à qual essa sub-rede está ligada.

É possível trazer seu próprio intervalo de endereço IPv4 público (BYOIP) para sua conta do {{site.data.keyword.cloud_notm}} VPC. Ao usar BYOIP, o {{site.data.keyword.cloud_notm}} deve configurar esses endereços IPv4 em recursos {{site.data.keyword.cloud_notm}}, que enviarão pacotes para e dos endereços fornecidos. Portanto, como resultado do uso de seu intervalo IPv4 fornecido no {{site.data.keyword.cloud_notm}}, esses endereços IP podem ser expostos à equipe de suporte da IBM e a terceiros como parte de seu uso desse serviço.
{:important}

![Visão geral do IBM Cloud VPC](images/vpc-experience.svg "Visão geral do IBM Cloud VPC")

### Usando prefixos de endereço para sub-redes
{: #using-address-prefixes-for-subnets}

Cada sub-rede deve existir dentro de um prefixo de endereço.
 * Para a sua nova sub-rede, é possível selecionar seu intervalo de endereços IP por meio dos prefixos de endereço existentes.
 * Se os prefixos de endereço da zona não forem adequados, um dos prefixos de endereço existentes poderá ser editado ou um novo prefixo de endereço poderá ser incluído para a zona (usando a API, a CLI ou a IU).

### Endereços IP disponíveis
{: #available-ip-addresses}

Estes são endereços IP disponíveis, conforme definido em **RFC 1918**:

 * 10.0.0.0 a 10.255.255.255
 * 172.16.0.0 a 172.31.255.255
 * 192.168.0.0 a 192.168.255.255

Se você usar um intervalo de IPs que está fora dos intervalos permitidos para uma sub-rede (fornecidos em seções anteriores), as instâncias anexadas a essa sub-rede poderão ser incapazes de acessar partes da Internet pública.

### Mais sobre a criação de uma sub-rede
{: #more-about-creating-a-subnet}

É possível especificar uma sub-rede para seu VPC de duas maneiras:
  * É possível criar uma sub-rede fornecendo o tamanho da sub-rede que você precisa, como o número de endereços suportados (por exemplo, 1024)
  * É possível criar uma sub-rede, fornecendo um intervalo de CIDR (como 10.0.0.8/29)

Como um exemplo de como especificar um bloco de 1024 usando CIDR, se você estiver especificando um intervalo de CIDR em vez de um tamanho de sub-rede, o bloco IPv4 `192.168.100.0/22` representará os 1024 endereços IPv4 de `192.168.100.0` a `192.168.103.255`.
{:tip}

