---
copyright:
  years: 2019
lastupdated: "2019-05-14"

keywords: security groups, traffic, firewall, stateful, filtering, access, control, list, ACL, stateless, traffic, resource

subcollection: vpc-on-classic-network
---

# 比較安全群組與存取控制清單
{: #compare-security-groups-and-access-control-lists}

使用您所指定的規則，安全群組及存取控制清單能提供多種控制 {{site.data.keyword.cloud}} VPC 中子網路及實例之間資料流量的方法。

下表彙總安全群組與 ACL 之間的一些重要差異：

|  | 安全群組 | ACL    |
|-------------|-----------------|---------|
| 控制層次  | VSI 實例    | 子網路  |
| 狀態   | 有狀態 - 一旦允許入埠連線，就容許其回覆 | 無狀態 - 必須同時明確容許入埠及出埠連線 |
| 支援的規則 | 僅使用容許規則 | 使用容許及拒絕規則|
| 如何套用規則 | 考量全部規則 | 依序處理規則 |
| 與關聯資源的關係 | 一個實例可以與多個安全群組相關聯| 多個子網路可以與相同的 ACL 相關聯|
