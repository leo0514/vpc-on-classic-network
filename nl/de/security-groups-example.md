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

# Sicherheitsgruppen über die CLI einrichten
{: #setting-up-security-groups-using-the-cli}

Im folgenden Beispiel erstellen Sie mithilfe der Befehlszeilenschnittstelle (CLI, Command Line Interface) eine virtuelle Serverinstanz (VSI) mit aktivierter Sicherheitsgruppe. Das Szenario sieht wie folgt aus.

![Sicherheitsgruppe für IBM VPC](images/security-groups-schematic.svg "Sicherheitsgruppe für IBM VPC")

Beachten Sie in der Abbildung, dass der VSI namens **SG4** neben seiner internen VPC-Adresse `10.0.0.5` zusätzlich die variable IP-Adresse `169.60.208.144` zugewiesen ist. Daher kann diese virtuelle Serverinstanz mit dem öffentlichen Internet kommunizieren. Die Sicherheitsgruppe, die der VSI **SG4** zugewiesen ist, heißt 'demosg'.

Die VSI namens **SG8** ist gegenüber der VPC rein intern und besitzt eine private IP-Adresse. Die Sicherheitsgruppe, die der VSI **SG8** zugewiesen ist, heißt 'my_vpc_sg'. Beide VSIs sind in der VPC mit dem Namen `sgvpc` wie auch in demselben Teilnetz `10.0.0.0/28` vorhanden, sodass sie miteinander kommunizieren können.

## Schritte zum Erstellen einer VSI mit angehängter Sicherheitsgruppe
{: #steps-for-creating-a-vsi-with-a-security-group-attached}

Die Sicherheitsgruppenregeln für 'my_vpc_sg' schließen SSH, PING und abgehenden TCP-Datenverkehr als Basisfunktionen ein.

Beachten Sie, dass Sie zuerst die Sicherheitsgruppe mit dem Befehl `ibmcloud is sgc` erstellen müssen und dann die VSI, die diese neu erstellte Sicherheitsgruppe enthalten wird.

Im vorliegenden Beispielcode werden einige Schritte übersprungen. Weitere Informationen finden Sie bei Bedarf hier:

 * Anweisungen zum Erstellen einer VPC und eines Teilnetzes finden Sie im Abschnitt [VPC erstellen](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli).

Aus diesem Beispiel-CLI-Code können Sie Befehle kopieren und einfügen, um mit der Erstellung einer VSI, der eine Sicherheitsgruppe angehängt ist, zu beginnen. Systemantworten werden in diesem Beispielcode nicht vollständig angezeigt. Es ist erforderlich, dass Sie die Befehle mit den korrekten Ressourcen-IDs für Ihr _VPC_, _Teilnetz_, _Image_, _Schlüssel_ und der zutreffenden _Sicherheitsgruppen-ID-Nummer_ aktualisieren..

1. Sicherheitsgruppe namens 'my_vpc_sg' erstellen

   ```
   ibmcloud is security-group-create my_vpc_sg $vpc
   ```
   {: pre}

   Speichern Sie die ID in einer Variablen, sodass sie zu einem späteren Zeitpunkt verwendet werden kann, zum Beispiel in der Variablen namens `sg`:

   ```
   sg=2d364f0a-a870-42c3-a554-000000632953
   ```
   {: pre}

2. Regeln hinzufügen, die SSH und PING und abgehendes TCP zulassen

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
ibmcloud is security-group-rule-add $sg outbound tcp
   ```
   {: codeblock}

3. VSI mit der neu erstellten Sicherheitsgruppe erstellen

   ```
   ibmcloud is instance-create test-instance $vpc us-south-2 b-4x16 $subnet 1000 \ 
   --image $image --keys $key --security-groups $sg
   ```
   {: pre}

## Cheat-Sheet zum Auflisten der Befehle
{: #command-list-cheat-sheet}

Sie können eine vollständige Auflistung aller verfügbaren VPC-CLI-Befehle für Sicherheitsgruppen abrufen, indem Sie Folgendes eingeben:

```
ibmcloud is help | grep sg
```
{: pre}

Wenn Ihre Sicherheitsgruppe und die zugehörigen Metadaten einschließlich Regeln angezeigt werden sollen, müssten Sie zum Beispiel (für das vorhergehende Beispiel) Folgendes eingeben:

```
ibmcloud is sg $sg
```
{: pre}

Wenn Sie eine Sicherheitsgruppenregel hinzufügen wollen, orientieren Sie sich an dem nachfolgenden Beispielbefehl zum Hinzufügen einer Regel für eingehende Pingsignale zu einer Sicherheitsgruppe:

```
ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0

```
{: pre}
