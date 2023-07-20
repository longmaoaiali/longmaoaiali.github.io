---
layout:     post
title:      Address Search Table
subtitle:   AST  
date:       2023-07-19
author:     BuBu
header-img: img/bule-background.jpg
catalog: true
tags:
    - WLAN Driver 
---

----------
### 1.Acronyms

MEC: Multicast Echo check       
UMAC: Upper MAC  
LMAC: Lower MAC  
TCL: Transmit(Tx) Classifier   
TQM: Tx Queue Manger  
WBM: Wireless Buffer Manager  
REO: Rx Re-Order Engine  
Tx/Rx PCU: Tx/Rx Protocol Unit  
EN/DE CR: Encryption /Decryption  
Tx/Rx OLE: Tx/Rx Offload Engine  
Tx/Rx DMA: Direct Memory Access  
PDG: PPDU Descriptor Generator   
HW SCH: Hardware Scheduler   

----------

### 2.Concept

#### 2.1.AST

AST table is just array and organized as a hash table, The hash value of the AST mainly depend on the MAC address of the AST entry. How to match the MAC address a frame to this table, depend on TX/RX/frame type.   
The AST table and entry will be accessed by HW TX (TCL), HW RX PCU, OLE, and FW.  
The AST table and entry are allocated in CMEM or SRAM uncached, and the physical address will populate to HW.   

<font color="#FF8C00">AST Data Structure:  </font> 
<img src="/img/post/2023-07-19-AST-Data-Structure.png"/>

	# AST Initialization
	ar_wal_ast_create()
	ar_wal_ast_attach()
	struct addr_search_entry {
		uint32_t mac_addr_31_0    
		uint32_t mac_addr_47_32   
				 mcast_echo_check   
		uint32_t peer_entry_addr_31_0   
		uint32_t peer_entry_addr_39_32
	}      

	Peer_entry_addr point to the physical address of wal_sw_peer_key
	Wal_sw_peer_key is a very basic structure to control HW TX/RX, it is allocated when peer allocation, we can access it from peer->u.default_peer_key                           

The **addr\_search\_entry** struct is created by FW and be used by FW/HW.   
The filed **mac\_addr\_31\_0** used to search, like MAC address, mcast, next_hop.  
The filed **mcast\_echo\_check** used to be a function, like **mcast\_echo\_check**, this is will instruct the HW to treat this entry as MEC entry and then it will inform FW it is a MEC frame once the entry is hit.   
For the **peer\_entry\_addr\_31\_0**. When the frame hit this AST, the **peer\_entry\_addr\_31\_0** will tell HW how to handle this frame for TX/RX. The **peer\_entry\_addr\_31\_0** point to **struct wal\_sw\_peer\_key**.     

<font color="#FF8C00">sw peer key Structure:  </font> 
<img src="/img/post/2023-07-19-Structure-sw-peer-key.png"/>


----------

#### 2.2.MEC

Multicast Echo check (MEC) feature ensures multicast or broadcast packets that are transmitted from a STA and echoed back to the same STA because of intra-BSS feature of AP gets dropped.   

In 802.11ax platform architecture, for each mcast or bcast packet received on PSTA VAP, host will compare the source mac address with all PSTA vdev mac addresses present on that radio. If it matches, then it is a mcast echo pkt, host will drop that pkt.   

Starting with 802.11be devices, MEC feature is being handled by FW with HW assist. FW will add MEC entry with STA vdev mac address, whenever a mcast or bcast pkt is transmitted in STA mode. This FW change is applicable for PSTA vap as well. So, when PSTA receives mcast or bcast pkt and if src mac matches with MEC entry, FW sets error code and host needs to look for the RxDMA error code "rxdma multicast echo err" and drop the pkt.

----------

### 3.AST Table for TX

Base the section 2.1, we know **wal\_sw\_peer\_key** was hit. The following steps show how TCL query AST table and send the MSDUs to TQM Q.  

1. Once TCL use the AST to search and it hit, then HW will continue access **who\_classify\_info** from **wal\_sw\_peer\_key**.
2. Based on the classification, find the right **txpt\_classify\_info**, then the frame will be Q to TQM or goes to FW or drop.
3. The **tqm\_flow\_handler** tells if goes to fw or TQM, the **msdu\_drop** is to drop the frame, the **tqm\_flow\_pointer** is the TQM MSDUQ address  

Firmware struct code:  

	struct hw_peer_entry {
    	struct rx_peer_entry_details peer_entry_rx; /* HW RX peer entry */
    	struct who_classify_info     tx_classify_info;
	};

	struct txpt_classify_info {
		uint32_t tqm_flow_pointer_non_udp_31_0                           : 32; // [31:0]
		uint32_t tqm_flow_pointer_udp_31_0                               : 32; // [31:0] msdu_flow_dma_ptr
		uint32_t tqm_flow_handler                                        :  2, // [1:0] TXPT_TO_FW or TXPT_TO_TQM
				tqm_flow_loop_handler                                   :  1, // [2:2]
				to_tqm_if_m0_fw                                         :  1, // [3:3]
				msdu_drop                                               :  1, // [4:4]
				metadata                                                : 11, // [15:5]
				tqm_flow_pointer_non_udp_39_32                          :  8, // [23:16]
				tqm_flow_pointer_udp_39_32                              :  8; // [31:24]
	};

	1. wal_peer_alloc-->ar_wal_sw_peer_key_alloc-->wal_sw_msdu_flows_alloc-->wal_tx_classify_info_alloc
	Allocate and init the MSDUQ, this is called when peer auth, no MSDUQ is allocated, the msdu_drop == 1

	2. wlan_vdev_dispatch_peer_assocwal_peer_set_param_active (WAL_PEER_PARAM_QOS_EN)-->wal_peer_alloc_default_queues
	When association, set QOS flag, will allocate the TID0 MSDUQ and initialize the txpt_classify_info and msdu_drop == 0. 
	From now on, the frame can go to TQM or FW.

----------

### 4.AST Table for RX

For RX, RX PCU will hit AST table entry. After RX PCU hit the AST table, the rx\_peer\_entry\_details will guide RX to decrypt, which REO Q should enter and some other functions.

<font color="#FF8C00">rx peer entry details Structure:  </font> 
<img src="/img/post/2023-07-19-Structure-rx_peer_entry_details.png"/>  

Firmware struct code:  

	struct rxpt_classify_info {
		uint32_t reo_destination_indication                              :  5, // [4:0]
				lmac_peer_id_msb                                        :  2, // [6:5]
				use_flow_id_toeplitz_clfy                               :  1, // [7:7]
				pkt_selection_fp_ucast_data                             :  1, // [8:8]
				pkt_selection_fp_mcast_data                             :  1, // [9:9]
				pkt_selection_fp_1000                                   :  1, // [10:10]
				rxdma0_source_ring_selection                            :  3, // [13:11]
				rxdma0_destination_ring_selection                       :  3, // [16:14]
				mcast_echo_drop_enable                                  :  1, // [17:17]
				wds_learning_detect_en                                  :  1, // [18:18]
				intrabss_check_en                                       :  1, // [19:19]
				use_ppe                                                 :  1, // [20:20]
				ppe_routing_enable                                      :  1, // [21:21]
				reserved_0b                                             : 10; // [31:22]
	};
	reo_destination_indication for REO2SW Q selection. 

----------

### 5.WDS Learning

<font color="#FF8C00">WDS Learning:  </font>   
<img src="/img/post/2023-07-20-WDS-Learning.png "/>    

<font color="#FF0000">**WDS learning:**</font>   
To get the **rx\_msdu\_end** in fw rx status ring, set the **WCSS\_DMAC\_RXOLE\_MC\_R0\_WDS\_CONFIG.mon\_ring\_mask\_override = 1**.  
To detect any new STA connecting behind the WDS STA, in the **rxpt\_classify\_info** of the WDS STA the wds\_learning\_detect\_en bit needs to set to one.     If this is set any data packet received with TO DS or from DS both set to 1 and with SA address not in AST table, will have its **rx\_msdu\_end** sent to FW status ring with **rx\_msdu\_end.wds\_learning\_event = 1**. Then FW must get SA address add the appropriate AST entry with **next\_hop\_entry** pointing to the TA peer’s AST entry.    
New fields will be added in **rx\_msdu\_end** to get SA mac address and TA **peer\_id**. After allocating the wds entry for that particulat **ast\_index** , the bitmaps** wds\_entry\_keep\_alive\_bitmask** and **wds\_entry\_bitmap** need to be set to one. Send over ring to RT thread.  

<font color="#FF0000">**WDS Roaming:**</font>  
To assist the FW in roaming the following configuration need to be done.   
Register : `WCSS_DMAC_RXOLE_MC_R0_WDS_UPDATE_THRESH` this value is the time limit beyond which the rxole will update the fw with `rx_msdu_end. wds_roaming_event = 1` in the fw status ring when it receives a rx packet with From DS / To DS set to 1, with SA mac address in the AST but TA `peer_id ` doesn’t match the current next hop `peer_id`. This indicates that backend STA has roamed to another WDS sta, so the AST entry needs to be updated appropriately. the bitmap `wds_entry_keep_alive_bitmask` needs to be set to one for that particular `ast_index`. While updating the wds entry we might have to set the drop msdu for the ast entry to prevent any data going to that flowq. This need to happen to RT thread. The `wds_roaming_detect_en` field needs to set in the wds entries as well as in any sta directly connecting to the ap for the roaming events to be generated. Also if we are getting roaming event for a ast entry that is present as a directly 
connected peer, we shouldn’t crash as before in lithium. With the new implementation a STA 
kickout needs to sent to host. After this STA would send the peer_delete for that sta appropriately.


<font color="#FF0000">**Code Flow:**</font>  
	rx_process_recv_status() --> ar_wal_rx_send_process_wds_rx_msdu() --> ar_wal_rx_send_wds_update_msg
	In BE thread, generate a WDS message to RT thread, the WDS message include three type, WDS_LEARNING_EVENT, WDS_ROAMING_EVENT, WDS_KEEP_ALIVE_EVENT.
	In rx_process_recv_status function, HW detect these three event and store it to rx_msdu_end. 

	rt_mlo_ipc_signal_hdlr() --> wal_ast_handle_ml_wds_msg() --> wal_wds_process_msg() 
	In RT thread, process the three type of the message.

	wal_wds_ageing_timer_hdlr()
	Timeout function to delete the WD AST entry.

----------

### 5.Duplicate AST Entry

If VAP MLO is enabled, when a MLO or legacy STA connected, the STA peer itself will have an AST entry, meanwhile, it will notify partner link to create a duplicate AST.  
The duplicate AST entry has the same MAC address, next hop.. Information as original one, that is the duplication. But the `pmac_id` is different, means this duplicate one is actually exist on another pmac. And the duplicate AST don’t have `wal_sw_peer_key` associated, that means, the duplicate one can’t be used for TX.   
Duplicate AST is not much used, it only exist on partner link to let sw and hw know the though this DA can’t be sent by this chip, but can be sent by partner one.  
This duplicate AST feature can be used for HW inter-BSS function.    

- AP 5G receive this STA1 frame, handle that.
- In RX HW, it realize the DA of the frame matches the duplicate AST, HW knows the frame can be TX by another link.
- The HW later send the frame directly to TX HW logic, such as TCL.  
- The HW intra-bss need to enable `CONFIG_INTRA_BSS_HW_ASSIST` in FW.

 
The `vdev_id` as 255 can be a signature as duplicate AST
<font color="#FF8C00">Duplicate AST Entry:  </font>   
<img src="/img/post/2023-07-20-Duplicate-AST-Entry.png "/>   

----------

