---

copyright:
  years: 2018, 2019

lastupdated: "2019-05-14"

keywords: default, security group, asynchronous, rules

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

# デフォルトのセキュリティー・グループの更新
{: #updating-the-default-security-group}


デフォルトのセキュリティー・グループは、他のセキュリティー・グループと非常によく似ていますが、削除できない点が異なります。

各 VPC にはデフォルトのセキュリティー・グループがあり、以下のトラフィックを許可するルールが含まれています。

* そのグループのすべてのメンバーからのインバウンド・トラフィック。
* すべてのアウトバウンド・トラフィック。

デフォルトのセキュリティー・グループのルールを編集すると、グループ内の現在および将来のすべてのサーバーにこれらの編集されたルールが適用されます。

ping と SSH を許可するインバウンド・ルールは、デフォルトのセキュリティー・グループに自動的には追加されません。 REST API、`ibmcloud cli`、または UI を使用して、デフォルトのセキュリティー・グループのルールを変更できます。

## 例: CLI を使用してデフォルトのセキュリティー・グループのルールを変更する
{: #example-modifying-the-default-security-group-rules-using-the-cli}

1. IBM Cloud にログインします。

   統合されたアカウントがある場合は、以下のようにします。
   ```
   ibmcloud login -sso
   ```
   {: pre}

   ない場合は、以下のコマンドを使用します。

   ```
   ibmcloud login
   ```
   {: pre}

2. VPC のデフォルトのセキュリティー・グループの ID と詳細を表示します

   以下の CLI コマンドを実行して、すべての VPC をリストします。

   ```
   ibmcloud is vpc
   ```
   {: pre}

   デフォルトのセキュリティー・グループの名前が、`Default Security Group` 列の下に表示されます。 (次で) セキュリティー・グループをリストして `ID` を調べるので、この名前をメモしておいてください。 
   
   次は、VPC 内のセキュリティー・グループをすべてリストします。

   ```
   ibmcloud is security-groups
   ```
   {: pre}

   後で使用できるように、(デフォルトのセキュリティー・グループの) セキュリティー・グループ ID を変数の中に保存します。 例えば、変数名 `sg` を使用します。

   ```
   sg=2d364f0a-a870-42c3-a554-000001162469
   ```
   {: pre}

   セキュリティー・グループの詳細を表示するために、以下の CLI コマンドを実行します。

   ```
   ibmcloud is security-group $sg
   ```
   {: pre}
   
   また、変数 `$sg` の代わりに実際の ID を挿入することもできます。

3. デフォルトのセキュリティー・グループを更新します -- SSH と PING を許可するルールを追加します

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
   ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
   ```
   {: codeblock}


セキュリティー・グループ・ルールの追加と削除は、非同期の操作です。 変更が有効になるには、通常 1 秒から 30 秒かかります。
{: note}
