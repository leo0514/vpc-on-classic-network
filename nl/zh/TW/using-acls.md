---

copyright:
  years: 2019

lastupdated: "2019-06-11"

keywords: ACLs, network, CLI, example, tutorial, firewall, subnet, inbound, outbound, rule

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}


# 設定網路 ACL
{: #setting-up-network-acls}
[comment]: # (鏈結的說明主題)

透過 {{site.data.keyword.cloud}} Virtual Private Cloud 中提供的「存取控制清單 (ACL)」功能，您可以控制與雲端上重要商業工作負載相關的所有送入及送出資料流量。ACL 是內建的虛擬防火牆，類似於安全群組。相對於安全群組，ACL 規則會控制進出_子網路_ 的資料流量，而不是進出_實例_ 的資料流量。

如需關於安全群組與 ACL 性質的比較，請參閱[比較表格](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-compare-security-groups-and-access-control-lists)。

本文件中提供的範例顯示如何使用 CLI 在 VPC 中建立網路 ACL，以保護子網路。

## API 及 CLI 可用於 ACL
{: #apis-and-clis-are-available-for-acls}

您可以透過 API、CLI 或使用者介面來設定及管理 ACL。

* 如需每個 API 的參數、要求內文及回應詳細資料，請參閱 [API 參考資料](https://{DomainName}/apidocs/vpc-on-classic)。

* 請參閱這個 [CLI 範例](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli)，以取得指令詳細資料、安裝 CLI 外掛程式的步驟及使用 VPC CLI 的必要步驟。

* 如需如何在 {{site.data.keyword.cloud_notm}} 主控台中設定 ACL 的相關資訊，請參閱[使用使用者介面配置 ACL](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#configuring-the-acl)。

ACL 規則可控制進出_子網路_的資料流量。VPC API、CLI 及使用者介面似乎支援自訂入埠規則的**目的地 IP 範圍**及出埠規則的**來源 IP 範圍**，不過，這些位址不可自訂。**入埠 ACL 規則的目的地 IP 範圍及出埠 ACL 規則的來源 IP 範圍，都與連接的子網路**相關聯。因此，實際上，應該使用 **0.0.0.0/0** 作為入埠規則的目的地 IP，以及出埠規則的來源 IP。
{: note}

## 使用 ACL 及 ACL 規則
{: #working-with-acls-and-acl-rules}

若要讓 ACL 生效，您要建立規則來決定如何處理入埠及出埠網路資料流量。您可以建立多個入埠和出埠規則。請參閱[配額](/docs/vpc-on-classic?topic=vpc-on-classic-quotas)，以取得您可以建立多少個規則的具體相關資訊。

* 使用入埠規則，您可以容許或拒絕來源 IP 範圍（具有指定的通訊協定及埠）的資料流量。目的地 IP 範圍則是由連接的子網路所決定。
* 使用出埠規則，您可以容許或拒絕目的地 IP 範圍（具有指定的通訊協定及埠）的資料流量。來源 IP 範圍則是由連接的子網路所決定。
* 將依序考量 ACL 規則。只有不符合優先順序號碼（例如 1）較低的規則時，才會評估優先順序號碼較高（例如 2）的規則。
* 入埠規則與出埠規則是分開的。
* 目前支援的通訊協定為 TCP、UDP 及 ICMP。您也可以使用 **all** 選項來指定_全部_ 或_其他_ 通訊協定（如果已指定具有較高優先順序的規則）。

如需在 ACL 規則中使用 ICMP、TCP 及 UDP 通訊協定的相關資訊，請參閱[瞭解網際網路通訊協定](/docs/infrastructure/network-infrastructure?topic=network-infrastructure-understanding-internet-communication-protocols)。

### 將 ACL 連接至子網路
{: #attaching-an-acl-to-a-subnet}

將 ACL 連接至子網路時，您有兩個選項：

* 您可以建立新的子網路，並指定要連接的 ACL。如果您未指定 ACL，則會連接預設網路 ACL。預設 ACL 容許所有預定用於此子網路的入埠資料流量，以及來自這個子網路的所有出埠資料流量。
* 您可以將 ACL 連接至現有的子網路。如果有另一個 ACL 已連接至這個子網路，則要先分離此 ACL，再連接新的 ACL。

## ACL 展示範例
{: #acl-demo-example}

在下面範例中，您將使用指令行介面 (CLI) 來建立兩個 ACL，並建立它們與兩個子網路的關聯。其情境範例如下：

![ACL 情境範例](images/vpc-acls.png)

如圖所示，您有兩個負責處理來自網際網路的要求的 Web 伺服器，以及兩個您想要隱藏而不公開的後端伺服器。在此範例中，您將伺服器分別放置在兩個個別的子網路 10.10.10.0/24 和 10.10.20.0/24，且您需要容許 Web 伺服器與後端伺服器交換資料。此外，您還想要容許來自後端伺服器的有限出埠資料流量。

### 範例規則
{: #acl-example-rules}

下面的範例規則顯示如何針對基本情境設定 ACL 規則（如先前所述）。

最佳作法是建議您將精細規則的優先順序設定為高於粗略規則。例如，如果您的規則是封鎖來自子網路 10.10.30.0/24 的所有資料流量，且其符合較高優先順序，則絕不會套用容許來自 10.10.30.5 之資料流量的較低優先順序的精細規則。
{:note}

**ACL-1 範例規則**：

| 入埠/出埠| 容許/拒絕 | 來源 IP | 目的地 IP | 通訊協定 | 埠 | 說明|
|--------------|-----------|------|------|------|------------------|-------|
| 入埠 | 容許 | 0.0.0.0/0 | 0.0.0.0/0 | TCP| 80 | 容許來自網際網路的 HTTP 資料流量|
| 入埠 | 容許 | 0.0.0.0/0 | 0.0.0.0/0 | TCP | 443 | 容許來自網際網路的 HTTPS 資料流量|
| 入埠 | 容許 | 10.10.20.0/24 | 0.0.0.0/0 |all|all| 容許其中放置後端伺服器的子網路 10.10.20.0/24 的所有入埠資料流量|
| 入埠 | 拒絕| 0.0.0.0/0| 0.0.0.0/0 |all| all| 拒絕所有其他入埠資料流量|
| 出埠 | 容許 | 0.0.0.0/0 | 0.0.0.0/0 | TCP|80 | 容許傳送至網際網路的 HTTP 資料流量|
| 出埠 | 容許 | 0.0.0.0/0 | 0.0.0.0/0 | TCP|443 | 容許傳送至網際網路的 HTTPS 資料流量|
| 出埠 | 容許 | 0.0.0.0/0 | 10.10.20.0/24 | all|all| 容許其中放置後端伺服器的子網路 10.10.20.0/24 的所有出埠資料流量|
| 出埠 | 拒絕| 0.0.0.0/0 | 0.0.0.0/0|all|all| 拒絕所有其他出埠資料流量|


**ACL-2 範例規則**：

| 入埠/出埠| 容許/拒絕 | 來源 IP | 目的地 IP | 通訊協定 | 埠 | 說明|
|--------------|-----------|------|------|------|------------------|--------|
| 入埠 | 容許 | 10.10.10.0/24 | 0.0.0.0/0 |all| all| 容許其中放置 Web 伺服器的子網路 10.10.10.0/24 的所有入埠資料流量|
| 入埠 | 拒絕| 0.0.0.0/0| 0.0.0.0/0 |all|all| 拒絕所有其他入埠資料流量|
| 出埠 | 容許 | 0.0.0.0/0 | 0.0.0.0/0 | TCP| 80 | 容許傳送至網際網路的 HTTP 資料流量|
| 出埠 | 容許 | 0.0.0.0/0 | 0.0.0.0/0 | TCP| 443 | 容許傳送至網際網路的 HTTPS 資料流量|
| 出埠 | 容許 | 0.0.0.0/0 | 10.10.10.0/24 |all|all| 容許其中放置 Web 伺服器的子網路 10.10.10.0/24 的所有出埠資料流量|
| 出埠 | 拒絕| 0.0.0.0/0 | 0.0.0.0/0|all|all| 拒絕所有其他出埠資料流量|

此範例僅說明一般情況。在您的情境中，您也可以對資料流量有更精細的控制：

* 您可能有一位網路管理者，他需要從遠端網路存取 10.10.10.0/24 子網路，才能進行作業。在該情況下，您需要容許 SSH 資料流量從網際網路傳送至這個子網路。
* 您可能想要縮小兩個子網路之間容許的通訊協定範圍。

### 範例步驟
{: #acl-example-steps}

下列範例步驟會跳過使用 CLI 來建立 VPC（必須先完成）的必要步驟。如需相關資訊，請參閱[使用 CLI 建立 VPC](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli)。


#### 步驟 1. 建立 ACL
{: #step-1-create-the-acls}

以下是您可以用來建立兩個 ACL 的 CLI 指令，名為 `my_web_subnet_acl` 和 `my_backend_subnet_acl`：

```
ibmcloud is network-acl-create my_web_subnet_acl
ibmcloud is network-acl-create my_backend_subnet_acl
```
{: codeblock}

回應包括新建立的 ACL ID。儲存這兩個 ACL 的 ID，以供稍後的指令使用。您可以使用變數 `webacl` 和 `bkacl`，如下所示：

```
webacl="ba9e785a-3e10-418a-811c-56cfe5669676"
bkacl="a4e28308-8ee7-46ab-8108-9f881f22bdbf"
```
{: codeblock}

#### 步驟 2. 擷取預設 ACL 規則
{: #step-2-retrieve-the-default-acl-rules}

新增新規則之前，請先擷取預設入埠及出埠 ACL 規則，讓您可以在它們之前插入新規則。

```
ibmcloud is network-acl-rules $webacl
ibmcloud is network-acl-rules $bkacl
```
{: codeblock}

此回應顯示預設入埠及出埠規則，其容許所有通訊協定中的所有 IPv4 資料流量。

```
Getting rules of network acl ba9e785a-3e10-418a-811c-56cfe5669676 under account Demo Account as user demouser...

inbound
ID                                     Name                                                          Action   IPv*   Protocol   Source      Destination   Created
e2b30627-1a1d-447b-859f-ac9431986b6f   allow-all-inbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6   allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago

outbound
ID                                     Name                                                         Action   IPv*   Protocol   Source      Destination   Created
173a3492-0544-472e-91c0-7828cbcb62d4   allow-all-outbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6   allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago
```
{: screen}

將這兩個 ACL 規則的 ID 儲存為變數，您可以在稍後的指令中使用它們。例如，您可以使用名為 `inrule` 和 `outrule` 的變數：

```
inrule="e2b30627-1a1d-447b-859f-ac9431986b6f"
outrule="173a3492-0544-472e-91c0-7828cbcb62d4"
```
{: codeblock}

#### 步驟 3. 如所述新增 ACL 規則
{: #step-3-add-new-acl-rules-as-decribed}

本節指出先新增入埠規則，然後再新增出埠規則。

在預設入埠規則之前插入新的入埠規則。

```
ibmcloud is network-acl-rule-add my_web_acl_rule200 $webacl deny inbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule $inrule
```
{: pre}

在每個步驟，將 ACL 規則的 ID 儲存為變數，它將用於後面的指令。例如，您可以使用變數名稱 `acl200`：

```
acl200="90930627-1a1d-447b-859f-ac9431986b6f"
```
{: pre}

現在，即會將規則新增至 `acl200`：

```
ibmcloud is network-acl-rule-add my_web_acl_rule100 $webacl allow inbound all 10.10.20.0/24 0.0.0.0/0 \
--before-rule $acl200
```
{: pre}

新增更多規則直到您的 ACL 設定完成為止，將每個 ID 儲存為變數。

```
acl100="78340627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule20 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 443 --port-min 443 --before-rule $acl100
acl20="32450627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule10 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 80 --port-min 80 --before-rule $acl20
```
{: codeblock}

在預設出埠規則之前插入新的出埠規則。

```
ibmcloud is network-acl-rule-add my_web_acl_rule200e $webacl deny outbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule $outrule
acl200e="11110627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule100e $webacl allow outbound all 0.0.0.0/0 10.10.20.0/24 \
--before-rule $acl200e
acl100e="22220627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule20e $webacl allow outbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 443 --port-min 443 --before-rule $acl100e
acl20e="33330627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule10e $webacl allow outbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 80 --port-min 80 --before-rule $acl20e
```
{: codeblock}

#### 步驟 4. 建立兩個具有新建的 ACL 的子網路
{: #step-4-create-the-two-subnets-with-the-newly-created-acl}

您將建立兩個子網路，讓每個 ACL 與其中一個新的子網路相關聯。

```
ibmcloud is subnet-create my_web_subnet my_VPC my_region --ipv4_cidr_block 10.10.10.0/24 \
--generation gc --network-acl $webacl
ibmcloud is subnet-create my_backend_subnet my_VPC my_region --ipv4_cidr_block 10.10.20.0/24 \
--generation gc --network-acl $bkacl
```
{: codeblock}


## 指令清單提要
{: #acl-cli-command-list-cheat-sheet}

若要顯示 ACL 可用 VPC CLI 指令的完整清單，請執行下列指令：

```
ibmcloud is network-acls
```
{: pre}

查看 ACL 及其 meta 資料，包括規則在內：

```
ibmcloud is network-acl $webacl
```
{: pre}

若要取得預設入埠 ACL 規則，請執行下列指令：

```
ibmcloud is network-acl-rules $webacl --direction inbound
```
{: pre}

## 入埠 `ping` 規則範例
{: #acl-example-inbound-ping-rule}

若要新增 ACL 規則，以下的範例指令會在預設入埠規則之前新增 `ping` 入埠規則：

```
ibmcloud is network-acl-rule-add --action allow --direction inbound --protocol icmp --icmp-type 8 --icmp-code --before-rule-name <default_acl_rule_name> <acl_name> <new_acl_rule_name>
```
{: pre}
