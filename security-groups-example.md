---

copyright:
  years: 2019

lastupdated: "2019-05-14"

keywords: security group, VSI, ping, TCP, outbound, SG4, rules, metadata, setting up, vpc, vpc network

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

# Setting up security groups using the CLI
{: #setting-up-security-groups-using-the-cli}

In the example that follows, you'll create a VSI with a security group enabled, using the command line interface (CLI). Here's what the scenario looks like.

![Security Groups for IBM VPC](images/security-groups-schematic.svg "Security Groups for IBM VPC"){: caption="Figure: Security Groups for IBM VPC" caption-side="top"}

Notice in the figure that the VSI named **SG4** has a floating IP `169.60.208.144` assigned to it, in addition to its internal VPC address `10.0.0.5`; therefore, it can talk to the public internet. The security group assigned to VSI **SG4** is named "demosg".

The VSI named **SG8** is internal-only to the VPC, with a private IP address. The security group assigned to VSI **SG8** is named "my_vpc_sg". Both of these VSIs exist within the VPC named `sgvpc` and also on the same subnet `10.0.0.0/28` so they can communicate with each other.

## Steps for creating a VSI with a security group attached
{: #steps-for-creating-a-vsi-with-a-security-group-attached}

The security group rules for "my_vpc_sg" will include basic functions of SSH, PING, and outbound TCP.

Notice that you must create the security group first, with the `ibmcloud is sgc` command, and then create the VSI that will include this newly-created security group.

This example code skips a few steps, so here's where you can find more information:

 * Instructions for creating a VPC and a subnet are available in our [Creating a VPC](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli) topic.

You can copy and paste commands from this example CLI code to get you started creating a VSI that has a security group attached. System responses are not shown completely in this sample code. You'll need to update your commands with the correct resource IDs for your _VPC_, _subnet_, _image_, and _key_, and the correct _security group ID number_.

1. Create a security group called “my_vpc_sg”

   ```
   ibmcloud is security-group-create my_vpc_sg $vpc
   ```
   {: pre}

   Save the ID in a variable so we can use it later, for example, in the variable named `sg`:

   ```
   sg=2d364f0a-a870-42c3-a554-000000632953
   ```
   {: pre}

2. Add rules allowing SSH, PING, and outbound TCP

   ```
   ibmcloud is security-group-rule-add $sg inbound tcp --port-min 22 --port-max 22
   ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0
   ibmcloud is security-group-rule-add $sg outbound tcp
   ```
   {: codeblock}

3. Create a VSI with the newly-created security group

   ```
   ibmcloud is instance-create test-instance $vpc us-south-2 b-4x16 $subnet 1000 \
   --image $image --keys $key --security-groups $sg
   ```
   {: pre}

## Command list cheat sheet
{: #command-list-cheat-sheet}

For a complete list of the available VPC CLI commands for security groups, type:

```
ibmcloud is help | grep sg
```
{: pre}

To see your security group and its metadata including rules, you'd type (for the previous example):

```
ibmcloud is sg $sg
```
{: pre}

To add a security group rule, here's an example command for adding a PING inbound rule to a security group:

```
ibmcloud is security-group-rule-add $sg inbound icmp --icmp-type 8 --icmp-code 0

```
{: pre}
