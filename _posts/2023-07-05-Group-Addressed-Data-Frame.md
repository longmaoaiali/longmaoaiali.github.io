---
layout:     post
title:      802.11 Group Addressed Data Frame 
subtitle:   multicast frame
date:       2023-07-05
author:     BuBu
header-img: img/yellow-background.jpg
catalog: true
tags: 
    - WLAN Driver
---

----------

### 1.Acronyms

**HMMC**: host-managed multicast   
**PE**: Packet Extension


----------

### 2.Multicast Enhancement(Group addressed Data frame)

#### 2.1.Multicast Enhancement Background

> Additional rules for group addressed frames. **An AP that transmits group addressed frames with an <HE-MCS, NSS> tuple where the HE-MCS is a mandatory HE-MCS and NSS = 1.**

Group addressed Data frame only be sent with mandatory MCS and NSS=1, this is the reason why we transmit multicast to unicast function (**multicast enhancement**) in Host Driver. For High band group addressed traffic, mandatory MCS and NSS=1 can't meet the throughput requirement.  

----------

#### 2.2.Multicast Enhancement Feature

**Configure host-managed multicast transmission:**

- Enable multicast enhancement: cfg80211tool athX mcastenhance 6
- Add entry into the HMMC list: wlanconfig athX hmmc add 224.0.67.67 240.0.0.0
- Dump the entries from the HMMC list: wlanconfig athX hmmc dump   


**DMA MAP operation modify dst mac address:** 

	dp_tx_me_send_convert_ucast()
	
	for (new_mac_idx = 0; new_mac_idx < new_mac_cnt; new_mac_idx++) {
		dstmac = newmac[new_mac_idx];
		qdf_mem_copy(mc_uc_buf->data, dstmac, QDF_MAC_ADDR_SIZE);
		// qdf_mem_map_nbytes_single() - Map memory for DMA, 
		// QDF_DMA_TO_DEVICE: data going from device to memory
		/**
		* qdf_mem_map_nbytes_single - Map memory for DMA
		* @osdev: pomter OS device context
		* @buf: pointer to memory to be dma mapped
		* @dir: DMA map direction
		* @nbytes: number of bytes to be mapped.
		* @phy_addr: ponter to recive physical address.
		*
		* Return: success/failure
		*/
		status = qdf_mem_map_nbytes_single(vdev->osdev, mc_uc_buf->data, QDF_DMA_TO_DEVICE, QDF_MAC_ADDR_SIZE, &paddr_mcbuf);
			
	}
	
	//the msdu_info include multiple unicast frame, each seg_info_new is a unicast frame from multicast frame transition
	seg_info_new->frags[0].vaddr =  (uint8_t *)mc_uc_buf;
	seg_info_tail->next = seg_info_new
	msdu_info.u.sg_info.curr_seg = seg_info_head;
	msdu_info.num_seg = curr_mac_cnt;
	msdu_info.frm_type = dp_tx_frm_me;
	//send the msdu frame 
	dp_tx_send_msdu_multiple(vdev, nbuf, &msdu_info);

----------

### 3.Packet Extension

#### 3.1.Multicast Packet Extension

> A STA transmitting an HE PPDU that carries a broadcast frame shall set the value of the **TXVECTOR parameter NOMINAL_PACKET_PADDING** (nominal packet padding) to **16 µs**. A STA transmitting an HE PPDU that carries a group addressed, but not broadcast, frame shall not set the value of the TXVECTOR parameter NOMINAL_PACKET_PADDING to a value that is less than that required for any of the recipients in the group. The nominal packet padding is used in computing the PE field duration (see 27.3.13).

> A PE field of duration 0 µs, 4 µs, 8 µs, 12 µs, or 16 µs is present in an HE PPDU. **The PE field provides additional receive processing time at the end of the HE PPDU. If packet extension and/or signal extension is applied, the PHY entity shall wait until the packet extension and/or signal extension expires before returning to the RX IDLE state, as shown in Figure 27-63.**

<font color="#FF8C00">A typical state machine implementation of the receive PHY:  </font> 
<img src="/img/post/2023-07-10-state-machine-of-the-receive-PHY.png"/>


> A STA transmitting an HE PPDU to a receiving STA shall include post-FEC padding determined by the pre-FEC padding factor (see 27.3.12); and after including the post-FEC padding,** the transmitting STA shall include a packet extension with a duration** indicated by the TXVECTOR parameter NOMINAL_PACKET_PADDING (see 27.3.13).

> If **BCC encoding **is used, the Data field shall consist of the SERVICE field, the PSDU, the pre-FEC PHY padding bits, the tail bits, and the post-FEC padding bits. If **LDPC encoding** is used, the Data field shall consist of the SERVICE field, the PSDU, the pre-FEC PHY padding bits, and the post-FEC padding bits. No tail bits are present if LDPC encoding is used. 

### 4.MLO Multicast(Group addressed Data frame)

#### 4.1.AP MLD duplicate multicast on both link

As per standard AP must duplicate multicast/broadcast packet on both link.  
Impact: Duplicate Rx packet at Station   
Standard Approach: MLD station to be configured to receive multicast/broadcast only one link.  
**802.11be Standard Original Text:**   
> An AP MLD shall use SNS11 in Table 10-5 (Transmitter sequence number spaces) maintained by the MLD to determine the sequence number of a group addressed Data frame that is transmitted by an AP affiliated with the AP MLD so that <font color="#FF0000"> the same group addressed Data frame transmitted over multiple links by the AP MLD uses the same sequence number for transmission on each link.</font>  

> An MLD shall implement RC16 in Table 10-6 (Receiver caches) maintained by the MLD to discard duplicate group addressed Data that are delivered from the associated MLD. <font color="#FF0000">A duplicated group addressed Data frame received on any link shall be discarded.</font> The method used to handle the sequence number wrap around for duplicate detection is implementation specific.     

> 35.3.15 Multi-link operation group addressed frames

----------

#### 4.2.NSTR mobile AP MLD TX multicast only on the primary link

>35.3.19 NSTR mobile AP MLD operation  

>An NSTR mobile AP MLD shall designate one of the links of an NSTR link pair as the primary link. The other link of the NSTR link pair is the nonprimary link. When the NSTR mobile AP MLD intends to change the channel/operating class for the primary link, it shall perform (#16959)the channel switch procedure. <font color="#FF0000">The NSTR mobile AP MLD shall schedule for transmissions of Beacon and Probe Response frames and group addressed Data frames only on the primary link. </font>  


