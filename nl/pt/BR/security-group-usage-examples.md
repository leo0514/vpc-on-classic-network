---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: security groups, RIAS, API, delete, create

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# Configurando grupos de segurança usando as APIs
{: #setting-up-security-groups-using-the-apis}

O exemplo a seguir demonstra como criar e gerenciar grupos de segurança usando as APIs do {{site.data.keyword.cloud}} VPC.

## Pré-requisitos
{: #security-group-prerequisites}

Para usar grupos de segurança, primeiro deve-se ter um {{site.data.keyword.cloud_notm}} VPC em execução.

As instruções para criar uma VPC e uma sub-rede estão disponíveis em nosso tópico [Criando uma VPC](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Etapa 1: criar um grupo de segurança
{: #step-1-create-a-security-group}

Crie um grupo de segurança denominado `my-security-group` em seu {{site.data.keyword.cloud_notm}} VPC.

```
curl -X POST "$rias_endpoint/v1/security_groups?version=2019-05-31&generation=1" \
  -H "Authorization: $iam_token" \
  -d '{
        "name": "my-security-group",
        "vpc": { "id": "'$vpc'" }
      }'
```
{: pre}

Salve o ID em uma variável para que possamos usá-lo posteriormente, por exemplo, a variável `sg`:

```
sg=2d364f0a-a870-42c3-a554-000000632953
```
{: pre}

## Etapa 2: incluir uma regra para permitir conexões SSH
{: #step-2-add-a-rule-to-allow-ssh-connections}

Crie uma regra sobre o grupo de segurança que permitirá conexões de entrada na porta 22.

```
curl -X POST "$rias_endpoint/v1/security_groups/$sg/rules?version=2019-05-31&generation=1" \
  -H "Authorization: $iam_token" \
  -d '{
        "direction": "inbound",
        "protocol": "tcp",
        "port_min": 22,
        "port_max": 22
      }'
```
{: pre}

## Etapa 3: excluir o grupo de segurança (opcional)
{: #step-3-delete-the-securiy-group-optional}

Para limpar o grupo de segurança, ele não pode ser associado a nenhuma interface de rede e não pode ser referenciado por uma regra em um grupo de segurança diferente.

```
curl -X DELETE "$rias_endpoint/v1/security_groups/$sg?version=2019-05-31&generation=1" \
  -H "Authorization: $iam_token"
```
{: pre}
