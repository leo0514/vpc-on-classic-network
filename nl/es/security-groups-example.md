---

copyright:
  years: 2019

lastupdated: "2019-05-14"

keywords: security group, VSI, ping, TCP, outbound, SG4, rules, metadata, setting up

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

# Configuración de grupos de seguridad mediante la CLI
{: #setting-up-security-groups-using-the-cli}

En el ejemplo siguiente, creará una VSI con un grupo de seguridad habilitado, utilizando la interfaz de línea de mandatos (CLI). Este es el aspecto que tiene el caso de ejemplo.

![Grupos de seguridad de IBM VPC](images/security-groups-schematic.svg "Grupos de seguridad de IBM VPC")

Observe en la figura que la VSI denominada **SG4** tiene una IP flotante `169.60.208.144` asignada, además de la dirección de VPC interna `10.0.0.5`; por lo tanto, puede comunicarse con el Internet público. El grupo de seguridad asignado a la VSI **SG4** se denomina "demosg".

La VSI denominada **SG8** es únicamente interna en la VPC con una dirección IP privada. El grupo de seguridad asignado a la VSI **SG8** se denomina "my_vpc_sg". Ambas VSI existen dentro de la VPC denominada `sgvpc` y también en la misma subred `10.0.0.0/28` para poder comunicarse entre sí.

## Pasos para crear una VSI con un grupo de seguridad adjunto
{: #steps-for-creating-a-vsi-with-a-security-group-attached}

Las reglas de grupo de seguridad para "my_vpc_sg" incluirán funciones básicas de SSH, PING y TCP de salida.

Tenga en cuenta que primero debe crear el grupo de seguridad, con el mandato `ibmcloud is sgc` y, a continuación, debe crear la VSI que incluirá el grupo de seguridad recién creado.

Este código de ejemplo omite algunos pasos, por lo que aquí encontrará más información:

 * Las instrucciones para la creación de una VPC y una subred están disponibles en nuestro tema sobre [Creación de una VCP](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli).

Puede copiar y pegar mandatos del código de CLI de ejemplo para empezar a crear una VSI con un grupo de seguridad adjunto. Las respuestas del sistema no se muestran por completo en este código de ejemplo. Deberá actualizar los mandatos con los ID de recursos adecuados para la _VPC_, la _subred_, la _imagen_ y la _clave_ y el _número de ID del grupo de seguridad_ correcto.

1. Cree un grupo de seguridad llamado “my_vpc_sg”

   ```
   ibmcloud is security-group-create my_vpc_sg $vpc
   ```
   {: pre}

   Guarde el ID en una variable para que se pueda utilizar posteriormente, por ejemplo en la variable llamada `sg`:

   ```
   sg=2d364f0a-a870-42c3-a554-000000632953
   ```
   {: pre}

2. Añada reglas que permitan SSH, PING y TCP de salida

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
ibmcloud is security-group-rule-add $sg outbound tcp
   ```
   {: codeblock}

3. Cree una VSI con el grupo de seguridad recién creado

   ```
   ibmcloud is instance-create test-instance $vpc us-south-2 b-4x16 $subnet 1000 \
   --image $image --keys $key --security-groups $sg
   ```
   {: pre}

## Hoja de apuntes de la lista de mandatos
{: #command-list-cheat-sheet}

Para obtener una lista completa de los mandatos de CLI de VPC disponibles para los grupos de seguridad, escriba:

```
ibmcloud is help | grep sg
```
{: pre}

Para ver el grupo de seguridad y los metadatos, incluidas las reglas, escriba (para el ejemplo anterior):

```
ibmcloud is sg $sg
```
{: pre}

Si desea añadir una regla de grupo de seguridad, a continuación, se muestra un mandato de ejemplo para añadir una regla de entrada PING a un grupo de seguridad:

```
ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0

```
{: pre}
