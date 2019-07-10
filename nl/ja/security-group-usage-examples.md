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

# API を使用したセキュリティー・グループのセットアップ
{: #setting-up-security-groups-using-the-apis}

以下の例では、{{site.data.keyword.cloud}} VPC API を使用してセキュリティー・グループを作成および管理する方法を説明します。

## 前提条件
{: #security-group-prerequisites}

セキュリティー・グループを使用するには、先に {{site.data.keyword.cloud_notm}} VPC を実行しておく必要があります。

VPC とサブネットの作成手順については、[VPC の作成](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis)のトピックを参照してください。

## 手順 1: セキュリティー・グループを作成する
{: #step-1-create-a-security-group}

{{site.data.keyword.cloud_notm}} VPC に `my-security-group` という名前のセキュリティー・グループを作成します。

```
curl -X POST "$rias_endpoint/v1/security_groups?version=2019-05-31&generation=1" \
  -H "Authorization: $iam_token" \
  -d '{
        "name": "my-security-group",
        "vpc": { "id": "'$vpc'" }
      }'
```
{: pre}

後で利用できるように、ID を変数に保存します。例えば、以下のように変数 `sg` に保存します。

```
sg=2d364f0a-a870-42c3-a554-000000632953
```
{: pre}

## 手順 2: SSH 接続を許可するルールを追加する
{: #step-2-add-a-rule-to-allow-ssh-connections}

ポート 22 でのインバウンド接続を許可するルールをセキュリティー・グループに作成します。

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

## 手順 3: セキュリティー・グループを削除する (オプション)
{: #step-3-delete-the-securiy-group-optional}

セキュリティー・グループをクリーンアップするためには、そのセキュリティー・グループにネットワーク・インターフェースが関連付けられていてはいけません。また、別のセキュリティー・グループのルールで参照されていてもいけません。

```
curl -X DELETE "$rias_endpoint/v1/security_groups/$sg?version=2019-05-31&generation=1" \
  -H "Authorization: $iam_token"
```
{: pre}
