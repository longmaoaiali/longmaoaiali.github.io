---
layout:     post
title:      Differences between NSTR and STR  
subtitle:   multicast frame
date:       2023-07-05
author:     BuBu
header-img: img/green-background.jpg
catalog: true
tags: 
    - WLAN Driver
---

----------

### 1.Acronyms

### 2.Management frame

> The AP affiliated with an **NSTR** mobile AP MLD and that is operating on the nonprimary link does not send a **Beacon** frame or respond to a **Probe Request frame**.  


----------

### 3.MLO Multicast(Group addressed Data frame)



#### 3.1.AP MLD duplicate multicast on both link

As per standard AP must duplicate multicast/broadcast packet on both link.  
Impact: Duplicate Rx packet at Station   
Standard Approach: MLD station to be configured to receive multicast/broadcast only one link.  
**802.11be Standard Original Text:**   
> An AP MLD shall use SNS11 in Table 10-5 (Transmitter sequence number spaces) maintained by the MLD to determine the sequence number of a group addressed Data frame that is transmitted by an AP affiliated with the AP MLD so that <font color="#FF0000"> the same group addressed Data frame transmitted over multiple links by the AP MLD uses the same sequence number for transmission on each link.</font>  

> An MLD shall implement RC16 in Table 10-6 (Receiver caches) maintained by the MLD to discard duplicate group addressed Data that are delivered from the associated MLD. <font color="#FF0000">A duplicated group addressed Data frame received on any link shall be discarded.</font> The method used to handle the sequence number wrap around for duplicate detection is implementation specific.     

> 35.3.15 Multi-link operation group addressed frames

----------

#### 3.2.NSTR mobile AP MLD TX group addressed Data frames only on the primary link

>35.3.19 NSTR mobile AP MLD operation  

>An NSTR mobile AP MLD shall designate one of the links of an NSTR link pair as the primary link. The other link of the NSTR link pair is the nonprimary link. When the NSTR mobile AP MLD intends to change the channel/operating class for the primary link, it shall perform (#16959)the channel switch procedure. <font color="#FF0000">The NSTR mobile AP MLD shall schedule for transmissions of Beacon and Probe Response frames and group addressed Data frames only on the primary link. </font>  





