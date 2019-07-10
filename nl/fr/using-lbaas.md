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

# Utilisation du service d'équilibrage de charge du VPC
{: #--using-load-balancers-in-ibm-cloud-vpc}

Le service d'**équilibrage de charge {{site.data.keyword.cloud}} pour VPC** répartit le trafic entre plusieurs instances de serveur dans la région de votre VPC. 

## Equilibreur de charge public
{: #public-load-balancer}

Votre instance de service d'équilibreur de charge se voit attribuer un nom de domaine complet (FQDN) accessible publiquement, que vous devez utiliser pour accéder à vos applications hébergées derrière l'équilibreur de charge IBM Cloud pour VPC. Vous pouvez enregistrer ce nom de domaine avec une ou plusieurs adresses IP publiques.

Au fil du temps, ces adresses IP publiques et le nombre d'adresses IP publiques peuvent changer en raison des activités de maintenance et de mise à l'échelle. Les instances de serveur de back end (VSI) hébergeant votre application doivent s'exécuter dans la même région et sous le même VPC.

## Equilibreur de charge privé
{: #private-load-balancer}

L'équilibreur de charge privé est accessible uniquement par les clients internes qui se trouvent sur vos sous-réseaux privés, dans la même région et le même VPC. Il accepte uniquement le trafic provenant des espaces d'adresses [RFC1918 ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")](https://tools.ietf.org/html/rfc1918){: new_window}.

Tout comme dans le cas d'un équilibreur de charge public, un nom de domaine complet est affecté à votre instance de service d'équilibreur de charge privé. Toutefois, ce nom de domaine est enregistré avec une ou plusieurs adresses IP publiques.

Les opérations IBM Cloud peuvent modifier le nombre et la valeur des adresses IP privées attribuées au fil du temps, en fonction des activités de maintenance et de mise à l'échelle. Les instances de serveur de back end (VSI) hébergeant votre application doivent s'exécuter dans la même région et sous le même VPC.

Consultez le tableau ci-après qui récapitule la comparaison des fonctions.

| Fonction | Equilibreur de charge public | Equilibreur de charge privé |
|--------|-------|-------|
| Accessible sur Internet ? |  Oui, avec un nom de domaine complet, via Internet | Non, uniquement par les clients internes, dans les mêmes région et VPC |
| Accepte tout le trafic ? | Oui | Non, RFC 1918 uniquement |
| Nombre d'adresses IP privées | Change au fil du temps | Change au fil du temps |
| Comment est enregistré le nom de domaine ? | Adresses IP publiques | Adresses IP privées |

## Programmes d'écoute frontaux et pools de back end
{: #front-end-listeners-and-back-end-pools}

Vous pouvez définir jusqu'à dix (10) programmes d'écoute frontaux (ports d'application) et les mapper aux pools de back end respectifs sur les serveurs d'applications de back end. Le nom de domaine complet (FQDN) attribué à votre instance de service d'équilibreur de charge et les ports d'écoute frontaux sont exposés à l'Internet public. Les demandes d'utilisateurs entrantes sont reçues sur ces ports.

Les protocoles d'écoute frontaux pris en charge sont les suivants :
* HTTP
* HTTPS
* TCP

Les protocoles de pool de back end pris en charge sont les suivants :
* HTTP
* TCP

Le trafic HTTPS entrant se termine au niveau de l'équilibreur de charge pour permettre une communication HTTP en texte brut avec le serveur de back end.

Vous pouvez associer jusqu'à cinquante (50) instances de serveur à un pool de back end. Chaque instance de serveur connecté doit avoir un port configuré. Le port peut être ou ne pas être identique au port d'écoute frontal.

La plage de ports de 56500 à 56520 est réservée à des fins de gestion ; ces ports ne peuvent pas être utilisés comme ports d'écoute frontaux.
{: note}

## Equilibrage de charge de couche 7
{: #layer-7-load-balancing}

Les équilibreurs de charge publics et privés prennent en charge l'équilibrage de charge de couche 7. Le trafic de données est distribué en fonction des stratégies et des règles configurées. Une _règle_ définit l'action à effectuer lorsque la demande entrante correspond aux règles associées à la stratégie. L'action indique la manière dont le trafic est distribué.

### Stratégie de couche 7
{: #layer-7-policy}

Une stratégie de couche 7 est associée à un programme d'écoute, et uniquement à un programme d'écoute HTTP ou HTTPS. Chaque police peut avoir un ensemble de règles. La stratégie est **uniquement** appliquée lorsque toutes ses règles correspondent.

Vous pouvez joindre plus d'une police à un programme d'écoute. En général, la stratégie dont la priorité est la moins élevée est évaluée en premier. La priorité doit être unique pour une stratégie donnée.

Si la requête entrante ne correspond à aucune des règles de la stratégie, la requête est redirigée vers le pool par défaut du programme d'écoute, s'il le pool par défaut est configuré.

Les actions prises en charge pour une stratégie de couche 7 sont les suivantes :

* **reject :** la requête est rejetée avec une réponse 403.
* **redirect :** la requête est réacheminée à une adresse URL et à un code de réponse configurés.
* **forward :** la requête est envoyée à un pool de back end spécifique.

Les stratégies associées à l'action `reject` sont évaluées en premier, quelle que soit leur priorité.

Après cela, les stratégies définies sur `redirect` sont évaluées.

Enfin, les stratégies définies sur `forward` sont évaluées.

Au sein de chaque catégorie d'action, les stratégies sont évaluées par ordre croissant de priorité (de la plus basse vers la plus élevée).

### Propriétés de la stratégie de couche 7
{: #layer-7-policy-properties}

Propriété  | Description
------------- | -------------
Nom | Nom de la stratégie. Le nom doit être unique au sein du programme d'écoute.
Action | Action à entreprendre lorsque toutes les règles conditionnelles correspondent. Les valeurs acceptables sont `reject`, `redirect` et `forward`. Une stratégie avec l'action `reject` est toujours évaluée en premier, quelle que soit sa priorité. Les stratégies avec les actions `redirect` sont évaluées par la suite, suivies des stratégies avec les actions `redirect`.
Priorité | Les stratégies sont évaluées par ordre ascendant de priorité. 
URL | Adresse URL vers laquelle la demande est redirigée, si l'action est définie sur `redirect`.
Code de statut HTTP | Code de statut de la réponse renvoyé par l'équilibreur de charge lorsque l'action est définie sur `redirect`. Les valeurs acceptables sont : 301, 302, 303, 307 ou 308.
Cible | Pool de back end des instances de serveur vers lesquelles la demande est transférée, si l'action est définie sur `forward`.

### Règles de couche 7 
{: #layer-7-rules}

Une règle de couche 7 définit ce à quoi une demande doit correspondre. Trois types sont pris en charge :

Type      |  Description
----------| -----------------------
`hostname` | La demande correspond au nom d'hôte spécifié (par exemple, `api.my_company.com`).
`header`    | La demande correspond à une zone d'en-tête HTTP (par exemple, `Cookie`).
`path`      | La demande correspond au chemin d'accès contenu dans l'URL (par exemple, `/index.html`).

Pour qu'une demande corresponde, une `condition` doit être définie dans la règle. Trois conditions sont prises en charge :

Condition |  Type d'évaluation
----------------|---------------------
`contains`        |  Vérifie si la zone extraite contient la chaîne fournie.
`equals`        |  Vérifie si la zone extraite est identique à la chaîne fournie.
`matches_regex`           |  Fait correspondre la zone extraite avec l'expression régulière fournie.

## Propriétés de la règle de couche 7
{: #layer-7-rule-properties}

Propriété  | Description
------------- | -------------
Type | Spécifie le type de règle. Les valeurs acceptables sont `hostname`, `header` ou `path`.
Condition | Spécifie la condition avec laquelle une règle est évaluée. La condition peut être : `contains`, `equals` ou `matches_regex`.
Zone | Spécifie le nom de la zone d'en-tête HTTP. La zone est uniquement applicable au type de règle `header`. Par exemple, pour faire correspondre un cookie dans l'en-tête HTTP, la zone doit être définie sur `Cookie`.
Valeur |  La valeur devant correspondre.

## Méthodes d'équilibrage de charge
{: #load-balancing-methods}

Les trois méthodes d'équilibrage de charge ci-dessous peuvent être utilisées pour répartir le trafic entre les serveurs d'applications de back end :

* **Circulaire :** Il s'agit de la méthode d'équilibrage de charge par défaut. Avec cette méthode, l'équilibreur de charge achemine les connexions client entrantes de façon circulaire vers les serveurs de back end. Chaque serveur de back end reçoit ainsi un nombre équivalent de connexions client.

* **Circulaire pondérée :** Avec cette méthode, l'équilibreur de charge achemine les connexions client entrantes vers les serveurs de back end au prorata du nombre de connexions affectées à ces serveurs. Chaque serveur reçoit une pondération par défaut de 50 connexions, qui peut être personnalisée sur n'importe quelle valeur comprise entre 0 et 100.

Par exemple, si trois serveurs d'applications A, B et C ont des pondérations personnalisées à 60, 60 et 30 respectivement, les serveurs A et B reçoivent un nombre égal de connexions, tandis que le serveur C reçoit la moitié de ce nombre de connexions.

* **Connexions minimum :** Avec cette méthode, l'instance de serveur qui prend en charge le plus petit nombre de connexions, à un moment donné, reçoit la connexion suivante.

**Caractéristiques supplémentaires de ces méthodes :**

* Si vous réinitialisez le poids d'un serveur sur '0', cela signifie qu'aucune nouvelle connexion n'est transmise à ce serveur, mais que tout le trafic existant continue de circuler. Utiliser un poids égal à '0' peut aider à mettre un serveur hors service sans heurt et à le retirer de la rotation du service.
* Les valeurs de pondération des serveurs ne sont applicables qu'avec la méthode circulaire pondérée. Elles sont ignorées avec les méthodes d'équilibrage de charge Circulaire et Connexions minimum.

## Mise à l'échelle horizontale
{: #horizontal-scaling}

L'équilibreur de charge ajuste automatiquement sa capacité en fonction de la charge. Lorsque cet ajustement se produit, vous pouvez remarquer un changement au niveau du nombre d'adresses IP associées au nom DNS de l'équilibreur de charge. 

## Diagnostics d'intégrité
{: #health-checks}

Les définitions de diagnostic d'intégrité sont obligatoires pour les pools de back end.

L'équilibreur de charge effectue des diagnostics d'intégrité périodiques pour surveiller l'intégrité des ports de back end et il leur transmet le trafic client en conséquence. Si un port de serveur de back end donné est jugé défaillant, aucune nouvelle connexion ne lui est transmise. L'équilibreur de charge continue de surveiller l'intégrité des ports défectueux et reprend leur utilisation s'ils redeviennent sains, à savoir s'ils ont réussi deux tentatives de diagnostic d'intégrité consécutives.

Les diagnostics d'intégrité des ports HTTP et TCP sont effectués comme suit :

* **HTTP :** Une demande `HTTP GET` fondée sur une URL prédéfinie est envoyée au port du serveur de back end. Le port est marqué comme intègre lorsqu'il reçoit une réponse `200 OK`. Le chemin d'intégrité `GET` par défaut est "/" via l'interface utilisateur et peut être personnalisé.

* **TCP :** L'équilibreur de charge tente d'établir une connexion TCP au serveur de back end sur un port TCP spécifique. Le port du serveur est marqué comme intègre si la tentative de connexion aboutit, puis la connexion est fermée.

L'intervalle de diagnostic d'intégrité par défaut est de 5 secondes, le délai d'attente par défaut pour une demande de diagnostic d'intégrité est de 2 secondes et le nombre de nouvelles tentatives par défaut est de 2.
{: note}

## Déchargement SSL et autorisations requises
{: #ssl-offloading-and-required-authorizations}

Pour toutes les connexions HTTPS entrantes, le service d'équilibreur de charge met fin à la connexion SSL et établit une communication HTTP en texte brut avec l'instance du serveur back end. Avec cette technique, les établissements de liaison SSL et les tâches de chiffrement ou de déchiffrement,consommateurs de ressources processeur, sont déplacés des instances de serveur back end, leur permettant ainsi d'utiliser tous leurs cycles de processeurs pour traiter le trafic des applications.

Un certificat SSL est nécessaire pour que l'équilibreur de charge procède aux tâches de déchargement SSL. Vous pouvez gérer les certificats SSL via [IBM Certificate Manager![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window}.

Pour qu'un équilibreur de charge puisse accéder à votre certificat SSL, vous devez activer l'**autorisation de service à service**, qui permet à votre instance de service d'équilibreur de charge d'accéder à votre instance Certificate Manager. Vous pouvez gérer ce type d'autorisation en suivant les instructions de la documentation [Octroi d'accès entre services ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")](/docs/iam?topic=iam-serviceauth#create-auth){: new_window}. Sélectionnez **Infrastructure VPC** en tant que service source, **Equilibreur de charge pour VPC** en tant que type de ressource, **Certificate Manager** en tant que service cible et attribuez le Rôle Accès au service **Auteur**. 

Si l'autorisation requise est supprimée, des erreurs peuvent survenir sur l'équilibreur de charge.
{: note}

## Identity and Access Management (IAM)
{: #identity-and-access-management-iam}

Vous pouvez configurer des règles d'accès pour une instance **Equilibreur de charge pour VPC**. Pour gérer vos règles d'accès utilisateur,
rendez-vous sur [Gestion de l'accès aux ressources ![Icône de lien externe](../icons/launch-glyph.svg "Icônede lien externe")](/docs/services/iam?topic=iam-iammanidaccser#resourceaccess){: new_window} pour plus d'informations sur la gestion des identités et des accès.


### Configuration des règles d'accès aux groupes de ressources pour les utilisateurs
{: #configuring-resource-group-access-policies-for-users}

Pour créer un équilibreur de charge, vous devez avoir accès à un groupe de ressources. L'utilisateur qui crée un équilibreur de charge doit avoir un accès approprié au groupe de ressources fourni ou au groupe de ressources par défaut, si aucun n'est fourni.

1. Accédez à **Gérer > Compte > Utilisateurs**. La liste des utilisateurs ayant accès à votre compte IBM Cloud apparaît.
2. Sélectionnez le nom de l'utilisateur auquel vous souhaitez attribuer une règle d'accès. Si l'utilisateur n'est pas affiché, cliquez sur **Inviter des utilisateurs** pour ajouter l'utilisateur à votre compte IBM Cloud.
3. Sélectionnez **Affecter un accès**.
4. Sélectionnez **Affecter l'accès au sein d'un groupe de ressources**.
5. Dans la liste déroulante **Groupe de ressources**, sélectionnez le groupe de ressources souhaité.
6. Dans la liste déroulante **Affecter l'accès à un groupe de ressources**, sélectionnez l'accès souhaité.
7. Dans la liste déroulante **Services**, sélectionnez les services souhaités.
9. Sélectionnez **Affecter** pour affecter la règle d'accès au groupe de ressources à l'utilisateur.

### Configuration des règles d'accès aux ressources pour les utilisateurs
{: #configuring-resource-access-policies-for-users}

| Rôle d'accès à la plateforme | Action de l'équilibreur de charge |
|-------------|-----|
| Administrateur | Créer/Afficher/Modifier/Supprimer un équilibreur de charge |
| Editeur | Créer/Afficher/Modifier/Supprimer un équilibreur de charge |
| Afficheur | Afficher l'équilibreur de charge |

1. Accédez à **Gérer > Compte > Utilisateurs**. La liste des utilisateurs ayant accès à votre compte IBM Cloud apparaît.
2. Sélectionnez le nom de l'utilisateur auquel vous souhaitez attribuer une règle d'accès. Si l'utilisateur n'est pas affiché, cliquez sur **Inviter des utilisateurs** pour ajouter l'utilisateur à votre compte IBM Cloud.
3. Sélectionnez **Affecter un accès**.
4. Sélectionnez **Affecter l'accès aux ressources**.
5. Dans la liste déroulante **Services**, sélectionnez **Infrastructure VPC**.
6. Dans la liste déroulante **Type de ressource**, sélectionnez **Equilibreur de charge pour VPC**.
7. Dans la liste déroulante **ID d'équilibreur de charge**, sélectionnez un ID d'instance d'équilibreur de charge ou utilisez la valeur par défaut, **Tous les équilibreurs de charge**.
8. Attribuez un rôle d'accès à la plateforme à l'utilisateur.
9. Sélectionnez **Affecter** pour affecter la règle d'accès à l'utilisateur.

## Intégration du dispositif de suivi des activités
{: #activity-tracker-integration}

Le service d'équilibrage de charge est intégré à **IBM Cloud Activity Tracker with LogDNA**, qui enregistre les événements déclenchés par les activités initiées par l'utilisateur qui changent l'état du service dans le cloud, conformément à la norme CADF.

Pour obtenir une liste détaillée des actions qui sont enregistrées en tant qu'événements d'audit sur les instances du service d'équilibrage de charge, veuillez vous reporter à [Evénements d'Activity tracker with LogDNA](/docs/vpc-on-classic?topic=vpc-on-classic-at-events#events-load-balancers).

Tous les événements d'audit sont enregistrés dans "IBM Cloud Activity Tracker with LogDNA" dans la région `us-south`, quelle que soit la région dans laquelle le service d'équilibreur de charge est mis à disposition.
{:note}

Pour afficher les événements, vous devez mettre à disposition une instance "IBM Cloud Activity Tracker with LogDNA" dans la région `us-south` sous votre compte. Les utilisateurs de votre compte doivent avoir un stratégie IAM qui accorde le rôle d'accès **Viewer** à la plateforme et le rôle d'accès **Reader** au service sur l'instance "IBM Cloud Activity Tracker with LogDNA".

Vous trouverez plus d'informations sur l'octroi d'accès dans la rubrique [Octroi de
droits pour l'affichage d'événements de compte. ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")](https://cloud.ibm.com/docs/services/cloud-activity-tracker/how-to/grant_permissions.html#grant_permissions){: new_window}

Les utilisateurs du compte IBM Cloud peuvent surveiller les opérations qui sont effectuées sur le service d'équilibrage de charge au niveau du compte.
{: tip}

Suivez les étapes décrites ci-dessous pour mettre à disposition une instance "IBM Cloud Activity Tracker with LogDNA" sous votre compte :

1. Connectez-vous à la console IBM Cloud. [Connectez-vous à la console IBM Cloud. ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")](https://cloud.ibm.com/){: new_window}
2. Cliquez sur ![icône Menu](../../icons/icon_hamburger.svg) dans le coin supérieur gauche. Dans le menu, sélectionnez **Observabilité > Activity Tracker**.
3. Dans le coin supérieur droit, cliquez sur **Créer une instance**.
4. Définissez un nom de service.
5. Sélectionnez la région `us-south` et le groupe de ressources
6. Choisissez un plan autre que `lite` si vous avez un compte payant.
7. Cliquez sur **Créer**.

Exemple de message de suivi de l'activité pour une opération de **création d'un programme d'écoute** :
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

## API disponibles
{: #lbaas-apis-available}

Pour effectuer des appels d'API, vous devez utiliser une forme de client REST. Par exemple, vous pouvez utiliser la commande `curl` pour récupérer tous les équilibreurs de charge existants :

```
curl -X GET "$rias_endpoint/v1/load_balancers?version=2019-05-31&generation=1" -H "Authorization: $iam_token"
```
{: pre}

La section suivante fournit des détails sur les API que vous pouvez utiliser pour les équilibreurs de charge dans votre environnement VPC. Pour obtenir les spécifications complètes, voir [référence d'API VPC on Classic](https://{DomainName}/apidocs/vpc-on-classic#list-all-load-balancers).

| Description | API |
|-------------|-----|
| Crée et met à disposition un équilibreur de charge | POST /load_balancers |
| Extrait tous les équilibreurs de charge | GET /load_balancers |
| Extrait un équilibreur de charge | GET /load_balancers/{id} |
| Supprime un équilibreur de charge | DELETE /load_balancers/{id} |
| Met à jour un équilibreur de charge | PATCH /load_balancers/{id} |
| Crée un programme d'écoute | POST /load_balancers/{id}/listeners |
| Extrait tous les programmes d'écoute de l'équilibreur de charge | GET /load_balancers/{id}/listeners |
| Extrait un programme d'écoute | GET /load_balancers/{id}/listeners/{listener_id} |
| Supprime un programme d'écoute | DELETE /load_balancers/{id}/listeners/{listener_id} |
| Met à jour un programme d'écoute | PATCH /load_balancers/{id}/listeners/{listener_id} |
| Crée un pool | POST /load_balancers/{id}/pools |
| Extrait tous les pools de l'équilibreur de charge | GET /load_balancers/{id}/pools |
| Extrait un pool | GET /load_balancers/{id}/pools/{pool_id} |
| Supprime un pool | DELETE /load_balancers/{id}/pools/{pool_id} |
| Met à jour un pool | PATCH /load_balancers/{id}/pools/{pool_id} |
| Crée un membre | POST /load_balancers/{id}/pools/{pool_id}/members |
| Extrait tous les membres du pool | GET /load_balancers/{id}/pools/{pool_id}/members |
| Extrait un membre |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Supprime un membre du pool | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Crée un membre | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Met à jour des membres du pool | PUT /load_balancers/{id}/pools/{pool_id}/members |
| Extrait les statistiques d'un équilibreur de charge | GET /load_balancers/{id}/statistics |
| Extrait toutes les stratégies du programme d'écoute |  GET /load_balancers/{id}/listeners/{listener_id}/policies
| Crée une stratégie pour le programme d'écoute | POST /load_balancers/{id}/listeners/{listener_id}/policies
| Supprime une stratégie du programme d'écoute  | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Extrait une stratégie du programme d'écoute | GET /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Met à jour une stratégie du programme d'écoute | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Extrait toutes les règles associées à une stratégie | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| Crée une règle pour la stratégie | POST /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| Supprime une règle de la stratégie | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| Extrait une règle de la stratégie  | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| Met à jour une règle de la politique | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}

## Exemple d'équilibreur de charge
{: #load-balancer-example}

Dans l'exemple suivant, vous utilisez l'API pour créer un équilibreur de charge devant 2 instances de serveur VPC (`192.168.100.5` et `192.168.100.6`) exécutant une application Web à l'écoute sur le port `80`. L'équilibreur de charge dispose d'un programme d'écoute frontal qui permet d'accéder à l'application Web en toute sécurité au moyen de HTTPS. Vous pouvez ensuite utiliser l'API pour obtenir des détails sur l'instance de l'équilibreur de charge après sa création et pour supprimer l'instance de l'équilibreur de charge.

### Exemple d'étapes
{: #lbaas-example-steps}

Les exemples d'étapes décrits ci-dessous ne décrivent pas les étapes prérequises de l'utilisation de l'[interface utilisateur IBM Cloud](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console), de l'[interface CLI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli) ou de l'[API VPC on Classic](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis) pour mettre à disposition un VPC, des sous-réseau et des instances.

Les étapes de l'exemple d'équilibreur de charge peuvent également être exécutées à l'aide de l'[interface de ligne de commande](/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference).
{: note}

**Etape 1. Création d'un équilibreur de charge avec un programme d'écoute, un pool et des instances de serveur associées (membres du pool)**

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

Exemple de sortie :
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

Sauvegardez l'ID de l'équilibreur de charge pour l'utiliser dans les étapes suivantes, par exemple, dans la variable `lbid`.

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

**Etape 2. Obtention d'un équilibreur de charge**

```
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

Prévoyez du temps pour la mise à disposition. Une fois que l'équilibreur de charge est prêt, il prendra le statut `online` et `active`, comme dans l'exemple de sortie suivant :

Exemple de sortie :

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

**Etape 3. Suppression d'un équilibreur de charge**

```bash
curl -H "Authorization: $iam_token" -X DELETE "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

## Exemples de couche 7 : Création d'une stratégie et de règles
{: #layer-7-examples-create-policy-and-rules}

Les deux exemples suivants indiquent les étapes à suivre pour créer des stratégies et des règles et les associer à un programme d'écoute.

### Exemple 1 : Créez un programme d'écoute HTTPS avec des stratégies et des règles. Les stratégies sont créées avec l'action `Redirect`

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

### Exemple 2 : Créez des stratégies avec l'action `Forward` dans les pools et associez-les à un programme d'écoute existant
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

## FAQ (Foires aux questions)
{: #load-balancer-faqs}

Cette section contient des réponses aux questions fréquemment posées sur le service **Equilibreur de charge pour VPC**.

### Puis-je utiliser un nom DNS différent pour mon équilibreur de charge ?
{: #can-i-use-a-different-dns-name-for-my-load-balancer}

Le nom DNS affecté automatiquement à l'équilibreur de charge n'est pas personnalisable. Toutefois, vous avez la possibilité d'ajouter un enregistrement CNAME (nom canonique) qui fasse pointer le nom DNS de votre choix vers le nom DNS affecté automatiquement. Par exemple, votre équilibreur de charge dans `us-south` a l'ID `dd754295-e9e0-4c9d-bf6c-58fbc59e5727`, le nom DNS de l'équilibreur de charge auto-assigné est `dd754295-us-south.lb.appdomain.cloud` et vous souhaitez utiliser le nom DNS `www.myapp.com`. Vous pouvez ajouter un enregistrement CNAME (via le fournisseur DNS que vous utilisez pour gérer `myapp.com`) faisant pointer `www.myapp.com` sur le nom DNS de l'équilibreur de charge `dd754295-us-south.lb.appdomain.cloud.lb.appdomain.cloud`.

### Quel est le nombre maximal de programmes d'écoute frontaux que je peux définir avec mon équilibreur de charge ?
{: #what-s-the-maximum-number-of-front-end-listeners-i-can-define-with-my-load-balancer}

10.

### Quel est le nombre maximal d'instances de serveur que je peux connecter à mon pool de back end ?
{: #what-s-the-maximum-number-of-server-instances-i-can-attach-to-my-back-end-pool}

50.

### L'équilibreur de charge est-il évolutif horizontalement ?
{: #is-the-load-balancer-horizontally-scalable}

Oui. L'équilibreur de charge ajuste automatiquement sa capacité en fonction de la charge. En cas de mise à l'échelle horizontale, le nombre d'adresses IP associées au nom DNS de l'équilibreur de charge est modifié.

### Que dois-je faire si j'utilise des ACL ou des groupes de sécurité sur les sous-réseaux utilisés pour déployer l'équilibreur de charge ?
{: #what-should-i-do-if-i-am-using-acls-or-security-groups-on-the-subnets-that-are-used-to-deploy-the-load-balancer}

Vous devez vous assurer que les règles ACL ou de groupe de sécurité appropriées sont en place pour autoriser le trafic entrant pour les ports d'écoute configurés et les ports de gestion (ports de 56500 à 56520). Le trafic entre l'équilibreur de charge et les instances de back end doit également être autorisé.

### Pourquoi est-ce que je reçois un message d'erreur : `certificate instance not found` ?

* Le CRN (nom de ressource de cloud) de l'instance de certificat peut être non valide.
* Vous n'avez peut-être pas accordé l'**autorisation de service à service**. Voir la section **Déchargement SSL** du présent document.

### Pourquoi est-ce que je reçois un code `401 Unauthorized Error` ?

Vérifiez les règles d'accès suivantes pour votre utilisateur :
* La règle d'accès pour le type de ressource de l'équilibreur de charge
* La règle d'accès pour le groupe de ressources
* Si des programmes d'écoute `HTTPS` sont utilisés, vérifiez également l'autorisation de service à service pour l'instance Certificate Manager.

### Pourquoi mon équilibreur de charge est-il à l'état `maintenance_pending` ?

L'équilibreur de charge est à l'état `maintenance_pending` au cours de diverses activités de maintenance telles que :
* Les activités de mise à l'échelle horizontale
* Les activités de reprise
* La mise à jour en continu pour remédier aux vulnérabilités et appliquer des correctifs de sécurité

### Pourquoi dois-je choisir plusieurs sous-réseaux lors de la mise à disposition ?
{: #why-do-I-need-to-choose-multiple-subnets-during-provisioning}

L'équilibreur de charge **Load Balancer for VPC** est compatible avec les régions multi-zone. Les dispositifs d'équilibreur de charge sont déployés sur les sous-réseaux que vous avez sélectionnés. Il est vivement recommandé de choisir des sous-réseaux de différentes zones pour proposer une disponibilité et une redondance plus élevées.

### Pourquoi l'intégrité des membres de back end sous mon pool porte-t-elle la mention `inconnu` ?

* Le pool n'est associé à aucun programme d'écoute.
* Des modifications de configuration ont peut-être été apportées au pool ou à son programme d'écoute associé.

### Quelle est la version TLS compatible avec le déchargement SSL ?
{: #which-tls-version-is-supported-with-ssl-offload}

L'équilibreur de charge **Load Balancer for VPC** prend en charge TLS 1.2 avec terminaison SSL.

La liste ci-dessous répertorie les chiffrements pris en charge (par ordre de priorité) :
* ECDHE-RSA-AES256-GCM-SHA384
* ECDHE-RSA-AES256-SHA384
* AES256-GCM-SHA384
* AES256-SHA256
* ECDHE-RSA-AES128-GCM-SHA256
* ECDHE-RSA-AES128-SHA256
* AES128-GCM-SHA256
* AES128-SHA256

### Quels sont les paramètres par défaut et les valeurs autorisées pour les paramètres de diagnostic d'intégrité ?
{: #what-are-the-default-settings-and-allowed-values-for-health-check-parameters}

Les paramètres par défaut et les valeurs autorisées sont répertoriés ci-dessous :
* Intervalle de diagnostic d'intégrité : la valeur par défaut est 5 secondes ; la plage admise est comprise entre 2 et 60 secondes.
* Délai d'attente de la réponse du diagnostic d'intégrité : la valeur par défaut est 2 secondes ; la plage admise est comprise entre 1 et 59 secondes.
* Nombre maximal de nouvelles tentatives : la valeur par défaut est 2 ; la plage admise est comprise entre 1 et 10 nouvelles tentatives.

La valeur du délai d'attente de la réponse du diagnostic d'intégrité doit toujours être inférieure à la valeur de l'intervalle de diagnostic d'intégrité.
{:note}

### Les adresses IP d'équilibreur de charge sont-elles fixes ?
{: #are-the-load-balancer-ip-addresses-fixed}

Il n'est pas garanti que les adresses IP d'équilibreur de charge soient fixes, en raison de l'élasticité intégrée du service. Lors de la mise à l'échelle horizontale, vous verrez des modifications dans les adresses IP disponibles qui sont associées au nom de domaine complet de votre équilibreur de charge.

Utilisez le nom de domaine complet plutôt que les adresses IP mises en cache.
{:note}

### L'équilibreur de charge prend-il en charge le basculement vers la couche 7 ?
{: #does-the-load-balancer-support-layer-7-switching}

Oui.

### Pourquoi la création ou la mise à jour du programme d'écoute HTTPS m'indique-t-elle que mon certificat n'est pas valide ?
{: #why-does-https-listener-creation-or-update-tell-me-that-my-certificate-is-invalid}

Vérifiez les points suivants :

* Le CRN (nom de ressource de cloud) du certificat fourni peut être non valide.
* L'instance de certificat fournie dans Certificate Manager n'est peut-être pas associée à une clé privée.
