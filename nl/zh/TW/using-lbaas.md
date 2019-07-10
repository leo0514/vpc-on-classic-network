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

# 使用 Load Balancers for VPC
{: #--using-load-balancers-in-ibm-cloud-vpc}

{{site.data.keyword.cloud}} **Load Balancer for VPC** 服務會將資料流量分散在您 VPC 相同地區內的多個伺服器實例之間。

## 公用負載平衡器
{: #public-load-balancer}

您的負載平衡器服務實例被指派一個可公開存取的完整網域名稱 (FQDN)，您必須使用此名稱來存取 IBM Cloud Load Balancer for VPC 之後管理的應用程式。這個網域名稱可能已登錄一個以上的公用 IP 位址。

經過一段時間之後，這些公用 IP 位址及公用 IP 位址數目可能會因為維護和調整活動而變更。代管應用程式的後端伺服器實例 (VSI) 必須在相同地區及相同的 VPC 下執行。

## 專用負載平衡器
{: #private-load-balancer}

在同一個地區及 VPC 內，只有您的專用子網路上的內部用戶端才能存取專用負載平衡器。專用負載平衡器僅接受來自 [RFC1918 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://tools.ietf.org/html/rfc1918){: new_window} 位址空間的資料流量。

與公用負載平衡器類似，也是已指派了完整網域名稱 (FQDN) 給您的專用負載平衡器服務實例。不過，此網域名稱是搭配一個以上的專用 IP 位址登錄的。

IBM Cloud 作業會根據維護和調整活動，在經過一段時間後可能會變更您所指派的專用 IP 位址的數目和值。代管應用程式的後端伺服器實例 (VSI) 必須在相同地區及相同的 VPC 下執行。

請參閱下表以取得特性比較的摘要：

| 特性 | 公用負載平衡器 | 專用負載平衡器 |
|--------|-------|-------|
| 可在網際網路上存取？ |  是，具有適用於網際網路的 FQDN | 不，僅限位於相同地區及 VPC 的內部用戶端 |
| 接受所有資料流量？ | 是 | 否，僅限於 RFC 1918 |
| 專用 IP 數目 | 一段時間後變更 | 一段時間後變更 |
| 如何登錄網域名稱？ | 公用 IP 位址 | 專用 IP 位址 |

## 前端接聽器及後端儲存區
{: #front-end-listeners-and-back-end-pools}

您最多可以定義 10 個前端接聽器（應用程式埠），並將它們對映至後端應用程式伺服器上的個別後端儲存區。將指派給負載平衡器服務實例的完整網域名稱 (FQDN) 以及前端系統接聽器埠公開至公用網際網路。會在這些埠上接收送入的使用者要求。

支援的前端接聽器通訊協定如下：
* HTTP
* HTTPS
* TCP

支援的後端儲存區通訊協定如下：
* HTTP
* TCP

送入的 HTTPS 資料流量會在負載平衡器終止，以容許與後端伺服器進行純文字 HTTP 通訊。

您最多可以將 50 個伺服器實例連接至後端儲存區。每個連接的伺服器實例都必須配置一個埠。這個埠不一定與前端接聽器埠相同。

保留 56500 至 56520 的埠範圍作為管理用途；這些埠無法作為前端接聽器埠。
{: note}

## 第 7 層負載平衡
{: #layer-7-load-balancing}

公用及專用負載平衡器都支援第 7 層負載平衡。資料流量是根據已配置的原則及規則來進行配送。_原則_ 會定義當送入的要求符合與原則相關聯的規則時採取的動作，這表示資料流量的配送方式。

### 第 7 層原則
{: #layer-7-policy}

第 7 層原則與接聽器相關聯，並只與 HTTP 或 HTTPS 接聽器相關聯。每一個原則都可以有一組規則。**只有**在原則的所有規則都符合時，才會套用原則。

您可以將多個原則附加至接聽器。一般而言，會先評估優先順序最低的原則。對於給定原則，優先順序必須是唯一的。

如果送入要求不符合任何原則規則，則會將該要求重新導向至接聽器的預設儲存區（如果已配置預設儲存區的話）。

以下是第 7 層原則的支援動作：

* **拒絕：**要求遭到拒絕，回應為 403。
* **重新導向：**要求會重新導向至已配置的 URL 及回應碼。
* **轉遞：**要求會傳送至特定的後端儲存區。

不論其優先順序為何，都會先評估設為 `reject` 的原則。

之後，會評估設為 `redirect` 的原則。

最後，即會評估設設為 `forward` 的原則。

在每個動作種類內，會依遞增的優先順序來評估原則（最低到最高）。

### 第 7 層原則內容
{: #layer-7-policy-properties}

內容  | 說明
------------- | -------------
名稱 | 原則的名稱。該名稱在接聽器內必須是唯一的。
動作 | 所有原則規則符合時要採取的動作。可接受的值為 `reject`、`redirect` 及 `forward`。不論其優先順序為何，一律先評估具有 `reject` 動作的原則。接下來評估具有 `redirect` 動作的原則，後面接著評估具有 `redirect` 動作的原則。
優先順序 | 根據遞增的優先順序來評估原則。
URL | 將要求重新導向至的 URL（如果動作設為 `redirect` 的話）。
HTTP 狀態碼 | 當動作設為 `redirect` 時，負載平衡器傳回之回應的狀態碼。可接受的值為：301、302、303、307 或 308。
目標 | 將要求轉遞至的伺服器實例後端儲存區（如果動作設為 `forward` 的話）。

### 第 7 層規則
{: #layer-7-rules}

第 7 層規則定義應如何比對要求。支援三種類型：

類型      | 說明
----------| -----------------------
`hostname` | 要求符合指定的主機名稱（例如，`api.my_company.com`）。
`header`    | 要求符合 HTTP 標頭欄位（例如，`Cookie`）。
`path`      | 要求符合 URL 中的路徑（例如，`/index.html`）。

若要比對要求，必須在規則中定義 `condition`。支援三個條件：

條件 |  評估類型
----------------|---------------------
`contains`        | 驗證擷取的欄位是否包含所提供的字串。
`equals`        | 驗證擷取的欄位是否與所提供的字串相同。
`matches_regex`           | 比對擷取的欄位與所提供的正規表示式。

## 第 7 層規則內容
{: #layer-7-rule-properties}

內容  | 說明
------------- | -------------
類型      | 指定規則的類型。可接受的值為 `hostname`、`header` 或 `path`。
條件 | 指定用來評估規則的條件。條件可以是：`contains`、`equals` 或 `matches_regex`。
欄位 | 指定 HTTP 標頭欄位名稱。這個欄位只適用於 `header` 規則類型。例如，若要比對 HTTP 標頭中的 Cookie，這個欄位可以設為 `Cookie`。
值 | 要比對的值。

## 負載平衡方法
{: #load-balancing-methods}

下列三種負載平衡方法可用來配送後端應用程式伺服器之間的資料流量：

* **循環式：**循環式是預設的負載平衡方法。使用此方法，負載平衡器會以循環方式將送入的用戶端連線轉遞至後端伺服器。因此，所有後端伺服器都會接收到約略相等數目的用戶端連線。

* **加權循環式：**使用此方法時，負載平衡器將送入的用戶端連線轉遞至後端伺服器時會與指派給這些伺服器的加權成正比。每部伺服器都會獲指派預設加權 50，您可以將此值自訂為 0 到 100 之間的任何值。

舉例來說，如果三個應用程式伺服器 A、B 和 C 分別自訂為 60、60 和 30 的加權，則伺服器 A 和 B 會收到相等數目的連線，而伺服器 C 會收到此連線數目的一半。

* **最少連線數：**如果使用此方法，則在給定的時間內提供最少連線數目的伺服器實例會接收下一個用戶端連線。

**這些方法的其他特徵：**

* 將伺服器加權重設為 '0'，表示沒有新的連線會轉遞至該伺服器，但任何現有資料流量會繼續傳送。使用加權 '0' 可協助溫和關閉伺服器，並將它從服務循環中移除。
* 伺服器加權值僅適用於加權循環式方法。循環式和最少連線數的負載平衡方法會忽略它們。

## 水平調整
{: #horizontal-scaling}

負載平衡器會根據負載自動調整其容量。進行這項調整時，您可能會看到與負載平衡器的 DNS 名稱相關聯的 IP 位址數目出現變更。

## 性能檢查
{: #health-checks}

對於後端儲存區而言，性能檢查定義是必要的。

負載平衡器會執行定期性能檢查，以監視後端埠的性能，並據此將用戶端資料流量轉遞給它們。如果發現給定的後端伺服器埠性能不佳，就不會轉遞任何新連線。負載平衡器會持續監視不健全埠的性能，如果它們的性能恢復，就會繼續使用，這表示它們會順利通過兩次連續性能檢查。

HTTP 和 TCP 埠的性能檢查執行如下：

* **HTTP：**會將針對預先指定之 URL 的 `HTTP GET` 要求傳送至後端伺服器埠。收到 `200 OK` 回應時，會將伺服器埠標示為性能良好。預設 `GET` 性能路徑是透過使用者介面的 "/"，可以自訂。

* **TCP：**「負載平衡器」嘗試在指定的 TCP 埠開啟與後端伺服器的 TCP 連線。如果連線嘗試成功，且連線已關閉，伺服器埠會被標示為健全。

預設性能檢查間隔為 5 秒，性能檢查要求的預設逾時為 2 秒，而預設的重試次數為 2。
{: note}

## SSL 卸載及必要的授權
{: #ssl-offloading-and-required-authorizations}

對於所有送入的 HTTPS 連線，負載平衡器服務會終止 SSL 連線，並建立與後端伺服器實例的純文字 HTTP 通訊。使用此技術，會將 CPU 密集 SSL 信號交換及加密或解密作業移出後端伺服器實例，從而容許它們使用所有 CPU 週期來處理應用程式資料流量。

負載平衡器需要有 SSL 憑證才能執行 SSL 卸載作業。您可以透過 [IBM Certificate Manager ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window} 來管理 SSL 憑證。

若要將 SSL 憑證存取權授與負載平衡器，您必須啟用**服務對服務授權**，這會將 Certificate Manager 實例的服務實例存取權授權給您的負載平衡器。您可以依循這個 [授與服務之間的存取權文件 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](/docs/iam?topic=iam-serviceauth#create-auth){: new_window} 來管理這類授權。請務必選擇 **VPC 基礎架構**作為來源服務，選擇 **Load Balancer for VPC** 作為資源類型、選擇 **Certificate Manager** 作為目標服務，並指派**撰寫者**服務存取角色。

如果移除了必要授權，您的負載平衡器可能會發生錯誤。
{: note}

## Identity and Access Management (IAM)
{: #identity-and-access-management-iam}

您可以配置 **Load Balancer for VPC** 實例的存取原則。若要管理您的使用者存取原則，請造訪 [管理資源存取 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](/docs/services/iam?topic=iam-iammanidaccser#resourceaccess){: new_window} 以取得 Identity and Access Management 的相關資訊。

### 配置使用者的資源群組存取原則
{: #configuring-resource-group-access-policies-for-users}

若要建立負載平衡器，您需要存取資源群組。建立負載平衡器的使用者，必須對所提供的資源群組或對 default 資源群組（如果未提供任何資源群組的話）具有適當的存取權。

1. 導覽至**管理 > 帳戶 > 使用者**。您會看到一份有權存取 IBM Cloud 帳戶的使用者清單。
2. 選取您要對其指派存取原則的使用者名稱。如果未顯示使用者，請按一下**邀請使用者**，將使用者新增至您的 IBM Cloud 帳戶。
3. 選取**指派存取權**。
4. 選取**在資源群組內指派存取權**。
5. 從**資源群組**下拉清單中，選取想要的資源群組。
6. 從**指派對資源群組的存取權**下拉清單中，選取想要的存取權。
7. 從**服務**下拉清單中，選取想要的服務。
9. 選取**指派**，將資源群組存取原則指派給使用者。

### 配置使用者的資源存取原則
{: #configuring-resource-access-policies-for-users}

| 平台存取角色 | 負載平衡器動作 |
|-------------|-----|
| 管理者 | 建立/檢視/編輯/刪除負載平衡器 |
| 編輯者 | 建立/檢視/編輯/刪除負載平衡器 |
| 檢視者 | 檢視負載平衡器 |

1. 導覽至**管理 > 帳戶 > 使用者**。您會看到有權存取 IBM Cloud 帳戶的使用者清單。
2. 選取您要對其指派存取原則的使用者名稱。如果未顯示使用者，請按一下**邀請使用者**，將使用者新增至您的 IBM Cloud 帳戶。
3. 選取**指派存取權**。
4. 選取**指派對資源的存取權**。
5. 從**服務**下拉清單中，選取 **VPC 基礎架構**。
6. 從**資源類型**下拉清單中，選取 **Load Balancer for VPC**。
7. 從**負載平衡器 ID** 下拉清單中，選取「負載平衡器」實例 ID，或使用預設值**所有負載平衡器**。
8. 指派平台存取角色給使用者。
9. 選取**指派**，將存取原則指派給使用者。

## Activity Tracker 整合
{: #activity-tracker-integration}

負載平衡器服務會與 **IBM Cloud Activity Tracker with LogDNA** 整合，以符合 CADF 標準方式記錄事件，並由使用者起始能變更雲端中服務狀態的活動來觸發。

如需負載平衡器服務實例上記錄為審核事件的詳細動作清單，請參閱 [Activity tracker with LogDNA 事件](/docs/vpc-on-classic?topic=vpc-on-classic-at-events#events-load-balancers)。

所有審核事件都記錄在 `us-south` 地區中的 "IBM Cloud Activity Tracker with LogDNA"。這與負載平衡器服務所佈建的地區無關。
{:note}

若要檢視事件，您必須在帳戶下的 `us-south` 地區中佈建 "IBM Cloud Activity Tracker with LogDNA" 實例。您帳戶中的使用者必須具備 IAM 原則，該原則可授與 "IBM Cloud Activity Tracker with LogDNA" 實例的**檢視者**平台存取角色及**讀者**伺服器存取角色。

如需其他授與存取權的相關資訊，請參閱[授與查看帳戶的許可權。![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://cloud.ibm.com/docs/services/cloud-activity-tracker/how-to/grant_permissions.html#grant_permissions){: new_window}

IBM Cloud 帳戶使用者可以監視在負載平衡器服務上執行的帳戶層次作業。
{: tip}

請遵循下列步驟，以在您的帳戶下佈建 "IBM Cloud Activity Tracker with LogDNA" 實例。

1. 登入 IBM Cloud 主控台。[登入 IBM Cloud 主控台。![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://cloud.ibm.com/){: new_window}
2. 按一下左上方的 ![「功能表」圖示](../../icons/icon_hamburger.svg)。從這裡選取**觀察 > Activity Tracker**。
3. 在右上角按一下**建立實例**。
4. 定義服務名稱。
5. 選取 `us-south` 作為地區，並選擇資源群組。
6. 如果您有付費帳戶，請選擇 `lite` 以外的方案。
7. 按一下**建立**。

以下是**建立接聽器**作業的 Activity Tracker 訊息範例：
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

## 可用的 API
{: #lbaas-apis-available}

若要進行 API 呼叫，您必須使用某種 REST 用戶端格式。例如，您可以使用 `curl` 指令來擷取所有現有負載平衡器：

```
curl -X GET "$rias_endpoint/v1/load_balancers?version=2019-05-31&generation=1" -H "Authorization: $iam_token"
```
{: pre}

下一節提供有關您可在 VPC 環境中用於負載平衡器之 API 的詳細資料。如需完整規格，請參閱 [VPC on Classic API 參考資料](https://{DomainName}/apidocs/vpc-on-classic#list-all-load-balancers)。

| 說明 | API |
|-------------|-----|
| 建立及佈建負載平衡器 | POST /load_balancers |
| 擷取所有負載平衡器 | GET /load_balancers |
| 擷取負載平衡器 | GET /load_balancers/{id} |
| 刪除負載平衡器 | DELETE /load_balancers/{id} |
| 更新負載平衡器 | PATCH /load_balancers/{id} |
| 建立接聽器 | POST /load_balancers/{id}/listeners |
| 擷取負載平衡器的所有接聽器 | GET /load_balancers/{id}/listeners |
| 擷取接聽器 | GET /load_balancers/{id}/listeners/{listener_id} |
| 刪除接聽器 | DELETE /load_balancers/{id}/listeners/{listener_id} |
| 更新接聽器 | PATCH /load_balancers/{id}/listeners/{listener_id} |
| 建立儲存區 | POST /load_balancers/{id}/pools |
| 擷取負載平衡器的所有儲存區 | GET /load_balancers/{id}/pools |
| 擷取儲存區 | GET /load_balancers/{id}/pools/{pool_id} |
| 刪除儲存區 | DELETE /load_balancers/{id}/pools/{pool_id} |
| 更新儲存區 | PATCH /load_balancers/{id}/pools/{pool_id} |
| 建立成員 | POST /load_balancers/{id}/pools/{pool_id}/members |
| 擷取儲存區的所有成員 | GET /load_balancers/{id}/pools/{pool_id}/members |
| 擷取成員 |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| 刪除儲存區的成員 | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| 更新成員 | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| 更新儲存區的成員 | PUT /load_balancers/{id}/pools/{pool_id}/members |
| 擷取負載平衡器統計資料 | GET /load_balancers/{id}/statistics |
| 擷取接聽器的所有原則 |  GET /load_balancers/{id}/listeners/{listener_id}/policies
| 建立接聽器的原則 | POST /load_balancers/{id}/listeners/{listener_id}/policies
| 從接聽器中刪除原則 | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| 擷取接聽器的原則 | GET /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| 更新接聽器原則 | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| 擷取與原則相關聯的所有規則 | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| 建立原則的規則 | POST /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| 從原則中刪除規則 | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| 從原則中擷取規則 | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| 更新原則規則 | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}

## 負載平衡器範例
{: #load-balancer-example}

在下列範例中，您將使用 API 在 2 個 VPC 伺服器實例（`192.168.100.5` 和 `192.168.100.6`）前面建立負載平衡器，其在埠 `80` 執行 Web應用程式接聽。負載平衡器具有前端接聽器，它容許透過 HTTPS 安全地存取 Web 應用程式。然後，您可以使用 API 在建立負載平衡器實例之後取得該實例的詳細資料，以及刪除負載平衡器實例。

### 範例步驟
{: #lbaas-example-steps}

接下來的範例步驟會跳過使用 [IBM Cloud 使用者介面](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console)、[CLI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli) 或 [VPC on Classic API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis) 來佈建 VPC、子網路及實例的必要步驟。

負載平衡器範例步驟也可以使用 [CLI](/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference) 來執行。
{: note}

**步驟 1. 建立具有接聽器、儲存區及連接伺服器實例（儲存區成員）的負載平衡器**

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

輸出範例：
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

儲存下一步要使用的負載平衡器 ID，例如，以變數 `lbid` 來儲存 ID。

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

**步驟 2. 取得負載平衡器**

```
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

等待一段時間以便佈建。負載平衡器備妥時，它將設為 `online` 和 `active` 狀態，如同您在下列輸出範例中所見：

輸出範例：

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

**步驟 3. 刪除負載平衡器**

```bash
curl -H "Authorization: $iam_token" -X DELETE "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

## 第 7 層範例：建立原則及規則
{: #layer-7-examples-create-policy-and-rules}

下列兩個範例提供一些步驟，其中顯示如何建立原則及規則，以及原則及規則如何與監聽器相關聯。

### 範例 1：使用原則及規則建立 HTTPS 接聽器。建立原則時會使用 `Redirect` 動作

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

### 範例 2：建立具有目標為儲存區之 `Forward` 動作的原則，並將其與現有接聽器相關聯
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

## 常見問題 (FAQ)
{: #load-balancer-faqs}

這一節包含有關 **Load Balancer for VPC** 服務的一些常見問題的回答。

### 我的負載平衡器可以使用不同的 DNS 名稱嗎？
{: #can-i-use-a-different-dns-name-for-my-load-balancer}

負載平衡器的自動指派 DNS 名稱不可自訂。不過，您可以新增 CNAME（標準名稱）記錄，將您偏好的 DNS 名稱指向自動指派的負載平衡器 DNS 名稱。例如，您在 `us-south` 中的負載平衡器具有 ID `dd754295-e9e0-4c9d-bf6c-58fbc59e5727`，自動指派的負載平衡器 DNS 名稱是 `dd754295-us-south.lb.appdomain.cloud`。您偏好的 DNS 名稱是 `www.myapp.com`。您可以新增 CNAME 記錄（透過您用來管理 `myapp.com` 的 DNS 提供者），將 `www.myapp.com` 指向負載平衡器 DNS 名稱 `dd754295-us-south.lb.appdomain.cloud`。

### 我可以使用負載平衡器定義的前端接聽器數目上限是多少？
{: #what-s-the-maximum-number-of-front-end-listeners-i-can-define-with-my-load-balancer}

10。

### 我可以連接至後端儲存區的伺服器實例數目上限是多少？
{: #what-s-the-maximum-number-of-server-instances-i-can-attach-to-my-back-end-pool}

50。

### 負載平衡器可以橫向擴充嗎？
{: #is-the-load-balancer-horizontally-scalable}

可以。負載平衡器會根據負載自動調整其容量。進行水平調整時，與負載平衡器的 DNS 名稱相關聯的 IP 位址數目會變更。

### 如果我在用來部署負載平衡器的子網路上使用 ACL 或安全群組，該怎麼辦？
{: #what-should-i-do-if-i-am-using-acls-or-security-groups-on-the-subnets-that-are-used-to-deploy-the-load-balancer}

您需要確定已設定適當的 ACL 或安全群組規則，來容許針對所配置的接聽器埠及管理埠的送入資料流量（56500 - 56520 範圍內的埠）。也應該容許負載平衡器與後端實例之間的資料流量。

### 為什麼我會收到錯誤訊息：`找不到憑證實例`？

* 憑證實例 CRN 可能無效。
* 您可能未授與**服務對服務授權**。請參閱本文件的 **SSL 卸載**一節。

### 為什麼我會收到 `401 未獲授權的錯誤`碼？

檢查使用者的下列存取原則：
* 負載平衡器資源類型的存取原則
* 資源群組的存取原則
* 如果使用 `HTTPS` 接聽器，請同時檢查 Certificate Manager 實例的服務對服務授權。

### 為什麼我的負載平衡器處於 `maintenance_pending` 狀態？

在各種維護活動期間，負載平衡器會處於 `maintenance_pending` 狀態，例如：
* 水平調整活動
* 回復活動
* 漸進式升級以處理漏洞及套用安全修補程式

### 為何需要在佈建期間選擇多個子網路？
{: #why-do-I-need-to-choose-multiple-subnets-during-provisioning}

**Load Balancer for VPC** 已準備好處理多區域地區 (MZR)。負載平衡器應用裝置會部署到您所選取的子網路。強烈建議您選擇不同區域中的子網路，以提供給您更高的可用性及備援。

### 為何我儲存區下的後端成員的性能是 `unknown`？

* 這個儲存區未與任何接聽器相關聯
* 該儲存區或與其相關聯接聽器的配置可能有變更

### 哪些 TLS 版本支援 SSL 卸載？
{: #which-tls-version-is-supported-with-ssl-offload}

**Load Balancer for VPC** 支援 TLS 1.2 搭配 SSL 終止。

下列清單詳述支援的密碼（依優先順序列出）：
* ECDHE-RSA-AES256-GCM-SHA384
* ECDHE-RSA-AES256-SHA384
* AES256-GCM-SHA384
* AES256-SHA256
* ECDHE-RSA-AES128-GCM-SHA256
* ECDHE-RSA-AES128-SHA256
* AES128-GCM-SHA256
* AES128-SHA256

### 性能檢查參數的預設值及容許值為何？
{: #what-are-the-default-settings-and-allowed-values-for-health-check-parameters}

預設值及容許值列出如下：
* 性能檢查間隔：預設為 5 秒，範圍是 2 到 60 秒。
* 性能檢查回應逾時：預設為 2 秒，範圍是 1 到 59 秒。
* 嘗試重試次數上限：預設為嘗試重試 2 次，範圍是重試 1 到 10 次。

性能檢查回應逾時值必須一律小於性能檢查間隔值。
{:note}

### 負載平衡器 IP 位址是固定的嗎？
{: #are-the-load-balancer-ip-addresses-fixed}

負載平衡器 IP 位址不保證是固定的，這是服務內建彈性的緣故。水平調整期間，您將會看到與負載平衡器的 FQDN 相關聯的可用 IP 發生變更。

不使用已快取 IP 位址，而是使用 FQDN。
{:note}

### 負載平衡器是否支援第 7 層切換？
{: #does-the-load-balancer-support-layer-7-switching}

可以。

### 為何建立或更新 HTTPS 接聽器時收到我的憑證無效的訊息？
{: #why-does-https-listener-creation-or-update-tell-me-that-my-certificate-is-invalid}

請檢查下列可能性：

* 提供的憑證 CRN 可能無效。
* Certificate Manager 中給定的憑證實例可能沒有相關聯的私密金鑰。
