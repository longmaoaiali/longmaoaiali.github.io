---
layout:     post
title:      TID-to-link mapping  
subtitle:   Select MLO link to TX  
date:       2023-10-13
author:     BuBu
header-img: img/blue-background.jpg
catalog: true
tags:
    - WFA
---

----------
### 1.Acronyms

T2LM: TID-to-link mapping


### 2.TID-to-link mapping  

	MLO TID-to-link mapping controls which link an MPDU can be transmitted.  
	Indicates links on which frames belonging to each TID can be exchanged.  

	TID-to-link mapping negotiation
	TID-to-link mapping operation (35.3.7.1 (TID-to-link mapping))  
	TID-To-Link Mapping (see 9.4.2.314 (TID-To-Link Mapping element))   
 

	9.4.2.314 TID-To-Link Mapping element  
	Element ID:255, Element ID Extension:109   
	The TID-To-Link Mapping element indicates links on which frames belonging to each TID can be exchanged. 
	The format of the TID-To-Link Mapping element is shown in Figure 9-1002ao (TID-To-Link Mapping element format).


	Figure 35-15—Example TID-to-link mapping frame exchange  
	35.3.7.1.8 Association procedures for TID-to-link mapping 
	
	default mapping (all TIDs mapped to all setup links),


### 3.Dynamic link transitions

	35.3.7.2 Dynamic link transitions

	A non-AP MLD may use the power states of its affiliated non-AP STAs (see 35.3.12 (Multi-link power management)) to dynamically change the link(s) on which it operates. 
	Figure 35-16 (Example of link transition operation by a single radio non-AP MLD using power states) provides an illustration of operation of a single radio non-AP MLD with default mapping (all TIDs mapped to all setup links), 
	where the non-AP MLD transitions from operating on link 1 with non-AP STA 1 to operating on link 2 with non-AP STA 2, where both non-AP STA 1 and non-AP STA 2 are affiliated with the non-AP MLD.



**The TID subfield identifies the TC or TS to which the corresponding MSDU** (or fragment thereof) or A-MSDU in the Frame Body field belongs.    
The TID subfield also identifies the TC or TS of traffic for which a **TXOP is being requested**, through the setting of TXOP duration requested or queue size.   
**The encoding of the TID subfield depends on the access policy** (see 9.4.2.29) and is shown in Table 9-12.   