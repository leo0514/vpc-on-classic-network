---


copyright:
  years: 2018, 2020
lastupdated: "2020-05-12"

keywords:

subcollection: vpc-on-classic-network

---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:note: .note}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}
{:external: target="_blank" .external}

# Using load balancers for VPC
{: #--using-load-balancers-in-ibm-cloud-vpc}

The {{site.data.keyword.cloud}} Load Balancer for VPC service distributes traffic among multiple server instances within the same region of your VPC.

## Public load balancer
{: #public-load-balancer}

Your load balancer service instance is assigned a publicly accessible, fully qualified domain name (FQDN), which you must use for access to your applications that are hosted behind the IBM Cloud Load Balancer for VPC. This domain name can be registered with one or more public IP addresses.

Over time, these public IP addresses and the number of public IP addresses might change, due to maintenance and scaling activities. The back-end server instances (VSIs) hosting your application must run in the same region, and under the same VPC.

Remember to use the assigned FQDN to send traffic to the public load balancer to avoid connectivity problems to your applications during system maintenance or scaling down activities.
{: important}

## Private load balancer
{: #private-load-balancer}

The private load balancer is accessible only to internal clients on your private subnets, within the same region and VPC. The private load balancer accepts traffic only from [RFC1918](https://tools.ietf.org/html/rfc1918){: external} address spaces.

Similar to a public load balancer, your private load balancer service instance is assigned a FQDN. However, this domain name is registered with one or more private IP addresses.

IBM Cloud operations might change the number and value of your assigned private IP addresses over time, based on maintenance and scaling activities. The back-end server instances (VSIs) hosting your application must run in the same region, and under the same VPC.

See the following table for a comparison of features:

| Feature | Public Load Balancer | Private Load Balancer |
|--------|-------|-------|
| Accessible on internet? |  Yes, with FQDN, internet | No, internal clients only, on same region and VPC |
| Accepts all traffic? | Yes | No, RFC 1918 only |
| Number of private IPs | changes over time | changes over time |
| How is domain name registered? | public IP addresses | private IP addresses |

Remember to use the assigned FQDN to send traffic to the private load balancer to avoid connectivity problems to your applications during system maintenance or scaling down activities.
{: important}

## Front-end listeners and back-end pools
{: #front-end-listeners-and-back-end-pools}

You can define up to ten front-end listeners (application ports) and map them to respective back-end pools on the back-end application servers. The FQDN assigned to your load balancer service instance and the front-end listener ports are exposed to the public internet. Incoming user requests are received on these ports.

The supported front-end listener protocols are:
* HTTP
* HTTPS
* TCP

The supported back-end pool protocols are:
* HTTP
* HTTPS
* TCP

You can configure an HTTP/HTTPS protocol front-end listener with an HTTP/HTTPS back-end pool. HTTP and HTTPS protocol listeners and pools are interchangeable. A TCP protocol front-end listener can only be configured with a TCP protocol back-end pool.

You can attach up to fifty server instances to a back-end pool. Each attached server instance must have a port configured. The port might or might not be the same as the front-end listener port.

The port range of `56500` to `56520` is reserved for management purposes; these ports cannot be used as front-end listener ports.
{: note}

## SSL offloading and required authorizations
{: #ssl-offloading-and-required-authorizations}

SSL offloading allows the load balancer service to terminate all incoming HTTPS connections.

When an HTTPS listener is configured with an HTTP pool, the HTTPS request is terminated at the front-end and the load balancer establishes a plain-text HTTP communication with the back-end server instance. With this technique, CPU-intensive SSL handshakes and encryption or decryption tasks are shifted away from the back-end server instances, allowing them to use all their CPU cycles for processing application traffic.

SSL offloading requires you to provide an SSL certificate for the load balancer to perform SSL offloading tasks. You can manage the SSL certificates through the [IBM Certificate Manager](/docs/certificate-manager?topic=certificate-manager-getting-started).

To give a load balancer access to your SSL certificate, you must enable **service-to-service authorization**, which grants your load balancer service instance access to your certificate manager instance. For more information, see [Granting access between services](/docs/iam?topic=iam-serviceauth#create-auth). Make sure to choose **Infrastructure Service** as the source service, **Load Balancer for VPC** as the resource type, **Certificate Manager** as the target service, and assign the **Writer** service access role.

The required authorization between the load balancer and certificate manager must be set to prevent errors in your load balancer.
{: important}

Only TLS 1.2 is supported. The following is a list of the supported ciphers (listed in order of precedence):

* TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
* TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
* TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256
* TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
* TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
* TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256

## End-to-End SSL encryption
{: #end-to-end-ssl-encryption}

Configuring an HTTPS listener with an HTTPS pool enables end-to-end SSL encryption. The load balancer service terminates the incoming HTTPS request at the front-end listener and establishes an HTTPS connection with the back-end instances. End-to-end encryption allows all traffic going through the load balancer to the back-end members to be encrypted over HTTPS protocol.

To configure end-to-end SSL encryption:

1. Configure an HTTPS protocol front-end listener with your desired SSL certificate as you would when configuring SSL offloading.
2. Configure an HTTPS protocol back-end pool.
3. Add your back-end member instance to the HTTPS protocol back-end pool. Your back-end member instances should be configured to handle HTTPS traffic.
4. Configure health checks with type HTTPS to perform encrypted health checks with your back-end members.

The load balancer does not verify the SSL certificates of the back-end member instances.
{: important}

## Load balancing methods
{: #load-balancing-methods}

The following three load balancing methods are available for distributing traffic among back-end application servers:

* **Round-robin:** Round-robin is the default load balancing method. With this method, the load balancer forwards
incoming client connections in round-robin fashion to the back-end servers. As a result, all back-end servers receive
roughly an equal number of client connections.

* **Weighted round-robin:** With this method, the load balancer forwards incoming client connections to the back-end
servers in proportion to the weight assigned to these servers. Each server is assigned a default weight of 50,
which can be customized to any value between 0 and 100.

As an example, if three application servers A, B and C, have weights customized to 60, 60, and 30 respectively, then servers A and B receive an equal number of connections, while server C receives half that number of connections.

* **Least connections:** With this method, the server instance serving the least number of connections at a given time receives the next client connection.

**Additional characteristics of these methods:**

* Re-setting a server weight to '0' means no new connections are forwarded to that server, but any existing traffic continues to flow. Using a weight of '0' can help bring down a server gracefully and remove it from service rotation.
* The server weight values are applicable only with the weighted round-robin method. They are ignored with round-robin and least connections load balancing methods.

## Horizontal scaling
{: #horizontal-scaling}

The load balancer adjusts its capacity automatically according to the load. When this adjustment occurs, you might see a change in the number of IP addresses associated with the load balancer's DNS name.

## Health checks
{: #health-checks}

Health check definitions are mandatory for back-end pools.

The load balancer conducts periodic health checks to monitor the health of the back-end ports and it forwards client traffic to them accordingly. If a given back-end server port is found to be unhealthy, no new connections are forwarded to it. The load balancer continues to monitor health of unhealthy ports, and it resumes their use if they become healthy again, which means that they successfully pass two consecutive health check attempts.

The health checks for HTTP, HTTPS, and TCP ports are conducted as follows:

* **HTTP:** An `HTTP GET` request against a pre-specified URL is sent to the health check port. The server port is marked healthy upon receiving a `200 OK` response. The default `GET` health path is "/".

* **HTTPS:** Similar to an HTTP health check, an `HTTPS GET` request is sent to the configured health check port and URL path. Health checks will utilize HTTPS protocol to encrypt traffic to back-end servers. A back-end server is deemed healthy upon receiving a `200 OK` response. The default `GET` health path is "/".

* **TCP:** The load balancer attempts to open a TCP connection with the back-end server on a specified TCP port. The server port is marked healthy if the connection attempt is successful, and the connection is closed.

The default health check interval is 5 seconds, the default timeout against a health check request is 2 seconds, and the default number of retrial attempts is 2.
{: note}

## Configuring ACLs for use with load balancers
{: #configuring-acls-for-use-with-load-balancers}

If you use access control lists (ACLs) to block traffic on the subnets where you deploy your load balancer, make sure you configure the ACL rules listed below.

| Inbound/Outbound| Protocol | Source IP | Source Port | Destination IP | Destination Port |
|--------------|------|------|------|------|------------------|
| Inbound | UDP/TCP |161.26.0.0/16 | 53, 80, 443, 1688, 8443| AnyIP | AnyPort|
| Inbound |TCP| AnyIP | AnyPort| AnyIP | 56501|
| Inbound |TCP| AnyIP | 443, 10514, 8834| AnyIP | AnyPort|
| Outbound | UDP/TCP | AnyIP | AnyPort| 161.26.0.0/16 | 53, 80, 443, 1688, 8443|
| Outbound | TCP | AnyIP | 56501| AnyIP | AnyPort|
| Outbound | TCP | AnyIP | AnyPort| AnyIP |443, 10514, 8834|

Additionally, if you configure your load balancer listeners, then you must also configure the corresponding inbound and outbound ACL rules for load balancer traffic to pass through.

## Configuring load balancers from the IBM Cloud Kubernetes Service (IKS)
{: #configure-iks-lb}
When you create a VPC Kubernetes cluster in {{site.data.keyword.cloud_notm}}, a VPC load balancer gets provisioned into your account with TCP listeners. This load balancer can be identified with a name prefix `kube-`. It is recommended that you perform any Kubernetes Application Load Balancer (ALB) configuration updates on the VPC load balancer from the IKS interface, **not** from the load balancer GUI page or by using load balancer APIs or CLI. For more information, see [IKS documentation](/docs/containers?topic=containers-loadbalancer-qs).

Deleting the IKS load balancer or updating the load balancer configuration from VPC load balancer interfaces makes the VPC IKS cluster in error or in an unwanted state.
{:note}
