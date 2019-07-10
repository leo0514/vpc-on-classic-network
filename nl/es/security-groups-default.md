---

copyright:
  years: 2018, 2019

lastupdated: "2019-05-14"

keywords: default, security group, asynchronous, rules

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Actualización del grupo de seguridad predeterminado
{: #updating-the-default-security-group}


El grupo de seguridad predeterminado es muy similar a cualquier otro grupo de seguridad, con la excepción de que no se puede suprimir.

Cada VPC tiene un grupo de seguridad predeterminado con reglas que permiten:

* Tráfico de entrada de todos los miembros del grupo.
* Todo el tráfico de salida.

Si edita las reglas del grupo de seguridad predeterminado, dichas reglas editadas se aplican a todos los servidores actuales y futuros del grupo.

Las reglas de entrada para permitir el ping y el SSH no se añaden automáticamente al grupo de seguridad predeterminado. Puede modificar las reglas del grupo de seguridad predeterminado mediante la API REST, la `cli ibmcloud` o la interfaz de usuario.

## Ejemplo: Modificación de las reglas del grupo de seguridad predeterminado mediante la CLI
{: #example-modifying-the-default-security-group-rules-using-the-cli}

1. Inicie una sesión en IBM Cloud.

   Si tiene una cuenta federada:
   ```
   ibmcloud login -sso
   ```
   {: pre}

   De lo contrario, utilice este mandato:

   ```
   ibmcloud login
   ```
   {: pre}

2. Obtenga el ID del grupo de seguridad predeterminado y los detalles de la VPC

   Ejecute este mandato de CLI para obtener una lista de todas las VPC:

   ```
   ibmcloud is vpc
   ```
   {: pre}

   El nombre del grupo de seguridad predeterminado se muestra bajo la columna `Grupo de seguridad predeterminado`. Anote el nombre para encontrar el `ID` cuando examine la lista de grupos de seguridad (a continuación). 
   
   Obtenga una lista de todos los grupos de seguridad de la VPC:

   ```
   ibmcloud is security-groups
   ```
   {: pre}

   Guarde el ID del grupo de seguridad (para el grupo de seguridad predeterminado) en una variable para poderlo utilizar posteriormente. Por ejemplo, si utiliza el nombre de variable `sg`:

   ```
   sg=2d364f0a-a870-42c3-a554-000001162469
   ```
   {: pre}

   Para obtener detalles sobre el grupo de seguridad, ejecute el siguiente mandato de CLI:

   ```
   ibmcloud is security-group $sg
   ```
   {: pre}
   
   Como alternativa, podría insertar el valor de ID real en lugar de la variable `$sg`.

3. Actualice el grupo de seguridad predeterminado -- añada reglas que permitan SSH y PING

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
   ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
   ```
   {: codeblock}


La adición y eliminación de reglas de grupo de seguridad es una operación asíncrona. El cambio suele tardar entre 1 y 30 segundos en tener efecto.
{: note}
