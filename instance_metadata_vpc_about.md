---

copyright:
  years: 2022
lastupdated: "2022-08-10"

keywords:

subcollection: vpc


---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:external: target="_blank" .external}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:preview: .preview}
{:table: .aria-labeledby="caption"}
{:DomainName: data-hd-keyref="APPDomain"}
{:DomainName: data-hd-keyref="DomainName"}

# About VPC Instance Metadata
{: #imd-about}

The VPC Instance Metadata service is free service that let's you access information about your virtual server instances. It provides a REST API you invoke within an instance to get information about that instance. Access to the API is unavailable from outside the instance. Before you can access the metadata, the service lets you generate an instance identity access token, for accessing the metadata service. You can optionally get an IAM token from this token to access all IAM-enabled services. 
{: shortdesc}

## Metadata service concepts
{: #imd-concepts}

You can programmatically access metadata about a virtual server instance and use it to initialize new instances. Metadata provided by API services pertains only to the instance from which the request is made. You can't use the metadata service within an instance to obtain information about another instance or to obtain information concerning resources not currently associated with the instance.

The instance metadata service uses a REST API and well-known IP address to retrieve metadata. You can enable the on all new instances you create from the VPC API, CLI, or UI. The metadata service is disabled by default. Metadata you can access includes the instance name, CRN, resource groups, user data, as well as information about SSH keys and placement groups. For a list of all metadata returned from the service see the [summary of data returned by the metadata service](/docs/vpc?topic=vpc-imd-metadata-summary).

The instance metadata service has two components:

* **[Instance identity token service](#imd-vpc-access-token)** that lets you aquire a JSON web token to access the VPC metadata service. To access other IBM Cloud IAM-enabled services, you can generate an IAM token from the instance identity token. For example, starting an instance might require a call to IBM Cloud Database to set up a database server.

* **[Instance metadata service](#imd-service)** that lets make API calls to retrieve the instance metadata. By calling the metadata service APIs, you can get instance initialization data, network interface, volume attachment, public SSH key, and placement group information.

### Limitations
{: imd-limitations}

The VPC Instance Metadata service is currently supported only on x86 systems.

### Instance identity token service
{: #imd-vpc-access-token}

The instance identity token service lets you generate an access token to use that token to access the metadata service. You generate it by calling this endpoint: `http://169.254.169.254/instance_identity/v1/token`. This endpoint is accessible to all commands, processes, and software applications running within a virtual server instance. Access to the API endpoint is not available outside the virtual server instance.

Use the instance identity access token when calling the metadata service. The token is valid until it reaches expiration, which by default is 5 minutes. For more information, see [Aquire an access token](/docs/vpc?topic=vpc-imd-configure-service#imd-json-token).

You can also generate an IAM token from the instance identity access token. This IAM token can be used to access all IAM-enabled services. It has it's own expiration date, based on the IAM token service default. [Generate an IAM token from an instance identity access token](/docs/vpc?topic=vpc-imd-configure-service&interface=api#imd-token-exchange).

### Metadata service
{: #imd-service}

You can enable the metadata service on all new instances to retrieve data about the instance. Use this information to configure and launch new instances.

Metadata is obtained by calling REST APIs that provide instance-specific information. You make `GET` calls to endpoints accessible within a running virtual server instance. You can retrieve metadata about instances, keys, and placement groups by making calls invoke a well-known, non-routable IP address:

* `http://169.254.169.254/metadata/v1/instance`
* `http://169.254.169.254/metadata/v1/keys`
* `http://169.254.169.254/metadata/v1/placement_groups`

For a list of all endpoints you can use to access instance metadata, see [Summary of data returned by the metadata service](/docs/vpc?topic=vpc-imd-metadata-summary).

You [enable the metadata service](/docs/vpc?topic=vpc-imd-get-metadata#imd-metadata-service-enable) by setting a toggle in the VPC UI, CLI, or API when creating a new instance or updating an existing instance.

The metadata service intercepts all requests to the service's IP and then routes them to the specific services to handle these requests. As part of the request to the metadata service, you include the instance identity access token.

#### Compute resource identities
{: #imd-comp-res-identity}

Through IAM, you can also assign access rights to instances by creating a [compute resource identity](/docs/vpc?topic=vpc-imd-trusted-profile-metadata), and then configure access rights to IBM-enabled services using that identity.

The compute resource identity service creates a trusted profile, against which you assign access rights to enable the instance to call IAM-enabled services, such as Cloud Object Storage and Key Protect. You create a trusted profile within the virtual server instance. Trusted profiles define authorization for all applications running on the instance.

#### User data
{: #imd-user-data}

The metadata you access from the instance includes _user data_. This is data you specified when provisioning the instance that is available when provisioning new instances. (It's the same user data available from cloud-init for VPC instances.) For example, information to load database software, custom software for worker nodes, or information to make decisions about how to initialize the instance are provided in user data. For more information, see this topic on [User data](/docs/vpc?topic=vpc-user-data).

The metadata you access from within the instance is not protected by any encryption method. Any user with access to the instance and metadata service can potentially see the metadata. As a precaution, do not store sensitive data, such as passwords or customer encryption keys, as user data.
{: important}

## Scenarios for using the metadata service
{: #imd-metadata-scenarios}

You can use the metadata service in two ways:

* Access the metadata from within the instance

    In this scenario, you retrieve metadata from within the instance to bootstrap the instance. For example, you might want to specify custom user data when creating the instance and then retrieve that custom user data when initializing the instance. For more information, see [Accessing metadata from an instance](/docs/vpc?topic=vpc-imd-access-instance-metadata).

* Access an instance identity IAM token and call IAM-enabled services

    In this scenario, you create a trusted profile for compute resource identity and assign a virtual server instance access rights for IAM-enabled services. Make a call to create a new instance, configured with the metadata service, then link the instance to a trusted profile. Call the metadata service to get an access token, then [generate an IAM token](/docs/vpc?topic=vpc-imd-configure-service&interface=api#imd-token-exchange) from that token. You can then call IAM-enabled services from an instance. Use this option when you want to call IAM-enabled services as part of instance bootstrapping. For example, you might want to set up a new COS bucket to be used by the instance workload. For more information, see [Using a trusted profile to call IAM-enabled services](/docs/vpc?topic=vpc-imd-trusted-profile-metadata).

Figure 1 illustrates these scenarios.

![Figure showing the instance metadata user workflows.](/images/metadata-service-user-workflow.png "Figure showing the instance metadata user workflows."){: caption="Figure 1. User scenarios for the instance metadata service" caption-side="bottom"}

## Data security
{: #imd-security}

IBM takes precaution to assure your data is secure. Metadata is retrieved only from the instance to which you have access. Data common to multiple instances will be retrieved but specific IP addresses of other instances are not exposed.

Calls to the metadata service are secure prior to leaving the compute host from which they originate. While access is over HTTP, data is wrapped using a secure protocol prior to leaving the compute host.

The initial call to the instance metadata service is over an HTTP connection. The purpose of this call is to acquire an identity token to access the metadata service. These services use a well-known URL. Although you see a non-secure connection, the VPC compute service takes additional actions to secure the token and metadata services.

The instance metadata service is not intended for sensitive data. The service end point is open to all processes on the instance. Information exposed through this service should be considered as shared information to all applications running inside the virtual server instance.
{: note}

For additional security measures you can take, see [Security best practices for the Instance Metadata service](/docs/vpc?topic=vpc-imd-security-best-practices).

## Next steps
{: #imd-next-steps-about}

* [Create an instance identity access token for accessing the metadata service](/docs/vpc?topic=vpc-imd-configure-service#imd-get-token).

* [Retrieve data using the metadata service](/docs/vpc?topic=vpc-imd-get-metadata).
