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


# API 和 CLI 命令比较
{: #comparison-of-commands-for-the-api-and-ci}

下面这组表比较了能通过 REST API 或通过 IBM Cloud CLI 调用的类似命令。

## 针对安全组的 API 和 CLI 命令比较
{: #comparing-api-and-cli-commands-for-security-groups}

|描述|API|CLI|
|-------------|-----|-----|
|检索 URL 路径中的标识指定的 VPC 的缺省安全组。|GET /vpcs/{vpc_id}/default_security_group| |
|检索此帐户的所有现有安全组。|GET /security_groups|`ibmcloud is security-groups`, `sgs`|
|检索 URL 路径中的标识指定的单个安全组。|GET /security_groups/{id}|`ibmcloud is security-group`, `sg`|
|基于安全组模板创建新的安全组。模板的构造方式与检索到的安全组相同。模板包含创建新安全组所需的信息。模板可能包含一组可选的安全组规则，这些规则将添加到组中。|POST /security_groups|`ibmcloud is security-group-create`, `sgc`|
|创建服务器实例，包含连接到服务器网络接口的指定现有安全组。|POST /instances（带安全组参数）|`ibmcloud is instance-create`|
|使用安全组补丁对象中提供的信息来更新安全组。安全组补丁对象的构造方式与检索到的安全组相同。它仅包含要更新的信息。|PATCH /security_groups/{id}|`ibmcloud is security-group-update`, `sgu`|
|检索特定安全组的所有规则。|GET /security_groups/{security_group_id}/rules|`ibmcloud is security-group-rules`, `sg-rules`|
|检索 URL 路径中的标识指定的单个安全组规则。|GET /security_groups/{security_group_id}/rules/{id}|`ibmcloud is security-group-rule`, `sg-rule`|
|基于安全组规则模板创建新的安全组规则。规则模板对象的构造方式与检索到的安全组规则相同。它包含创建规则所需的信息。规则将应用于安全组中的所有联网接口。|POST /security_groups/{security_group_id}/rules|`ibmcloud is security-group-rule-add`, `sg-rulec`|
|使用规则补丁对象中提供的信息来更新安全组规则。补丁对象的构造方式与检索到的安全组规则相同。它应仅包含要更新的信息。|PATCH /security_groups/{security_group_id}/rules/{id}| |
|删除安全组规则。此删除操作不会终止该规则允许的任何现有连接。此操作不可撤销。|DELETE /security_groups/{security_group_id}/rules/{id}|`ibmcloud is security-group-rule-delete`, `sg-ruled`|
|删除安全组。如果某个安全组是 VPC 的缺省安全组，或者如果其他任何安全组包含将该安全组作为远程项引用的规则，那么无法删除该安全组。此操作不可撤销。|DELETE /security_groups/{id}|`ibmcloud is security-group-delete`, `sgd`|
|将服务器的现有网络接口添加到现有安全组。此功能将组的规则应用于该接口，允许与接口之间建立符合这些规则的新进出连接。<br /><br />此外，现在已向此网络接口授予由任何远程组中引用此组的规则所指定的访问权。<br /><br />请参阅下面的“限制”。|PUT /security_groups/{security_group_id}/network_interfaces/{id}|`ibmcloud is security-group-network-interface-add`, `sg-nica`|
|从安全组中除去服务器的网络接口。该组的规则不再用于允许在该接口上建立新的网络连接。<br /><br />此外，不再向此网络接口授予由任何远程组中引用此组的规则所指定的访问权。<br /><br />此组的成员资格允许的现有连接不会结束。|DELETE /security_groups/{security_group_id}/network_interfaces/{id}|`ibmcloud is security-group-network-interface-remove`, `sg-nicd`|
|检索与安全组关联的所有网络接口。这些是应用了该组中规则的网络接口。|GET /security_groups/{security_group_id}/network_interfaces|`ibmcloud is security-group-network-interfaces`, `sg-nics`|
|检索 URL 路径中的标识指定的单个网络接口。网络接口必须是安全组的现有成员。|GET /security_groups/{security_group_id}/network_interfaces/{id}|`ibmcloud is security-group-network-interface`, `sg-nic`|



## 针对 ACL 的 API 和 CLI 命令比较
{: #comparing-api-and-cli-commands-for-acls}

|描述|API|CLI|
|-------------|-----|-----|
|创建 ACL|`POST /network_acls`|`ibmcloud is network-acl-create`|
|检索所有 ACL|`GET /network_acls`|`ibmcloud is network-acls`|
|检索特定 ACL|`GET /network_acls/{id}`|`ibmcloud is network-acl`|
|更新特定 ACL|`PATCH /network_acls/{id}`|`ibmcloud is network-acl-update`|
|删除特定 ACL|`DELETE /network_acls/{id}`|`ibmcloud is network-acl-delete`|
|创建 ACL 规则|`POST /network_acls/{network_acl_id}/rules`|`ibmcloud is network-acl-rule-add`|
|检索 ACL 的所有规则|`GET /network_acls/{network_acl_id}/rules`|`ibmcloud is network-acl-rules`|
|检索特定 ACL 规则|`GET /network_acls/{network_acl_id}/rules/{id}`|`ibmcloud is network-acl-rule`|
|更新特定 ACL 规则|`PATCH /network_acls/{network_acl_id}/rules/{id}`|`ibmcloud is network-acl-rule-update`|
|删除特定 ACL 规则|`DELETE /network_acls/{network_acl_id}/rules/{id}`|`ibmcloud is network-acl-rule-delete`|
|将 ACL 连接到子网|`PUT /subnets/{id}/network_acl`|`ibmcloud is subnet-network-acl-attach`|
