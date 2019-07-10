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

# VPC nos bastidores
{: #vpc-behind-the-curtain}

Esta página apresenta uma figura conceitual mais detalhada do que está acontecendo "nos bastidores" em relação à rede VPC. Espera-se que os leitores tenham alguma experiência de rede.

## Isolamento de rede
{: #network-isolation}

O isolamento de rede VPC ocorre em três níveis:

* **Hypervisor**: as VSIs (instâncias do servidor virtual) são isoladas pelo próprio hypervisor. Uma VSI não poderá atingir diretamente outras VSIs hospedadas pelo mesmo hypervisor se elas não estiverem na mesma VPC.

* **Rede**: o isolamento ocorre no nível de rede por meio do uso de **virtual network identifiers** (VNIs). Esses identificadores têm o escopo definido para a zona local. Esses VNIs são incluídos em todos os pacotes de dados que entram em qualquer zona do VPC: que entram por meio do hypervisor, quando enviados por uma VSI, ou que entram na zona por meio da nuvem, quando enviados pela função de roteamento implícito.

Um pacote que sai de uma zona tem o VNI removido. Quando o pacote atinge sua zona de destino, entrando por meio da função de roteamento implícito, o roteador implícito sempre inclui o VNI adequado para essa zona.
{: note}

* **Roteador**: a _função de roteador implícito_ fornece isolamento para cada VPC, fornecendo uma **virtual routing function** (VRF) e uma VPN com MPLS (multi-protocol label switching) no backbone de nuvem. A VRF de cada VPC tem um identificador exclusivo e esse isolamento permite que cada VPC tenha acesso à sua própria cópia do espaço de endereço IPv4. A VPN MPLS permite federar todas as bordas da nuvem: Infraestrutura clássica, Link direto e VPC.

## Prefixos de endereço
{: #address-prefixes}

Os prefixos de endereço são as informações de resumo usadas pela função de roteamento implícito de uma VPC para localizar uma _VSI de destino_, independentemente da zona de disponibilidade na qual a VSI de destino está localizada. A função primária dos prefixos de endereço é otimizar o roteamento sobre a VPN MPLS, ao mesmo tempo em que evita casos de roteamento patológico. Todas as sub-redes criadas em uma VPC devem estar contidas em um prefixo de endereço, de modo que todas as VSIs em uma VPC sejam atingíveis por meio de todas as outras VSIs na VPC.

## Fluxos de pacote de dados e o roteador implícito
{: #data-packet-flows-and-the-implicit-router}

Seis tipos diferentes de fluxos de pacote de dados de VSI ocorrem em uma VPC. Em ordem crescente de complexidade, esses fluxos são:

* Intrassub-rede, intra-host (mesmo hypervisor)
* Intrassub-rede, inter-host
* Intersub-rede, intrazona
* Intersub-rede, interzona
* Serviço Extra-VPC (para acesso IaaS ou CSE)
* Internet Extra-VPC (para acesso à Internet)

Fluxos de dados **Intrassub-rede, intra-host**: esses são os mais simples. Os pacotes fluem entre as VSIs no hypervisor e nenhum pacote está saindo do hypervisor.

Fluxos de dados **Intrassub-rede, inter-host**: esses fluxos envolvem pacotes que saem do hypervisor. Cada pacote é identificado com o VNI adequado (identificador de rede virtualizado) para assegurar o isolamento dos dados e, em seguida, é enviado para o hypervisor de destino, que hospeda a VSI de destino. O hypervisor de destino remove o VNI e encaminha o pacote de dados para a VSI de destino.

Fluxos de dados **Intersub-rede, intrazona**: esses fluxos envolvem pacotes que usam a função de roteador implícito do VPC, que conecta todas as sub-redes criadas no VPC. Ela roteia o pacote de dados para o hypervisor de destino correto. Se o hypervisor de destino for diferente do hypervisor de origem, o pacote de dados será identificado com o VNI adequado e será enviado para o hypervisor de destino. Lá, o VNI é removido e o pacote de dados é encaminhado para a VSI de destino. (Essas últimas etapas são as mesmas que aqueles descritas no tipo de fluxo de dados anterior.)

Fluxo de dados **Intersub-rede, interzonas**: para esses fluxos, a função de roteador implícito remove o VNI e encaminha o pacote na VPN MPLS do VPC para trânsito através do backbone de nuvem. Na zona de destino, a função de roteador implícito identifica o pacote de dados com o VNI apropriado. Em seguida, o pacote é encaminhado para o hypervisor de destino, em que a VNI está (conforme descrito anteriormente) removida novamente para que o pacote de dados possa ser encaminhado para a VSI de destino.

Fluxos de dados do **Serviço Extra-vpc**: os pacotes destinados aos serviços IaaS ou IBM Cloud Service Endpoint (CSE) utilizam a função de roteador implícito do VPC e também utilizam uma função de conversão de endereço de rede (NAT). A função de conversão substitui o endereço de VSI por um endereço IPv4, que identifica a VPC para o serviço IaaS ou CSE que está sendo solicitado.

Fluxos de dados **Internet de VPC extra**: os pacotes destinados à Internet são os mais complexos. Além de utilizar a função de roteador implícito da VPC, cada um desses fluxos também depende de uma das duas funções de conversão de endereço de rede (NAT) do roteador implícito:

  * um NAT explícito de um-para-muitos por meio de uma função de gateway público que atende a todas as sub-redes conectadas a ele.
  * um NAT de um-para-um designado a VSIs individuais.

Após a conversão NAT, o roteador implícito encaminha esses pacotes com destino à Internet para a Internet, usando o backbone de nuvem.

## Acesso clássico
{: #classic-access}

O recurso [**Acesso clássico**](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc) para VPC é realizado reutilizando o identificador VRF da Conta de infraestrutura clássica do {{site.data.keyword.cloud}} como o identificador VRF para VPC. Essa implementação permite que a função de roteador implícito da VPC se associe à mesma VPN MPLS que é usada pela Conta de infraestrutura clássica. Portanto, a VPC tem acesso a recursos clássicos e a qualquer outra coisa que for atingível por meio de conexões existentes de Link direto.
