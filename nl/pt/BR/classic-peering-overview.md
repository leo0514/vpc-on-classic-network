---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-29"

keywords: peering, classic, infrastructure, VRF, resources

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

# Peering clássico para o VPC
{: #setting-up-classic-peering-for-vpc}

O {{site.data.keyword.cloud}} VPC é capaz de fazer peering com os recursos clássicos do IBM Cloud. Suas contas devem atender a alguns requisitos para que o peer seja possível.

Para obter instruções passo a passo sobre como criar um VPC com o peering clássico ativado, consulte nosso [documento de infraestrutura da VPC](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#setting-up-access-to-your-classic-infrastructure-from-vpc) que abrange esse procedimento com mais detalhes.

## Requisitos
{: #classic-peering-requirements}

1. O VPC deve ser designado para "peering clássico" no momento da criação, não pode ser designado em nenhum momento posterior.

2. A conta vinculada à qual você está fazendo peering de VPC deve ser ativada para VRF. Para obter mais informações sobre VRF, consulte [esse documento explicativo](/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud).

3. Também se deve incluir uma rota de volta para seu VPC ativado para o peering clássico por meio de cada instância que você está usando no lado da infraestrutura clássica.

## Restrições
{: #classic-peering-restrictions}

Somente um VPC por região pode ser ativado para o peering clássico.
