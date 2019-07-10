---
copyright:
  years: 2019
lastupdated: "2019-05-14"

keywords: security groups, traffic, firewall, stateful, filtering, access, control, list, ACL, stateless, traffic, resource

subcollection: vpc-on-classic-network
---

# Confronto tra gruppi di sicurezza e ACL (Access Control List)
{: #compare-security-groups-and-access-control-lists}

I gruppi di sicurezza e gli ACL (Access Control List) forniscono dei modi per controllare il traffico tra le sottoreti e le istanze nel tuo {{site.data.keyword.cloud}} VPC, utilizzando le regole che specifichi.

La seguente tabella riepiloga alcune differenze chiave tra i gruppi di sicurezza e gli ACL:

|  | Gruppi di sicurezza | ACL    |
|-------------|-----------------|---------|
| Livello di controllo  | Istanza VSI    | Sottorete  |
| Stato   | Con stato - Una volta abilitata una connessione in entrata, è consentita la risposta | Senza stato - Sia le connessioni in entrata che in uscita devono essere esplicitamente consentite |
| Regole supportate | Utilizza solo le regole di assenso | Utilizza sia le regole di assenso che di negazione|
| Modalità di applicazione delle regole | Vengono considerate tutte le regole | Le regole vengono elaborate in sequenza |
| Relazione con la risorsa associata | Un'istanza può essere associata a più gruppi di sicurezza| Più sottoreti possono essere associate allo stesso ACL|
