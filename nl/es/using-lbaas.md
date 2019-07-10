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

# Utilización de equilibradores de carga en VPC
{: #--using-load-balancers-in-ibm-cloud-vpc}

El servicio **equilibrador de carga para VPC** de {{site.data.keyword.cloud}} distribuye el tráfico entre varias instancias de servidor de la misma región de la VPC.

## Equilibrador de carga público
{: #public-load-balancer}

A la instancia de servicio del equilibrador de carga se le ha asignado un nombre de dominio completo y accesible públicamente (FQDN), que debe utilizar para acceder a las aplicaciones alojadas detrás del servicio de equilibrador de carga de IBM para VPC. Es posible registrar este nombre de dominio con una o varias direcciones IP públicas.

Con el tiempo, dichas direcciones IP públicas y el número de direcciones IP públicas pueden cambiar debido a las actividades de mantenimiento y escalado. Las instancias de servicio de fondo (VSI) que alojan la aplicación deben ejecutarse en la misma región y en el mismo VPC.

## Equilibrador de carga privado
{: #private-load-balancer}

Solo pueden acceder al equilibrador de carga privado los clientes internos de las subredes privadas, dentro de la misma región y VPC. El equilibrador de carga privado solo acepta tráfico procedente de los espacios de direcciones [RFC1918 ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://tools.ietf.org/html/rfc1918){: new_window}.

De forma similar a lo que sucede con un equilibrador de carga público, se asigna a la instancia del servicio equilibrador de carga privado un nombre de dominio completo (FQDN). Sin embargo, este nombre de dominio se registra con una o varias direcciones IP privadas.

Con el tiempo, las operaciones de IBM Cloud pueden cambiar el número y el valor de las direcciones IP privadas asignadas, en función de las actividades de mantenimiento y escalado. Las instancias de servicio de fondo (VSI) que alojan la aplicación deben ejecutarse en la misma región y en el mismo VPC.

Consulte la tabla siguiente para ver un resumen de la comparación entre características:

| Característica | Equilibrador de carga público | Equilibrador de carga privado |
|--------|-------|-------|
| ¿Accesible en internet? |  Sí, con FQDN, Internet | No, solo clientes internos, en la misma región y VPC |
| ¿Acepta todo el tráfico? | Sí | No, solo RFC 1918 |
| Número de IP privadas | cambia con el tiempo | cambia con el tiempo |
| ¿Cómo se registra el nombre de dominio? | direcciones IP públicas | direcciones IP privadas |

## Escuchas frontales y agrupaciones de fondo
{: #front-end-listeners-and-back-end-pools}

Puede definir hasta diez (10) escuchas frontales (puertos de aplicación) y correlacionarlas con las respectivas agrupaciones de fondo en los servidores de aplicaciones de fondo. El nombre de dominio completo (FQDN) asignado a la instancia de servicio del equilibrador de carga y a los puertos de escucha frontales están expuestos a Internet público. Las solicitudes de usuario entrantes se reciben en dichos puertos.

Los protocolos de escucha frontales soportados son:
* HTTP
* HTTPS
* TCP

Los protocolos de agrupación de fondo soportados son:
* HTTP
* TCP

El tráfico HTTPS entrante finaliza en el equilibrador de carga, para permitir la comunicación HTTP en texto sin formato con el servidor de fondo.

Puede adjuntar hasta cincuenta (50) instancias de servidor a una agrupación de fondo. Cada instancia de servidor adjunta debe tener un puerto configurado. El puerto puede ser o no el mismo que el puerto de escucha frontal.

El rango de puertos de 56500 a 56520 está reservado para fines de gestión; estos puertos no se pueden utilizar como puertos de escucha frontales.
{: note}

## Equilibrio de carga de Capa 7
{: #layer-7-load-balancing}

Tanto los equilibradores de carga públicos como los privados admiten el equilibrio de carga de Capa 7. El tráfico de datos se distribuye en función de las políticas y reglas configuradas. Una _política_ define la acción (es decir, cómo se distribuye el tráfico) cuando la solicitud de entrada coincide con las reglas asociadas con la política.

### Política de Capa 7
{: #layer-7-policy}

Se asocia una política de Capa 7 a un escucha y sólo un escucha HTTP o HTTPS. Cada política puede tener un conjunto de reglas. La política **sólo** se aplica cuando todas sus reglas coinciden.

Puede conectar más de una política a un escucha. En general, se evalúa primero una política con una prioridad más baja. La prioridad debe ser exclusiva para una política determinada.

Si la solicitud de entrada no coincide con ninguna de las reglas de la política, la solicitud se redirige a la agrupación predeterminada del escucha, si se ha configurado la agrupación predeterminada.

Éstas son las acciones soportadas para una política de Capa 7:

* **Reject:** Se deniega la solicitud con una respuesta 403.
* **Redirect:** Se redirige la solicitud a un URL y un código de respuesta configurado.
* **Forward:** La solicitud se envía a una agrupación de fondo específica.

Las políticas establecidas como `reject` se evalúan en primer lugar, independientemente de su prioridad.

Después, se evalúan las políticas establecidas como `redirect`.

Finalmente, se evalúan las políticas establecidas como `forward`.

Dentro de cada categoría de acción, las políticas se evalúan en orden ascendente de prioridad (de la más baja a la más alta).

### Propiedades de política de Capa 7
{: #layer-7-policy-properties}

Propiedad  | Descripción
------------- | -------------
Nombre | El nombre de la política. El nombre debe ser exclusivo dentro del escucha.
Acción | La acción que se debe llevar a cabo cuando todas las reglas de política coinciden. Los valores aceptables son `reject`, `redirect` y `forward`. Una política con la acción `reject` siempre se evalúa en primer lugar, independientemente de su prioridad. Después se evalúan las políticas con acciones `redirect`, seguidas de las políticas con la acción `redirect`.
Prioridad | Las políticas se evalúan por orden ascendente de prioridad.
URL | El URL al que se redirige la solicitud, si la acción se ha establecido como `redirect`.
Código de estado HTTP | Código de estado de la respuesta devuelta por el equilibrador de carga cuando la acción se establece como `redirect`. Los valores aceptables son: 301, 302, 303, 307 o 308.
Destino | La agrupación de programas de fondo de las instancias de servidor a las que se reenvía la solicitud, si la acción se establece como `forward`.

### Reglas de Capa 7
{: #layer-7-rules}

Una regla de Capa 7 define cómo se debe emparejar una solicitud. Se admiten tres tipos:

Tipo      |  Descripción
----------| -----------------------
`hostname` | La solicitud coincide con el nombre de host especificado (por ejemplo, `api.my_company.com`).
`header`    | La solicitud coincide con un campo de cabecera HTTP (por ejemplo, `Cookie`).
`path`      | La solicitud coincide con la vía de acceso en el URL (por ejemplo, `/index.html`).

Para emparejar una solicitud, en la regla se debe haber definido una condición (`condition`). Se admiten tres condiciones:

Condición |  Tipo de evaluación
----------------|---------------------
`contains`        |  Verificar si el campo extraído contiene la serie proporcionada.
`equals`        |  Verificar si el campo extraído es idéntico a la serie proporcionada.
`matches_regex`           |  Emparejar el campo extraído con la expresión regular suministrada.

## Propiedades de regla de Capa 7
{: #layer-7-rule-properties}

Propiedad  | Descripción
------------- | -------------
Tipo | Especifica el tipo de la regla. Los valores aceptables son `hostname`, `header` o `path`.
Condición | Especifica la condición con la que se evalúa una regla. La condición puede ser: `contains`, `equals` o `matches_regex`.
Campo | Especifica el nombre del campo de cabecera HTTP. Este campo sólo es aplicable al tipo de regla `header`. Por ejemplo, para emparejar una cookie de la cabecera HTTP, el campo se puede establecer en `Cookie`.
Valor |  El valor que se debe emparejar.

## Métodos de equilibrio de carga
{: #load-balancing-methods}

Los siguientes tres métodos de equilibrio de carga están disponibles para distribuir el tráfico entre los servidores de aplicaciones back-end:

* **Round-robin:** round-robin es el método de equilibrio de carga predeterminado. Con este método, el equilibrador de carga reenvía las conexiones entrantes del cliente de forma rotativa a los servidores back-end. En consecuencia, todos los servidores back-end reciben aproximadamente el mismo número de conexiones de cliente.

* **Round-robin ponderado:** con este método, el equilibrador de carga reenvía las conexiones entrantes del cliente a los servidores back-end en proporción a la ponderación asignada a estos servidores. Cada servidor tiene asignada una ponderación predeterminada de 50, valor que se puede personalizar del 0 al 100.

A modo de ejemplo, si tres servidores de aplicaciones A, B y C tienen ponderaciones personalizadas de 60, 60 y 30 respectivamente, los servidores A y B reciben un número igual de conexiones, mientras que el servidor C recibe la mitad de conexiones.

* **Menos conexiones** con este método, la instancia de servidor que sirve el menor número de conexiones en un momento dado recibe la siguiente conexión de cliente.

**Características adicionales de estos métodos:**

* Si vuelve a establecer la ponderación en '0', no se reenviarán nuevas conexiones a dicho servidor, pero el tráfico existente continuará fluyendo. La utilización de una ponderación de '0' puede ayudar a desactivar un servidor y eliminarlo de la rotación de servicio.
* Los valores de ponderación de servidor solo se aplican cuando se utiliza el método 'round-robin ponderado'. Se pasan por alto con los métodos de equilibrio de carga round-robin y menos conexiones.

## Escalada horizontal
{: #horizontal-scaling}

El equilibrador de carga ajusta su capacidad automáticamente en función de la carga. Cuando se produce este ajuste, puede cambiar el número de direcciones IP asociadas al nombre DNS del equilibrador de carga.

## Comprobaciones de estado
{: #health-checks}

Las definiciones de comprobación de estado son obligatorias para las agrupaciones de fondo.

El equilibrador de carga lleva a cabo comprobaciones de estado periódicas para supervisar el estado de los puertos de fondo y reenvía el tráfico de cliente en consecuencia. Si un puerto de servidor de fondo tiene mal estado, no se le reenviarán nuevas conexiones. El equilibrador de carga continúa supervisando el estado de los puertos con mal estado y reanuda su uso cuando vuelven a tener un buen estado, lo que significa que pasan correctamente dos intentos de comprobación de estado consecutivos.

Las comprobaciones de estado de los puertos HTTP y TCP se llevan a cabo de la forma siguiente:

* **HTTP:** se envía una solicitud `HTTP GET` con un URL especificado previamente al puerto del servidor back-end. Se marca que el puerto del servidor se encuentra en buen estado tras haber recibido una respuesta `200 OK`. La vía de acceso del estado de `GET` predeterminada es "/" mediante la IU, y se puede personalizar.

* **TCP:** el equilibrador de carga intenta abrir una conexión con el servidor back-end en un puerto TCP especificado. Se marca que el puerto del servidor se encuentra en buen estado si la conexión finaliza correctamente y la conexión se cierra.

El intervalo de comprobación de estado predeterminado es 5 segundos, el tiempo de espera excedido predeterminado de una solicitud de comprobación de estado es 2 segundos y el número predeterminado de reintentos es 2.
{: note}

## Descarga de SSL y autorizaciones necesarias
{: #ssl-offloading-and-required-authorizations}

Para todas las conexiones HTTPS de entrada, el servicio de equilibrador de carga finaliza la conexión SSL y establece una comunicación HTTP en texto sin formato con la instancia de servidor de fondo. Con esta técnica, el reconocimiento SSL intensivo de CPU y las tareas de cifrado o descifrado se desplazan de las instancias de servidor de fondo, permitiéndoles utilizar todos los ciclos de CPU para procesar el tráfico de aplicaciones.

Se necesita un certificado SSL para que el equilibrador de carga pueda realizar tareas de descarga de SSL. Puede gestionar los certificados SSL a través de [IBM Certificate Manager ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window}.

Para dar acceso a un equilibrador de carga al certificado SSL, debe habilitar la **autorización de servicio a servicio**, que otorga a la instancia del servicio del equilibrador de carga acceso a su instancia del gestor de certificados. Para gestionar este tipo de autorización, consulte la documentación sobre [Cómo otorgar acceso entre servicios ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](/docs/iam?topic=iam-serviceauth#create-auth){: new_window}. Asegúrese de elegir **Infraestructura de VPC** como servicio de origen, **Equilibrador de carga para VPC** como tipo de recurso y **Gestor de certificados** como servicio de destino y de asignar el rol de acceso al servicio **Escritor**.

Si se elimina la autorización necesaria, es posible que se produzcan errores en el equilibrador de carga.
{: note}

## Identity and Access Management (IAM)
{: #identity-and-access-management-iam}

Puede configurar políticas de acceso para la instancia del **Equilibrador de carga para VPC**. Para gestionar las políticas de acceso de usuario, visite [Gestión del acceso a recursos ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](/docs/services/iam?topic=iam-iammanidaccser#resourceaccess){: new_window} para obtener más información sobre la característica Identity and Access Management.

### Configuración de políticas de acceso de grupo de recursos para los usuarios
{: #configuring-resource-group-access-policies-for-users}

Para crear un equilibrador de carga necesitará acceso a un grupo de recursos. El usuario que está creando un equilibrador de carga debe tener el acceso adecuado al grupo de recursos proporcionado o al grupo de recursos predeterminado si no se proporciona ninguno.

1. Vaya a **Gestionar > Cuenta > Usuarios**. Verá una lista de los usuarios con acceso a su cuenta de IBM Cloud.
2. Seleccione el nombre del usuario al que desea asignar una política de acceso. Si no se muestra el usuario, pulse **Invitar a usuarios** para añadir el usuario a la cuenta de IBM Cloud.
3. Seleccione **Asignar acceso**.
4. Seleccione **Asignar acceso dentro de un grupo de recursos**.
5. En la lista desplegable **Grupo de recursos**, seleccione el grupo de recursos deseado.
6. En la lista desplegable **Asignar acceso a un grupo de recursos**, seleccione el acceso deseado.
7. En la lista desplegable **Servicios**, seleccione los servicios deseados.
9. Seleccione **Asignar** para asignar la política de acceso del grupo de recursos al usuario.

### Configuración de políticas de acceso de recursos para los usuarios
{: #configuring-resource-access-policies-for-users}

| Rol de acceso a la plataforma | Acción de equilibrador de carga |
|-------------|-----|
| Administrador | Crear/Visualizar/Editar/Suprimir equilibrador de carga |
| Editor | Crear/Visualizar/Editar/Suprimir equilibrador de carga |
| Visor | Ver equilibrador de carga |

1. Vaya a **Gestionar > Cuenta > Usuarios**. Verá una lista de los usuarios con acceso a su cuenta de IBM Cloud.
2. Seleccione el nombre del usuario al que desea asignar una política de acceso. Si no se muestra el usuario, pulse **Invitar a usuarios** para añadir el usuario a la cuenta de IBM Cloud.
3. Seleccione **Asignar acceso**.
4. Seleccione **Asignar acceso a recursos**.
5. En la lista desplegable **Servicios**, seleccione **Infraestructura de VPC**.
6. En la lista desplegable **Tipo de recurso**, seleccione **Equilibrador de carga para VPC**.
7. En la lista desplegable **ID de equilibrador de carga**, seleccione un ID de instancia de equilibrador de carga o utilice el valor predeterminado **Todos los equilibradores de carga**.
8. Asigne un rol de acceso a la plataforma al usuario.
9. Seleccione **Asignar** para asignar la política de acceso al usuario.

## Integración de Activity Tracker
{: #activity-tracker-integration}

El servicio de equilibrador de carga se integra con **IBM Cloud Activity Tracker with LogDNA**, que registra sucesos, conforme al estándar CADF, como desencadenados por actividades iniciadas por el usuario que cambian el estado del servicio en la nube.

Para obtener una lista detallada de las acciones que se registran como sucesos de auditoría en las instancias de servicio del equilibrador de carga, consulte [sucesos de Activity Tracker with LogDNA](/docs/vpc-on-classic?topic=vpc-on-classic-at-events#events-load-balancers).

Todos los sucesos de auditoría se registran en "IBM Cloud Activity Tracker with LogDNA" en la región `us-south`. No importa en qué región se suministre el servicio equilibrador de carga.
{:note}

Para ver los sucesos, debe suministrar una instancia de "IBM Cloud Activity Tracker with LogDNA" en la región `us-south` bajo su cuenta. Los usuarios de la cuenta deben tener una política IAM que otorgue el rol de acceso a la plataforma de **Visor** y el rol de acceso al servicio de **Lector** para la instancia de "IBM Cloud Activity Tracker with LogDNA".

Encontrará más información sobre cómo otorgar acceso en el apartado sobre [Cómo otorgar permisos para ver sucesos de cuenta. ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://cloud.ibm.com/docs/services/cloud-activity-tracker/how-to/grant_permissions.html#grant_permissions){: new_window}

Los usuarios de la cuenta de IBM Cloud pueden supervisar las operaciones a nivel de cuenta realizadas en el servicio equilibrador de carga.
{: tip}

Siga estos pasos para suministrar una instancia de "IBM Cloud Activity Tracker with LogDNA" bajo su cuenta:

1. Inicie sesión en la consola de IBM Cloud. [Iniciar sesión en la consola de IBM Cloud. ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://cloud.ibm.com/){: new_window}
2. Pulse en el ![Icono de menú](../../icons/icon_hamburger.svg) en la parte superior izquierda. Desde ahí, seleccione **Observabilidad > Activity Tracker**.
3. En la parte superior derecha, pulse **Crear instancia**.
4. Defina un nombre de servicio.
5. Seleccione la región `us-south` y elija el grupo de recursos
6. Elija un plan distinto de `lite` si tiene una cuenta de pago.
7. Pulse **Crear**.

A continuación se muestra un mensaje de ejemplo de seguimiento de actividad correspondiente a la operación **Crear escucha**:
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

Para realizar llamadas de API debe utilizar algún tipo de cliente REST. Por ejemplo, puede utilizar el mandato `curl` para recuperar todos los equilibradores de carga existentes:

```
curl -X GET "$rias_endpoint/v1/load_balancers?version=2019-05-31&generation=1" -H "Authorization: $iam_token"
```
{: pre}

En la sección siguiente se proporcionan detalles sobre las API que puede utilizar para el equilibrador de carga en el entorno VPC. Para ver la especificación completa, revise la [Consulta de API de VPC on Classic](https://{DomainName}/apidocs/vpc-on-classic#list-all-load-balancers).

| Descripción | API |
|-------------|-----|
| Crea y suministra un equilibrador de carga | POST /load_balancers |
| Recupera todos los equilibradores de carga | GET /load_balancers |
| Recupera un equilibrador de carga | GET /load_balancers/{id} |
| Suprime un equilibrador de carga | DELETE /load_balancers/{id} |
| Actualiza un equilibrador de carga | PATCH /load_balancers/{id} |
| Crea una escucha | POST /load_balancers/{id}/listeners |
| Recupera todas las escuchas del equilibrador de carga | GET /load_balancers/{id}/listeners |
| Recupera una escucha | GET /load_balancers/{id}/listeners/{listener_id} |
| Suprime una escucha | DELETE /load_balancers/{id}/listeners/{listener_id} |
| Actualiza una escucha | PATCH /load_balancers/{id}/listeners/{listener_id} |
| Crea una agrupación | POST /load_balancers/{id}/pools |
| Recupera todas las agrupaciones del equilibrador de carga | GET /load_balancers/{id}/pools |
| Recupera una agrupación | GET /load_balancers/{id}/pools/{pool_id} |
| Suprime una agrupación | DELETE /load_balancers/{id}/pools/{pool_id} |
| Actualiza una agrupación | PATCH /load_balancers/{id}/pools/{pool_id} |
| Crea un miembro | POST /load_balancers/{id}/pools/{pool_id}/members |
| Recupera todos los miembros de la agrupación | GET /load_balancers/{id}/pools/{pool_id}/members |
| Recupera un miembro |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Suprime un miembro de la agrupación | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Actualiza un miembro | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Actualiza miembros de la agrupación | PUT /load_balancers/{id}/pools/{pool_id}/members |
| Recupera estadísticas de un equilibrador de carga | GET /load_balancers/{id}/statistics |
| Recupera todas las políticas del escucha |  GET /load_balancers/{id}/listeners/{listener_id}/policies
| Crea una política para el escucha | POST /load_balancers/{id}/listeners/{listener_id}/policies
| Suprime una política del escucha | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Recupera una política del escucha | GET /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Actualiza una política del escucha | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Recupera todas las reglas asociadas a una política | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| Crea una regla para la política | POST /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| Suprime una regla de la política | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| Recupera una regla de la política | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| Actualiza una regla de la política | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}

## Ejemplo de equilibrador de carga
{: #load-balancer-example}

En el ejemplo siguiente, utilizará la API para crear un equilibrado de carga delante de 2 instancias de servidor de VPC (`192.168.100.5` y `192.168.100.6`) que ejecutan una aplicación web que está a la escucha en el puerto `80`. El equilibrador de carga tiene una escucha frontal que permite el acceso a la aplicación web de forma segura mediante HTTPS. A continuación, puede utilizar la API para obtener detalles de la instancia del equilibrador de carga una vez creada y para suprimir la misma instancia.

### Pasos de ejemplo
{: #lbaas-example-steps}

En los pasos siguientes del ejemplo se omiten los pasos de requisito previo de utilizar la [IU de IBM Cloud](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console), la [CLI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli) o la [VPC en la API Clásica](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis) para suministrar una VPC, subredes e instancias.

Los pasos de ejemplo del equilibrador de carga también se pueden ejecutar mediante la [CLI](/docs/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference).
{: note}

**Paso 1. Crear un equilibrador de carga con escucha, agrupación e instancias de servidor conectadas (miembros de agrupación)**

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

Salida de ejemplo:
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

Guarde el ID del equilibrador de carga para utilizarlo en los pasos siguientes, por ejemplo, guárdelo en la variable `lbid`.

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

**Paso 2. Obtener un equilibrador de carga**

```
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

Deje pasar un tiempo para que se complete el suministro. Cuando el equilibrador de carga esté listo, se establecerá en el estado `en línea` y `activo`, tal como verá en la siguiente salida de ejemplo:

Salida de ejemplo:

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

**Paso 3. Suprimir un equilibrador de carga**

```bash
curl -H "Authorization: $iam_token" -X DELETE "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

## Ejemplos de Capa 7: Crear política y reglas
{: #layer-7-examples-create-policy-and-rules}

Los pasos de los dos ejemplos siguientes que muestran cómo se crean las políticas y las reglas y cómo se asocian con un escucha.

### Ejemplo 1: Crear un escucha HTTPS con políticas y reglas. Las políticas se crean con la acción `Redirect`

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

### Ejemplo 2: Crear políticas con la acción `Forward` a las agrupaciones y asociarlas con un escucha existente
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

## Preguntas más frecuentes
{: #load-balancer-faqs}

Esta sección contiene respuestas a algunas de las preguntas más frecuentes sobre el servicio **Equilibrador de carga para VPC**.

### ¿Puedo utilizar un nombre de DNS distinto para mi equilibrador de carga?
{: #can-i-use-a-different-dns-name-for-my-load-balancer}

El nombre de DNS asignado automáticamente para el equilibrador de carga no se puede personalizar. Sin embargo, puede añadir un registro CNAME (nombre canónico) que indique su nombre de DNS preferido en el nombre de DNS del equilibrador de carga asignado automáticamente. Por ejemplo, el equilibrador de carga de `us-south` tiene el ID `dd754295-e9e0-4c9d-bf6c-58fbc59e5727`, el nombre DNS del equilibrador de carga asignado automáticamente es `dd754295-us-south.lb.appdomain.cloud`. Su nombre DNS preferido es `www.myapp.com`. Puede añadir un registro de CNAME (mediante el proveedor de DNS que utiliza para gestionar `myapp.com`) que indique `www.myapp.com` en el nombre de DNS del equilibrador de carga `dd754295-us-south.lb.appdomain.cloud`.

### ¿Cuál es el número máximo de escuchas frontales que puedo definir con mi equilibrador de carga?
{: #what-s-the-maximum-number-of-front-end-listeners-i-can-define-with-my-load-balancer}

10.

### ¿Cuál es el número máximo de instancias de servidor que puedo conectar a mi agrupación de fondo?
{: #what-s-the-maximum-number-of-server-instances-i-can-attach-to-my-back-end-pool}

50.

### ¿Es el equilibrador de carga escalable horizontalmente?
{: #is-the-load-balancer-horizontally-scalable}

Sí. El equilibrador de carga ajusta automáticamente su capacidad en función de la carga. Cuando se realiza un escalado horizontal, cambia el número de direcciones IP asociadas al nombre DNS del equilibrador de carga.

### ¿Qué debo hacer si estoy utilizando las ACL o los grupos de seguridad en las subredes que se utilizan para desplegar el equilibrador de carga?
{: #what-should-i-do-if-i-am-using-acls-or-security-groups-on-the-subnets-that-are-used-to-deploy-the-load-balancer}

Deberá asegurarse de que las reglas de grupo de seguridad o de ACL adecuadas estén en vigor para permitir el tráfico de entrada a puertos de escucha y de gestión configurados (los puertos del 56500 al 56520). También se debe permitir el tráfico entre el equilibrador de carga y las instancias de fondo.

### ¿Por qué recibo el mensaje de error: `no se encuentra la instancia de certificado`?

* Es posible que el CRN de la instancia de certificado no sea válido.
* Es posible que no se le haya otorgado **autorización de servicio a servicio**. Consulte la sección **Descarga de SSL** de este documento.

### ¿Por qué recibo el código `401 Unauthorized Error`?

Compruebe las políticas de acceso siguientes de su usuario:
* La política de acceso del tipo de recurso del equilibrador de carga
* La política de acceso para el grupo de recursos
* Si se utilizan escuchas `HTTPS`, compruebe también la autorización de servicio a servicio de la instancia del gestor de certificados.

### ¿Por qué mi balanceador de carga está en el estado `maintenance_pending`?

El equilibrador de carga tendrá el estado `maintenance_pending` durante diversas actividades de mantenimiento como las siguientes:
* Actividades de escalado horizontal
* Actividades de recuperación
* Actualización continua para solucionar vulnerabilidades y aplicar parches de seguridad

### ¿Por qué tengo que elegir varias subredes durante el suministro?
{: #why-do-I-need-to-choose-multiple-subnets-during-provisioning}

El **equilibrador de carga para VPC** está preparado para multizona y región (MZR). Los dispositivos del equilibrador de carga se despliegan en las subredes que ha seleccionado. Se recomienda elegir subredes de distintas zonas para conseguir una mayor disponibilidad y redundancia.

### ¿Por qué el estado del miembro de fondo bajo mi agrupación es `desconocido`?

* La agrupación no está asociada a ningún escucha
* Es posible que se hayan realizado cambios en la configuración de la agrupación o del escucha asociado

### ¿Qué versión de TLS recibe soporte con la descarga SSL?
{: #which-tls-version-is-supported-with-ssl-offload}

El **equilibrador de carga para VPC** da soporte a TLS 1.2 con terminación SSL.

En la siguiente lista se muestran los cifrados que reciben soporte (en orden de precedencia):
* ECDHE-RSA-AES256-GCM-SHA384
* ECDHE-RSA-AES256-SHA384
* AES256-GCM-SHA384
* AES256-SHA256
* ECDHE-RSA-AES128-GCM-SHA256
* ECDHE-RSA-AES128-SHA256
* AES128-GCM-SHA256
* AES128-SHA256

### ¿Cuáles son los valores predeterminados y los permitidos de los parámetros de comprobación de estado?
{: #what-are-the-default-settings-and-allowed-values-for-health-check-parameters}

Los valores predeterminados y los permitidos se enumeran a continuación:
* Intervalo de comprobación de estado: el valor predeterminado es 5 segundos, puede estar comprendido entre 2 y 60 segundos.
* Tiempo de espera excedido de respuesta de comprobación de estado: el valor predeterminado es 2 segundos, puede estar comprendido entre 1 y 59 segundos.
* Número máximo de reintentos: el valor predeterminado es 2 reintentos, puede estar comprendido entre 1 y 10 reintentos.

El valor de tiempo de espera excedido de respuesta de comprobación de estado siempre debe ser menor que el valor de intervalo de comprobación de estado.
{:note}

### ¿Son fijas las direcciones IP de equilibrador de carga?
{: #are-the-load-balancer-ip-addresses-fixed}

No se garantiza que las direcciones IP del equilibrador de carga sean fijas debido a la elasticidad incorporada del servicio. Durante un escalado horizontal, verá cambios en las direcciones IP asociadas con el FQDN del equilibrador de carga.

Utilice FQDN en lugar de direcciones IP colocadas en memoria caché.
{:note}

### ¿Da soporte el equilibrador de carga a la conmutación de capa 7?
{: #does-the-load-balancer-support-layer-7-switching}

Sí.

### ¿Por qué la función de creación o de actualización de escucha HTTPS me dice que mi certificado no es válido?
{: #why-does-https-listener-creation-or-update-tell-me-that-my-certificate-is-invalid}

Compruebe estas posibilidades:

* Es posible que el CRN de certificado no sea válido.
* Es posible que la instancia de certificado especificado en el gestor de certificados no tenga una clave privada asociada.
