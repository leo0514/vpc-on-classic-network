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

# 使用 CLI 設定安全群組
{: #setting-up-security-groups-using-the-cli}

在下面範例中，您將使用指令行介面 (CLI) 來建立已啟用安全群組的 VSI。其情境範例如下。

![IBM VPC 的安全群組](images/security-groups-schematic.svg "IBM VPC 的安全群組"){: caption="圖：IBM VPC 的安全群組" caption-side="top"}

請注意，圖中名為 **SG4** 的 VSI，除了它內部 VPC 位址 `10.0.0.5` 之外，還獲指派浮動 IP `169.60.208.144`；因此，它可以與公用網際網路進行交談。指派給 VSI **SG4** 的安全群組名稱為 "demosg"。

名稱為 **SG8** 的 VSI 僅供 VPC 內部使用，具有專用 IP 位址。指派給 VSI **SG8** 的安全群組名稱為 "my_vpc_sg"。這兩個 VSI 都存在於名為 `sgvpc` 的 VPC 內，同時位於相同的子網路 `10.0.0.0/28`，因此它們可以彼此通訊。

## 建立已連接安全群組的 VSI 的步驟
{: #steps-for-creating-a-vsi-with-a-security-group-attached}

"my_vpc_sg" 的安全群組規則將包括 SSH、PING 及出埠 TCP 的基本功能。

請注意，您必須先使用 `ibmcloud is sgc` 指令建立安全群組，然後建立 VSI 來包括這個新建的安全群組。

此程式碼範例會跳過一些步驟，因此這裡您可以找到相關資訊：

 * 如需建立 VPC 及子網路的指示，請參考[建立 VPC](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli) 主題。

您可以複製並貼上這個範例 CLI 程式碼中的指令，以開始建立已連接安全群組的 VSI。此範例程式碼未完整顯示系統回應。您必須使用 _VPC_、_subnet_、_image_ 和 _key_ 的正確資源 ID 以及正確的_安全群組 ID 號碼_，來更新指令。

1. 建立一個稱為 "my_vpc_sg" 的安全群組

   ```
   ibmcloud is security-group-create my_vpc_sg $vpc
   ```
   {: pre}

   將 ID 儲存在變數中，以供稍後使用，例如，儲存在名為 `sg` 的變數中：

   ```
   sg=2d364f0a-a870-42c3-a554-000000632953
   ```
   {: pre}

2. 新增容許 SSH、PING 及出埠 TCP 的規則

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
   ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
   ibmcloud is security-group-rule-add $sg outbound tcp
   ```
   {: codeblock}

3. 建立含有新建安全群組的 VSI

   ```
   ibmcloud is instance-create test-instance $vpc us-south-2 b-4x16 $subnet 1000 \ 
   --image $image --keys $key --security-groups $sg
   ```
   {: pre}

## 指令清單提要
{: #command-list-cheat-sheet}

如需安全群組可用 VPC CLI 指令的完整清單，請鍵入：

```
ibmcloud is help | grep sg
```
{: pre}

若要查看安全群組及其 meta 資料（包括規則），請鍵入（針對前一個範例）：

```
ibmcloud is sg $sg
```
{: pre}

若要新增安全群組規則，以下的範例指令會將 PING 入埠規則新增至安全群組：

```
ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0

```
{: pre}
