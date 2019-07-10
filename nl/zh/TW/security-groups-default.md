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

# 更新預設安全群組
{: #updating-the-default-security-group}


預設安全群組與任何其他安全群組非常類似，只是您無法刪除該群組。

每個 VPC 都具有一個預設安全群組，其規則容許：

* 來自該群組所有成員的入埠資料流量。
* 所有出埠資料流量。

如果您編輯預設安全群組的規則，則那些已編輯的規則隨後會套用至該群組的所有現行及未來伺服器。

容許 ping 及 SSH 的入埠規則不會自動新增至預設安全群組。您可以使用 REST API、`ibmcloud cli` 或使用者介面來修改預設安全群組的規則。

## 範例：使用 CLI 修改預設安全群組規則
{: #example-modifying-the-default-security-group-rules-using-the-cli}

1. 登入 IBM Cloud。

   如果您具有聯合帳戶，請執行下列指令：
   ```
   ibmcloud login -sso
   ```
   {: pre}

   否則，請使用下列指令：

   ```
   ibmcloud login
   ```
   {: pre}

2. 取得 VPC 的預設安全群組 ID 及詳細資料

   執行下列 CLI 指令，以列出所有 VPC：

   ```
   ibmcloud is vpc
   ```
   {: pre}

   預設安全群組名稱會顯示在`預設安全群組`直欄下。請記下它的名稱，以在您列出安全群組時可以找到 `ID`（下一步）。 
   
   現在會列出 VPC 中的所有安全群組：

   ```
   ibmcloud is security-groups
   ```
   {: pre}

   以變數儲存安全群組 ID（用於預設安全群組），供稍後使用。例如，使用變數名稱 `sg`：

   ```
   sg=2d364f0a-a870-42c3-a554-000001162469
   ```
   {: pre}

   若要取得安全群組的詳細資料，請執行下列 CLI 指令：

   ```
   ibmcloud is security-group $sg
   ```
   {: pre}
   
   或者，您也可以插入實際 ID 值來取代變數 `$sg`。

3. 更新預設安全群組 -- 新增容許 SSH 及 PING 的規則

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
   ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
   ```
   {: codeblock}


新增及移除安全群組規則為非同步作業。通常需要 1 到 30 秒才能讓變更生效。
{: note}
