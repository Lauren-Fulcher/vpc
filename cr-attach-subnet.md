---

copyright:
  years: 2020, 2021
lastupdated: "2021-03-02"

keywords: custom routes

subcollection: vpc

---

{{site.data.keyword.attribute-definition-list}}

# Attaching subnets to a routing table
{: #attach-subnets-routing-table}

When you view details of a routing table, you can also view details of attached subnets, or attach a subnet to apply routing table routes to a particular subnet. Note that a routing table can be associated with multiple subnets. However, each subnet can only have one routing table assigned to it.
By default, any subnet not associated with a routing table is associated with the default routing table. You can also reassign the routing table of a subnet to another subnet.
{: shortdesc}

You can attach a subnet to a routing table, or reassign a routing table to a particular subnet by using the UI, CLI, or API.

## Attaching subnets to a routing table using the UI
{: #cr-attach-subnet-ui}
{: ui}

To attach a routing table to a subnet by using the {{site.data.keyword.cloud_notm}} console, follow these steps:

1. From the [{{site.data.keyword.cloud_notm}} console](https://{DomainName}/vpc-ext){: external}, select the Menu icon ![Menu icon](/images/menu_icon.png), then click **VPC Infrastructure > Routing tables** in the Network section. The Routing tables for VPC page appear.
1. Click the name of the routing table in which you want to view subnet details. Alternatively, you can click the number of attached subnets.  
1. From the Subnets tab, click **Attach subnet**. The Attach subnets to routing table side panel shows.
1. Select a subnet from the drop-down list. The current routing table, IP range, and location of the subnet shows.
1. Click **Attach**.   

   ![Attaching subnets to routing table side panel](/images/attach-subnet-routing-table.png "Attaching subnets to routing table side panel"){: caption="Figure 1. Attaching subnets to routing table side panel" caption-side="bottom"}

To reassign a routing table to a subnet, follow these steps:

1. From the routing table details page, click the Subnets tab.
1. Click the overflow menu ![overflow menu](images/overflow.png) next to the subnet, then click **Reassign routing table**.
1. From the Reassign subnet routing table side panel, click the subnet that you want to assign to this routing table.
1. Click **Reassign**.

   ![Reassign routing table side panel](/images/reassign-routing-table.png "Reassign routing table side panel"){: caption="Figure 2. Reassign routing table side panel" caption-side="bottom"}
   
## Attaching subnets to a routing table by using the CLI
{: #cr-attach-subnets-using-the-cli}
{: cli}

Before you begin, make sure to [set up your CLI environment](/docs/vpc?topic=vpc-infrastructure-cli-plugin-vpc-reference).

To attach subnets to a routing table by using the CLI, run the following command:

```sh
ibmcloud is subnet-update SUBNET_ID --routing-table-id ROUTING_TABLE_ID
```
{: pre}

Where:
* **SUBNET_ID** - Is the ID of the subnet you want to update.  
* **ROUTING_TABLE_ID** - Is the ID of the routing table that you want to assign the subnet to. 


## Attaching subnets to a routing table using the API
{: #cr-attach-subnets-using-the-api}
{: api}

To attach subnets to a routing table by using the API, follow these steps:

1. Set up your [API environment](/docs/vpc?topic=vpc-set-up-environment#api-prerequisites-setup).
1. Store the `VpcId` value in a variable to be used in the API command:

    ```sh
    export VpcId=<your_vpc_id>
    ```
    {: pre}

1. Attach a subnet to a routing table:

    ```sh
    curl -X PUT "$vpc_api_endpoint/v1/subnets/$subnet_id/routing_table?version=$api_version&generation=2"
    -H "Authorization: $iam_token"
    -d '{
      "id": "37491549-51c0-4acc-9e7b-9ab3628e1a68"
        }'
    ```
    {: codeblock}
