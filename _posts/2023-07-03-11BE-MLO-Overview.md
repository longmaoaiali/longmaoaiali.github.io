---
layout:     post
title:      11BE MLO Overview
subtitle:   MLO Demo
date:       2023-07-03
author:     BuBu
header-img: img/orange-background.jpg
catalog: true
tags:
    - 11BE EHT
    - WLAN Driver
    - MLO
    - IEEE802.11 be
---

----------
### 1.Acronyms
MLD: Multi-Link Device  
MLO: Multi-Link Operation  
NSTR: Nonsimultaneous transmit and receive (NSTR) operation  
STR: Simultaneous transmit and receive (STR) operation  
WM: wireless medium (WM)  
EMLMR: enhanced multi-link multi-radio  
EMLSR: enhanced multi-link single radio  

----------

### 2.Concept of MLO

#### 2.1.What is MLO? 
Multi-Link Operation (MLO) enables data flows to be sent across multiple OTA channels to a Multi-Link device.  
<img src="/img/post/2023-07-03-MLO-Sample.png"/>

#### 2.2.Benefits of MLO 
1. Provides reduced latency through opportunistic channel selection  
2. Increased peak throughput: Packets of the same flow can be sent over multiple links  
3. Improved latency: Due to increased channel access opportunities on multiple links

----------

### 3.Types of MLO (Multi-link Operation)  
**Multi-link single radio (MLSR)**: Able to RX and TX over one radio at a time. (One radio)  
**Enhanced MLSR(eMLSR)**: Enhances MLSR with a reduced function radio to choose best link.    
**Non-simultaneous TX and RX multi-link multi radio(Non-STR MLMR)**: Able to simultaneously RX and TX over ≥ 2 radios, but only under certain constraints (eg: aligned TX/RX)  
**STR MLMR**: Able to simultaneously RX and TX over ≥ 2 radios

Test Result of different types of MLO:
<img src="/img/post/2023-07-03-Type-of-MLO-Test.png"/>

----------

### 4.MLO Key feature
#### 4.1.Packet Level Aggregation  
1. Packets of the same TID can be sent on one or more radios  
2. Helps with low-latency and peak throughput improvement  
#### 4.2.Cross-wake-up signaling for power-save  
1. AP indicates buffered unit indication of other links in a link that STA is monitoring  
2. STA can indicate wake-up of a link using another link  
3. STA can monitor a link in idle mode to get BSS/TIM information of other links  
#### 4.3.Fast link-transition  
1. The active link(s) can be switched dynamically to adapt to load /co-ex conditions  
2. Beneficial for 11be single-radio STAs  
#### 4.4.Multi-primary channel access  
1. Needed for latency improvements  
#### 4.5.Shared Single Session across links  
1. Single Block-ack session per TID, shared sequence number space  
2. Single authentication and key derivation for unicast packets  
3. Separate group-keys for broadcast /groupcast packets  

----------



