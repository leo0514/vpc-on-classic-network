---

copyright:
  years: 2017, 2020
lastupdated: "2020-02-28"

keywords: peering, Vyatta, connection, secure, remote, vpc, vpc network

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

These examples are based on the Vyatta version: AT&T vRouter 5600 1801d
{: note}

## Architecture
{: #vyatta-architecture}

The topology for connecting to a remote Vyatta peer is similar to creating a VPN connection between two {{site.data.keyword.cloud_notm}} VPCs with one side replaced by the Vyatta.

![enter image description here](images/vpc-vpn-vy-figure.png)

## Before you begin
{: #vyatta-preparation}

To set up your remote Vyatta peer, make sure that the following prerequisites are met.

* A VPC
* A subnet in the VPC
* A VPN gateway in the VPC without connections

   After the VPN gateway gets provisioned, note its public IP address.
   {: tip}
* The Vyatta public IP address
* The Vyatta subnet that you want to connect using a VPN

## Configuring the Vyatta 
{: #configure-vyatta-peer}

There are two ways you can run the configuration on your Vyatta:

1. Log in to the Vyatta and run the file `create_vpn.vcli` using the commands that follow.
2. Run the following commands in the Vyatta console. 

Remember to:
* Choose `IKEv2` in authentication
* Enable `DH-group 2` 
* Set `lifetime = 36000` 
* Disable PFS 
* Set `lifetime = 18000`  

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

## Example
Commands appear similar to the following:

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
set security vpn ipsec esp-group 169.61.247.167_test_ipsec lifetime 18000
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
{: screen}

Finally, make note of your `{{ psk }}` value, as you need it to set up the VPN connection in the next step.

### Troubleshooting and more examples  
{: #troubleshooting-and-more-examples}

Remember to:
* Config your ACL to allow port 500 and 4500

Allow IKE and ESP traffic for IPsec:

```
# set rule 100 action 'accept' 
# set rule 100 destination port '500'
# set rule 100 protocol 'udp'
# set rule 200 action 'accept'
# set rule 200 protocol 'esp'
```

Allow L2TP over IPsec:

```
# set rule 210 action 'accept'
# set rule 210 destination port '1701'
# set rule 210 ipsec 'match-ipsec'
# set rule 210 protocol 'udp'
```

Allow NAT traversal of IPsec:

```
# set rule 250 action 'accept'
# set rule 250 destination port '4500'
# set rule 250 protocol 'udp'
```

#### For beginners, we provide more Vyatta examples here. You can modify these examples for your own deployment.

Filtering on source IP:

```
vyatta@R1# show security firewall name FWTEST-1
rule 1 {
	action accept
	source {
		address 172.16.0.26
	}
}
vyatta@R1# show interfaces dataplane dp0p1p1
address 172.16.1.1/24
	firewall FWTEST-1 {
	in {
	}
}
```

Filtering on source and destination IP:

```
vyatta@R1# show security firewall name FWTEST-2
rule 1 {
	action accept
	destination {
		address 10.10.40.101
	}
	source {
		address 10.10.30.46
	}
}
vyatta@R1# show interfaces dataplane dp0p1p2
vif 40 {
	firewall {
	out FWTEST-2
	}
}
```

Filtering traffic between zones example:

```
vyatta@R1# show security zone-policy

zone dmz {
	description DMZ
	interface dp0p1p3
	to private {
		firewall to_private
	}
	to public {
		firewall to_public
	}
}
zone private {
	description PRIVATE
	interface dp0p1p1
	interface dp0p1p2
	to dmz {
		firewall to_dmz
	}
	to public {
		firewall to_public
	}
}
zone public {
	description PUBLIC
	interface dp0p1p4
	to dmz{
		firewall to_dmz
	}
	to private {
		firewall to_private
	}
}
```


### Configuring the VPN gateway
{: #configure-vpn-gateway}

To configure the VPN gateway, add a new VPN connection with the following settings:

* `Peer gateway address` as your Vyatta public IP address
* `Preshared key` as the `{{ psk }}` value
* `Local subnets` as the VPN gateway subnet
* `Peer subnets` as your Vyatta subnet

![Creat VPN connection form](images/vpc-vpn-vy-connection.png)

### Checking the status of the secure connection
{: #vyatta-check-the-status-of-the-secure-connection}

You can check the status of your connection on the {{site.data.keyword.cloud_notm}} console. You can also perform a `ping` from site-to-site using the virtual server instances.

![Active connection status](images/vpc-vpn-vy-status.png)
