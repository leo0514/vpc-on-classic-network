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


# Comparação de comandos para a API e a CLI
{: #comparison-of-commands-for-the-api-and-ci}

Esse conjunto de tabelas oferece uma maneira de comparar comandos semelhantes que podem ser chamados por meio da API de REST ou por meio da CLI do IBM Cloud.

## Comparando comandos da API e da CLI para grupos de segurança
{: #comparing-api-and-cli-commands-for-security-groups}

| Descrição | API | CLI |
|-------------|-----|-----|
|Recuperar o grupo de segurança padrão para a VPC especificada pelo identificador no caminho da URL. | GET /vpcs/{vpc_id}/default_security_group| |
|Recuperar todos os grupos de segurança existentes da conta|GET /security_groups| `ibmcloud is security-groups`, `sgs`|
|Recuperar um único grupo de segurança especificado pelo identificador no caminho da URL. | GET /security_groups/{id}| `ibmcloud is security-group`, `sg`|
|Criar um novo grupo de segurança por meio de um modelo do grupo de segurança. O modelo é estruturado da mesma maneira que um grupo de segurança recuperado. Ele contém as informações necessárias para criar o novo grupo de segurança. O modelo pode incluir uma matriz opcional de regras do grupo de segurança, que será incluída no grupo. | POST /security_groups| `ibmcloud is security-group-create`, `sgc`|
|Criar uma instância do servidor, com os grupos de segurança existentes especificados anexados às interfaces de rede do servidor. |POST /instances (com parâmetros do grupo de segurança)| `ibmcloud is instance-create` |
|Atualizar um grupo de segurança com as informações fornecidas em um objeto de correção do grupo de segurança. O objeto de correção do grupo de segurança é estruturado da mesma maneira que um grupo de segurança recuperado. Ele contém somente as informações a serem atualizadas. | PATCH /security_groups/{id}| `ibmcloud is security-group-update`, `sgu`|
|Recuperar todas as regras para um grupo de segurança específico. | GET /security_groups/{security_group_id}/rules| `ibmcloud is security-group-rules`, `sg-rules`|
|Recuperar uma única regra do grupo de segurança especificada pelo identificador no caminho da URL. | GET /security_groups/{security_group_id}/rules/{id}| `ibmcloud is security-group-rule`, `sg-rule`|
|Criar uma nova regra do grupo de segurança por meio de um modelo de regra do grupo de segurança. O objeto de modelo de regra é estruturado da mesma maneira que uma regra do grupo de segurança recuperada. Ele contém as informações necessárias para criar a regra. A regra é aplicada a todas as interfaces de rede no grupo de segurança. | POST /security_groups/{security_group_id}/rules| `ibmcloud is security-group-rule-add`, `sg-rulec`|
|Atualizar uma regra do grupo de segurança com as informações fornecidas em um objeto de correção de regra. O objeto de correção é estruturado da mesma maneira que uma regra do grupo de segurança recuperada. Ele deve conter somente as informações a serem atualizadas. | PATCH /security_groups/{security_group_id}/rules/{id}| |
|Excluir uma regra do grupo de segurança. Essa exclusão não finaliza nenhuma conexão existente que tenha sido permitida por essa regra. Essa operação não pode ser revertida. | DELETE /security_groups/{security_group_id}/rules/{id}| `ibmcloud is security-group-rule-delete`, `sg-ruled`|
|Excluir um grupo de segurança. Um grupo de segurança não poderá ser excluído se ele for o grupo de segurança padrão para uma VPC ou se qualquer outro grupo de segurança contiver regras que se refiram a ele como remoto. Essa operação não pode ser revertida. | DELETE /security_groups/{id}| `ibmcloud is security-group-delete`, `sgd`|
|Incluir a interface de rede existente de um servidor em um grupo de segurança existente. Essa função aplica as regras do grupo a essa interface, permitindo que novas conexões sejam feitas, para e da interface, que se adéquam a essas regras.<br /><br />Além disso, essa interface de rede agora tem acesso ao acesso especificado pelas regras de quaisquer grupos remotos que se referem a esse grupo.<br /><br />Veja Limitações, abaixo.  | PUT /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface-add`, `sg-nica`|
|Remover a interface de rede de um servidor de um grupo de segurança. As regras do grupo não são mais usadas para permitir novas conexões nessa interface. <br /><br />Além disso, essa interface de rede não recebe mais o acesso especificado pelas regras de quaisquer grupos remotos que se referem a esse grupo. <br /><br />As conexões existentes que foram permitidas pela associação desse grupo não foram terminadas.| DELETE /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface-remove`, `sg-nicd`|
|Recuperar todas as interfaces de rede associadas ao grupo de segurança. Essas são as interfaces de rede às quais as regras do grupo são aplicadas. | GET /security_groups/{security_group_id}/network_interfaces| `ibmcloud is security-group-network-interfaces`, `sg-nics`|
|Recuperar uma única interface de rede especificada pelo identificador no caminho da URL. A interface de rede deve ser um membro existente do grupo de segurança. | GET /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface`, `sg-nic`|



## Comparando comandos da API e da CLI para ACLs
{: #comparing-api-and-cli-commands-for-acls}

| Descrição | API | CLI |
|-------------|-----|-----|
|Cria uma ACL |`POST /network_acls` | `ibmcloud is network-acl-create`|
|Recupera todas as ACLs |`GET /network_acls` | `ibmcloud is network-acls`|
|Recupera uma ACL específica| `GET /network_acls/{id}`| `ibmcloud is network-acl`|
|Atualiza uma ACL específica| `PATCH /network_acls/{id}`| `ibmcloud is network-acl-update`|
|Exclui uma ACL específica| `DELETE /network_acls/{id}`| `ibmcloud is network-acl-delete`|
|Cria uma regra de ACL| `POST /network_acls/{network_acl_id}/rules`| `ibmcloud is network-acl-rule-add`|
|Recupera todas as regras de uma ACL| `GET /network_acls/{network_acl_id}/rules`| `ibmcloud is network-acl-rules`|
|Recupera uma regra de ACL específica| `GET /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule`|
|Atualiza uma regra de ACL específica| `PATCH /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule-update`|
|Exclui uma regra de ACL específica| `DELETE /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule-delete`|
|Anexa uma ACL a uma sub-rede|`PUT /subnets/{id}/network_acl`| `ibmcloud is subnet-network-acl-attach` |
