---

copyright:
  years: 2018, 2022
lastupdated: "2022-06-15"

keywords: listener, pool, round-robin, weighted, layer 7, datapath logging, http2, websocket

subcollection: vpc

---

{{site.data.keyword.attribute-definition-list}}

# About {{site.data.keyword.cloud_notm}} {{site.data.keyword.alb_full}}
{: #load-balancers}

Use {{site.data.keyword.cloud}} {{site.data.keyword.alb_full}} (ALB) to distribute traffic among multiple server instances within the same region of your VPC.
{: shortdesc}

The following diagram illustrates the deployment architecture for the ALB.

![Application load balancer for VPC](images/alb_arc.png "Application load balancer"){: caption="Application load balancer" caption-side="bottom"}

In this diagram, "Client Resources" represents the resources (VPCs and subnets, for instance) that belong to the client ecosystem.
{: note}

## Types of application load balancers
{: #types-load-balancer}

You can create a public or private ALB. Table 1 shows a comparison of public versus private features.

| Feature | Public load balancer | Private load balancer |
|--------|-------|-------|
| Accessible on internet? |  Yes, with a fully qualified domain name (FQDN) | No, internal clients only, on same region and VPC |
| Accepts all traffic? | Yes | Yes  \n (The restriction to accept traffic only from the RFC-1918 address space has been removed) |
| How is domain name registered? | Public IP addresses | Private IP addresses |
{: caption="Table 1. Comparison of public and private load balancers" caption-side="bottom"}

### Public application load balancer
{: #public-load-balancer}

A public application load balancer service instance is assigned a publicly accessible fully qualified domain name (FQDN), which you must use to access your applications that are hosted behind the load balancer. This domain name can be registered with one or more public IP addresses.

Over time, the number and value of these public IP addresses might change due to maintenance and scaling activities. The back-end virtual server instances hosting your application must run in the same region and under the same VPC.

Use the assigned FQDN to send traffic to the public application load balancer to avoid connectivity problems to your applications during system maintenance or scaling down activities.
{: important}

### Private application load balancer
{: #private-load-balancer}

A private application load balancer is accessible through your private subnets that you configured to create the load balancer.

Similar to a public application load balancer, your private application load balancer service instance is assigned an FQDN. However, this domain name is registered with one or more private IP addresses.

{{site.data.keyword.cloud_notm}} operations might change the number and value of your assigned private IP addresses over time, based on maintenance and scaling activities. The back-end virtual server instances hosting your application must run in the same region, and under the same VPC.

Use the assigned FQDN to send traffic to the private application load balancer to avoid connectivity problems to your applications during system maintenance or scaling down activities.
{: important}

## Load-balancing methods
{: #load-balancing-methods}

Three load-balancing methods are available for distributing traffic across the back-end application servers:

### Round-robin
{: #round-robin}

Round-robin is the default load-balancing method. With this method, an application load balancer forwards incoming client connections in round-robin fashion to the back-end servers. As a result, all back-end servers receive roughly an equal number of client connections.

### Weighted round-robin
{: #weighted-round-robin}

With this method, an application load balancer forwards incoming client connections to the back-end servers in proportion to the weight assigned to these servers. Each server is assigned a default weight of `50`, which can be customized to any value in the range `0` - `100`.

For example, if application servers A, B and C have the weights `60`, `60`, and `30`, then servers A and B receive an equal number of connections, while server C receives half that number of connections.

Setting a server weight to `0` means that no new connections are forwarded to that server, but any existing traffic continues to flow. Using a weight of `0` can help bring down a server gracefully and remove it from service rotation.
{: tip}

The server weight values are applicable only with the weighted round-robin method. They are ignored with round-robin and least-connections load-balancing methods.

### Least connections
{: #least-connections}

With this method, the back-end server instance that serves the least number of connections at a given time receives the next client connection.

## Front-end listeners and back-end pools
{: #front-end-listeners-and-back-end-pools}

Front-end listeners are load balancer application ports for receiving incoming requests, while back-end pools are the application servers behind the load balancers.

### HTTPS redirect listener
{: #https-redirect-listener}

HTTPS redirect listeners redirect the traffic from an HTTP listener to an HTTPS listener. This does not require any rules applied on the listener.

For instance, if a service listens on port 443 with HTTPS and a user tries to access the service on port 80 using HTTP, then the request automatically redirects to port 443 with HTTPS.

If policies are present on the HTTPS redirect listener, then the policies are evaluated first. If thre are no policy matches, then the request redirects to a configured HTTPS listener.

### HTTPS redirect listener properties
{: #https_redirect_listener_properties}

Property  | Description
------------- | -------------
Listener | The HTTPS listener to which a request redirects. 
HTTP status code | Status code of the response returned by the application load balancer. The acceptable values are: `301`, `302`, `303`, `307`, or `308`.
URI | The relative URI to which a request redirects. This is an optional property.
{: caption="Table 2. HTTPS redirect listener properties" caption-side="bottom"}

### Guidelines for using listeners
{: #listener-guidelines}

* You can define up to 10 front-end listeners and map them to back-end pools on the back-end application servers. 
* The FQDN assigned to your load balancer and the front-end listener ports are exposed to the public internet. Incoming user requests are received on these ports.
* The supported front-end listener and back-end pool protocols are HTTP, HTTPS, and TCP. 
* You can configure an HTTP/HTTPS front-end listener with an HTTP/HTTPS back-end pool. 
* HTTP/2 is supported for listeners only. 
* HTTP and HTTPS listeners and pools are interchangeable. 
* You can only configure a TCP front-end listener with a TCP back-end pool.
* You can attach up to 50 virtual server instances to a back-end pool. Traffic is sent to each instance on its specified data port. This data port does not need to be the same one as the front-end listener port.
* "Private only" endpoints for Certificate Manager are not supported with HTTPS listeners. To configure an HTTPS listener in an ALB, you must upload your TLS certificates to a "Public and private" endpoint.

Effective 31st July 2021, application load balancers will convert all HTTP request and response headers to lower case. This applies to all traffic handled by HTTP and HTTPS listeners. This is being done to provide support for HTTP2, but will be applied to older HTTP 1.x versions as well for uniformity. For most applications this will not require changes, but any application that depends on the case sensitivity of headers needs to be updated to handle them irrespective of capitalization. This is in accordance to networking standards (RFC 2616 Sec 4.2 and RFC 7230 Sec 3.2) which state that HTTP headers are case insensitive.
{: important}

## Elasticity
{: #alb-elasticity}

The application load balancer scales out by adding compute resources when load increases.

## SSL offloading and required authorizations
{: #ssl-offloading-and-required-authorizations}

SSL offloading allows the application load balancer service to terminate all incoming HTTPS connections.

When an HTTPS listener is configured with an HTTP pool, the HTTPS request is terminated at the front-end and the load balancer establishes a plain-text HTTP communication with the back-end server instance. With this technique, CPU-intensive SSL handshakes and encryption or decryption tasks are shifted away from the back-end server instances, allowing them to use all their CPU cycles for processing application traffic.

SSL offloading requires you to provide an SSL certificate for the application load balancer to perform SSL offloading tasks. You can manage the SSL certificates through the [{{site.data.keyword.secrets-manager_full_notm}}](/docs/secrets-manager?topic=secrets-manager-getting-started). 

Application load balancer will continue to support [{{site.data.keyword.cloudcerts_full_notm}}](/docs/certificate-manager?topic=certificate-manager-getting-started) until September 30 2022. Please see the [deprecation announcement](/docs/certificate-manager?topic=certificate-manager-getting-started) for more information. In order to update the certificates hosted in {{site.data.keyword.cloudcerts_full_notm}} with ones hosted in Secrets Manager, the listener configuration must be updated with the new certificate CRN.
{: deprecated}

{{site.data.content.load-balancer-grant-service-auth}} 

To prevent errors, you must establish the required authorization between your load balancer and {{site.data.keyword.secrets-manager_full_notm}}.
{: important}

Transport Layer Security (TLS) 1.2 and 1.3 are supported. However, TLS 1.3 is used by default unless you specifically configure the client side to utilize 1.2. Application load balancers honor all supported TLS 1.3 ciphers sent by the client-side request.
{: note}

The following lists the supported ciphers (in order of precedence):

* `TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384`
* `TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256`
* `TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256`
* `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384`
* `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`
* `TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256`

### Locating the certificate CRN 
{: #locating-alb-crn}

When configuring authentication for an application load balancer during provisioning using the UI, you can choose to specify the certificate manager and SSL certificate, or the certificate's CRN. You might want to do this if you cannot view the certificate manager in the drop-down menu, which means you don't have access to the certificate manager instance. Keep in mind that you must enter the CRN if using the API to create an ALB.

To obtain the CRN, you must have permission to access the certificate manager instance.
{: note}

To find a certificate's CRN, follow these steps:

1. In the [{{site.data.keyword.cloud_notm}} console](https://{DomainName}/vpc-ext){: external}, go to **Menu icon ![Menu icon](../icons/icon_hamburger.svg) > Resource list**. 
1. Click to expand **Services and software**, then select the certificate manager that you want to find the CRN for.
1. Select anywhere in the table row of the certificate to open the Certificate details side panel. The certificate CRN is listed. 

   ![Service instance CRN](images/vpn-crn.png "Service instance CRN"){: caption="Service instance CRN" caption-side="bottom"}

## End-to-end SSL encryption
{: #end-to-end-ssl-encryption}

Configuring an HTTPS listener with an HTTPS pool enables end-to-end SSL encryption. The application load balancer service terminates the incoming HTTPS request at the front-end listener and establishes an HTTPS connection with the back-end instances. End-to-end encryption allows all traffic going through the load balancer to the back-end members to be encrypted over HTTPS.

To configure end-to-end SSL encryption:

1. Configure an HTTPS front-end listener with your SSL certificate as you would when configuring SSL offloading.
2. Configure an HTTPS back-end pool.
3. Add your back-end member instance to the HTTPS back-end pool. Ensure that your back-end member instances are configured to handle HTTPS traffic.
4. Configure health check with type HTTPS to perform encrypted health checks with your back-end members.

An application load balancer does not verify the SSL certificates of the back-end member instances.
{: important}

## Horizontal scaling
{: #horizontal-scaling}

An application load balancer adjusts its capacity automatically according to the load. When this adjustment occurs, you might see a change in the number of IP addresses associated with the load balancer's DNS name.

## MZR support
{: #mzr-support}

{{site.data.keyword.cloud_notm}} Application Load Balancer for VPC supports Multi-Zone-Regions (MZRs). You can achieve high availability and redundancy by deploying an application load balancer with subnets from different zones. When subnets from multiple zones are used to provision an application load balancer, the load balancer appliances get deployed to multiple zones.

## Integration with instance groups
{: #lbaas-integration-with-instance-groups-overview}

{{site.data.keyword.cloud_notm}} Application Load Balancer for VPC integrates with instance groups, which can `auto scale` your back-end members. Pool members are dynamically added and deleted based on your usage and requirements.

## Datapath log forwarding
{: #datapath-log-forwarding}

With datapath logging enabled, load balancer logs are forwarded to the [{{site.data.keyword.la_full_notm}}](https://cloud.ibm.com/catalog/services/ibm-log-analysis){: external} service, where you can view your datapath logs.

## HTTP2 support
{: #http2-support}
Application load balancers support end-to-end HTTP2 traffic, and works with listener protocols set as either HTTPS or TCP.

## WebSocket support
{: #websocket-support}
WebSocket provides full-duplex communication channels over a single TCP connection. Application load balancers support WebSocket with every type of listener protocol (HTTP/HTTPS/TCP). 

## Related links
{: #permissions-related-links-alb}

* [Load balancer CLI reference](/docs/vpc?topic=vpc-infrastructure-cli-plugin-vpc-reference#alb-anchor)
* [Load balancer API reference](https://{DomainName}/apidocs/vpc#list-load-balancer-profiles)
* [ALB for VPC infrastructure resources for Terraform](https://registry.terraform.io/providers/IBM-Cloud/ibm/latest/docs/data-sources/is_lb){: external} (VPC infrastructure > Resources)
* [Required permissions for VPC resources](/docs/vpc?topic=vpc-resource-authorizations-required-for-api-and-cli-calls)
* [{{site.data.keyword.cloudaccesstraillong_notm}} events](/docs/vpc?topic=vpc-at-events#events-load-balancers)
* [FAQs for application load balancers](/docs/vpc?topic=vpc-load-balancer-faqs)
* [Quotas](/docs/vpc?topic=vpc-quotas#load-balancer-quotas)
* [VPC SOC 3 report on security and availability](https://www.ibm.com/downloads/cas/ZVYQK9N5){: external}
