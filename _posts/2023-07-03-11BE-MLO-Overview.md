---
layout:     post
title:      11BE MLO Overview
subtitle:   MLO Overview
date:       2023-07-03
author:     BuBu
header-img: img/orange-background.jpg
catalog: true
tags:
    - 11BE EHT
    - WLAN Driver
    - MLO
    - IEEE802.11 be
---

----------
### 1.Acronyms

**MLD**: Multi-Link Device.  
**MLO**: Multi-Link Operation. MLO defines a set of procedures allowing communication over (#18068)one or more links between MLDs. An MLD manages such communication over (#18068)one or more links. Communication across links <font color="#FF0000">using different frequency bands or channels </font> can occur simultaneously or not depending on the capabilities of both the AP MLD and the non-AP MLD.    
**NSTR**: Nonsimultaneous transmit and receive (NSTR) operation.  
**STR**: Simultaneous transmit and receive (STR) operation.  
**WM**: wireless medium.   
**EMLMR**: enhanced multi-link multi-radio.  
**EMLSR**: enhanced multi-link single radio.  
**MME**: MLD Management Entity.  

----------

### 2.Concept of MLO

#### 2.1.What is MLO? 

Multi-Link Operation (MLO) enables data flows to be sent across multiple OTA channels to a Multi-Link device.  
Communication across links <font color="#FF0000">using different frequency bands or channels </font>.  
<font color="#FF8C00">MLO Block Diagram:  </font> 
<img src="/img/post/2023-07-03-MLO-Sample.png"/>

<font color="#FF8C00">MLO Block Diagram Packet Level:  </font> 
<img src="/img/post/2023-07-04-MLO-Block-Diagram-Packet-Level.png"/>

----------

#### 2.2.Benefits of MLO 

- Provides reduced latency through opportunistic channel selection.  
- Increased peak throughput: Packets of the same flow can be sent over multiple links.  
- Improved latency: Due to increased channel access opportunities on multiple links.

#### 2.3.MLO architecture  

Before the implementation of MLO, MAC service data units (MSDUs) belonging to the same traffic flow cannot be transmitted across different bands. As a result, stations are tied to a single band, preventing a dynamic and seamless inter-band operation. That is, one link is selected between transmitter and receiver to carry out data exchanges, remaining the other unused.   

To enable a concurrent operation across multiple interfaces, 11be introduces the concept of multi-link capable device (MLD), which consists of a single device with multiple wireless PHY interfaces that provides a unique MAC instance to the upper layers.   

In other words, the upper-layer protocols consider the MLD as a single device. Despite having multiple physical radio interfaces, MLD has a single MAC address, and the sequence numbers are generated uniquely from the same sequence number space.   
This solution simplifies fragments and packets reassembly, duplication detection, and dynamic link switching. They allow packet retransmission on any link regardless of the link of the initial transmission of the packet.   

**The unified upper MAC (UMAC) performs link agnostic operations such as A-MSDU aggregation/de-aggregation and sequence number assignment. It implements a queue that buffers the traffic received from the upper-layers. That is, traffic awaits in the UMAC before it is assigned to a specific interface to be transmitted. Also, the UMAC provides some management functions such as multi-link setup, association and authentication.**    

**The independent lower MAC (LMAC) performs link specific functionalities. There, we find the link specific EDCA queues (one for each access category) that hold the traffic until its transmission. Also, procedures such as MAC header and cyclic redundancy check (CRC) creation/validation, in addition to management (e.g., beacons) and control (e.g., RTS/CTS and ACKs) frame generation are implemented at the LMAC.**  

An MLD has a single MLD MAC address that identifies the MLD Management Entity (MME). Each STA (MAC/PHY instance) has its own per-link attributes.  
**MME (Upper MAC)**  
 Single interface to upper layer  
 MSDU Buffering, Re-sequencing, replay-attack check  
 Single BA session: Shared sequence # space per TID, tracks BA score-board  
 Security: Authentication, Shared PN# per TID across links  
 Management signaling, Association, AID assignment  
**STA /AP (Lower MAC)**   
 Channel contention (CCA, EDCA Back-off)    
 Time critical control responses (Acknowledgments, CTS)    
 Power-save states (Doze, Awake)    
 Fragmentation, encryption, message integrity-check    
 PPDU formation and transmission (MCS, SS#, PPDU duration)   
**Beacon is sent per link basis**  
 All power save and other things are taking place in per-link basis. So, beacons are sent on per-link basis.  
 For the discovery, you need to discover the whole multi-link device. So, the beacons in one link need to carry basic info about the other links, so that other mechanisms such as probe request/response or listening on the other channels for the other beacons may be used to discover complete info about all the links.  
 
----------

### 3.MLO modes
 
#### 3.1.STR (simultaneous Tx and Rx)

An MLD that is capable of simultaneous Tx /Rx on multiple links for the given set of links.  
<font color="#FF8C00">STR mode:  </font> 
<img src="/img/post/2023-07-04-MLO-STR.png"/>

----------

#### 3.2.Non-STR (NSTR)

An MLD that is not capable of simultaneous Tx /Rx on multiple links for the given set of links, it can only do Tx/Tx or Rx/Rx on all links.  
<font color="#FF8C00">NSTR mode:  </font> 
<img src="/img/post/2023-07-04-MLO-NSTR.png"/>

----------

#### 3.3.Multi-link channel access mode(Transmission Modes)

Three modes of transmission under considerations: Basic mode, Asynchronous mode, Synchronous mode.

##### 3.3.1.Basic Mode

Basic (Multi-Primary with single link transmission): STA/AP counts down on both links. Transmits only on the link that wins the medium. Other link 
gets blocked by in-device-interference > -62dBm.
A STA that is affiliated with an MLD shall contend for the WM on its link independently from the other STA(s) affiliated with the same MLD, unless explicitly stated otherwise in the sub clauses below.

##### 3.3.2.Asynchronous Mode

Asynchronous mode: STA/AP counts down on both links. PPDU start/end can happen independently on each link. Possible when the device can support simultaneous Tx/Rx.

##### 3.3.3.Synchronous Mode

Synchronous PPDU: To avoid that a device transmit and receive frames on multi-link simultaneously, it synchronizes the ending times of the PPDU 
transmissions on multi-link.   
There are two types Sync PPDU with PIFS Access and Sync PPDU without PIFS Access, depending on regulatory rule.

PIFS access allowed. For fairness reasons, ED(energy detection) on 2nd link is checked at -72dBm. <font color="#FF8C00">Sync PPDU with PIFS Access:  </font> 
<img src="/img/post/2023-07-04-Sync-PPDU-with-PIFS-Access.png"/>

PIFS access NOT allowed. Start times may be different. End time is aligned. <font color="#FF8C00">Sync PPDU without PIFS Access:  </font> 
<img src="/img/post/2023-07-04-Sync-PPDU-without-PIFS-Access.png"/>

##### 3.3.4.Why do we need Sync PPDU?

Multi-link maybe be different channels same bands, it will have self link interference. 11be protocol needs to be able to handle bi-directional traffic. However, when bi-directional traffic is present, the UL traffic would wipeout a significant portion of the DL.

<font color="#FF8C00">Pitfalls of No Sync PPDUs:  </font> 
<img src="/img/post/2023-07-04-Pitfalls-of-No-Sync-PPDUs.png"/>

----------

#### 3.4.MLO Variations

#### 3.4.1.Async MLMR/STR MLMR(Asynchronous Multi-Link Multi-Radio)  
Two MAC, Two PHY → each link contends for channel access independently and updates the common queue after transmission.  

<font color="#FF8C00">Async MLMR:  </font> 
<img src="/img/post/2023-07-04-Asynchronous-Multi-Link-Multi-Radio.png"/>


#### 3.4.2.Sync MLMR/NSTR MLMR(Synchronous Multi-Link Multi-Radio)  
Two MAC, Two PHY → each link contends for the channel and updates the common queue after transmission. Requires time synchronization with coordinated transfers across both bands to eliminate self-interference.   

<font color="#FF8C00">Sync MLMR:  </font> 
<img src="/img/post/2023-07-04-Synchronous-Multi-Link-Multi-Radio.png"/>

#### 3.4.3.MLSR(Multi-Link Single Radio)  
Device has configured datapath and MAC for two channels with a tunable radio that can listen and transmit on either band, but only one at a time. 
Key benefit is reduced cost for client devices. AP must support interoperability but generally has both radios active. 

#### 3.4.4.eMLSR(Enhanced Multi-Link Single-Radio)  
Device has configured datapath and MAC for two channels with a tunable radio that can listen on both channels and can transmit on either band  at a time.  

<font color="#FF8C00">MLSR & eMLSR:  </font> 
<img src="/img/post/2023-07-04-MLSR-eMLSR.png"/>

<font color="#FF8C00">eMLSR Operation:  </font> 
<img src="/img/post/2023-07-04-eMLSR-Operation.png"/>

#### 3.4.5.Test Result of different variations of MLO
<font color="#FF8C00">Test Result of different types of MLO:  </font> 
<img src="/img/post/2023-07-03-Type-of-MLO-Test.png"/>

----------

### 4.MLO Key feature  

- Packet Level Aggregation: Packets of the same TID can be sent on one or more radios. Helps with low-latency and peak throughput improvement.   

- Cross-wake-up signaling for power-save: AP indicates buffered unit indication of other links in a link that STA is monitoring. STA can indicate wake-up of a link using another link. STA can monitor a link in idle mode to get BSS/TIM information of other links.  

- Fast link-transition: The active link(s) can be switched dynamically to adapt to load /co-ex conditions. Beneficial for 11be single-radio STAs.  

- Multi-primary channel access: Needed for latency improvements.  

- Shared Single Session across links: Single Block-ack session per TID, shared sequence number space. Single authentication and key derivation for unicast packets. Separate group-keys for broadcast /groupcast packets.  

----------

### 5.MLO MAC Layer Architecture

MLO MAC Layer Consists of two logical entities:
    
- MME (MLD Management Entity Upper MAC): Responsible for non-time critical functions  
- STA /AP (Lower MAC): Responsible for time critical functions  

<font color="#FF8C00">MLO MAC Layer Architecture:  </font> 
<img src="/img/post/2023-07-04-MLO-MAC-Layer-Architecture.png"/>  

----------

#### 5.1.MLD Management Entity Upper MAC

- Single interface to upper layer  
- MSDU Buffering, Re-sequencing, replay-attach check  
- Single BA session: Shared sequence # space per TID, tracks BA score-board  
- Security: Authentication, Shared PN# per TID across links  
- Management signaling, Association, AID assignment  

----------

#### 5.2.Lower MAC  

- Channel contention (CCA, EDCA Back-off etc.)  
- Time critical control responses (Acknowledgements, CTS etc.)  
- Power-save states (Doze, Awake)  
- Fragmentation, encryption, message integrity-check  
- PPDU formation & transmission (MCS, SS#, PPDU duration etc.)  

----------

#### 5.3.Beacon is sent per link basis

- All power save and other things are taking place in per-link basis. So, beacons are sent on per-link basis.  
- For the discovery, you need to discover the whole multi-link device. So, the beacons in one link need to carry the info about the other link.  

----------

### 6.BA procedure with multi-link

Independent BA response on each link.  

In multi-link operation, MPDUs are assigned from a common queue to each STA for transmission. A STA of an MLD may not have knowledge of the MPDUs owned by another STA for transmission.  
On the Rx-side, a STA may not have knowledge of the most recently received MPDUs by another STA of the MLD. There can be a lag in updating the MPDU Rx information cross STAs of an MLD.  
Furthermore, MPDUs may be in transit on one link when BA is being sent on another link.Therefore, a transmitting STA can accurately determine the Rx status only for MPDUs that it had transmitted on its own link. A value 1 in the BA Bitmap indicates the MPDU was successfully received.  

----------

#### 6.1.Rx Side BA procedure

Recipient STA of an MLD provides receive status for MPDUs received on the link that it is operating on. BA for an eliciting PPDU is sent on the same link that it received the PPDU.  
Additionally a recipient STA of an MLD may provide receive status for an MPDU successfully received by another STA of that MLD. Indicating successful Rx status on behalf of another link can help the originator quickly advance the transmit window, and can also help in scenarios where a BA was lost.  

----------

#### 6.2.Tx Side BA procedure

The transmitting MLD consolidates (ORs) the BA reports from different STAs to update the common BA scoreboard and make decisions regarding. Advancing the Tx window and Retransmitting failed MPDUs.   
<font color="#FF8C00">BA procedure with multi-link:  </font> 
<img src="/img/post/2023-07-04-BA-procedure-with-multi-link.png"/>  

----------

#### 7.Workflow of MLO operations  

- Multi-Link Discovery: AP MLD advertises Cap and Op parameters for each link in beacons and probe responses (subject to optimizations to reduce bloat – work in progress).  
- Multi-Link Setup: Initiated by non-AP MLD with an exchange of Cap and Op parameters for all links, negotiation of some parameters. A single association context is used across multiple links.  
- Security: Common PTK across links for unicast and per-link GTK for multicast/broadcast.   
- Multi-link BA setup: Set up of common BA session for a TID across multiple links with mapping of TID to links.  
- Remapping of TIDs: Perform ADDBA to remap a TID to a different (set of) link(s)  

----------

#### 8.TID-to-Link Mapping infrastructure  

Compliance with IEEE 802.11be draft version 2.1 has been implemented for T2LM.   
Support is introduced for broadcast advertisement of T2LM in beacon/probe response to announce that a link is temporarily unavailable for all the non-AP STAs associated with the unavailable AP. Both **peer-to-peer TID-to-link (T2LM) **mapping negotiation and broadcast T2LM negotiation are supported.  

The TID-to-link mapping mechanism allows an AP MLD and a non-AP MLD that performed or are performing multi-link setup to determine how UL and DL Qos traffic corresponding to TID values between 0 and 7 will be assigned to the setup links for the non-AP MLD.  


##### 8.1.Negotiation of TID-to-link mapping (T2LM)    

 An MLD that supports TID-to-link mapping negotiation shall set to a nonzero value the TID-to-link Mapping Negotiation Support subfield in the MLD Capabilities and Operations field of the Basic Multi-Link element that it transmits.   
 During a multi-link (re)setup procedure, a non-AP MLD may initiate a TID-to-link mapping negotiation by including the TID-to-link Mapping element in the (Re)Association Request frame if an AP MLD has indicated a support of TID-to-link mapping negotiation.  
 After the multi-link (re)setup is successful and 4-way handshake is complete (if RSNA is required), to negotiate a new TID-to-link mapping, an initiating non-AP MLD shall send an individually addressed TID-to-link Mapping Request frame to a responding MLD that has indicated support of TID-to-link mapping negotiation.    
 An AP MLD that initiates a TID-to-link mapping negotiation may perform one of the following:  
a. Send an individually addressed TID-to-link Mapping Request frame to a non-AP MLD  
b. Advertise a TID-to-link mapping by including a TID-To-Link Mapping element in Beacon and Probe Response frames.  

----------

#### 9.Provision MLD  

RM is new user application that provides improved network management toolbox by using an underlying protocol of MLO and enhanced admission control per band. It uses TID-to-Link Map (T2LM) protocol and telemetry to provision MLD clients.   

The Resource Manager user application will be responsible for processing periodic statistics and running link provisioning algorithms. It will inform link provisioning decisions via cfg80211 Netlink commands to Wi-Fi driver.

RM decision making algorithms   
Resource manager’s provisioned-MLO service will trigger link provision in two scenarios:   
1. when new MLO capable client associates   
2. when congestion metrics crosses threshold on specific link   
**Association handling—Handling of client association in RM app.** When new MLO capable client associates, available capacity for all links is computed based on client and link capabilities. If client has not requested any specific link in its association request, RM will provision this client on highest capacity link via T2LM message.  
**Load Balancing or Network Optimization** handling—Periodic stats received from Telemetry agent will be used to evaluate if any links have crossed Congestion thresholds. Once congestion threshold is crossed, load balancing algorithm will be triggered to perform link load balancing.  

	Enable resource manager application and PMLO service
	uci set rsrcmgr.config.Enable=1
	uci set rsrcmgr.pmlo.Enable=1
	uci set rsrcmgr.config.Respawn=1
	uci commit

