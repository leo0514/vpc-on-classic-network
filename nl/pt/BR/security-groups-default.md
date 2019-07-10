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

# Atualizando o grupo de segurança padrão
{: #updating-the-default-security-group}


O grupo de segurança padrão é muito semelhante a qualquer outro grupo de segurança, com a exceção de que ele não pode ser excluído.

Cada VPC tem um grupo de segurança padrão, com regras para permitir:

* Tráfego de entrada de todos os membros do grupo.
* Todo o tráfego de saída.

Se você editar as regras do grupo de segurança padrão, essas regras editadas serão aplicadas a todos os servidores atuais e futuros no grupo.

As regras de entrada para permitir ping e SSH não são incluídas automaticamente no grupo de segurança padrão. É possível modificar as regras do grupo de segurança padrão usando a API de REST, o `ibmcloud cli` ou a IU.

## Exemplo: modificando as regras do grupo de segurança padrão usando a CLI
{: #example-modifying-the-default-security-group-rules-using-the-cli}

1. Efetue login no IBM Cloud.

   Se você tiver uma conta federada:
   ```
   ibmcloud login -sso
   ```
   {: pre}

   Caso contrário, use este comando:

   ```
   ibmcloud login
   ```
   {: pre}

2. Obtenha o ID do grupo de segurança padrão e os detalhes para a VPC

   Execute o comando da CLI a seguir para listar todas as VPCs:

   ```
   ibmcloud is vpc
   ```
   {: pre}

   O nome do grupo de segurança padrão é mostrado sob a coluna `Default Security Group`. Anote esse nome para que seja possível localizar o `ID` quando listar os grupos de segurança (seguinte). 
   
   Agora, liste todos os grupos de segurança na VPC:

   ```
   ibmcloud is security-groups
   ```
   {: pre}

   Salve o ID do grupo de segurança (para o grupo de segurança padrão) em uma variável para que seja possível usá-la posteriormente. Por exemplo, usando o nome de variável `sg`:

   ```
   sg=2d364f0a-a870-42c3-a554-000001162469
   ```
   {: pre}

   Para obter detalhes sobre o grupo de segurança, execute o comando da CLI a seguir:

   ```
   ibmcloud is security-group $sg
   ```
   {: pre}
   
   Como alternativa, é possível inserir o valor do ID real no lugar da variável `$sg`.

3. Atualize o grupo de segurança padrão -- inclua regras que permitam SSH e PING

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
   ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
   ```
   {: codeblock}


A inclusão e a remoção de regras do grupo de segurança são uma operação assíncrona. Geralmente, leva de 1 a 30 segundos para que a mudança entre em vigor.
{: note}
