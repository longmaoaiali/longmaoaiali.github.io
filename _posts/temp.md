#### TSO  
TCP segmentation offload (TSO) is an optimization technique that reduces the CPU overhead in TCP/IP-related network operations. With TSO enabled, a network internet controller (NIC) splits large data chunks traveling over a network into smaller TCP segments. Without TSO, the segmentation is performed by the CPU, which creates an overhead. TSO is also known as LSO (Large Segment Offload).
Even if the TSO segmentation is done within the (host) CPU rather than the hardware, a CPU utilization benefit can be achieved by invoking the upper layers of the protocol stack less frequently than with the MTU-sized frames.  
`ol_tx_tso_sg_process_skb`  

#### LRO  
In computer networking, Large Receive Offload (LRO) is a technique for increasing the 
inbound throughput of high-bandwidth network connections, by reducing the CPU overhead. It 
works by aggregating multiple incoming TCP packets from a single stream into a larger buffer 
before they are passed higher up the networking stack; this reduces the number of packets that 
have to be processed.

#### Extended NSS

The existing 802.11 standard does not support negotiation of NSS as a function of channel bandwidth. As a result, certain workarounds are required to be used to advertise separate NSS capabilities for 80 MHz and 160 MHz modes. Recently, IEEE has made certain amendments to the standard called Extended NSS, which refers to signaling support for 160 and 80+80 MHz channel bandwidths with NSS support that is different from the maximum NSS signaled to legacy 80 MHz devices. The operating mode notification (OMN) switching to and from 160/80+80 MHz is also part of extended NSS signaling. 


#### WLAN data path statistics
cfg80211tool ath0 txrx_stats 258 Rx rate statistics  
cfg80211tool ath0 txrx_stats 259 Tx rate statistics  
cfg80211tool ath0 txrx_stats 260 Rx PDEV statistics  
cfg80211tool ath0 txrx_stats 261 Tx PDEV statistics  
cfg80211tool ath0 txrx_stats 257 Clear statistics  
cfg80211tool wifi0 fc_peer_stats <mac_addr> Host peer statistics  
cfg80211tool wifi0 dp_fw_peer_stats <mac_addr>:0x3 Firmware peer statistics  
cfg80211tool wifi0 txrx_stats <1-16> Firmware statistics  


802.11 format (native Wi-Fi format)


athstats -i wifiX: This is per-radio level statistic
apstats -v -i athx: This is per-VAP level statistic

wlanconfig ath0 list: This is per client-level statistic
iwconfig ath0: This is per-VAP level statistic
apstats -s -m <mac_addr>: This is per-client level statistic

cfg80211tool ath0 fc_peer_stats <mac_addr>: This is per-client DP level statistic.  
Average Jitter : Exponential Rolling Average Jitter between Tx HW enqueue to Completion  
Average Delay : Exponential Rolling Average delay between Tx HW enqueue to Completion  

There is a separate command to enable this statistic: cfg80211tool wifiX enable_statsv3 2
The sample output of cfg80211tool ath0 fc_peer_stats <mac_addr> command is as follows:

The jitter is calculated between the Tx enqueue and the Tx completions. Every buffer is 
timestamped when they enter the Wi-Fi package. As every buffer/packet needs to carry the in-timestamps, they are stored as part of the packet control buffer. During the Tx completions, the in-timestamp and the current timestamp are used to measure the current transit time and jitter.  

The capability is introduced to measure the jitter per client and per TID. Jitter is defined as a 
statistical variance of the RTP (Real Time protocol) data packet inter-arrival time. In the Real Time 
Protocol, jitter is measured in timestamp units.


Vendors use several metrics to determine the performance of each Wi-Fi link in a network.  
 PHY rates
 RSSI/SNR
 Packet errors
peerratestats <wifiX> &  //Need to run traffic flow

Mirrored Stream Classification Service (MSCS)

VxLAN is a technology devised to create more overlay virtual networks then what could be 
supported by VLANs. VLANs have an inherent limitation due to the 12-bit ID field such that these 
can only support 4094 virtual networks.
VxLAN uses the 24-bit Virtual Network ID (VNI) and hence can support up to 16 million virtual 
networks.



