---

copyright:
  years: 2024
lastupdated: "2024-1-16"

keywords:

subcollection: vpc

---

{{site.data.keyword.attribute-definition-list}}

# Publishing a Private Path service
{: #pps-activating}

The beta release of IBM Cloud Private Path services is only available to allowlisted users. Contact your IBM Support representative if you are interested in getting early access to this beta offering.
{: beta}

After you [verify successful connectivity to your Private Path service](/docs/vpc?topic=vpc-pps-verify), it's time to publish your service for deployment to consumers.
{: shortdesc}

You can publish an {{site.data.keyword.cloud}} Private Path service using the UI, CLI, or API.

## Activating a Private Path service in the UI
{: #pps-ui-activating-private-path-service}
{: ui}

To publish a Private Path service with the {{site.data.keyword.cloud_notm}} console, follow these steps:

1. From your browser, open the [{{site.data.keyword.cloud_notm}} console](/login){: external} and log in to your account.
1. Select the Menu icon ![Menu icon](../icons/icon_hamburger.svg), then click **VPC Infrastructure**.
1. Click **Private Path services** in the Network section.
1. Locate the name of your new Private Path service in the table, then click **Publish** in the Actions menu ![Actions menu](images/overflow.png).

   The Published column in the Private Path services for VPC table changes to `Yes`.

After the service is published, it cannot be unpublished.
{: important}

The Private Path service is now exposed for other accounts to connect to the service through Virtual Private Endpoint (VPE) gateways.

## Activating a Private Path service from the CLI
{: #pps-cli-publish-private-path-service}
{: cli}

The following example shows how to use the CLI to publish a Private Path service.

Before you begin, make sure to [set up your CLI environment](/docs/vpc?topic=vpc-set-up-environment&interface=cli){: external}.

You must first export the feature flag to use the CLI for Private Path beta release offerings.
{: important}

To export the feature flag, enter the following commands:

```sh
export IBMCLOUD_IS_FEATURE_PRIVATE_PATH_SERVICE_GATEWAY=true
export IBMCLOUD_IS_FEATURE_PP_NLB_SUPPORT=true
```
{: pre}

To publish a Private Path service from the CLI, follow these steps:

1. Enter the following command:

```sh
ibmcloud is private-path-service-gateway-update PRIVATE_PATH_SERVICE_GATEWAY
    [--name NEW_NAME]
    [--default-access-policy | deny | permit | review]
    [--load-balancer LOAD_BALANCER]
    [--published]
    [--zonal-affinity false | true]
    [--output JSON] [-q, --quiet]
```
{: pre}

Where:

`PRIVATE_PATH_SERVICE_GATEWAY`
:   Indicates ID or name of the Private Path service.

`--name`
:   Indicates name of the Private Path service.

`--default-access-policy`
:   Indicates the policy to use for bindings from accounts without an explicit account policy. One of: `deny`, `permit`, `review`. (default: `deny`).

`--load-balancer`
:   Indicates ID or name of load balancer for this Private Path service.

`--published**`
:   Indicates the availability of this Private Path service. If passed, value is set to true (any account can request access to this Private Path service).
`--zonal-affinity`
:   Indicates whether this Private Path service has zonal affinity. One of: `false`, `true`.

`--output`
:   Indicates output format, only JSON is supported. One of: `JSON`.

`-q, --quiet`
:   Suppresses verbose output.

### Command examples
{: #cli-cmd-examples-activating-private-path-service}
{: cli}

- Publish a Private Path service:
   `ibmcloud is private-path-service-gateway-update r006-2e671f14-19fc-4089-9ad3-973176711259 --name cli-ppsg-2 --default-access-policy deny --load-balancer r006-d-740be75d-be41-48bd-b6e4-89946ddcac4a --zonal-affinity false --published`

- Publish and rename a Private Path service:
   `ibmcloud is private-path-service-gateway-update cli-ppsg-2 --name cli-ppsg-0 --default-access-policy review --load-balancer my-cli-nlb-1 --zonal-affinity false --published`

## Publishing a Private Path service with the API
{: #pps-api-publishing-private-path-service}
{: api}

To publish a Private Path service with the API, follow these steps:

1. Set up your [API environment](/docs/vpc?topic=vpc-set-up-environment&interface=cli).
1. Store the following values in variables to be used in the API command:

   * `ppsgId` - Get your Private Path service and then populate the variable:

      ```sh
      export ppsgId=<your_ppsg_id>
      ```
      {: pre}

1. When all variables are initiated, do one of the following:

   * To publish your Private Path service:

      ```sh
      curl -X PATCH -sH "Authorization:${iam_token}" \
      "$vpc_api_endpoint/v1/private_path_service_gateways/$ppsgId?version=$api_version&generation=2" \
      -d {
         "published": true
       }'
      ```
      {: codeblock}

## Next steps
{: #pps-next-steps-after-activation}

1. [Communicate connection information to consumers](/docs/vpc?topic=vpc-pps-ui-communicate)
1. [Review connection requests](/docs/vpc?topic=vpc-pps-ui-reviewing&interface=ui) and [Create account policies](/docs/vpc?topic=vpc-pps-create-account-policy&interface=ui)
