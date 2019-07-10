---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, Cisco, ASAv, connection, secure, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc} 
{:new_window: target="_blank"} 
{:DomainName: data-hd-keyref="DomainName"} 
{:note: .note} 
{:important: .important} 
{:deprecated: .deprecated} 
{:generic: data-hd-programlang="generic"}

# Criando uma conexão segura com um peer Cisco ASAv remoto
{: #creating-a-secure-connection-with-a-remote-cisco-asav-peer}

Este documento é baseado no Cisco ASAv, Cisco Adaptive Security Appliance Software Versão 9.10(1).

As etapas de exemplo a seguir ignoram as etapas de pré-requisito de uso da API ou da CLI do {{site.data.keyword.cloud}} para criar Nuvens Particulares Virtuais. Para obter mais informações, consulte [Introdução](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) e [Configuração de VPC com APIs](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Etapas de exemplo
{: #cisco-example-steps}

A topologia para conexão com o peer do Cisco ASAv remoto é semelhante à criação de uma conexão VPN entre duas Nuvens Particulares Virtuais do {{site.data.keyword.cloud}}. No entanto, um lado é substituído pela unidade Cisco ASAv.

![inserir a descrição de imagem aqui](./images/vpc-vpn-asav-figure.png)

### Para criar uma conexão segura com o peer Cisco ASAv remoto
{: #to-create-a-secure-connection-with-the-remote-cisco-asav-peer}

A primeira etapa na configuração do Cisco ASA para uso com o VPN do IBM VPC é assegurar que as seguintes condições obrigatórias tenham sido configuradas:

* O Cisco ASAv estar on-line e funcional com uma licença adequada
* Uma senha para o Cisco ASAv estar ativada
* Haver pelo menos uma interface interna funcional configurada e verificada
* Haver pelo menos uma interface externa funcional configurada e verificada

Quando uma unidade Cisco ASAv recebe uma solicitação de conexão de um peer VPN remoto, ela usa os parâmetros da Fase 1 do IPsec para estabelecer uma conexão segura e autenticar esse peer VPN. Em seguida, se a política de segurança permitir a conexão, o Cisco ASAv estabelecerá o túnel utilizando os parâmetros da Fase 1 do IPsec e aplicará a política de segurança do IPsec. Os serviços de gerenciamento de chaves, de autenticação e de segurança são negociados dinamicamente por meio do protocolo IKE.

**Para suportar essas funções, as etapas de configuração geral a seguir devem ser executadas pela unidade Cisco ASAv:**

* Definir os parâmetros da Fase 1 que a unidade Cisco ASAv requer para autenticar o peer remoto e estabelecer uma conexão segura.
* Definir os parâmetros da Fase 2 que a unidade Cisco ASAv requer para criar um túnel VPN com o peer remoto.

Criar um objeto de proposta do Internet Key Exchange (IKE) versão 2. Os objetos de proposta do IKEv2 contêm os parâmetros necessários para criar propostas do IKEv2 ao definir o acesso remoto e as políticas de VPN de site para site. O IKE é um protocolo de gerenciamento de chaves que facilita o gerenciamento das comunicações baseadas em IPsec. Ele é usado para autenticar os peers IPsec, negociar e distribuir chaves de criptografia de IPsec e estabelecer automaticamente associações de segurança de IPsec (SAs). 

```
group-policy GroupPolicy_161.156.80.10 internal
group-policy GroupPolicy_161.156.80.10 attributes
 vpn-tunnel-protocol ikev1 ikev2
tunnel-group 161.156.80.10 type ipsec-l2l
tunnel-group 161.156.80.10 general-attributes
 default-group-policy GroupPolicy_161.156.80.10
tunnel-group 161.156.80.10 ipsec-attributes
 ikev1 pre-shared-key <key value>
 ikev2 remote-authentication pre-shared-key <key value>
 ikev2 local-authentication pre-shared-key <key value>
```

Crie uma configuração de política IKEv2 para a conexão IPsec. O bloco de política IKEv2 configura os parâmetros para a troca IKE. Nesse bloco, os parâmetros a seguir são configurados:
* Algoritmo de criptografia: configurado como AES-256 para esse exemplo
* Algoritmo de integridade: configurado como SHA256 para esse exemplo
* Grupo Diffie-Hellman: o IPsec usa o algoritmo Diffie-Hellman para gerar a chave de criptografia inicial entre os peers. Neste exemplo, ele é configurado para o grupo 14
* Pseudo-Random Function (PRF): o IKEv2 requer um método separado usado como o algoritmo para derivar material de chave e operações de hashing necessários para a criptografia do túnel IKEv2. Isso é chamado de função pseudoaleatória e é configurado como SHA
* Tempo de vida de SA: configurar o tempo de vida das associações de segurança (após o qual uma reconexão ocorrerá). Configurado para 36.000 segundos.
* Tipo de operação: mantenha-o como o valor padrão, bidirecional. (Ele não é explícito na exibição "mostrar em execução".)

Conforme mostrado no exemplo de código a seguir, essa política de amostra usa AES-256 para criptografar o canal seguro. O algoritmo hash SHA512 é usado para validar a identidade do peer remoto e o grupo Diffie-Hellman 14 é usado para a geração de chave. O grupo 14 usa blocos de criptografia de 2048 bits. Por fim, um
tempo de vida para a associação de segurança é configurado como 36.000 segundos.

```
crypto ikev2 policy 100
encryption aes-256
integrity sha-1
group 14
prf sha
lifetime seconds 36000
```

* Defina a lista de acesso e o mapa criptográfico para a VPN:

```
access-list outside_cryptomap_1 extended permit ip object NETWORK_OBJ_192.168.236.0_24 object vpc 
crypto map outside_map 1 match address outside_cryptomap_1
crypto map outside_map 1 set peer 161.156.80.10 
crypto map outside_map 1 set ikev1 transform-set ESP-AES-128-SHA ESP-AES-128-MD5 ESP-AES-192-SHA ESP-AES-192-MD5 ESP-AES-256-SHA ESP-AES-256-MD5 ESP-3DES-SHA ESP-3DES-MD5 ESP-DES-SHA ESP-DES-MD5
crypto map outside_map 1 set ikev2 ipsec-proposal AES256 AES192 AES 3DES DES
crypto map outside_map interface outside
nat (any,outside) source static NETWORK_OBJ_192.168.236.0_24 NETWORK_OBJ_192.168.236.0_24 destination static vpc vpc no-proxy-arp route-lookup
```

## Para criar uma conexão segura com o IBM Cloud VPC local
{: #to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

Para criar uma conexão segura, você criará a conexão VPN dentro de seu VPC, que é semelhante ao exemplo 2 de VPC.

* Crie um gateway VPN em sua sub-rede da VPC juntamente com uma conexão VPN entre a VPC e o Cisco ASAv, configurando `local_cidrs` para o valor de sub-rede na VPC e `peer_cidrs` para o valor de sub-rede no Cisco ASAv.

O status do gateway aparece como `pending` enquanto o gateway VPN está sendo criado e o status se torna `available` assim que a criação é concluída. A criação pode levar algum tempo. 
{:note}


![inserir descrição de imagem aqui](./images/vpc-vpn-asav-connection.png)

### Verifique o status da conexão segura
{: #cisco-check-the-status-of-the-secure-connection}

É possível verificar o status de sua conexão por meio do console do IBM Cloud. Além disso, é possível tentar executar um `ping` de site para site usando os VSIs.

![inserir descrição de imagem aqui](./images/vpc-vpn-asav-status.png)
