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

# Sobre a rede para a VPC
{: #about-networking-for-vpc}

Uma nuvem particular virtual (VPC) é uma rede virtual que está ligada à sua conta de cliente. Ela fornece segurança em nuvem com a capacidade de escalar dinamicamente, fornecendo controle de baixa granularidade sobre a infraestrutura virtual e sua segmentação de tráfego de rede.

Este documento abrange alguns conceitos de rede da forma como eles se aplicam ao {{site.data.keyword.cloud}} VPC. Os casos de uso e as características descritos incluem:

* regiões
* zonas
* endereços IP reservados
* gateways públicos
* interfaces vNIC
* sub-redes
* endereços IP flutuantes
* conexões VPN

## Visão geral
{: #subnets-overview}

Três opções estão disponíveis para acesso à Internet por meio de seu VPC:
* Usar um gateway público para o tráfego de Internet de toda a sub-rede;
* Usar um IP flutuante para o tráfego de Internet de uma instância e para ela;
* Use uma VPN para conectividade externa segura.

Lembre-se:
* Alguns intervalos de CIDR de endereço de sub-rede são reservados pela IBM.
* Deve-se criar sua VPC antes de criar sub-redes dentro dessa VPC.
* O suporte ao IPv6 não está disponível.

Opcionalmente, você pode criar um VPC de acesso clássico para se conectar à sua infraestrutura clássica do IBM Cloud.
{:note}

Conforme mostrado na figura a seguir:
* As sub-redes em seu VPC podem se conectar à Internet pública por meio de um Gateway público (PGW) opcional.
* É possível designar um endereço IP flutuante (FIP) a qualquer instância para acessá-la por meio da Internet ou vice-versa, independentemente de a sub-rede estar conectada a um gateway público.
* As sub-redes dentro do IBM Cloud VPC oferecem conectividade privada. Elas podem conversar umas com as outras por um link privado, por meio do roteador implícito. A configuração de rotas não é necessária.

![Conectividade e segurança do IBM VPC](images/vpc-connectivity-and-security.svg "Conectividade e segurança do IBM VPC")

## Terminologia
{: #network-terminology}

Esse [glossário](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary#vpc-glossary) contém definições e informações sobre termos usados nesse documento para o IBM Cloud VPC. Ao trabalhar com sua VPC, você precisará estar familiarizado com os conceitos básicos de _região_ e _zona_ da forma como eles se aplicam à sua implementação.

### Regiões
{: #subnet-regions}

Uma região é uma abstração relacionada à área geográfica na qual uma VPC é implementada. Cada região contém múltiplas zonas, que representam domínios de falha independentes. Uma VPC do IBM Cloud pode abranger múltiplas zonas dentro de sua região designada.

### Zonas
{: #subnet-zones}

Uma zona é uma abstração que se refere ao data center físico que hospeda os recursos de computação, de rede e de armazenamento, assim como o resfriamento e a energia relacionados, que fornece serviços e aplicativos. O isolamento de zonas melhora a tolerância a falhas geral do sistema, diminui a latência e evita a criação de um único ponto de falha compartilhado. Uma zona garante as propriedades a seguir:

 * Cada zona é um domínio de falha independente e é extremamente improvável que duas zonas em uma região falhem simultaneamente.
 * O tráfego entre zonas em uma região terá menos de 2 ms de latência.

## Características das sub-redes na VPC
{: #characteristics-of-subnets}

Uma sub-rede consiste em um intervalo de endereços IP especificado (bloco CIDR). As sub-redes são ligadas a uma única zona e não podem abranger múltiplas zonas ou regiões. No entanto, uma sub-rede pode abranger a totalidade das abstrações da zona dentro de sua nuvem particular virtual. As sub-redes no mesmo IBM Cloud VPC estão conectadas umas às outras.

### Endereços IP reservados
{: #reserved-ip-addresses}

Determinados endereços IP são reservados para uso pela IBM ao operar a nuvem particular virtual. Aqui estão os endereços reservados (esses endereços IP assumem que o intervalo de CIDR da sub-rede é 10.10.10.0/24):

  * Primeiro endereço no intervalo CIDR (10.10.10.0): endereço de rede
  * Segundo endereço no intervalo CIDR (10.10.10.1): endereço do gateway
  * Terceiro endereço no intervalo CIDR (10.10.10.2): reservado pela IBM
  * Quarto endereço no intervalo de CIDR (10.10.10.3): reservado pela IBM para uso futuro
  * Último endereço no intervalo CIDR (10.10.10.255): endereço de transmissão de rede

### Usar um gateway público para conectividade externa de uma sub-rede
{: #use-a-public-gateway}

Um **Gateway público (PGW)** permite que uma sub-rede (com todas as instâncias anexadas à sub-rede) se conecte à Internet. Observe que as sub-redes são privadas por padrão; no entanto, opcionalmente, é possível criar um PGW e conectar uma sub-rede a ele. Depois que uma sub-rede é conectada ao PGW, todas as instâncias nessa sub-rede podem se conectar à Internet.

O PGW usa a _NAT de muitos para 1_, o que significa que milhares de instâncias com endereços privados usarão um endereço IP público para comunicação com a Internet pública. O PGW não permite que a Internet inicie uma conexão com essas instâncias. Use a API para conectar e separar sub-redes para e do PGW.

A figura a seguir resume o escopo atual dos serviços de gateway.

| SNAT | DNAT | ACL | VPN |
| ---- | ---- | --- | --- |
| As instâncias podem ter acesso somente de saída à Internet | Permite conectividade de entrada da Internet a um IP privado | Fornece acesso de entrada restrito da Internet a instâncias ou sub-redes | A VPN site para site manipula clientes de qualquer tamanho e locais únicos ou múltiplos |
| As sub-redes inteiras compartilham o mesmo terminal público de saída | Fornece acesso limitado a um único servidor privado | Restringe a entrada de acesso da Internet com base em serviço, protocolo ou porta | O rendimento alto (até 10 Gbps) fornece aos clientes a capacidade de transferir arquivos de dados grandes de forma segura e rápida |
| Protege instâncias; não é possível iniciar o acesso a instâncias por meio do terminal público | O serviço DNAT pode ter a capacidade aumentada ou diminuída com base nos requisitos | As ACLs stateless permitem o controle granular do tráfego | Cria conexões seguras com criptografia padrão de mercado |

Um gateway público é criado em uma VPC, mas o gateway não faz nada até que seja anexado a uma sub-rede. É possível criar apenas um gateway público por zona, o que significa, por exemplo, que você poderia ter 3 (três) gateways públicos por VPC em um ambiente com três zonas.
{:note}

## Limitações de sub-redes
{: #limitations-of-subnets}

Para ver uma lista completa de limitações conhecidas e de recursos não suportados no momento, consulte o documento [Limitações conhecidas](/docs/vpc-on-classic?topic=vpc-on-classic-known-limitations).

### Restrições sobre a exclusão de uma sub-rede
{: #restrictions-on-deleting-a-subnet}

Não será possível excluir uma sub-rede se os recursos (como uma Instância de Servidor Virtual (VSI) ou IPs flutuantes) estiverem em uso nessa sub-rede. Os recursos deverão ser excluídos primeiro.

### Limitações na atualização de uma sub-rede existente
{: #limitations-on-updating-an-existing-subnet}

* Não é possível redimensionar uma sub-rede existente. Por exemplo, 10.10.16.0/24 não pode ser redimensionado para 10.10.16.0/20.
* Não é possível mover uma sub-rede existente. Por exemplo, 10.10.10.0/24 não pode ser movido para 10.10.11.0/24.

## Conectividade externa
{: #external-connectivity}

A conectividade externa pode ser alcançada por meio de um endereço IP flutuante anexado a uma instância, por um gateway externo conectado a uma sub-rede ou por um túnel VPN.

### Usar um endereço IP flutuante para conectividade externa de uma instância
{: #use-floating-ip}

**Endereços IP flutuantes** são endereços IP fornecidos pelo sistema e podem ser acessados por meio da Internet pública.

É possível reservar um endereço IP flutuante do conjunto de endereços IP flutuantes disponíveis fornecidos pela IBM e é possível associá-lo ou desassociá-lo de qualquer instância na mesma nuvem particular virtual por meio do vNIC dessa instância. Qualquer endereço IP flutuante pode ser associado a vários tipos de instâncias de servidor virtual (VSIs), por exemplo, um balanceador de carga ou um gateway VPN.

O endereço IP flutuante não pode ser associado a múltiplas interfaces. Deve-se especificar a interface na VSI que será associada a esse IP flutuante individual. Essa interface também terá um endereço IP privado. O sistema back-end executa operações _1-to-1 NAT_ entre o IP flutuante e o IP privado dessa interface. Um IP flutuante é liberado apenas quando você solicita especificamente a ação de liberação. 

**Notas:**
* **A associação de um endereço IP flutuante com uma VSI remove a VSI da NAT muitos para um do PGW mencionado anteriormente. **
* **Atualmente, o IP flutuante suporta apenas endereços IPv4.**
* **Não é possível trazer seu próprio endereço IP público para ser usado como um IP flutuante.**

Para obter mais informações sobre as operações NAT, consulte [o documento RFC relacionado à Internet ![Ícone de link externo](../../icons/launch-glyph.svg "Ícone de link externo")](http://www.faqs.org/rfcs/rfc1631.html){: new_window}.

### Usar o VPN para conectividade externa segura
{: #use-vpn}

O serviço Virtual Private Network (VPN) está disponível para que os usuários se conectem com segurança ao IBM Cloud VPC por meio da Internet.

**Recursos do VPN**
  * Capacidade de CRUD (criar, ler, atualizar e excluir) o serviço VPN (site IPSEC de site para site) para manipulação de transferência de dados.
  * Capacidade de associar o serviço VPN a um IBM Cloud VPC do cliente ou rede virtual.
  * Capacidade de identificar sub-redes no site do cliente, seja pela definição estática ou pelo roteamento dinâmico.
  * Suporte para cifras seguras, como SHA256, AES, 3DES, IKEv2.
  * Confiabilidade de serviço integrada.
  * Monitoramento contínuo do funcionamento da conexão VPN.
  * Suporte para os modos do inicializador e do respondente; isto é, o tráfego pode ser iniciado por qualquer lado do túnel.

## Aprenda mais
{: #subnets-learn-more}
   * [Segurança do IBM VPC](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-security-in-your-ibm-cloud-vpc)
   * [Endereços, regiões e sub-redes do IBM VPC](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-working-with-ip-address-ranges-address-prefixes-regions-and-subnets)
