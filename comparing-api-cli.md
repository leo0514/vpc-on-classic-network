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


# Comparison of commands for the API and CLI
{: #comparison-of-commands-for-the-api-and-ci}

This set of tables offers a way to compare similar commands that can be called through the REST API or through the IBM Cloud CLI.

## Comparing API and CLI commands for security groups
{: #comparing-api-and-cli-commands-for-security-groups}

| Description | API | CLI |
|-------------|-----|-----|
|Retrieve the default security group for the VPC specified by the identifier in the URL path. | GET /vpcs/{vpc_id}/default_security_group| |
|Retrieve all this account's existing security groups|GET /security_groups| `ibmcloud is security-groups`, `sgs`|
|Retrieve a single security group specified by the identifier in the URL path. | GET /security_groups/{id}| `ibmcloud is security-group`, `sg`|
|Create a new security group from a security group template. The template is structured in the same way as a retrieved security group. It contains the information necessary to create the new security group. The template may include an optional array of security group rules, which will be added to the group. | POST /security_groups| `ibmcloud is security-group-create`, `sgc`|
|Create a server instance, with the specified existing security groups attached to the server's network interfaces. |POST /instances (with security group parameters)| `ibmcloud is instance-create` |
|Update a security group with the information provided in a security group patch object. The security group patch object is structured in the same way as a retrieved security group. It contains only the information to be updated. | PATCH /security_groups/{id}| `ibmcloud is security-group-update`, `sgu`|
|Retrieve all the rules for a particular security group. | GET /security_groups/{security_group_id}/rules| `ibmcloud is security-group-rules`, `sg-rules`|
|Retrieve a single security group rule specified by the identifier in the URL path. | GET /security_groups/{security_group_id}/rules/{id}| `ibmcloud is security-group-rule`, `sg-rule`|
|Create a new security group rule from a security group rule template. The rule template object is structured in the same way as a retrieved security group rule. It contains the information necessary to create the rule. The rule is applied to all the networking interfaces in the security group. | POST /security_groups/{security_group_id}/rules| `ibmcloud is security-group-rule-add`, `sg-rulec`|
|Update a security group rule with the information provided in a rule patch object. The patch object is structured in the same way as a retrieved security group rule. It should contain only the information to be updated. | PATCH /security_groups/{security_group_id}/rules/{id}| |
|Delete a security group rule. This deletion does not terminate any existing connections that were allowed by that rule. This operation cannot be reversed. | DELETE /security_groups/{security_group_id}/rules/{id}| `ibmcloud is security-group-rule-delete`, `sg-ruled`|
|Delete a security group. A security group cannot be deleted if it is the default security group for a VPC or if any other security groups contain rules that refer to it as a remote. This operation cannot be reversed. | DELETE /security_groups/{id}| `ibmcloud is security-group-delete`, `sgd`|
|Add an server's existing network interface to an existing security group. This function applies the group's rules to that interface, allowing new connections to be made, to and from the interface, that conform to those rules.<br /><br />In addition, this network interface is now granted the access specified by the rules of any remote groups that refer to this group.<br /><br />See Limitations, below.  | PUT /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface-add`, `sg-nica`|
|Remove a server's network interface from a security group. The group's rules are no longer used to permit new network connections on that interface. <br /><br />In addition, this network interface is no longer granted the access specified by the rules of any remote groups that refer to this group. <br /><br />Existing connections that were permitted by membership of this group are not ended.| DELETE /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface-remove`, `sg-nicd`|
|Retrieve all the the network interfaces associated with the security group. These are the network interfaces to which the rules in the group are applied. | GET /security_groups/{security_group_id}/network_interfaces| `ibmcloud is security-group-network-interfaces`, `sg-nics`|
|Retrieve a single network interface specified by the identifier in the URL path. The network interface must be an existing member of the security group. | GET /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface`, `sg-nic`|



## Comparing API and CLI commands for ACLs
{: #comparing-api-and-cli-commands-for-acls}

| Description | API | CLI |
|-------------|-----|-----|
|Creates an ACL |`POST /network_acls` | `ibmcloud is network-acl-create`|
|Retrieves all ACLs |`GET /network_acls` | `ibmcloud is network-acls`|
|Retrieves a specific ACL| `GET /network_acls/{id}`| `ibmcloud is network-acl`|
|Updates a specific ACL| `PATCH /network_acls/{id}`| `ibmcloud is network-acl-update`|
|Deletes a specific ACL| `DELETE /network_acls/{id}`| `ibmcloud is network-acl-delete`|
|Creates an ACL rule| `POST /network_acls/{network_acl_id}/rules`| `ibmcloud is network-acl-rule-add`|
|Retrieves all rules of an ACL| `GET /network_acls/{network_acl_id}/rules`| `ibmcloud is network-acl-rules`|
|Retrieves a specific ACL rule| `GET /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule`|
|Updates a specific ACL rule| `PATCH /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule-update`|
|Deletes a specific ACL rule| `DELETE /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule-delete`|
|Attaches an ACL to a subnet|`PUT /subnets/{id}/network_acl`| `ibmcloud is subnet-network-acl-attach` |
