---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, Cisco, ASAv, connection, secure, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc} 
{:new_window: target="_blank"} 
{:DomainName: data-hd-keyref="DomainName"} 
{:note: .note} 
{:important: .important} 
{:deprecated: .deprecated} 
{:generic: data-hd-programlang="generic"}

# Sichere Verbindung mit einem fernen Cisco ASAv-Peer erstellen
{: #creating-a-secure-connection-with-a-remote-cisco-asav-peer}

Dieses Dokument basiert auf Cisco ASAv, Cisco Adaptive Security Appliance Software Version 9.10(1).

Bei den nachfolgenden Beispielschritten werden die vorausgesetzten Schritte zum Erstellen von VPCs unter Verwendung der {{site.data.keyword.cloud}}-API oder -CLI übersprungen. Weitere Informationen finden Sie in der [Einführung](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) und in [VPC-Einrichtung mit APIs](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Beispielschritte
{: #cisco-example-steps}

Die Topologie für die Verbindungsherstellung zum fernen Cisco ASAv-Peer hat Ähnlichkeit mit der Erstellung einer VPN-Verbindung zwischen zwei Instanzen von {{site.data.keyword.cloud}} Virtual Private Cloud. Eine Seite wird jedoch durch die Cisco ASAv-Einheit ersetzt.

![Abbildungsbeschreibung](./images/vpc-vpn-asav-figure.png)

### Vorgehensweise zum Erstellen einer sicheren Verbindung mit dem fernen Cisco ASAv-Peer
{: #to-create-a-secure-connection-with-the-remote-cisco-asav-peer}

Als ersten Schritt bei der Konfiguration Ihres Cisco ASA für die Verwendung mit dem IBM VPC-VPN müssen Sie sicherstellen, dass die folgenden Vorbedingungen erfüllt sind:

* Cisco ASAv ist online und mit einer ordnungsgemäßen Lizenz betriebsbereit
* Für Cisco ASAv ist ein Kennwort aktiviert
* Es ist mindestens eine konfigurierte und verifizierte funktionale interne Schnittstelle vorhanden
* Es ist mindestens eine konfigurierte und verifizierte funktionale externe Schnittstelle vorhanden

Wenn eine Cisco ASAv-Einheit eine Verbindungsanfrage von einem fernen VPN-Peer empfängt, verwendet sie IPsec-Parameter der Phase 1, um eine sichere Verbindung herzustellen und diesen VPN-Peer zu authentifizieren. Wenn die Sicherheitsrichtlinie die Verbindung zulässt, erstellt die Cisco ASAv-Einheit mithilfe der IPsec-Parameter der Phase 1 den Tunnel und wendet die IPsec-Sicherheitsrichtlinie an. Schlüsselmanagement, Authentifizierung und Sicherheitsservices werden dynamisch über das IKE-Protokoll verhandelt.

**Damit diese Funktionen unterstützt werden, müssen die folgenden allgemeinen Konfigurationsschritte von der Cisco ASAv-Einheit ausgeführt werden:**

* Definieren der Parameter für Phase 1, die die Cisco ASAv-Einheit benötigt, um den fernen Peer zu authentifizieren und eine sichere Verbindung herzustellen.
* Definieren der Parameter von Phase 2, die die Cisco ASAv-Einheit benötigt, um einen VPN-Tunnel mit dem fernen Peer zu erstellen.

Erstellen Sie ein IKEv2-Vorschlagsobjekt (Internet Key Exchange, Version 2). IKEv2-Vorschlagsobjekte enthalten die Parameter, die zum Erstellen von IKEv2-Vorschlägen erforderlich sind, wenn Fernzugriff und Site-to-Site-VPN-Richtlinien definiert werden. IKE ist ein Schlüsselmanagementprotokoll, das die Verwaltung der IPsec-basierten Kommunikation vereinfacht. Es wird zum Authentifizieren von IPsec-Peers, Verhandeln und Verteilen von IPsec-Verschlüsselungsschlüsseln sowie zum automatischen Einrichten von IPsec-Sicherheitszuordnungen verwendet. 

```
group-policy GroupPolicy_161.156.80.10 internal
group-policy GroupPolicy_161.156.80.10 attributes
 vpn-tunnel-protocol ikev1 ikev2
tunnel-group 161.156.80.10 type ipsec-l2l
tunnel-group 161.156.80.10 general-attributes
 default-group-policy GroupPolicy_161.156.80.10
tunnel-group 161.156.80.10 ipsec-attributes
 ikev1 pre-shared-key <Schlüsselwert>
 ikev2 remote-authentication pre-shared-key <Schlüsselwert>
 ikev2 local-authentication pre-shared-key <Schlüsselwert>
```

Erstellen Sie eine IKEv2-Richtlinienkonfiguration für die IPsec-Verbindung. Der IKEv2-Richtlinienblock legt die Parameter für den IKE-Austausch fest. In diesem Block werden die folgenden Parameter festgelegt:
* Verschlüsselungsalgorithmus - Legen Sie für dieses Beispiel den Wert 'AES-256' fest.
* Integritätsalgorithmus - Legen Sie für dieses Beispiel den Wert 'SHA256' fest.
* Diffie-Hellman-Gruppe - IPsec verwendet den Diffie-Hellman-Algorithmus, um den anfänglichen Chiffrierschlüssel zwischen den Peers zu generieren. In diesem Beispiel ist Gruppe 14 festgelegt.
* Pseudozufallsfunktion - IKEv2 erfordert die Verwendung einer separaten Methode, die als Algorithmus benutzt wird, um die für die IKEv2-Tunnelverschlüsselung erforderlichen Operationen der Schlüssel- und Hashverfahren abzuleiten. Dies wird als Pseudozufallsfunktion (PRF, Pseudo-Random Function) bezeichnet und es ist der Wert SHA festgelegt.
* Lebensdauer von Sicherheitszuordnungen - Legen Sie die Lebensdauer der Sicherheitszuordnungen fest (nach deren Erreichen eine Verbindungswiederholung erfolgt). Für die Lebensdauer ist ein Wert von 36.000 Sekunden festgelegt.
* Operationstyp - Behalten Sie den Wert 'bi-directional' als Standardwert bei. (In der Anzeige der aktiven Operationen ist dieser Wert nicht explizit angegeben.)

Wie das folgende Codebeispiel zeigt, verwendet diese Beispielrichtlinie AES-256 zum Verschlüsseln des sicheren Kanals. Mit dem Hashalgorithmus SHA512 wird die Identität des fernen Peers überprüft und die Diffie-Hellman-Gruppe 14 wird zur Schlüsselerstellung verwendet. Gruppe 14 verwendet Verschlüsselungsblöcke von 2048 Bit. Als Lebensdauer für die Sicherheitszuordnung ist schließlich ein Wert von 36.000 Sekunden festgelegt.

```
crypto ikev2 policy 100
encryption aes-256
integrity sha-1
group 14
prf sha
lifetime seconds 36000
```

* Definieren Sie die Zugriffsliste und die Kryptozuordnung für VPN:

```
access-list outside_cryptomap_1 extended permit ip object NETWORK_OBJ_192.168.236.0_24 object vpc 
crypto map outside_map 1 match address outside_cryptomap_1
crypto map outside_map 1 set peer 161.156.80.10 
crypto map outside_map 1 set ikev1 transform-set ESP-AES-128-SHA ESP-AES-128-MD5 ESP-AES-192-SHA ESP-AES-192-MD5 ESP-AES-256-SHA ESP-AES-256-MD5 ESP-3DES-SHA ESP-3DES-MD5 ESP-DES-SHA ESP-DES-MD5
crypto map outside_map 1 set ikev2 ipsec-proposal AES256 AES192 AES 3DES DES
crypto map outside_map interface outside
nat (any,outside) source static NETWORK_OBJ_192.168.236.0_24 NETWORK_OBJ_192.168.236.0_24 destination static vpc vpc no-proxy-arp route-lookup
```

## Vorgehensweise zum Erstellen einer sicheren Verbindung mit der lokalen IBM Cloud-VPC
{: #to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

Zum Erstellen einer sicheren Verbindung erstellen Sie die VPN-Verbindung innerhalb Ihrer VPC, was dem Beispiel mit den 2 VPCs ähnelt.

* Erstellen Sie ein VPN-Gateway auf Ihrem VPC-Teilnetz sowie eine VPN-Verbindung zwischen der VPC und der Cisco ASAv-Einheit. Legen Sie dabei für `local_cidrs` den Wert für das Teilnetz auf der VPC und für `peer_cidrs` den Wert für das Teilnetz auf der Cisco ASAv-Einheit fest.

Für das Gateway wird der Status 'Anstehend' (`pending`) angezeigt, solange das VPN-Gateway erstellt wird. Der Status wechselt zu 'Verfügbar' (`available`), sobald der Erstellungsvorgang abgeschlossen ist. Die Erstellung kann einige Zeit in Anspruch nehmen. 
{:note}


![Abbildungsbeschreibung](./images/vpc-vpn-asav-connection.png)

### Status der sicheren Verbindung prüfen
{: #cisco-check-the-status-of-the-secure-connection}

Sie können den Status Ihrer Verbindung über die IBM Cloud-Konsole prüfen. Außerdem könnten Sie auch versuchen, von Site zu Site ein `Pingsignal` mit den VSIs abzusetzen.

![Abbildungsbeschreibung](./images/vpc-vpn-asav-status.png)
