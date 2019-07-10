---

copyright:
  years: 2019

lastupdated: "2019-05-20"

keywords: security groups, traffic, firewall, stateful, filtering

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
{:DomainName: data-hd-keyref="DomainName"}

# Usando grupos de segurança
{: #using-security-groups}
[comment]: # (tópico da ajuda vinculado)

Os grupos de segurança fornecem uma maneira conveniente de aplicar regras que estabelecem filtragem para cada interface de rede de uma instância de servidor virtual (VSI), com base no endereço IP. Ao criar um novo recurso do grupo de segurança, você o atualizará para criar os padrões de tráfego de rede desejados.

Para obter uma comparação das características dos grupos de segurança e de ACLs, consulte a [tabela de comparação](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-compare-security-groups-and-access-control-lists).

Por padrão, um grupo de segurança nega todo o tráfego. Como as regras são incluídas em um grupo de segurança, ele define o tráfego que o grupo de segurança permite.

As regras são _stateful_, o que significa que o tráfego reverso em resposta ao tráfego permitido é automaticamente permitido. Portanto, por exemplo, uma regra que permite o tráfego TCP de entrada na porta 80 também permite responder ao tráfego TCP de saída na porta 80 de volta ao host de origem, sem a necessidade de uma regra adicional.

Os grupos de segurança têm o escopo definido para uma única VPC. Essa definição de escopo implica que um grupo de segurança pode ser anexado _somente_ às interfaces de rede de VSIs dentro do mesmo VPC.

Quando uma VSI é criada sem nenhum grupo de segurança especificado, a interface de rede primária da VSI é colocada no grupo de segurança _padrão_ da VPC da VSI. Essa liberação do {{site.data.keyword.cloud}} VPC definiu um grupo de segurança padrão que permite determinado tráfego. Para obter mais informações, veja [Atualizando o grupo de segurança padrão](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-updating-the-default-security-group).

É possível configurar grupos de segurança usando a API de REST, a CLI ou a IU:

* [Configurando grupos de segurança usando a API](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-apis)
* [Configurando grupos de segurança usando a CLI](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-cli)
* [Configurando grupos de segurança usando a IU](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#configuring-the-security-group-for-the-instance)
