---
layout:     post
title:      CCA & CAC     
subtitle:     
date:       2023-07-26
author:     BuBu
header-img: img/yellow-background.jpg
catalog: true
tags:
    - PHY    
  
---

----------
### 1.Acronyms

CAC: Channel available check  
CCA: Clear Channel Assessment   
CS/CCA: Carrier sense/clear channel assessment  
CCA-ED: CCA-Energy Detect  
NAV: network allocation vector  


----------

### CCA

Refer to `802.11-2016.pdf`.

> The **DSSS PHY** shall provide the capability to perform CCA according to both:   
    — CCA Mode 1: Energy above threshold. CCA shall report a busy medium upon detection of any energy above the ED threshold.  
	— CCA Mode 2: CS only. CCA shall report a busy medium only upon detection of a DSSS signal. This signal may be above or below the ED threshold.  
	— CCA Mode 3: CS with energy above threshold. CCA shall report a busy medium upon detection of a DSSS signal with energy above the ED threshold.  
	— CCA Mode 6: CCA shall report a busy medium upon detection of any energy above –62 dBm.  

> A busy channel shall be indicated by issuing a PHY-CCA.indication(BUSY) primitive.   
> A clear channel shall be indicated by issuing a PHY-CCA.indication(IDLE) primitive.
     
  
>The **HR/DSSS PHY** shall provide the capability to perform CCA according to both:  
— CCA Mode 1: Energy above threshold. CCA shall report a busy medium upon detecting any energy above the ED threshold.  
— CCA Mode 4: CS with timer. CCA shall start a timer whose duration is 3.65 ms and report a busy medium only upon the detection of a HR/DSSS PHY signal. CCA shall report an IDLE medium after the timer expires and no HR/DSSS PHY signal is detected. The 3.65 ms timeout is the duration of the longest possible 5.5 Mb/s PSDU.   
— CCA Mode 5: A combination of CS and energy above threshold. CCA shall report busy at least while a HR/DSSS PPDU is being received with energy above the ED threshold at the antenna.
— CCA Mode 6: CCA shall report a busy medium upon detection of any energy above –62 dBm.  

> A busy channel shall be indicated by issuing a PHY-CCA.indication(BUSY) primitive. A clear channel shall be indicated by issuing a PHY-CCA.indication(IDLE) primitive.  

> The PHY shall indicate a medium busy condition by issuing a PHY-CCA.indication primitive when the carrier sense/clear channel assessment (CS/CCA) mechanism detects a channel busy condition.   
For the operating classes requiring CCA-Energy Detect (CCA-ED), the PHY shall also indicate a medium busy condition when CCA-ED detects a channel busy condition

**clear channel assessment (CCA) function:** The logical function in the physical layer (PHY) that determines the current state of use of the wireless medium (WM).  

**network allocation vector (NAV):** An indicator, maintained by each station (STA), of time periods when transmission onto the wireless medium (WM) is not initiated by the STA regardless of whether the STA’s clear channel assessment (CCA) function senses that the WM is busy.   

----------

### CAC  

Perform Wi-Fi DFS - Channel Availability Check (CAC)

