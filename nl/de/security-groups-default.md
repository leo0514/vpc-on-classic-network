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

# Standardsicherheitsgruppe aktualisieren
{: #updating-the-default-security-group}


Die Standardsicherheitsgruppe hat große Ähnlichkeit mit jeder beliebigen anderen Sicherheitsgruppe, jedoch mit der Ausnahme, dass sie nicht gelöscht werden kann.

Jede VPC besitzt eine Standardsicherheitsgruppe mit Regeln, die Folgendes zulassen:

* Eingehenden Datenverkehr von allen Mitgliedern der Gruppe
* Den gesamten abgehenden Datenverkehr

Wenn Sie die Regeln der Standardsicherheitsgruppe bearbeiten, gelten diese bearbeiteten Regeln dann für alle aktuellen und zukünftigen Server in der Gruppe.

Regeln für eingehenden Datenverkehr, die Pingsignale und SSH zulassen, werden nicht automatisch zur Standardsicherheitsgruppe hinzugefügt. Entsprechende Regeländerungen an der Standardsicherheitsgruppe können Sie über die REST-API, die Befehlszeilenschnittstelle (`ibmcloud cli`) oder die Benutzerschnittstelle (UI) vornehmen.

## Beispiel: Änderungen an den Regeln der Standardsicherheitsgruppe über die CLI vornehmen
{: #example-modifying-the-default-security-group-rules-using-the-cli}

1. Melden Sie sich bei IBM Cloud an.

   Wenn Sie über ein föderiertes Konto verfügen, geben Sie folgenden Befehl ein:
   ```
   ibmcloud login -sso
   ```
   {: pre}

   Verwenden Sie andernfalls den folgenden Befehl:

   ```
   ibmcloud login
   ```
   {: pre}

2. ID und Details der Standardsicherheitsgruppe für die virtuelle private Cloud abrufen

   Führen Sie den folgenden CLI-Befehl auf, damit alle VPCs aufgelistet werden:

   ```
   ibmcloud is vpc
   ```
   {: pre}

   Der Name der Standardsicherheitsgruppe wird in der Spalte `Standardsicherheitsgruppe` angezeigt. Notieren Sie ihren Namen, sodass Sie die `ID` in der Auflistung der Sicherheitsgruppen (im nächsten Schritt) finden können. 
   
   Listen Sie jetzt alle Sicherheitsgruppen in der VPC auf:

   ```
   ibmcloud is security-groups
   ```
   {: pre}

   Speichern Sie die Sicherheitsgruppen-ID (für die Standardsicherheitsgruppe) in einer Variablen, so dass Sie sie zu einem späteren Zeitpunkt verwenden können. Verwenden Sie zum Beispiel den Variablennamen `sg`:

   ```
   sg=2d364f0a-a870-42c3-a554-000001162469
   ```
   {: pre}

   Führen Sie zum Abrufen von Details zur Sicherheitsgruppe den folgenden CLI-Befehl aus:

   ```
   ibmcloud is security-group $sg
   ```
   {: pre}
   
   Alternativ könnten Sie an Stelle der Variablen `$sg` den tatsächlichen ID-Wert einfügen.

3. Standardsicherheitsgruppe aktualisieren -- Regeln hinzufügen, die SSH und PING zulassen

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
   ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
   ```
   {: codeblock}


Das Hinzufügen und das Entfernen von Sicherheitsgruppenregeln erfolgt jeweils als asynchrone Operation. In der Regel dauert es zwischen einer und 30 Sekunden, bis die Änderung in Kraft tritt.
{: note}
