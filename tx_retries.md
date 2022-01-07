## BZ# 1747531 - VHost TX Retries

VHost Tx Retries is a mechanism in OVS where if packets are dropped may be due to temporarily slow guest/ limited queue size for an interface, this mechanism quickly tries to re-transmit the packets that had failed to send.

There was no way to know if these retries were occurring as they were not reported.

Additionally, max-vhost-retries was hardcoded which means no way for tuning or disabling and also reporting this feature.

Also, if a busy system is sending these extra retry packets, it may also be possible that packets are dropped elsewhere (maybe on any other interface). Hence it is only helpful in avoiding the packet drops due to a temporary slow guest. 

This patch [1] added visibility of how many retries are occurring and also a way to configure max_retries, where a value of 0 means disabling vhost retries.

[1]  https://mail.openvswitch.org/pipermail/ovs-dev/2019-June/359967.html

For vhost-user-client interfaces, the max amount of retries can be changed from the default 8 by setting tx-retries-max
 
ovs-vsctl set Interface vhuclient1 options:tx-retries-max= <value>

ovs-vsctl get interface <vhost-user-client-iface> statistics
{ovs_rx_qos_drops=0, ovs_tx_failure_drops=0, ovs_tx_invalid_hwol_drops=0, ovs_tx_mtu_exceeded_drops=0, ovs_tx_qos_drops=0, ovs_tx_retries=0, rx_1024_to_1522_packets=11139, rx_128_to_255_packets=0, rx_1523_to_max_packets=0, rx_1_to_64_packets=0, rx_256_to_511_packets=0, rx_512_to_1023_packets=0, rx_65_to_127_packets=0, rx_bytes=0, rx_dropped=0, rx_errors=0, rx_packets=0, tx_bytes=0, tx_dropped=0, tx_packets=0}







