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

# Usando o Load Balancers for VPC
{: #--using-load-balancers-in-ibm-cloud-vpc}

O serviço {{site.data.keyword.cloud}} **Load Balancer for VPC** distribui o tráfego entre várias instâncias do servidor dentro da mesma região de seu VPC.

## Balanceador de carga público
{: #public-load-balancer}

A instância de serviço do balanceador de carga é designada a um nome completo de domínio (FQDN) acessível publicamente, que deve ser usado para acesso aos seus aplicativos hospedados por trás do
IBM Cloud Load Balancer for VPC. Esse nome de domínio pode ser registrado com um ou mais endereços IP públicos.

Ao longo do tempo, esses endereços IP públicos e o número de endereços IP públicos podem mudar, devido a atividades de manutenção e de ajuste de escala. As instâncias do servidor de back-end (VSIs) que hospedam seu aplicativo devem ser executadas na mesma região e sob a mesma VPC.

## Load Balancer privado
{: #private-load-balancer}

O balanceador de carga privado é acessível somente para clientes internos em suas sub-redes privadas, dentro da mesma região e VPC. O balanceador de carga privado aceita o tráfego somente nos espaços de endereço do [RFC1918![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")](https://tools.ietf.org/html/rfc1918){: new_window}.

Semelhante a um balanceador de carga público, sua instância de serviço do balanceador
de carga privado é designada a um nome completo do domínio (FQDN). Entretanto, esse nome de domínio é registrado com um ou mais endereços IP privados.

As operações do IBM Cloud podem mudar o número e o valor de seus endereços IP privados
designados ao longo do tempo, com base em atividades de manutenção e ajuste de escala. As instâncias do servidor de back-end (VSIs) que hospedam seu aplicativo devem ser executadas na mesma região e sob a mesma VPC.

Veja a tabela a seguir para obter uma comparação resumida dos recursos:

| Recurso | Load Balancer público | Load Balancer privado |
|--------|-------|-------|
| Acessível na Internet? |  Sim, com FQDN, Internet | Não, somente clientes internos, na mesma região e VPC |
| Aceita todo o tráfego? | Sim | Não, somente RFC 1918 |
| Número de IPs privados | mudanças ao longo do tempo | mudanças ao longo do tempo |
| Como o nome de domínio está registrado? | endereços IP públicos | endereços IP privados |

## Listeners de front-end e conjuntos de back-end
{: #front-end-listeners-and-back-end-pools}

É possível definir até dez (10) listeners de front-end (portas de aplicativos) e mapeá-los
para os respectivos conjuntos de back-end nos servidores de aplicativos de back-end. O nome completo do domínio (FQDN) designado para a instância de serviço do balanceador de carga e as portas do listener de front-end são expostos para a Internet pública. As solicitações recebidas de usuário são recebidas nessas portas.

Os protocolos suportados do listener de front-end são:
* HTTP (Protocolo de Transporte de Hipertexto)
* HTTPS
* TCP

Os protocolos suportados do conjunto de back-end são:
* HTTP (Protocolo de Transporte de Hipertexto)
* TCP

O tráfego HTTPS de entrada é finalizado no balanceador de carga, para permitir a comunicação HTTP de texto simples com o servidor de back-end.

Você pode conectar até cinquenta (50) instâncias de servidores a um conjunto de back-end. Cada instância do servidor conectada deve ter uma porta configurada. A porta pode ou não ser a mesma que a porta do listener de front-end.

O intervalo de portas de 56500 a 56520 é reservado para propósitos de gerenciamento; essas portas não podem ser usadas como portas do listener de front-end.
{: note}

## Balanceamento de carga da Camada 7
{: #layer-7-load-balancing}

Os balanceadores de carga públicos e privados suportam o balanceamento de carga da Camada 7. O tráfego de dados é distribuído com base em políticas e regras configuradas. Uma _política_ define a ação (ou seja, como o tráfego é distribuído) quando a solicitação recebida corresponde às regras associadas à política.

### Política de Camada 7
{: #layer-7-policy}

Uma política da Camada 7 é associada a um listener e apenas um listener HTTP ou HTTPS. Cada política pode ter um conjunto de regras. A política é aplicada **somente** quando todas as suas regras são correspondidas.

É possível anexar mais de uma política a um listener. Em geral, uma política com a prioridade mais baixa é avaliada primeiro. A prioridade deve ser exclusiva para uma determinada política.

Se a solicitação recebida não corresponder a nenhuma das regras de política, a solicitação será redirecionada para o conjunto padrão do listener, se o conjunto padrão estiver configurado.

Essas são as ações suportadas para uma política Camada 7:

* **Rejeitar:** a solicitação é negada com uma resposta 403.
* **Redirecionamento:** a solicitação é redirecionada para uma URL configurada e um código de resposta.
* **Encaminhamento:** a solicitação é enviada para um conjunto de back-end específico.

As políticas configuradas como `reject` são avaliadas primeiro, independentemente de sua prioridade.

Depois disso, as políticas configuradas como `redirect` são avaliadas.

Finalmente, as políticas configuradas como `forward` são avaliadas.

Dentro de cada categoria de ação, as políticas são avaliadas em ordem crescente de prioridade (mais baixa para a mais alta).

### Propriedades de política da Camada 7
{: #layer-7-policy-properties}

Property  | Descrição
------------- | -------------
Nome | O nome da política. O nome deve ser exclusivo dentro do listener.
Ações | A ação a ser tomada quando todas as regras de política são correspondidas. Os valores aceitáveis são `reject`, `redirect` e `forward`. Uma política com a ação `reject` é sempre avaliada primeiro, independentemente de sua prioridade. As políticas com ações `redirect` são avaliadas em seguida, seguidas por políticas com a ação `redirect`.
Prioridade | As políticas são avaliadas com base na ordem crescente de prioridade.
Url | A URL para a qual a solicitação será redirecionada se a ação estiver configurada como `redirect`.
Código de Status HTTP | Código de status da resposta retornado pelo balanceador de carga quando a ação está configurada como `redirect`. Os valores aceitáveis são: 301, 302, 303, 307 ou 308.
Destino | O conjunto de back-end de instâncias do servidor para as quais a solicitação será encaminhada se a ação estiver configurada como `forward`.

### Regras de Camada 7
{: #layer-7-rules}

Uma regra Camada 7 define como uma solicitação deve ser correspondida. Três tipos são suportados:

Tipo      |  Descrição
----------| -----------------------
`hostname` | A solicitação corresponde ao nome do host especificado (por exemplo, `api.my_company.com`).
`header`   | A solicitação corresponde a um campo de cabeçalho de HTTP (por exemplo, `Cookie`).
`path`     | A solicitação corresponde ao caminho na URL (por exemplo, `/index.html`).

Para corresponder a uma solicitação, `condition` deve ser definido em uma regra. Três condições são suportadas:

Condição |  Tipo de avaliação
----------------|---------------------
`contains`        |  Verifique se o campo extraído contém a sequência fornecida.
`equals`          |  Verifique se o campo extraído é idêntico à cadeia fornecida.
`matches_regex`   |  Corresponder o campo extraído à expressão regular fornecida.

## Propriedades de regra da Camada 7
{: #layer-7-rule-properties}

Property  | Descrição
------------- | -------------
Tipo | Especifica o tipo de regra. Os valores aceitáveis são `hostname`, `header` ou `path`.
Condição | Especifica a condição com a qual uma regra é avaliada. A condição pode ser: `contains`, `equals` ou `matches_regex`.
Campo | Especifica o nome do campo de cabeçalho de HTTP. Esse campo é aplicável apenas ao tipo de regra `header`. Por exemplo, para corresponder a um cookie no cabeçalho de HTTP, o campo pode ser configurado para `Cookie`.
Valor |  O valor a ser correspondido.

## Métodos de balanceamento de carga
{: #load-balancing-methods}

Os três métodos de balanceamento de carga a seguir estão disponíveis para distribuir tráfego entre servidores de aplicativos backend:

* **Round-robin:** round-robin é o método de balanceamento de carga padrão. Com esse método, o balanceador de carga encaminha conexões do cliente recebidas no modo round-robin para os servidores de back-end. Como resultado, todos os servidores de back-end recebem praticamente um número igual de conexões do cliente.

* **Round-robin ponderado:** com esse método, o balanceador de carga encaminha conexões do cliente recebidas para os servidores de
back-end em proporção à ponderação designada a esses servidores. A cada servidor é designado um peso padrão de 50, que pode ser customizado para qualquer valor entre 0 e 100.

Como um exemplo, se três servidores de aplicativos A, B e C tiverem ponderações customizadas para 60, 60 e 30 respectivamente, os servidores A e B receberão um número igual de conexões, enquanto o servidor C receberá metade desse número de conexões.

* **Conexões mínimas:** com esse método, a instância de servidor que atende ao número mínimo de conexões em um determinado momento recebe a próxima conexão do cliente.

**Características adicionais desses métodos:**

* A reconfiguração de um peso do servidor para '0' significa que nenhuma nova conexão é
encaminhada para esse servidor, mas qualquer tráfego existente continuará a fluir. O uso de um peso de '0' pode ajudar a desligar um servidor corretamente e removê-lo da rotação de serviço.
* Os valores de ponderação do servidor são aplicáveis somente com o método round-robin ponderado. Eles são ignorados com métodos de balanceamento de carga de round-robin e de conexões mínimas.

## Escala horizontal
{: #horizontal-scaling}

O balanceador de carga ajusta sua capacidade automaticamente de acordo com a carga. Quando esse ajuste ocorrer, você poderá ver uma mudança no número de endereços IP associados ao nome DNS do balanceador de carga.

## Verificações de funcionamento
{: #health-checks}

As definições de verificação de funcionamento são obrigatórias para conjuntos de back-end.

O balanceador de carga conduz verificações periódicas de funcionamento para monitorar o funcionamento das portas de back-end e encaminha o tráfego do cliente para elas de acordo. Se uma determinada porta do servidor de back-end for detectada como inoperante, nenhuma nova conexão será encaminhada para ela. O balanceador de carga continuará a monitorar o funcionamento de portas inoperantes e retomará seu uso se elas se tornarem funcionais novamente, o que significa passar com sucesso em duas tentativas de verificação de funcionamento consecutivas.

As verificações de funcionamento para as portas HTTP e TCP são conduzidas da seguinte forma:

* **HTTP:** uma solicitação de `HTTP GET` em uma URL pré-especificada é enviada para a porta do servidor de backend. A porta do servidor é marcada como funcional ao receber uma resposta `200 OK`. O caminho de funcionamento `GET` padrão
é "/" por meio da IU e pode ser customizado.

* **TCP:** o Load Balancer tenta abrir uma conexão TCP com o servidor de back-end em uma porta TCP especificada. A porta do servidor será marcada como funcional se a tentativa de conexão for bem-sucedida e a conexão será encerrada.

O intervalo de verificação de funcionamento padrão é 5 segundos, o tempo limite padrão com relação a uma solicitação de verificação de funcionamento é 2 segundos e o número padrão de novas tentativas é 2.
{: note}

## Transferência de SSL e autorizações necessárias
{: #ssl-offloading-and-required-authorizations}

Para todas as conexões HTTPS de entrada, o serviço do balanceador de carga finaliza a conexão SSL e estabelece uma comunicação HTTP de texto sem formatação com a instância do servidor de back-end. Com essa técnica, os handshakes SSL de uso intensivo da CPU e as tarefas de criptografia ou decriptografia são deslocados para longe das instâncias do servidor de back-end, permitindo que eles usem todos os seus ciclos de CPU para processar o tráfego de aplicativos.

Um certificado SSL é necessário para que o balanceador de carga execute tarefas de transferência de SSL. Você pode gerenciar os certificados SSL por meio do [IBM Certificate Manager ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window}.

Para conceder a um balanceador de carga acesso ao seu certificado SSL, deve-se ativar a **autorização de serviço para serviço**, que concede à instância de serviço do balanceador de carga acesso à instância do gerenciador de certificados. É possível gerenciar essa autorização ao seguir esta documentação [Concedendo acesso entre os serviços ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")](/docs/iam?topic=iam-serviceauth#create-auth){: new_window}. Certifique-se de escolher **VPC Infrastructure** como o serviço de origem, **Load Balancer for VPC** como o tipo de recurso, **Gerenciador de certificados** como o serviço de destino e designar a função de acesso ao serviço **Escritor**.

Se a autorização necessária for removida, erros poderão ocorrer para o balanceador de carga.
{: note}

## Identity and Access Management (IAM)
{: #identity-and-access-management-iam}

É possível configurar políticas de acesso para uma instância do **Load Balancer for VPC**. Para gerenciar suas políticas de acesso de usuário, visite [Gerenciando o acesso aos recursos ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")](/docs/services/iam?topic=iam-iammanidaccser#resourceaccess){: new_window} para obter mais informações sobre o Identity and Access Management.

### Configurando as políticas de acesso do grupo de recursos para usuários
{: #configuring-resource-group-access-policies-for-users}

Para criar um balanceador de carga, você precisará de acesso a um grupo de recursos. O usuário que está criando um balanceador de carga deve ter acesso apropriado ao grupo de recursos fornecido ou para o grupo de recursos padrão se nenhum for fornecido.

1. Navegue para  ** Gerenciar > Conta > Usuários **. Você verá uma lista dos usuários com acesso à sua conta do IBM Cloud.
2. Selecione o nome do usuário para quem você deseja designar uma política de acesso. Se o usuário não for mostrado, clique em **Convidar usuários** para incluir o usuário em sua conta do IBM Cloud.
3. Selecione **Designar acesso**.
4. Selecione **Designar acesso em um grupo de recursos**.
5. Na lista suspensa **Grupo de recursos**, selecione o grupo de recursos desejado.
6. Na lista suspensa **Designar acesso a um grupo de recursos**, selecione o acesso desejado.
7. Na lista suspensa **Serviços**, selecione os serviços desejados.
9. Selecione **Designar** para designar a política de acesso ao grupo de recursos para o usuário.

### Configurando políticas de acesso de recurso para usuários
{: #configuring-resource-access-policies-for-users}

| Função de acesso à plataforma | Ação do balanceador de carga |
|-------------|-----|
| Administrator | Criar/visualizar/editar/excluir balanceador de carga |
| Editor | Criar/visualizar/editar/excluir balanceador de carga |
| Visualizador | Visualizar balanceador de carga |

1. Navegue para  ** Gerenciar > Conta > Usuários **. Você verá a lista de usuários com acesso à sua conta do IBM Cloud.
2. Selecione o nome do usuário para quem você deseja designar uma política de acesso. Se o usuário não for mostrado, clique em **Convidar usuários** para incluir o usuário em sua conta do IBM Cloud.
3. Selecione **Designar acesso**.
4. Selecione **Designar acesso a recursos**.
5. Na lista suspensa **Serviços**, selecione **VPC Infrastructure**.
6. Na lista suspensa **Tipo de recurso**, selecione **Load Balancer for VPC**.
7. Na lista suspensa **ID do balanceador de carga**, selecione um ID da instância do balanceador de carga ou use o valor padrão, **Todos os balanceadores de carga**.
8. Designe uma função de acesso à plataforma para o usuário.
9. Selecione **Designar** para designar a política de acesso para o usuário.

## Integração do rastreador de atividade
{: #activity-tracker-integration}

O serviço de balanceador de carga é integrado ao **IBM Cloud Activity Tracker with LogDNA**, que registra eventos, de maneira compatível com o padrão CADF, conforme acionado por atividades iniciadas pelo usuário que mudam o estado de serviço na nuvem.

Para obter uma lista detalhada de ações que são registradas como eventos de auditoria nas instâncias de serviço do balanceador de carga, consulte [Rastreador de atividade com eventos LogDNA](/docs/vpc-on-classic?topic=vpc-on-classic-at-events#events-load-balancers).

Todos os eventos de auditoria são registrados no "IBM Cloud Activity Tracker with LogDNA" na região `us-south`. Não importa em qual região o serviço de balanceador de carga é provisionado.
{:note}

Para visualizar eventos, deve-se provisionar uma instância "IBM Cloud Activity Tracker with LogDNA" na região `us-south` sob sua conta. Os usuários em sua conta devem ter uma política do IAM que conceda a função de acesso à plataforma **Visualizador** e a função de acesso ao serviço **Leitor** na instância "IBM Cloud Activity Tracker with LogDNA".

Mais informações sobre como conceder acesso estão disponíveis em [Concedendo
permissões para ver eventos da conta. ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")](https://cloud.ibm.com/docs/services/cloud-activity-tracker/how-to/grant_permissions.html#grant_permissions){: new_window}

Os usuários da conta do IBM Cloud podem monitorar as operações de nível de conta executadas no serviço de balanceador de carga.
{: tip}

Siga estas etapas para provisionar uma instância do "IBM Cloud Activity Tracker with LogDNA" em sua conta:

1. Efetue login no console do IBM Cloud. [Efetue login no console do IBM Cloud. ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")](https://cloud.ibm.com/){: new_window}
2. Clique no ![Ícone de menu](../../icons/icon_hamburger.svg) na parte superior esquerda. De lá, selecione **Observabilidade > Rastreador de atividade**.
3. Na parte superior direita, clique em **Criar instância**.
4. Defina um nome de serviço.
5. Selecione `us-south` como a região e escolha o grupo de recursos
6. Escolha um plano diferente de `lite` se você tiver uma conta paga.
7. Clique em **Criar**.

Aqui está a mensagem do rastreador de atividades de amostra para uma operação **Criar listener**:
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

## APIs disponíveis
{: #lbaas-apis-available}

Para fazer as chamadas de API, deve-se usar algum formulário de cliente REST. Por exemplo, é possível usar o comando `curl` para recuperar todos os balanceadores de carga existentes:

```
curl -X GET "$rias_endpoint/v1/load_balancers?version=2019-05-31&generation=1" -H "Authorization: $iam_token"
```
{: pre}

A seção a seguir fornece detalhes sobre as APIs que podem ser usadas para balanceadores de carga em seu ambiente de VPC. Para obter a especificação completa, consulte a [Referência da API VPC on Classic](https://{DomainName}/apidocs/vpc-on-classic#list-all-load-balancers).

| Descrição | API |
|-------------|-----|
| Cria e fornece um balanceador de carga | POST /load_balancers |
| Recupera todos os balanceadores de carga | GET /load_balancers |
| Recupera um balanceador de carga | GET /load_balancers/{id} |
| Exclui um balanceador de carga | DELETE /load_balancers/{id} |
| Atualiza um balanceador de carga | PATCH /load_balancers/{id} |
| Cria um listener | POST /load_balancers/{id}/listeners |
| Recupera todos os listeners do balanceador de carga | GET /load_balancers/{id}/listeners |
| Recupera um listener | GET /load_balancers/{id}/listeners/{listener_id} |
| Exclui um listener | DELETE /load_balancers/{id}/listeners/{listener_id} |
| Atualiza um listener | PATCH /load_balancers/{id}/listeners/{listener_id} |
| Cria um conjunto | POST /load_balancers/{id}/pools |
| Recupera todos os conjuntos do balanceador de carga | GET /load_balancers/{id}/pools |
| Recupera um conjunto | GET /load_balancers/{id}/pools/{pool_id} |
| Exclui um conjunto | DELETE /load_balancers/{id}/pools/{pool_id} |
| Atualiza um conjunto | PATCH /load_balancers/{id}/pools/{pool_id} |
| Cria um membro | POST /load_balancers/{id}/pools/{pool_id}/members |
| Recupera todos os membros do conjunto | GET /load_balancers/{id}/pools/{pool_id}/members |
| Recupera um membro |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Exclui um membro do conjunto | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Atualiza um membro | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Atualiza os membros do conjunto | PUT /load_balancers/{id}/pools/{pool_id}/members |
| Recupera estatísticas de um balanceador de carga | GET /load_balancers/{id}/statistics |
| Recupera todas as políticas do listener |  GET /load_balancers/{id}/listeners/{listener_id}/policies
| Cria uma política para o listener | POST /load_balancers/{id}/listeners/{listener_id}/policies
| Exclui uma política do listener | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Recupera uma política do listener | GET /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Atualiza uma política do listener | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Recupera todas as regras associadas a uma política | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| Cria uma regra para a política | POST /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| Exclui uma regra da política | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| Recupera uma regra da política | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| Atualiza uma regra da política | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}

## Exemplo de balanceador de carga
{: #load-balancer-example}

No exemplo a seguir, você usará a API para criar um balanceador de carga na frente de duas instâncias do servidor da VPC (`192.168.100.5` e `192.168.100.6`) que executam um aplicativo da web atendendo na porta `80`. O balanceador de carga tem um listener de front-end, que permite o acesso ao aplicativo da web com segurança por meio de HTTPS. Em seguida, é possível usar a API para obter detalhes da instância do balanceador de carga após sua criação e para excluir a instância do balanceador de carga.

### Etapas de exemplo
{: #lbaas-example-steps}

As etapas de exemplo a seguir ignoram as etapas de pré-requisito do uso da [IU do IBM Cloud](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console), da [CLI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli) ou da [API VPC on Classic](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis) para provisionar um VPC, as sub-redes e as instâncias.

As etapas de exemplo do balanceador de carga também podem ser executadas usando a [CLI](/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference).
{: note}

**Etapa 1. Criar um balanceador de carga com as instâncias do listener, do conjunto e de instâncias do servidor conectado (membros do conjunto)**

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
                        "port": 80, "target": {
                            "address": "192.168.100.5"
                        },
                        "weight": 50
                    },
                    {
                        "port": 80, "target": {
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

Saída de amostra:
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
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004", "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004" }
    ],
    "name": "example-balancer",
    "operating_status": "offline",
    "pools": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004", "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004", "name": "example-pool" }
    ],
    "provisioning_status": "create_pending",
    "resource_group": {
        "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
    }, "subnets": [ {
            "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e", "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e", "name": "example-subnet" }
    ]
}
```
{: screen}

Salve o ID do balanceador de carga a ser usado nas próximas etapas, por exemplo, salve-o na variável `lbid`.

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

**Etapa 2. Obter um balanceador de carga**

```
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

Permita algum tempo para fornecimento. Quando o balanceador de carga estiver pronto, ele será configurado como `online` e o status `active`, conforme você verá na saída de amostra a seguir:

Saída de amostra:

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
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004", "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004" }
  ],
  "operating_status": "online",
  "pools": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004", "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004", "name": "example-pool" }
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
  }, "subnets": [ {
      "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e", "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e", "name": "example-subnet" }
  ]
}
```
{: screen}

**Etapa 3. Excluir um balanceador de carga**

```bash
curl -H "Authorization: $iam_token" -X DELETE "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

## Exemplos de Camada 7: criar política e regras
{: #layer-7-examples-create-policy-and-rules}

Os dois exemplos a seguir fornecem etapas que mostram como as políticas e as regras são criadas e associadas a um listener.

### Exemplo 1: crie um listener HTTPS com políticas e regras. As políticas são criadas com a ação `Redirect`

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

### Exemplo 2: criar políticas com ação `Forward` para conjuntos e associá-la a um listener existente
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

## FAQs
{: #load-balancer-faqs}

Esta seção contém respostas para algumas perguntas mais frequentes sobre o serviço **Load Balancer for VPC**.

### Posso usar um nome DNS diferente para meu balanceador de carga?
{: #can-i-use-a-different-dns-name-for-my-load-balancer}

O nome DNS designado automaticamente para o balanceador de carga não é customizável. No entanto, é possível incluir um registro CNAME (Nome Canônico) que aponte seu nome DNS preferencial para o nome DNS do balanceador de carga designado automaticamente. Por exemplo, o balanceador de carga em `us-south` tem o ID `dd754295-e9e0-4c9d-bf6c-58fbc59e5727`, o nome DNS do balanceador de carga designado automaticamente é `dd754295-us-south.lb.appdomain.cloud`. Seu nome DNS preferencial é `www.myapp.com`. É possível incluir um registro CNAME (por meio do provedor DNS que você usa para gerenciar `myapp.com`) apontando `www.myapp.com` para o nome DNS do balanceador de carga `dd754295-us-south.lb.appdomain.cloud`.

### Qual é o número máximo de listeners de front-end que posso definir com meu balanceador de carga?
{: #what-s-the-maximum-number-of-front-end-listeners-i-can-define-with-my-load-balancer}

10.

### Qual é o número máximo de instâncias do servidor que eu posso conectar ao meu conjunto de back-end?
{: #what-s-the-maximum-number-of-server-instances-i-can-attach-to-my-back-end-pool}

50.

### O balanceador de carga é escalável horizontalmente?
{: #is-the-load-balancer-horizontally-scalable}

Sim. O balanceador de carga ajusta automaticamente sua capacidade com base no carregamento. Quando o ajuste de escala horizontal ocorre, o número de endereços IP associados ao nome DNS do balanceador de carga mudará.

### O que deverei fazer se estiver usando ACLs ou grupos de segurança nas sub-redes que são usadas para implementar o balanceador de carga?
{: #what-should-i-do-if-i-am-using-acls-or-security-groups-on-the-subnets-that-are-used-to-deploy-the-load-balancer}

Será necessário assegurar que as regras apropriadas da ACL ou do grupo de segurança estejam estabelecidas para permitir o tráfego de entrada para as portas do listener e as portas de gerenciamento configuradas (portas de 56500 a 56520). O tráfego entre o balanceador de carga e as instâncias de back-end também deve ser permitido.

### Por que estou recebendo uma mensagem de erro: `certificate instance not found`?

* O CRN da instância de certificado pode ser inválido.
* Você pode não ter concedido a **autorização de serviço a serviço**. Veja a seção **Transferência de SSL** deste documento.

### Por que estou recebendo um código `401 Unauthorized Error`?

Verifique as políticas de acesso a seguir para seu usuário:
* a política de acesso para o tipo de recurso do balanceador de carga;
* a política de acesso para o grupo de recursos;
* Se os listeners `HTTPS` forem usados, verifique também a autorização de serviço a serviço para a instância do Certificate Manager.

### Por que meu balanceador de carga está no estado `maintenance_pending`?

O balanceador de carga estará no estado `maintenance_pending` durante várias atividades de manutenção, tais como:
* Atividades de escala horizontal
* Atividades de recuperação
* Upgrade contínuo para tratar vulnerabilidades e aplicar correções de segurança

### Por que eu preciso escolher múltiplas sub-redes durante o fornecimento?
{: #why-do-I-need-to-choose-multiple-subnets-during-provisioning}

O **Load Balancer for VPC** está pronto para Multi-Zone-Region (MZR). Os dispositivos do balanceador de carga são implementados nas sub-redes selecionadas. É altamente recomendável escolher sub-redes de diferentes zonas, para fornecer a você maior disponibilidade e redundância.

### Por que o funcionamento do membro de back-end está sob meu conjunto `unknown`?

* O conjunto não está associado a nenhum listener
* Mudanças na configuração podem ter sido feitas no conjunto ou em seu listener associado

### Qual versão do TLS é suportada com transferência de SSL?
{: #which-tls-version-is-supported-with-ssl-offload}

O **Load Balancer for VPC** suporta TLS 1.2 com rescisão de SSL.

A lista a seguir detalha as cifras suportadas (listadas em ordem de precedência):
* ECDHE-RSA-AES256-GCM-SHA384
* ECDHE-RSA-AES256-SHA384
* AES256-GCM-SHA384
* AES256-SHA256
* ECDHE-RSA-AES128-GCM-SHA256
* ECDHE-RSA-AES128-SHA256
* AES128-GCM-SHA256
* AES128-SHA256

### Quais são as configurações padrão e os valores permitidos para os parâmetros de
verificação de funcionamento?
{: #what-are-the-default-settings-and-allowed-values-for-health-check-parameters}

As configurações padrão e os valores permitidos são listados abaixo:
* Intervalo de verificação de funcionamento: o padrão é cinco segundos, o intervalo é de dois
a 60 segundos.
* Tempo limite de resposta de verificação de funcionamento: o padrão é dois segundos, o intervalo é de um a 59 segundos.
* Máximo de novas tentativas: o padrão é duas novas tentativas e o intervalo é de um a 10 novas tentativas.

O valor de tempo limite de resposta de verificação de funcionamento sempre deve sempre ser menor
que o valor de intervalo de verificação de funcionamento.
{:note}

### Os endereços IP do balanceador de carga são corrigidos?
{: #are-the-load-balancer-ip-addresses-fixed}

Não há garantia de que os endereços IP do balanceador de carga serão corrigidos, devido à
elasticidade integrada do serviço. Durante o ajuste de escala horizontal, você verá as mudanças nos
IPs disponíveis associados ao FQDN do seu balanceador de carga.

Use FQDN, em vez de endereços IP em cache.
{:note}

### O balanceador de carga suporta alternância da Camada 7?
{: #does-the-load-balancer-support-layer-7-switching}

Sim.

### Por que a criação ou atualização do listener HTTPS informa que meu certificado é inválido?
{: #why-does-https-listener-creation-or-update-tell-me-that-my-certificate-is-invalid}

Verifique estas possibilidades:

* O CRN do certificado fornecido pode ser inválido.
* A instância de certificado fornecida no Certificate Manager pode não ter uma chave privada associada.
