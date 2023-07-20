---
layout:     post
title:      MLO Network Device Model
subtitle:   MLO Single Network Device
date:       2023-07-19
author:     BuBu
header-img: img/orange-background.jpg
catalog: true
tags:
    - 11BE EHT
    - WLAN Driver
    - MLO
    - Network
---

----------
### 1.Acronyms

NDP: Neighbor Discovery Protocol  
NS: Neighbor Solicitation  
NA: Neighbor Advertisement


----------

### 2.DHCP & ARP operation with MLO 

MLO link network device will add to br-lan bridge, DHCP and ARP operation only happen on MLO primary link.   

<font color="#FF8C00">DHCP & ARP operation with MLO:  </font> 
<img src="/img/post/2023-07-14-IPV4-DHCP-ARP-MLO-Operation.png"/>

<font color="#FF8C00">AP Discard DHCP ARP Operation For Second Link:  </font> 
<img src="/img/post/2023-07-14-AP-Discard-DHCP-ARP-Operation-For-Second-Link.png"/>

----------

### 3.DHCPV6 & NDP operation with MLO 

The NDP protocol is used for address resolution in IPv6. NS,NA frame in IPV6 has the same effect as arp request,reply in IPV4.   

<font color="#FF8C00">Neighbor Discovery Protocol:  </font> 
<img src="/img/post/2023-07-19-Neighbor-Discovery-Protocol.png"/>



DHCPV6 protocol has multiple frame exchange option:    

- DHCPv6 Stateful Mode four-step frame exchange  
- DHCPv6 Stateful Mode two-step fast frame exchange
- DHCPv6 stateless mode

<font color="#FF8C00">DHCPv6 Stateful Mode four-step frame exchange:  </font>   
<img src="/img/post/2023-07-19-DHCPv6-Stateful-four-step-frame-exchange.png"/>

<font color="#FF8C00">DHCPv6 Stateful Mode two-step fast frame exchange:  </font>   
<img src="/img/post/2023-07-19-DHCPv6-Stateful-two-step-fast-frame-exchange.png"/>

<font color="#FF8C00">DHCPv6 Stateless Mode frame exchange:  </font>   
<img src="/img/post/2023-07-19-DHCPv6-stateless-mode-frame-exchange.png"/>

----------

### 4.MLO Single Network Device

