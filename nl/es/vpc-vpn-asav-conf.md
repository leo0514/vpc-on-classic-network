---

copyright:
  years: 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, Cisco, ASAv, connection, secure, remote

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc} 
{:new_window: target="_blank"} 
{:DomainName: data-hd-keyref="DomainName"} 
{:note: .note} 
{:important: .important} 
{:deprecated: .deprecated} 
{:generic: data-hd-programlang="generic"}

# Creación de una conexión segura con un igual de Cisco ASAv remoto
{: #creating-a-secure-connection-with-a-remote-cisco-asav-peer}

Este documento se basa en Cisco ASAv, Cisco Adaptative Security Appliance versión 9.10(1).

Los pasos de ejemplo que siguen omiten los requisitos previos de utilizar la API o CLI de {{site.data.keyword.cloud}} para crear nubes privadas virtuales (VPC). Para obtener más información, consulte [Guía de inicio](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) y [Configuración de VPC con API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Pasos de ejemplo
{: #cisco-example-steps}

La topología para conectarse con el igual de Cisco ASAv remoto es similar a la creación de una conexión de VPN entre dos VPC de {{site.data.keyword.cloud}}. Sin embargo, una de las partes se sustituye por la unidad Cisco ASAv.

![especificar descripción de imagen aquí](./images/vpc-vpn-asav-figure.png)

### Para crear una conexión segura con un igual de Cisco ASAv remoto
{: #to-create-a-secure-connection-with-the-remote-cisco-asav-peer}

El primer paso en la configuración de Cisco ASAv para utilizarlo con la VPN de la VPC de IBM es garantizar que las condiciones de requisitos previos siguientes se han establecido:

* Cisco ASAv está en línea y funciona con una licencia adecuada
* Se ha habilitado una contraseña para Cisco ASAv
* Hay al menos una interfaz interna funcional configurada y verificada
* Hay al menos una interfaz externa funcional configurada y verificada

Cuando una unidad Cisco ASAv recibe una solicitud de conexión de un igual de VPN remoto, utiliza los parámetros de fase 1 de IPsec para establecer una conexión segura y autenticar ese igual de VPN. A continuación, si la política de seguridad permite la conexión, la unidad Cisco ASAv establece el túnel utilizando los parámetros de fase 1 de IPsec y aplica la política de seguridad de IPsec. Los servicios de seguridad, autenticación y gestión de claves se negocian dinámicamente mediante el protocolo IKE.

**Para ofrecer soporte a estas funciones, la unidad Cisco ASAv debe realizar los pasos de configuración general siguientes:**

* Defina los parámetros de fase 1 que la unidad de Cisco ASAv necesita para autenticar el igual remoto y establecer una conexión segura.
* Defina los parámetros de fase 2 que la unidad Cisco ASAv necesita para crear un túnel VPN con el igual remoto.

Cree un objeto de propuesta de Internet Key Exchange (IKE) versión 2. Los objetos de propuesta de IKEv2 contienen los parámetros necesarios para crear propuestas IKEv2 al definir el acceso remoto y las políticas VPN de sitio a sitio. IKE es un protocolo de gestión de claves que facilita la gestión de las comunicaciones basadas en IPsec. Se utiliza para autenticar los iguales de IPsec, negociar y distribuir claves de cifrado IPSec y establecer automáticamente asociaciones de seguridad (SA) IPSec. 

```
group-policy GroupPolicy_161.156.80.10 internal
group-policy GroupPolicy_161.156.80.10 attributes
 vpn-tunnel-protocol ikev1 ikev2 
tunnel-group 161.156.80.10 type ipsec-l2l
tunnel-group 161.156.80.10 general-attributes
 default-group-policy GroupPolicy_161.156.80.10
tunnel-group 161.156.80.10 ipsec-attributes
 ikev1 pre-shared-key <key value>
 ikev2 remote-authentication pre-shared-key <key value>
 ikev2 local-authentication pre-shared-key <key value>
```

Cree una configuración de política IKEv2 para la conexión IPsec. El bloque de política de IKEv2 establece los parámetros para el intercambio de IKE. En este bloque, se establecen los parámetros siguientes:
* Algoritmo de cifrado: establecido en AES-256 en este ejemplo
* Algoritmo de integridad: establecido en SHA256 en este ejemplo
* Grupo Diffie-Hellman: IPsec utiliza el algoritmo Diffie-Hellman para generar la clave de cifrado inicial entre los iguales. En este ejemplo, se establece en el grupo 14
* Función pseudoaleatoria (PRF): IKEv2 requiere utilizar un método independiente como algoritmo para derivar el material de claves y las operaciones de hashing necesarios para el cifrado de túnel de IKEv2. Este método se conoce como función pseudoaleatoria y se establece en SHA.
* Tiempo de vida de las asociaciones de seguridad: establece el tiempo de vida de las asociaciones de seguridad (después de las cuales se producirá una reconexión). Se establece en 36.000 segundos.
* Tipo de operación: mantiene este valor como el valor predeterminado, bidireccional. (No es explícito en la visualización "mostrar en ejecución").

Tal y como se muestra en el ejemplo de código siguiente, esta política de muestra utiliza AES-256 para cifrar el canal seguro. El algoritmo de hash SHA512 se utiliza para validar la identidad del igual remoto y el grupo Diffie-Hellman 14 se utiliza para la generación de claves. El grupo 14 utiliza bloques de cifrado de 2048 bits. Por último, el tiempo de vida de la asociación de seguridad se establece en 36.000 segundos.

```
crypto ikev2 policy 100
encryption aes-256
integrity sha-1
group 14
prf sha
lifetime seconds 36000
```

* Defina la lista de acceso y la correlación criptográfica de la VPN:

```
access-list outside_cryptomap_1 extended permit ip object NETWORK_OBJ_192.168.236.0_24 object vpc 
crypto map outside_map 1 match address outside_cryptomap_1
crypto map outside_map 1 set peer 161.156.80.10 
crypto map outside_map 1 set ikev1 transform-set ESP-AES-128-SHA ESP-AES-128-MD5 ESP-AES-192-SHA ESP-AES-192-MD5 ESP-AES-256-SHA ESP-AES-256-MD5 ESP-3DES-SHA ESP-3DES-MD5 ESP-DES-SHA ESP-DES-MD5
crypto map outside_map 1 set ikev2 ipsec-proposal AES256 AES192 AES 3DES DES
crypto map outside_map interface outside
nat (any,outside) source static NETWORK_OBJ_192.168.236.0_24 NETWORK_OBJ_192.168.236.0_24 destination static vpc vpc no-proxy-arp route-lookup
```

## Para crear una conexión segura con la VPC de IBM Cloud local
{: #to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

Para crear una conexión segura, creará la conexión VPN dentro de la VPC, que es similar al ejemplo de 2 VPC.

* Cree una pasarela de VPN en la subred de VPC junto con una conexión VPN entre la VPC y la unidad Cisco ASAv, estableciendo `local_cidrs` en el valor de subred de la VPC y `peer_cidrs` en el valor de subred de Cisco ASAv.

El estado de la pasarela aparece como `pendiente` mientras se crea la pasarela VPN y el estado pasa a ser `disponible` una vez completada la creación. La creación puede tardar un poco. 
{:note}


![especificar descripción de imagen aquí](./images/vpc-vpn-asav-connection.png)

### Comprobar el estado de la conexión segura
{: #cisco-check-the-status-of-the-secure-connection}

Puede comprobar el estado de la conexión a través de la consola de IBM Cloud. También puede intentar hacer `ping` de sitio a sitio utilizando los VSI.

![especificar descripción de imagen aquí](./images/vpc-vpn-asav-status.png)
