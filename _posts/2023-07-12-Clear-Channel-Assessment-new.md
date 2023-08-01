---
layout:     post
title:      CCA & CAC     
subtitle:   CCA & CAC   
date:       2023-07-26
author:     BuBu
header-img: img/yellow-background.jpg
catalog: true
tags:
    - PHY 
    - RF Driver     
  
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

> 27.3.20.6.2 CCA sensitivity for operating classes requiring CCA-ED  
For the operating classes requiring CCA-Energy Detect (CCA-ED), the PHY shall indicate a medium busy condition if CCA-ED detects a channel busy condition. For improved spectrum sharing, CCA-ED is required in some bands. The behavior class indicating CCA-ED is given in Table D-2. The operating classes requiring the corresponding CCA-ED behavior class are given in E.1. The PHY of a STA that is operating within an operating class that requires CCA-ED shall operate with CCA-ED.

> CCA-ED for a STA that is attempting a non-preamble **puncturing** transmission shall detect a channel busy condition if the received signal strength exceeds the CCA-ED threshold as given by dot11OFDMEDThreshold for the primary 20 MHz channel, dot11OFDMEDThreshold for the secondary 20 MHz channel (if present), dot11OFDMEDThreshold + 3 dB for the secondary 40 MHz channel(if present), and dot11OFDMEDThreshold **+ 6 dB for the secondary 80 MHz channel** (if present). The CCA-ED thresholds for the operating classes requiring CCA-ED are subject to the criteria in D.2.5.


**clear channel assessment (CCA) function:** The logical function in the physical layer (PHY) that determines the current state of use of the wireless medium (WM).  

**network allocation vector (NAV):** An indicator, maintained by each station (STA), of time periods when transmission onto the wireless medium (WM) is not initiated by the STA regardless of whether the STA’s clear channel assessment (CCA) function senses that the WM is busy.   

	wal_set_cca_threshold()  
	phyrfBdfDeviceSetCCAThresh()  //RF Driver: HCA_CMDID_BDF_SETCCATHRESH 
	SetCCAThresh (rfCmd->bdf_SetCCAThresh.pHandle, rfCmd->bdf_SetCCAThresh.ccaThresh);
	ProgramAGCEnergyDetThr(pHandle, chan); //set channel ED-CCA threshold 
	phyProgAGC(phyInput, &agcInput, NULL); 
	//Different band(2G/5G/6G) and bandwidth have different requirement, fom example puncturing  
	//PHY Device lib
	PHYDEVLIB_API RECIPE_RC phyProgAGC()

----------

### CAC  

Perform Wi-Fi DFS - Channel Availability Check (CAC).

**When a DFS channel is selected for an AP radio, the AP radio scans the channel to check for any radar signals before transmitting any frames in the DFS frequency. This process is called Channel Availability Check (CAC).**   
CAC is executed before you set a DFS channel for the radio.  
If the AP detects that a radar is using a specific DFS channel, the AP marks the channel as non-available and excludes it from the list of available channels. This state lasts for 30 minutes after which the AP checks again to see, if the channel can be used for WiFi transmissions.  

	void dfs_cac_timer_attach(struct wlan_dfs *dfs)
	qdf_timer_init(NULL,&(dfs->dfs_cac_timer),dfs_cac_timeout,(void *)(dfs),QDF_TIMER_TYPE_WAKE_APPS)
	qdf_timer_mod(&dfs->dfs_cac_timer, cac_timeout * 1000)
	static os_timer_func(dfs_cac_timeout)
		/* Send a CAC timeout, VAP up event to user space */
		dfs_mlme_deliver_event_up_after_cac(dfs->dfs_pdev_obj);

----------
