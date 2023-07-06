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
