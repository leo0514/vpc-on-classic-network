---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: security groups, RIAS, API, delete, create

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# Sicherheitsgruppen über die APIs einrichten
{: #setting-up-security-groups-using-the-apis}

Das folgende Beispiel veranschaulicht, wie Sie über die VPC-APIs von {{site.data.keyword.cloud}} Sicherheitsgruppen erstellen und verwalten.

## Voraussetzungen
{: #security-group-prerequisites}

Um Sicherheitsgruppen verwenden zu können, müssen Sie zuerst über eine aktive {{site.data.keyword.cloud_notm}}-VPC verfügen.

Anweisungen zum Erstellen einer VPC und eines Teilnetzes finden Sie im Abschnitt [VPC erstellen](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Schritt 1: Sicherheitsgruppe erstellen
{: #step-1-create-a-security-group}

Erstellen Sie in Ihrer {{site.data.keyword.cloud_notm}}-VPC eine Sicherheitsgruppe namens `my-security-group`.

```
curl -X POST "$rias_endpoint/v1/security_groups?version=2019-05-31&generation=1" \
  -H "Authorization: $iam_token" \
  -d '{
        "name": "my-security-group",
        "vpc": { "id": "'$vpc'" }
      }'
```
{: pre}

Speichern Sie die ID in einer Variablen, sodass sie zu einem späteren Zeitpunkt verwendet werden kann, zum Beispiel in der Variablen namens `sg`:

```
sg=2d364f0a-a870-42c3-a554-000000632953
```
{: pre}

## Schritt 2: Regel hinzufügen, die SSH-Verbindungen zulässt
{: #step-2-add-a-rule-to-allow-ssh-connections}

Erstellen Sie eine Regel für die Sicherheitsgruppe, die eingehende Verbindungen an Port 22 zulässt.

```
curl -X POST "$rias_endpoint/v1/security_groups/$sg/rules?version=2019-05-31&generation=1" \
  -H "Authorization: $iam_token" \
  -d '{
        "direction": "inbound",
        "protocol": "tcp",
        "port_min": 22,
        "port_max": 22
      }'
```
{: pre}

## Schritt 3: Sicherheitsgruppe löschen (optional)
{: #step-3-delete-the-securiy-group-optional}

Damit die Sicherheitsgruppe gelöscht werden kann, darf sie keinen Netzschnittstellen zugeordnet sein und darf auch nicht von Regeln in anderen Sicherheitsgruppen referenziert werden.

```
curl -X DELETE "$rias_endpoint/v1/security_groups/$sg?version=2019-05-31&generation=1" \
  -H "Authorization: $iam_token"
```
{: pre}
