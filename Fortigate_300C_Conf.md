---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-05-14"

keywords: peering, FortiGate, connection, secure, vpc

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Creating a secure connection with a remote FortiGate peer
{: #creating-a-secure-connection-with-a-remote-fortigate-peer}

This document is based on FortiGate 300C, Firmware Version	v5.2.13, build762 (GA).

The example steps that follow skip the prerequisite steps of using {{site.data.keyword.cloud}} API or CLI to create Virtual Private Clouds. For more information, see [Getting Started](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started) and [VPC setup with APIs](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis).

## Example steps
{: #fortigate-example-steps}

The topology for connecting to the remote FortiGate peer is similar to creating a VPN connection between two VPCs. However, one side is replaced by the FortiGate 300C unit.

![enter image description here](./images/vpc-vpn-fg-figure.png)

### To create a secure connection with the remote Fortigate peer
{: #to-create-a-secure-connection-with-the-remote-fortigate-peer}

Go to **VPN \> IPsec \> Tunnels** and create a new custom tunnel or edit an existing tunnel.

![vpc-vpn-fg-start](./images/vpc-vpn-fg-start.JPG)

When a FortiGate unit receives a connection request from a remote VPN peer, it uses IPsec Phase 1 parameters to establish a secure connection and authenticate that VPN peer. Then, if the security policy permits the connection, the FortiGate unit establishes the tunnel using IPsec Phase parameters and applies the IPsec security policy. Key management, authentication, and security services are negotiated dynamically through the IKE protocol.

**To support these functions, the following general configuration steps must be performed by the FortiGate unit:**

* Define the Phase 1 parameters that the FortiGate unit requires to authenticate the remote peer and establish a secure connection.

* Define the Phase 2 parameters that the FortiGate unit requires to create a VPN tunnel with the remote peer.

* Create security policies to control the permitted services and permitted direction of traffic between the IP source and destination addresses.

By default, some typical Phase 1 and Phase 2 parameters have been set.

### Configuring for the IBM Cloud VPC VPN
{: #fortigate-configuring-for-the-ibm-cloud-vpc-vpn}

To connect to the IBM Cloud VPC's VPN capability, we recommend the following configuration:

1. Choose IKEv2 in authentication;
2. Enable `DH-group 2` in the Phase 1 proposal
3. Set `lifetime = 36000` in the Phase 1 proposal
4. Disable PFS in the Phase 2 proposal
5. Set `lifetime = 10800` in the Phase 2 proposal
6. Input your subnet's information in Phase 2
7. The remaining parameters keep their default values.

## To create a secure connection with the local IBM Cloud VPC
{: #fortigate-to-create-a-secure-connection-with-the-local-ibm-cloud-vpc}

To create a secure connection, you'll create the VPN connection within your VPC, which is similar to the 2 VPC example.

* Create a VPN gateway on your VPC subnet  along with a VPN connection between the VPC and the FortiGate unit, setting `local_cidrs` to the subnet value on the VPC, and `peer_cidrs` to the subnet value on the FortiGate.

**Note:** The gateway status appears as `pending` while the VPN gateway is being created, and the status becomes `available` once creation is complete. Creation may take some time.

![enter image description here](images/vpc-vpn-fg-connection.png)

### Check the status of the secure connection
{: #fortigate-check-the-status-of-the-secure-connection}

You can check the status of your connection through the IBM Cloud console. Also, you could try to do a `ping` from site to site using the VSIs.

![enter image description here](images/vpc-vpn-fg-status.JPG)
