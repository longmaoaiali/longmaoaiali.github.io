---
layout:     post
title:      Airtime fairness      
subtitle:   ATF    
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

### 2.ATF allocation

The Airtime Fairness (ATF) requirement primarily focuses on scheduling fairness for transmission of traffic from Access Point (AP), and efficient Wi-Fi bandwidth utilization. Its algorithm does not deal with transmission of frames from other clients.  

- SSID-based airtime allocation – The airtime management configuration on a per-SSID basis shall be configured as a percentage of available airtime, to be allocated to an SSID or group of SSIDs.  
- Equal airtime allocation – The clients within the BSS should get equal airtime allocation.  
- Unused airtime redistribution – Redistribute unused airtime. The ATF algorithm should be able to share the unused bandwidth from home users to the hotspot user, if there is no traffic in the home network.  
- Per station airtime allocation – This mechanism shall primarily be used to make sure that the STAs are allocated enough bandwidth to perform their respective tasks (such as video streaming).  

First, configure the airtime for a VAP before configuring a airtime for a station (STA). The correct command sequence to assign airtime to a VAP and STA is as follows:  
 
- Assign airtime to VAP  
wlanconfig ath0 addssid vap1 100 // Assigns 100% airtime to VAP1
- Assign airtime to STA  
wlanconfig ath0 addsta 000102030405 10 vap1 //Assign 10% airtime to specified MAC when associated to VAP1


#### 2.1.ATF SSID grouping allocation  

ATF implementation also supports SSID grouping, where groups can be created, with each groups containing an SSID or multiple SSIDs. Airtime can then be assigned to a group, which is distributed within and across groups. The user has the option to select either strict or fair queue scheduling within the group & across the groups. Separate commands are available to configure both Intra-group & Inter-group scheduling policy.   

This section explains the configuration of ATF SSID grouping functionality. BSSIDs can be grouped together to allow a percentage of airtime to be shared across multiple BSSIDs. **Transmit opportunities are allocated based on configurable weight (percentage of airtime)**. All clients in a BSSID group share the airtime percentage allocated to the BSSID group.   

**The following commands are added for ATF SSID grouping configuration:**   
- wlanconfig <vap name> addatfgroup <group name> <ssid> : Creates a group (if it does not exist) and adds ssid to the group created. If the group already exist, adds the ssid to the group.    
- wlanconfig <vap name> configatfgroup <airtime %> : Reserve airtime for a group.  
- wlanconfig <vap name> delatfgroup <group name>: Deletes a group.   
- wlanconfig <vap name> showatfgroup: Displays the group configured.   
- wlanconfig <vap name> showatftable: Displays the group and client-wise airtime allocation.  

#### 2.2.ATF per SSID scheduling policy configuration  

ATF implementation also supports per SSID scheduling policy configuration. The configuration change is to mark/configure any SSIDs that should follow **strict scheduling policy**. By default, all SSIDs will be configured to follow the **fair scheduling policy**. A user must enter a command to configure a particular SSID in Strict mode. This configuration will come into effect only when radio level ATF scheduling policy is set to Fair.  




