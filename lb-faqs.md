---

copyright:
  years: 2018, 2021
lastupdated: "2021-05-25"


keywords: load balancer, public, listener, back-end, front-end, pool, round-robin, weighted, connections, methods, policies, APIs, access, ports, vpc network, layer 7, auto scale, managed pool, instance group

subcollection: vpc

---

{{site.data.keyword.attribute-definition-list}}

# FAQs for {{site.data.keyword.cloud_notm}} {{site.data.keyword.alb_full}}
{: #load-balancer-faqs}

The following sections contain answers to some frequently asked questions about the {{site.data.keyword.cloud}} {{site.data.keyword.alb_full}} (ALB).

## Can I use a different DNS name for my ALB?
{: #different-dns-name}
{: faq}

The auto-assigned DNS name for the application load balancer is not customizable. However, you can add a CNAME (Canonical Name) record that points your preferred DNS name to the auto-assigned load balancer DNS name. For example, your load balancer in `us-south` has ID `dd754295-e9e0-4c9d-bf6c-58fbc59e5727`, and the auto-assigned load balancer DNS name is `dd754295-us-south.lb.appdomain.cloud`. Your preferred DNS name is `www.myapp.com`. You can add a CNAME record (through the DNS provider that you use to manage `myapp.com`) pointing `www.myapp.com` to the load balancer DNS name `dd754295-us-south.lb.appdomain.cloud`.

## What's the maximum number of front-end listeners I can define with my application load balancer?
{: #max-number-listeners}
{: faq}

10 is the maximum number of front-end listeners that you can define with your ALB.

## What's the maximum number of virtual server instances I can attach to my back-end pool?
{: #server-instances-back-end-pool-alb}
{: faq}

50 is the maximum number of virtual server instances that you can attach to a back-end pool.

## What's the maximum number of subnets I can attach to my application load balancer?
{: #max-number-subnets-alb}
{: faq}

15 is the maximum number of subnets that you can define with your ALB.

## Is the load balancer horizontally scalable?
{: #horizontally-scalable}
{: faq}

Yes. The {{site.data.keyword.alb_full}} automatically adjusts its capacity based on the load. When horizontal scaling takes place, the number of IP addresses associated with the application load balancer's DNS changes.

## What do I do if I use ACLs on the subnets that are used to deploy the application load balancer?
{: #using-acls-on-subnets}

Make sure that the proper ACL rules are in place to allow incoming traffic for configured listener ports. Traffic between the application load balancer and back-end instances need also be allowed.

## Why am I receiving an error message: `certificate instance not found`?
{: #error-certificate-instance}
{: faq}

* The Secrets Manager instance CRN might not be valid.
* You probably don't have authorization granted service-to-service. See [SSL offloading](#ssl-offloading-and-required-authorizations) for details.

## Why am I receiving a `401 Unauthorized Error` code?
{: #alb-401-unauthorized-error}
{: faq}

Check the following access policies for your user:

* The access policy for the ALB resource type
* The access policy for the resource group
* If `HTTPS` listeners are used, also check the service-to-service authorization for the Secrets Manager instance.

## Why is my application load balancer in `maintenance_pending` state?
{: #lb-maintenance-pending}
{: faq}

The application load balancer is in `maintenance_pending` state during various maintenance activities, such as:

* Horizontal scaling activities
* Recovery activities
* Rolling upgrades to address vulnerabilities and apply security patches

## Why do I need to choose multiple subnets during provisioning?
{: #subnets-during-provisioning}
{: faq}

The {{site.data.keyword.alb_full}} (ALB) is Multi-Zone Region (MZR) ready. Load balancer appliances are deployed to the subnets you selected. To achieve higher availability and redundancy, deploy the application load balancer to subnets in different zones.

## Do I need extra IPs in the subnet for application load balancer operations?
{: #additional-ips-subnet}
{: faq}

It is recommended to allocate 8 extra IPs per MZR to accommodate horizontal scaling and maintenance operations. If you provision your application load balancer with one subnet, allocate 16 extra IPs.

## Why is back-end member health under my pool `unknown`?
{: #member-health-unknown}
{: faq}

Either the pool is not associated with any listeners, or configuration changes were made to the pool or its associated listener.

## Why is back-end member health under my pool `faulted`?
{: #member-health-failing}
{: faq}

Verify the following configurations:

* Does the port of the configured back-end protocol match the port that the application is listening on?
* Does the configured health-check have the correct protocol (TCP or HTTP), port, and URL (for HTTP) information? For HTTP, make sure that your application responds with `200 OK` for the configured health check URL.
* Is the back-end server a virtual server with an enabled security group? If so, make sure that the security group rules allow traffic between the load balancer and the virtual server.

For more information, see[Health Checks](#health-checks).

## What are the default settings and allowed values for health check parameters?
{: #alb-health-check-parameters}
{: faq}

* Health check interval - Default is 5 seconds, and the range is 2 - 60 seconds.
* Health check response timeout - Default is 2 seconds, and the range is 1 - 59 seconds.
* Maximum retry attempts - Default is two retry attempts, and the range is 1-10 retries.

The health check response timeout value must be less than the health check interval value.
{: tip}

## Are the ALB IP addresses fixed?
{: #ip-addresses-fixed}
{: faq}

Application load balancer IP addresses are not guaranteed to be fixed. During system maintenance or horizontal scaling, you see changes in the available IPs associated with the FQDN of your load balancer.

Use FQDN, rather than cached IP addresses.
{: important}

## Does the load balancer support layer 7 switching?
{: #layer-7-switching-alb}
{: faq}

Yes, the load balancer supports layer 7 switching.

## Why does HTTPS listener creation or update tell me that my certificate is invalid?
{: #why-does-https-listener-creation-or-update-tell-me-that-my-certificate-is-invalid}
{: faq}

Check for these possibilities:

* The provided certificate CRN might not be valid.
* The certificate instance in the Secrets Manager might not have an associated private key.

### What is the role of application load balancer front-end listeners?
{: #role-load-balancer-listeners}
{: faq}

Load balancer front-end listeners are the listening ports for the application. They act as proxies for back-end pools.

### Why are there only 2 IPs instead of 3?
{: #why-only-2-ips}
{: faq}

The {{site.data.keyword.alb_full}} (ALB) operates in ACTIVE-ACTIVE mode, a configuration that makes it highly available. Horizontal scaling might further add extra appliances when your load increases. The recommendation is that you choose subnets in different zones to make your load balancers support MZR. This way, if a zone is negatively impacted, a new load balancer is provisioned in a different zone.

### Why am I receiving a 409 Conflict Error code when I update or delete a load balancer?
{: #409-lbaas-conflict-error}
{: faq}

If you are receiving the HTTP status code `409`, it might be for one of the following reasons:
* You cannot `update` the load balancer configuration if the load balancer status is not `ACTIVE`.
* You cannot `delete` a load balancer if the load balancer status is not `ACTIVE` or `ERROR`.
* You cannot `delete` a load balancer that is already in `DELETE_PENDING` state.
* You cannot `delete` a load balancer if it has one or more pools that are attached to instance groups.
* You cannot `delete` a load balancer pool if it is attached to an instance group.
* You cannot `add`/`delete`/`update` members to a pool that is managed by an instance group.


### If a pool is attached to an instance group, what is the maximum number of back-end members that I can have in a pool?
{: #lbaas-ig-pool-member}
{: faq}

The maximum number of back-end members that are allowed in a pool is 50. So if an instance group is attached to a pool, the number of instances in the group can't scale up beyond this limit.

### Why is my listener not receiving traffic?
{: #lbaas-listener-security-group}
{: faq}

Make sure that the security group rules that are attached to your load balancer allow incoming ingress and outgoing egress traffic on your listener's port. Security groups attached to your load balancer can be found on your load balancer's overview page. Locate the `Attached security groups` tab from the load balancer overview, then select the security groups that you want to view and modify their rules.

### Why is traffic not reaching my back-end members?
{: #lbaas-member-security-group}
{: faq}

Ensure that:

* Your back-end pool health checks are succeeding
* Your back-end member application is up and running
* Your load balancer security group rules allow outgoing traffic from load balancers on your back-end member port
* Your back-end member's security group rules, if any, allow all incoming traffic from the load balancer

## Does IBM complete quarterly ASV scans of data-plane LBaaS appliances?
{: #alb-asv}
{: faq}

Approved Scanning Vendor (ASV) quarterly scanning is a requirement of the Payment Card Industry (PCI) Security Standards Council. ASV scanning of LBaaS data-plane appliances is solely a customer responsibility. IBM does not use ASVs to scan data-plane appliances because these scans can negatively impact customer workload functions and performance.

## How can I migrate from Certificate Manager to Secrets Manager for my existing HTTPS listeners?
{: #migrate-to-secrets-manager}
{: faq}

Application load balancer will continue to support [IBM Certificate Manager](/docs/certificate-manager?topic=certificate-manager-getting-started) until September 30 2022. To migrate your existing certificates to Secrets Manager, refer to the information in [this topic](https://cloud.ibm.com/docs/secrets-manager?topic=secrets-manager-migrate-from-certificate-manager). After your certificates are migrated, you must then update the existing listener configuration with the new certificate CRN.

## How are active connections handled when a load balancer is scaled down?
{: #faqs-active-connections}
{: faq}

When a load balancer appliance undergoes a scale down due to horizontal scaling or maintenance, the service waits for the active connections to close to allow for traffic to move to other appliances. After 24 hours, the service will complete its scale down event, which may terminate any active connections on those scaled down appliances.
