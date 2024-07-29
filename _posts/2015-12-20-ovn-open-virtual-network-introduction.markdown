---
date: 2015-12-20 05:22:45+00:00
layout: post
title: OVN (Open Virtual Network) Introduction
description: Virtual Networking based on Open vSwitch (OVS)
image: '/images/mike-gattorna-zCIFIYEYhio-unsplash.jpg'
tags:
- Computer and Networking
language:
- English
keywords:
- GENEVE
- Open vSwitch
- OVN
- OVS
- STT
- VXLAN
---

OVN (Open Virtual Network) is an open source software that provides virtual networking for features like L2, L3 and ACL, etc. OVN works with OVS (Open vSwitch) which has been adopted widely. OVN has been developed by the same community as that of OVS and it is treated almost like a sub-project of OVS. Just like OVS, the development of OVN is 100% open and discussion is being made on a public mailing list and IRC. While VMware and RedHat are the primary contributors today, it is open to everybody who wishes to contribute to OVN.

The target platform for OVN is Linux-derived hypervisors such as KVM and Xen and containers. DPDK is also supported. As of this writing, there is no ESXi support. Because OVS has been ported to Hyper-V (still in progress, though), OVN may support Hyper-V in the future.

It is important to note that OVN is CMS (Cloud Management System) agnostic and takes OpenStack, Docker and Mesos into the scope. Among these CMPs, OpenStack would have the highest significance for OVN. A better integration with OpenStack (compared to Neutron OVS plugin as of today) is probably one of the driving factors of OVN development.

OVN share the same goal as OVS, that is, supporting large scale deployment consisting of thousands of hypervisors with production quality.

Please see the diagram depicted below showing the high level architecture of OVN.

![OVN Architecture]({{site.baseurl}}/images/OVN-Architecture.png)
*OVN Architecture*

As you can see, OVN has two new components (processes). One is "ovn-northd" and another is "ovn-controller". As its name implies, ovn-northd provides a northbound interface to CMS. As of this writing, only one ovn-northd exists in one OVN deployment. However, this part will be enhanced in the future to support some sort of clustering for redundancy and the scale-out.

One of the most unique architectures of OVN is probably the fact that the integration point with CMS is a database (DB). Although many people would expect that RESTful API is the integration point as a northbound interface, ovn-northd is designed to interact with CMS via DB.

ovn-northd uses two databases. One is called "Northbound DB", which holds a "desired state" of virtual network. More specifically, it holds information about logical switch, logical port, logical router, logical router port, and ACL. It is worth noting that Northbound DB doesn't hold any "physical" information. An entity sitting north side of ovn-northd (typically CMS) writes information to this Northbound DB to interact with ovn-northd. Another database that ovn-northd takes care of is "Southbound DB" which holds runtime state such as physical, logical and binding information. Specifically, Southbound DB includes information related to chassis, datapath binding, encapsulation port binding, and logical flows. One of the important roles of ovn-northd is to read Northbound DB, translates it to logical flows and write them to Southbound DB.

On the other hand, ovn-controller is a distributed controller, which will be installed on every single hypervisor. ovn-controller reads information from Southbound DB and configures ovsdb-server and ovs-vswitchd running on the same host accordingly. ovn-controller translates logical flow information populated in Southbound DB to physical flow information and then install it to OVS.

Today OVN uses OVSDB as a database system. Inherently database can be anything and doesn't have to be OVSDB. However, as OVSDB is an intrinsic part of OVS, on which OVN always depends, and developers of OVN know the characteristics of OVSDB very well, they decided to use OVSDB as a database system for OVN for the time being.

Basically OVN provides three features; L2, L3 and ACL (Security Group).

As an L2 feature, OVN provides a logical switch. Specifically, OVN creates an L2-over-L3 overlay tunnel between hosts automatically. OVN uses Geneve as a default encapsulation. Considering the necessity of metadata support, multipath capability and hardware acceleration, Geneve would be the most desired encapsulation of choice. In case where hardware acceleration on the NIC for Geneve is not available, OVN allows to use STT as the second choice. In general, HW-VTEP doesn't support Geneve/STT today, OVN uses VXLAN when talking to HW-VTEPs.

In terms of L3 feature, OVN provides so called a "distributed logical routing". L3 features provided by OVN are not centralized meaning each host executes L3 function autonomously. L3 topology that OVN supports today is very simple. It routes traffic between logical switches directly connected to it and to the default gateway. As of this writing it is not possible to configure a static route other than the default route. It is simple but sufficient to support basic L3 function that OpenStack Neutron requires. NAT will be supported soon. 

OVS plugin in OpenStack Neutron today implements Security Group by applying iptables to tap interface (vnet) on Linux Bridge. Having both OVS and Linux Bridge at the same time makes the architecture somewhat complex. 

![under-the-hood-scenario-1-ovs-compute]({{site.baseurl}}/images/under-the-hood-scenario-1-ovs-compute.png)
*The conventional architecture of OpenStack Neutron OVS plugin (source: http://docs.ocselected.org/openstack-manuals/kilo/networking-guide/content/figures/6/a/a/common/figures/under-the-hood-scenario-1-ovs-compute.png)*

Since OVS 2.4, OVS has been integrated with "conntrack" feature available on Linux, so it is possible to implement stateful ACL by OVS without relying on iptables. OVN takes advantages of this OVS & conntrack integration to implement the ACL. 

Since conntrack integration is an OVS feature, one can use OVS+conntrack without OVN. However, OVN allows you to use stateful ACL without explicit awareness of conntrack because OVN compiles logical ACL rules to conntrack-based rules automatically, which would be appreciated by many people. 

I will go into a bit more detail about L2, L3 and ACL features of OVN in the subsequent posts.

Photo by <a href="https://unsplash.com/@mikegattorna?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Mike Gattorna</a> on <a href="https://unsplash.com/photos/a-close-up-of-a-stove-and-oven-in-a-kitchen-zCIFIYEYhio?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
  