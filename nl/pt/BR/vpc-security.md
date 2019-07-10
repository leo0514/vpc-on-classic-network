---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-29"

keywords: security, ACLs, security groups, traffic, subnet, instance, VSI, firewall, encryption

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# Segurança em seu IBM Cloud VPC
{: #security-in-your-ibm-cloud-vpc}

É possível manter seu VPC e as cargas de trabalho seguras controlando o tráfego de rede usando grupos de segurança (SGs), usando as listas de controle de acesso à rede (ACLs) ou usando ambos os tipos de controle. Lembre-se:

* Os grupos de segurança controlam o tráfego em uma base por instância (VSI).
* As listas de controle de acesso controlam o tráfego em uma base por sub-rede.

## Visão geral da segurança
{: #security-overview}

* O tráfego para e de uma sub-rede pode ser controlado pelas listas de controle de acesso (ACLs)
* Os grupos de segurança (SGs) podem controlar o tráfego no nível da VSI
* Configure um gateway público para acesso de sub-rede à Internet, guardado por ACLs
* Configure um IP flutuante para acesso de VSI à Internet, guardado por SGs

![Conectividade e segurança do IBM VPC](images/vpc-connectivity-and-security.svg "Conectividade e segurança do IBM VPC")

## Definições
{: #definitions}

Esse [glossário](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary) fornece definições e descrições de ACLs e SGs e as ações que podem ser executadas com eles. A seção a seguir descreve a funcionalidade básica de ACLs e grupos de segurança e as maneiras nas quais o VPC suporta criptografia de ponta a ponta.

### Lista de controle de acesso
{: #access-control-list}

Uma **Lista de controle de acesso (ACL)** pode gerenciar (ou seja, permitir ou negar) o tráfego de entrada e saída para uma sub-rede. Uma ACL é stateless, o que significa que as regras de entrada e de saída devem ser especificadas separadamente e explicitamente. Cada ACL consiste em regras baseadas em um *IP de origem*, *porta de origem*, *IP de destino*, *porta de destino* e *protocolo*.

No {{site.data.keyword.cloud}} VPC, cada sub-rede é criada com uma ACL padrão, que permite tráfego de entrada e de saída, mas os clientes podem criar ACLs customizadas. Apenas uma ACL é anexada a uma sub-rede a qualquer momento, mas uma ACL pode ser anexada a múltiplas sub-redes. Para obter mais informações sobre como usar as ACLs, consulte o nosso [Guia de ACL](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls).

### Grupo de Segurança
{: #security-group}

Um **grupo de segurança** age como um firewall virtual que controla o tráfego para um ou mais servidores (VSIs). Um grupo de segurança é uma coleção de regras que especificam se o tráfego deve ser permitido para uma VSI associada. 

Quando um cliente cria um VSI, ele pode associar um ou mais grupos de segurança a essa VSI. Com as permissões corretas, os clientes podem modificar as regras do grupo de segurança usando o IBM Console, a CLI ou a API.

Para obter mais informações sobre como criar uma VSI que usa grupos de segurança e mais sobre os recursos dos grupos de segurança, consulte o nosso [Guia de grupos de segurança](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-using-security-groups).

### Criptografia de ponta a ponta
{: #end-to-end-encryption}

Embora o IBM Cloud VPC não forneça criptografia de ponta a ponta, ele a permite. Por exemplo, se você tiver um terminal seguro em um servidor virtual (por exemplo, um servidor HTTPS na Porta 443), será possível conectar um IP flutuante a esse servidor e, em seguida, sua conexão será criptografada de ponta a ponta do cliente até o servidor na Porta 443.  Nada no caminho força uma decriptografia.

Observe, no entanto, que, se você usar um protocolo inseguro, como HTTP na Porta 80, seus dados estarão às claras de ponta a ponta.
