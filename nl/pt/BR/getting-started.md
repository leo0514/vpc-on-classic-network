---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-06-06"

keywords: provisioning, resources, permissions

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:important: .important}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Tutorial de introdução
{: #getting-started}

Para começar a usar redes para o {{site.data.keyword.cloud}} Virtual Private Cloud:

1. Crie uma nuvem particular virtual.
2. Crie uma ou mais sub-redes na nuvem particular virtual em uma ou mais zonas.
3. Crie um gateway público (PGW) em uma sub-rede se você desejar que os recursos em sua sub-rede tenham acesso à Internet ou vice-versa.

## Pré-requisitos

 * **Permissões do usuário**: certifique-se de que seu usuário tenha permissões suficientes para criar e gerenciar recursos na VPC. Para obter uma lista de permissões necessárias, consulte [Concedendo permissões necessárias para usuários da VPC](/docs/vpc-on-classic?topic=vpc-on-classic-managing-user-permissions-for-vpc-resources).

## Usar a IU, a CLI ou a API de REST para provisionar os recursos de rede VPC

O fornecimento e o gerenciamento de recursos de rede VPC podem ser feitos por meio da IU, da CLI ou da API de REST.

* Para acesso por meio da interface com o usuário, efetue login no [Console do IBM Cloud ![Ícone de link externo](../../icons/launch-glyph.svg "Ícone de link externo")]( https://{DomainName}/vpc){: new_window}. Siga o [guia da IU](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console) se você precisar de ajuda.
* Para usar a interface da linha de comandos, siga o exemplo [Hello World](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli).
* Para usuários mais avançados, é possível chamar as [APIs de REST](https://{DomainName}/apidocs/vpc-on-classic) diretamente. Siga o tutorial [código de exemplo](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis) para introdução às APIs de REST.

## Próximas etapas

Depois que sua rede Nuvem Particular Virtual tiver sido provisionada, explore mais.

* [Criando e gerenciando instâncias de servidor virtual](/docs/vpc-on-classic?topic=vpc-on-classic-creating-and-managing-virtual-server-instances)
* [Usando grupos de segurança](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-cli)
* [Usando ACLs de rede](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls)
* [Usando a VPN](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-vpn-with-your-vpc)
* [Usando Load Balancers](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-load-balancers-in-ibm-cloud-vpc)
