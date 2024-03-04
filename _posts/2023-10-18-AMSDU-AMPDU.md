---
layout:     post
title:      A-MSDU & A-MPDU  
subtitle:   
date:       2023-10-18
author:     BuBu
header-img: img/green-background.jpg
catalog: true
tags:
    - 802.11
---

----------
### 1.Acronyms

MMPDU
HC

 

### 2.A-MSDU

The **A-MSDU Present subfield indicates the presence of an A-MSDU**. The A-MSDU Present subfield is set to 1 to indicate that the Frame Body field contains an entire A-MSDU as defined in 9.3.2.2.   
QoS Control field Bit7 A-MSDU present is 1 when A-MSDU is included in an MPDU. Sniffer log will show MPDU frame.

<font color="#FF8C00">AMSDU Present subfield in QoS Control field </font> 
<img src="/img/post/2023-10-18-AMSDU-Present-in-QoS-Control-field.png"/>

<font color="#FF8C00">AMSDU Present subfield in sniffer </font> 
<img src="/img/post/2023-10-18-AMSDU-Present-subfield-in-sniffer.png"/>


**Each MSDU, A-MSDU, or MMPDU is assigned a sequence number**. See 10.3.2.14. Sequence numbers are not assigned to Control frames, as the Sequence Control field is not present in those frames. The Sequence Number field in Data frames indicates the sequence number of the MSDU or A-MSDU.  

<font color="#FF8C00">AMSDU sequence number in sniffer </font> 
<img src="/img/post/2023-10-18-AMSDU-sequence-number-in-sniffer.png"/>

**The TID subfield identifies the TC or TS to which the corresponding MSDU** (or fragment thereof) or A-MSDU in the Frame Body field belongs.    

#### 2.1.A-MSDU operation


The **A-MSDU Supported subfield within the ADDBA Request frame** shall be set to 1 (A-MSDU permitted).  

	ADDBA request filter:wlan.fixed.action_code == 0x00

<font color="#FF8C00">AMSDU Supported subfield within the ADDBA Request frame </font> 
<img src="/img/post/2023-10-18-AMSDU-Supported subfield within the ADDBA Request frame.png"/>

The **A-MSDU Supported subfield is set to 1** by a STA to indicate that it supports an A-MSDU carried in a QoS Data frame sent under this block ack agreement.     
The use of an A-MSDU carried in a QoS Data frame under a block ack agreement is determined for each block ack agreement.   
A STA shall not transmit an A-MSDU within a QoS Data frame under a block ack agreement unless the recipient indicates support for A-MSDU by **setting the A-MSDU Supported field to 1 in its BlockAck Parameter Set field of the ADDBA Response frame**.

	ADDBA response filter:wlan.fixed.action_code == 0x01

<font color="#FF8C00">AMSDU Supported subfield within the ADDBA Response frame </font> 
<img src="/img/post/2023-10-18-AMSDU-Supported subfield within the ADDBA Response frame.png"/>



	
	



### A-MPDU

	Figure 11-26—Block ack setup 

	Block ACK bitmap field represent A-MPDU packed in an PPDU. Start sequence number offset also show the A-MPDU depth. 

### TXOP Duration

	9.2.4.5.7 TXOP Duration Requested subfield
	The TXOP Duration Requested subfield indicates the duration, in units of 32 us, that the sending STA
	determines it needs for its next TXOP for the specified TID. The TXOP Duration Requested subfield is set
	to 0 to indicate that no TXOP is requested for the specified TID in the current SP. The TXOP Duration
	Requested subfield is set to a nonzero value to indicate a requested TXOP duration in the range 32 us
	to 8160 us in increments of 32 s. See 10.23.3.5.2.

	10.23.3.5.2 TXOP requests
	STAs may send TXOP requests during polled TXOPs or EDCA TXOPs using the QoS Control field in a QoS
	Data frame or a QoS Null frame directed to the HC, with the TXOP Duration Requested or Queue Size
	subfield value and TID subfield value indicated to the HC. APs indicate whether they process TXOP request or
	queue size in the QoS Info field in the Beacon, Probe Response, and (Re)Association Response frames. An AP
	shall process requests in at least one format. The AP may reallocate TXOPs if the request belongs to TS or
	update the EDCA parameter set if the above request belongs to TC. A STA shall use only the request format
	that the AP indicates it can process.

	In transmissions under EDCA by a STA that initiates a TXOP, there are two classes of duration settings:
	single protection and multiple protection. In single protection, the Duration/ID field of the frame can set a 
	network allocation vector (NAV) value at receiving STAs that protects up to the end of any following Data,
	Management, or response frame plus any additional overhead frames as described below. In multiple
	protection, the Duration/ID field of the frame can set a NAV that protects up to the estimated end of a sequence of multiple frames. 

### EDCA

	The EDCA mechanism provides differentiated, distributed access to the WM for STAs using eight different UPs. The EDCA mechanism defines four access categories (ACs) that provide support for the delivery of traffic with UPs at the STAs. 