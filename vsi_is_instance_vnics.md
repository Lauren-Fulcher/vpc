---

copyright:
  years: 2018, 2022
lastupdated: "2022-07-27"

keywords:

subcollection: vpc

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:tip: .tip}
{:note: .note}

# Managing network interfaces
{: #using-instance-vnics}

After you've created a virtual server instance, you can add new network interfaces or edit the interfaces that are already associated with the instance. When you edit a network interface, you can change its name, associate or unassociate a floating IP address, or access the security group associated with an interface.  
{: shortdesc}

## About network interfaces
{: #about-network-interfaces}

A network interface connects a virtual server instance to a network. When you create a virtual server
instance, you can use network interfaces to assign multiple IP addresses.

The following list highlights how network interfaces work with your instance.

* You can create and assign up to 5 network interfaces for each virtual server instance.
* You can attach each network interface to a different subnet in the same zone.
* Each network is assigned a unique and immutable Media Access Control (MAC) address and receives a private IP address from the subnet range.
* You can attach one floating IP address to the virtual server instance. Initially, the floating IP address must be attached to the primary network interface to establish the data path. By default the primary interface is `eth0` in {{site.data.keyword.cloud_notm}} console.
* You can associate and unassociate a single floating IP address for the instance to and from each network interface.
* You can assign security groups to each network interface.
* You can change the name of any existing network interface.

Bandwidth is distributed across the network interfaces that are attached to the virtual server instance. For more information, see [Network performance notes for profiles](/docs/vpc?topic=vpc-profiles#network-perf-notes-for-profiles).

If you assign a new network interface to a virtual server instance while it is running, you must take action to configure the network interface for the instance to use. You can either stop and then restart the instance, or you can manually configure the interface in the guest operating system. For example, on a Linux-based operating sytem, you can use the `ip link set dev <interface> up` to retrieve the IP address configuration for the interface.
{: note}


## Adding or editing network interfaces
{: #editing-network-interfaces}

To add or edit the network interfaces associated with your virtual server instances, complete the following steps.
1. In the {{site.data.keyword.cloud_notm}} console, go to **Menu icon ![Menu icon](../icons/icon_hamburger.svg) > VPC Infrastructure > Compute > Virtual server instances**.
2. Click the name of a virtual server instance that includes the network interface that you want to edit. Or you can add a new network interface to the virtual server instance.
3. On the Instance details page, find the **Network interfaces** section.  
4. For specific steps for adding a floating IP address or adding a network interface, see the following sections.

### Adding a floating IP address
{: #adding-floating-ip}

If you want to add a floating IP address to a network interface to allow traffic from the internet to access your virtual
server instance, complete the following steps.

1. If you are adding a floating IP address to the virtual server instance for the first time, identify the primary network interface in the **Network interfaces** section of the Instance details page.
By default, the first interface is named `eth0`. Initially associating the floating IP address with the primary network interface helps
establish the data path. Later, you can associate the floating IP to a different network interface if you desire.
2. Click the pencil icon to edit the primary network interface.
3. On the **Edit network interface** page, locate the **Floating IP address** field. You can select **Reserve a new floating IP** or you can select an existing
floating IP address.
4. After making your selection, click **Save**.

### Adding a network interface
{: #add-network-interface}

You can add up to 5 network interfaces to your virtual server instance. A network interface establishes a unique IP address,
giving the option of multiple IP address for your virtual server instance. Before you begin adding a new interface, make sure that
you have already created a unique subnet to associate with the new network interface. Each network interface should be on a different subnet within
the same zone.

To add a new network interface to your virtual server instance, complete the following steps.

1. In the **Network interfaces** section of the Instance details page, click **New interface**.
2. On the **New network interface** page, the interface name defaults to an incremented number. If this is the first new interface after your primary interface, the default name is `eth1`. You can change the name if you want.
3. Select a subnet that is unique from the subnets that are assigned to your existing network interfaces.
4. Select the security group that you want to associate with the network interface.
5. Click **Create**.
6. If your virtual server instance was running when you added the network interface, you must take action to configure the network interface for the instance. You can either stop and then restart the instance, or you can manually configure the interface in the guest operating system. For example, on a Linux-based operating sytem, you can use the `ip link set dev <interface> up` to retrieve the IP address configuration for the interface.

## Configuring a virtual server instance with multiple interfaces
{: #configure-multiple-interfaces}

Sometimes virtual servers instances communicate with other instances through multiple network interfaces, for example, to use different subnets for different communication purposes. You might want one subnet for data communication and another subnet for control communication. Because a virtual server instance is configured with a single default route that is associated with the primary network interface, communication on other interfaces might not work initially. (IP spoofing checks for security purposes can also prevent the communication flow on non-primary network interfaces.)

You can establish communication for a secondary network interface by using one of the following options. These examples are for a Linux platform such as `Ubuntu`.

* Add a static route for the second subnet.
* Add a separate routing table for the second subnet.

The routing tables for virtual server instances are generated by default when an instance is provisioned. Modifying the generated routing table is a non-trivial task. If you need the ability to modify the routing table of a virtual server instance, when you provision the instance you can include user-data to describe and reconfigure the instance's routing table. Another option is to use a custom image to provision an instance and control what is created for multiple network interfaces. When you use the default images to provision instances, the network configuration is created for you. You have limited control of what is generated.
{: important}

The following sections describe these solutions in more detail.

Here is the virtual server setup:

* There are two virtual servers (`Virtual-server-1` and `Virtual-server-2`) where both virtual servers belong to the same VPC.
* Each virtual server has two interfaces from two subnets:
    * _Virtual-server-1_ has interface `eth0` from `net_1_0`, and interface `eth1` from `net_1_1`.
    * _Virtual-server-2_ has interface `eth0` from `net_2_0`, and interface `eth1` from `net_2_1`.
* Each virtual server's `net_*_0` is set up with a default route.
* Each subnet's gateway IP and CIDR are known.

    Alphabetic names are used here. The real gateway ID would be similar to `192.168.100.1`, the CIDR `192.168.100.0/24`, and the IP address `192.168.100.6`.
    {: note}

| Subnet | Gateway IP | Subnet CIDR | Interface IP |
| --- | --- | --- | --- |
| `net_1_0` | `gw_ip_1_0` | `cidr_1_0` | `ip_1_0` |
| `net_1_1` | `gw_ip_1_1` | `cidr_1_1` | `ip_1_1` |
| `net_2_0` | `gw_ip_2_0` | `cidr_2_0` | `ip_2_0` |
| `net_2_1` | `gw_ip_2_1` | `cidr_2_1` | `ip_2_1` |

### Adding a static route for the second interface
{: #adding-static-route-second-interface}

This solution makes one subnet the default gateway (automatically by virtual server instance creation), and adds a static route for the second subnet:

* On `Virtual-server-1`:

   ```
   ip route add cidr_2_1 via gw_ip_1_1 dev eth1
   ```
   {: pre}

* On `Virtual-server-2`:

   ```
   ip route add cidr_1_1 via gw_ip_2_1 dev eth1
   ```
   {: pre}

### Adding a separate routing table for the second interface
{: #adding-route-table-second-interface}

* On `Virtual-server-1`:

```
   echo 201 eth1tab >> /etc/iproute2/rt_tables
   ip route add cidr_1_1 dev eth1 proto kernel scope link src ip_1_1 table eth1tab
   ip route add default via gw_1_1 dev eth1 table eth1tab
```
{: codeblock}

* On `Virtual-server-2`:

```
echo 201 eth1tab >> /etc/iproute2/rt_tables
ip route add cidr_2_1 dev eth1 proto kernel scope link src ip_2_1 table eth1tab
ip route add default via gw_2_1 dev eth1 table eth1tab
ip rule add from ip_2_1 table eth1tab
```
{: codeblock}

Though this approach is a bit more complicated than adding a static route, it provides a way to customize routing policies for different interfaces.
