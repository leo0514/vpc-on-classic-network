---



copyright:
  years: 2017,2018, 2019
lastupdated: "2019-06-04"

keywords: VPN, network, encryption, authentication, algorithm, IKE, IPsec, policies, gateway, auto-negotiation

subcollection: vpc-on-classic-network


---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# Using VPN with your VPC
{: #--using-vpn-with-your-vpc}
[comment]: # (linked help topic)

The {{site.data.keyword.cloud}} VPC VPN service allows you to connect private networks in a secure fashion. You can use VPN to set up an IPsec site-to-site tunnel between your VPC and your on-premise private network or another VPC.

For the current {{site.data.keyword.cloud}} VPC release, only policy-based routing is supported.

## Features
{: #vpn-features}

* IKEv1 and IKEv2
* Authentication algorithms: `md5`, `sha1`, `sha256`
* Encryption algorithms: `3des`, `aes128`, `aes256`
* Diffie-Hellman (DH) groups: 2, 5, 14
* IKE negotiation mode: main
* IPSec transform protocol: ESP
* IPSec encapsulation mode: tunnel
* Perfect Forward Secrecy (PFS)
* Dead Peer Detection
* Routing: Policy-based
* Authentication Mode: Pre-shared key
* HA Support in Active/Standby mode only

## APIs available
{: #apis-available}

The following section gives details about APIs you can use for VPN in your IBM Cloud VPC environment. Please see the [VPC REST APIs](https://{DomainName}/apidocs/vpc-on-classic#retrieves-all-ike-policies) page for more details.

### VPN gateways and VPN connections
{: #vpn-gateways-and-vpn-connections}

| Description | API |
|----------------------------|-------------|
| Creates a VPN gateway | POST /vpn_gateways |
| Retrieves VPN gateways | GET /vpn_gateways |
| Retrieves a VPN gateway | GET /vpn_gateways/{id} |
| Deletes a VPN gateway | DELETE /vpn_gateways/{id} |
| Updates a VPN gateway | PATCH /vpn_gateways/{id} |
| Creates a new VPN connection | POST /vpn_gateways/{vpn_gateway_id}/connections |
| Retrieves VPN connections | GET /vpn_gateways/{vpn_gateway_id}/connections |
| Retrieves a VPN connection | GET /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Deletes a VPN connection | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Updates a VPN connection | PATCH /vpn_gateways/{vpn_gateway_id}/connections/{id} |
| Retrieves all local CIDRs for a VPN connection | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs |
| Deletes a local CIDR from a VPN connection | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Checks if a specific local CIDR exists on a VPN connection | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Sets a local CIDR on a VPN connection | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/local_cidrs/{prefix_address}/{prefix_length} |
| Retrieves all peer CIDRs for a VPN connection | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs |
| Deletes a peer CIDR from a VPN connection | DELETE /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Checks if a specific peer CIDR exists on a VPN connection | GET /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |
| Sets a peer CIDR on a VPN connection | PUT /vpn_gateways/{vpn_gateway_id}/connections/{id}/peer_cidrs/{prefix_address}/{prefix_length} |

### IKE policies
{: #ike-policies}

| Description | API |
|-----------------------------|--------------|
| Retrieves all IKE policies | GET /ike_policies |
| Creates an IKE policy | POST /ike_policies |
| Deletes an IKE policy | DELETE /ike_policies/{id} |
| Retrieves an IKE policy | GET /ike_policies/{id} |
| Updates an IKE policy | PATCH /ike_policies/{id} |
| Retrieves all the connections that use the specified IKE policy | GET /ike_policies/{id}/connections |

### IPsec policies
{: #ipsec-policies}

| Description | API |
|---------------------|-------------|
| Retrieves all IPSec policies | GET /ipsec_policies |
| Creates an IPSec policy | POST /ipsec_policies |
| Deletes an IPSec policy | DELETE /ipsec_policies/{id} |
| Retrieves an IPSec policy | GET /ipsec_policies/{id} |
| Updates an IPSec policy | PATCH /ipsec_policies/{id} |
| Retrieves all the connections that use the specified IPsec policy | GET /ipsec_policies/{id}/connections |

## VPN example
{: #vpn-example}

In the following example, you'll be able to connect two VPCs together using VPN, which means you can connect
subnets in two separate VPCs as if they were a single network. The IP addresses of the subnets must not overlap.
Here is what the scenario looks like (with some VMs added in each VPC):

![VPN for IBM VPC](images/vpc-vpn.svg "VPN for IBM VPC"){: caption="Figure: VPN for IBM VPC" caption-side="top"}

### Example steps
{: #vpn-example-steps}

The example steps that follow skip the prerequisite steps of using IBM Cloud API or CLI to create VPCs. For more information, see [Getting Started](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) and [VPC setup with APIs](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

Optionally, you can create a VPN gateway using the UI. Steps can be found in the [Console Tutorial document](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn).

#### Step 1. Create a VPN gateway in your VPC subnet
{: #step-1-create-a-vpn-gateway-in-your-vpc-subnet}

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways?version=2019-05-31&generation=1" \
    -d '{
            "name": "vpn-gateway-1",
            "subnet": {"id": $subnet1}
        }'
```
{: codeblock}

Sample output:
```
{
    "id": "7fd72524-6e2d-49a6-b975-0071efccd89a",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:7fd72524-6e2d-49a6-b975-0071efccd89a",
    "name": "vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a",
    "created_at": "2018-07-06T19:19:28.694388Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.167"
    },
    "subnet": {
        "id": "f45ee0be-cf3f-41ca-a279-23139110aa58",
        "name": "subnet-1",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f45ee0be-cf3f-41ca-a279-23139110aa58"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

Ensure the following fields are saved for subsequent steps.
* `id`. This is the VPN gateway ID, and it will be referred to as `$gwid1`.
* `address`. This is the public IP address of the VPN gateway, and it will be referred to as `$gwaddress1`.

The gateway status appears as `pending` while the VPN gateway is being created, and the status becomes `available` once creation is complete. Creation may take some time.
{: note}


You can check the gateway's status with the following command:

```bash
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/vpn_gateways/$gwid1?version=2019-05-31&generation=1"
```
{: codeblock}

#### Step 2. Create a second VPN gateway on a different VPC
{: #step-2-create-a-second-vpn-gateway-on-a-different-vpc}

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-gateway-2",
                "subnet": {"id": $subnet2}
            }'
```
{: codeblock}

Sample output:
```
{
    "id": "f72559a3-2fac-4958-b937-54474e6a8a8d",
    "crn": "crn:v1:bluemix:public:is:us-south:a/b668aa2600ac21c890aef16a6210b2fd::vpn:f72559a3-2fac-4958-b937-54474e6a8a8d",
    "name": "vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d",
    "created_at": "2018-07-06T19:33:23.789675Z",
    "status": "pending",
    "public_ip": {
        "address": "169.61.161.150"
    },
    "subnet": {
        "id": "f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4",
        "name": "subnet-2",
        "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/f72c7f7c-0fa5-42d1-9bdc-9e0acad53cb4"
    },
    "resource_group": {
        "id": "d28a2jsiw1pl2g22q8462tyr321416z2",
        "href": "https://resource-manager.bluemix.net/v1/resource_groups/d28a2jsiw1pl2g22q8462tyr321416z2"
    }
}
```
{: screen}

Be sure to save the following fields for subsequent steps.
* `id`. This is the VPN gateway ID, and it will be referred to as `$gwid2`.
* `address`. This is the public IP address of the VPN gateway, and it will be referred to as `$gwaddress2`.


#### Step 3. Create a VPN connection from the first VPN gateway to the second VPN gateway
{: #step-3-create-a-vpc-connection-from-the-first-vpn-gateway-to-the-second-vpn-gateway}

When you're creating the connection, set `local_cidrs` to the subnet on **VPC one** and `peer_cidrs` to the subnet on **VPC two**.

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-2",
                "peer_address": $gwaddress2,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

Sample output:
```
{
    "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "name": "vpn-connection-to-vpn-gateway-2",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.0.0/24"
    ],
    "peer_address": "169.61.161.150",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
        "interval": 30,
        "timeout": 120
    },
    "created_at": "2018-07-06T19:50:49.252072Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### Step 4. Create a VPN connection from the second VPN gateway to the first VPN gateway
{: #step-4-create-a-vpn-connection-from-the-second-vpn-gateway-to-the-first-vpn-gateway}

When you're creating the connection, set `local_cidrs` to the subnet on **VPC two** and `peer_cidrs` to the subnet on **VPC one**.

```bash
curl -H "Authorization: $iam_token" -X POST "$rias_endpoint/v1/vpn_gateways/$gwid2/connections?version=2019-05-31&generation=1" \
        -d '{
                "name": "vpn-connection-to-vpn-gateway-1",
                "peer_address": $gwaddress1,
                "psk": "VPNDemoPassword",
                "local_cidrs": [ "<LOCAL-CIDR>" ],
                "peer_cidrs": [ "<PEER-CIDR>" ]
            }'
```
{: codeblock}

Sample output:
```
{
    "id": "1d4dbacq-673d-2qed-hf68-858961739gf0",
    "name": "vpn-connection-to-vpn-gateway-1",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/f72559a3-2fac-4958-b937-54474e6a8a8d/connections/1d4dbacq-673d-2qed-hf68-858961739gf0",
    "local_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_cidrs": [
        "192.168.100.0/24"
    ],
    "peer_address": "169.61.161.167",
    "admin_state_up": true,
    "psk": "VPNDemoPassword",
    "dead_peer_detection": {
        "action": "none",
        "interval": 30,
        "timeout": 120
    },
    "created_at": "2018-07-06T19:54:14.961597Z",
    "route_mode": "policy",
    "authentication_mode": "psk",
    "status": "down"
}
```
{: screen}

#### Step 5. Verify connectivity
{: #step-5-verify-connectivity}

Once the VPN connection is established, you will be able to reach your instances on subnet two from subnet one, and vice versa.

You can check the status of the VPN connection as follows:
```bash
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/vpn_gateways/$gwid1/connections?version=2019-05-31&generation=1"
```
{: codeblock}

Sample output:
```
{
    "first": {
        "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections?limit=10"
    },
    "limit": 10,
    "connections": [
        {
            "id": "a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "name": "vpn-connection-to-vpn-gateway-2",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/vpn_gateways/7fd72524-6e2d-49a6-b975-0071efccd89a/connections/a252d380-0784-45ff-8fc0-c2b58e446b4d",
            "local_cidrs": [
                "192.168.100.0/24"
            ],
            "peer_cidrs": [
                "192.168.0.0/24"
            ],
            "peer_address": "169.61.161.150",
            "admin_state_up": true,
            "psk": "VPNDemoPassword",
            "dead_peer_detection": {
                "action": "none",
                "interval": 30,
                "timeout": 120
            },
            "created_at": "2018-07-06T19:50:49.252072Z",
            "route_mode": "policy",
            "authentication_mode": "psk",
            "status": "up"
        }
    ]
}
```
{: screen}

## Quotas
{: #see-vpn-quotas}

See our [VPC Quotas](/docs/vpc-on-classic?topic=vpc-on-classic-quotas#vpn-quotas) topic for VPN quotas.

## Policy auto-negotiation
{: #policy-auto-negotiation}

The use of IKE and IPsec policies to configure a VPN connection is optional. When no policy is selected, default proposals are chosen automatically for a process referred to as _auto-negotiation_. 

The IBM Cloud auto-negotiation uses **IKEv2** and therefore, the on-premise device must also use **IKEv2**. Use a customized IKE policy if your on-premise device does not support **IKEv2**.
{: note}

### IKE auto-negotiation (Phase 1)
{: #ike-auto-negotiation-phase-1}

The following encryption, authentication and Diffie-Hellman Group options may be used in any combination:

|    | Encryption | Authentication | DH Group |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | 2  |
| 2  | aes256 | sha256 | 5  |
| 3  | 3des   | md5    | 14 |

### IPsec auto-negotiation (Phase 2)
{: #ipsec-auto-negotiation-phase-2}

The following encryption and authentication options may be used in any combination:

|    | Encryption | Authentication | DH Group |
|----|------------|----------------|----------|
| 1  | aes128 | sha1   | disabled  |
| 2  | aes256 | sha256 | disabled  |
| 3  | 3des   | md5    | disabled  |

## FAQ
{: #vpn-faq}

**When I create a VPN gateway, can I create VPN connections at the same time?**

If you use the API or CLI, VPN connections must be created after the VPN gateway is created. In the IBM Cloud console, you can create the gateway and a connection at the same time.

**If I delete a VPN gateway that has VPN connections attached, what will happen to the connections?**

The VPN connections are deleted along with the VPN gateway.

**Will IKE or IPSec policies be deleted if I delete a VPN gateway or VPN connection?**

No, IKE and IPSec policies can apply to multiple connections.

**What happens to a VPN gateway if I try to delete the subnet that the gateway is located on?**

The subnet cannot be deleted if any instances are present, including the VPN gateway.

**Are there default IKE and IPsec policies?**

When you create a VPN connection without referencing a policy ID (IKE or IPsec), auto-negotiation is used.

**Why do I need to choose a subnet during VPN gateway provisioning?**

The subnet connects the VPN gateway with other resources in your VPC. The recommended best practice is to create
a dedicated subnet for the VPN gateway, with no other VPC instances on this subnet, to ensure there
are enough free private IPs in the subnet. A VPN gateway needs 8 private IP addresses to accomodate HA and rolling upgrades.

**In which zone does the VPN gateway reside?**

The VPN gateway resides in the zone with the subnet you chose during provisioning. Remember that the VPN gateway only serves the VPC instances in the same zone and the same VPC. Therefore, VPC instances in other zones can't leverage the VPN gateway
to communicate with an on-premise private network. For zone fault tolerance, you must deploy one VPN gateway per zone.

**What should I do if I am using ACLs on the subnet that is used to deploy the VPN gateway?**

Make sure the following ACL rules are in place to allow management traffic and VPN tunnel traffic:

* **Inbound rules**
    - Allow protocol TCP source port 9091
    - Allow protocol TCP source port 10514
    - Allow protocol TCP source port 443
    - Allow protocol TCP source port 80
    - Allow protocol TCP source port 53
    - Allow protocol UDP source port 53
    - Allow protocol ALL source IP is VPN peer gateway public IP
    - Allow protocol TCP destination port 443
    - Allow protocol TCP destination port 56500
    - Allow traffic between instances in VPC and your on-premise private network
    - Allow ICMP traffic

* **Outbound rules**
   - Allow all traffic

**What should I do if I am using ACLs or security groups on the subnets which need to communicate with on-premise private network?**

You'll need to ensure that the ACL rules or security group rules are in place to allow traffic between instances in your VPC and your on-premise private network.

**Does VPN for VPC support HA configurations?**

Yes, it supports HA in an Active / Standby configuration.

**Are there plans to support SSL VPN?**

No, only IPsec site-to-site is supported.

**Are there any caps on throughput for site-to-site VPNaaS?**

We support up to 650 Mbps of throughput.

**Is PSK and Certificate based IKE authentication supported for VPNaaS?**

Only PSK authentication is supported.

**Can you use VPN for VPC as a VPN Gateway for your IBM Cloud Infrastructure Classic?**

No, in order to use VPN Gateway in your IBM Cloud Infrastructure Classic environment, you must use the [IPsec VPN] (https://cloud.ibm.com/catalog/infrastructure/ipsec-vpn).
