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

# Sobre a rede para a VPC
{: #about-networking-for-vpc}

Nesse documento, você encontrará conceitos que descrevem casos de uso e recursos no {{site.data.keyword.cloud}} VPC para sub-redes, endereços IP flutuantes e conexões VPN.

![Conectividade e segurança do IBM VPC](images/vpc-connectivity-and-security.svg "Conectividade e segurança do IBM VPC")

Conforme mostrado na figura:

* As sub-redes podem se conectar à Internet pública por meio de um gateway público (PGW) opcional.
* É possível designar um endereço IP flutuante (FIP) a qualquer VSI para alcançá-lo por meio da Internet ou vice-versa.
* As sub-redes no IBM Cloud VPC oferecem conectividade privada; elas podem conversar umas com as outras por meio de um link privado. Não é necessário configurar nenhuma rota.
* Para obter mais informações, veja [Sobre a infraestrutura de VPC](/docs/vpc-on-classic?topic=vpc-on-classic-about).

## Terminologia
{: #terminology}

Esse [glossário](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary) contém definições e informações sobre termos usados nesse documento para o IBM Cloud VPC.

## Características das sub-redes na VPC
{: #characteristics-of-subnets-in-the vpc}

Uma sub-rede consiste em um intervalo de endereços IP especificado (bloco CIDR). As sub-redes são ligadas a uma única zona e não podem abranger múltiplas zonas ou regiões. No entanto, uma sub-rede pode abranger a totalidade das abstrações de Zona dentro de sua Nuvem Particular Virtual. As sub-redes no mesmo IBM Cloud VPC estão conectadas umas às outras.

### Zonas
{: #zones}

Uma Zona é uma abstração projetada para ajudar com a tolerância a falhas melhorada e a latência diminuída. Uma zona garante as propriedades a seguir:

 * Cada zona é um domínio de falha independente e é extremamente improvável que duas zonas em uma região falhem simultaneamente
 * O tráfego entre zonas em uma região será latência de < 2 ms

### Endereços IP reservados
{: #reserved-ip-addresses}

Determinados endereços IP são reservados para uso pela IBM ao operar a nuvem particular virtual. Aqui estão os endereços reservados (os endereços IP fornecidos presumem que o intervalo de CIDR da sub-rede é 10.10.10.0/24):

  * Primeiro endereço no intervalo CIDR (10.10.10.0): endereço de rede
  * Segundo endereço no intervalo CIDR (10.10.10.1): endereço do gateway
  * Terceiro endereço no intervalo CIDR (10.10.10.2): reservado pela IBM
  * Quarto endereço no intervalo de CIDR (10.10.10.3): reservado pela IBM para uso futuro
  * Último endereço no intervalo CIDR (10.10.10.255): endereço de transmissão de rede

### Usar um gateway público para conectividade externa de uma sub-rede
{: #use-a-public-gateway-for-external-connectivity-of-a-subnet}

Um **Gateway público (PGW)** permite que uma sub-rede (com todas as VSIs conectadas à sub-rede) se conecte à Internet. Observe que as sub-redes são privadas por padrão; no entanto, opcionalmente, é possível criar um PGW e conectar uma sub-rede a ele. Depois que uma sub-rede é conectada ao PGW, todas as VSIs nessa sub-rede podem se conectar à Internet.

O PGW usa a _NAT muitos para um_, o que significa que milhares de VSIs com endereços particulares usarão um endereço IP público para conversar com a Internet pública. O PGW não permite que a Internet inicie uma conexão com essas instâncias. Use a API para conectar e separar sub-redes para e do PGW.

A figura a seguir resume o escopo de serviços de gateway.

![serviços de gateway](images/scope-of-gateway-services.png)

### Usar um endereço IP flutuante para conectividade externa de uma VSI 
{: #use-a-floating-ip-address-for-external-connectivity-of-a-vsi}

**Endereços IP flutuantes** são endereços IP fornecidos pelo sistema e podem ser acessados por meio da Internet pública.

É possível reservar um endereço IP flutuante do conjunto de endereços IP flutuantes disponíveis fornecidos pela IBM e é possível associá-lo ou desassociá-lo de qualquer instância na mesma nuvem particular virtual por meio do vNIC dessa instância. Qualquer endereço IP flutuante pode ser associado a vários tipos de instâncias de servidor virtual (VSIs), por exemplo, um balanceador de carga ou um gateway VPN.

O endereço IP flutuante não pode ser associado a múltiplas interfaces. Deve-se especificar a interface na VSI que será associada a esse IP flutuante individual. Essa interface também terá um endereço IP privado. O sistema back-end executa operações _NAT 1 para 1_ entre o IP flutuante e o IP privado dessa interface. Um IP flutuante é liberado apenas quando você solicita especificamente a ação de liberação. 

**Notas:**
* **A associação de um endereço IP flutuante com uma VSI remove a VSI da NAT muitos para um do PGW mencionado anteriormente. **
* **Atualmente, o IP flutuante suporta apenas endereços IPv4.**
* **Não é possível trazer seu próprio endereço IP público para ser usado como um IP flutuante.**

Para obter mais informações sobre as operações NAT, consulte [o documento RFC relacionado à Internet ![Ícone de link externo](../../icons/launch-glyph.svg "Ícone de link externo")](http://www.faqs.org/rfcs/rfc1631.html){: new_window}.

### Usar uma VPN para conectividade externa segura
{: #use-a-vpn-for-secure-external-connectivity}

O serviço Virtual Private Network (VPN) está disponível para que os usuários se conectem com segurança ao IBM Cloud VPC por meio da Internet. Para obter instruções passo a passo, consulte nosso [Guia da IU do IBM Console](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console).

**Recursos do VPN**
  * Capacidade de CRUD (criar, ler, atualizar e excluir) o serviço VPN (site IPSEC de site para site) para manipulação de transferência de dados.
  * Capacidade de associar o serviço VPN a um IBM Cloud VPC do cliente ou rede virtual.
  * Capacidade de identificar sub-redes no site do cliente, seja pela definição estática ou pelo roteamento dinâmico.
  * Suporte para cifras seguras, tais como SHA256, AES, 3DES, IKEv2
  * Confiabilidade de serviço integrada.
  * Monitoramento de funcionamento da conexão VPN.
  * Suporte para os modos do inicializador e do respondente; isto é, o tráfego pode ser iniciado por qualquer lado do túnel.
