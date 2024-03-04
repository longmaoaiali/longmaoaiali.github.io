---
layout:     post
title:      UBUS & netifd        
subtitle:   Openwrt, Linux      
date:       2024-01-31
author:     BuBu
header-img: img/yellow-background.jpg
catalog: true
tags:
    - Openwrt, Linux   
  
---

----------
### 1.Acronyms



----------

### 2.UBUS    

UBUS is openWRT IPC program, netifd register object and method to ubus. "ubus call" can invoke these methods. 

<font color="#FF8C00">ubus object source code:  </font> 
<img src="/img/post/2024-01-31-ubus-object.png"/>

root@OpenWrt:/# ubus -v  list network  
root@OpenWrt:/# ubus -v  list network.interface  
root@OpenWrt:/# ubus call network.interface.lan status  

### 3.Netifd 

Netifd manage the network interfaces and the interface events. The netifd read the network configuration and initialize the device.  

<font color="#FF8C00">UML sequence: "ubus call network reload" code flow： </font>  <font color="#00000">**UBUS-Netifd.drawio**   </font> 

Netifd bonding interface: bonding_create()  
Netifd Vlan(athX.staX) interface: 


### 4.Procd in Openwrt system  

Reference:   
https://openwrt.org/docs/guide-developer/procd-init-scripts 
https://blog.csdn.net/pyt1234567890/article/details/111370310  
 

	/etc/init.d/network start_service()

	start_service() {
        is_kmodloader_done
        init_switch

        procd_open_instance
        procd_set_param command /sbin/netifd
        #procd_set_param limits core="unlimited"
        procd_set_param respawn
        procd_set_param watch network.interface
		#ubus namespaces network.interface to watch and wait for incoming ubus events
        [ -e /proc/sys/kernel/core_pattern ] && {
                procd_set_param limits core="unlimited"
        }
        procd_close_instance
	}


