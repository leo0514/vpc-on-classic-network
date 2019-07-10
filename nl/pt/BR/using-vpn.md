---



copyright:
  years: 2017,2018, 2019
lastupdated: "2019-06-04"

keywords: VPN, network, encryption, authentication, algorithm, IKE, IPsec, policies, gateway, auto-negotiation

subcollection: vpc-on-classic-network


---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Usando a VPN com seu VPC
{: #--using-vpn-with-your-vpc}
[comment]: # (tópico da ajuda vinculado)

O serviço VPN do {{site.data.keyword.cloud}} VPC permite conectar redes privadas de maneira segura. É possível usar a VPN para configurar um túnel de site para site do IPsec entre a VPC e sua rede privada no local ou outra VPC.

Para a liberação atual do {{site.data.keyword.cloud}} VPC, somente roteamento baseado em política é suportado.

## Recursos
{: #vpn-features}

* IKEv1 e IKEv2
* Algoritmos de autenticação: `md5`, `sha1`, `sha256`
* Algoritmos de criptografia: `3des`, `aes128`, `aes256`
* Grupos Diffie-Hellman (DH): 2, 5, 14
* Modo de negociação IKE: principal
* Protocolo de transformação IPSec: ESP
* Modo de encapsulamento IPSec: túnel
* Perfect Forward Secrecy (PFS)
* Dead Peer Detection
* Roteamento: baseado em política
* Modo de Autenticação: Chave pré-compartilhada
* Suporte de HA no modo Ativo/Espera somente

## APIs disponíveis
{: #apis-available}

A seção a seguir fornece detalhes sobre as APIs que podem ser usadas para a VPN em seu ambiente do IBM Cloud VPC. Consulte a página [APIs de REST da VPC](https://{DomainName}/apidocs/vpc-on-classic#list-all-ike-policies) para obter mais detalhes.

### Gateways VPN e conexões VPN
{: #vpn-gateways-and-vpn-connections}

| Descrição | API |
|----------------------------|-------------|
| Cria um gateway VPN | POST /vpn_gateways |
| Recupera gateways VPN | GET /vpn_gateways |
| Recupera um gateway VPN | GET /vpn_gateways/{id} |
| Exclui um gateway VPN | DELETE /vpn_gateways/{id} |
| Atualiza um gateway VPN | PATCH /vpn_gateways/{id} |
| Cria uma nova conexão VPN | POST /vpn_gateways/{vpn_gateway_id}/connections |
| Recupera conexões VPN | GET /vpn_gateways/{vpn_gateway_id}/connections |
| Recupera uma conexão VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Exclui uma conexão VPN | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Atualiza uma conexão VPN | PATCH /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Recupera todos os CIDRs locais para uma conexão VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs |
| Exclui um CIDR local de uma conexão VPN | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Verifica se existe um CIDR local específico em uma conexão VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Configura um CIDR local em uma conexão VPN | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Recupera todos os CIDRs de peer para uma conexão VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs |
| Exclui um CIDR de peer de uma conexão VPN | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Verifica se existe um CIDR de peer específico em uma conexão VPN | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Configura um CIDR de peer em uma conexão VPN | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |

### Políticas do IKE
{: #ike-policies}

| Descrição | API |
|-----------------------------|--------------|
| Recupera todas as políticas de IKE | GET /ike_policies |
| Cria uma política de IKE | POST /ike_policies |
| Exclui uma política de IKE | DELETE /ike_policies/{id} |
| Recupera uma política de IKE | GET /ike_policies/{id} |
| Atualiza uma política de IKE | PATCH /ike_policies/{id} |
| Recupera todas as conexões que usam a política de IKE especificada | GET /ike_policies/{id}/connections |

### Políticas do IPsec
{: #ipsec-policies}

| Descrição | API |
|---------------------|-------------|
| Recupera todas as políticas de IPSec | GET /ipsec_policies |
| Cria uma política de IPSec | POST /ipsec_policies |
| Exclui uma política de IPSec | DELETE /ipsec_policies/{id} |
| Recupera uma política de IPSec | GET /ipsec_policies/{id} |
| Atualiza uma política de IPSec | PATCH /ipsec_policies/{id} |
| Recupera todas as conexões que usam a política de IPsec especificada | GET /ipsec_policies/{id}/connections |

## Exemplo de VPN
{: #vpn-example}

No exemplo a seguir, será possível conectar duas VPCs juntas usando a VPN, o que significa que será possível conectar
sub-redes em duas VPCs separadas como se fossem uma única rede. Os endereços IP das sub-redes não devem se sobrepor.
Aqui está a aparência do cenário (com algumas VMs incluídas em cada VPC):

![VPN para o IBM VPC](images/vpc-vpn.svg "VPN para o IBM VPC")

### Etapas de exemplo
{: #vpn-example-steps}

As etapas de exemplo a seguir ignoram as etapas de pré-requisito de uso da API ou da CLI do IBM Cloud para criar as VPCs. Para obter mais informações, consulte [Introdução](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) e [Configuração de VPC com APIs](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

Opcionalmente, é possível criar um gateway VPN utilizando a UI. As etapas podem ser localizadas no [Documento do tutorial do console](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn).

#### Etapa 1. Criar um gateway VPN em sua sub-rede VPC
{: #step-1-create-a-vpn-gateway-in-your-vpc-subnet}

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways?version=2019-05-31&generation=1" \
    -d '{
            "name": "vpn-gateway-1",
            "subnet": {"id": $subnet1}
        }'
```
{: codeblock}

Saída de amostra:
```
{
    "id": "7fd72524-6e2d-49a6-b975-0071efccd89a",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:7fd72524-6e2d-49a6-b975-0071efccd89a",
    "name": "vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a",
    "created_at": "2018-07-06T19:19:28.694388Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.167"
    },
    "subnet": {
        "id": "f45ee0be-cf3f-41ca-a279-23139110aa58",
        "name": "subnet-1",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f45ee0be-cf3f-41ca-a279-23139110aa58"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

Assegure-se de que os campos a seguir sejam salvos para as etapas subsequentes.
* `id`. Esse é o ID do gateway de VPN e ele será chamado de `$gwid1`.
* `address`. Esse é o endereço IP público do gateway VPN e ele será referido como `$gwaddress1`.

O status do gateway aparece como `pending` enquanto o gateway VPN está sendo criado e o status se torna `available` assim que a criação é concluída. A criação pode levar algum tempo.
{: note}


É possível verificar o status do gateway com o comando a seguir:

```bash
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/vpn_gateways/$gwid1?version=2019-05-31&generation=1"
```
{: codeblock}

#### Etapa 2. Criar um segundo gateway VPN em uma VPC diferente
{: #step-2-create-a-second-vpn-gateway-on-a-different-vpc}

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-gateway-2",
                "subnet": {"id": $subnet2}
            }'
```
{: codeblock}

Saída de amostra:
```
{
    "id": "f72559a3-2fac-4958-b937-54474e6a8a8d",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:f72559a3-2fac-4958-b937-54474e6a8a8d",
    "name": "vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d",
    "created_at": "2018-07-06T19:33:23.789675Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.150"
    },
    "subnet": {
        "id": "f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4",
        "name": "subnet-2",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

Certifique-se de salvar os campos a seguir para as etapas subsequentes.
* `id`. Esse é o ID do gateway VPN e ele será referido como `$gwid2`.
* `address`. Esse é o endereço IP público do gateway VPN e ele será referido como `$gwaddress2`.


#### Etapa 3. Criar uma conexão de VPN por meio do primeiro gateway de VPN para o segundo gateway de VPN
{: #step-3-create-a-vpc-connection-from-the-first-vpn-gateway-to-the-second-vpn-gateway}

Quando você estiver criando a conexão, configure `local_cidrs` como a sub-rede em **VPC um** e `peer_cidrs` como a sub-rede em **VPC dois**.

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-2",
                "peer_address": $gwaddress2,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

Saída de amostra:
```
{
    "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "name": "vpn-connection-to-vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.0.0/24"
    ],
    "peer_address": "169.61.161.150",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
                "interval": 30,
                "timeout": 120
    },
    "created_at": "2018-07-06T19:50:49.252072Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### Etapa 4. Criar uma conexão de VPN do segundo gateway de VPN ao primeiro gateway de VPN
{: #step-4-create-a-vpn-connection-from-the-second-vpn-gateway-to-the-first-vpn-gateway}

Quando você estiver criando a conexão, configure `local_cidrs` como a sub-rede em **VPC dois** e `peer_cidrs` como a sub-rede em **VPC um**.

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways/$gwid2/connections?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-1",
                "peer_address": $gwaddress1,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

Saída de amostra:
```
{
    "id": "1d4dbacq-673d-2qed-hf68-858961739gf0",
    "name": "vpn-connection-to-vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d/connections/1d4dbacq-673d-2qed-hf68-858961739gf0",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_address": "169.61.161.167",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
                "interval": 30,
                "timeout": 120
    },
    "created_at": "2018-07-06T19:54:14.961597Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### Etapa 5. Verificar a conectividade
{: #step-5-verify-connectivity}

Após a conexão VPN ser estabelecida, você será capaz de atingir suas instâncias na sub-rede dois por meio da sub-rede um e vice-versa.

É possível verificar o status da conexão VPN conforme a seguir:
```bash
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-05-31&generation=1"
```
{: codeblock}

Saída de amostra:
```
{
    "first": {
        "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections?limit=10"
    },
    "limit": 10,
    "connections": [
        {
            "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "name": "vpn-connection-to-vpn-gateway-2",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "local_cidrs": [
                "192.168.100.0/24"
            ],
            "peer_cidrs": [
                "192.168.0.0/24"
            ],
            "peer_address": "169.61.161.150",
            "admin_state_up": true,
            "psk": "VPNDemoPassword",
            "dead_peer_detection": {
                "action": "none",
                "interval": 30,
                "timeout": 120
            },
            "created_at": "2018-07-06T19:50:49.252072Z",
            "route_mode": "policy",
            "authentication_mode": "psk",
            "status": "up"
        }
    ]
}
```
{: screen}

## Cotas
{: #see-vpn-quotas}

Veja nosso tópico [Cotas de VPC](/docs/vpc-on-classic?topic=vpc-on-classic-quotas#vpn-quotas) para cotas de VPN.

## Negociação automática de política
{: #policy-auto-negotiation}

O uso de políticas IKE e IPsec para configurar uma conexão de VPN é opcional. Quando nenhuma política é selecionada, as propostas padrão são escolhidas automaticamente para um processo chamado de _negociação automática_. 

A negociação automática do IBM Cloud usa o **IKEv2** e, portanto, o dispositivo no local também deve usar **IKEv2**. Use uma política IKE customizada se o dispositivo no local não suportar o **IKEv2**.
{: note}

### Negociação automática do IKE (fase 1)
{: #ike-auto-negotiation-phase-1}

As opções de criptografia, autenticação e Grupo Diffie-Hellman a seguir podem ser usadas em qualquer combinação:

|    | Encryption | Authentication | Grupo DH |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | 2  |
| 2  | aes256 | sha256 | 5  |
| 3  | 3des   | md5    | 14 |

### Negociação automática do IPsec (fase 2)
{: #ipsec-auto-negotiation-phase-2}

As opções de criptografia e autenticação a seguir podem ser usadas em qualquer combinação:

|    | Encryption | Authentication | Grupo DH |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | desativado  |
| 2  | aes256 | sha256 | desativado  |
| 3  | 3des   | md5    | desativado  |

## Perguntas frequentes
{: #vpn-faq}

**Quando eu criar um gateway VPN, poderei criar conexões VPN ao mesmo tempo?**

Se você usar a API ou a CLI, as conexões VPN deverão ser criadas após a criação do gateway VPN. No console do IBM Cloud, é possível criar o gateway e uma conexão ao mesmo tempo.

**Se eu excluir um gateway VPN que tem conexões VPN anexadas, o que acontecerá com as conexões?**

As conexões VPN serão excluídas juntamente com o gateway VPN.

**As políticas de IKE ou de IPSec serão excluídas se eu excluir um gateway VPN ou uma conexão VPN?**

Não, as políticas de IKE e de IPSec podem se aplicar a múltiplas conexões.

**O que acontecerá com um gateway VPN se eu tentar excluir a sub-rede em que o gateway está localizado?**

A sub-rede não poderá ser excluída se alguma instância estiver presente, incluindo o gateway VPN.

**Há políticas de IKE e IPsec padrão?**

Quando você cria uma conexão de VPN sem referenciar um ID de política (IKE ou IPsec), a negociação automática é usada.

**Por que eu preciso escolher uma sub-rede durante o fornecimento do gateway de VPN?**

A sub-rede conecta o gateway de VPN com outros recursos em seu VPC. A melhor prática recomendada é criar uma sub-rede dedicada para o gateway de VPN, com nenhuma outra instância de VPC nessa sub-rede, para assegurar que haja IPs privados livres suficientes na sub-rede. Um gateway de VPN precisa de oito endereços IP privados para acomodar os upgrades contínuos e de HA.

**Em qual zona o gateway de VPN reside?**

O gateway de VPN reside na zona com a sub-rede que você escolheu durante o fornecimento. Lembre-se de que o gateway de VPN só atende às instâncias de VPC na mesma zona e no mesmo VPC. Portanto, as instâncias de VPC em outras zonas não podem aproveitar o gateway de VPN para se comunicarem com uma rede privada no local. Para tolerância a falhas de zona, deve-se implementar um gateway de VPN por zona.

**O que deverei fazer se eu estiver usando ACLs na sub-rede que é usada para implementar o gateway de VPN?**

Certifique-se de que as regras de ACL a seguir estejam em vigor para permitir o tráfego de gerenciamento e o tráfego de túnel VPN:

* **Regras de entrada**
    - Permitir porta de origem TCP do protocolo 9091
    - Permitir porta de origem TCP do protocolo 10514
    - Permitir porta de origem TCP do protocolo 443
    - Permitir porta de origem TCP do protocolo 80
    - Permitir porta de origem TCP do protocolo 53
    - Permitir porta de origem UDP do protocolo 53
    - Permitir protocolo IP de origem ALL é IP público de gateway de peer VPN
    - Permitir porta de destino TCP do protocolo 443
    - Permitir porta de destino TCP do protocolo 56500
    - Permitir tráfego entre instâncias no VPC e a sua rede privada no local
    - Permitir tráfego de ICMP

* **Regras de saída**
   - Permitir todo o tráfego

**O que deverei fazer se estiver usando ACLs ou grupos de segurança nas sub-redes que precisam se comunicar com a rede privada no local?**

Você precisará assegurar que as regras de ACL ou as regras do grupo de segurança estejam em vigor para permitir o tráfego entre as instâncias em seu VPC e a sua rede privada no local.

**A VPN para VPC suporta as configurações de HA?**

Sim, ela suporta HA em uma configuração Ativo/Espera.

**Há planos para suportar a VPN SSL?**

Não, somente o IPsec site para site é suportado.

**Há algum valor máximo no rendimento para o VPNaaS de site para site?**

Suportamos até 650 Mbps de rendimento.

**A autenticação IKE baseada em PSK e Certificado é suportada para VPNaaS?**

Somente a autenticação PSK é suportada.

**É possível usar uma VPN para o VPC como um Gateway de VPN para sua infraestrutura clássica do IBM Cloud?**

Não, para usar o Gateway de VPN em seu ambiente da infraestrutura clássica do IBM Cloud, deve-se usar a [VPN IPsec](https://cloud.ibm.com/catalog/infrastructure/ipsec-vpn).
