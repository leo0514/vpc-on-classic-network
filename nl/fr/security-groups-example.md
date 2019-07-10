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

# Configuration de groupes de sécurité à l'aide de l'interface de ligne de commande
{: #setting-up-security-groups-using-the-cli}

Dans l'exemple qui suit, vous allez créer une VSI avec un groupe de sécurité activé, à l'aide de l'interface de ligne de commande (CLI). Le scénario se présente comme suit.

![Groupes de sécurité du VPC IBM](images/security-groups-schematic.svg "Groupes de sécurité du VPC IBM")

Dans la figure, notez que la VSI nommée **SG4** comporte une adresse IP flottante `169.60.208.144` qui lui est attribuée, en plus de son adresse VPC interne `10.0.0.5` ; par conséquent, elle peut communiquer avec l'internet public. Le groupe de sécurité attribué à la VSI **SG4** est nommé "demosg".

La VSI nommée **SG8** est uniquement interne au VPC, avec une adresse IP privée. Le groupe de sécurité attribué à la VSI **SG8** est nommé "my_vpc_sg". Ces deux VSI existent dans le VPC nommé `sgvpc` et sur le même sous-réseau `10.0.0.0/28` pour pouvoir communiquer entre eux.

## Procédure de création d'une VSI avec connexion d'un groupe de sécurité
{: #steps-for-creating-a-vsi-with-a-security-group-attached}

Les règles de groupe de sécurité pour "my_vpc_sg" incluent les fonctions de base de SSH, PING et du protocole TCP sortant.

Notez que vous devez d'abord créer le groupe de sécurité, avec la commande `ibmcloud is sgc`, puis créer la VSI qui inclut le groupe de sécurité créé.

Dans la mesure où cet exemple de code ignore des étapes, voici où vous pouvez trouver plus d'informations :

 * Les instructions de création d'un VPC et d'un sous-réseau sont disponibles dans notre rubrique [Création d'un VPC](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli).

Vous pouvez copier et coller des commandes à partir de cet exemple de code CLI pour vous permettre de créer une VSI à laquelle est connecté un groupe de sécurité. Les réponses système ne sont pas affichées complètement dans cet exemple de code. Vous devez mettre à jour vos commandes avec les ID de ressources corrects pour vos _VPC_, _sous-réseau_, _image_ et _clé_, et le _numéro d'identification du groupe de sécurité_ correct.

1. Créez un groupe de sécurité appelé "my_vpc_sg".

   ```
   ibmcloud is security-group-create my_vpc_sg $vpc
   ```
   {: pre}

   Enregistrez l'ID dans une variable afin de pouvoir l'utiliser ultérieurement, par exemple, dans la variable appelée `sg` :

   ```
   sg=2d364f0a-a870-42c3-a554-000000632953
   ```
   {: pre}

2. Ajoutez des règles autorisant SSH, PING et le protocole TCP sortant.

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
ibmcloud is security-group-rule-add $sg outbound tcp
   ```
   {: codeblock}

3. Créez une VSI avec le groupe de sécurité récemment créé.

   ```
   ibmcloud is instance-create test-instance $vpc us-south-2 b-4x16 $subnet 1000 \ 
   --image $image --keys $key --security-groups $sg
   ```
   {: pre}

## Aide-mémoire sur la liste de commandes
{: #command-list-cheat-sheet}

Pour obtenir une liste complète des commandes CLI de VPC disponibles pour les groupes de sécurité, entrez :

```
ibmcloud is help | grep sg
```
{: pre}

Pour afficher votre groupe de sécurité et ses métadonnées, y compris les règles, entrez (pour l'exemple précédent) :

```
ibmcloud is sg $sg
```
{: pre}

Pour ajouter une règle de groupe de sécurité, voici un exemple de commande permettant d'ajouter une règle entrante PING à un groupe de sécurité :

```
ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0

```
{: pre}
