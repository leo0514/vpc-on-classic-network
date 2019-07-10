---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, Vyatta, connection, secure, remote

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


# Criando uma conexão segura com um peer Vyatta remoto
{: #creating-a-secure-connection-with-a-remote-vyatta-peer}

Este documento é baseado na versão Vyatta: AT&T vRouter 5600 1801d.

As etapas de exemplo a seguir ignoram as etapas de pré-requisito de uso da API ou da CLI do {{site.data.keyword.cloud}} para criar Nuvens Particulares Virtuais. Para obter mais informações, consulte [Introdução](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) e [Configuração de VPC com APIs](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Etapas de exemplo
{: #vyatta-example-steps}

A topologia para conexão com o peer do Vyatta remoto é semelhante à criação de uma conexão VPN entre duas VPCs do {{site.data.keyword.cloud_notm}}. No entanto, um lado é substituído pela unidade Vyatta.

![inserir descrição de imagem aqui](images/vpc-vpn-vy-figure.png)

### Para criar uma conexão segura com o peer Vyatta remoto
{: #vyatta-to-create-a-secure-connection-with-the-remote-vyatta-peer}

Quando um peer do VPN recebe uma solicitação de conexão de um peer do VPN remoto, ele usa os parâmetros da Fase 1 do IPsec para estabelecer uma conexão segura e autenticar esse peer do VPN. Em seguida, se a política de segurança permitir a conexão, a unidade Vyatta estabelecerá o túnel usando os parâmetros da Fase 2 do IPsec e aplicará a política de segurança IPsec. Os serviços de gerenciamento de chaves, de autenticação e de segurança são negociados dinamicamente por meio do protocolo IKE.

É possível acessar a linha de comandos do Vyatta para configurar o túnel IPsec ou fazer upload de um arquivo `*.vcli` para carregar a sua configuração.

**Para suportar essas funções, as etapas de configuração geral a seguir devem ser executadas pela unidade Vyatta: **

* Defina os parâmetros da Fase 1 que a unidade Vyatta requer para autenticar o peer remoto e estabelecer uma conexão segura.

* Defina os parâmetros da Fase 2 que a unidade Vyatta requer para criar um túnel VPN com o peer remoto.

Para se conectar ao recurso VPN do IBM Cloud VPC, recomendamos a configuração a seguir:

1. Escolha `IKEv2` na autenticação;
2. Ative `DH-group 2` na proposta da Fase 1
3. Configure `lifetime = 36000` na proposta da Fase 1
4. Desativar PFS na proposta da Fase 2
5. Configure `lifetime = 10800` na proposta da Fase 2
6. Insira as informações dos peers e das sub-redes na Fase 2

```
vim vyatta_temp/create_vpn.vcli
#!/bin/vcli -f
configure

set security vpn ipsec ike-group 169.61.161.151_test_ike
set security vpn ipsec ike-group 169.61.161.151_test_ike dead-peer-detection timeout 120
set security vpn ipsec ike-group 169.61.161.151_test_ike lifetime 36000
set security vpn ipsec ike-group 169.61.161.151_test_ike ike-version 2

set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 dh-group 2
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 encryption aes256
set security vpn ipsec ike-group 169.61.161.151_test_ike proposal 1 hash sha2_256
set security vpn ipsec esp-group 169.61.161.151_test_ipsec compression disable
set security vpn ipsec esp-group 169.61.161.151_test_ipsec lifetime 10800
set security vpn ipsec esp-group 169.61.161.151_test_ipsec mode tunnel
set security vpn ipsec esp-group 169.61.161.151_test_ipsec pfs disable


set security vpn ipsec esp-group 169.61.161.151_test_ipsec proposal 1 encryption aes256
set security vpn ipsec esp-group 169.61.161.151_test_ipsec proposal 1 hash sha2_256
set security vpn ipsec site-to-site peer 169.61.161.151 authentication mode pre-shared-secret
set security vpn ipsec site-to-site peer 169.61.161.151 authentication pre-shared-secret ******
set security vpn ipsec site-to-site peer 169.61.161.151 ike-group 169.61.161.151_test_ike
set security vpn ipsec site-to-site peer 169.61.161.151 default-esp-group 169.61.161.151_test_ipsec
set security vpn ipsec site-to-site peer 169.61.161.151 description "automation test"
set security vpn ipsec site-to-site peer 169.61.161.151 local-address 169.45.74.119
set security vpn ipsec site-to-site peer 169.61.161.151 connection-type initiate


set security vpn ipsec site-to-site peer 169.61.161.151 tunnel 1 local prefix 192.168.200.0/24
set security vpn ipsec site-to-site peer 169.61.161.151 tunnel 1 remote prefix 192.168.17.0/28

commit
```
{: screen}

Em seguida, é possível fazer upload deste arquivo `*.vcli` para o Vyatta usando o SCP para aplicar a configuração.

`scp example.vcli <vyatta_username>@<vyatta_ip>`

### Para criar uma conexão segura com o IBM Cloud VPC local
{: #vyatta-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

 Para criar uma conexão segura, você criará a conexão VPN dentro de seu VPC, que é semelhante ao exemplo 2 de VPC.

* Crie um gateway VPN em sua sub-rede da VPC juntamente com uma conexão VPN entre a VPC e a unidade Vyatta, configurando `local_cidrs` para o valor de sub-rede na VPC e `peer_cidrs` para o valor de sub-rede na unidade Vyatta.

O status do gateway aparece como `pending` enquanto o gateway VPN está sendo criado e o status se torna `available` assim que a criação é concluída. A criação pode levar algum tempo.
{: note}

![inserir descrição de imagem aqui](images/vpc-vpn-vy-connection.png)

### Verifique o status da conexão segura
{: #vyatta-check-the-status-of-the-secure-connection}

É possível verificar o status de sua conexão por meio do console do {{site.data.keyword.cloud_notm}}. Além disso, é possível tentar executar um `ping` de site para site usando os VSIs.

![inserir descrição de imagem aqui](images/vpc-vpn-vy-status.png)
