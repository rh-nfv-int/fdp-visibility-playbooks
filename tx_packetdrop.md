## BZ#1762725  Tx Packet drop in OVS-DPDK

OVS may not be able to transmit packets for various reasons and initially, there was only a single counter to track packet drops due to any of these reasons. Having a separate drop counter for every possible reason for packet drop will give a clear insight into the reason for drops. 

The possible reasons why the packet drop counter may be incremented during TX in OVS-DPDK are :

Drops due to egress policing
Drops due to MTU exceeded
Drops reported from DPDK PMD Tx function

This is a proposed patch upstream [1] that adds counters which deal with drops due to the above reasons. These counters are displayed along with other counters in the  “ovs-vsctl get interface <iface> statistics” command.

[1] https://mail.openvswitch.org/pipermail/ovs-dev/2019-September/362872.html


ovs-vsctl get interface <iface> statistics
{ovs_rx_qos_drops=0, ovs_tx_failure_drops=0, ovs_tx_invalid_hwol_drops=0, ovs_tx_mtu_exceeded_drops=0, ovs_tx_qos_drops=0, ovs_tx_retries=0, rx_1024_to_1522_packets=0, rx_128_to_255_packets=0, rx_1523_to_max_packets=0, rx_1_to_64_packets=0, rx_256_to_511_packets=0, rx_512_to_1023_packets=0, rx_65_to_127_packets=0, rx_bytes=0, rx_dropped=0, rx_errors=0, rx_packets=0, tx_bytes=0, tx_dropped=0, tx_packets=0}


####Below is a detailed explanation of these counters:
1. Drops due to egress/ingress policing
*Egress/ingress (Qos/Rate Limiting) rate exceeding configured egress/ingress rate.

*Configuration of the egress/ingress rates can be done by the below commands
ovs-vsctl set port <port> qos=@newqos -- --id=@newqos create qos type=egress-policer other-config:cir=<rate> other-config:cbs=<size>
where  cir = Committed Information Rate , cbs= Committed Burst Size
ovs-vsctl set interface <iface> ingress_policing_limit=<rate>

*Current configured Qos or Egress policy can be examined with 
ovs-appctl -t ovs-vswitchd qos/show <iface>

*Current configured Rate Limiting or Ingress policy can be examined with
ovs-vsctl list interface <iface>

*The logic of counters incrementation
Qos drops = total number of packets - number of packets processed by   egress/ingress policer

*The counters dealing with these drops are  ovs_rx_qos_drops,ovs_tx_qos_drops

2. MTU exceeded drops
*Tx Packet length > configured MTU of the device

*The current configured MTU can be examined with
ovs-vsctl list interface <iface>

*The logic of counters incrementation
mtu drops = total number of packets - number of packets processed by packet len filter

*The counter dealing with these drops is ovs_tx_mtu_exceeded_drops

3.Queue full drops
*Packets are dropped because the transmission queue is full.
Ovs_tx_failure_drops  deals with these drops.



To prevent these drops Qos andRate Limiting and MTU drops configure those values on interfaces accordingly. 
Commands to remove Qos and Rate Limiting are “ovs-vsctl -- destroy Qos <iface> -- clear Port <iface> qos” and  “ovs-vsctl set interface <iface> ingress_policing_rate=0” respectively.
Command to modify MTU “ovs-vsctl set Interface <iface> mtu_request=1500”
And to prevent tx_failure drops one of the solutions is to increase the queue size.

