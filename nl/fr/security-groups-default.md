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

# Mise à jour du groupe de sécurité par défaut
{: #updating-the-default-security-group}


Le groupe de sécurité par défaut est très similaire à tout autre groupe de sécurité, à l'exception du fait qu'il ne peut pas être supprimé.

Chaque VPC comporte un groupe de sécurité par défaut, avec des règles pour autoriser :

* Le trafic entrant de tous les membres du groupe.
* Tout le trafic sortant.

Si vous modifiez les règles du groupe de sécurité par défaut, ces règles modifiées s'appliquent ensuite à tous les serveurs actuels et futurs du groupe.

Les règles entrantes autorisant le ping et SSH ne sont pas automatiquement ajoutées au groupe de sécurité par défaut. Vous pouvez modifier les règles du groupe de sécurité par défaut à l'aide de l'API REST, de l'`interface de ligne de commande IBM Cloud`, ou de l'interface utilisateur.

## Exemple : modification des règles du groupe de sécurité par défaut à l'aide de l'interface de ligne de commande
{: #example-modifying-the-default-security-group-rules-using-the-cli}

1. Connectez-vous à IBM Cloud.

   Si vous disposez d'un compte fédéré :
   ```
   ibmcloud login -sso
   ```
   {: pre}

   Sinon, utilisez cette commande :

   ```
   ibmcloud login
   ```
   {: pre}

2. Obtenez l'ID du groupe de sécurité par défaut et les détails du VPC.

   Exécutez la commande de l'interface de ligne de commande suivante pour répertorier tous les VPC :

   ```
   ibmcloud is vpc
   ```
   {: pre}

   Le nom du groupe de sécurité par défaut s'affiche sous la colonne `Groupe de sécurité par défaut`. Notez-le afin de pouvoir trouver l'`ID` lorsque vous répertoriez les groupes de sécurité (ultérieurement). 
   
   Répertoriez maintenant tous les groupes de sécurité dans le VPC :

   ```
   ibmcloud is security-groups
   ```
   {: pre}

   Sauvegardez l'ID du groupe de sécurité (pour le groupe de sécurité par défaut) dans une variable afin de pouvoir l'utiliser ultérieurement. Par exemple, à l'aide du nom de variable `sg` :

   ```
   sg=2d364f0a-a870-42c3-a554-000001162469
   ```
   {: pre}

   Pour obtenir des détails sur le groupe de sécurité, exécutez la commande de l'interface de ligne de commande suivante :

   ```
   ibmcloud is security-group $sg
   ```
   {: pre}
   
   Vous pouvez également insérer la valeur d'ID réelle à la place de la variable `$sg`.

3. Mettez à jour le groupe de sécurité par défaut -- ajoutez des règles autorisant les opérations SSH et PING

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
   ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
   ```
   {: codeblock}


L'ajout et la suppression de règles de groupe de sécurité sont des opérations asynchrones. L'entrée en vigueur de la modification prend habituellement de 1 à 30 secondes.
{: note}
