---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, StrongSwan, connection, secure, Linux, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{: note: .note}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Criando uma conexão segura com um peer StrongSwan remoto
{: #creating-a-secure-connection-with-a-remote-strongswan-peer}

Este documento é baseado no Strongswan, versão Linux StrongSwan U5.3.5/K4.4.0-133-generic.

As etapas de exemplo a seguir ignoram as etapas de pré-requisito de uso da API ou da CLI do {{site.data.keyword.cloud}} para criar Nuvens Particulares Virtuais. Para obter mais informações, consulte [Introdução](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) e [Configuração de VPC com APIs](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Etapas de exemplo
{: #strongswan-example-steps}

A topologia para conexão com o peer do StrongSwan remoto é semelhante à criação de uma conexão VPN entre duas VPCs. No entanto, um lado da conexão é substituído pela unidade StrongSwan.

![inserir descrição de imagem aqui](./images/vpc-vpn-sw-figure.png)

### Para criar uma conexão segura com um peer StrongSwan remoto
{: #to-create-a-secure-connection-with-a-remote-strongswan-peer}

Acesse **/etc** e crie seu novo arquivo de configuração de túnel customizado com um nome semelhante a **ipsec.abc.conf**. Edite **/etc/ipsec.conf** para incluir **ipsec.abc.conf** inserindo esta linha:

    include /etc/ipsec.abc.conf

Quando um peer do VPN recebe uma solicitação de conexão de um peer do VPN remoto, ele usa os parâmetros da Fase 1 do IPsec para estabelecer uma conexão segura e autenticar esse peer do VPN. Em seguida, se a política de segurança permitir a conexão, a unidade StrongSwan estabelecerá o túnel usando os parâmetros da Fase 2 do IPsec e aplicará a política de segurança IPsec. Os serviços de gerenciamento de chaves, de autenticação e de segurança são negociados dinamicamente por meio do protocolo IKE.

**Para suportar essas funções, as etapas de configuração geral a seguir devem ser executadas pela unidade StrongSwan:**

* Defina os parâmetros da Fase 1 que o StrongSwan requer para autenticar o peer remoto e estabelecer uma conexão segura.

* Defina os parâmetros da Fase 2 que o StrongSwan requer para criar um túnel VPN com o peer remoto.
Para se conectar ao recurso VPN do IBM Cloud VPC, recomendamos a configuração a seguir:

1. Escolha `IKEv2` na autenticação;
2. Ative `DH-group 2` na proposta da Fase 1
3. Configure `lifetime = 36000` na proposta da Fase 1
4. Desativar PFS na proposta da Fase 2
5. Configure `lifetime = 10800` na proposta da Fase 2
6. Insira as informações de seu peer e sub-rede na proposta da Fase 2

```
    vim /etc/ipsec.abc.conf
    conn all
           type=tunnel
           auto=route
           #aggressive=no
           esp=aes256-sha256!
           ike=aes128-sha1-modp1024!
           left=169.45.74.119
           leftsubnet=10.160.26.64/26
           rightsubnet=192.168.17.0/28
           right=169.61.181.116
           leftauth=psk
           rightauth=psk
           leftid="169.45.74.119"
           keyexchange=ikev2
           rightid="169.61.181.116"
           lifetime=10800s
           ikelifetime=36000s
           dpddelay=30s
           dpdaction=restart
           dpdtimeout=120s
```
{: screen}

Configure a chave pré-compartilhada em `/etc/ipsec.secrets`

```
vim ipsec.secrets
# This file holds shared secrets or RSA private keys for authentication.

169.45.74.119 169.61.181.116 : PSK "******"

```
{: screen}

Depois que o arquivo de configuração terminar a execução, reinicie a unidade StrongSwan.

```
 ipsec restart
```
{: screen}

### Para criar uma conexão segura com o IBM Cloud VPC local
{: #strongswan-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

* Para criar uma conexão segura, você criará a conexão VPN dentro de seu VPC, que é semelhante ao exemplo 2 de VPC.

* Crie um gateway de VPN na sua sub-rede do VPC, juntamente com uma conexão de VPN entre o VPC e o StrongSwan, configurando `local_cidrs` como o valor de sub-rede no VPC e `peer_cidrs` como o valor de sub-rede no StrongSwan.

O status do gateway aparece como `pending` enquanto o gateway VPN está sendo criado e o status se torna `available` assim que a criação é concluída. A criação pode levar algum tempo.
{: note}

![vpc-vpn-sw-connection](./images/vpc-vpn-sw-connection.png)

### Verifique o status para uma conexão segura
{: #strongswan-check-the-status-for-a-secure-connection}

É possível verificar o status de sua conexão por meio do console do IBM Cloud. Além disso, é possível tentar executar um `ping` de site para site usando os VSIs.

![vpc-vpn-sw-status.png](./images/vpc-vpn-sw-status.png)
