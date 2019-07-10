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


# API 和 CLI 指令的比較
{: #comparison-of-commands-for-the-api-and-ci}

這組表格提供一種方法來比較可透過 REST API 或透過 IBM Cloud CLI 呼叫的類似指令。

## 比較安全群組的 API 與 CLI 指令
{: #comparing-api-and-cli-commands-for-security-groups}

| 說明 | API | CLI |
|-------------|-----|-----|
|對 URL 路徑中的 ID 所指定的 VPC 擷取預設安全群組。| GET /vpcs/{vpc_id}/default_security_group| |
|擷取此帳戶所有的現有安全群組|GET /security_groups| `ibmcloud is security-groups`, `sgs`|
|擷取 URL 路徑中的 ID 所指定的單一安全群組。| GET /security_groups/{id}| `ibmcloud is security-group`, `sg`|
|從安全群組範本建立新的安全群組。範本的結構方式與所擷取的安全群組相同。它包含建立新安全群組所需的資訊。範本可包括選用的安全群組規則陣列，這些將新增至該群組中。| POST /security_groups| `ibmcloud is security-group-create`, `sgc`|
|以指定的現有安全群組連接至伺服器的網路介面，來建立伺服器實例。|POST /instances（含安全群組參數）| `ibmcloud is instance-create` |
|以安全群組修補程式物件中提供的資訊來更新安全群組。安全群組修補程式物件的結構方式與所擷取的安全群組相同。它只包含要更新的資訊。| PATCH /security_groups/{id}| `ibmcloud is security-group-update`, `sgu`|
|擷取特定安全群組的所有規則。| GET /security_groups/{security_group_id}/rules| `ibmcloud is security-group-rules`, `sg-rules`|
|擷取 URL 路徑中的 ID 所指定的單一安全群組規則。| GET /security_groups/{security_group_id}/rules/{id}| `ibmcloud is security-group-rule`, `sg-rule`|
|從安全群組規則範本建立新的安全群組規則。規則範本物件的結構方式與所擷取的安全群組規則相同。它包含建立規則所需的資訊。此規則適用於安全群組中的所有網路介面。| POST /security_groups/{security_group_id}/rules| `ibmcloud is security-group-rule-add`, `sg-rulec`|
|以規則修補程式物件中提供的資訊來更新安全群組。修補程式物件的結構方式與所擷取的安全群組規則相同。它只能包含要更新的資訊。| PATCH /security_groups/{security_group_id}/rules/{id}| |
|刪除安全群組規則。此刪除動作不會終止該規則已容許的任何現有連線。此作業無法回復。| DELETE /security_groups/{security_group_id}/rules/{id}| `ibmcloud is security-group-rule-delete`, `sg-ruled`|
|刪除安全群組。如果安全群組是 VPC 的預設安全群組，或者有任何其他安全群組包含的規則參照它作為遠端群組，則無法刪除該安全群組。此作業無法回復。| DELETE /security_groups/{id}| `ibmcloud is security-group-delete`, `sgd`|
|將伺服器的現有網路介面新增至現有安全群組。此功能會將群組的規則套用至該介面，容許與介面之間建立符合這些規則的新連線。<br /><br />此外，此網路介面現在被授與參照此群組之任何遠端群組的規則所指定的存取權。<br /><br />請參閱下面的「限制」。| PUT /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface-add`, `sg-nica`|
|從安全群組中移除伺服器的網路介面。不再使用該群組的規則來允許該介面上的新網路連線。<br /><br />此外，此網路介面不再被授與參照此群組之任何遠端群組的規則所指定的存取權。<br /><br />此群組的成員資格所允許的現有連線不會結束。| DELETE /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface-remove`, `sg-nicd`|
|擷取所有與安全群組相關聯的網路介面。這些是套用該群組之規則的網路介面。| GET /security_groups/{security_group_id}/network_interfaces| `ibmcloud is security-group-network-interfaces`, `sg-nics`|
|擷取 URL 路徑中的 ID 所指定的單一網路介面。網路介面必須是安全群組的現有成員。| GET /security_groups/{security_group_id}/network_interfaces/{id}| `ibmcloud is security-group-network-interface`, `sg-nic`|



## 比較 ACL 的 API 與 CLI 指令
{: #comparing-api-and-cli-commands-for-acls}

| 說明 | API | CLI |
|-------------|-----|-----|
|建立 ACL|`POST /network_acls` | `ibmcloud is network-acl-create`|
|擷取所有 ACL|`GET /network_acls` | `ibmcloud is network-acls`|
|擷取特定的 ACL| `GET /network_acls/{id}`| `ibmcloud is network-acl`|
|更新特定的 ACL| `PATCH /network_acls/{id}`| `ibmcloud is network-acl-update`|
|刪除特定的 ACL| `DELETE /network_acls/{id}`| `ibmcloud is network-acl-delete`|
|建立 ACL 規則| `POST /network_acls/{network_acl_id}/rules`| `ibmcloud is network-acl-rule-add`|
|擷取 ACL 的所有規則| `GET /network_acls/{network_acl_id}/rules`| `ibmcloud is network-acl-rules`|
|擷取特定的 ACL 規則| `GET /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule`|
|更新特定的 ACL 規則| `PATCH /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule-update`|
|刪除特定的 ACL 規則| `DELETE /network_acls/{network_acl_id}/rules/{id}`| `ibmcloud is network-acl-rule-delete`|
|將 ACL 連接至子網路|`PUT /subnets/{id}/network_acl`| `ibmcloud is subnet-network-acl-attach` |
