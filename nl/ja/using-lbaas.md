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

# Load Balancers for VPC の使用
{: #--using-load-balancers-in-ibm-cloud-vpc}

{{site.data.keyword.cloud}} **Load Balancer for VPC** サービスは、VPC の同一リージョン内の複数のサーバー・インスタンス間にトラフィックを分散させます。

## パブリック・ロード・バランサー
{: #public-load-balancer}

ロード・バランサー・サービス・インスタンスには、パブリック・アクセスが可能な完全修飾ドメイン・ネーム (FQDN) が割り当てられます。IBM Cloud Load Balancer for VPC の背後でホストされているアプリケーションにアクセスする際にはこれを使用する必要があります。 このドメイン・ネームは、1 つ以上のパブリック IP アドレスに登録できます。

パブリック IP アドレスの場所と数は、メンテナンスとスケーリングの作業を行っていく過程で変更される可能性があります。 アプリケーションをホストするバックエンド・サーバー・インスタンス (VSI) は、同一 VPC 内の同一リージョンで実行する必要があります。

## プライベート・ロード・バランサー
{: #private-load-balancer}

プライベート・ロード・バランサーは、同じリージョンと VPC 内のプライベート・サブネット上の内部クライアントにのみアクセスできます。 プライベート・ロード・バランサーは、[RFC1918 ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")](https://tools.ietf.org/html/rfc1918){: new_window} アドレス・スペースからのトラフィックのみ受け入れます。

パブリック・ロード・バランサーと同様に、プライベート・ロード・バランサーのサービス・インスタンスにも完全修飾ドメイン・ネーム (FQDN) が割り当てられます。 ただし、このドメイン・ネームは 1 つ以上のプライベート IP アドレスに登録されます。

IBM Cloud の操作では、メンテナンスとスケーリングの作業を行っていく過程で、割り当てられるプライベート IP アドレスの数と値が変更される可能性があります。 アプリケーションをホストするバックエンド・サーバー・インスタンス (VSI) は、同一 VPC 内の同一リージョンで実行する必要があります。

以下の表は、各フィーチャーの比較を要約したものです。

| フィーチャー | パブリック・ロード・バランサー | プライベート・ロード・バランサー |
|--------|-------|-------|
| インターネットでアクセス可能ですか? |  はい。FQDN を使用して、インターネットでアクセス可能です | いいえ。同一のリージョンと VPC 上の内部クライアントのみアクセス可能です |
| すべてのトラフィックを受け入れますか? | はい | いいえ。RFC 1918 のみ受け入れます |
| プライベート IP の数 | 時間の経過とともに変更 | 時間の経過とともに変更 |
| ドメイン・ネームの登録方法 | パブリック IP アドレス | プライベート IP アドレス |

## フロントエンド・リスナーおよびバックエンド・プール
{: #front-end-listeners-and-back-end-pools}

最大 10 個のフロントエンド・リスナー (アプリケーション・ポート) を定義し、それらをバックエンド・アプリケーション・サーバーの対応するバックエンド・プールにマップできます。 ロード・バランサー・サービス・インスタンスに割り当てられている完全修飾ドメイン・ネーム (FQDN) とフロントエンド・リスナーのポートは、パブリック・インターネットに公開されます。 着信ユーザー要求はこれらのポートで受信されます。

サポートされるフロントエンド・リスナー・プロトコルは次のとおりです。
* HTTP
* HTTPS
* TCP

サポートされるバックエンド・プール・プロトコルは次のとおりです。
* HTTP
* TCP

着信 HTTPS トラフィックはロード・バランサーで終端処理されるので、バックエンド・サーバーとはプレーン・テキストの HTTP 通信が可能になります。

バックエンド・プールには最大 50 個のサーバー・インスタンスを接続できます。 接続される各サーバー・インスタンスではポートを構成しておく必要があります。 このポートは、フロントエンド・リスナー・ポートと同じ場合もあれば同じでない場合もあります。

ポート範囲 56500 から 56520 は管理目的用に予約されています。これらのポートは、フロントエンド・リスナー・ポートとしては使用できません。
{: note}

## レイヤー 7 ロード・バランシング
{: #layer-7-load-balancing}

パブリックとプライベートのロード・バランサーはどちらもレイヤー 7 のロード・バランシングをサポートします。データ・トラフィックは、構成したポリシーおよびルールに基づいて分散されます。_ポリシー_は、着信要求がポリシーに関連付けられたルールと一致する場合のアクション (つまりトラフィックを分散させる方法) を定義します。

### レイヤー 7 のポリシー
{: #layer-7-policy}

レイヤー 7 のポリシーは、リスナー (HTTP リスナーまたは HTTPS リスナーのみ) に関連付けられます。ポリシーごとにルールのセットを指定できます。ポリシーは、そのすべてのルールが一致した場合に**のみ**適用されます。

1 つのリスナーに複数のポリシーを関連付けることができます。一般には、優先順位が最も低いポリシーが最初に評価されます。優先順位はポリシーごとに固有でなければなりません。

どのポリシー・ルールにも一致しない着信要求は、リスナーのデフォルト・プールにリダイレクトされます (デフォルト・プールが構成されている場合)。

レイヤー 7 のポリシーでサポートされるアクションは以下のとおりです。

* **reject (拒否):** 要求は 403 応答で拒否されます。
* **redirect (リダイレクト):** 要求は、構成した URL および応答コードにリダイレクトされます。
* **forward (転送):** 要求は特定のバックエンド・プールに送信されます。

優先順位に関係なく、`reject` に設定されたポリシーが最初に評価されます。

その後、`redirect` に設定されたポリシーが評価されます。

最後に、`forward` に設定されたポリシーが評価されます。

各アクション・カテゴリー内では、ポリシーは優先順位の昇順に (最低から最高への順に) 評価されます。

### レイヤー 7 のポリシーのプロパティー
{: #layer-7-policy-properties}

プロパティー  | 説明
------------- | -------------
名前 | ポリシーの名前。この名前はリスナー内で固有でなければなりません。
アクション | すべてのポリシー・ルールが一致した場合に実行するアクション。許容値は `reject`、`redirect`、および `forward` です。`reject` アクションを指定したポリシーは、優先順位に関係なく常に最初に評価されます。`redirect` アクションを指定したポリシーが次に評価され、その後に `redirect` アクションを指定したポリシーが評価されます。
優先順位 | ポリシーは、優先順位の昇順に評価されます。 
URL | アクションを `redirect` に設定する場合に要求をリダイレクトする URL。
HTTP 状況コード | アクションを `redirect` に設定する場合にロード・バランサーから返す応答の状況コード。許容値は 301、302、303、307、または 308 です。
ターゲット | アクションを `forward` に設定する場合に要求を転送するサーバー・インスタンスのバックエンド・プール。

### レイヤー 7 のルール
{: #layer-7-rules}

レイヤー 7 のルールは、要求のマッチング方法を定義します。サポートされるタイプは、以下の 3 つです。

タイプ      |  説明
----------| -----------------------
`hostname` | 指定したホスト名 (例: `api.my_company.com`) で要求をマッチングします。
`header`    | HTTP ヘッダー・フィールド (例: `Cookie`) で要求をマッチングします。
`path`      | URL 内のパス (例: `/index.html`) で要求をマッチングします。

要求とマッチングするには、ルールに `condition` を定義する必要があります。以下の 3 つの条件がサポートされています。

条件 |  評価のタイプ
----------------|---------------------
`contains`        |  抽出されたフィールドに、指定した文字列が含まれているかどうかを検証します。
`equals`        |  抽出されたフィールドが、指定した文字列と同じかどうかを検証します。
`matches_regex`           |  抽出されたフィールドを、指定した正規表現とマッチングします。

## レイヤー 7 のルールのプロパティー
{: #layer-7-rule-properties}

プロパティー  | 説明
------------- | -------------
タイプ | ルールのタイプを指定します。 許容値は、`hostname`、`header`、または `path`です。
条件 | ルールを評価する条件を指定します。条件には、`contains`、`equals`、または `matches_regex` を指定できます。
フィールド |HTTP ヘッダーのフィールド名を指定します。このフィールドは、`header` ルール・タイプにのみ適用されます。例えば、HTTP ヘッダー内の Cookie とマッチングするには、このフィールドを`「Cookie」`に設定します。
値 |  マッチングされる値。

## ロード・バランシングの方式
{: #load-balancing-methods}

バックエンド・アプリケーション・サーバー間でトラフィックを分散するために、以下の 3 つのロード・バランシング方式が使用可能です。

* **ラウンドロビン:** ラウンドロビンがデフォルトのロード・バランシング方式です。 この方式では、ロード・バランサーは、着信クライアント接続をラウンドロビン形式でバックエンド・サーバーに転送します。 その結果として、すべてのバックエンド・サーバーは、ほぼ同数のクライアント接続を受信します。

* **重み付きラウンドロビン:** この方式では、ロード・バランサーは、バックエンド・サーバーに割り当てられた重みに比例して、着信クライアント接続をバックエンド・サーバーに転送します。 各サーバーには、デフォルトの重み 50 が割り当てられます。
この重みは、0 から 100 の間の任意の値にカスタマイズできます。

例えば、3 台のアプリケーション・サーバー A、B、および C の重みがそれぞれ 60、60、および 30 にカスタマイズされている場合、サーバー A と B は同数の接続を受信し、サーバー C はその半数の接続を受信します。

* **最小接続:** この方式では、特定の時間に処理している接続数が最も少ないサーバー・インスタンスが、次のクライアント接続を受信します。

**これらの方式のその他の特性:**

* サーバーの重みを「0」にリセットするということは、そのサーバーには新しい接続が転送されず、既存のトラフィックはすべてフローし続けることを意味します。 重み「0」を使用すると、サーバーを正常終了し、サービス・ローテーションからそのサーバーを削除するのに役立ちます。
* サーバーの重みの値は、「重み付きラウンドロビン」方式を使用する場合にのみ適用されます。 ラウンドロビンおよび最小接続のロード・バランシング方式では、これらの値は無視されます。

## 水平スケーリング
{: #horizontal-scaling}

ロード・バランサーは、負荷に応じて自動的にその容量を調整します。 このような調整が行われた場合は、ロード・バランサーの DNS 名に関連付けられている IP アドレスの数に変化が見られることがあります。

## ヘルス・チェック
{: #health-checks}

バックエンド・プールでは、ヘルス・チェックの定義は必須です。

ロード・バランサーは定期的にヘルス・チェックを実行して、バックエンド・ポートの正常性をモニターし、それに応じてそれらのポートにクライアント・トラフィックを転送します。 正常でないことが検出されたバックエンド・サーバー・ポートには、新しい接続は転送されません。 ロード・バランサーは正常でないポートの正常性を引き続きモニターします。ポートが再び正常な状態になる (つまり、ヘルス・チェックの試行が 2 回連続して合格になる) と、その使用が再開されます。

HTTP ポートおよび TCP ポートのヘルス・チェックは、以下のように実行されます。

* **HTTP:** 事前指定 URL に対する `HTTP GET` 要求がバックエンド・サーバー・ポートに送信されます。 `200 OK` 応答を受信すると、サーバー・ポートに正常のマークが付けられます。 UI ではデフォルトの `GET` ヘルス・パスは「/」ですが、これはカスタマイズできます。

* **TCP:** ロード・バランサーは、指定された TCP ポートでバックエンド・サーバーとの TCP 接続をオープンします。 接続試行が成功するとサーバー・ポートに正常のマークが付けられ、接続がクローズされます。

デフォルトのヘルス・チェック間隔は 5 秒、ヘルス・チェック要求に対するデフォルトのタイムアウトは 2 秒、デフォルトの再試行回数は 2 回です。
{: note}

## SSL オフロードおよび必要な許可
{: #ssl-offloading-and-required-authorizations}

すべての着信 HTTPS 接続について、ロード・バランサー・サービスは SSL 接続を終了し、バックエンド・サーバー・インスタンスとのプレーン・テキスト HTTP 通信を確立します。 この手法によって、バックエンド・サーバー・インスタンスでは、CPU 集中型の SSL ハンドシェークおよび暗号化/復号のタスクを行う必要がなくなり、アプリケーション・トラフィックの処理にすべての CPU サイクルを使用できるようになります。

ロード・バランサーで SSL オフロード・タスクを実行するには、SSL 証明書が必要です。 SSL 証明書は、[IBM Certificate Manager ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window} で管理できます。

ロード・バランサーが SSL 証明書にアクセスできるようにするには、**サービス間許可**を有効にする必要があります。有効にすると、ロード・バランサーのサービス・インスタンスが証明書マネージャー・インスタンスへのアクセスを許可されます。このような許可は、[サービス間のアクセスの認可 ![外部リンク・アイコン ](../icons/launch-glyph.svg "外部リンク・アイコン")](/docs/iam?topic=iam-serviceauth#create-auth){: new_window} の説明に従って管理できます。 ソース・サービスとして**「VPC インフラストラクチャー」**、リソース・タイプとして**「Load Balancer for VPC」**、ターゲット・サービスとして**「Certificate Manager」**を選択し、**「ライター」**サービス・アクセス役割を割り当ててください。

必要な許可が削除されると、ロード・バランサーでエラーが発生する可能性があります。
{: note}

## ID およびアクセスの管理 (IAM)
{: #identity-and-access-management-iam}

**Load Balancer for VPC** インスタンスのアクセス・ポリシーを構成できます。 ユーザーのアクセス・ポリシーを管理するには、[リソースに対するアクセス権限の管理 ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")](/docs/services/iam?topic=iam-iammanidaccser#resourceaccess){: new_window} を参照して、ID およびアクセスの管理についての詳細を確認してください。

### ユーザーのリソース・グループ・アクセス・ポリシーの構成
{: #configuring-resource-group-access-policies-for-users}

ロード・バランサーを作成するには、リソース・グループにアクセスする必要があります。 ロード・バランサーを作成するユーザーには、指定されたリソース・グループに対する適切なアクセス権限が必要です。リソース・グループが指定されていない場合には、デフォルトのリソース・グループに対する適切なアクセス権限が必要です。

1. **「管理」>「アカウント」>「ユーザー」**に移動します。 IBM Cloud アカウントへのアクセス権限を持つユーザーのリストが表示されます。
2. アクセス・ポリシーの割り当て対象ユーザーの名前を選択します。 ユーザーが表示されていない場合は、**「ユーザーの招待」**をクリックして、IBM Cloud アカウントにユーザーを追加します。
3. **「アクセス権限の割り当て」**を選択します。
4. **「リソース・グループ内のアクセス権限の割り当て」**を選択します。
5. **「リソース・グループ」**ドロップダウン・リストから、必要なリソース・グループを選択します。
6. **「リソース・グループへのアクセス権限の割り当て」**ドロップダウン・リストから、必要なアクセス権限を選択します。
7. **「サービス」**ドロップダウン・リストから、必要なサービスを選択します。
9. **「割り当て」**を選択して、ユーザーにリソース・グループ・アクセス・ポリシーを割り当てます。

### ユーザーのリソース・アクセス・ポリシーの構成
{: #configuring-resource-access-policies-for-users}

| プラットフォームのアクセス役割 | ロード・バランサーのアクション |
|-------------|-----|
| 管理者 | ロード・バランサーの作成/表示/編集/削除 |
| エディター | ロード・バランサーの作成/表示/編集/削除 |
| ビューアー | ロード・バランサーの表示 |

1. **「管理」>「アカウント」>「ユーザー」**に移動します。 IBM Cloud アカウントへのアクセス権限を持つユーザーのリストが表示されます。
2. アクセス・ポリシーの割り当て対象ユーザーの名前を選択します。 ユーザーが表示されていない場合は、**「ユーザーの招待」**をクリックして、IBM Cloud アカウントにユーザーを追加します。
3. **「アクセス権限の割り当て」**を選択します。
4. **「リソースへのアクセス権限の割り当て」**を選択します。
5. **「サービス」**ドロップダウン・リストから**「VPC インフラストラクチャー」**を選択します。
6. **「リソース・タイプ」**ドロップダウン・リストから**「Load Balancer for VPC」**を選択します。
7. **「ロード・バランサー ID (Load Balancer ID)」**ドロップダウン・リストから、ロード・バランサー・インスタンス ID を選択するか、デフォルト値の**「すべてのロード・バランサー」**を使用します。
8. プラットフォームのアクセス役割をユーザーに割り当てます。
9. **「割り当て」**を選択して、ユーザーにアクセス・ポリシーを割り当てます。

## Activity Tracker との統合
{: #activity-tracker-integration}

ロード・バランサー・サービスは、CADF 標準に準拠する方法でイベントを記録する **IBM Cloud Activity Tracker** と統合されています。これらのイベントは、クラウド内のサービスの状態を変更するユーザー開始アクティビティーによってトリガーされます。

ロード・バランサー・サービス・インスタンスの監査イベントとして記録されるアクションの詳細なリストについては、[Activity tracker with LogDNA events](/docs/vpc-on-classic?topic=vpc-on-classic-at-events#events-load-balancers) を参照してください。

すべての監査イベントは `us-south` リージョンの「IBM Cloud Activity Tracker with LogDNA」に記録されます。ロード・バランサー・サービスがどのリージョンでプロビジョンされているかは関係ありません。
{:note}

イベントを表示するには、アカウントで `us-south` リージョンに「IBM Cloud Activity Tracker with LogDNA」インスタンスをプロビジョンする必要があります。アカウント内のユーザーは、**「ビューアー」**プラットフォーム・アクセス役割と、「IBM Cloud Activity Tracker with LogDNA」インスタンスに対する**「リーダー」**サービス・アクセス役割を付与する IAM ポリシーを持っている必要があります。

アクセス権限の付与について詳しくは、[アカウント・イベントを表示する許可の付与 ![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")](https://cloud.ibm.com/docs/services/cloud-activity-tracker/how-to/grant_permissions.html#grant_permissions){: new_window} を参照してください。

IBM Cloud アカウントのユーザーは、ロード・バランサー・サービスで実行されたアカウント・レベルの操作をモニターできます。
{: tip}

アカウントで「IBM Cloud Activity Tracker with LogDNA」インスタンスをプロビジョンするには、以下の手順を実行します。

1. IBM Cloud コンソールにログインします。[IBM Cloud コンソールにログインします。![外部リンク・アイコン](../icons/launch-glyph.svg "外部リンク・アイコン")](https://cloud.ibm.com/){: new_window}
2. 左上にある ![メニュー・アイコン](../../icons/icon_hamburger.svg) をクリックします。そこから、**「プログラム識別情報」>「Activity Tracker」**を選択します。
3. 右上の**「インスタンスの作成」**をクリックします。
4. サービス名を定義します。
5. リージョンとして `us-south` を選択し、リソース・グループを選択します。
6. 有料アカウントの場合は、`ライト`以外のプランを選択してください。
7. **「作成」**をクリックします。

以下に、**リスナーの作成**操作についての Activity Tracker のサンプル・メッセージを示します。
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

## 使用可能な API
{: #lbaas-apis-available}

API 呼び出しを行うには、何らかの形式の REST クライアントを使用する必要があります。 例えば、既存のすべてのロード・バランサーを取得するには `curl` コマンドを使用できます。

```
curl -X GET "$rias_endpoint/v1/load_balancers?version=2019-05-31&generation=1" -H "Authorization: $iam_token"
```
{: pre}

次のセクションで、VPC 環境でロード・バランサーに使用できる API について詳しく説明します。 完全な仕様については、[VPC on Classic API リファレンス](https://{DomainName}/apidocs/vpc-on-classic#list-all-load-balancers)を参照してください。

| 説明 | API |
|-------------|-----|
| ロード・バランサーの作成とプロビジョン | POST /load_balancers |
| すべてのロード・バランサーの取得 | GET /load_balancers |
| 1 つのロード・バランサーの取得 | GET /load_balancers/{id} |
| ロード・バランサーの削除 | DELETE /load_balancers/{id} |
| ロード・バランサーの更新 | PATCH /load_balancers/{id} |
| リスナーの作成 | POST /load_balancers/{id}/listeners |
| ロード・バランサーのすべてのリスナーの取得 | GET /load_balancers/{id}/listeners |
| リスナーの取得 | GET /load_balancers/{id}/listeners/{listener_id} |
| リスナーの削除 | DELETE /load_balancers/{id}/listeners/{listener_id} |
| リスナーの更新 | PATCH /load_balancers/{id}/listeners/{listener_id} |
| プールの作成 | POST /load_balancers/{id}/pools |
| ロード・バランサーのすべてのプールの取得 | GET /load_balancers/{id}/pools |
| プールの取得 | GET /load_balancers/{id}/pools/{pool_id} |
| プールの削除 | DELETE /load_balancers/{id}/pools/{pool_id} |
| プールの更新 | PATCH /load_balancers/{id}/pools/{pool_id} |
| メンバーの作成 | POST /load_balancers/{id}/pools/{pool_id}/members |
| プールのすべてのメンバーの取得 | GET /load_balancers/{id}/pools/{pool_id}/members |
| メンバーの取得 |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| プールからのメンバーの削除 | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| メンバーの更新 | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| プールのメンバーの更新 | PUT /load_balancers/{id}/pools/{pool_id}/members |
| ロード・バランサーの統計の取得 | GET /load_balancers/{id}/statistics |
| リスナーのすべてのポリシーの取得 |  GET /load_balancers/{id}/listeners/{listener_id}/policies
| リスナーのポリシーの作成 | POST /load_balancers/{id}/listeners/{listener_id}/policies
| リスナーのポリシーの削除 | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| リスナーのポリシーの取得 | GET /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| リスナーのポリシーの更新 | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| ポリシーに関連付けられたすべてのルールの取得 | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| ポリシーのルールの作成 | POST /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| ポリシーのルールの削除 | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| ポリシーのルールの取得 | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| ポリシーのルールの更新 | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}

## ロード・バランサーの例
{: #load-balancer-example}

以下の例では、API を使用してロード・バランサーを作成します。これは、ポート `80` で listen する Web アプリケーションを実行する 2 つの VPC サーバー・インスタンス (`192.168.100.5` および `192.168.100.6`) の前に配置されます。 ロード・バランサーにはフロントエンド・リスナーがあるので、HTTPS を使用して Web アプリケーションに安全にアクセスできます。 次いでその API を使用して、作成されたロード・バランサー・インスタンスの詳細を取得し、そのロード・バランサー・インスタンスを削除できます。

### 手順の例
{: #lbaas-example-steps}

以下の手順の例では、[IBM Cloud UI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console)、[CLI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli)、または [VPC on Classic API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis) を使用して VPC、サブネット、およびインスタンスをプロビジョンするという前提条件となる手順を省略しています。

このロード・バランサーの例の手順は、[CLI](/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference) を使用して実行することもできます。
{: note}

**手順 1. リスナー、プール、および接続対象サーバー・インスタンス (プール・メンバー) を指定してロード・バランサーを作成する**

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

サンプル出力:
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

次の手順で使用するために、ロード・バランサーの ID を変数 `lbid` などに保存してください。

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

**手順 2. ロード・バランサーを取得する**

```
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

プロビジョニングにはある程度時間がかかることを想定しておいてください。 ロード・バランサーの準備ができると、次の出力例に示されているように、`online` および `active` の状況に設定されます。

サンプル出力:

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

**手順 3. ロード・バランサーを削除する**

```bash
curl -H "Authorization: $iam_token" -X DELETE "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

## レイヤー 7 の例: ポリシーおよびルールの作成
{: #layer-7-examples-create-policy-and-rules}

以下の 2 つの例は、ポリシーおよびルールを作成してリスナーに関連付ける手順を示しています。

### 例 1: ポリシーおよびルールを指定して HTTPS リスナーを作成する。ポリシーは、アクション `Redirect` を指定して作成する

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

### 例 2: プールに転送するアクション `Forward` を指定してポリシーを作成し、既存のリスナーに関連付ける
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

## FAQ
{: #load-balancer-faqs}

このセクションでは、**Load Balancer for VPC** サービスについてよくある質問への回答を記載しています。

### 自分のロード・バランサーに対して別の DNS 名を使用できますか。
{: #can-i-use-a-different-dns-name-for-my-load-balancer}

ロード・バランサーに自動的に割り当てられる DNS 名はカスタマイズできません。 ただし、使用したい DNS 名を、自動的に割り当てられたロード・バランサー DNS 名に差し向ける CNAME (Canonical Name) レコードを追加できます。 例えば、`us-south` のロード・バランサーの ID が `dd754295-e9e0-4c9d-bf6c-58fbc59e5727` である場合、自動的に割り当てられるロード・バランサー DNS 名は `dd754295-us-south.lb.appdomain.cloud` です。優先的に `www.myapp.com` という DNS 名を使用したいとします。 その場合は、(`myapp.com` の管理に使用している DNS プロバイダーで) `www.myapp.com` をロード・バランサー DNS 名 `dd754295-us-south.lb.appdomain.cloud` に差し向ける CNAME レコードを追加します。

### ロード・バランサー・サービスで定義できるフロントエンド・リスナーの最大数はいくつですか?
{: #what-s-the-maximum-number-of-front-end-listeners-i-can-define-with-my-load-balancer}

10 個です。

### バックエンド・プールに接続できるサーバー・インスタンスの最大数はいくつですか?
{: #what-s-the-maximum-number-of-server-instances-i-can-attach-to-my-back-end-pool}

50 個です。

### ロード・バランサーは水平方向にスケーラブルですか?
{: #is-the-load-balancer-horizontally-scalable}

はい。 ロード・バランサーは負荷に応じてその容量を自動的に調整します。 水平スケーリングが行われるときに、ロード・バランサーの DNS 名に関連付けられている IP アドレスの数が変わります。

### ロード・バランサーのデプロイに使用するサブネットで ACL またはセキュリティー・グループを使用している場合は、どうしたらよいですか?
{: #what-should-i-do-if-i-am-using-acls-or-security-groups-on-the-subnets-that-are-used-to-deploy-the-load-balancer}

構成されているリスナー・ポートおよび管理ポート (ポート 56500 から 56520) の着信トラフィックを許可するために、適切な ACL またはセキュリティー・グループ・ルールが設定されていることを確認する必要があります。 ロード・バランサーとバックエンド・インスタンス間のトラフィックも許可する必要があります。

### エラー・メッセージ`「証明書インスタンスが見つかりませんでした (certificate instance not found)」`を受け取るのはなぜですか?

* 証明書インスタンス CRN が無効である可能性があります。
* **サービス間許可**を付与していない可能性があります。 この資料の **SSL オフロード**のセクションを参照してください。

### `401 Unauthorized Error` コードが表示されるのはなぜですか?

ユーザーの次のアクセス・ポリシーを確認してください。
* ロード・バランサーのリソース・タイプのアクセス・ポリシー
* リソース・グループのアクセス・ポリシー
* `HTTPS` リスナーを使用する場合は、証明書マネージャー・インスタンスのサービス間許可も確認してください。

### ロード・バランサーが `maintenance_pending` 状態になるのはなぜですか?

ロード・バランサーは、さまざまなメンテナンス・アクティビティーの実施中に `maintenance_pending` 状態になります。次に例を示します。
* 水平スケーリング・アクティビティー
* リカバリー・アクティビティー
* 脆弱性に対処してセキュリティー・パッチを適用するためのローリング・アップグレード

### プロビジョン中に複数のサブネットを選択する必要があるのはなぜですか?
{: #why-do-I-need-to-choose-multiple-subnets-during-provisioning}

**Load Balancer for VPC** はマルチ・ゾーン・リージョン (MZR) に対応しています。 ロード・バランサー・アプライアンスは、選択したサブネットにデプロイされます。 可用性と冗長性を強化するために、異なるゾーンのサブネットを選択することを強くお勧めします。

### プールのバックエンド・メンバーの正常性が `unknown` になるのはなぜですか?

* プールがどのリスナーにも関連付けられていません
* プールまたはその関連リスナーの構成が変更された可能性があります

### SSL オフロードでは、どの TLS バージョンがサポートされますか?
{: #which-tls-version-is-supported-with-ssl-offload}

**Load Balancer for VPC** は、TLS 1.2 を SSL 終端処理とともにサポートしています。

以下のリストは、サポートされる暗号の詳細を示しています (優先順でリスト)。
* ECDHE-RSA-AES256-GCM-SHA384
* ECDHE-RSA-AES256-SHA384
* AES256-GCM-SHA384
* AES256-SHA256
* ECDHE-RSA-AES128-GCM-SHA256
* ECDHE-RSA-AES128-SHA256
* AES128-GCM-SHA256
* AES128-SHA256

### ヘルス・チェック・パラメーターのデフォルト設定と許可される値を教えてください。
{: #what-are-the-default-settings-and-allowed-values-for-health-check-parameters}

デフォルト設定および許可される値を以下にリストします。
* ヘルス・チェックの間隔: デフォルトは 5 秒。2 秒から 60 秒の範囲。
* ヘルス・チェックの応答タイムアウト: デフォルトは 2 秒。1 秒から 59 秒の範囲。
* 最大再試行回数: デフォルトは 2 回の再試行。1 回から 10 回の範囲。

ヘルス・チェックの応答タイムアウトの値は、常にヘルス・チェックの間隔の値よりも小さくなければなりません。
{:note}

### ロード・バランサー IP アドレスは固定ですか?
{: #are-the-load-balancer-ip-addresses-fixed}

ロード・バランサー IP アドレスは固定であるとは限りません。これは、サービスに弾力性を組み込むためです。 水平スケーリング時に、ロード・バランサーの FQDN に関連付けられた使用可能な IP が変化することが分かります。

キャッシュされた IP アドレスではなく、FQDN を使用してください。
{:note}

### ロード・バランサーはレイヤー 7 のスイッチングをサポートしますか?
{: #does-the-load-balancer-support-layer-7-switching}

はい。

### HTTPS リスナーの作成や更新の際に、証明書が無効であると示されるのはなぜですか?
{: #why-does-https-listener-creation-or-update-tell-me-that-my-certificate-is-invalid}

以下の可能性を検討してください。

* 指定された証明書 CRN が無効の可能性があります。
* 証明書マネージャーに指定された証明書インスタンスに、秘密鍵が関連付けられていない可能性があります。
