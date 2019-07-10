---


copyright:
  years: 2018, 2019
lastupdated: "2019-06-12"

keywords: load balancer, public, listener, back-end, front-end, pool, round-robin, weighted, connections, methods, policies, APIs, access, ports

subcollection: vpc-on-classic-network

---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:note: .note}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# 使用 Load Balancer for VPC
{: #--using-load-balancers-in-ibm-cloud-vpc}

{{site.data.keyword.cloud}} **Load Balancer for VPC** 服务在 VPC 的同一区域内的多个服务器实例之间分发流量。

## 公共负载均衡器
{: #public-load-balancer}

负载均衡器服务实例分配有可公开访问的标准域名 (FQDN)，必须使用该域名来访问 IBM Cloud Load Balancer for VPC 后面托管的应用程序。此域名可能注册有一个或多个公共 IP 地址。

随着时间的推移，这些公共 IP 地址和公共 IP 地址数量可能会因维护和缩放活动而更改。托管应用程序的后端服务器实例 (VSI) 必须在同一 VPC 下的同一区域中运行。

## 专用负载均衡器
{: #private-load-balancer}

专用负载均衡器仅可由位于相同区域和 VPC 中的专用子网上的内部客户机访问。专用负载均衡器仅接受来自 [RFC1918 ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")](https://tools.ietf.org/html/rfc1918){: new_window} 地址空间的流量。

与公共负载均衡器类似，专用负载均衡器服务实例也分配有标准域名 (FQDN)。但是，此域名注册了一个或多个专用 IP 地址。

一段时间后，IBM Cloud 操作可能会根据维护和缩放活动更改分配的专用 IP 地址的数量和值。托管应用程序的后端服务器实例 (VSI) 必须在同一 VPC 下的同一区域中运行。

请查看下表，以了解功能的摘要比较：

| 功能 | 公共负载均衡器 |专用负载均衡器|
|--------|-------|-------|
| 是否可在因特网上访问？|  是，使用 FQDN，因特网 | 否，仅限内部客户机，位于相同区域及 VPC 上 |
| 是否接受所有流量？| 是 | 否，仅限 RFC 1918 |
| 专用 IP 数 | 随时间更改 | 随时间更改 |
| 如何注册域名？| 公共 IP 地址 | 专用 IP 地址 |

## 前端侦听器和后端池
{: #front-end-listeners-and-back-end-pools}

您最多可定义十 (10) 个前端侦听器（应用程序端口），然后将其映射到后端应用程序服务器上的相应后端池。分配给 Load Balancer 服务实例的标准域名 (FQDN) 和前端侦听器端口会公开给公用因特网。在这些端口上会接收入局用户请求。

支持的前端侦听器协议为：
* HTTP
* HTTPS
* TCP

支持的后端池协议为：
* HTTP
* TCP

入局 HTTPS 流量在负载均衡器处终止，以允许与后端服务器进行明文 HTTP 通信。

最多可将五十 (50) 个服务器实例连接到后端池。每个连接的服务器实例必须配置有一个端口。该端口可与前端侦听器端口相同，也可以不同。

端口范围 56500 到 56520 保留用于管理目的；这些端口不能用作前端侦听器端口。
{: note}

## 第 7 层负载均衡
{: #layer-7-load-balancing}

公共和专用负载均衡器都支持第 7 层负载均衡。数据流量基于配置的策略和规则进行分发。_策略_定义了在入局请求匹配与策略关联的规则时的操作（表示流量的分发方式）。

### 第 7 层策略
{: #layer-7-policy}

第 7 层策略与侦听器相关联，并且只能是 HTTP 或 HTTPS 侦听器。每个策略都有一组规则。**只有**当所有规则都匹配时，才会应用策略。

您可以将多个策略附加到一个侦听器。通常，首先对具有最低优先级的策略求值。对于给定策略，优先级必须唯一。

如果入局请求与任何策略规则都不匹配，那么会将该请求重定向到侦听器的缺省池（如果配置了缺省池）。

下面是第 7 层策略支持的操作：

* **reject：**请求被拒绝，并带有 403 响应。
* **redirect：**请求重定向到配置的 URL 和响应代码。
* **forward：**请求发送到特定后端池。

将首先对设置为 `reject` 的策略求值，而不考虑这些策略的优先级。

随后将对设置为 `redirect` 的策略求值。

最后将对设置为 `forward` 的策略求值。

在每个操作类别中，会按优先级升序（从最低到最高）对策略求值。

### 第 7 层策略属性
{: #layer-7-policy-properties}

属性|描述
------------- | -------------
名称|策略的名称。名称在侦听器中必须唯一。
操作| 所有策略规则匹配时要执行的操作。可接受的值为 `reject`、`redirect` 和 `forward`。将始终先对带有 `reject` 操作的策略求值，而不考虑其优先级。接下来会对有 `redirect` 操作的策略求值，然后对带有 `forward` 操作的策略求值。
优先级|将根据优先级升序对策略求值。
URL|在操作设置为 `redirect` 的情况下，请求将重定向到的 URL。
HTTP 状态码|在操作设置为 `redirect` 时负载均衡器返回的响应的状态码。可接受的值为：301、302、303、307 或 308。
目标|在操作设置为 `forward` 的情况下，请求将转发到的服务器实例的后端池。

### 第 7 层规则
{: #layer-7-rules}

第 7 层规则定义应如何匹配请求。支持三种类型：

类型|描述
----------| -----------------------
`hostname` | 请求与指定的主机名（例如，`api.my_company.com`）相匹配。
`header`    | 请求与 HTTP 头字段（例如，`Cookie`）相匹配。
`path`      | 请求与 URL 中的路径（例如，`/index.html`）相匹配。

要匹配请求，必须在规则中定义`条件`。支持三种条件：

条件|求值类型
----------------|---------------------
`contains`        |  验证抽取的字段是否包含提供的字符串。
`equals`        |  验证抽取的字段是否与提供的字符串相同。
`matches_regex`           |  将抽取的字段与提供的正则表达式相匹配。

## 第 7 层规则属性
{: #layer-7-rule-properties}

属性|描述
------------- | -------------
类型|指定规则的类型。可接受的值为 `hostname`、`header` 或 `path`。
条件|指定用于对规则求值的条件。条件可以为：`contains`、`equals` 或 `matches_regex`。
字段|指定 HTTP 头字段名称。此字段仅适用于 `header` 规则类型。例如，要匹配 HTTP 头中的 cookie，可以将该字段设置为 `Cookie`。
值|要匹配的值。

## 负载均衡方法
{: #load-balancing-methods}

以下三种负载均衡方法可用于在后端应用程序服务器之间分配流量：

* **循环法：**循环法是缺省负载均衡方法。使用此方法，负载均衡器以循环方式将入局客户机连接转发到后端服务器。因此，所有后端服务器会收到大致相等数量的客户机连接。

* **加权循环法：**使用此方法，负载均衡器会根据分配给后端服务器的权重，成比例地将入局客户机连接转发到这些服务器。每个服务器分配的缺省权重为 50，可以定制为 0 到 100 之间的任何值。

例如，如果三个应用程序服务器 A、B 和 C 的定制权重分别为 60、60 和 30，那么服务器 A 和 B 将接收相等数量的连接，而服务器 C 收到的连接数是 A 和 B 的一半。

* **最少连接数：**使用此方法，在给定时间提供最少连接数的服务器实例会接收下一个客户机连接。

**这些方法的其他特征：**

* 将服务器权重重置为“0”表示新连接不会转发到该服务器，但任何现有流量都将继续流动。使用权重“0”可帮助正常关闭服务器，并将其从服务循环中除去。
* 服务器权重值仅适用于加权循环法。循环法和最少连接数负载均衡方法将忽略服务器权重值。

## 水平缩放
{: #horizontal-scaling}

负载均衡器会根据负载自动调整其容量。发生这种调整时，您可能会看到与负载均衡器的 DNS 名称相关联的 IP 地址数发生变化。

## 运行状况检查
{: #health-checks}

对于后端池，运行状况检查定义是必需的。

负载均衡器会定期执行运行状况检查，以监视后端端口的运行状况，并相应地将客户机流量转发给这些端口。如果发现给定的后端服务器端口运行不正常，那么不会向其转发任何新连接。负载均衡器会继续监视未正常运行的端口的运行状况，如果这些端口重新变得运行正常（这意味着这些端口成功通过了两次连续运行状况检查尝试），那么会恢复使用这些端口。

HTTP 和 TCP 端口的运行状况检查按如下所示来执行：

* **HTTP：**针对预先指定的 URL 的 `HTTP GET` 请求会发送到后端服务器端口。在收到 `200 正常`响应时，会将服务器端口标记为正常运行。使用 UI 时，缺省 `GET` 运行状况路径为“/”，此路径可以定制。

* **TCP：**负载均衡器尝试在指定 TCP 端口上打开与后端服务器的 TCP 连接。如果连接尝试成功，那么服务器端口会标记为正常运行，随后该连接会关闭。

缺省运行状况检查时间间隔为 5 秒，针对运行状况检查请求的缺省超时为 2 秒，缺省重试次数为 2。
{: note}

## SSL 卸载和必需的授权
{: #ssl-offloading-and-required-authorizations}

对于所有入局 HTTPS 连接，Load Balancer 服务会终止 SSL 连接，并与后端服务器实例建立明文 HTTP 通信。通过这种方法，CPU 密集型 SSL 握手和加密或解密任务可从后端服务器实例转移到其他位置，从而允许这些实例使用其所有 CPU 周期来处理应用程序流量。

负载均衡器需要 SSL 证书才能执行 SSL 卸载任务。可以通过 [IBM Certificate Manager ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window} 来管理 SSL 证书。

要使负载均衡器能够访问您的 SSL 证书，必须启用**服务到服务的授权**，用于授予负载均衡器服务实例对证书管理器实例的访问权。可以遵循文档[授予服务之间的访问权 ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")](/docs/iam?topic=iam-serviceauth#create-auth){: new_window} 来管理此类授权。请确保选择 **VPC 基础架构**作为源服务，选择 **Load Balancer for VPC** 作为资源类型，选择 **Certificate Manager** 作为目标服务，并分配**写入者**服务访问角色。

如果除去了必需的授权，那么负载均衡器可能会发生错误。
{: note}

## Identity and Access Management (IAM)
{: #identity-and-access-management-iam}

您可以为 **Load Balancer for VPC** 实例配置访问策略。要管理用户访问策略，请访问[管理对资源的访问权 ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")](/docs/services/iam?topic=iam-iammanidaccser#resourceaccess){: new_window}，以获取有关身份和访问管理的更多信息。

### 为用户配置资源组访问策略
{: #configuring-resource-group-access-policies-for-users}

要创建负载均衡器，您需要访问资源组。创建负载均衡器的用户必须具有对所提供资源组的正确访问权，或者如果未提供任何资源组，那么必须具有对缺省资源组的正确访问权。

1. 浏览至**管理 > 帐户 > 用户**。您将看到有权访问 IBM Cloud 帐户的用户的列表。
2. 选择要为其分配访问策略的用户的名称。如果未显示所需用户，请单击**邀请用户**以将该用户添加到 IBM Cloud 帐户。
3. 选择**分配访问权**。
4. 选择**在资源组中分配访问权**。
5. 从**资源组**下拉列表中，选择所需的资源组。
6. 从**分配对资源组的访问权**下拉列表中，选择所需的访问权。
7. 从**服务**下拉列表中，选择所需的服务。
9. 选择**分配**以将资源组访问策略分配给用户。

### 为用户配置资源访问策略
{: #configuring-resource-access-policies-for-users}

|平台访问角色|负载均衡器操作|
|-------------|-----|
|管理员|创建/查看/编辑/删除负载均衡器|
|编辑者|创建/查看/编辑/删除负载均衡器|
|查看者|查看负载均衡器|

1. 浏览至**管理 > 帐户 > 用户**。您将看到有权访问 IBM Cloud 帐户的用户的列表。
2. 选择要为其分配访问策略的用户的名称。如果未显示所需用户，请单击**邀请用户**以将该用户添加到 IBM Cloud 帐户。
3. 选择**分配访问权**。
4. 选择**分配对资源的访问权**。
5. 从**服务**下拉列表中，选择 **VPC 基础架构**。
6. 从**资源类型**下拉列表中，选择 **Load Balancer for VPC**。
7. 从**负载均衡器标识**下拉列表中，选择负载均衡器实例标识，或使用缺省值**所有负载均衡器**。
8. 为用户分配平台访问角色。
9. 选择**分配**以将访问策略分配给用户。

## Activity Tracker 集成
{: #activity-tracker-integration}

Load Balancer 服务已与 **IBM Cloud Activity Tracker with LogDNA** 集成，后者会以符合 CADF 标准的方式记录事件，这由用户发起的会更改云中服务状态的活动触发。

有关在负载均衡器服务实例上记录为审计事件的操作的详细列表，请参阅 [Activity Tracker with LogDNA 事件](/docs/vpc-on-classic?topic=vpc-on-classic-at-events#events-load-balancers)。

所有审计事件都记录到 `us-south` 区域中的“IBM Cloud Activity Tracker with LogDNA”。供应 Load Balancer 服务的区域无关紧要。
{:note}

要查看事件，必须在 `us-south` 区域中您的帐户下供应“IBM Cloud Activity Tracker with LogDNA”实例。您帐户中的用户必须有 IAM 策略授予了对“IBM Cloud Activity Tracker with LogDNA”实例的**查看者**平台访问角色和**读取者**服务访问角色。

[授予查看帐户事件的许可权 ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")](https://cloud.ibm.com/docs/services/cloud-activity-tracker/how-to/grant_permissions.html#grant_permissions){: new_window} 中提供了有关授予访问权的更多信息。

IBM Cloud 帐户用户可以监视对负载均衡器服务执行的帐户级别操作。
{: tip}

执行以下步骤以在您的帐户下供应“IBM Cloud Activity Tracker with LogDNA”实例：

1. 登录到 IBM Cloud 控制台。[登录到 IBM Cloud 控制台。![外部链接图标](../icons/launch-glyph.svg "外部链接图标")](https://cloud.ibm.com/){: new_window}
2. 单击左上方的 ![“菜单”图标](../../icons/icon_hamburger.svg)。在该处选择**可观察性 > Activity Tracker**。
3. 在右上方，单击**创建实例**。
4. 定义服务名称。
5. 选择 `us-south` 作为区域并选择资源组
6. 如果您有付费帐户，请选择除`轻量`以外的套餐。
7. 单击**创建**。

下面提供了**创建侦听器**操作的样本 Activity Tracker 消息：
```bash
{
    "logSourceCRN": "crn:v1:bluemix:public:is:eu-gb:a/<ACCOUNT_ID>::load-balancer:4633518f-8aac-48a1-a694-d15ee6bd70e3",
    "meta": {
        "serviceProviderName": "is-load-balancer",
        "serviceProviderProjectId": "48a7a7b7-6642-4aa1-8af9-c1be4ef82050",
        "serviceProviderRegion": "ng",
        "userAccountIds": [
            <ACCOUNT_ID>
        ]
    },
    "payload": {
        "action": "is.load-balancer.load-balancer.listener.create",
        "eventTime": "2019-05-30T18:42:48.96+0000",
        "eventType": "activity",
        "id": "e4ee1906d01a35efe8bd8303ce0a734e",
        "initiator": {
            "credential": {
                "type": "token"
            },
            "host": {
                "address": <CLIENT_IP>,
                "agent": "python-requests/2.19.1"
            },
            "id": <USER-ID>,
            "name": <USER_ID>,
            "project_id": <ACCOUNT_ID>,
            "typeURI": "service/security/account/user"
        },
        "message": "is.load-balancer: create listener 4633518f-8aac-48a1-a694-d15ee6bd70e3 success",
        "observer": {
            "id": "activity-tracker.ng.bluemix.net",
            "name": "ActivityTracker",
            "typeURI": "security/edge/activity-tracker"
        },
        "outcome": "success",
        "reason": {
            "reasonCode": 201,
      "reasonType": "https://www.iana.org/assignments/http-status-codes/http-status-codes.xml"
   },
        "requestData": "{\"headers\":{\"RayID\":\"4df2d9911a3ac2bd\"},\"extraData\":{\"resourceName\":\"4633518f-8aac-48a1-a694-d15ee6bd70e3\"}}",
        "requestPath": "/v1/load_balancers/4633518f-8aac-48a1-a694-d15ee6bd70e3/listeners",
        "severity": "normal",
        "target": {
            "host": {
                "address": <API_END_POINT>
            },
            "id": "crn:v1:bluemix:public:is:eu-gb:a/<ACCOUNT_ID>::load-balancer:4633518f-8aac-48a1-a694-d15ee6bd70e3",
            "name": "4633518f-8aac-48a1-a694-d15ee6bd70e3",
            "typeURI": "/v1/load_balancers/4633518f-8aac-48a1-a694-d15ee6bd70e3/listeners"
        },
        "typeURI": "http://schemas.dmtf.org/cloud/audit/1.0/event"
    },
    "saveServiceCopy": true
}
```

## 可用 API
{: #lbaas-apis-available}

要进行 API 调用，必须使用某种形式的 REST 客户机。例如，可以使用 `curl` 命令来检索所有现有负载均衡器：

```
curl -X GET "$rias_endpoint/v1/load_balancers?version=2019-05-31&generation=1" -H "Authorization: $iam_token"
```
{: pre}

以下部分提供了有关可用于 VPC 环境中负载均衡器的 API 的详细信息。有关完整规范，请参阅 [VPC on Classic API 参考](https://{DomainName}/apidocs/vpc-on-classic#list-all-load-balancers)。

|描述|API|
|-------------|-----|
|创建和供应负载均衡器|POST /load_balancers|
|检索所有负载均衡器|GET /load_balancers|
|检索负载均衡器|GET /load_balancers/{id}|
|删除负载均衡器|DELETE /load_balancers/{id}|
|更新负载均衡器|PATCH /load_balancers/{id}|
|创建侦听器|POST /load_balancers/{id}/listeners|
|检索负载均衡器的所有侦听器|GET /load_balancers/{id}/listeners|
|检索侦听器|GET /load_balancers/{id}/listeners/{listener_id}|
|删除侦听器|DELETE /load_balancers/{id}/listeners/{listener_id}|
|更新侦听器|PATCH /load_balancers/{id}/listeners/{listener_id}|
|创建池|POST /load_balancers/{id}/pools|
|检索负载均衡器的所有池|GET /load_balancers/{id}/pools|
|检索池|GET /load_balancers/{id}/pools/{pool_id}|
|删除池|DELETE /load_balancers/{id}/pools/{pool_id}|
|更新池|PATCH /load_balancers/{id}/pools/{pool_id}|
|创建成员|POST /load_balancers/{id}/pools/{pool_id}/members|
|检索池的所有成员|GET /load_balancers/{id}/pools/{pool_id}/members|
|检索成员|GET /load_balancers/{id}/pools/{pool_id}/members/{member_id}|
|从池中删除成员|DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id}|
|更新成员|PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id}|
|更新池的成员|PUT /load_balancers/{id}/pools/{pool_id}/members|
|检索负载均衡器的统计信息|GET /load_balancers/{id}/statistics |
| 检索侦听器的所有策略 |  GET /load_balancers/{id}/listeners/{listener_id}/policies
| 为侦听器创建策略 | POST /load_balancers/{id}/listeners/{listener_id}/policies
| 从侦听器中删除策略 | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| 检索侦听器的策略 | GET /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| 更新侦听器的策略 | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| 检索与策略关联的所有规则 | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| 为策略创建规则 | POST /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| 从策略中删除规则 | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| 检索策略中的规则 | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| 更新策略的规则 | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}

## 负载均衡器示例
{: #load-balancer-example}

在以下示例中，您将使用 API 在运行 Web 应用程序（用于侦听端口 `80`）的 2 个 VPC 服务器实例（`192.168.100.5` 和 `192.168.100.6`）前面创建负载均衡器。负载均衡器具有前端侦听器，用于允许通过 HTTPS 安全地访问 Web 应用程序。然后，可以使用 API 在创建负载均衡器实例后获取其详细信息，也可以删除负载均衡器实例。

### 示例步骤
{: #lbaas-example-steps}

以下示例步骤跳过了使用 [IBM Cloud UI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console)、[CLI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli) 或 [VPC on Classic API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis) 来供应 VPC、子网和实例的先决条件步骤。

负载均衡器示例步骤还可以使用 [CLI](/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference) 来运行。
{: note}

**步骤 1. 使用侦听器、池和连接的服务器实例（池成员）创建负载均衡器**

```bash
curl -H "Authorization: $iam_token" -X POST
"$rias_endpoint/v1/load_balancers?version=2019-05-31&generation=1" \
    -d '{
        "name": "example-balancer",
        "is_public": true,
        "listeners": [
            {
                "certificate_instance": {
                    "crn": "crn:v1:bluemix:public:cloudcerts:us-south:a/123456:b8877ea4-b8eg-467e-912a-da1eb7f031cg:certificate:43219c4c97d013fb2a95b21dddde1234"
                },
                "port": 443,
                "protocol": "https",
                "default_pool": {
                    "name": "example-pool"
                }
            }
        ],
        "pools": [
            {
                "algorithm": "round_robin",
                "health_monitor": {
                    "delay": 5,
                    "max_retries": 2,
                    "timeout": 2,
                    "type": "http",
                    "url_path": "/"
                },
                "name": "example-pool",
                "protocol": "http",
                "session_persistence": {
                    "cookie_name": "string",
                    "type": "source_ip"
                },
                "members": [
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.5"
                        },
                        "weight": 50
                    },
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.6"
                        },
                        "weight": 50
                    }
                ]
            }
        ],
        "subnets": [
            {
                "id": "7ec87131-1c7e-4990-b4f0-a26f2e61f98e"
            }
        ]
        }'
```
{: codeblock}

样本输出：
```
{
    "created_at": "2018-07-12T23:17:07.5985381Z",
    "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "hostname": "ac34687d.lb.appdomain.cloud",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "is_public": true,
    "listeners": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
        }
    ],
    "name": "example-balancer",
    "operating_status": "offline",
    "pools": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
            "name": "example-pool"
        }
    ],
    "provisioning_status": "create_pending",
    "resource_group": {
        "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
    },
    "subnets": [
        {
            "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "name": "example-subnet"
        }
    ]
}
```
{: screen}

保存负载均衡器的标识以在后续步骤中使用，例如，保存在变量 `lbid` 中。

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

**步骤 2. 获取负载均衡器**

```
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

请留出一些时间等待供应。负载均衡器在准备就绪后将设置为 `online` 和 `active` 状态，如以下样本输出中所示：

样本输出：

```bash
{
  "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "name": "example-balancer",
  "created_at": "2018-07-13T22:22:24.489Z",
  "hostname": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud",
  "is_public": true,
  "listeners": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
    }
  ],
  "operating_status": "online",
  "pools": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
      "name": "example-pool"
    }
  ],
  "private_ips": [
    {
      "address": "192.168.10.5"
    },
    {
      "address": "192.168.10.6"
    }
  ],
  "provisioning_status": "active",
  "public_ips": [
    {
        "address": "169.11.111.115"
    },
    {
        "address": "169.11.111.116"
    }
  ],
  "resource_group": {
    "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
  },
  "subnets": [
    {
      "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "name": "example-subnet"
    }
  ]
}
```
{: screen}

**步骤 3. 删除负载均衡器**

```bash
curl -H "Authorization: $iam_token" -X DELETE "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

## 第 7 层示例：创建策略和规则
{: #layer-7-examples-create-policy-and-rules}

以下两个示例提供了说明如何创建策略和规则并将其与侦听器关联的步骤。

### 示例 1：使用策略和规则创建 HTTPS 侦听器。策略将使用操作 `redirect` 来创建

```
bash
curl -H "Authorization: $iam_token" -X POST
"$rias_endpoint/v1/load_balancers/$lbId/listeners" \
    -d '{
            "certificate_instance": {
                    "crn": "crn:v1:bluemix:public:cloudcerts:us-south:a/1111111111111111111111111111:22222222-3333-4444-5555-666666666666:certificate:77777777777777777777777777777777"
            },
            "connection_limit": 2000,
            "port": 443,
            "protocol": "https",
            "policies": [
                {
                    "name": "hostname_header",
                    "action": "redirect",
                    "priority": 1,
                    "target": {
                        "url": "https://www.examples.com/",
                        "http_status_code": 307
                    },
                    "rules": [
                        {
                            "condition": "contains",
                            "type": "header",
                            "field": "aheader",
                            "value": "avalue"
                        },
                        {
                            "condition": "equals",
                            "type": "hostname",
                            "value": "abc.com"
                        }
                    ]
                },
                {
                    "name": "header_cookie",
                    "action": "redirect",
                    "priority": 5,
                    "target": {
                        "url": "https://www.mycookies.com/",
                        "http_status_code": 302
                    },
                    "rules": [
                        {
                            "condition": "contains",
                            "type": "header",
                            "field": "aheader",
                            "value": "avalue"
                        },
                        {
                            "condition": "equals",
                            "type": "header",
                            "field": "cookie",
                            "value": "flavor=oatmeal"
                        }
                    ]
                },
                {
                    "name": "path_hostname",
                    "action": "redirect",
                    "priority": 10,
                    "target": {
                        "url": "https://www.myexamples.com/",
                        "http_status_code": 301
                    },
                    "rules": [
                        {
                            "condition": "contains",
                            "type": "hostname",
                            "value": "abc"
                        },
                        {
                            "condition": "equals",
                            "value": "/test",
                            "type": "path"
                          }
                    ]
                }
            ]
        }'
```
{: codeblock}

### 示例 2：使用针对池的 `forward` 操作创建策略，并将其与现有侦听器关联
```
bash
curl -H "Authorization: $iam_token" -X POST
"$rias_endpoint/v1/load_balancers/$lbId/listeners/$listenerId/policies" \
    -d '{
            "policies": [
                {
                    "action": "forward",
                    "priority": 1,
                    "target": {
                        "id": "7df616da-4dd6-43d3-881d-801ae29e29fe"
                    },
                    "rules": [
                        {
                            "condition": "equals",
                            "type": "header",
                            "field": "cookie",
                            "value": "flavor=oatmeal"
                        }
                    ]
                },
                {
                    "action": "forward",
                    "priority": 5,
                    "target": {
                        "id": "8061c411-0d50-4c79-b475-102666796434"
                    },
                    "rules": [
                        {
                            "condition": "contains",
                            "type": "header",
                            "field": "aheader",
                            "value": "avalue"
                        }
                    ]
                },
                {
                    "action": "forward",
                    "priority": 10,
                    "target": {
                        "id": "62914e09-3928-4d89-b7f7-1bb7a6d7fe85"
                    },
                    "rules": [
                        {
                            "condition": "matches_regex",
                            "type": "hostname",
                            "value": "abc[a-z]*.com"
                        }
                    ]
                },
                {
                    "action": "forward",
                    "priority": 6,
                    "target": {
                        "id": "62914e09-3928-4d89-b7f7-1bb7a6d7fe85"
                    },
                    "rules": [
                        {
                            "condition": "equals",
                            "type": "path",
                            "value": "/test/testtest"
                        }
                    ]
                }
            ]
        }'
```
{: codeblock}

## 常见问题
{: #load-balancer-faqs}

此部分包含有关 **Load Balancer for VPC** 服务的一些常见问题的解答。

### 可以对负载均衡器使用其他 DNS 名称吗？
{: #can-i-use-a-different-dns-name-for-my-load-balancer}

负载均衡器的 DNS 名称是自动分配的，不可定制。但是，可以添加 CNAME（规范名称）记录，用于将您的首选 DNS 名称指向自动分配的负载均衡器 DNS 名称。例如，`us-south` 中您的负载均衡器的标识为 `dd754295-e9e0-4c9d-bf6c-58fbc59e5727`，自动分配的负载均衡器 DNS 名称为 `dd754295-us-south.lb.appdomain.cloud.lb.appdomain.cloud`。您的首选 DNS 名称为 `www.myapp.com`。您可以添加 CNAME 记录（通过用于管理 `myapp.com` 的 DNS 提供程序），将 `www.myapp.com` 指向负载均衡器 DNS 名称 `dd754295-us-south.lb.appdomain.cloud.lb.appdomain.cloud`。

### 使用负载均衡器最多可以定义多少个前端侦听器？
{: #what-s-the-maximum-number-of-front-end-listeners-i-can-define-with-my-load-balancer}

10 个。

### 最多可以将多少个服务器实例连接到后端池？
{: #what-s-the-maximum-number-of-server-instances-i-can-attach-to-my-back-end-pool}

50 个。

### 负载均衡器可水平缩放吗？
{: #is-the-load-balancer-horizontally-scalable}

是。负载均衡器根据负载自动调整其容量。水平缩放发生时，与负载均衡器的 DNS 名称相关联的 IP 地址数将发生变化。

### 如果要在用于部署负载均衡器的子网上使用 ACL 或安全组，应该怎么做？
{: #what-should-i-do-if-i-am-using-acls-or-security-groups-on-the-subnets-that-are-used-to-deploy-the-load-balancer}

您需要确保落实了正确的 ACL 或安全组规则，以允许配置的侦听器端口和管理端口（从 56500 到 56520 的端口）的入局流量。还应允许负载均衡器与后端实例之间的流量。

### 为什么会收到错误消息：`找不到证书实例`？

* 证书实例 CRN 可能无效。
* 您可能尚未授予**服务到服务的授权**。请参阅本文档的 **SSL 卸载**部分。

### 为什么会收到 `401 未授权错误`代码？

检查用户的以下访问策略：
* 负载均衡器资源类型的访问策略
* 资源组的访问策略
* 如果使用了 `HTTPS` 侦听器，还请检查 Certificate Manager 实例的服务到服务的授权。

### 为什么负载均衡器会处于 `maintenance_pending` 状态？

负载均衡器在各种维护活动期间会处于 `maintenance_pending` 状态，例如以下活动：
* 水平缩放活动
* 恢复活动
* 用于解决漏洞并应用安全补丁的卷动升级

### 为什么在供应期间需要选择多个子网？
{: #why-do-I-need-to-choose-multiple-subnets-during-provisioning}

**Load Balancer for VPC** 已准备就绪，可用于多专区区域 (MZR)。负载均衡器设备会部署到所选的子网。强烈建议选择不同专区中的子网，以提供更高的可用性和冗余性。

### 为什么我的池下面的后端成员运行状况`未知`？

* 池未与任何侦听器关联
* 可能对池或其关联的侦听器进行了配置更改

### SSL 卸载支持哪个 TLS 版本？
{: #which-tls-version-is-supported-with-ssl-offload}

**Load Balancer for VPC** 支持 TLS 1.2 用于 SSL 终止。

以下列表详细说明受支持的密码（按优先顺序列出）：
* ECDHE-RSA-AES256-GCM-SHA384 
* ECDHE-RSA-AES256-SHA384
* AES256-GCM-SHA384
* AES256-SHA256
* ECDHE-RSA-AES128-GCM-SHA256
* ECDHE-RSA-AES128-SHA256
* AES128-GCM-SHA256
* AES128-SHA256

### 运行状况检查参数的缺省设置和允许值是什么？
{: #what-are-the-default-settings-and-allowed-values-for-health-check-parameters}

缺省设置和允许值如下所示：
* 运行状况检查时间间隔：缺省值为 5 秒，范围从 2 到 60 秒。
* 运行状况检查响应超时：缺省值为 2 秒，范围从 1 到 59 秒。
* 最大重试次数：缺省值为 2 次重试，范围从 1 到 10 次重试。

运行状况检查响应超时值必须始终小于运行状况检查时间间隔值。
{:note}

### 负载均衡器 IP 地址是固定的吗？
{: #are-the-load-balancer-ip-addresses-fixed}

由于服务内置弹性，负载均衡器 IP 地址无法保证为固定。在水平缩放期间，您将发现与负载均衡器的 FQDN 相关联的可用 IP 数发生变化。

请使用 FQDN，而非高速缓存的 IP 地址。
{:note}

### 负载均衡器支持第 7 层切换吗？
{: #does-the-load-balancer-support-layer-7-switching}

是。

### 为什么在创建或更新 HTTPS 侦听器时显示我的证书无效？
{: #why-does-https-listener-creation-or-update-tell-me-that-my-certificate-is-invalid}

请检查是否存在以下可能的情况：

* 提供的证书 CRN 可能无效。
* Certificate Manager 中提供的证书实例可能不具有关联的的专用密钥。
