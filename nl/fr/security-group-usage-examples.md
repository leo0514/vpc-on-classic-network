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

# Configuration des groupes de sécurité à l'aide des API
{: #setting-up-security-groups-using-the-apis}

L'exemple suivant montre comment créer et gérer les groupes de sécurité à l'aide des API du VPC {{site.data.keyword.cloud}}.

## Prérequis
{: #security-group-prerequisites}

Pour utiliser les groupes de sécurité, vous devez d'abord disposer d'une instance {{site.data.keyword.cloud_notm}} fonctionnelle.

Les instructions de création d'un VPC et d'un sous-réseau sont disponibles dans notre rubrique [Création d'un VPC](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Etape 1 : création d'un groupe de sécurité
{: #step-1-create-a-security-group}

Créez un groupe de sécurité appelé `my-security-group` dans votre instance de VPC {{site.data.keyword.cloud_notm}}.

```
curl -X POST "$rias_endpoint/v1/security_groups?version=2019-05-31&generation=1" \
  -H "Authorization: $iam_token" \
  -d '{
        "name": "my-security-group",
        "vpc": { "id": "'$vpc'" }
      }'
```
{: pre}

Enregistrez l'ID dans une variable afin de pouvoir l'utiliser ultérieurement, par exemple, dans la variable `sg` :

```
sg=2d364f0a-a870-42c3-a554-000000632953
```
{: pre}

## Etape 2 : ajout d'une règle pour autoriser les connexions SSH
{: #step-2-add-a-rule-to-allow-ssh-connections}

Créez une règle sur le groupe de sécurité qui autorise les connexions entrantes sur le port 22.

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

## Etape 3 : suppression du groupe de sécurité (facultatif)
{: #step-3-delete-the-securiy-group-optional}

Le groupe de sécurité peut être uniquement être supprimé s'il n'est pas associé à une interface réseau, ni référencé par une règle d'un autre groupe de sécurité.

```
curl -X DELETE "$rias_endpoint/v1/security_groups/$sg?version=2019-05-31&generation=1" \
  -H "Authorization: $iam_token"
```
{: pre}
