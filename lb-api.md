---

copyright:
  years: 2018, 2020
lastupdated: "2020-04-02"

keywords:

subcollection: vpc-on-classic-network

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

# Load balancer API examples
{: #lbaas-apis-available}

To make API calls, you must use a REST client. For example, you can use a `curl` command to retrieve all existing load balancers:

```
curl -X GET "$api_endpoint/v1/load_balancers?version=$api_version&generation=2" -H "Authorization: $iam_token"
```
{: pre}

Use the APIs described in Table 1 for load balancers in your VPC environment. For the full specification, see the [VPC API reference](https://{DomainName}/apidocs/vpc#list-all-load-balancers).

| Description | API |
|-------------|-----|
| Creates and provisions a load balancer | POST /load_balancers |
| Retrieves all load balancers | GET /load_balancers |
| Retrieves a load balancer | GET /load_balancers/{id} |
| Deletes a load balancer | DELETE /load_balancers/{id} |
| Updates a load balancer | PATCH /load_balancers/{id} |
| Creates a listener | POST /load_balancers/{id}/listeners |
| Retrieves all listeners of the load balancer | GET /load_balancers/{id}/listeners |
| Retrieves a listener | GET /load_balancers/{id}/listeners/{listener_id} |
| Deletes a listener | DELETE /load_balancers/{id}/listeners/{listener_id} |
| Updates a listener | PATCH /load_balancers/{id}/listeners/{listener_id} |
| Creates a pool | POST /load_balancers/{id}/pools |
| Retrieves all pools of the load balancer | GET /load_balancers/{id}/pools |
| Retrieves a pool | GET /load_balancers/{id}/pools/{pool_id} |
| Deletes a pool | DELETE /load_balancers/{id}/pools/{pool_id} |
| Updates a pool | PATCH /load_balancers/{id}/pools/{pool_id} |
| Creates a member | POST /load_balancers/{id}/pools/{pool_id}/members |
| Retrieves all members of the pool | GET /load_balancers/{id}/pools/{pool_id}/members |
| Retrieves a member |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Deletes a member from the pool | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Updates a member | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Updates members of the pool | PUT /load_balancers/{id}/pools/{pool_id}/members |
| Retrieves statistics of a load balancer | GET /load_balancers/{id}/statistics |
| Retrieves all policies of the listener |  GET /load_balancers/{id}/listeners/{listener_id}/policies
| Creates a policy for the listener | POST /load_balancers/{id}/listeners/{listener_id}/policies
| Deletes a policy from the listener | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Retrieves a policy of the listener | GET /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Updates a policy of the listener | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{id}
| Retrieves all the rules associated with a policy | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| Creates a rule for the policy | POST /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules
| Deletes a rule from the policy | DELETE /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| Retrieves a rule from the policy | GET /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}
| Updates a rule of the policy | PATCH /load_balancers/{id}/listeners/{listener_id}/policies/{policy_id}/rules/{rule_id}

## Load balancer example
{: #load-balancer-example}

In the following example, you'll use the API to create a load balancer in front of two VPC server instances (`192.168.100.5` and `192.168.100.6`) running a web application listening on port `80`. The load balancer has a front-end listener, which allows access to the web application securely by means of HTTPS. Then you can use the API to get details of the load balancer instance after it is created, and to delete the load balancer instance.

### Example steps
{: #lbaas-example-steps}

The example steps that follow skip the prerequisite steps of using the [IBM Cloud UI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console), [CLI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli), or [VPC API](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-rest-apis) to provision a VPC, subnets, and instances.

The load balancer example steps also can be run using the [CLI](/docs/vpc-on-classic?topic=vpc-on-classic-vpc-reference#load-balancer).
{: note}

**Step 1. Create a load balancer with listener, pool, and attached server instances (pool members)**

```bash
curl -H "Authorization: $iam_token" -X POST
"$rias_endpoint/v1/load_balancers?version=2019-05-31&generation=1" \
    -d '{
        "name": "example-balancer",
        "is_public": true,
        "listeners": [
            {
                "certificate_instance": {
                    "crn": "crn:v1:staging:public:cloudcerts:us-south:a/123456:b8877ea4-b8eg-467e-912a-da1eb7f031cg:certificate:43219c4c97d013fb2a95b21dddde1234"
                },
                "port": 443,
                "protocol": "https",
                "default_pool": {
                    "name": "example-pool"
                }
            }
        ],
        "pools": [
            {
                "algorithm": "round_robin",
                "health_monitor": {
                    "delay": 5,
                    "max_retries": 2,
                    "timeout": 2,
                    "type": "http",
                    "url_path": "/"
                },
                "name": "example-pool",
                "protocol": "http",
                "session_persistence": {
                    "cookie_name": "string",
                    "type": "source_ip"
                },
                "members": [
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.5"
                        },
                        "weight": 50
                    },
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.6"
                        },
                        "weight": 50
                    }
                ]
            }
        ],
        "subnets": [
            {
                "id": "7ec87131-1c7e-4990-b4f0-a26f2e61f98e"
            }
        ]
        }'
```
{: codeblock}

Sample output:
```
{
    "created_at": "2018-07-12T23:17:07.5985381Z",
    "crn": "crn:v1:staging:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "hostname": "ac34687d.lb.appdomain.cloud",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "is_public": true,
    "listeners": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
        }
    ],
    "name": "example-balancer",
    "operating_status": "offline",
    "pools": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
            "name": "example-pool"
        }
    ],
    "provisioning_status": "create_pending",
    "resource_group": {
        "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
    },
    "subnets": [
        {
            "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "name": "example-subnet"
        }
    ]
}
```
{: screen}

Save the ID of the load balancer to use in the next steps, for example, save it in the variable `lbid`.

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

**Step 2. Get a load balancer**

```
curl -H "Authorization: $iam_token" -X GET "$rias_endpoint/v1/load_balancers/$lbid?version=2019-05-31&generation=1"
```
{: pre}

Allow some time for provisioning. When the load balancer is ready, it will be set to `online` and `active` status, as you'll see in the following sample output:

Sample output:

```bash
{
  "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "crn": "crn:v1:staging:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "name": "example-balancer",
  "created_at": "2018-07-13T22:22:24.489Z",
  "hostname": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud",
  "is_public": true,
  "listeners": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
    }
  ],
  "operating_status": "online",
  "pools": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
      "name": "example-pool"
    }
  ],
  "private_ips": [
    {
      "address": "192.168.10.5"
    },
    {
      "address": "192.168.10.6"
    }
  ],
  "provisioning_status": "active",
  "public_ips": [
    {
        "address": "169.11.111.115"
    },
    {
        "address": "169.11.111.116"
    }
  ],
  "resource_group": {
    "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
  },
  "subnets": [
    {
      "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "name": "example-subnet"
    }
  ]
}
```
{: screen}

**Step 3. Delete the load balancer**

```bash
curl -H "Authorization: $iam_token" -X DELETE "$api_endpoint/v1/load_balancers/$lbid?version=$api_version&generation=1"
```
{: pre}
