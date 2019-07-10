---
copyright:
  years: 2019
lastupdated: "2019-05-14"

keywords: security groups, traffic, firewall, stateful, filtering, access, control, list, ACL, stateless, traffic, resource

subcollection: vpc-on-classic-network
---

# Sicherheitsgruppen und Zugriffskontrolllisten vergleichen
{: #compare-security-groups-and-access-control-lists}

Sicherheitsgruppen und Zugriffssteuerungslisten bieten Möglichkeiten zur Steuerung des Datenverkehrs in den {{site.data.keyword.cloud}} VPC-Teilnetzen und -Instanzen mithilfe von Regeln, die Sie angeben.

Die folgende Tabelle enthält eine Zusammenfassung einiger der wichtigsten Unterschiede zwischen Sicherheitsgruppen und ACLs:

|  | Sicherheitsgruppen | Zugriffssteuerungslisten (ACLs)    |
|-------------|-----------------|---------|
| Steuerebene  | Virtuelle Serverinstanz (VSI)    | Teilnetz  |
| Zustand   | Statusabhängig - Sobald eine eingehende Verbindung zulässig ist, kann sie antworten. | Statusunabhängig - Sowohl eingehende als auch abgehende Verbindungen müssen explizit zulässig sein. |
| Unterstützte Regeln | Verwendet nur Regeln zum Zulassen | Verwendet Regeln zum Zulassen wie auch zum Zurückweisen|
| Anwendungsweise von Regeln | Alle Regeln werden berücksichtigt | Regeln werden der Reihe nach verarbeitet |
| Beziehung zur zugeordneten Ressource | Eine Instanz kann mehreren Sicherheitsgruppen zugeordnet sein| Derselben ACL können mehrere Teilnetze zugeordnet sein|
