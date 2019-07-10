---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-06-06"

keywords: provisioning, resources, permissions

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:important: .important}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Lernprogramm zur Einführung
{: #getting-started}

Gehen Sie wie folgt vor, um mit dem {{site.data.keyword.cloud}}-Netzbetrieb für Virtual Private Cloud zu beginnen:

1. Erstellen Sie eine virtuelle private Cloud.
2. Erstellen Sie in dieser virtuellen privaten Cloud in einer oder mehreren Zonen ein oder mehrere Teilnetze.
3. Erstellen Sie ein öffentliches Gateway (PGW, Public Gateway) in einem Teilnetz, wenn Ressourcen in dem Teilnetz Zugriff auf das Internet erhalten sollen oder umgekehrt.

## Voraussetzungen

 * **Benutzerberechtigungen**: Stellen Sie sicher, dass Ihr Benutzer über ausreichende Berechtigungen zum Erstellen und Verwalten von Ressourcen in Ihrer VPC verfügt. Eine Liste der erforderlichen Berechtigungen finden Sie in [Für Benutzer von VPC benötigte Berechtigungen erteilen](/docs/vpc-on-classic?topic=vpc-on-classic-managing-user-permissions-for-vpc-resources).

## Verwenden der UI, CLI oder REST-API zur Bereitstellung von VPC-Netzressourcen

Die Einrichtung, Bereitstellung und Verwaltung von VPC-Netzressourcen kann über die UI (User Interface, Benutzerschnittstelle), die CLI (Command Line Interface, Befehlszeilenschnittstelle) oder die REST-API (Application Programming Interface, Anwendungsprogrammierschnittstelle) vorgenommen werden.

* Um Zugriff über die Benutzerschnittstelle (UI) zu erhalten, melden Sie sich an der [IBM Cloud-Konsole ![Symbol für externen Link](../../icons/launch-glyph.svg "Symbol für externen Link")]( https://{DomainName}/vpc){: new_window} an. Folgen Sie den Anweisungen im [Handbuch zur UI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console), falls Sie Hilfe benötigen.
* Wenn Sie die Befehlszeilenschnittstelle verwenden möchten, folgen Sie dem Beispiel [Hello World](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli).
* Fortgeschrittene Benutzer können die [REST-APIs](https://{DomainName}/apidocs/vpc-on-classic) direkt aufrufen. Gehen Sie gemäß dem Lernprogramm für [Beispielcode](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis) vor, um den Einstieg in das Arbeiten mit REST-APIs zu finden.

## Nächste Schritte

Nachdem der Netzbetrieb für Ihre virtuelle private Cloud eingerichtet worden ist, können Sie weitere Aspekte erkunden.

* [Virtuelle Serverinstanzen (VSIs) erstellen und verwalten](/docs/vpc-on-classic?topic=vpc-on-classic-creating-and-managing-virtual-server-instances)
* [Sicherheitsgruppen verwenden](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-security-groups-using-the-cli)
* [Netz-ACLs verwenden](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls)
* [VPN verwenden](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-vpn-with-your-vpc)
* [Lastausgleichsfunktionen verwenden](/docs/vpc-on-classic-network?topic=vpc-on-classic-network---using-load-balancers-in-ibm-cloud-vpc)
