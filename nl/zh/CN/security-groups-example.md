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

# 使用 CLI 设置安全组
{: #setting-up-security-groups-using-the-cli}

在以下示例中，将使用命令行界面 (CLI) 来创建启用了安全组的 VSI。下图说明了此场景。

![Security Groups for IBM VPC](images/security-groups-schematic.svg "Security Groups for IBM VPC")

请注意，在图中，名为 **SG4** 的 VSI 除了有内部 VPC 地址 `10.0.0.5` 外，还分配有浮动 IP `169.60.208.144`；因此，该 VSI 可以与公用因特网进行对话。分配给 VSI **SG4** 的安全组名为“demosg”。

名为 **SG8** 的 VSI 仅供 VPC 内部使用，并带有专用 IP 地址。分配给 VSI **SG8** 的安全组名为“my_vpc_sg”。这两个 VSI 都存在于名为 `sgvpc` 的 VPC 中，而且还位于同一子网 `10.0.0.0/28` 中，以便它们可以相互通信。

## 用于创建连接有安全组的 VSI 的步骤
{: #steps-for-creating-a-vsi-with-a-security-group-attached}

“my_vpc_sg”的安全组规则将包括 SSH、PING 和出站 TCP 的基本功能。

请注意，必须先使用 `ibmcloud is sgc` 命令创建安全组，然后创建将包含此新创建的安全组的 VSI。

此示例代码将跳过几个步骤，但在此处可以找到更多信息：

 * 创建 VPC 和子网的指示信息在[创建 VPC](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli) 主题中提供。

可以复制并粘贴此示例 CLI 代码中的命令，以开始创建连接了安全组的 VSI。此样本代码中未完全显示系统响应。您将需要使用 _VPC_、_子网_、_映像_和_密钥_的正确资源标识以及正确的_安全组标识号_来更新命令。

1. 创建名为“my_vpc_sg”的安全组

   ```
   ibmcloud is security-group-create my_vpc_sg $vpc
   ```
   {: pre}

   将标识保存在变量中，以便稍后可以使用，例如保存在名为 `sg` 的变量中：

   ```
   sg=2d364f0a-a870-42c3-a554-000000632953
   ```
   {: pre}

2. 添加允许 SSH、PING 和出站 TCP 的规则

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
   ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
   ibmcloud is security-group-rule-add $sg outbound tcp
   ```
   {: codeblock}

3. 使用新创建的安全组来创建 VSI

   ```
   ibmcloud is instance-create test-instance $vpc us-south-2 b-4x16 $subnet 1000 \ 
   --image $image --keys $key --security-groups $sg
   ```
   {: pre}

## 命令列表备忘单
{: #command-list-cheat-sheet}

要获取安全组的可用 VPC CLI 命令的完整列表，请输入以下命令：

```
ibmcloud is help | grep sg
```
{: pre}

要查看安全组及其元数据（包括规则），您应输入以下命令（对于前一个示例）：

```
ibmcloud is sg $sg
```
{: pre}

要添加安全组规则，下面是向安全组添加 PING 入站规则的示例命令：

```
ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0

```
{: pre}
