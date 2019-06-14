---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: security groups, RIAS, API, delete, create

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}

# Setting up security groups using the APIs
{: #setting-up-security-groups-using-the-apis}

The following example demonstrates how to create and manage security groups, using the {{site.data.keyword.cloud}} VPC APIs.

## Prerequisites
{: #security-group-prerequisites}

To use security groups, first you must have a running {{site.data.keyword.cloud_notm}} VPC.

Instructions for creating a VPC and a subnet are available in our [Creating a VPC](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis) topic.

## Step 1: Create a security group
{: #step-1-create-a-security-group}

Create a security group named `my-security-group` in your {{site.data.keyword.cloud_notm}} VPC.

```
curl -X POST "$rias_endpoint/v1/security_groups?version=2019-05-31&generation=1" \
  -H "Authorization: $iam_token" \
  -d '{
        "name": "my-security-group",
        "vpc": { "id": "'$vpc'" }
      }'
```
{: pre}

Save the ID in a variable so we can use it later, for example, the variable `sg`:

```
sg=2d364f0a-a870-42c3-a554-000000632953
```
{: pre}

## Step 2: Add a rule to allow SSH connections
{: #step-2-add-a-rule-to-allow-ssh-connections}

Create a rule on the security group that will allow inbound connections on port 22.

```
curl -X POST "$rias_endpoint/v1/security_groups/$sg/rules?version=2019-05-31&generation=1" \
  -H "Authorization: $iam_token" \
  -d '{
        "direction": "inbound",
        "protocol": "tcp",
        "port_min": 22,
        "port_max": 22
      }'
```
{: pre}

## Step 3: Delete the security group (Optional)
{: #step-3-delete-the-securiy-group-optional}

To clean up the security group, it cannot be associated with any network interfaces, and it cannot be referenced by a rule in a different security group.

```
curl -X DELETE "$rias_endpoint/v1/security_groups/$sg?version=2019-05-31&generation=1" \
  -H "Authorization: $iam_token"
```
{: pre}
