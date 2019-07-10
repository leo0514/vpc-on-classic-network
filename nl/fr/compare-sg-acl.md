---
copyright:
  years: 2019
lastupdated: "2019-05-14"

keywords: security groups, traffic, firewall, stateful, filtering, access, control, list, ACL, stateless, traffic, resource

subcollection: vpc-on-classic-network
---

# Comparaison des groupes de sécurité et des listes de contrôle d'accès
{: #compare-security-groups-and-access-control-lists}

Les groupes de sécurité et les listes de contrôle d'accès permettent de contrôler le trafic sur les sous-réseaux et les instances de votre VPC {{site.data.keyword.cloud}}, en utilisant des règles que vous indiquez.

Le tableau suivant récapitule certaines des principales différences entre les groupes de sécurité et les ACL :

|  | Groupes de sécurité | ACL    |
|-------------|-----------------|---------|
| Niveau de contrôle  | Instance VSI    | Sous-réseau  |
| Etat   | Avec état - Une fois qu'une connexion entrante est autorisée, une réponse est possible | Sans état - Les connexions entrantes et sortantes doivent être explicitement autorisées  |
| Règles prises en charge | Utilise des règles d'autorisation uniquement | Utilise des règles d'autorisation et de refus|
| Mode d'application des règles | Toutes les règles sont prises en compte | Les règles sont traitées dans l'ordre |
| Relation avec la ressource associée | Une instance peut être associée à plusieurs groupes de sécurité| Plusieurs sous-réseaux peuvent être associés à la même ACL|
