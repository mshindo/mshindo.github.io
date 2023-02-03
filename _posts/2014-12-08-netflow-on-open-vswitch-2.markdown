---
date: 2014-12-08 15:04:28+00:00
layout: post
title: NetFlow on Open vSwitch
tags:
- Computer and Networking
language:
- English
post_translations:
- pll_54864e1186265
keywords:
- NetFlow
- Open vSwitch
- OVS
---

Open vSwitch (OVS) has been supporting NetFlow for a long time (since 2009). To enable NetFlow on OVS, you can use the following command for example:

    
    # ovs-vsctl -- set Bridge br0 netflow=@nf -- --id=@nf create NetFlow targets=\"10.127.1.67\"


When you want to disable NetFlow, you can do it in the following way:

    
    # ovs-vsctl -- clear Bridge br0 netflow


NetFlow has several versions. V5 and V9 are the most commonly used versions today. OVS supports NetFlow V5 only. NetFlow V9 is not supported as of this writing (and very unlikely to be supported because OVS already supports IPFIX, a direct successor of NetFlow V9).

NetFlow V5 packet consists of a header and flow records (up to 30) following the header (see below).

[![NetFlow V5 header format]({{site.baseurl}}/images/NetFlow-V5-Header.svg)NetFlow V5 header format

[![NetFlow V5 flow record format]({{site.baseurl}}/images/NetFlow-V5-Flow-Record.svg)NetFlow V5 flow record format

NetFlow V5 cannot handle IPv6 flow records. If you need to monitor IPv6 traffic, you must use sFlow or IPFIX.

Comparing to NetFlow implementation on typical routers and switches, the one on OVS has a couple of unique points that you should keep in mind. I will describe such points below.

Most NetFlow-capable switches and routers support so called a “sampling”, where only subset of packets are processed for NetFlow (there are couple of ways how to sample the packet but it is beyond the scope of this blog post). NetFlow on OVS doesn't support sampling. If you need to sample the traffic, you need to use sFlow or IPFIX instead.

Somewhat related to the fact that NetFlow on OVS doesn’t do sampling, it is worth noting that “byte count (dOctets)” and “packet count (dPkts)” fields in a NetFlow flow record (both are 32bit fields) may wrap around in case of elephant flows. In order to circumvent this issue, OVS sends multiple flow records when bytes or packets count exceeds 32bit maximum so that it can tell the collector with the accurate bytes and packets count.

Typically, NetFlow-capable switches and routers have a per-interface configuration to enable/disable NetFlow in addition to a global NetFlow configuration. OVS on the other hand doesn’t have a per-interface configuration. Instead it is a per-bridge configuration that allows you to enable/disable NetFlow on a per-bridge basis.

Most router/switch-based NetFlow exporters allow to configure the source IP address of NetFlow packet to be exported (and if it is the case, loopback address is a reasonable choice for this address). OVS, however, doesn’t have this capability. The source IP address of NetFlow packet is determined by the IP stack of host operating system and it is usually an IP address associated with the outgoing interface. Since NetFlow V5 doesn’t have a concept like “agent address” in sFlow, most collectors distinguish the exporters by the source IP address of NetFlow packets. Because OVS doesn’t allow us to configure the source IP address of NetFlow packets explicitly, we'd better be aware that there is a chance for the source IP address of NetFlow packets to change when the outgoing interface changes.

Although it is not clearly described in the document, OVS, in fact, supports multiple collectors as shown in the example below. This configuration provides redundancy of collectors.

    
    # ovs-vsctl -- set Bridge br0 netflow=@nf -- --id=@nf create NetFlow targets=\[\"10.127.1.67:2055\",\"10.127.1.68:2055\"\]


When flow-based network management is adopted, In/Out interface number included in a flow record has a significant importance because it is often used to fitter the traffic of interest. Most commercial collector products have such a sophisticated filtering capability. Router/switch-based NetFlow exporter uses SNMP’s IfIndex to report In/Out interface number. NetFlow on OVS on the other hand uses OpenFlow port number instead to report In/Out interface number in the flow record. OpenFlow port number can be found by ovs-ofctl command as follows:

    
    # ovs-ofctl show br0
    OFPT_FEATURES_REPLY (xid=0x2): dpid:0000000c29eed295
    n_tables:254, n_buffers:256
    capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
    actions: output enqueue set_vlan_vid set_vlan_pcp strip_vlan mod_dl_src mod_dl_dst mod_nw_src mod_nw_dst mod_nw_tos mod_tp_src mod_tp_dst
    1(eth1): addr:00:0c:29:ee:d2:95
    config: 0
    state: 0
    current: 1GB-FD COPPER AUTO_NEG
    advertised: 10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-FD COPPER AUTO_NEG
    supported: 10MB-HD 10MB-FD 100MB-HD 100MB-FD 1GB-FD COPPER AUTO_NEG
    speed: 1000 Mbps now, 1000 Mbps max
    LOCAL(br0): addr:00:0c:29:ee:d2:95
    config: 0
    state: 0
    speed: 0 Mbps now, 0 Mbps max
    OFPT_GET_CONFIG_REPLY (xid=0x4): frags=normal miss_send_len=0


In this example, OpenFlow port number of "eth1" is 1. Some In/Out interface number have a special meaning in OVS. An interface local to the host (interfaces that is labeled as "LOCAL" in the example above) is represented as 65534. Output interface number 65535 is used for broadcast/multicast packets whereas most router/switch-based NetFlow exporters use "0" for both these two cases.

Those who are aware that IfIndex information was added to “Interface” table in OVS relatively recently may think that using IfIndex instead of OpenFlow port number is the right thing to do. May be true, but it is not that simple. For example, tunnel interface created by OVS doesn't have an IfIndex, so it is not possible to export a flow record for traffic that traverses over tunnel interfaces if we simply chose IfIndex as NetFlow In/Out interface number.

NetFlow V5 header has a field called “Engine ID” and “Engine Type”. How these fields are set in OVS by default depends on the type of datapath. If OVS is run in a user space using netdev, Engine Type and Engine ID are derived from a hash value based on the datapath name and Engine ID and Engine Type are the most significant and least significant 8 bit of the hash value respectively. On the other hand, in the case of Linux kernel datapath using netlink, IfIndex of datapath is set to both Engine ID and Engine Type. You can find the IfIndex of OVS datapath by the following command:

    
    # cat /proc/sys/net/ipv4/conf/ovs-system


If you don't like the default value of these fields, you can configure them explicitly as shown below:

    
    # ovs-vsctl set Bridge br0 netflow=@nf -- --id=@nf create NetFlow targets=\”10.127.1.67:2055\” engine_id=10 engine_type=20


The example above shows the case where Engine Type and Engine ID are set at the same time when NetFlow configuration is created for the first time. You can also change NetFlow-related parameters after NetFlow configuration is created by doing like:

    
    # ovs-vsctl clear NetFlow br0 engine_id=10 engine_type=20


In general, typical use case of Engine Type and Engine ID is to distinguish logically separated but physically a single exporter. Good example of such a case is Cisco 6500, which has MSFC and PFC in a single chassis but having separate NetFlow export engines. In OVS case, it can be used to distinguish two or more bridges that is generating NetFlow flow records. As I mentioned earlier, the source IP address of NetFlow packet that OVS exports is determined by the standard IP stack (and it is usually not the IP address associated to the bridge interface in OVS that is NetFlow-enabled.) Therefore it is not possible to use the source IP address of NetFlow packets to tell which bridge is exporting the flow records. By setting a distinct Engine Type and Engine ID on each bridge, then you can tell it to the collector. To my knowledge, not many collectors can use Engine Type and/or Engine ID to distinguish multiple logical exporters however.

There is another use case for Engine ID. As I already explained that OVS uses OpenFlow port number as an In/Out interface number in NetFlow flow record. Because OpenFlow port number is a per bridge unique number, there is a chance for these numbers to collide across bridges. To get around this problem, you can set “add_to_interface” to true.

    
    # ovs-vsctl set Bridge br0 netflow=@nf -- --id=@nf create NetFlow targets=\"10.127.1.67:2055\" add_to_interface=true


When this parameter is set to true, the 7 most significant bits of In/Out interface number is replaced with the 7 least significant bits of Engine ID. This will help interface number collision happen less likely.

Similar to typical router/switch-based NetFlow exporters, OVS also has a concept of active and inactive timeout. You can explicitly configure active timeout (in seconds) using the following command:

    
    # ovs-vsctl set Bridge br0 netflow=@nf -- --id=@nf create NetFlow targets=\"10.127.1.67:2055\" active_timeout=60


If it is not explicitly specified, it defaults to 600 seconds. If it is specified as -1, then active timeout will be disabled.

While OVS has an inactive timeout mechanism for NetFlow, it doesn’t have an explicit configuration knob for it. When flow information that OVS maintains is removed from the datapath, information about those flows will also be exported via NetFlow. This timeout is dynamic; it varies depending on many factors like the version of OVS, CPU and memory load etc., but typically it is 1 to 2 seconds in recent OVS. This duration is considerably shorter than that of typical router/switch-based NetFlow exporters which is 15 seconds in most cases.

As with most router/switch-based NetFlow exporters, OVS exports flow records for ICMP packet by filling the number calculated by "ICMP Type * 256 + Code” to source port number in a flow record. Destination port number for ICMP packets is always set to 0.

NextHop, source and destination of AS number, source and destination of Netmask are always set to 0. This is an expected behavior as OVS is inherently a “switch”.

While there are some caveats as I described above, NetFlow on OVS is a very useful tool if you want to monitor traffic handled by OVS. One advantage of NetFlow over sFlow or IPFIX is that there are so many open source or commercial collectors available today. Whatever flow collector you choose most likely supports NetFlow V5. Please give it a try. It will give you a great visibility about the traffic of your network.
