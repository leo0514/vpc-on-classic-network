---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, StrongSwan, connection, secure, Linux, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{: note: .note}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Sichere Verbindung mit einem fernen StrongSwan-Peer erstellen
{: #creating-a-secure-connection-with-a-remote-strongswan-peer}

Dieses Dokument basiert auf Strongswan, Version Linux StrongSwan U5.3.5/K4.4.0-133-generic.

Bei den nachfolgenden Beispielschritten werden die vorausgesetzten Schritte zum Erstellen von VPCs unter Verwendung der {{site.data.keyword.cloud}}-API oder -CLI übersprungen. Weitere Informationen finden Sie in der [Einführung](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) und in [VPC-Einrichtung mit APIs](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Beispielschritte
{: #strongswan-example-steps}

Die Topologie für die Verbindungsherstellung zum fernen StrongSwan-Peer hat Ähnlichkeit mit der Erstellung einer VPN-Verbindung zwischen zwei VPCs. Eine Seite der Verbindung wird jedoch durch die StrongSwan-Einheit ersetzt.

![Abbildungsbeschreibung](./images/vpc-vpn-sw-figure.png)

### Vorgehensweise zum Erstellen einer sicheren Verbindung mit einen fernen StrongSwan-Peer
{: #to-create-a-secure-connection-with-a-remote-strongswan-peer}

Wechseln Sie zu **/etc** und erstellen Sie Ihre neue angepasste Tunnelkonfigurationsdatei mit einem Namen wie **ipsec.abc.conf**. Bearbeiten Sie **/etc/ipsec.conf** so, dass **ipsec.abc.conf** einbezogen wird. Fügen Sie dazu die folgende Zeile hinzu:

    include /etc/ipsec.abc.conf

Wenn ein VPN-Peer eine Verbindungsanforderung von einem fernen VPN-Peer empfängt, verwendet sie IPsec-Parameter für Phase 1, um eine sichere Verbindung herzustellen und diesen VPN-Peer zu authentifizieren. Wenn die Sicherheitsrichtlinie die Verbindung zulässt, richtet die StrongSwan-Einheit dann unter Verwendung der IPsec-Parameter für Phase 2 den Tunnel ein und wendet die IPsec-Sicherheitsrichtlinie an. Schlüsselmanagement, Authentifizierung und Sicherheitsservices werden dynamisch über das IKE-Protokoll verhandelt.

**Damit diese Funktionen unterstützt werden, müssen die folgenden allgemeinen Konfigurationsschritte von der StrongSwan-Einheit ausgeführt werden:**

* Definieren der Parameter für Phase 1, die die StrongSwan-Einheit benötigt, um den fernen Peer zu authentifizieren und eine sichere Verbindung herzustellen.

* Definieren der Parameter von Phase 2, die die StrongSwan-Einheit benötigt, um einen VPN-Tunnel mit dem fernen Peer zu erstellen.
Um eine Verbindung mit der VPN-Funktionalität von IBM Cloud VPC herzustellen, wird die folgende Konfiguration empfohlen:

1. Wählen Sie im Schritt für die Authentifizierung `IKEv2` aus.
2. Aktivieren Sie im Vorschlag für Phase 1 den Eintrag `DH-group 2`.
3. Legen Sie im Vorschlag für Phase 1 den Wert `lifetime = 36000` fest.
4. Inaktivieren Sie im Vorschlag für Phase 2 den Wert 'PFS'.
5. Legen Sie im Vorschlag für Phase 2 den Wert `lifetime = 10800` fest.
6. Geben Sie beim Vorschlag für Phase 2 die Informationen für Ihren Peer und Ihr Teilnetz ein.

```
    vim /etc/ipsec.abc.conf
    conn all
           type=tunnel
           auto=route
           #aggressive=no
           esp=aes256-sha256!
           ike=aes128-sha1-modp1024!
           left=169.45.74.119
           leftsubnet=10.160.26.64/26
           rightsubnet=192.168.17.0/28
           right=169.61.181.116
           leftauth=psk
           rightauth=psk
           leftid="169.45.74.119"
           keyexchange=ikev2
           rightid="169.61.181.116"
           lifetime=10800s
           ikelifetime=36000s
           dpddelay=30s
           dpdaction=restart
           dpdtimeout=120s
```
{: screen}

Legen Sie den vorab verteilten Schlüssel in `/etc/ipsec.secrets` fest.

```
vim ipsec.secrets
# Diese Datei enthält gemeinsam genutzte geheime Schlüssel oder RSA-Private-Keys für die Authentifizierung.

169.45.74.119 169.61.181.116 : PSK "******"

```
{: screen}

Nachdem die Ausführung der Konfigurationsdatei abgeschlossen ist, starten Sie die StrongSwan-Einheit erneut.

```
 ipsec restart
```
{: screen}

### Vorgehensweise zum Erstellen einer sicheren Verbindung mit der lokalen IBM Cloud-VPC
{: #strongswan-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

* Zum Erstellen einer sicheren Verbindung erstellen Sie die VPN-Verbindung innerhalb Ihrer VPC, was dem Beispiel mit den 2 VPCs ähnelt.

* Erstellen Sie ein VPN-Gateway auf Ihrem VPC-Teilnetz sowie eine VPN-Verbindung zwischen der VPC und der StrongSwan-Einheit. Legen Sie dabei für `local_cidrs` den Wert für das Teilnetz auf der VPC und für `peer_cidrs` den Wert für das Teilnetz auf der StrongSwan-Einheit fest.

Für das Gateway wird der Status 'Anstehend' (`pending`) angezeigt, solange das VPN-Gateway erstellt wird. Der Status wechselt zu 'Verfügbar' (`available`), sobald der Erstellungsvorgang abgeschlossen ist. Die Erstellung kann einige Zeit in Anspruch nehmen.
{: note}

![vpc-vpn-sw-connection](./images/vpc-vpn-sw-connection.png)

### Status einer sicheren Verbindung prüfen
{: #strongswan-check-the-status-for-a-secure-connection}

Sie können den Status Ihrer Verbindung über die IBM Cloud-Konsole prüfen. Außerdem könnten Sie auch versuchen, von Site zu Site ein `Pingsignal` mit den VSIs abzusetzen.

![vpc-vpn-sw-status.png](./images/vpc-vpn-sw-status.png)
