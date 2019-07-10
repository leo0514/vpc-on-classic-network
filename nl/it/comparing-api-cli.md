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


# Confronto tra i comandi per l'API e la CLI
{: #comparison-of-commands-for-the-api-and-ci}

Questa serie di tabelle offre un modo per confrontare dei comandi simili che possono essere richiamati tramite l'API REST o la CLI IBM Cloud.

## Confronto dei comandi API e CLI per i gruppi di sicurezza
{: #comparing-api-and-cli-commands-for-security-groups}

| Descrizione | API | CLI |
|-------------|-----|-----|
|Richiama il gruppo di sicurezza predefinito per il VPC specificato dall'identificatore nel percorso URL. | GET /vpcs/{vpc_id}/default_security_group| |
|Richiama tutti i gruppi di sicurezza esistenti dell'account|GET /security_groups| `ibmcloud is security-groups`, `sgs`|
|Richiama un solo gruppo di sicurezza specificato dall'identificatore nel percorso URL. | GET /security_groups/{id}| `ibmcloud is security-group`, `sg`|
|Crea un nuovo gruppo di sicurezza da un template del gruppo di sicurezza. Il template è strutturato nello stesso modo di un gruppo di sicurezza richiamato. Contiene le informazioni necessarie per creare il nuovo gruppo di sicurezza. Il template può includere un array facoltativo di regole del gruppo di sicurezza, che sarà aggiunto al gruppo. | POST /security_groups| `ibmcloud is security-group-create`, `sgc`|
|Crea un'istanza del server, con i gruppi di sicurezza esistenti specificati collegati alle interfacce di rete del server. |POST /instances (con i parametri del gruppo di sicurezza)| `ibmcloud is instance-create` |
|Aggiorna un gruppo di sicurezza con le informazioni fornite in un oggetto patch del gruppo di sicurezza. L'oggetto patch del gruppo di sicurezza è strutturato nello stesso modo di un gruppo di sicurezza richiamato. Contiene solo le informazioni da aggiornare. | PATCH /security_groups/{id}| `ibmcloud is security-group-update`, `sgu`|
|Richiama tutte le regole per un gruppo di sicurezza particolare. | GET /security_groups/{security_group_id}/rules| `ibmcloud is security-group-rules`, `sg-rules`|
|Richiama una sola regola del gruppo di sicurezza specificata dall'identificatore nel percorso URL. | GET /security_groups/{security_group_id}/rules/{id}| `ibmcloud is security-group-rule`, `sg-rule`|
|Crea una nuova regola del gruppo di sicurezza da un template di regole del gruppo di sicurezza. L'oggetto template di regole è strutturato nello stesso modo di una regola del gruppo di sicurezza richiamata. Contiene le informazioni necessarie per creare la regola. La regola viene applicata a tutte le interfacce di rete nel gruppo di sicurezza. | POST /security_groups/{security_group_id}/rules| `ibmcloud is security-group-rule-add`, `sg-rulec`|
|Aggiorna una regola del gruppo di sicurezza con le informazioni fornite in un oggetto patch della regola. L'oggetto patch è strutturato nello stesso modo di una regola del gruppo di sicurezza richiamata. Dovrebbe contenere solo le informazioni da aggiornare. | PATCH /security_groups/{security_group_id}/rules/{id}| |
|Elimina una regola del gruppo di sicurezza. Questa eliminazione non termina alcuna connessione esistente che è stata consentita da tale regola. Questa operazione non può essere annullata. | DELETE /security_groups/{security_group_id}/rules/{id}| `ibmcloud is security-group-rule-delete`, `sg-ruled`|
|Elimina un gruppo di sicurezza. Un gruppo di sicurezza non può essere eliminato se è il gruppo di sicurezza predefinito per un VPC o se un altro qualsiasi gruppo di sicurezza contiene delle regole che fanno riferimento a esso in remoto. Questa operazione non può essere annullata. | DELETE /security_groups/{id}| `ibmcloud is security-group-delete`, `sgd`|
|Aggiunge l'interfaccia di rete esistente di un server a un gruppo di sicurezza esistente. Questa funzione applica le regole del gruppo a tale interfaccia, consentendo di creare nuove connessioni, da e verso l'interfaccia, conformi a queste regole.<br /><br />Inoltre, a questa interfaccia di rete viene ora concesso l'accesso specificato dalle regole di tutti i gruppi remoti che fanno riferimento a questo gruppo.<br /><br />Consulta le seguenti limitazioni.  | PUT /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface-add`, `sg-nica`|
|Rimuove l'interfaccia di rete di un server da un gruppo di sicurezza. Le regole del gruppo non vengono più utilizzate per consentire delle nuove connessione di rete su tale interfaccia. <br /><br />Inoltre, a questa interfaccia di rete non viene più concesso l'accesso specificato dalle regole di tutti i gruppi remoti che fanno riferimento a questo gruppo. <br /><br />Le connessioni esistenti che sono state consentite dall'appartenenza a questo gruppo non vengono terminate.| DELETE /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface-remove`, `sg-nicd`|
|Richiama tutte le interfacce di rete associate con il gruppo di sicurezza. Queste sono le interfacce di rete a cui vengono applicate le regole nel gruppo. | GET /security_groups/{security_group_id}/network_interfaces| `ibmcloud is security-group-network-interfaces`, `sg-nics`|
|Richiama una singola interfaccia di rete specificata dall'identificatore nel percorso URL. L'interfaccia di rete deve essere un membro esistente del gruppo di sicurezza. | GET /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface`, `sg-nic`|



## Confronto dei comandi API e CLI per gli ACL
{: #comparing-api-and-cli-commands-for-acls}

| Descrizione | API | CLI |
|-------------|-----|-----|
|Crea un ACL |`POST /network_acls` | `ibmcloud is network-acl-create`|
|Richiama tutti gli ACL |`GET /network_acls` | `ibmcloud is network-acls`|
|Richiama un ACL specifico| `GET /network_acls/{id}`| `ibmcloud is network-acl`|
|Aggiorna un ACL specifico| `PATCH /network_acls/{id}`| `ibmcloud is network-acl-update`|
|Elimina un ACL specifico| `DELETE /network_acls/{id}`| `ibmcloud is network-acl-delete`|
|Crea una regola dell'ACL| `POST /network_acls/{network_acl_id}/rules`| `ibmcloud is network-acl-rule-add`|
|Richiama tutte le regole di un ACL| `GET /network_acls/{network_acl_id}/rules`| `ibmcloud is network-acl-rules`|
|Richiama una regola dell'ACL specifica| `GET /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule`|
|Aggiorna una regola dell'ACL specifica| `PATCH /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule-update`|
|Elimina una regola dell'ACL specifica| `DELETE /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule-delete`|
|Collega un ACL a una sottorete|`PUT /subnets/{id}/network_acl`| `ibmcloud is subnet-network-acl-attach` |
