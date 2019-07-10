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


# Netz-ACLs einrichten
{: #setting-up-network-acls}
[comment]: # (Verlinktes Hilfethema)

Anhand der in {{site.data.keyword.cloud}} Virtual Private Cloud verfügbaren Funktionalität für Zugriffssteuerungslisten (ACLs, Access Control Lists) können Sie den eingehenden und den abgehenden Datenverkehr für kritische geschäftliche Workloads in der Cloud steuern. Bei einer ACL handelt es sich um eine integrierte virtuelle Firewall, die einer Sicherheitsgruppe ähnelt. Im Gegensatz zu Sicherheitsgruppen steuern ACL-Regeln den Datenverkehr zu und von den _Teilnetzen_ und nicht zu und von den _Instanzen_. 

In der [Vergleichstabelle](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-compare-security-groups-and-access-control-lists) finden Sie einen Vergleich der Merkmale von Sicherheitsgruppen und ACLs.

Das in diesem Dokument erläuterte Beispiel zeigt, wie Sie Netz-ACLs in Ihrer VPC über die CLI erstellen und dadurch Ihre Teilnetze schützen.

## Für ACLs verfügbare APIs und CLIs
{: #apis-and-clis-are-available-for-acls}

ACLs können über die Anwendungsprogrammierschnittstelle (API), die Befehlszeilenschnittstelle (CLI) oder die Benutzerschnittstelle (UI) eingerichtet und verwaltet werden.

* Informationen zu den Parametern, dem Anforderungshauptteil und den Antwortdetails für jede Anwendungsprogrammierschnittstelle (API) finden Sie in der [API-Referenz](https://{DomainName}/apidocs/vpc-on-classic).

* Informationen zum Abrufen von Befehlsdetails, zu den Schritten zur Installation des CLI-Plug-ins und den vorausgesetzten Schritten für die Verwendung der Befehlszeilenschnittstelle (CLI) von VPC enthält dieses [CLI-Beispiel](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli).

* Informationen zum Einrichten von ACLs in der {{site.data.keyword.cloud_notm}}-Konsole finden Sie unter [ACL unter Verwendung der Benutzerschnittstelle konfigurieren](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#configuring-the-acl).

Die ACL-Regeln steuern den Datenverkehr zu und von den _Teilnetzen_. Es scheint zwar, dass die VPC-API, die Befehlszeilenschnittstelle und die Benutzerschnittstelle Unterstützung bereitstellen, um den **Ziel-IP-Bereich** für eingehende Regeln und den **Quellen-IP-Bereich** für abgehende Regeln anzupassen. Diese Adressen sind jedoch nicht anpassbar. **Der Ziel-IP-Bereich der eingehenden ACL-Regeln und der Quellen-IP-Bereich der abgehenden ACL-Regeln sind jeweils an die angehängten Teilnetze gebunden**. In der Praxis sollte **0.0.0.0/0** deshalb als Ziel-IP für eingehende Regeln und als Quellen-IP für abgehende Regeln verwendet werden.
{: note}

## Mit ACLs und ACL-Regeln arbeiten
{: #working-with-acls-and-acl-rules}

Um Ihre Zugriffssteuerungslisten (ACLs, Access Control Lists) möglichst wirksam zu gestalten, erstellen Sie Regeln, die festlegen, wie die Handhabung Ihres eingehenden und abgehenden Netzverkehrs zu erfolgen hat. Dabei können Sie mehrere Regeln für eingehenden und für abgehenden Datenverkehr erstellen. Spezielle Informationen dazu, wie viele Regeln Sie erstellen dürfen, finden Sie in [Kontingente](/docs/vpc-on-classic?topic=vpc-on-classic-quotas).

* Mit Regeln für den eingehenden Datenverkehr können Sie Datenverkehr von einem Quellen-IP-Bereich mit den angegebenen Protokollen und Ports zulassen oder aber zurückweisen. Der Ziel-IP-Bereich wird durch das angehängte Teilnetz bestimmt.
* Mit Regeln für den abgehenden Datenverkehr können Sie Datenverkehr von einem Ziel-IP-Bereich mit den angegebenen Protokollen und Ports zulassen oder aber zurückweisen. Der Quellen-IP-Bereich wird durch das angehängte Teilnetz bestimmt.
* ACL-Regeln werden der Reihenfolge nach berücksichtigt. Regeln mit einer höheren Prioritätsnummer (wie z. B. 2) werden nur dann ausgewertet, wenn Regeln mit einer niedrigeren Prioritätsnummer (wie z. B. 1) nicht übereinstimmen. 
* Regeln für eingehenden Datenverkehr sind von den Regeln für abgehenden Datenverkehr getrennt.
* Gegenwärtig werden die Protokolle TCP, UDP und ICMP unterstützt. Sie können auch die Option **alle** verwenden, um _alle_ Protokolle festzulegen, und Sie können die Option _andere_ verwenden, sofern eine Regel mit einer höheren Priorität angegeben ist.

Relevante Informationen zur Verwendung der Protokolle ICMP, TCP und UDP in Ihren ACL-Regeln finden Sie in [Internet-Kommunikationsprotokolle verstehen](/docs/infrastructure/network-infrastructure?topic=network-infrastructure-understanding-internet-communication-protocols).

### ACL an ein Teilnetz anhängen
{: #attaching-an-acl-to-a-subnet}

Beim Anhängen einer ACL an ein Teilnetz stehen Ihnen zwei Optionen offen:

* Sie könne ein neues Teilnetz erstellen und eine ACL angeben, die angehängt werden soll. Wenn Sie keine ACL angeben, wird eine Standardnetz-ACL angehängt. Die Standard-ACL lässt den gesamten eingehenden Datenverkehr für dieses Teilnetz und den gesamten von diesem Teilnetz abgehenden Datenverkehr zu.
* Sie können eine ACL an ein vorhandenes Teilnetz anhängen. Wenn bereits eine andere ACL an dieses Teilnetz angehängt ist, wird diese bereit angehängte ACL abgehängt, bevor die neue ACL angehängt wird.

## ACL-Anschauungsbeispiel
{: #acl-demo-example}

Im nachfolgenden Beispiel können Sie zwei Zugriffssteuerungslisten (ACLs) erstellen und diese über die Befehlszeilenschnittstelle (CLI, Command Line Interface) zwei Teilnetzen zuordnen. Das Szenario sieht wie folgt aus:

![ACL-Beispielszenario](images/vpc-acls.png)

Wie die Abbildung veranschaulicht verfügen Sie über zwei Webserver, die Anfragen aus dem Internet verarbeiten, und über zwei Backend-Server, die vor der Öffentlichkeit verborgen werden sollen. In diesem Beispiel ordnen Sie die Server in zwei separaten Teilnetzen an (10.10.10.0/24 und 10.10.20.0/24). Außerdem müssen Sie den Webservern den Datenaustausch mit den Back-End-Servern ermöglichen. Darüber soll in begrenztem Maß abgehender Datenverkehr von den Back-End-Servern zugelassen werden.

### Beispielregeln
{: #acl-example-rules}

Die nachfolgend aufgeführten Beispielregeln zeigen, wie Sie die ACL-Regeln für ein Basisszenario wie zuvor beschrieben einrichten.

Als bewährtes Verfahren wird empfohlen, stärker differenzierten Regeln eine höhere Priorität zuzuweisen als allgemeiner definierten Regeln. Wenn zum Beispiel eine Ihrer Regeln den gesamten Datenverkehr aus dem Subnetz 10.10.30.0/24 blockiert und mit einer höheren Priorität versehen ist, so findet die stärker differenzierte Regel mit niedrigerer Priorität, gemäß der der Datenverkehr von 10.10.30.5 zugelassen werden soll, niemals Anwendung.
{:note}

**Beispielregeln für die Zugriffssteuerungsliste ACL-1**:

| Eingehend/Abgehend| Zulassen/Zurückweisen | Quellen-IP | Ziel-IP | Protokoll | Port | Beschreibung|
|--------------|-----------|------|------|------|------------------|-------|
| Eingehend | Zulassen | 0.0.0.0/0 | 0.0.0.0/0 | TCP| 80 | HTTP-Datenverkehr aus dem Internet zulassen|
| Eingehend | Zulassen | 0.0.0.0/0 | 0.0.0.0/0 | TCP | 443 | HTTPS-Datenverkehr aus dem Internet zulassen|
| Eingehend | Zulassen| 10.10.20.0/24 | 0.0.0.0/0 |Alle| Alle| Gesamten eingehenden Datenverkehr von Teilnetz 10.10.20.0/24, in dem sich die Back-End-Server befinden, zulassen|
| Eingehend | Zurückweisen| 0.0.0.0/0| 0.0.0.0/0 |Alle| Alle| Allen übrigen eingehenden Datenverkehr zurückweisen|
| Abgehend | Zulassen | 0.0.0.0/0 | 0.0.0.0/0 | TCP|80 | HTTP-Datenverkehr ins Internet zulassen|
| Abgehend | Zulassen | 0.0.0.0/0 | 0.0.0.0/0 | TCP|443 | HTTPS-Datenverkehr ins Internet zulassen|
| Abgehend | Zulassen| 0.0.0.0/0 | 10.10.20.0/24 |Alle| Alle| Gesamten abgehenden Datenverkehr ins Teilnetz 10.10.20.0/24, in dem sich die Back-End-Server befinden, zulassen|
| Abgehend | Zurückweisen| 0.0.0.0/0 | 0.0.0.0/0|Alle| Alle| Allen übrigen abgehenden Datenverkehr zurückweisen|


**Beispielregeln für die Zugriffssteuerungsliste ACL-2**:

| Eingehend/Abgehend| Zulassen/Zurückweisen | Quellen-IP | Ziel-IP | Protokoll| Port | Beschreibung|
|--------------|-----------|------|------|------|------------------|--------|
| Eingehend | Zulassen| 10.10.10.0/24 | 0.0.0.0/0 |Alle| Alle| Gesamten eingehenden Datenverkehr von Teilnetz 10.10.10.0/24, in dem sich die Web-Server befinden, zulassen|
| Eingehend | Zurückweisen| 0.0.0.0/0| 0.0.0.0/0 |Alle| Alle| Allen übrigen eingehenden Datenverkehr zurückweisen|
| Abgehend | Zulassen | 0.0.0.0/0 | 0.0.0.0/0 | TCP| 80 | HTTP-Datenverkehr ins Internet zulassen|
| Abgehend | Zulassen | 0.0.0.0/0 | 0.0.0.0/0 | TCP| 443 | HTTPS-Datenverkehr ins Internet zulassen|
| Abgehend | Zulassen| 0.0.0.0/0 | 10.10.10.0/24 |Alle| Alle| Gesamten abgehenden Datenverkehr ins Teilnetz 10.10.10.0/24, in dem sich die Back-End-Server befinden, zulassen|
| Abgehend | Zurückweisen| 0.0.0.0/0 | 0.0.0.0/0|Alle| Alle| Allen übrigen abgehenden Datenverkehr zurückweisen|

Dieses Beispiel veranschaulicht nur allgemeine Fälle. In Ihren Szenarien streben Sie möglicherweise eine differenziertere Steuerung des Datenverkehrs an:

* Einer Ihrer Netzwerkadministratoren könnte beispielsweise zu Betriebszwecken aus einem fernen Netz Zugriff auf das Teilnetz 10.10.10.0/24 benötigen. In diesem Fall müssten Sie den SSH-Datenverkehr aus dem Internet in dieses Teilnetz zulassen.
* Möglicherweise möchten Sie den Protokollbereich einschränken, den Sie zwischen Ihren beiden Teilnetzen zulassen.

### Beispielschritte
{: #acl-example-steps}

Bei den nachfolgenden Beispielschritten werden die vorausgesetzten Schritte zum Erstellen einer VPC über die CLI übersprungen. Diese Schritte müssen zuerst ausgeführt werden. Weitere Informationen finden Sie in [VPC über die CLI erstellen](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli).


#### Schritt 1. ACLs erstellen
{: #step-1-create-the-acls}

Verwenden zum Erstellen von zwei ACLs namens `my_web_subnet_acl` und `my_backend_subnet_acl` die folgenden CLI-Befehle:

```
ibmcloud is network-acl-create my_web_subnet_acl
ibmcloud is network-acl-create my_backend_subnet_acl
```
{: codeblock}

Die Antwort enthält die IDs für die neu erstellten ACLs. Speichern Sie die IDs für beide ACLs zur Verwendung zu einem späteren Zeitpunkt. Dazu könnten Sie Variablen namens `webacl` und `bkacl` wie folgt verwenden:

```
webacl="ba9e785a-3e10-418a-811c-56cfe5669676"
bkacl="a4e28308-8ee7-46ab-8108-9f881f22bdbf"
```
{: codeblock}

#### Schritt 2. Standard-ACL-Regeln abrufen
{: #step-2-retrieve-the-default-acl-rules}

Bevor Sie neue Regeln hinzufügen, sollten Sie die standardmäßigen ACL-Regeln für ein- und abgehenden Datenverkehr abrufen, sodass Sie vor Ihnen neue Regeln einfügen können.

```
ibmcloud is network-acl-rules $webacl
ibmcloud is network-acl-rules $bkacl
```
{: codeblock}

Die Antwort zeigt die Standardregeln für eingehenden und abgehenden Datenverkehr, die den gesamten IPv4-Datenverkehr in allen Protokollen zulassen.

```
Abrufen der Regeln der Netz-ACL ba9e785a-3e10-418a-811c-56cfe5669676 unter Konto Demo Account als Benutzer demouser...

Eingehend
ID                                     Name                                                          Aktion   IPv*   Protokoll  Quelle      Ziel          Erstellt
e2b30627-1a1d-447b-859f-ac9431986b6f   allow-all-inbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6   allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago

Abgehend
ID                                     Name                                                          Aktion   IPv*   Protokoll  Quelle      Ziel          Erstellt
173a3492-0544-472e-91c0-7828cbcb62d4   allow-all-outbound-rule-2d86bc3f-58e4-436a-8c1a-9b0a710556d6  allow    ipv4   all        0.0.0.0/0   0.0.0.0/0     2 months ago
```
{: screen}

Speichern Sie die IDs beider ACL-Regeln als Variablen, da Sie sie zu einem späteren Zeitpunkt in Befehlen verwenden werden. Sie könnten zum Beispiel Variablen namens `inrule` und `outrule` verwenden:

```
inrule="e2b30627-1a1d-447b-859f-ac9431986b6f"
outrule="173a3492-0544-472e-91c0-7828cbcb62d4"
```
{: codeblock}

#### Schritt 3. Neue ACL-Regeln wie beschrieben hinzufügen
{: #step-3-add-new-acl-rules-as-decribed}

In diesem Abschnitt wird zuerst gezeigt, wie Regeln für eingehenden Datenverkehr hinzugefügt werden, bevor dargelegt wird, wie Sie Regeln für abgehenden Datenverkehr hinzufügen.

Fügen Sie vor der Standardregel für eingehende Daten neue Regeln für eingehenden Datenverkehr ein.

```
ibmcloud is network-acl-rule-add my_web_acl_rule200 $webacl deny inbound all 0.0.0.0/0 0.0.0.0/0 \
--before-rule $inrule
```
{: pre}

Speichern Sie bei jedem Schritt die ID der ACL-Regel in einer Variablen, denn sie wird zu einem späteren Zeitpunkt in Befehlen verwendet. Sie könnten zum Beispiel den Variablennamen `acl200` verwenden:

```
acl200="90930627-1a1d-447b-859f-ac9431986b6f"
```
{: pre}

Fügen Sie nun die Regel zu `acl200` hinzu:

```
ibmcloud is network-acl-rule-add my_web_acl_rule100 $webacl allow inbound all 10.10.20.0/24 0.0.0.0/0 \
--before-rule $acl200
```
{: pre}

Fügen Sie solange weitere Regeln hinzu, bis Ihre ACL-Konfiguration abgeschlossen ist. Speichern Sie dabei jede ID als Variable.

```
acl100="78340627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule20 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 443 --port-min 443 --before-rule $acl100
acl20="32450627-1a1d-447b-859f-ac9431986b6f"
ibmcloud is network-acl-rule-add my_web_acl_rule10 $webacl allow inbound tcp 0.0.0.0/0 0.0.0.0/0 \
--port-max 80 --port-min 80 --before-rule $acl20
```
{: codeblock}

Fügen Sie vor der Standardregel für abgehende Daten neue Regeln für abgehenden Datenverkehr ein.

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

#### Schritt 4. Zwei Teilnetze mit der neu erstellten ACL erstellen
{: #step-4-create-the-two-subnets-with-the-newly-created-acl}

Sie erstellen zwei Teilnetze, sodass jede Ihrer ACLs einem der beiden neuen Teilnetze zugeordnet ist.

```
ibmcloud is subnet-create my_web_subnet my_VPC my_region --ipv4_cidr_block 10.10.10.0/24 \
--generation gc --network-acl $webacl
ibmcloud is subnet-create my_backend_subnet my_VPC my_region --ipv4_cidr_block 10.10.20.0/24 \
--generation gc --network-acl $bkacl
```
{: codeblock}


## Cheat-Sheet zum Auflisten der Befehle
{: #acl-cli-command-list-cheat-sheet}

So rufen Sie eine vollständige Auflistung aller verfügbaren VPC-CLI-Befehle für ACLs ab:

```
ibmcloud is network-acls
```
{: pre}

So zeigen Sie Ihre ACL mit den zugehörigen Metadaten einschließlich Regeln an:

```
ibmcloud is network-acl $webacl
```
{: pre}

So rufen Sie die ACL-Standardregel für eingehenden Datenverkehr ab:

```
ibmcloud is network-acl-rules $webacl --direction inbound
```
{: pre}

## Beispielregel für eingehende `ping`-Signale
{: #acl-example-inbound-ping-rule}

Wenn Sie eine ACL-Regel hinzufügen wollen, orientieren Sie sich an dem nachfolgenden Beispielbefehl, um eine Regel für eingehende `ping`-Signale vor der Standardegel für eingehenden Datenverkehr hinzuzufügen:

```
ibmcloud is network-acl-rule-add --action allow --direction inbound --protocol icmp --icmp-type 8 --icmp-code --before-rule-name <Name_der_ACL-Standardregel> <ACL-Name> <Name_der_neuen_ACL-Regel>
```
{: pre}
