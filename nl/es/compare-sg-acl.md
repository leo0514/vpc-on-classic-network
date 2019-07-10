---
copyright:
  years: 2019
lastupdated: "2019-05-14"

keywords: security groups, traffic, firewall, stateful, filtering, access, control, list, ACL, stateless, traffic, resource

subcollection: vpc-on-classic-network
---

# Comparación entre grupos de seguridad y listas de control de acceso
{: #compare-security-groups-and-access-control-lists}

Los grupos de seguridad y las listas de control de acceso ofrecen métodos para controlar el tráfico entre las subredes y las instancias de {{site.data.keyword.cloud}} VPC, mediante las reglas que especifique.

La tabla siguiente resume algunas diferencias claves entre los grupos de seguridad y las ACL:

|  | Grupos de seguridad | Listas ACL    |
|-------------|-----------------|---------|
| Nivel de control  | Instancia de VSI    | Subred  |
| Estado   | Con estado - Una vez que se permite una conexión de entrada, se le permite responder | Sin estado - Se deben permitir explícitamente tanto las conexiones de entrada como las de salida |
| Reglas soportadas | Solo utiliza reglas que permiten | Utiliza reglas que permiten y que deniegan|
| Cómo se aplican las reglas | Se tienen en cuenta todas las reglas | Las reglas se procesan en secuencia |
| Relación con el recurso asociado | Una instancia puede estar asociada a varios grupos de seguridad| Es posible asociar varias subredes a la misma ACL|
