---
layout:     post
title:      Clear Channel Assessment  
subtitle:   CCA
date:       2023-07-12
author:     BuBu
header-img: img/indigo-background.jpg
catalog: true
tags: 
    - WLAN PHY
---

----------

### 1.Acronyms

**CCA**: Clear Channel Assessment  

----------



CCA Count Statistics
wlan_proc/wlan/mac_core/src/wal/AR/dev/wal_phy_dev_hw.c
wal_dev_hw_collect_cca_stats()
wal_pdev_get_cca_counters()
whalGetCCACounters()
p_cca_counter->rxFrameUsec = HWIO_INXI(LMAC_HWSCH_BASE(p_whal_mac_struct), HWSCH_R0_LOW_POWER_CNTR_n, 9);