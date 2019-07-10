---
copyright:
  years: 2019
lastupdated: "2019-05-14"

keywords: security groups, traffic, firewall, stateful, filtering, access, control, list, ACL, stateless, traffic, resource

subcollection: vpc-on-classic-network
---

# 比较安全组与访问控制表
{: #compare-security-groups-and-access-control-lists}

安全组和访问控制表提供了通过您指定的规则，在 {{site.data.keyword.cloud}} VPC 中的各个子网和实例中控制流量的方法。

下表总结了安全组与 ACL 之间的一些关键差异：

|  |安全组|ACL|
|-------------|-----------------|---------|
|控制级别|VSI 实例|子网|
|状态| 有状态 - 一旦允许入站连接，就可以对其进行回复 | 无状态 - 必须显式允许入站和出站连接 |
|支持的规则|仅使用允许规则|使用允许和拒绝规则|
|规则应用方式|考虑所有规则|规则按顺序处理|
|与关联资源的关系|一个实例可以与多个安全组相关联|多个子网可以与同一 ACL 相关联|
