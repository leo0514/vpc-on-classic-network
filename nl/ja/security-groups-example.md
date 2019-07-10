---

copyright:
  years: 2019

lastupdated: "2019-05-14"

keywords: security group, VSI, ping, TCP, outbound, SG4, rules, metadata, setting up

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# CLI を使用したセキュリティー・グループのセットアップ
{: #setting-up-security-groups-using-the-cli}

以下の例では、コマンド・ライン・インターフェース (CLI) を使用して、セキュリティー・グループが有効な VSI を作成します。 シナリオは次のようになります。

![IBM VPC のセキュリティー・グループ](images/security-groups-schematic.svg "IBM VPC のセキュリティー・グループ")

この図では、**SG4** という名前の VSI に、内部 VPC アドレス `10.0.0.5` に加えて浮動 IP `169.60.208.144` が割り当てられているため、この VSI はパブリック・インターネットと通信できます。 VSI **SG4** に割り当てられているセキュリティー・グループの名前は「demosg」です。

**SG8** という名前の VSI は VPC 内部専用であり、プライベート IP アドレスが割り当てられています。 VSI **SG8** に割り当てられているセキュリティー・グループの名前は「my_vpc_sg」です。 これらの VSI は両方とも、`sgvpc` という名前の VPC 内の同じサブネット `10.0.0.0/28` 上に存在しているので、相互に通信できます。

## セキュリティー・グループを付加して VSI を作成する手順
{: #steps-for-creating-a-vsi-with-a-security-group-attached}

「my_vpc_sg」のセキュリティー・グループ・ルールには、SSH、PING、アウトバウンド TCP の基本的な機能を含めます。

まずは `ibmcloud is sgc` コマンドを使用してセキュリティー・グループを作成してから、その新しく作成したセキュリティー・グループを含める VSI を作成する必要がある点に注意してください。

このサンプル・コードではいくつかのステップが省略されているため、詳細情報を以下で確認できます。

 * VPC とサブネットの作成手順については、[VPC の作成](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli)のトピックを参照してください。

セキュリティー・グループが付加された VSI の作成を開始するために、このサンプル CLI コードのコマンドをコピー・アンド・ペーストできます。 このサンプル・コードには、システム応答全体は示されていません。 ご使用の _VPC_、_サブネット_、_イメージ_、および_鍵_ の正しいリソース ID と、正しい_セキュリティー・グループ ID 番号_ を使用して、コマンドを更新する必要があります。

1. 「my_vpc_sg」というセキュリティー・グループを作成します

   ```
   ibmcloud is security-group-create my_vpc_sg $vpc
   ```
   {: pre}

   後で利用できるように、ID を変数に保存します。例えば、以下のように `sg` という名前の変数に保存します。

   ```
   sg=2d364f0a-a870-42c3-a554-000000632953
   ```
   {: pre}

2. SSH、PING、アウトバウンド TCP を許可するルールを追加します

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
   ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
   ibmcloud is security-group-rule-add $sg outbound tcp
   ```
   {: codeblock}

3. 新しく作成したセキュリティー・グループを使用して VSI を作成します

   ```
   ibmcloud is instance-create test-instance $vpc us-south-2 b-4x16 $subnet 1000 \
   --image $image --keys $key --security-groups $sg
   ```
   {: pre}

## コマンド・リストの説明
{: #command-list-cheat-sheet}

セキュリティー・グループで使用可能なすべての VPC CLI コマンドのリストを表示するには、次のように入力します。

```
ibmcloud is help | grep sg
```
{: pre}

セキュリティー・グループとそのメタデータ (ルールを含む) を表示するには、次のように入力します (前の例の場合)。

```
ibmcloud is sg $sg
```
{: pre}

セキュリティー・グループ・ルールの追加に関して、PING インバウンド・ルールをセキュリティー・グループに追加するサンプル・コマンドを次に示します。

```
ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0

```
{: pre}
