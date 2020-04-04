---

copyright:
  years: 2018, 2020
lastupdated: "2020-04-02"

keywords: load balancer, public, listener, back-end, front-end, pool, round-robin, weighted, connections, methods, policies, APIs, access, ports, vpc, vpc network, layer-7

subcollection: vpc

---

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

# Advanced traffic management
{: #advanced-traffic-management}

The following advanced traffic management features are available.

## Max connections
{: #max-connections}

Configure the number of maximum connections to limit the number of concurrent connections for a given listener. By default, no limit is configured.

## Session persistence
{: #session-persistence}

The load balancer supports session persistence based on the source IP of the connection. If you have `source IP` type session persistence enabled for port 80 (HTTP), then subsequent HTTP connections from the same source IP are persistent on the same back-end server. This feature is available for all three supported protocols (HTTP, HTTPS and TCP).

## HTTP keep alive
{: #http-keep-alive}

The load balancer supports `HTTP keep alive` as long as it is enabled on both the client and back-end servers. The load balancer attempts to reuse the server-side HTTP connections to increase connection efficiency and reduce latency.

## Connection timeouts
{: #connection-timeouts}

The following timeout values are used by the load balancer. You cannot customize these values at this time.

| Name | Description | Timeout |                                                                              
| ------------------------------------------ | --------------------------------------------------- | ------------------- |
| Server-side connection attempt    | The maximum time that the load balancer will wait to establish a TCP connection with the back-end server. If the connection attempt is unsuccessful after this time interval, the load balancer tries the next available server, according to the load-balancing method configured. | 5 seconds   |
| Client-side idle connection  | The maximum time the load balancer will wait on an idle client connection before terminating the connection. | 50 seconds  |
| Server-side idle connection | The maximum time the load balancer will wait on an idle server connection before terminating the connection. With the back-end protocol configuration of HTTP, if the load balancer fails to receive a response to its HTTP request within the idle timeout window, it will return an error message to the client.  | 50 seconds |
{: caption="Table 1. Load Balancer timeout values" caption-side="top"}

## Preserving end-client IP address
{: #preserving-end-client-ip-address}

Load Balancer for VPC works as a reverse proxy which terminates incoming traffic from the client. The load balancer establishes a separate connection to the back-end server instance, using its own IP address. For HTTP connections with the backend servers (against front-end HTTP or HTTPS connections), the load balancer preserves the original client IP address by including it inside the `X-Forwarded-For` HTTP header. For TCP connections, the original client IP information is not preserved.
