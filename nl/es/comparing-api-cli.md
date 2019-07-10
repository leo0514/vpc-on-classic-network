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


# Comparación entre mandatos para API y para CLI
{: #comparison-of-commands-for-the-api-and-ci}

Este conjunto de tablas ofrece un método para comparar mandatos parecidos que se pueden invocar mediante la API REST o la CLI de IBM Cloud.

## Comparación entre mandatos de API y de CLI para grupos de seguridad
{: #comparing-api-and-cli-commands-for-security-groups}

| Descripción | API | CLI |
|-------------|-----|-----|
|Recuperar el grupo de seguridad predeterminado para la VPC especificada por el identificador en la vía de acceso de URL. | GET /vpcs/{vpc_id}/default_security_group| |
|Recuperar todos los grupos de seguridad existentes de esta cuenta|GET /security_groups| `ibmcloud is security-groups`, `sgs`|
|Recuperar un solo grupo de seguridad especificado por el identificador en la vía de acceso del URL. | GET /security_groups/{id}| `ibmcloud is security-group`, `sg`|
|Crear un nuevo grupo de seguridad a partir de una plantilla de grupo de seguridad. La plantilla está estructurada de la misma forma que un grupo de seguridad recuperado. Contiene la información necesaria para crear el nuevo grupo de seguridad. La plantilla puede incluir una matriz opcional de reglas de grupo de seguridad, que se agregará al grupo. | POST /security_groups| `ibmcloud is security-group-create`, `sgc`|
|Crear una instancia de servidor, con los grupos de seguridad especificados conectados a las interfaces de red del servidor. |POST /instances (con parámetros de grupo de seguridad)| `ibmcloud is instance-create` |
|Actualizar un grupo de seguridad con la información que se proporciona en un objeto de parche del grupo de seguridad. El objeto de parche del grupo de seguridad está estructurado de la misma forma que un grupo de seguridad recuperado. Solo contiene la información que se va a actualizar. | PATCH /security_groups/{id}| `ibmcloud is security-group-update`, `sgu`|
|Recuperar todas las reglas correspondientes a un determinado grupo de seguridad. | GET /security_groups/{security_group_id}/rules| `ibmcloud is security-group-rules`, `sg-rules`|
|Recuperar una sola regla de grupo de seguridad especificada por el identificador en la vía de acceso del URL. | GET /security_groups/{security_group_id}/rules/{id}| `ibmcloud is security-group-rule`, `sg-rule`|
|Crear una nueva regla de grupo de seguridad a partir de una plantilla de regla de grupo de seguridad. El objeto de plantilla de regla está estructurado de la misma forma que una regla de grupo de seguridad recuperado. Contiene la información necesaria para crear la regla. La regla se aplica a todas las interfaces de red del grupo de seguridad. | POST /security_groups/{security_group_id}/rules| `ibmcloud is security-group-rule-add`, `sg-rulec`|
|Actualizar una regla de grupo de seguridad con la información que se proporciona en un objeto de parche de regla. El objeto de parche está estructurado de la misma forma que una regla de grupo de seguridad recuperado. Solo debería contener la información que se va a actualizar. | PATCH /security_groups/{security_group_id}/rules/{id}| |
|Suprimir una regla de grupo de seguridad. Esta supresión no termina ninguna de las conexiones existentes permitidas por dicha regla. Esta operación no se puede invertir. | DELETE /security_groups/{security_group_id}/rules/{id}| `ibmcloud is security-group-rule-delete`, `sg-ruled`|
|Suprimir un grupo de seguridad. Un grupo de seguridad no se puede suprimir si es el grupo de seguridad predeterminado para una VPC o si cualquier otro grupo de seguridad contiene reglas que hacen referencia al mismo como un remoto. Esta operación no se puede invertir. | DELETE /security_groups/{id}| `ibmcloud is security-group-delete`, `sgd`|
|Añadir la interfaz de red existente de un servidor a un grupo de seguridad existente. Esta función aplica las reglas del grupo a dicha interfaz, lo que permite establecer nuevas conexiones, con la interfaz y desde esta, que conforman las reglas.<br /><br />Además, a esta interfaz de red se le otorga el acceso especificado por las reglas de cualquier grupo remoto que haga referencia a este grupo.<br /><br />Consulte las limitaciones a continuación.  | PUT /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface-add`, `sg-nica`|
|Eliminar la interfaz de red de un servidor de un grupo de seguridad. Las reglas del grupo dejan de utilizarse para permitir nuevas conexiones de red en dicha interfaz. <br /><br />Además, a esta interfaz de red se le deja de otorgar el acceso especificado por las reglas de cualquier grupo remoto que haga referencia a este grupo. <br /><br />Las conexiones existentes que estaban permitidas por la pertenencia a este grupo no se interrumpen.| DELETE /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface-remove`, `sg-nicd`|
|Recupere todas las interfaces de red asociadas con el grupo de seguridad. Se trata de las interfaces de red a las que se aplican las reglas del grupo. | GET /security_groups/{security_group_id}/network_interfaces| `ibmcloud is security-group-network-interfaces`, `sg-nics`|
|Recuperar una ola interfaz de red especificada por el identificador en la vía de acceso del URL. La interfaz de red debe ser un miembro existente del grupo de seguridad. | GET /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface`, `sg-nic`|



## Comparación entre mandatos de API y de CLI para ACL
{: #comparing-api-and-cli-commands-for-acls}

| Descripción | API | CLI |
|-------------|-----|-----|
|Crear una ACL |`POST /network_acls` | `ibmcloud is network-acl-create`|
|Recuperar todas las ACL |`GET /network_acls` | `ibmcloud is network-acls`|
|Recuperar una ACL específica| `GET /network_acls/{id}`| `ibmcloud is network-acl`|
|Actualizar una ACL específica| `PATCH /network_acls/{id}`| `ibmcloud is network-acl-update`|
|Suprimir una ACL específica| `DELETE /network_acls/{id}`| `ibmcloud is network-acl-delete`|
|Crear una regla de ACL| `POST /network_acls/{network_acl_id}/rules`| `ibmcloud is network-acl-rule-add`|
|Recuperar todas las reglas de una ACL| `GET /network_acls/{network_acl_id}/rules`| `ibmcloud is network-acl-rules`|
|Recuperar una regla de ACL específica| `GET /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule`|
|Actualizar una regla de ACL específica| `PATCH /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule-update`|
|Suprimir una regla de ACL específica| `DELETE /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule-delete`|
|Adjuntar una ACL a una subred|`PUT /subnets/{id}/network_acl`| `ibmcloud is subnet-network-acl-attach` |
