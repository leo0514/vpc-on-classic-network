---

copyright:
  years: 2018, 2019

lastupdated: "2019-05-14"

keywords: API, CLI, commands, comparison, security groups, ACL

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}


# API と CLI のコマンドの比較
{: #comparison-of-commands-for-the-api-and-ci}

以下の一連の表により、REST API を使用して呼び出せるコマンドと、IBM Cloud CLI を使用して呼び出せる類似したコマンドを比較できます。

## セキュリティー・グループに関する API コマンドと CLI コマンドの比較
{: #comparing-api-and-cli-commands-for-security-groups}

| 説明 | API | CLI |
|-------------|-----|-----|
|URL パスの中に ID で指定した VPC のデフォルトのセキュリティー・グループを取得します。 | GET /vpcs/{vpc_id}/default_security_group| |
|このアカウントの既存のセキュリティー・グループをすべて取得します。|GET /security_groups| `ibmcloud is security-groups`、`sgs`|
|URL パスの中に ID で指定した単一のセキュリティー・グループを取得します。 | GET /security_groups/{id}| `ibmcloud is security-group`、`sg`|
|セキュリティー・グループのテンプレートからセキュリティー・グループを新規作成します。 テンプレートは、取得されるセキュリティー・グループと同じように構造化されています。 テンプレートには、セキュリティー・グループを新規作成するために必要な情報が含まれています。 オプションでセキュリティー・グループのルールの配列をテンプレートに含めることができます。そうすると、それらのルールがグループに追加されます。 | POST /security_groups| `ibmcloud is security-group-create`、`sgc`|
|サーバー・インスタンスを作成します。指定した既存のセキュリティー・グループがそのサーバーのネットワーク・インターフェースに付加されます。 |POST /instances (セキュリティー・グループ・パラメーターを指定)| `ibmcloud is instance-create` |
|セキュリティー・グループのパッチ・オブジェクトに指定されている情報を使用してセキュリティー・グループを更新します。 セキュリティー・グループのパッチ・オブジェクトは、取得されるセキュリティー・グループと同じように構造化されています。 このオブジェクトには、更新する情報のみが含まれています。 | PATCH /security_groups/{id}| `ibmcloud is security-group-update`、`sgu`|
|特定のセキュリティー・グループのすべてのルールを取得します。 | GET /security_groups/{security_group_id}/rules| `ibmcloud is security-group-rules`、`sg-rules`|
|URL パスの中に ID で指定した単一のセキュリティー・グループのルールを取得します。 | GET /security_groups/{security_group_id}/rules/{id}| `ibmcloud is security-group-rule`、`sg-rule`|
|セキュリティー・グループ・ルールのテンプレートからセキュリティー・グループ・ルールを新規作成します。 このルール・テンプレート・オブジェクトは、取得されるセキュリティー・グループ・ルールと同じように構造化されています。 このオブジェクトには、ルールを作成するために必要な情報が含まれています。 このルールは、セキュリティー・グループ内のすべてのネットワーキング・インターフェースに適用されます。 | POST /security_groups/{security_group_id}/rules| `ibmcloud is security-group-rule-add`、`sg-rulec`|
|ルールのパッチ・オブジェクトに指定した情報を使用してセキュリティー・グループのルールを更新します。 このパッチ・オブジェクトは、取得されるセキュリティー・グループのルールと同じように構造化されています。 このオブジェクトには、更新する情報のみを含める必要があります。 | PATCH /security_groups/{security_group_id}/rules/{id}| |
|セキュリティー・グループのルールを削除します。 削除しても、このルールで許可された既存の接続は終了しません。 この操作は元に戻せません。 | DELETE /security_groups/{security_group_id}/rules/{id}| `ibmcloud is security-group-rule-delete`、`sg-ruled`|
|セキュリティー・グループを削除します。 VPC のデフォルトのセキュリティー・グループは削除できません。また、他のセキュリティー・グループのルールでリモートとして参照されているセキュリティー・グループも削除できません。 この操作は元に戻せません。 | DELETE /security_groups/{id}| `ibmcloud is security-group-delete`、`sgd`|
|サーバーの既存のネットワーク・インターフェースを既存のセキュリティー・グループに追加します。 この機能を使用すると、グループのルールがそのインターフェースに適用されるので、そのインターフェースとの間でそれらのルールに準拠した新しい接続を行うことができます。<br /><br />また、そのネットワーク・インターフェースには、このグループを参照しているリモート・グループのルールで指定されているアクセス権が付与されます。<br /><br />後述の制限事項を参照してください。  | PUT /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface-add`、`sg-nica`|
|セキュリティー・グループからサーバーのネットワーク・インターフェースを削除します。 そのインターフェースでは、このグループのルールが新規ネットワーク接続の許可に使用されなくなります。 <br /><br />また、そのネットワーク・インターフェースには、このグループを参照しているリモート・グループのルールで指定されているアクセス権が付与されなくなります。 <br /><br />このグループのメンバーシップによって許可された既存の接続は終了しません。| DELETE /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface-remove`、`sg-nicd`|
|セキュリティー・グループに関連付けられているネットワーク・インターフェースをすべて取得します。 つまり、グループのルールが適用されるネットワーク・インターフェースです。 | GET /security_groups/{security_group_id}/network_interfaces| `ibmcloud is security-group-network-interfaces`、`sg-nics`|
|URL パスの中に ID で指定した単一のネットワーク・インターフェースを取得します。 このネットワーク・インターフェースは、セキュリティー・グループの既存のメンバーでなければなりません。 | GET /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface`、`sg-nic`|



## ACL に関する API コマンドと CLI コマンドの比較
{: #comparing-api-and-cli-commands-for-acls}

| 説明 | API | CLI |
|-------------|-----|-----|
|ACL の作成 |`POST /network_acls` | `ibmcloud is network-acl-create`|
|すべての ACL の取得 |`GET /network_acls` | `ibmcloud is network-acls`|
|特定の ACL の取得| `GET /network_acls/{id}`| `ibmcloud is network-acl`|
|特定の ACL の更新| `PATCH /network_acls/{id}`| `ibmcloud is network-acl-update`|
|特定の ACL の削除| `DELETE /network_acls/{id}`| `ibmcloud is network-acl-delete`|
|ACL ルールの作成| `POST /network_acls/{network_acl_id}/rules`| `ibmcloud is network-acl-rule-add`|
|ACL のすべてのルールの取得| `GET /network_acls/{network_acl_id}/rules`| `ibmcloud is network-acl-rules`|
|特定の ACL ルールの取得| `GET /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule`|
|特定の ACL ルールの更新| `PATCH /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule-update`|
|特定の ACL ルールの削除| `DELETE /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule-delete`|
|サブネットへの ACL の付加|`PUT /subnets/{id}/network_acl`| `ibmcloud is subnet-network-acl-attach` |
