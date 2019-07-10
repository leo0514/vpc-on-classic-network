---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: VPC, subnet, address prefixes, design, addressing

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Projetando um plano de endereçamento para um VPC 
{: #vpc-addressing-plan-design}

A primeira etapa no design de seu VPC deve ser projetar seu plano de endereçamento. Um plano de endereçamento adequadamente executado tem dois objetivos:

* Ele atende aos requisitos de comunicação de instâncias de VPC.
* Ele mantém a flexibilidade para o crescimento futuro. 

Este documento fornece um exemplo de design do plano de endereçamento para um aplicativo da web com três camadas, em que cada camada é suportada por múltiplas zonas.

Embora cada {{site.data.keyword.cloud}} VPC seja implementado em uma região específica, o VPC pode abranger todas as zonas dentro dessa região. O {{site.data.keyword.cloud_notm}} VPC define um prefixo de endereço padrão para cada zona. Esses prefixos de endereço permitem a comunicação entre as instâncias do {{site.data.keyword.cloud_notm}} VPC em diferentes zonas, fornecendo as informações de roteamento relacionadas à zona de que o [roteador implícito](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-glossary#implicit-router) precisa.

As mesmas etapas de design estão envolvidas, não importa se o aplicativo está completamente contido na nuvem ou se partes do aplicativo estão em execução em outro local.
{: tip}

## Suposições e considerações de design
{: #design-considerations-and-assumptions}

Ao projetar o plano de endereçamento para um aplicativo, a consideração principal é manter os blocos CIDR usados para criar sub-redes dentro de uma única zona o mais contíguos possível. Ao fazer isso, o designer permite que eles sejam resumidos em um único prefixo de endereço, deixando espaço para crescimento futuro.

Outra consideração é o número de endereços disponíveis que uma sub-rede pode precisar para escala horizontal. A tabela a seguir lista o número de endereços disponíveis em uma sub-rede com base em seu tamanho de bloco CIDR especificado:

| Tamanho do bloco CIDR | Endereços disponíveis |
| --------------- | ------------------- |
|      /22        |        1019         |
|      /23        |         507         |
|      /24        |         251         |
|      /25        |         123         |
|      /26        |          59         |
|      /27        |          27         |
|      /28        |          11         |

Com base nessas duas considerações, aqui estão as suposições que fazemos para este exemplo:

* Para este exemplo, os intervalos de CIDR do bloco 172.16.0.0/12 de endereços do RFC 1918 serão usados para todas as sub-redes.
* Presumimos que a camada de apresentação do aplicativo tenha uma aparência fina acima de uma API de REST. Portanto, a escala horizontal afeta a camada do meio mais do que afeta a camada de apresentação.

## Determine o tamanho da sub-rede da camada
{: #determine-each-tier-s-subnet-size}

A próxima etapa é determinar o tamanho da sub-rede de cada camada (em termos de endereços disponíveis). Cada camada do aplicativo tem uma presença em cada zona, portanto, cada zona precisará de três sub-redes.

Aqui estão as considerações que estamos usando ao planejar o tamanho de sub-rede de cada camada:

* A camada de banco de dados (o back-end) é a que tem menor probabilidade de precisar de ajuste de escala dinâmico, de modo que essas sub-redes podem ser as menores. Ou seja, essas sub-redes podem conter o menor número de endereços disponíveis. 
    * _Este exemplo usa um bloco CIDR `/27`, que permite 27 endereços nessa camada._
* A camada do meio é a que tem maior probabilidade de precisar de ajuste de escala dinâmico, portanto, essas sub-redes serão as maiores. Ou seja, elas devem conter o maior número de endereços disponíveis. 
    * _Este exemplo usa um bloco CIDR `/25`, que permite 123 endereços nessa camada._
* A camada de front-end se ajusta entre as outras camadas. Ela não precisará de muitos endereços como a camada do meio, mas precisará de mais do que a camada de banco de dados. 
    * _Este exemplo usa um bloco CIDR `/26`, que permite 59 endereços nessa camada._

## Combinando as sub-redes e selecionando os prefixos de endereço
{: #combining-the-subnets-and-selecting-the-address-prefixes}

Para selecionar um prefixo de endereço aceitável para cada zona, você precisará de um tamanho de sub-rede grande o suficiente para acomodar todas as três sub-redes em cada camada e ainda deixar espaço para a escala horizontal e a expansão futura. 

Um prefixo de endereço `/24` é o menor prefixo no qual essas três sub-redes podem ser combinadas (27 + 123 + 59). Como uma melhor prática, é recomendado selecionar o próximo tamanho de sub-rede maior, não o menor. A designação do próximo tamanho de sub-rede maior (`/23`) permite a escala horizontal além dos limites fornecidos anteriormente, pois permite incluir novas sub-redes em cada camada, de dentro do mesmo prefixo de endereço.

Agora que determinamos o tamanho de sub-rede correto, os prefixos de endereço reais podem ser designados, um para cada zona:

|  Zona  | Prefixo do endereço  |
| ------ | --------------- |
| Zona 1 | `172.16.0.0/23` |
| Zona 2 | `172.16.2.0/23` |
| Zona 3 | `172.16.4.0/23` |

E, por meio dessa base, as três sub-redes dentro de cada zona podem ser designadas:

|  Zona  |   Camada   |    CIDR de Sub-rede    |
| ------ | -------- | ----------------- |
| Zona 1 |  Meio  |  `172.16.0.0/25`  |
| Zona 1 |  Frente   |  `172.16.1.0/26`  |
| Zona 1 | Base de dados | `172.16.1.128/27` |
| Zona 2 |  Meio  |  `172.16.2.0/25`  |
| Zona 2 |  Frente   |  `172.16.3.0/26`  |
| Zona 2 | Base de dados | `172.16.3.128/27` |
| Zona 3 |  Meio  |  `172.16.4.0/25`  |
| Zona 3 |  Frente   |  `172.16.5.0/26`  |
| Zona 3 | Base de dados | `172.16.5.128/27` |

## Considerações para estender uma infraestrutura existente
{: #considerations-for-extending-an-existing-infrastructure}

Quando você está planejando um VPC que estende uma infraestrutura existente, é possível seguir as etapas anteriores, independentemente de essa infraestrutura ser sua infraestrutura no local, outro VPC ou mesmo outra nuvem. Tenha em mente que não se deve reutilizar intervalos de endereços existentes. Ao evitar a reutilização de endereço, é possível maximizar sua capacidade de tirar proveito dos recursos do {{site.data.keyword.cloud_notm}} VPC.
