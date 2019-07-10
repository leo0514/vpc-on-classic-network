---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-29"

keywords: peering, classic, infrastructure, VRF, resources

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

# VPC のクラシック・ピアリング
{: #setting-up-classic-peering-for-vpc}

{{site.data.keyword.cloud}} VPC では、クラシック IBM Cloud リソースとピアリングすることができます。 ピアリングできるためには、アカウントでいくつかの要件を満たす必要があります。

クラシック・ピアリングに対応した VPC を作成するためのステップごとの説明については、その手順の詳細が記載されている [VPCインフラストラクチャーの資料](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#setting-up-access-to-your-classic-infrastructure-from-vpc)を参照してください。

## 要件
{: #classic-peering-requirements}

1. VPC は作成時に「クラシック・ピアリング」用に設計されている必要があります。クラシック・ピアリングを後で指定することはできません。

2. VPC のピアリング先となるリンクされたアカウントは VRF に対応している必要があります。 VRF について詳しくは、[この資料](/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud)の説明を参照してください。

3. また、クラシック・インフラストラクチャー側で使用する各インスタンスからクラシック対応 VPC に戻るための経路を追加する必要もあります。

## 制限
{: #classic-peering-restrictions}

クラシック・ピアリングを有効にできる VPC は、リージョンごとに 1 つのみです。
