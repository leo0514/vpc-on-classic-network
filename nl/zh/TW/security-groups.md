---

copyright:
  years: 2019

lastupdated: "2019-05-20"

keywords: security groups, traffic, firewall, stateful, filtering

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

# 使用安全群組
{: #using-security-groups}
[comment]: # (鏈結的說明主題)

安全群組提供一種方便的方式，讓您根據 IP 位址來套用規則，以建立虛擬伺服器實例 (VSI) 的每個網路介面的過濾。當您建立新的安全群組資源時，要更新它來建立您想要的網路資料流量型樣。

如需關於安全群組與 ACL 性質的比較，請參閱[比較表格](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-compare-security-groups-and-access-control-lists)。

依預設，安全群組會拒絕所有資料流量。當規則新增至安全群組時，它會定義安全群組允許的資料流量。

規則是_有狀態_，這表示會自動允許反向資料流量以回應容許的資料流量。因此，例如容許埠 80 的入埠 TCP 資料流量的規則也容許回覆埠 80 的出埠 TCP 資料流量回到原始主機，而不需要其他規則。

安全群組的範圍限定為單一 VPC。此範圍限定意味著安全群組_只能_ 連接至相同 VPC 內的 VSI 網路介面。

在未指定任何安全群組的情況下建立 VSI 時，VSI 的主要網路介面會被放到該 VSI 之 VPC 的_預設_ 安全群組。這個 {{site.data.keyword.cloud}} VPC 版本已定義容許特定資料流量的預設安全群組。如需相關資訊，請參閱[更新預設安全群組](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-updating-the-default-security-group)。

您可以使用 REST API、CLI 或使用者介面來設定安全群組：

* [使用 API 設定安全群組](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-apis)
* [使用 CLI 設定安全群組](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-cli)
* [使用使用者介面設定安全群組](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#configuring-the-security-group-for-the-instance)
