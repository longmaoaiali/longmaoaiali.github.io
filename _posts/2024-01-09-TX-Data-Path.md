---
layout:     post
title:      WLAN Firmware       
subtitle:   Firmware    
date:       2024-01-09
author:     BuBu
header-img: img/yellow-background.jpg
catalog: true
tags:
    - WLAN Firmware   
  
---

----------
### 1.Acronyms

**QCU**: Queue control unit   
**TxBF**: Transmit Beamforming  
**WAL**: WLAN Abstraction Layer  
**Packet Number(PN)**: A monotonically increasing value that is guaranteed unique for each MACsec frame transmitted using a given Secure Association Key (SAK).    
**traffic identifier (TID)**: Any of the identifiers usable by higher layer entities to distinguish medium access control (MAC) service data units (MSDUs) to MAC entities that support quality of service (QoS) within the MAC data service. NOTE—There are 16 possible TID values; eight identify traffic categories (TCs), and the other eight identify parameterized traffic streams (TSs). The TID is assigned to an MSDU in the layers above the MAC. TID subfield include in ADDBA action frame.   
**HWSCH**: hardware scheduler  
**PDG**: PPDU descriptor generator  
**SEQCTRL**: Tx sequence control  
**DSR**: Deferred Service Routines  
**ISR**: interrupt service routine  
**multi-user multiple input, multiple output (MU-MIMO)**: A technique by which multiple stations(STAs), each with one or more antennas, either  **simultaneously transmit** to a single STA or **simultaneously receive** from a single STA **independent data streams over the same radio frequencies**.                                          
  
**downlink multi-user multiple input, multiple output (DL-MU-MIMO)**: A technique by which an access point (AP) with more than one antenna transmits a physical layer (PHY) protocol data unit (PPDU) to multiple receiving non-AP stations (STAs) over the same radio frequencies, wherein each non-AP STA simultaneously receives one or more distinct space-time streams.  

**PCU**: packet checksum unit  

----------

### 2.Tx data path  

Tx path software serves both access point (AP) and repeater solutions with 802.11be and with a higher data rate. The hardware contains per-MSDU operations for the data plane. **The firmware Tx path is responsible for per-MSDU/MPDU operation on the management path and per-PPDU scheduling decisions and control for both the data and management paths**.  


Each sequence scheduled by the medium scheduler translates to series of commands to execute, which is called the **sequence command list**. Each sequence command (**SEQCTRL_CMD**) summarizes the action to transmit to the PPDU. It encodes the PPDU frame type rate, control selection, and so on. The DSR-ISR state machine goes through SEQCTRL until sequence end. 

**The SEQCTRL_CMD encoding includes the following components:**  
■ WAL_TXSEND_FTYPE  
■ User ID mask (identifies the TID/rate parameters)  
■ Rate selection (CTRL/data)  
■ Command-specific data  


Figure 10-26—Illustration of TXOP sharing and PPDU construction (DL MU-MIMO)


80-36301-2_REV_AC_Wi-Fi_7_WLAN_Firmware_Programming_Guide.pdf
