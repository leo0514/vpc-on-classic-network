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
{:note: .note}
{:table: .aria-labeledby="caption"}
{:download: .download}


# Creating a secure connection with a remote FortiGate peer
{: #creating-a-secure-connection-with-a-remote-fortigate-peer}

You can connect a FortiGate peer to a VPN gateway in an existing {{site.data.keyword.cloud}} Virtual Private Cloud (VPC).
{: shortdesc}

These examples are based on FortiGate 300C, Firmware Version	v5.2.13, build762 (GA).
{: note}

## Topology
{: #fortigate-architecture}

The following diagram shows a VPN gateway in an IBM Cloud VPC connecting to a FortiGate peer.

![Tunnel with FortiGate peer](./images/vpc-vpn-fg-figure.png)

## Configuring the FortiGate peer
{: #fortigate-example-steps}

Go to **VPN \> IPsec \> Tunnels** and create a new custom tunnel or edit an existing tunnel.

![FortiGate user interface](./images/vpc-vpn-fg-start.jpg)

When a FortiGate unit receives a connection request from a remote VPN peer, it uses IPsec Phase 1 parameters to establish a secure connection and authenticate that VPN peer. Then, if the security policy permits the connection, the FortiGate unit establishes the tunnel using IPsec Phase parameters and applies the IPsec security policy. Key management, authentication, and security services are negotiated dynamically through the IKE protocol.

**To support these functions, the following general configuration steps must be performed by the FortiGate unit:**

* Define the Phase 1 parameters that the FortiGate unit requires to authenticate the remote peer and establish a secure connection.
* Define the Phase 2 parameters that the FortiGate unit requires to create a VPN tunnel with the remote peer.
* Create security policies to control the permitted services and permitted direction of traffic between the IP source and destination addresses. By default, some typical Phase 1 and Phase 2 parameters have been set.

## Configuring the VPN gateway
{: #fortigate-configuring-for-the-ibm-cloud-vpc-vpn}

To connect to the IBM Cloud VPC's VPN capability, we recommend the following configuration:

* Choose IKEv2 in authentication;
* Enable `DH-group 2` in the Phase 1 proposal
* Set `lifetime = 36000` in the Phase 1 proposal
* Disable PFS in the Phase 2 proposal
* Set `lifetime = 10800` in the Phase 2 proposal
* Input your subnet's information in Phase 2

The remaining parameters keep their default values.

* To create a secure connection, you'll create the VPN connection within your VPC, which is similar to the two VPC example.
* Create a VPN gateway on your VPC subnet  along with a VPN connection between the VPC and the FortiGate unit, setting `local_cidrs` to the subnet value on the VPC, and `peer_cidrs` to the subnet value on the FortiGate.

The gateway status appears as `pending` while the VPN gateway is being created, and the status becomes `available` once creation is complete. Creation may take some time.
{: note}

![vpn gateway configuration screen](images/vpc-vpn-fg-connection.png)

## Checking the status of the secure connection
{: #fortigate-check-the-status-of-the-secure-connection}

You can check the status of your connection through the IBM Cloud console. Also, you could try to do a `ping` from site to site using the VSIs.

![connection status](images/vpc-vpn-fg-status.jpg)
