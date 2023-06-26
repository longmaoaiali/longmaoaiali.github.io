---
layout:     post
title:      WFA认证测试问题梳理
subtitle:   Wi-Fi Alliance CERTIFIED Test
date:       2023-06-16
author:     BuBu
header-img: img/blue-background.jpg
catalog: true
tags:
    - WFA
    - WLAN Driver
    - WiFi
    - IEEE802.11
---

>WFA认证测试问题梳理




# Wi-Fi6 11AX

#### WFA HE-4.41.1

	Test Item：4.41.1 Basic HE UL MU frame exchange sequence for 2.4 GHz, 20 MHz test
	Applicability: Mandatory
	Issue: How to Configure APUT to transmit M-BA in SU PPDU as the response of frames in TB PPDU During?
	
	Refer to Wi-Fi6_CERTIFIED_6_Test_Plan_v1.0_0.pdf
	M-BA implies Basic Trigger to solicit HE TB PPDU from the STAs.

	Refer to 802.11ax-2021.pdf
	Trigger frames where the Trigger Type field refer to "Table 9-29c—Trigger Type subfield encoding"

	APUT API: ath_set_trigger_type_0
	wifitool athX setUnitTestCmd 0x47 2 43 6

	#define WAL_SCHED_TX_MODE_MAX 13 /* 11be modes: MU_MIMO_BE/MU_OFDMA_BE/MUMIMO_OFDMA_BE/UL_MUMIMO_OFDMA_BE/UL_MUMIMO_OFDMA_BSR_TRIG */ 
	#define WAL_SCHED_TX_MODE_MAX 8  /* SCHED_TX_MODE_MAX SU/ MUMIMO AC/ MU MIMO AX/ MU OFDMA/ UL MUMIMO AX/ UL MU OFDMA AX/ UL BSR TRIG */
	
	"name2val": {
		"SCHED_TX_MODE_MAX": 8,
        "SCHED_TX_MODE_MU_MIMO_AC": 1,
        "SCHED_TX_MODE_MU_MIMO_AX": 2,
        "SCHED_TX_MODE_MU_OFDMA_AX": 3,
        "SCHED_TX_MODE_NTBR": 7,
        "SCHED_TX_MODE_SU": 0,
        "SCHED_TX_MODE_UL_BSR_TRIG": 6,
        "SCHED_TX_MODE_UL_MU_MIMO_AX": 4,
        "SCHED_TX_MODE_UL_MU_OFDMA_AX": 5
    },
	
	NTBR (non-TB ranging): A ranging measurement procedure that uses NDP, and is not initiated by a Ranging Trigger frame, refer to 802.11az-2022.pdf





