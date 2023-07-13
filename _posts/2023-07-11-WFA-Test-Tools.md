---
layout:     post
title:      WFA(Wi-Fi Alliance) Test Tools   
subtitle:   WTS & QuickTrack Test Tool
date:       2023-07-11
author:     BuBu
header-img: img/blue-background.jpg
catalog: true
tags: 
    - WFA
---

----------

### 1.Acronyms

**WFA**: Wi-Fi Alliance  
**WTS**: Wi-Fi Test Suite  
**CA**: Control Agent  
**DUT**: Device under test  
**STAUT**: Station under Test  
**TA**: Traffic Agent  
**UCC**: Unified CAPI Command Console  
**CAPI**: Control API  

----------

### 2.WFA Certification Paths

Wi-Fi CERTIFIED products have undergone rigorous testing by one of three certification paths:

- **FlexTrack**: Tailored to sophisticated product design built from the ground up. FlexTrack allows the most flexible Wi-Fi functionality customization built into components. Testing is completed at an Authorized Test Laboratory (ATL).

- **QuickTrack**: Tailored to products based on components that have already completed full Wi-Fi functionality testing in a Qualified Solution. QuickTrack allows targeted changes to Wi-Fi functionality. Testing can be completed at an ATL or member testing site. Programs for which QuickTrack is available include: Wi-Fi CERTIFIED 6® and Wi-Fi 6E operation, Wi-Fi CERTIFIED ac, Wi-Fi CERTIFIED n, Wi-Fi Agile Multiband​™, Wi-Fi Data Elements™, Protected Management Frames, WPA2™, WPA3™, Wi-Fi Enhanced Open™, Wi-Fi Protected Setup™, and Wi-Fi Direct®.

- **Derivative**: For copies of a Wi-Fi CERTIFIED device Source Product, such as the same chipset used for multiple laptop models. Members may apply for certification of derivative products without the testing requirement.

<font color="#FF8C00">WFA Certification Paths:  </font> 
<img src="/img/post/2023-07-11-WFA-Certification-Paths.png"/>

----------

### 3.Wi-Fi Test Suite

Wi-Fi Test Suite is a software platform developed by Wi-Fi Alliance to support certification program development and device certification. Wi-Fi Test Suite is used by ATLs to certify Wi-Fi Alliance members’ products primarily utilizing the **FlexTrack certification path.** 

Wi-Fi Test Suite is an integrated platform that automates testing Wi-Fi components or devices. Wi-Fi Test Suite provides the following services:

    Configure - Automatically configure devices to execute test cases.
    Traffic Generation - Generate traffic streams with specified parameters.
    Test - Execute test scripts by controlling test bed device operation.
    Results Analysis - Determine pass/fail results based on a given test case or script criteria.

A separate “sigma-dut” binary for WTS which is a control application for WTS test cases.   

----------

#### 3.1.WTS Basic communication paradigm  

**The Control Agent performs a passive TCP open on a selected port and waits for the UCC to connect to it.** The UCC is configured to perform an active TCP open to the Control Agent. Once the TCP connection is established, any API calls made by the UCC are sent to the Control Agent as ASCII formatted commands over the TCP connection. The UCC must handle one, and only one, command at a time.  
    
Each command is terminated by a new-line (CR+LF). The Control Agent uses the same format for sending responses. If the TCP connection terminates for any reason, the Control Agent should wait passively for the next connection attempt from the UCC.   
  
**A typical CAPI command sequence is shown below:**    

	UCC: device_list_interfaces,interfaceType,802.11
	CA: status,RUNNING
	CA: status,COMPLETE,interfaceType,802.11,interfaceID,interfaceId_1 interfaceId_2
	UCC: sta_get_info,interface,interfaceId_1
	CA: status,RUNNING
	CA: status,COMPLETE,vendorInfo_1,value_1,vendorInfo_2,value_2   

Commands are executed sequentially. In this example, the UCC will not respond with “STA_GET_INFO” until the device CA responds with the completed device list.  
Note that users need to append the characters line feed \r\n to every response so that a new line will separate the commands on the UCC console output or the logs.  

----------

#### 3.2.sigma-dut  

Take **8.23 AP_SET_WIRELESS** as an example, simga-dut code path as below:

	int main(int argc, char *argv[])
	open_socket(&sigma_dut, port) //Init socket server
	run_loop()
	process_conn()
	recv(conn->s, conn->buf + conn->pos, MAX_CMD_LEN + 5 - conn->pos, 0);
	process_cmd(dut, conn, conn->buf);
	res = h->process(dut, conn, &c);
	
	sigma_dut_reg_cmd("ap_set_wireless", NULL, cmd_ap_set_wireless);
	static enum sigma_cmd_result cmd_ap_set_wireless()

----------

### 4.QuickTrack Test Tool  

The QuickTrack Test Tool software provides conformance testing based on Wi-Fi Alliance test plans using the **QuickTrack certification path**.   

The QuickTrack Test Tool is tailored to test and certify products based on Qualified Solutions that **have already completed full Wi-Fi® functionality testing—modules, chipsets, and other solutions developed by Solution Providers**, which have undergone the prerequisite interoperabilty and conformance testing.   

----------

#### 4.1.QuickTrack API messages Architecture

Each message may have any number of TLVs based on its requirements up to 1400 bytes. TLVs may arrive in any order provided the TLV is completely intact. Any TLV not understood by the DUT Control App or the QuickTrack Test Tool can be ignored and further processing of any following TLVs should continue.  

<font color="#FF8C00">Sample API message transaction and TLVs:  </font> 
<img src="/img/post/2023-07-12-QuickTrack-API-message-transaction-and-TLVs.png"/> 

----------

#### 4.2.Ctrl APP

A separate “ctrl app dut” binary for Quick Track which is a control application for Quick Track test cases. Refer to ***QuickTrack API Specification v2.1***      
Take API message ***AP_CONFIGURE*** as an example, ctrl-app code path as below:

	ctrl_app_dut running on DUT to parse QuickTrack message and configure AP DUT:
	usage: For wifi2 is 6G, wifi0 – 5G and wifi1 – 2.4Ghz – below is the command to start the app application. 
	app -p 9005 -R wifi2 -R wifi0 -R wifi1 -i 6:ath2,6:ath21,5:ath0,5:ath01,2:ath1,2:ath11

	int main(int argc, char* argv[])
	control_socket_init(int port)
	eloop_register_read_sock(s, control_receive_message, NULL, NULL)
	control_receive_message(int sock, void *eloop_ctx, void *sock_ctx);
	api = get_api_by_id(req.hdr.type);
	api->handle(&req, &resp)
	register_api(API_AP_CONFIGURE, NULL, configure_ap_handler);
	int configure_ap_handler(struct packet_wrapper *req, struct packet_wrapper *resp)
	int generate_hostapd_config(char *output, int output_size, struct packet_wrapper *wrapper, struct interface_info* wlanp)


  

