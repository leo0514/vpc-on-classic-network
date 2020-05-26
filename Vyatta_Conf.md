---

copyright:
  years: 2017, 2020
lastupdated: "2020-05-26"

keywords:

subcollection: vpc-on-classic-network

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:codeblock: .codeblock}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:table: .aria-labeledby="caption"}
{:download: .download}

# Creating a secure connection with a remote Vyatta peer
{: #creating-a-secure-connection-with-a-remote-vyatta-peer}

You can connect a Vyatta peer to a VPN gateway in an existing {{site.data.keyword.cloud}} Virtual Private Cloud (VPC).
{: shortdesc}

These examples are based on the Vyatta version: AT&T vRouter 5600 1801d.
{: note}

## Topology
{: #vyatta-architecture}

The following diagram shows a VPN gateway in an IBM Cloud VPC connecting to a Vyatta peer.

![Tunnel with Vyatta peer](images/vpc-vpn-vy-figure.png)

## Before you begin
{: #vyatta-preparation}

To set up your remote Vyatta peer, you need:

* The public IP address of a VPN gateway in a VPC
* The subnet of the VPN gateway
* The public IP address of the Vyatta
* The subnet of the Vyatta that you want to connect using a VPN

## Configuring the Vyatta peer
{: #configure-vyatta-peer}

There are two ways you can run the configuration on your Vyatta:

1. Log in to the Vyatta and run the file `create_vpn.vcli` using the commands that follow.
2. Run the following commands in the Vyatta console.

Remember to:
* Choose `IKEv2` in authentication
* Enable `DH-group 2`
* Set `lifetime = 36000`
* Disable PFS
* Set `lifetime = 10800`  

The following commands use the following variables where:

* `{{ peer_address }}` is the VPN gateway public IP address.
* `{{ peer_cidr }}` is the VPN gateway subnet.
* `{{ vyatta_address }}` is the Vyatta public IP address.
* `{{ vyatta_cidr }}` is the Vyatta subnet.

```
vim vyatta_temp/create_vpn.vcli
#!/bin/vcli -f
configure

set security vpn ipsec ike-group {{ peer_address }}_{{ ike["name"] }}
set security vpn ipsec ike-group {{ peer_address }}_{{ ike["name"] }} dead-peer-detection timeout 120
set security vpn ipsec ike-group {{ peer_address }}_{{ ike["name"] }} lifetime {{ ike["lifetime"] }}
set security vpn ipsec ike-group {{ peer_address }}_{{ ike["name"] }} ike-version {{ ike["version"] }}

set security vpn ipsec ike-group {{ peer_address }}_{{ ike["name"] }} proposal {{ loop.index }}
set security vpn ipsec ike-group {{ peer_address }}_{{ ike["name"] }} proposal {{ loop.index }} dh-group {{ proposal["dhgroup"] }}
set security vpn ipsec ike-group {{ peer_address }}_{{ ike["name"] }} proposal {{ loop.index }} encryption {{ proposal["encryption"] }}
set security vpn ipsec ike-group {{ peer_address }}_{{ ike["name"] }} proposal {{ loop.index }} hash {{ proposal["integrity"] }}


set security vpn ipsec esp-group {{ peer_address }}_{{ ipsec["name"] }} compression disable
set security vpn ipsec esp-group {{ peer_address }}_{{ ipsec["name"] }} lifetime {{ ipsec["lifetime"] }}
set security vpn ipsec esp-group {{ peer_address }}_{{ ipsec["name"] }} mode tunnel
set security vpn ipsec esp-group {{ peer_address }}_{{ ipsec["name"] }} pfs {{ ipsec["proposals"][0]["dhgroup"] }}

set security vpn ipsec esp-group {{ peer_address }}_{{ ipsec["name"] }} proposal {{ loop.index }} encryption {{ proposal["encryption"] }}
set security vpn ipsec esp-group {{ peer_address }}_{{ ipsec["name"] }} proposal {{ loop.index }} hash {{ proposal["integrity"] }}

set security vpn ipsec site-to-site peer {{ peer_address }} authentication mode pre-shared-secret
set security vpn ipsec site-to-site peer {{ peer_address }} authentication pre-shared-secret {{ psk }}
set security vpn ipsec site-to-site peer {{ peer_address }} ike-group {{ peer_address }}_{{ ike["name"] }}
set security vpn ipsec site-to-site peer {{ peer_address }} default-esp-group {{ peer_address }}_{{ ipsec["name"] }}
set security vpn ipsec site-to-site peer {{ peer_address }} description "automation test"
set security vpn ipsec site-to-site peer {{ peer_address }} local-address {{ vyatta_address }}
set security vpn ipsec site-to-site peer {{ peer_address }} connection-type {{ connection_type }}
set security vpn ipsec site-to-site peer {{ peer_address }} authentication remote-id {{ peer_address }}

set security vpn ipsec site-to-site peer {{ peer_address }} tunnel {{ ns.tunnel_index }} local prefix {{ vyatta_cidr }}
set security vpn ipsec site-to-site peer {{ peer_address }} tunnel {{ ns.tunnel_index }} remote prefix {{ peer_cidr }}

commit
end_configure
```
{: codeblock}

For example, you can run the following commands:

```
#!/bin/vcli -f
configure

set security vpn ipsec ike-group 169.61.247.167_test_ike
set security vpn ipsec ike-group 169.61.247.167_test_ike dead-peer-detection timeout 120
set security vpn ipsec ike-group 169.61.247.167_test_ike lifetime 36000
set security vpn ipsec ike-group 169.61.247.167_test_ike ike-version 2

set security vpn ipsec ike-group 169.61.247.167_test_ike proposal 1
set security vpn ipsec ike-group 169.61.247.167_test_ike proposal 1 dh-group 2
set security vpn ipsec ike-group 169.61.247.167_test_ike proposal 1 encryption aes256
set security vpn ipsec ike-group 169.61.247.167_test_ike proposal 1 hash sha2_256
set security vpn ipsec esp-group 169.61.247.167_test_ipsec compression disable
set security vpn ipsec esp-group 169.61.247.167_test_ipsec lifetime 10800
set security vpn ipsec esp-group 169.61.247.167_test_ipsec mode tunnel
set security vpn ipsec esp-group 169.61.247.167_test_ipsec pfs disable


set security vpn ipsec esp-group 169.61.247.167_test_ipsec proposal 1 encryption aes256
set security vpn ipsec esp-group 169.61.247.167_test_ipsec proposal 1 hash sha2_256
set security vpn ipsec site-to-site peer 169.61.247.167 authentication mode pre-shared-secret
set security vpn ipsec site-to-site peer 169.61.247.167 authentication pre-shared-secret ***YOUR-PSK***
set security vpn ipsec site-to-site peer 169.61.247.167 ike-group 169.61.247.167_test_ike
set security vpn ipsec site-to-site peer 169.61.247.167 default-esp-group 169.61.247.167_test_ipsec
set security vpn ipsec site-to-site peer 169.61.247.167 description "automation test"
set security vpn ipsec site-to-site peer 169.61.247.167 local-address 169.63.66.53
set security vpn ipsec site-to-site peer 169.61.247.167 connection-type initiate
set security vpn ipsec site-to-site peer 169.61.247.167 authentication remote-id 169.61.247.167


set security vpn ipsec site-to-site peer 169.61.247.167 tunnel 1 local prefix 10.65.15.104/29
set security vpn ipsec site-to-site peer 169.61.247.167 tunnel 1 remote prefix 10.240.0.0/24


commit
end_configure

```
{: codeblock}

Finally, make note of your `{{ psk }}` value, as you need it to set up the VPN connection in the next step.

## Configuring the VPN gateway
{: #configure-vpn-gateway}

To configure the VPN gateway side, add a new VPN connection with the following settings:

* `Peer gateway address` as your Vyatta public IP address
* `Preshared key` as the `{{ psk }}` value
* `Local subnets` as the VPN gateway subnet
* `Peer subnets` as your Vyatta subnet

![Creat VPN connection form](images/vpc-vpn-vy-connection.png)

## Checking the status of the secure connection
{: #vyatta-check-the-status-of-the-secure-connection}

You can check the status of your connection on the {{site.data.keyword.cloud_notm}} console. You can also perform a `ping` from site to site using the virtual server instances.

![Active connection status](images/vpc-vpn-vy-status.png)

## Troubleshooting
{: #troubleshooting-and-more-examples}

Remember to:

* if you enable CPP firewall on Vyatta, you have to configure the rules to allow traffic from IBM gateway. For example, if your CPP firewall name is `GATEWAY_CPP`, add these rules to the firewall:

   ```
   # set security firewall name GATEWAY_CPP rule 250 source address 169.61.247.167
   # set security firewall name GATEWAY_CPP rule 250 action accept
   ```
   {: codeblock}

* If you are applying the firewall to the interface, you have to permit the traffic from IBM VPC. For example:

   ```
   # set security firewall name to-vpc rule 20 destination address 10.240.0.0/24
   # set security firewall name to-vpc rule 20 action accept
   # set security firewall name from-vpc rule 20 source address 10.240.0.0/24
   # set security firewall name from-vpc rule 20 action accept
   # set interfaces bonding dp0bond0 vif 862 firewall out to-vpc
   # set interfaces bonding dp0bond0 vif 862 firewall in from-vpc
   ```
   {: codeblock}

 	You might need to add other rules according to your network requirement to allow other traffic.

* If you are using a zone firewall with IPsec, see [Setting up an IPsec tunnel that works with zone firewalls](/docs/virtual-router-appliance?topic=virtual-router-appliance-setting-up-an-ipsec-tunnel-that-works-with-zone-firewalls).
