---

copyright:
  years: 2024, 2024
lastupdated: "2024-10-30"

keywords:

subcollection: vpc

---

{{site.data.keyword.attribute-definition-list}}

# Automatic attachments for reservations (beta)
{: #automatic-reservation-vpc}

Automatic attachments for reservations is a beta feature that is available for evaluation and testing purposes.
{: beta}

If a bare metal or virtual server is configured to enable automatic attachments, the system attempts to automatically attach the resource to a reservation in the same account that allows automatic attachments. The reservation must be configured to support automatic attachments and must contain matching properties (account, profile, zone) with available capacity.
{: shortdesc}

The billing discount for the attached compute resource is applied immediately and automatically.
{: note}

## Limitations
{: #automatic-reservations-limitations-vpc}

* You can't target a specific reservation and configure a resource automatically. They are mutually exclusive.

* For virtual servers, when an auto-attached compute resource is stopped or deleted, the resource automatically releases its hold on reserved capacity. Targeted attachments maintain their hold on reserved capacity when they are stopped.

* If a bare metal server is stopped or powered off, the auto-attached reservation is not automatically released. The auto-attached reservation is only released when the bare metal server is canceled or de-provisioned.

* When a compute resource detaches and releases its hold on a reservation, the system automatically attempts to attach another matching resource to the reservation.

## Quotas and reservations
{: #quotas-and-reservations}

Reservations count against the appropriate quota for the resource type. So bare metal reservations count against the bare metal server quota and virtual server reservations count against instance quotas. However, compute resources that are attached to a reservation don't count against quotas.

When you create a new compute resource that is configured for automatic attachment, space is dynamically determined in an existing, matching reservation and counts the quota. So if no capacity is available in a matching automatic reservation, compute resources count against your quota. If capacity is available in a matching automatic reservation, then the new resource isn't counted against your quota.

The same dynamic quota calculation applies to new automatic reservations. Typically, the full quantity of the automatic reservation counts against your account quota. However, if you have automatic compute resources that match the reservation, they are considered. Then, the difference of the reservation and the number of resources are deducted from your account quota.

## Default behavior
{: #ar-default-behavior}

TBD
