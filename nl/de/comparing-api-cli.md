---

copyright:
  years: 2018, 2019

lastupdated: "2019-05-14"

keywords: API, CLI, commands, comparison, security groups, ACL

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}


# Vergleich der Befehle für die API und die CLI
{: #comparison-of-commands-for-the-api-and-ci}

Diese Gruppe von Tabellen ermöglicht einen Vergleich ähnlicher Befehle, die über die REST-API oder über die IBM Cloud-CLI aufgerufen werden können.

## Vergleich von API- und CLI-Befehlen für Sicherheitsgruppen
{: #comparing-api-and-cli-commands-for-security-groups}

| Beschreibung | API | CLI |
|-------------|-----|-----|
|Ruft die Standardsicherheitsgruppe für die VPC ab, die durch die Kennung im URL-Pfad angegeben ist. | GET /vpcs/{vpc_id}/default_security_group| |
|Ruft alle vorhandenen Sicherheitsgruppen für dieses Konto ab.|GET /security_groups| `ibmcloud is security-groups`, `sgs`|
|Ruft eine einzelne Sicherheitsgruppe ab, die durch die Kennung im URL-Pfad angegeben ist. | GET /security_groups/{ID}| `ibmcloud is security-group`, `sg`|
|Erstellt eine neue Sicherheitsgruppe aus einer Vorlage für Sicherheitsgruppen. Die Vorlage ist auf dieselbe Art strukturiert wie eine abgerufene Sicherheitsgruppe. Sie enthält die Informationen, die zum Erstellen der neuen Sicherheitsgruppe erforderlich sind. Die Vorlage kann ein optionales Array von Sicherheitsgruppenregeln einschließen, die zur Gruppe hinzugefügt werden. | POST /security_groups| `ibmcloud is security-group-create`, `sgc`|
|Erstellt eine Serverinstanz, wobei die angegebenen vorhandenen Sicherheitsgruppen an die Netzschnittstellen des Servers angehängt werden. |POST /instances (mit Sicherheitsgruppenparametern)| `ibmcloud is instance-create` |
|Aktualisiert eine Sicherheitsgruppe mit den Informationen, die von einem Patchobjekt für Sicherheitsgruppen bereitgestellt werden. Das Patchobjekt für die Sicherheitsgruppe ist auf dieselbe Art strukturiert wie eine abgerufene Sicherheitsgruppe. Es enthält nur die Informationen, die aktualisiert werden sollen. | PATCH /security_groups/{ID}| `ibmcloud is security-group-update`, `sgu`|
|Ruft alle Regeln für eine bestimmte Sicherheitsgruppe ab. | GET /security_groups/{sicherheitsgruppen-ID}/rules| `ibmcloud is security-group-rules`, `sg-rules`|
|Ruft eine einzelne Sicherheitsgruppenregel ab, die durch die Kennung im URL-Pfad angegeben ist. | GET /security_groups/{sicherheitsgruppen-ID}/rules/{ID}| `ibmcloud is security-group-rule`, `sg-rule`|
|Erstellt eine neue Sicherheitsgruppenregel aus einer Vorlage für Sicherheitsgruppenregeln. Das Regelvorlagenobjekt ist auf dieselbe Art strukturiert wie eine abgerufene Sicherheitsgruppenregel. Sie enthält die Informationen, die zum Erstellen der Regel erforderlich sind. Die Regel wird auf alle Netzschnittstellen in der Sicherheitsgruppe angewendet. | POST /security_groups/{sicherheitsgruppen-ID}/rules| `ibmcloud is security-group-rule-add`, `sg-rulec`|
|Aktualisiert eine Sicherheitsgruppenregel mit den Informationen, die von einem Patchobjekt für Regeln bereitgestellt werden. Das Patchobjekt ist auf dieselbe Art strukturiert wie eine abgerufene Sicherheitsgruppenregel. Es darf nur die Informationen enthalten, die aktualisiert werden sollen. | PATCH /security_groups/{sicherheitsgruppen-ID}/rules/{ID}| |
|Löscht eine Sicherheitsgruppenregel. Dieser Löschvorgang bewirkt nicht die Beendigung bestehender Verbindungen, die durch diese Regel erlaubt waren. Dieser Vorgang kann nicht rückgängig gemacht werden. | DELETE /security_groups/{sicherheitsgruppen-ID}/rules/{ID}| `ibmcloud is security-group-rule-delete`, `sg-ruled`|
|Löscht eine Sicherheitsgruppe. Eine Sicherheitsgruppe kann nicht gelöscht werden, wenn sie als Standardsicherheitsgruppe für eine VPC dient oder wenn andere Sicherheitsgruppen Regeln enthalten, die auf sie als ferne Regeln verweisen. Dieser Vorgang kann nicht rückgängig gemacht werden. | DELETE /security_groups/{ID}| `ibmcloud is security-group-delete`, `sgd`|
|Fügt die vorhandene Netzschnittstelle eines Servers zu einer vorhandenen Sicherheitsgruppe hinzu. Diese Funktion wendet die Regeln der Gruppe auf diese Schnittstelle an, sodass neue Verbindungen, die diesen Regeln entsprechen, zu und von der Schnittstelle hergestellt werden können.<br /><br />Darüber hinaus erhält diese Netzschnittstelle nun den Zugriff, wie er durch die Regeln jeglicher entfernter Gruppen, die sich auf diese Gruppe beziehen, festgelegt ist.<br /><br />Die entsprechenden Einschränkungen sind unten aufgeführt.  | PUT /security_groups/{sicherheitsgruppen-ID}/network_interfaces/{ID}| `ibmcloud is security-group-network-interface-add`, `sg-nica`|
|Entfernt die Netzschnittstelle eines Servers aus einer Sicherheitsgruppe. Die Regeln der Gruppe werden nicht mehr zum Zulassen neuer Netzverbindungen auf dieser Schnittstelle verwendet. <br /><br />Darüber hinaus erhält diese Netzschnittstelle nicht mehr den Zugriff, wie er durch die Regeln jeglicher entfernter Gruppen, die sich auf diese Gruppe beziehen, festgelegt ist. <br /><br />Bestehende Verbindungen, die durch die Zugehörigkeit zu dieser Gruppe zugelassen wurden, werden nicht beendet.| DELETE /security_groups/{sicherheitsgruppen-ID}/network_interfaces/{ID}| `ibmcloud is security-group-network-interface-remove`, `sg-nicd`|
|Ruft alle Netzschnittstellen ab, die der Sicherheitsgruppe zugeordnet sind. Dabei handelt es sich um diejenigen Netzschnittstellen, auf die die Regeln in der Gruppe angewendet werden. | GET /security_groups/{sicherheitsgruppen-ID}/network_interfaces| `ibmcloud is security-group-network-interfaces`, `sg-nics`|
|Ruft eine einzelne Netzschnittstelle ab, die durch die Kennung im URL-Pfad angegeben ist. Die Netzschnittstelle muss der Sicherheitsgruppe als vorhandenes Mitglied angehören. | GET /security_groups/{sicherheitsgruppen-ID}/network_interfaces/{ID}| `ibmcloud is security-group-network-interface`, `sg-nic`|



## Vergleich von API- und CLI-Befehlen für Zugriffssteuerungslisten (ACLs)
{: #comparing-api-and-cli-commands-for-acls}

| Beschreibung | API | CLI |
|-------------|-----|-----|
|Erstellen einer ACL |`POST /network_acls` | `ibmcloud is network-acl-create`|
|Abrufen aller ACLs |`GET /network_acls` | `ibmcloud is network-acls`|
|Abrufen einer bestimmten ACL| `GET /network_acls/{ID}`| `ibmcloud is network-acl`|
|Aktualisieren einer bestimmten ACL| `PATCH /network_acls/{ID}`| `ibmcloud is network-acl-update`|
|Löschen einer bestimmten ACL| `DELETE /network_acls/{ID}`| `ibmcloud is network-acl-delete`|
|Erstellen einer ACL-Regel| `POST /network_acls/{netz-ACL-ID}/rules`| `ibmcloud is network-acl-rule-add`|
|Abrufen aller Regeln einer ACL| `GET /network_acls/{netz-ACL-ID}/rules`| `ibmcloud is network-acl-rules`|
|Abrufen einer bestimmten ACL-Regel| `GET /network_acls/{netz-ACL-ID}/rules/{ID}`| `ibmcloud is network-acl-rule`|
|Aktualisieren einer bestimmten ACL-Regel| `PATCH /network_acls/{netz-ACL-ID}/rules/{ID}`| `ibmcloud is network-acl-rule-update`|
|Löschen einer bestimmten ACL-Regel| `DELETE /network_acls/{netz-ACL-ID}/rules/{ID}`| `ibmcloud is network-acl-rule-delete`|
|Anhängen einer ACL an ein Teilnetz|`PUT /subnets/{ID}/network_acl`| `ibmcloud is subnet-network-acl-attach` |
