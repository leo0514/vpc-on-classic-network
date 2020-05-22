---

copyright:
  years:  2020
lastupdated: "2020-05-21"

keywords: Sysdig, IBM Cloud, monitoring, platform metrics, metrics, VPN metrics

subcollection: vpc_vpn

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:codeblock: .codeblock}
{:tip: .tip}
{:download: .download}
{:important: .important}
{:note: .note}

# Sysdig monitoring metrics
{: #sysdig-monitoring-metrics}

{{site.data.keyword.mon_full_notm}} is operated by Sysdig in partnership with {{site.data.keyword.IBM_notm}} and collects basic VPN metrics on {{site.data.keyword.cloud_notm}} for VPC, such as VPN gateway status, VPN gateway packets input/output, and VPN connection bytes input/output. These metrics are stored in Sysdig. If you have a Sysdig account, then metrics are displayed for that Sysdig instance. You can access metrics through the prebuilt dashboard.
{:shortdesc}

## Platform metrics overview
{: platform-metrics-overview}

You can view platform metrics when you enable {{site.data.keyword.mon_full_notm}} on your {{site.data.keyword.cloud_notm}} platform. A Sysdig instance must be configured in a region to monitor these metrics. For more information, see [Enabling platform metrics](/docs/Monitoring-with-Sysdig?topic=Sysdig-platform_metrics_enabling).

Before you enable {{site.data.keyword.mon_full_notm}} on your platform, keep the following information in mind:

* You can configure only one instance of the {{site.data.keyword.mon_full_notm}} service per region to collect platform metrics.
* Platform metrics are regional. Metrics are monitored only from {{site.data.keyword.mon_full_notm}} services, which are in the same region of the instance that you want to monitor.
* Metrics are collected automatically and are available for monitoring through the {{site.data.keyword.mon_full_notm}}-enabled instance.

For more information about {{site.data.keyword.mon_full_notm}} platform metrics, see [Sysdig monitoring metrics](/docs/virtual-servers?topic=virtual-servers-sysdig-monitoring-metrics).

## Metrics available by service plan
{: metrics-by-plan}

| Metric name |
|-----------|
| [VPN connection bytes received](#ibm_is_vpn_connection_bytes_in) |
| [VPN connection bytes sent](#ibm_is_vpn_connection_bytes_out) |
| [VPN connection packets received](#ibm_is_vpn_connection_packets_in) |
| [VPN connection packets sent](#ibm_is_vpn_connection_packets_out) |
| [VPN connection status](#ibm_is_vpn_connection_status) |
| [VPN gateway bytes received](#ibm_is_vpn_gateway_bytes_in) |
| [VPN gateway bytes sent](#ibm_is_vpn_gateway_bytes_out) |
| [VPN gateway packets received](#ibm_is_vpn_gateway_packets_in) |
| [VPN gateway packets sent](#ibm_is_vpn_gateway_packets_out) |
| [VPN gateway status](#ibm_is_vpn_gateway_status) |
{: caption="Table 1: Metrics available by plan names" caption-side="top"}

## VPN metric definitions
{: metric-definitions}

The following tables define the basic VPN metrics on {{site.data.keyword.cloud_notm}} for VPC.

### VPN gateway bytes received
{: #ibm_is_vpn_gateway_bytes_in}

Bytes per minute received for a given VPN gateway

| Metadata | Description |
|----------|-------------|
| `Metric name` | `ibm_is_vpn_gateway_bytes_in`|
| `Metric type` | `gauge` |
| `Value type`  | `byte` |
| `Segment by` | `Service instance, Service instance name, VPN name, IBM IS Generation 1` |
{: caption="Table 7: VPN gateway bytes received" caption-side="top"}

### VPN gateway bytes sent
{: #ibm_is_vpn_gateway_bytes_out}

Bytes per minute sent for a given VPN gateway

| Metadata | Description |
|----------|-------------|
| `Metric name` | `ibm_is_vpn_gateway_bytes_out`|
| `Metric type` | `gauge` |
| `Value type`  | `byte` |
| `Segment by` | `Service instance, Service instance name, VPN name, IBM IS Generation 1` |
{: caption="Table 8: VPN gateway bytes sent" caption-side="top"}

### VPN gateway packets received
{: #ibm_is_vpn_gateway_packets_in}

Packets per minute received for a given VPN gateway

| Metadata | Description |
|----------|-------------|
| `Metric name` | `ibm_is_vpn_gateway_packets_in`|
| `Metric type` | `gauge` |
| `Value type`  | `none` |
| `Segment by` | `Service instance, Service instance name, VPN name, IBM IS Generation 1` |
{: caption="Table 9: VPN gateway packets received" caption-side="top"}

### VPN gateway packets sent
{: #ibm_is_vpn_gateway_packets_out}

Packets per minute sent for a given VPN gateway

| Metadata | Description |
|----------|-------------|
| `Metric name` | `ibm_is_vpn_gateway_packets_out`|
| `Metric type` | `gauge` |
| `Value type`  | `none` |
| `Segment by` | `Service instance, Service instance name, VPN name, IBM IS Generation 1` |
{: caption="Table 10: VPN gateway packets sent" caption-side="top"}

### VPN gateway status
{: #ibm_is_vpn_gateway_status}

Status for a given VPN gateway (for example, `1`=available, `0`=unavailable)

| Metadata | Description |
|----------|-------------|
| `Metric name` | `ibm_is_vpn_gateway_status`|
| `Metric type` | `gauge` |
| `Value type`  | `none` |
| `Segment by` | `Service instance, Service instance name, VPN name, IBM IS Generation 1` |
{: caption="Table 11: VPN gateway status" caption-side="top"}

### VPN connection bytes received
{: #ibm_is_vpn_connection_bytes_in}

Bytes per minute received for a given VPN gateway's connection

| Metadata | Description |
|----------|-------------|
| `Metric name` | `ibm_is_vpn_connection_bytes_in`|
| `Metric type` | `gauge` |
| `Value type`  | `byte` |
| `Segment by` | `Service instance, VPN name, Connection name, Connection ID, IBM IS Generation 1` |
{: caption="Table 2: VPN connection bytes received" caption-side="top"}

### VPN connection bytes sent
{: #ibm_is_vpn_connection_bytes_out}

Bytes per minute sent for a given VPN gateway's connection

| Metadata | Description |
|----------|-------------|
| `Metric name` | `ibm_is_vpn_connection_bytes_out`|
| `Metric type` | `gauge` |
| `Value type`  | `byte` |
| `Segment by` | `Service instance, VPN name, Connection name, Connection ID, IBM IS Generation 1` |
{: caption="Table 3: VPN connection bytes sent" caption-side="top"}

### VPN connection packets received
{: #ibm_is_vpn_connection_packets_in}

Packets per minute received for a given VPN gateway's connection

| Metadata | Description |
|----------|-------------|
| `Metric name` | `ibm_is_vpn_connection_packets_in`|
| `Metric type` | `gauge` |
| `Value type`  | `none` |
| `Segment by` | `Service instance, VPN Name, Connection Name, Connection ID, IBM IS Generation 1` |
{: caption="Table 4: VPN connection packets received" caption-side="top"}

### VPN connection packets output
{: #ibm_is_vpn_connection_packets_out}

Packets per minute sent for a given VPN gateway's connection

| Metadata | Description |
|----------|-------------|
| `Metric name` | `ibm_is_vpn_connection_packets_out`|
| `Metric type` | `gauge` |
| `Value type`  | `none` |
| `Segment by` | `Service instance, VPN name, Connection name, Connection ID, IBM IS Generation 1` |
{: caption="Table 5: VPN connection packets sent" caption-side="top"}

### VPN connection status
{: #ibm_is_vpn_connection_status}

Status for a given VPN gateway's connection (for example, `1`=up, `0`=down)

| Metadata | Description |
|----------|-------------|
| `Metric name` | `ibm_is_vpn_connection_status`|
| `Metric type` | `gauge` |
| `Value type`  | `none` |
| `Segment by` | `Service instance, VPN name, Connection name, Connection ID, IBM IS Generation 1` |
{: caption="Table 6: VPN connection status" caption-side="top"}

## Attributes for segmentation
{: attributes}

### Global attributes
{: global-attributes}

The following attributes are available for segmenting all of the VPN metrics:

| Attribute | Attribute name | Attribute description |
|-----------|----------------|-----------------------|
| `Cloud type` | `ibm_ctype` | A value of public, dedicated or local |
| `Location` | `ibm_location` | The location of the monitored resource. This is a region, data center, or global |
| `Resource` | `ibm_resource` | The resource that is measured by the service - typically an identifying name or GUID |
| `Resource type` | `ibm_resource_type` | The type of resource that is measured by the service |
| `Resource group` | `ibm_resource_group_name` | The resource group where the service instance was created |
| `Scope` | `ibm_scope` | The scope of the account, organization, or space GUID that is associated with this metric |
| `Service name` | `ibm_service_name` | Name of the service that generated this metric |
{: caption="Table 12: VPN metric attributes" caption-side="top"}

### Additional attributes
{: additional-attributes}

The following attributes are available for segmenting one or more attributes as described in the previous reference. See the individual metrics for segmentation options.

| Attribute | Attribute name | Attribute description |
|-----------|----------------|-----------------------|
| `Connection ID` | `ibm_is_vpn_connection_id` | IBM VPN for VPC gateway connection ID |
| `Connection name` | `ibm_is_vpn_connection_name` | IBM VPN for VPC gateway connection name |
| `IBM IS generation 1` | `ibm_is_generation` | IBM IS Generation; for example, 1  |
| `Service instance` | `ibm_service_instance` | Identifies the instance that the metric is associated with |
| `Service instance name` | `ibm_service_instance_name` | Provides the user-provided name of the service instance. This name isn't necessarily a unique value that depends on the name that is provided. |
| `VPN gateway name` | `ibm_is_vpn_gateway_name` | IBM VPN for VPC gateway name |
{: caption="Table 13: VPN segmentation metric attributes" caption-side="top"}
