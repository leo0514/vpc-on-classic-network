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

# 更新缺省安全组
{: #updating-the-default-security-group}


缺省安全组与其他任何安全组都十分相似，只是不能将其删除。

每个 VPC 都有一个缺省安全组，包含的规则允许：

* 组中所有成员的入站流量。
* 所有出站流量。

如果编辑缺省安全组的规则，那么这些编辑后的规则将应用于组中的所有当前和未来的服务器。

用于允许 ping 和 SSH 的入站规则不会自动添加到缺省安全组。您可以使用 REST API、`ibmcloud cli` 或 UI 来修改缺省安全组的规则。

## 示例：使用 CLI 修改缺省安全组规则
{: #example-modifying-the-default-security-group-rules-using-the-cli}

1. 登录到 IBM Cloud。

   如果您有联合帐户：
   ```
   ibmcloud login -sso
   ```
   {: pre}

   如果没有，请使用以下命令：

   ```
   ibmcloud login
   ```
   {: pre}

2. 获取 VPC 的缺省安全组标识和详细信息

   运行以下 CLI 命令以列出所有 VPC：

   ```
   ibmcloud is vpc
   ```
   {: pre}

   缺省安全组名称会显示在 `Default Security Group` 列下。请记下该名称，以便在列出安全组（下一个命令）时，可以找到相应 `ID`。 
   
   现在列出 VPC 中的所有安全组：

   ```
   ibmcloud is security-groups
   ```
   {: pre}

   在变量中保存安全组标识（缺省安全组的标识），以便稍后可以使用。例如，使用变量名称 `sg`：

   ```
   sg=2d364f0a-a870-42c3-a554-000001162469
   ```
   {: pre}

   要获取有关安全组的详细信息，请运行以下 CLI 命令：

   ```
   ibmcloud is security-group $sg
   ```
   {: pre}
   
   或者，可以插入实际标识值以代替变量 `$sg`。

3. 更新缺省安全组 - 添加允许 SSH 和 PING 的规则

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
   ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
   ```
   {: codeblock}


添加和除去安全组规则是异步操作。更改通常需要 1 到 30 秒才能生效。
{: note}
