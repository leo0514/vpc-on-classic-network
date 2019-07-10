---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, FortiGate, connection, secure

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Sichere Verbindung mit einem fernen FortiGate-Peer erstellen
{: #creating-a-secure-connection-with-a-remote-fortigate-peer}

Dieses Dokument basiert auf FortiGate 300C, Firmwareversion	5.2.13, Build 762 (GA).

Bei den nachfolgenden Beispielschritten werden die vorausgesetzten Schritte zum Erstellen von VPCs unter Verwendung der {{site.data.keyword.cloud}}-API oder -CLI übersprungen. Weitere Informationen finden Sie in der [Einführung](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) und in [VPC-Einrichtung mit APIs](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Beispielschritte
{: #fortigate-example-steps}

Die Topologie für die Verbindungsherstellung zum fernen FortiGate-Peer hat Ähnlichkeit mit der Erstellung einer VPN-Verbindung zwischen zwei VPCs. Eine Seite wird jedoch durch die FortiGate 300C-Einheit ersetzt.

![Abbildungsbeschreibung](./images/vpc-vpn-fg-figure.png)

### Vorgehensweise zum Erstellen einer sicheren Verbindung mit dem fernen Fortigate-Peer
{: #to-create-a-secure-connection-with-the-remote-fortigate-peer}

Wechseln Sie zu **VPN \> IPsec \> Tunnel** und erstellen Sie einen neuen angepassten Tunnel oder bearbeiten Sie einen vorhandenen.

![vpc-vpn-fg-start](./images/vpc-vpn-fg-start.JPG)

Wenn eine FortiGate-Einheit eine Verbindungsanforderung von einem fernen VPN-Peer empfängt, verwendet sie IPsec-Parameter für Phase 1, um eine sichere Verbindung herzustellen und diesen VPN-Peer zu authentifizieren. Wenn die Sicherheitsrichtlinie die Verbindung zulässt, richtet die FortiGate-Einheit dann unter Verwendung der IPsec-Parameter für Phase 1 den Tunnel ein und wendet die IPsec-Sicherheitsrichtlinie an. Schlüsselmanagement, Authentifizierung und Sicherheitsservices werden dynamisch über das IKE-Protokoll verhandelt.

**Damit diese Funktionen unterstützt werden, müssen die folgenden allgemeinen Konfigurationsschritte von der FortiGate-Einheit ausgeführt werden:**

* Definieren der Parameter für Phase 1, die die FortiGate-Einheit benötigt, um den fernen Peer zu authentifizieren und eine sichere Verbindung herzustellen.

* Definieren der Parameter von Phase 2, die die FortiGate-Einheit benötigt, um einen VPN-Tunnel mit dem fernen Peer zu erstellen.

* Erstellen von Sicherheitsrichtlinien zur Steuerung der zulässigen Services und der zulässigen Richtung für den Datenverkehr zwischen der IP-Quelle und den Zieladressen.

Standardmäßig sind bereits einige typische Parameter für Phase 1 und 2 festgelegt.

### Für IBM Cloud VPC-VPN konfigurieren
{: #fortigate-configuring-for-the-ibm-cloud-vpc-vpn}

Um eine Verbindung mit der VPN-Funktionalität von IBM Cloud VPC herzustellen, wird die folgende Konfiguration empfohlen:

1. Wählen Sie im Schritt für die Authentifizierung 'IKEv2' aus.
2. Aktivieren Sie im Vorschlag für Phase 1 den Eintrag `DH-group 2`.
3. Legen Sie im Vorschlag für Phase 1 den Wert `lifetime = 36000` fest.
4. Inaktivieren Sie im Vorschlag für Phase 2 den Wert 'PFS'.
5. Legen Sie im Vorschlag für Phase 2 den Wert `lifetime = 10800` fest.
6. Geben Sie bei Phase 2 die Informationen für Ihr Teilnetz ein.
7. Für die restlichen Parameter bleiben die Standardwerte unverändert.

![Abbildungsbeschreibung](./images/vpc-vpn-fg-network.JPG)

### Netzparameter
{: #fortigate-network-parameters}

![Abbildungsbeschreibung](./images/vpc-vpn-fg-authentication.JPG)

### Authentifizierungsparameter
{: #fortigate-authentication-parameters}

![Abbildungsbeschreibung](./images/vpc-vpn-fg-phase1.JPG)

### Parameter für Phase 1
{: #fortigate-phase-1-parameters}

![Abbildungsbeschreibung](./images/vpc-vpn-fg-phase2.JPG)

### Parameter für Phase 2
{: #fortigate-phase-2-parameters}

## Vorgehensweise zum Erstellen einer sicheren Verbindung mit der lokalen IBM Cloud-VPC
{: #fortigate-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

Zum Erstellen einer sicheren Verbindung erstellen Sie die VPN-Verbindung innerhalb Ihrer VPC, was dem Beispiel mit den 2 VPCs ähnelt.

* Erstellen Sie ein VPN-Gateway auf Ihrem VPC-Teilnetz sowie eine VPN-Verbindung zwischen der VPC und der FortiGate-Einheit. Legen Sie dabei für `local_cidrs` den Wert für das Teilnetz auf der VPC und für `peer_cidrs` den Wert für das Teilnetz auf der FortiGate-Einheit fest.

**Hinweis:** Für das Gateway wird der Status 'Anstehend' (`pending`) angezeigt, solange das VPN-Gateway erstellt wird. Der Status wechselt zu 'Verfügbar' (`available`), sobald der Erstellungsvorgang abgeschlossen ist. Die Erstellung kann einige Zeit in Anspruch nehmen.

![Abbildungsbeschreibung](images/vpc-vpn-fg-connection.png)

### Status der sicheren Verbindung prüfen
{: #fortigate-check-the-status-of-the-secure-connection}

Sie können den Status Ihrer Verbindung über die IBM Cloud-Konsole prüfen. Außerdem könnten Sie auch versuchen, von Site zu Site ein `Pingsignal` mit den VSIs abzusetzen.

![Abbildungsbeschreibung](images/vpc-vpn-fg-status.JPG)
