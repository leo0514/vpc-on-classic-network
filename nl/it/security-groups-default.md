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

# Aggiornamento del gruppo di sicurezza predefinito
{: #updating-the-default-security-group}


Il gruppo di sicurezza predefinito è molto simile a qualsiasi altro gruppo di sicurezza, con l'eccezione che non può essere eliminato.

Ogni VPC ha un gruppo di sicurezza predefinito, con delle regole per consentire:

* Il traffico in entrata da tutti i membri del gruppo.
* Tutto il traffico in uscita.

Se modifichi le regole del gruppo di sicurezza predefinito, tali regole modificate vengono poi applicate a tutti i server correnti e futuri nel gruppo.

Le regole in entrata per consentire ping e SSH non vengono automaticamente aggiunte al gruppo di sicurezza predefinito. Puoi modificare le regole del gruppo di sicurezza predefinito utilizzando l'API REST, l'`ibmcloud cli` o l'IU.

## Esempio: Modifica delle regole del gruppo di sicurezza predefinito utilizzando la CLI
{: #example-modifying-the-default-security-group-rules-using-the-cli}

1. Accedi a IBM Cloud.

   Se hai un account federato:
   ```
   ibmcloud login -sso
   ```
   {: pre}

   Altrimenti utilizza questo comando:

   ```
   ibmcloud login
   ```
   {: pre}

2. Ottieni i dettagli e l'ID del gruppo di sicurezza predefinito per il VPC

   Immetti il seguente comando CLI per elencare tutti i VPC:

   ```
   ibmcloud is vpc
   ```
   {: pre}

   Il nome del gruppo di sicurezza predefinito viene mostrato nella colonna `Default Security Group`. Prendi nota del nome in modo da poter trovare l'`ID` quando elenchi i gruppi di sicurezza (successivamente). 
   
   Ora elenca tutti i gruppi di sicurezza nel VPC:

   ```
   ibmcloud is security-groups
   ```
   {: pre}

   Salva l'ID del gruppo di sicurezza (per il gruppo di sicurezza predefinito) in una variabile in modo da poterlo utilizzare successivamente. Ad esempio, utilizzando il nome della variabile `sg`:

   ```
   sg=2d364f0a-a870-42c3-a554-000001162469
   ```
   {: pre}

   Per ottenere i dettagli sul gruppo di sicurezza, immetti il seguente comando CLI:

   ```
   ibmcloud is security-group $sg
   ```
   {: pre}
   
   In alternativa, potresti inserire il valore ID reale al posto della variabile `$sg`.

3. Aggiorna il gruppo di sicurezza predefinito -- aggiungi le regole che consentono SSH e PING

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
   ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
   ```
   {: codeblock}


L'aggiunta e la rimozione delle regole del gruppo di sicurezza sono delle operazioni asincrone. Normalmente servono da 1 a 30 secondi perché la modifica diventi effettiva.
{: note}
