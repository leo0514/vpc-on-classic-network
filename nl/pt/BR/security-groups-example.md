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

# Configurando grupos de segurança usando a CLI
{: #setting-up-security-groups-using-the-cli}

No exemplo a seguir, você criará uma VSI com um grupo de segurança ativado usando a interface da linha de comandos (CLI). Aqui está como o cenário se parece.

![Grupos de segurança para IBM VPC](images/security-groups-schematic.svg "Grupos de segurança para IBM VPC")

Observe na figura que a VSI denominada **SG4** tem um IP flutuante `169.60.208.144` designado a ela, além de seu endereço da VPC interno `10.0.0.5`; portanto, ela pode conversar com a Internet pública. O grupo de segurança designado à VSI **SG4** é denominado "demosg".

A VSI denominada **SG8** é interna apenas para a VPC, com um endereço IP privado. O grupo de segurança designado à VSI **SG8** é denominado "my_vpc_sg". Ambas as VSIs existem dentro da VPC chamada `sgvpc` e também na mesma sub-rede `10.0.0.0/28` para que possam se comunicar entre si.

## Etapas para criar uma VSI com um grupo de segurança anexado
{: #steps-for-creating-a-vsi-with-a-security-group-attached}

As regras do grupo de segurança para "my_vpc_sg" incluirão funções básicas de SSH, PING e TCP de saída.

Observe que se deve criar primeiro o grupo de segurança, com o comando `ibmcloud is sgc` e, em seguida, criar a VSI que incluirá esse grupo de segurança recém-criado.

Esse código de exemplo ignora algumas etapas, portanto, aqui é possível localizar mais informações:

 * As instruções para criar uma VPC e uma sub-rede estão disponíveis em nosso tópico [Criando uma VPC](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli).

É possível copiar e colar comandos desse código de CLI de exemplo para começar a criar uma VSI que tenha um grupo de segurança conectado. As respostas do sistema não são mostradas completamente nesse código de amostra. Será necessário atualizar seus comandos com os IDs de recursos corretos para sua _VPC_, sua _sub-rede_, sua _imagem_ e sua _chave_ e o _número do ID do grupo de segurança_ correto.

1. Crie um grupo de segurança chamado “my_vpc_sg”

   ```
   ibmcloud is security-group-create my_vpc_sg $vpc
   ```
   {: pre}

   Salve o ID em uma variável para que possamos usá-lo posteriormente, por exemplo, na variável denominada `sg`:

   ```
   sg=2d364f0a-a870-42c3-a554-000000632953
   ```
   {: pre}

2. Inclua regras que permitam SSH, PING e TCP de saída

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
ibmcloud is security-group-rule-add $sg outbound tcp
   ```
   {: codeblock}

3. Crie uma VSI com o grupo de segurança recém-criado

   ```
   ibmcloud is instance-create test-instance $vpc us-south-2 b-4x16 $subnet 1000 \ 
   --image $image --keys $key --security-groups $sg
   ```
   {: pre}

## Folha de dicas da lista de comandos
{: #command-list-cheat-sheet}

Para obter uma lista completa dos comandos da CLI da VPC disponíveis para grupos de segurança, digite:

```
ibmcloud is help | grep sg
```
{: pre}

Para ver seu grupo de segurança e seus metadados, incluindo as regras, digite (para o exemplo anterior):

```
ibmcloud is sg $sg
```
{: pre}

Para incluir uma regra de grupo de segurança, aqui está um comando de exemplo para incluir uma regra de entrada PING em um grupo de segurança:

```
ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0

```
{: pre}
