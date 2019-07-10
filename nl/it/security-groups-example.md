---

copyright:
  years: 2019

lastupdated: "2019-05-14"

keywords: security group, VSI, ping, TCP, outbound, SG4, rules, metadata, setting up

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

# Configurazione dei gruppi di sicurezza utilizzando la CLI
{: #setting-up-security-groups-using-the-cli}

Nel seguente esempio, creerai una VSI con un gruppo di sicurezza abilitato, utilizzando l'interfaccia riga di comando (CLI). Ecco come appare lo scenario.

![Gruppi di sicurezza per IBM VPC](images/security-groups-schematic.svg "Gruppi di sicurezza per IBM VPC")

Nota che nella figura la VSI denominata **SG4** ha un IP mobile `169.60.208.144` assegnato ad essa, in aggiunta al proprio indirizzo VPC interno `10.0.0.5`; pertanto, può comunicare con internet pubblicamente. Il gruppo di sicurezza assegnato alla VSI **SG4** è denominato "demosg".

La VSI denominata **SG8** è soltanto interna al VPC, con un indirizzo IP privato. Il gruppo di sicurezza assegnato alla VSI **SG8** è denominato "my_vpc_sg". Entrambe queste VSI si trovano all'interno del VPC denominato `sgvpc` e anche nella stessa sottorete `10.0.0.0/28` in modo che possano comunicare tra loro.

## Istruzioni per la creazione di una VSI con un gruppo di sicurezza collegato
{: #steps-for-creating-a-vsi-with-a-security-group-attached}

Le regole del gruppo di sicurezza per "my_vpc_sg" includeranno le funzioni di base di SSH, PING e TCP in uscita.

Nota che devi creare prima il gruppo di sicurezza, con il comando `ibmcloud is sgc` e poi la VSI che includerà questo gruppo di sicurezza appena creato.

Questo codice di esempio salta alcuni passi, qui è dove puoi trovare ulteriori informazioni:

 * Le istruzioni per la creazione di un VPC e di una sottorete sono disponibili nel nostro argomento [Creazione di un VPC](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli).

Puoi copiare e incollare i comandi da questo codice CLI di esempio per iniziare a creare una VSI che dispone di un gruppo di sicurezza collegato. Le risposte del sistema non vengono del tutto mostrate in questo codice di esempio. Dovrai aggiornare i tuoi comandi con gli ID della risorsa corretti per il _VPC_, la _sottorete_, l'_immagine_, la _chiave_ e il _numero dell'ID del gruppo di sicurezza_ corretto.

1. Crea un gruppo di sicurezza denominato “my_vpc_sg”

   ```
   ibmcloud is security-group-create my_vpc_sg $vpc
   ```
   {: pre}

   Salva l'ID in una variabile in modo che possiamo utilizzarlo successivamente, ad esempio, nella variabile denominata `sg`:

   ```
   sg=2d364f0a-a870-42c3-a554-000000632953
   ```
   {: pre}

2. Aggiungi le regole che consentono SSH, PING e TCP in uscita

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
ibmcloud is security-group-rule-add $sg outbound tcp
   ```
   {: codeblock}

3. Crea una VSI con il gruppo di sicurezza appena creato

   ```
   ibmcloud is instance-create test-instance $vpc us-south-2 b-4x16 $subnet 1000 \ 
   --image $image --keys $key --security-groups $sg
   ```
   {: pre}

## Scheda di riferimento dell'elenco di comandi
{: #command-list-cheat-sheet}

Per un elenco completo di comandi CLI VPC disponibili per i gruppi di sicurezza, immetti:

```
ibmcloud is help | grep sg
```
{: pre}

Per visualizzare il tuo gruppo di sicurezza e i suoi metadati incluse le regole, dovrai immettere (per l'esempio precedente):

```
ibmcloud is sg $sg
```
{: pre}

Per aggiungere una regola del gruppo di sicurezza, ecco un comando di esempio per l'aggiunta di una regola in entrata PING a un gruppo di sicurezza:

```
ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0

```
{: pre}
