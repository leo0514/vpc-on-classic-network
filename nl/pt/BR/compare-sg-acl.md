---
copyright:
  years: 2019
lastupdated: "2019-05-14"

keywords: security groups, traffic, firewall, stateful, filtering, access, control, list, ACL, stateless, traffic, resource

subcollection: vpc-on-classic-network
---

# Comparando grupos de segurança e listas de controle de acesso
{: #compare-security-groups-and-access-control-lists}

Os grupos de segurança e as listas de controle de acesso fornecem maneiras de controlar o
tráfego entre as sub-redes e as instâncias em seu {{site.data.keyword.cloud}} VPC, usando
regras que você especifica.

A tabela a seguir resume algumas diferenças chave entre grupos de segurança e ACLs:

|  | Grupos de Segurança | ACLs    |
|-------------|-----------------|---------|
| Nível de controle  | Instância da VSI    | Sub-rede  |
| Estado   | Stateful - Assim que uma conexão de entrada é permitida, é permitido responder | Stateless - As conexões de entrada e de saída devem ser explicitamente permitidas |
| Regras suportadas | Usa somente regras de permissão | Usa regras de permissão e negação|
| Como as regras são aplicadas | Todas as regras são consideradas | As regras são processadas na sequência |
| Relacionamento com o recurso associado | Uma instância pode ser associada a múltiplos grupos de segurança| Múltiplas sub-redes podem ser associadas à mesma ACL|
