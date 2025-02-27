---

copyright:
  years: 2021, 2022
lastupdated: "2022-05-19"

keywords:

subcollection: vpc

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:important: .important}
{:screen: .screen}
{:pre: .pre}
{:tip: .tip}
{:preview: .preview}
{:table: .aria-labeledby="caption"}
{:note: .note}

# About File Storage for VPC
{: #file-storage-vpc-about}

{{site.data.keyword.cloud}} File Storage for {{site.data.keyword.vpc_full}} (VPC) is a zonal file storage offering that provides NFS-based file storage services. You create file shares in an availability zone within a region. You can share them with multiple virtual server instances within the same zone across multiple VPCs.
{: shortdesc}

File Storage for VPC is available for customers with special approval to preview this service in the Washington, Dallas, Frankfurt, London, Sydney, Sao Paulo, and Tokyo regions. Contact your IBM Sales representative if you are interested in getting access.
{: preview}

## Overview
{: #file-storage-overview}

File Storage for VPC provides file shares within the bounds of a VPC. You create a file share in a single availability zone. You then a create mount target to mount the file share. You can also set up replication between the source file share and a replica file share, and failover to the replica file share in case of outage at the primary site. For more information, see [About file share replication](/docs/vpc?topic=vpc-file-storage-replication).

You pay for only the capacity you need. File share capacity ranges from 10 GB up to 32,000 GB for all available profiles. You can [increase capacity](/docs/vpc?topic=vpc-file-storage-expand-capacity) of an existing file share and [adjust IOPS](/docs/vpc?topic=vpc-adjusting-share-iops) up or down to meet your performance needs. Billing is adjusted accordingly.

File share data is encrypted by default with IBM-managed encryption for data-at-rest. For added security, you can also use your own root keys to protect your file shares. For more information, see [File share encryption](#FS-encryption). 

You can also add user tags to your file shares when creating a new share or updating an existing share. Add user tags when creating new file files or to existing file shares with the UI, CLI, or API. For more information, see [User tags for file shares](#fs-about-user-tags).

File Storage for VPC is integrated with the Security and Compliance Center to help you manage security and compliance for your organization. For more information, see [Managing security and compliance](/docs/vpc?topic=vpc-file-storage-managing#fs-vpc-manage-security).

### File storage IOPS profiles
{: #fs-profiles-intro}

You have two options for creating file shares, by using an IOPS tier profile or by using a custom IOPS profile.

IOPS tiers provide a guaranteed level of performance for your workloads. You can select from three tiers:

* 3 IOPS/GB,
* 5 IOPS/GB,
* 10 IOPS/GB.

With a Custom IOPS profile you can select the performance you want, by selecting custom allocated I/O operations per second (IOPS).

For more information about these options, see [File Storage for VPC profiles](/docs/vpc?topic=vpc-file-storage-profiles#fs-tiers).

### Zonal file shares
{: #fs-zonal-file-shares}

File Storage for VPC lets you create a file share at the zonal level. This means that file shares are accessible only within the zone in which you created it, for example, `us-south-1`. File shares are identified by name and associated with a resource group in your {{site.data.keyword.cloud_notm}} customer account.

You create a file share using the UI, CLI, or API. You access file shares from virtual server instances or Kubernetes clusters by way of an NFS mount. To create an NFS mount path, you need to create mount targets.

You can [increase the file share size](/docs/vpc?topic=vpc-file-storage-expand-capacity) from its original capacity in GB increments up to 32,000 GB capacity, depending on your file share profile. You can also [increase or decrease file share IOPS](/docs/vpc?topic=vpc-adjusting-share-iops) to meet your performance needs. Adjust IOPs by specifying a different IOPS tier profile or different IOPS value withing a custom IOPS band. There's no outage or lack of access to the storage while increasing capacity or adjusting IOPS.

### Mount targets for file shares
{: #fs-share-mount-targets}

A mount target for a file share is a network endpoint or path. When you create a mount target, an NFS mount path is created for the file share. You use the mount path to attach the file share to virtual server instances or Kubernetes clusters in the same VPC and zone.

To mount a file share to an instance with the API, you create a mount target by providing a VPC or subnet information. If you want to connect a file share to instances that are running in multiple VPCs in the same zone, you can create multiple mount targets for different VPCs. After creating the mount target (or targets), you can SSH into the virtual server instance and attach the file share.

### NFS version
{: #fs-nfs-version}

File Storage for VPC requires NFS versions v4.1 or higher.

## Limitations in this release
{: #fs-limitations}

The following limitations apply to this release of File Storage for VPC:

* File share size cannot be increased after it is created.
* File share profile cannot be changed after the file share is created.
* Granular Host Authorization for VSI level access control in not supported.
* Minimum capacity is 10 GB per file share.
* Maximum capacity is 32,000 GB per file share.
* There is no data retention policy for deleted file shares. You cannot undelete a file share after you delete it.

## File share encryption
{: #FS-encryption}

By default, file share data are encrypted at rest with IBM-managed encryption.

You can bring your own customer root key (CRK) to the cloud for customer-managed encryption or you can have a key management service (KMS) generate a key for you.

You can select this key when you [create a new encrypted file share](/docs/vpc?topic=vpc-file-storage-vpc-encryption). For more information about this encryption option, see [Customer-managed encryption](/docs/vpc?topic=vpc-vpc-encryption-about#vpc-customer-managed-encryption).

After you specified an encryption type for a file share, you can't change it.

## File share replication and failover
{: #fs-repl-failover-overview}

You can create replicas of your file shares by setting up a replication relationship between primary file shares in one zone to replica file shares in another zone. Using replication is a good way to recover from incidents at the primary site when data becomes inaccessible or applications fail. [Failover](/docs/vpc?topic=vpc-file-storage-failover) to the replica share makes it the new, writeable primary share. For more information, see [About file share replication](/docs/vpc?topic=vpc-file-storage-replication).

## Supplemental IDs and Groups for file shares
{: #FS-supplemental-ids}

When a process runs on Unix/Linux, the operating system identifies a user with a user ID (UID) and group with a group ID (GID). These IDs determine which system resources a user or group can access. For example, if the target file storage user ID is 12345 and its group ID is 6789, then the mount on the host node and in a container must have those same IDs. The container’s main process must match one or both of those IDs to access the file share.

with the API, you can set these attributes for controlling access to your file shares when you create a file share. The API provides an `initial owner` property where you can set the `UID` and `GID` values. Wherever you mount the file share, the root folder where you mount it uses that UID or GID owner. For more information, see [Add supplemental IDs when you create a file share](/docs/vpc?topic=vpc-file-storage-create&interface=api#fs-add-supplemental-id-api).

## User tags for file shares
{: #fs-about-user-tags}

File storage for VPC is enabled for Global Searching and Tagging (GhoST). From the file service you can create user tags to organize your file shares across the VPC. You can create new user tags or add existing tags when provisioning a new file share or updating an existing file share. You can sreate, view, and manage tags from the UI or API, and remove then at any time.

Tags are uniquely identified by a Cloud Resource Name (CRN) identifier. When you create a tag, you provide a unique name within your billing account. You can define tags in label or key-value format. Behind the scenes, the file service sends and receives tags directly to the GhoST service. GhoST stores its key attributes and the array of tags. GhoST also stores user resource information, which let's you view, tag, and search for resources you own.

For information about adding tags to file shares, see [Add user tags to file shares](/docs/vpc?topic=vpc-file-storage-managing&interface=ui#fs-add-user-tags). For more information about managing tags for your account, see [Working with tags](/docs/account?topic=account-tag&interface=ui).

## Related information
{: #related-info-file-storage-vpc}

For more information about the {{site.data.keyword.cloud_notm}} VPC, see [Virtual Private Cloud](https://www.ibm.com/cloud/learn/vpc){: external}.

For VPC documentation, see [Getting started with Virtual Private Cloud](/docs/vpc?topic=vpc-getting-started).

## Next steps
{: #file-storage-vpc-next-steps}

* [Plan for creating file shares and mount targets](/docs/vpc?topic=vpc-file-storage-planning).
* [Create a file share and mount targets](/docs/vpc?topic=vpc-file-storage-create).
