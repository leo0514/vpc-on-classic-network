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


# Comparaison des commandes pour l'API et l'interface de ligne de commande
{: #comparison-of-commands-for-the-api-and-ci}

Cet ensemble de tableaux permet de comparer des commandes similaires qui peuvent être appelées via l'API REST ou l'interface de ligne de commande IBM Cloud.

## Comparaison des commandes de l'API et de l'interface de ligne de commande pour les groupes de sécurité
{: #comparing-api-and-cli-commands-for-security-groups}

| Description | API | Interface de ligne de commande |
|-------------|-----|-----|
|Extrait le groupe de sécurité par défaut pour le VPC spécifié par l'identificateur dans le chemin de l'URL. | GET /vpcs/{vpc_id}/default_security_group| |
|Extrait tous les groupes de sécurité existants de ce compte.|GET /security_groups| `ibmcloud is security-groups`, `sgs`|
|Extrait un seul groupe de sécurité spécifié par l'identificateur dans le chemin de l'URL. | GET /security_groups/{id}| `ibmcloud is security-group`, `sg`|
|Crée un groupe de sécurité à partir d'un modèle de groupe de sécurité. Le modèle est structuré de la même façon qu'un groupe de sécurité extrait. Il contient les informations nécessaires à la création du groupe de sécurité. Le modèle peut inclure un tableau facultatif des règles du groupe de sécurité, qui seront ajoutées au groupe. | POST /security_groups| `ibmcloud is security-group-create`, `sgc`|
|Crée une instance de serveur, en joignant les groupes de sécurité existants spécifiés aux interfaces réseau du serveur. |POST /instances (avec les paramètres du groupe de sécurité)| `ibmcloud is instance-create` |
|Met à jour un groupe de sécurité avec les informations fournies dans un objet de correctif du groupe de sécurité. L'objet de correctif du groupe de sécurité est structuré de la même façon qu'un groupe de sécurité extrait. Il contient uniquement les informations à mettre à jour. | PATCH /security_groups/{id}| `ibmcloud is security-group-update`, `sgu`|
|Extrait toutes les règles pour un groupe de sécurité spécifique. | GET /security_groups/{security_group_id}/rules| `ibmcloud is security-group-rules`, `sg-rules`|
|Extrait une seule règle de groupe de sécurité spécifiée par l'identificateur dans le chemin de l'URL. | GET /security_groups/{security_group_id}/rules/{id}| `ibmcloud is security-group-rule`, `sg-rule`|
|Crée une règle de groupe de sécurité à partir d'un modèle de règle de groupe de sécurité. L'objet de modèle de règle est structuré de la même façon qu'une règle de groupe de sécurité extraite. Il contient les informations nécessaires à la création de la règle. La règle s'applique à toutes les interfaces de mise en réseau dans le groupe de sécurité. | POST /security_groups/{security_group_id}/rules| `ibmcloud is security-group-rule-add`, `sg-rulec`|
|Met à jour une règle de groupe de sécurité avec les informations fournies dans un objet de correctif de règle. L'objet de correctif est structuré de la même façon qu'une règle de groupe de sécurité extraite. Il doit contenir uniquement les informations à mettre à jour. | PATCH /security_groups/{security_group_id}/rules/{id}| |
|Supprime une règle de groupe de sécurité. Cette suppression ne met pas fin aux connexions existantes qui ont été autorisées par cette règle. Cette opération est irréversible. | DELETE /security_groups/{security_group_id}/rules/{id}| `ibmcloud is security-group-rule-delete`, `sg-ruled`|
|Supprime un groupe de sécurité. Un groupe de sécurité ne peut pas être supprimé s'il s'agit du groupe de sécurité par défaut d'un VPC ou si d'autres groupes de sécurité contiennent des règles qui y font référence à distance. Cette opération est irréversible. | DELETE /security_groups/{id}| `ibmcloud is security-group-delete`, `sgd`|
|Ajoute l'interface réseau existante d'un serveur à un groupe de sécurité existant. Cette fonction applique les règles du groupe à cette interface, permettant ainsi l'établissement de nouvelles connexions, vers et depuis l'interface, qui respectent ces règles.<br /><br />En outre, l'accès spécifié par les règles de groupes distants qui font référence à ce groupe est désormais accordé à cette interface réseau.<br /><br />Consultez les limitations ci-dessous.  | PUT /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface-add`, `sg-nica`|
|Supprime l'interface réseau d'un serveur d'un groupe de sécurité. Les règles du groupe ne sont plus utilisées pour autoriser de nouvelles connexions réseau sur cette interface. <br /><br />En outre, l'accès spécifié par les règles de groupes distants qui font référence à ce groupe n'est plus accordé à cette interface réseau. <br /><br />Il n'est pas mis fin aux connexions existantes qui étaient autorisées par l'appartenance à ce groupe.| DELETE /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface-remove`, `sg-nicd`|
|Extrait toutes les interfaces réseau associées au groupe de sécurité. Il s'agit des interfaces réseau auxquelles les règles du groupe sont appliquées. | GET /security_groups/{security_group_id}/network_interfaces| `ibmcloud is security-group-network-interfaces`, `sg-nics`|
|Extrait une seule interface réseau spécifiée par l'identificateur dans le chemin de l'URL. L'interface réseau doit être membre du groupe de sécurité. | GET /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface`, `sg-nic`|



## Comparaison des commandes de l'API et de l'interface de ligne de commande pour les listes de contrôle d'accès (ACL)
{: #comparing-api-and-cli-commands-for-acls}

| Description | API | Interface de ligne de commande |
|-------------|-----|-----|
|Crée une ACL |`POST /network_acls` | `ibmcloud is network-acl-create`|
|Extrait toutes les ACL |`GET /network_acls` | `ibmcloud is network-acls`|
|Extrait une ACL spécifique| `GET /network_acls/{id}`| `ibmcloud is network-acl`|
|Met à jour une ACL spécifique| `PATCH /network_acls/{id}`| `ibmcloud is network-acl-update`|
|Supprime une ACL spécifique| `DELETE /network_acls/{id}`| `ibmcloud is network-acl-delete`|
|Crée une règle de liste de contrôle d'accès| `POST /network_acls/{network_acl_id}/rules`| `ibmcloud is network-acl-rule-add`|
|Extrait toutes les règles d'une ACL| `GET /network_acls/{network_acl_id}/rules`| `ibmcloud is network-acl-rules`|
|Extrait une règle de liste de contrôle d'accès spécifique| `GET /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule`|
|Met à jour une règle de liste de contrôle d'accès spécifique| `PATCH /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule-update`|
|Supprime une règle de liste de contrôle d'accès spécifique| `DELETE /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule-delete`|
|Joint une ACL à un sous-réseau|`PUT /subnets/{id}/network_acl`| `ibmcloud is subnet-network-acl-attach` |
