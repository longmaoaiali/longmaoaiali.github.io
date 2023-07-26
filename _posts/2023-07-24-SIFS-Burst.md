---
layout:     post
title:      SIFS Burst  
subtitle:   SIFS spaced bursting    
date:       2023-07-24
author:     BuBu
header-img: img/red-background.jpg
catalog: true
tags:
    - WLAN Driver   
    - MAC  
---

----------
### 1.Acronyms

SIFS: Short Inter-Frame Space  
AIFS: Arbitration Interframe Space   

----------

### 2.SIFS burst  

SIFS spaced bursting is done by sending AMPDUs in SIFS period following the Block ack from a station. This ensures that medium is occupied by transmission more and increases the MAC efficiency. 

#### 2.1.Theory of operation
This kind of back-to-back transmits within SIFS period can be done for a controlled transmit opportunity (TxOP) period, which is 12 successive AMPDUs (maximum 12 ms) in current implementation. The result of this will be higher throughput over non-SIFS burst in OTA environment. **SIFS still exists, but reduced AIFS and backoff.**  

<font color="#FF8C00"> SIFS Burst Overview:  </font> 
<img src="/img/post/2023-07-24-SIFS-Burst-Overview.png"/>  

#### 2.2.Implementation

The SIFS bursting is implemented in firmware using the `sifs_burst_continuation` bit in the TX PPDU descriptor. The HW MAC transmits the next frame in the HW queue using SIFS bursting if the VMF bit in the descriptor is set.   
The HW MAC ensures that all the frames in a burst transmission belong to same **AC**. If there is no frame in HW queue corresponding to an access category then burst will be broken and MAC will re-contend (AIFS+random backoff) for medium access.   
Currently When SIFS Burst is enabled, AMPDU are limited to a maximum of 2000 us. Also SIFS burst will be done for a maximum consecutive 10 AMPDUs. Once this limit is reached, or if no frames available from same AC, then SIFS burst will be broken by unsetting `sifs_burst_continuation` bit.   
The SIFS bursting can be dynamically enabled or disabled using `burst/get_burst` command.  

----------

### 3.SU multi-destination SIFS bursting feature
SU multi-destination SIFS bursting feature is supported on 802.11ax chipsets.
In earlier generation chipsets, bursting 802.11 PPDUs was supported within SIFS interval but to a single station. In Wi-Fi6 and later chipsets, this feature is extended to do SIFS bursting across multiple stations. SIFS bursting across multiple stations will help reduce collision probability and eliminate the need to do random back off for each PPDU.   

<font color="#FF8C00"> SU multi-destination SIFS bursting:  </font> 
<img src="/img/post/2023-07-24-SU-multi-destination-SIFS-bursting.png"/>   

----------
