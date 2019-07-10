---
copyright:
  years: 2019
lastupdated: "2019-05-14"

keywords: security groups, traffic, firewall, stateful, filtering, access, control, list, ACL, stateless, traffic, resource

subcollection: vpc-on-classic-network
---

# セキュリティー・グループとアクセス制御リストの比較
{: #compare-security-groups-and-access-control-lists}

セキュリティー・グループとアクセス制御リストを使用することにより、指定したルールに基づいて {{site.data.keyword.cloud}} VPC 内のサブネットとインスタンスにまたがってトラフィックを制御できます。

次の表に、セキュリティー・グループと ACL の主な相違点を要約します。

|  | セキュリティー・グループ | ACL    |
|-------------|-----------------|---------|
| 制御レベル  | VSI インスタンス    | サブネット  |
| 状態   | ステートフル - インバウンド接続が許可されると応答が許可される | ステートレス - インバウンド接続とアウトバウンド接続の両方が明示的に許可される必要がある |
| サポートされているルール | 許可ルールのみ使用 | 許可ルールと拒否ルールを使用|
| ルールの適用方法 | すべてのルールが評価される | ルールが順番に処理される |
| 関連リソースとの関係 | 1 つのインスタンスに複数のセキュリティー・グループを関連付けることができる| 複数のサブネットに同一の ACL を関連付けることができる|
