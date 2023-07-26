---
layout:     post
title:      Adaptive Noise Immunity    
subtitle:   ANI    
date:       2023-07-24
author:     BuBu
header-img: img/red-background.jpg
catalog: true
tags:
    - WLAN Driver   
    - Baseband
    - PhyRF  
---

----------
### 1.Acronyms

ANI: Adaptive Noise Immunity    
AIFS: Arbitration Interframe Space   

----------

### 2.ANI

Adaptive Noise Immunity is the algorithm used by the WLAN software to make the **baseband receiver** immune to particular types of noise by dynamically tuning the receive parameters.   

#### 2.1.Theory of operation

ANI is used to **reduce the false packet detect rate** when the WLAN hardware has to operate in the presence of a source of interference such as a spur or another non-WLAN transmitter in the same channel/band.  
The receiver in the WLAN hardware **detects a frame by searching for a repeating sequence (the STFs of the 802.11 frame preamble).** It does this by correlating the time of the received signal with a delayed version of the signal. This is called **weak signal detection**. When a source of interference 
such as a spur or a non-WLAN transmitter is present in the same channel on which the WLAN receiver is receiving frames, the WLAN receiver may detect it as a frame if the interference has periodicity in its signal. **Subsequent receiver checks (such as the LTFs or SIGNAL field) will detect that the interference signal is not a valid WLAN frame and the receiver will restart its receive state machine. This is called a false detection which is recorded as an OFDM or CCK PHY error (hardware can be configured to count PHY errors). Each false detection takes at least 8 µs (duration of the STFs) and may take longer, as it takes time to detect an invalid frame.** The false detection impacts the WLAN performance in two ways:    

- Because the receiver has spent time trying to detect a frame that was actually interference, it is unable to transmit at the same time—so there is a loss of airtime which could have been spent transmitting a frame.  
- The false detection could prevent the reception of a legitimate WLAN frame being transmitted at the same time, or slightly later (during the 8 µs when the WLAN receiver is false detecting).   

Therefore, if the false detects are very frequent (which will be the case if it is a spur or a continuous wave interference), performance will be impacted. As indicated above, both transmission and reception are affected.  

**ANI makes the receiver immune to such sources of interference, by reducing the receiver sensitivity. If the interference signal is sufficiently low power, then it will no longer trigger false detections. If the interference is not detected, the WLAN receiver can receive valid incoming frames or transmit frames.** The drawback of ANI is that it impacts receiver sensitivity; that is, the range. Note that ANI **can only help if the interferer signal power is low enough to be rejected.**    

The environment in which WLAN products must work is unpredictable, and often imperfect. There are noise and interference sources, such as microwave ovens, Bluetooth devices, and emissions from nearby CPUs, which can degrade the performance of our system. There are also noise effects from our own board and chip, such as spurs, which will vary based on channel, board design, and individual board and component variance. These noise sources will impair our ability to receive packets, as would any other noise source. More troubling is that with our most sensitive AGC settings, we may begin to false trigger on these noise sources. This will decrease our possible throughput, since we may miss reception of a packet while processing a false detection, or we may 
delay transmission while the medium is falsely declared busy. However, if we blindly reduce our sensitivity so that we will get fewer false detects, we will be lowering our sensitivity in all cases. For this reason, it makes sense to have an adaptive algorithm to determine the proper amount of noise immunity to use to maximize sensitivity while minimize false detection in possibly changing environments.   

The solution to this problem is to use software to periodically determine if there are excessive false detections, and then adaptively modify the noise and spur immunity parameters in the chip until the false detections fall to a reasonable level. Given that there are multiple changes to be made, we propose that the software use the flowchart in the following figure to guide the changes made.  

ANI timer runs in home channel for every 10 ms (older release uses 100 ms). Each ANI level changes the receive sensitvity by 1 dB. Each PHY error has different weightage. To avoid frequent ANI level variation, the level update done only if there is consecutive errors.

#### 2.2.Implementation  

Dynamic ANI trigger:  

	//USAGE - wifitool ath<0/1> setUnitTestCmd 67 <NO_OF_ARGS> 12 <PHYID> <DL_MIN> <DL_MAX> <ERROR_THRESHOLD>
    //EXAMPLE - wifitool ath0 setUnitTestCmd 67 5 12 0 -5 15 10
	//When error threshold arrive, will trigger ANI level. ANI Level value will between DL_MIN and DL_MAX.
	
	cfg80211tool wifiX ani_ofdm_level
	phyrf_ani_SetAniOfdmImmunLevel() 
		if(value == 0x80) # 
		pAniIniStruct->ani_trigger[phyId] = 1;
		pAniStruct->aniMode = NORMAL_MODE; 

    else if(pAniStruct->aniMode == NORMAL_MODE)
    {
        pAniIniStruct->ani_dl_max[phyID] = phyrf_bdf_GetAniDlMax(pHandle);
        pAniIniStruct->ani_dl_min[phyID] = phyrf_bdf_GetAniDlMin(pHandle);
        pAniIniStruct->error_threshold = phyrf_bdf_GetAniErrorThreshold(pHandle);

        pAniParam->dl_max[phyID] = get_dl_max(pHandle);
        pAniParam->dl_min = phyrf_bdf_GetAniDlMin(pHandle);
        pAniAFStruct->dl_ofdm = phyrf_bdf_GetAniDlMin(pHandle);
        pAniParam->error_threshold = phyrf_bdf_GetAniErrorThreshold(pHandle);

        pAniIniStruct->increment_desense_step_size[phyID] = phyrf_bdf_GetAniIncrementStepSize(pHandle);
        pAniIniStruct->decrement_desense_step_size[phyID] = phyrf_bdf_GetAniDecrementStepSize(pHandle);
        pAniStruct->aniPollPeriod = phyrf_bdf_GetAniPollingPeriod(pHandle);
        pAniStruct->aniListenPeriod = phyrf_bdf_GetAniListenPeriod(pHandle);
	    halphyLog(1,"pAniIniStruct->ani_dl_max[%d]=%d",phyID, pAniIniStruct->ani_dl_max[phyID]);
        halphyLog(1,"pAniIniStruct->ani_dl_min[%d]=%d",phyID, pAniIniStruct->ani_dl_min[phyID]);
        halphyLog(1,"pAniIniStruct->error_threshold=%d",pAniIniStruct->error_threshold);
        halphyLog(1,"pAniStruct->aniPollPeriod=%d",pAniStruct->aniPollPeriod);
        halphyLog(1,"pAniStruct->aniListenPeriod=%d",pAniStruct->aniListenPeriod);
    }

	if (ofdm_phy_err_rate < pAniParam->error_threshold)
	//compare phy error rate with error threshold  
	phyrf_hds_AdtAniOfdmaDesenseLevel(pHandle, &pAniAFStruct->dl_ofdm, NULL);
	//Desense ANI Level

	__HALPHY_ROM__ void phyrfAniDeviceSetAniOfdmDesenseLevel(PHYRF_HANDLE pHandle, A_INT32 level)
	{
		HCA_CMD_VAR_DEF;
		HCA_CMD_BB_VAR_DEF;
	
		HCA_CMD_VAR.pCmd = (void*) &HCA_CMD_BB_VAR;
		HCA_CMD_VAR.id = HCA_CMDID_ANI_SETANIOFDMDESENSELEVEL;
		HCA_CMD_VAR.length = sizeof(HCA_CMD_ANI_SETANIOFDMDESENSELEVEL);
		HCA_CMD_BB_VAR.ani_SetAniOfdmDesenseLevel.pHandle = pHandle;
		HCA_CMD_BB_VAR.ani_SetAniOfdmDesenseLevel.level = level;
		HCA_BB_PROCESS_CMD(&HCA_CMD_VAR);
	}
	
	SetAniOfdmDesenseLevel(bbCmd->ani_SetAniOfdmDesenseLevel.pHandle, bbCmd->ani_SetAniOfdmDesenseLevel.level);
	HcaHwComponentBbQcn9224_ani::SetAniOfdmDesenseLevel(PHYRF_HANDLE pHandle, A_INT32 level) 
	//aniInput.level=level;
	phyAniSetDesenseLevel(phyInput, &aniInput, &aniOutput);
	//PHY team API
	phyDevLib->phyAniSetDesenseLevel()
	
----------




