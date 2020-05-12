---



copyright:
  years: 2017,2018, 2019, 2020
lastupdated: "2020-05-06"

keywords:

subcollection: vpc-on-classic-network


---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:important: .important}
{:new_window: target="_blank"}
{:faq: data-hd-content-type='faq'}
{:support: data-reuse='support'}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}
{:external: target="_blank" .external}

# Using VPN with your VPC
{: #--using-vpn-with-your-vpc}
[comment]: # (linked help topic)

The {{site.data.keyword.cloud}} VPN for Virtual Private Cloud allows you to connect private networks in a secure fashion. You can use VPN to set up an IPsec site-to-site tunnel between your VPC and your on-premise private network or another VPC.
{: shortdesc}

Newer generation available. For more information, see [Using VPN](/docs/vpc?topic=vpc-using-vpn) for generation 2 compute resources.
{:important}

## Features
{: #vpn-features}

* IKEv1 and IKEv2
* Authentication algorithms: `md5`, `sha1`, `sha256`
* Encryption algorithms: `3des`, `aes128`, `aes256`
* Diffie-Hellman (DH) groups: 2, 5, 14
* IKE negotiation mode: main
* IPsec transform protocol: ESP
* IPsec encapsulation mode: tunnel
* Perfect Forward Secrecy (PFS)
* Dead Peer Detection
* Routing: Policy-based
* Authentication Mode: Pre-shared key
* HA Support in Active/Standby mode only

## VPN example
{: #vpn-example}

In the following example, you'll be able to connect two VPCs together using VPN, which means you can connect
subnets in two separate VPCs as if they were a single network. The IP addresses of the subnets must not overlap.
Here is what the scenario looks like (with some VMs added in each VPC):

![VPN for IBM VPC](images/vpc-vpn.svg "VPN for IBM VPC"){: caption="Figure: VPN for IBM VPC" caption-side="top"}

While the diagram above does not show them, the VPN Gateways use Floating IPs
(FIPs) to communicate across regions. This means that the encrypted traffic
will traverse the public internet and will incur charges.
{: note}

### Example steps
{: #vpn-example-steps}

The example steps that follow skip the prerequisite steps of using IBM Cloud API or CLI to create VPCs. For more information, see [Getting Started](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) and [VPC setup with APIs](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

Optionally, you can create a VPN gateway using the UI. Steps can be found in [Creating a VPC using the IBM Cloud console](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn).

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
        "192.168.0.0/24"
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

## Access service endpoints using VPN
{: #build-se-connectivity-using-vpn}

VPC allows access to your service endpoint (SE) from an on-premise network, by means of a virtual private network (VPN). Here are the steps for setting up access to a service endpoint:

* **Step 1:** Get the IP of your service endpoint. IBM Cloud supports two types of service endpoints: Infrastructure as a Service (IaaS) endpoints and Cloud Service Endpoints (CSE). The IaaS endpoints are hosted in the IP address ranges 161.26.0.0/16; cloud service endpoints are hosted in the IP address ranges 166.8.0.0/14.
* **Step 2:** Create a VPN gateway. If you already have one, you can skip this step.
* **Step 3:** Create a VPN connection. In the VPN configuration, your local subnets should include the range for the service endpoints `166.8.0.0/14` or `161.26.0.0/16`.

## Limitations
{: #vpn-limitations}

* See our [VPC Quotas](/docs/vpc-on-classic?topic=vpc-on-classic-quotas#vpn-quotas) topic for VPN quotas.
* Only policy based routing is supported at this time (not route based). IBM VPN is following these industry standards, and all compatible devices with these RFCs should be interoperable with IBM VPN using policy based VPN mode:
  * [RFC 4109](http://www.faqs.org/rfcs/rfc4109.html){: external},
  * [RFC 7296](http://www.faqs.org/rfcs/rfc7296.html){: external},
  * [RFC 3947](http://www.faqs.org/rfcs/rfc3947.html){: external},
  * [RFC 7383](http://www.faqs.org/rfcs/rfc7383.html){: external},
  * [RFC 3706](http://www.faqs.org/rfcs/rfc3706.html){: external}.

## Troubleshooting
{: #vpn-troubleshooting}

We are sorry you are having technical problems with your VPN Gateway. If the following suggestions do not resolve your problem, [contact support](/docs/vpc-on-classic-network?topic=vpc-on-classic-getting-help-and-support) and be ready to provide the IBM VPN gateway ID, public IPs, connection names, subnet information (to and from), device types, and the VPN gateway configuration on the peer side.

### VPN gateway goes from status of `pending` to status of `failed` after 30 minutes
{: #vpn-ts-failed}
{: troubleshoot}
{: support}

Check the network ACLs of the subnet where the VPN gateway was provisioned. The subnet traffic rules may be blocking the successful provision of the VPN gateway. If necessary, permit all traffic on both inbound and outbound during provisioning.

### VPN gateway is available, but the VPN connection is down
{: #vpn-gateway-available-but-vpn-connection-down}
{: troubleshoot}
{: support}

- A VPN gateway will initiate the VPN connection negotiation when a new connection is added and will retry up to 5 times. If the connection remains down, try initiating the connection again.
- Make sure the peer VPN gateway is using matched IKE/IPsec policy and pre-shared key. See [IKE auto-negotiation (Phase 1)](#ike-auto-negotiation-phase-1) and [IPsec auto-negotiation (Phase 2)](#ipsec-auto-negotiation-phase-2) for options.
- Make sure traffic is not blocked by firewall between IBM VPN gateway and peer VPN gateway. Use tools, like ping, to test the network connectivity between the gateways.
- View the VPN logs to determine any other factors which leave the VPN connection down. See [Using LogDNA to view VPN logs](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-using-logdna-to-view-vpn-logs) to find out how to view these logs.

### The connection is up but you cannot access resources in VPC from on-premise network
{: #vpn-connection-up-but-cannot-access-resources-in-vpc-from-onpremises}
{: troubleshoot}
{: support}

- Check the routing configuration on the on-premise network and make sure the traffic to IBM VPC is routed to peer VPN gateway properly.
- Make sure firewall rules on the peer VPN gateway allow traffic between the subnet in IBM and on-premise network.
- Make sure network ACLs and security group rules in IBM VPC allow traffic between subnet in IBM and on-premise network.

## FAQs
{: #vpn-faq}

### When I create a VPN gateway, can I create VPN connections at the same time?
{: #when-i-create-a-vpn-gateway-can-i-create-vpn-connections}
{: faq}
{: support}

If you use the API or CLI, VPN connections must be created after the VPN gateway is created. In the IBM Cloud console, you can create the gateway and a connection at the same time.

### If I delete a VPN gateway that has VPN connections attached, what will happen to the connections?
{: #if-i-delete-vpn-gateway-that-has-vpn-connections-attached}
{: faq}
{: support}

If you delete a VPN gateway that has attached connections, the VPN connections are deleted along with the VPN gateway.
 
### Will IKE or IPsec policies be deleted if I delete a VPN gateway or VPN connection?
{: #will-ike-or-ipsec-policies-be-deleted}
{: faq}
{: support}

IKE and IPsec policies can apply to multiple connections. Therefore, IKE or IPsec policies are not deleted if you delete a VPN gateway or VPN connection.

### What happens to a VPN gateway if I try to delete the subnet that the gateway is located on?
{: #what-happens-to-a-vpn-gateway-if-i-try-to-delete-the-subnet}
{: faq}
{: support}

A subnet cannot be deleted if any instances are present, including the VPN gateway.

### Are there default IKE and IPsec policies?
{: #are-there-default-ike-ipsec-policies}
{: faq}
{: support}

When you create a VPN connection without referencing a policy ID (IKE or IPsec), auto-negotiation is used.

### Why do I need to choose a subnet during VPN gateway provisioning?
{: #why-do-i-need-to-choose-a-subnet-during-provisioning}
{: faq}
{: support}

A subnet connects the VPN gateway with other resources in your VPC. The recommended best practice is to create
a dedicated subnet for the VPN gateway, with no other VPC instances on this subnet, to ensure there
are enough free private IPs in the subnet. A VPN gateway needs 8 private IP addresses to accommodate HA and rolling upgrades.

### In which zone does the VPN gateway reside?
{: #in-which-zone-does-the-vpn-gateway-reside}
{: faq}
{: support}

The VPN gateway resides in the zone with the subnet you chose during provisioning. Remember that the VPN gateway only serves the VPC instances in the same zone and the same VPC. Therefore, VPC instances in other zones can't leverage the VPN gateway
to communicate with an on-premise private network. For zone fault tolerance, you must deploy one VPN gateway per zone.

### What should I do if I am using ACLs on the subnet that is used to deploy the VPN gateway?
{: #what-should-i-do-if-i-am-using-acls-on-the-subnet-used-to-deploy-the-vpn}
{: faq}
{: support}

If you are using ACLs on the subnet used to deploy the VPN gateway, make sure to follow the link to allow the [ACLs required for instance creation](/docs/vpc-on-classic?topic=vpc-on-classic-rias-error-messages#acl_rule_does_not_allow).

In addition, make sure the following ACL rules are in place to allow management traffic and VPN tunnel traffic:

* **Inbound rules**
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

### What should I do if I am using ACLs or security groups on the subnets that must communicate with an on-premises private network?
{: #what-should-i-do-if-i-am-using-acls-or-security-groups-on-the-subnet}
{: faq}
{: support}

If you are using ACLs or security groups on the subnets that must communicate with an on-premises private network, you'll need to ensure that the ACL rules or security group rules are in place to allow traffic between instances in your VPC and your on-premise private network.

### Does VPN for VPC support HA configurations?
{: #does-vpn-for-vpc-support-ha}
{: faq} 

VPN for VPC supports high availability in an Active / Standby configuration.

### Are there plans to support SSL VPN?
{: #are-there-plans-to-support-ssl-vpn}
{: faq} 

Only IPsec site-to-site is supported, not SSL VPN.

### Are there any caps on throughput for site-to-site VPNaaS?
{: #are-there-any-caps-on-throughput-for-site-to-site-VPNaaS}
{: faq} 

We support up to 650 Mbps of throughput for site-to-site VPNaaS.

### Is PSK and certificate-based IKE authentication supported for VPNaaS?
{: #is-certificate-based-ike-authentication-supported-for-vpnaas}
{: faq} 

Only PSK authentication is supported, not certificate-based IKE.

### Can you use VPN for VPC as a VPN gateway for your IBM Cloud infrastructure classic?
{: #can-you-use-vpn-for-vpc-as-a-vpn-gateway-for-infrastructure-classic]
{: faq} 

To use a VPN gateway in an IBM Cloud infrastructure classic environment, you must use the [IPsec VPN](https://{DomainName}/catalog/infrastructure/ipsec-vpn){: external}.

### Can VPN for VPC, along with a Classic Access VPC, access IBM Cloud infrastructure classic resources?
{: #can-vpn-for-vpc-with-classic-access-access-classic-resources}
{: faq} 

Currently, VPN for VPC (with a Classic Access VPC) cannot access IBM Cloud infrastructure classic resources.

### What will rekey collision cause?
{: #what-will-rekey-collsion-cause}
{: faq} 

If you use IKEv1, rekey collision deletes the IKE/IPsec SA. To recreate the IKE/IPsec SA, set the connection admin state to `down` and then `up` again. You can use IKEv2 to minimize rekey collisions.

### Is it possible to connect to route-based VPNs with multiple tunnels?
{: #is-it-possible-to-connect-to-route-based-vpns-with-multiple-tunnels}
{: faq} 

Currently, it is not possible to connect to route-based VPNs with multiple tunnels.

### Is it possible to view logs from the VPN gateway for debugging purposes?
{: #is-it-possible-to-view-logs}
{: faq} 

To view VPN gateway logs for debugging purposes, see [Using LogDNA to view VPN logs](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-using-logdna-to-view-vpn-logs).
