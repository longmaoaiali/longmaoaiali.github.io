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

#### How to Configure APUT to transmit M-BA in SU PPDU as the response of frames in TB PPDU During WFA HE-4.41.1.

	Test Item：4.41.1 Basic HE UL MU frame exchange sequence for 2.4 GHz, 20 MHz test
	Applicability: Mandatory
	
	Refer to Wi-Fi6_CERTIFIED_6_Test_Plan_v1.0_0.pdf
	M-BA implies Basic Trigger to solicit HE TB PPDU from the STAs.

	Refer to 802.11ax-2021.pdf
	Trigger frames where the Trigger Type field is Basic Trigger, MU-BAR Trigger, BQRP Trigger or BSRP Trigger.

	APUT API: ath_set_trigger_type_0
	wifitool athX setUnitTestCmd 0x47 2 43 6




