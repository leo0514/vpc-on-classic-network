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

# Klassisches Peering für VPC
{: #setting-up-classic-peering-for-vpc}

Mit {{site.data.keyword.cloud}} VPC ist Peering mit den klassischen IBM Cloud-Ressourcen möglich. Hierfür müssen Ihre Konten eine Reihe von Voraussetzungen erfüllen.

Wenn Sie detailliertere Informationen zu dieser Prozedur benötigen, finden Sie im Dokument zur [VPC-Infrastruktur](/docs/vpc-on-classic?topic=vpc-on-classic-setting-up-access-to-your-classic-infrastructure-from-vpc#setting-up-access-to-your-classic-infrastructure-from-vpc) Anleitungen zur Erstellung einer VPC mit aktiviertem klassischen Peering in einzelnen Schritten.

## Voraussetzungen
{: #classic-peering-requirements}

1. Klassisches Peering muss für die VPC bei der Erstellung festgelegt werden, dies ist zu einem späteren Zeitpunkt nicht möglich.

2. Das für das Peering der VPC verwendete verknüpfte Konto muss für VRF aktiviert sein. Weitere Informationen zu VRF finden Sie in den [Erläuterungen in diesem Dokument](/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud).

3. Sie müssen auch eine Route zurück zu Ihrem Classic-fähigen VPC von jeder Instanz hinzufügen, die Sie auf der Seite "Klassischer Infrastruktur" verwenden.

## Einschränkungen
{: #classic-peering-restrictions}

Nur 1 VPC pro Region kann für klassisches Peering aktiviert werden.
