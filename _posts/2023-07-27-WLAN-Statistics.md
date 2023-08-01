---
layout:     post
title:      WLAN statistics       
subtitle:   HTT TLV    
date:       2023-07-24
author:     BuBu
header-img: img/orange-background.jpg
catalog: true
tags:
    - WLAN Driver   
  
---

----------
### 1.Acronyms

ATF: Airtime fairness     

----------

#### AP statistics   

	apstats -a -R

#### Tx and Rx statistics parameters  

	cfg80211tool athX txrx_stats <1-16> Firmware radio-level statistics
	cfg80211tool athX txrx_stats <257-262> Host radio-level statistics
	cfg80211tool wifiX dp_peer_stats <mac_addr>:0x3 Firmware per-peer Statistics
	cfg80211tool wifiX fc_peer_stats <mac_addr> Host per-peer Statistics




Copy engine (CE) latency statistics
CE_TASKLET_DEBUG_ENABLE
hif_ce_state->stats.tasklet_exec_entry_ts[ce_id] =
151 					qdf_get_log_timestamp_usecs();