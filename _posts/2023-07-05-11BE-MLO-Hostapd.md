---
layout:     post
title:      11BE MLO Hostapd
subtitle:   MLO Hostapd
date:       2023-07-05
author:     BuBu
header-img: img/yellow-background.jpg
catalog: true
tags:
    - MLO
---

----------



### 1.MLO Demo

#### 1.1.MLO Demo Diagram 

> An example of an AP MLD with two affiliated APs (Link 1 and Link 2) is shown in Figure 4-30a (Example MLD and the affiliated STA communication system). The figure shows an AP MLD with MLD MAC address M and the MLD lower MAC sublayers of two affiliated APs (AP1 with MAC address w and AP2 with MAC address x). The AP MLD is associated with a non-AP MLD with MLD MAC address P and the MLD lower MAC sublayers of two affiliated non-AP STAs (non-AP STA1 with MAC 
address y and non-AP STA2 with MAC address z) are shown. 

> Link 1 is established between AP1 and non-AP STA1 and link 2 is established between AP2 and non-AP STA2. **In general, the MAC address of an MLD and the MAC addresses of the STAs affiliated with the MLD are all different (e.g., M, P, w, x, y, and z have different values). However, the architecture supports an implementation where M could equal either w or x, and where P could equal y or z.**

<font color="#FF8C00">Example MLD communication system:  </font> 
<img src="/img/post/2023-07-07-Example-MLD-communication-system.png"/> 

----------

#### 1.2.MLO Demo AP Side

<font color="#FF8C00">AP MLD Link info:  </font> 
<img src="/img/post/2023-07-07-AP-MLO-Demo-info.png"/>
  
 
<img src="/img/post/2023-07-07-AP-MLO-Link-info.png"/>

----------

#### 1.3.MLO Demo STA Side

<font color="#FF8C00">STA MLD Link info:  </font> 
<img src="/img/post/2023-07-07-STA-MLO-Demo-info.png"/>
  
 
<img src="/img/post/2023-07-07-STA-MLO-Link-info.png"/>

----------

### 2.MLO Hostapd 

----------

#### 2.1.Script Code to ADD MLO BSS
	wpa_cli -g /var/run/hostapd/global raw ADD bss_config=ath1:/var/run/hostapd-ath1.conf 
	
	#cat /var/run/hostapd-ath1.conf
	driver=nl80211
	interface=ath1
	logger_syslog=127
	logger_syslog_level=2
	logger_stdout=127
	logger_stdout_level=2
	channel=52
	ieee80211ac=1
	ieee80211n=1
	ieee80211ax=1
	ieee80211be=1
	eht_oper_chwidth=0
	eht_oper_centr_freq_seg0_idx=52
	ht_capab=[LDPC][SMPS-DYNAMIC][TX-STBC][RX-STBC-1][MAX-AMSDU-7935][DSSS_CCK-40] [SHORT-GI-20]
	vht_capab=[MAX-MPDU-11454][VHT160][RXLDPC][SHORT-GI-80][SHORT-GI-160][TX-STBC-2BY1][RX-STBC1][SU-BEAMFORMER][SOUNDING-DIMENSION-2][SU-BEAMFORMEE][MAX-A-MPDU-LEN-EXP7][MU-BEAMFORMER][RX-ANTENNA-PATTERN][TX-ANTENNA-PATTERN]
	hw_mode=a
	wmm_enabled=1
	dtim_period=1
	ignore_broadcast_ssid=0
	mld_link_macs=00:03:7f:12:59:d7 00:03:7f:12:f4:a7
	mld_link_ids=1 2
	mld_mac_addr=00:0A:F5:22:12:31
	owe_ptk_workaround=1
	ctrl_interface=/var/run/hostapd-wifi1
	send_probe_response=0
	wpa_passphrase=12345678
	auth_algs=1
	wpa=2
	wpa_pairwise=CCMP
	ssid=gate
	bridge=br-lan
	ieee80211w=0
	wpa_key_mgmt=WPA-PSK
	wps_cred_add_sae=0
 
#### 2.2.How does hostapd know this is creating an MLO BSS?

These BSS configuration options will update to beacon frame.  

	mld_link_macs=00:03:7f:12:59:d7 00:03:7f:12:f4:a7  //MLD affiliated link mac address 
	mld_link_ids=1 2                //MLD affiliated link ID
	mld_mac_addr=00:0A:F5:22:12:31  //MLD mac address

----------

#### 2.3.MLD How to select Primary Link?  

In the 802.11 protocol, only NSTR mentions the primary link.  

>35.3.19 NSTR mobile AP MLD operation  
An NSTR mobile AP MLD shall designate one of the links of an NSTR link pair as the primary link. The other link of the NSTR link pair is the nonprimary link. When the NSTR mobile AP MLD intends to change the channel/operating class for the primary link, it shall perform (#16959)the channel switch procedure. The NSTR mobile AP MLD shall schedule for transmissions of Beacon and Probe Response frames and group addressed Data frames only on the primary link. 

<font color="#FF0000"> Approach:  
Use the assoc link as the primary link, STA MLD decides which link to connect and configure the primary umac. When AP MLD receive Auth frame, AP MLD will configure the primary umac on assoc link.</font>  

**<font color="#FF0000">STA Side wpa_supplicant log: </font>** 

	# From scan result select BSS: _wpa_supplicant_event_scan_results 
	# Select Candidate MLO BSS: wpa_mlo_common_candidate_selection
	ath21: Radio work 'scan'@0x559fb4dfc0 done in 4.426161 seconds
	ath21: radio_work_free('scan'@0x559fb4dfc0): num_active_works --> 0
	ath21: ML_STA: All Link Scan Done, going for MLD Connect
	ath21: ML_STA: MLO Candidate Selection - START
	ath21: ML_STA: Is a 2 link ML-STA
	ath21: Selecting BSS from priority group 0
	ath21: 0: 00:03:7f:12:f4:a7 ssid='gate' wpa_ie_len=0 rsn_ie_len=20 caps=0x511 level=-28 freq=5520 
	ath21: CTRL-EVENT-SSID-REENABLED id=0 ssid="gate"
	ath21:  ML_STA: ie.pairwise=16,ie.group_cipher=16, ie.key_mgmt=2, ie.capabilities=12
	ath21:    selected based on RSN IE
	ath21:    selected BSS 00:03:7f:12:f4:a7 ssid='gate'
	ath21: ML_STA: MLO Candidate Selection - END
	ath21: ML_STA: Considerig MLO Connect selected network is MLD AP
	ath21: ML_STA: Found on 2 interfaces
	ath21: ML_STA: reassociat=0 wpa_s->wpa_state=3
	ath21: ML_STA: Start ML RNR-IE parsing to map wdev to link-id
	ath21: ML_STA: Completed ML RNR-IE Parsing
	ath21: ML_STA: End ML RNR-IE parsing to map wdev to link-id
	ath21: Link-ID = 2
	ath21: Considering connect request: reassociate: 0  selected: 00:03:7f:12:f4:a7  bssid: 00:00:00:00:00:00  pending: 00:00:00:00:00:00  wpa_state: SCANNING  ssid=0x559fb48700  current_ssid=0
	ath21: Request association with 00:03:7f:12:f4:a7
	ath21: ML_STA: Update Assoc-Link wpa_sm
	ath21: ML_STA: Update Partner Links to ASSOCIATING State	
	ath21: ML_STA: Update Partner Links as MLO_LINK_DEVICE
	ath21: Starting radio work 'connect'@0x559fb4df60 after 0.000557 second wait
	#Mark current link is primary link: selected_wpa_s->primary_link = true;

**<font color="#FF0000">STA Side Host Driver log: </font>** 

	# NL80211_CMD_CONNECT wpa_driver_nl80211_try_connect / nl80211_connect  
	# 00:03:7f:12:f4:a7 is the mac address of an AP affiliated with an AP MLD
	# 00:0a:f5:22:12:31 is the mac address of an AP MLD
	wlan: [2231:I:ANY] wlan_cfg80211_connect: DES SSID SET=gateY
	wlan: [2231:I:ANY] wlan_cfg80211_connect: DES BSSID SET=00:03:7f:12:f4:a7
	wlan: [2231:I:ANY] osif_update_op_class_from_mlie:  Link Op-class update in app-ie buffer
	wlan: [0:I:ANY] [NODE] vap-1(ath21):ieee80211_setup_node forcing sta to associate in 33 mode
	wlan: [0:I:MLO_MGR] mlo_peer_allocate_primary_umac: MLD ID 0 ML Peer 00:0a:f5:22:12:31 primary umac soc 1 
	wlan: [0:I:MLO_MGR] wlan_mlo_peer_create: MLD ID 0 ML Peer 00:0a:f5:22:12:31 allocated 1f39421c
	wlan: [0:I:CMN_MLME] vdev 1 cm_id 0xc010001: Connecting to gate 00:03:7f:12:f4:a7 rssi: -48 freq: 5745 akm 0x2 cipher: uc 0x8 mc 0x8, wps 0 osen 0 force RSN 0 CC:

**<font color="#FF0000">AP Side Host Driver log: </font>** 

	#06:03:7f:01:59:56 is the mac address of an STA affiliated with an STA MLD
	#00:04:3d:11:4d:22 is the mac address of an STA MLD
	wlan: [0:I:ANY] [AUTH] vap-0(ath2): [06:03:7f:01:59:56]recv auth frame with algorithm 0 seq 1
	wlan: [0:I:ANY] [ASSOC] vap-0(ath2): [06:03:7f:01:59:56]Received assoc req
	wlan: [0:I:MLO_MGR] mlo_ap_ml_peerid_alloc:  ML peer id 2 is allocated
    wlan: [0:I:MLO_MGR] mlo_peer_allocate_primary_umac: MLD ID 0 ML Peer 00:04:3d:11:4d:22 avg RSSI -50 primary umac soc 1 
	wlan: [0:I:MLO_MGR] wlan_mlo_peer_create: MLD ID 0 ML Peer 00:04:3d:11:4d:22 allocated 6bf2ca69  

#### 2.4.Which link EAPOL frame will be sent out?

<font color="#FF0000"> Assoc link, also primary link. </font>    
>A non-AP MLD shall perform frame exchanges during the authentication, (re)association, and 4-way handshake procedures only on the primary link of the NSTR mobile AP MLD.  

----------


#### 2.5. MLO with 2 links, one of links is disconnected, what is expected behavior? 
Spec has not defined to fallback to SLO by tearing down the MLO link nodes or continue MLO with just one link active. Both approaches have pros and cons.   
  
- Approach 1: Tear down MLO and go to SLO. If the client device moves close, we will still be SLO unless the upper layer initiates an MLO connection. 
- Approach 2: Continue MLO with single link active. We carry overhead of maintaining and logistics in software (ex: ml_peer_ids, peer management, rx reordering) for MLD link even though the user is single link active and we don’t know when STA will come back to range for MLO. 

Currently we are with approach 1 to tear down MLO and stay with legacy connection.

----------
