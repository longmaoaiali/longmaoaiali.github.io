---
layout:     post
title:      Compile iperf3 for windows
subtitle:   Compile iperf3 for windows
date:       2023-10-11
author:     BuBu
header-img: img/yellow-background.jpg
catalog: true
tags:
    - Case
    - Source code 
---

----------
### 1.Preparation  

1. Install Cygwin on windows: [Download Link](http://cygwin.com/setup-x86.exe). For install details, refer to [Install Cygwin](https://www.cygwin.com/install.html).    
2. Download iperf3 source code from github [iperf](https://github.com/esnet/iperf).   
 

### 2.Compile Steps

Copy iperf source code to Cygwin directory, unzip the iperf folder and copy the main directory to this (C:\cygwin64\home\user_name). Open the Cygwin Terminal, go to "C:\cygwin64\home\yliu25\iperf3.1.3" directory.     

<font color="#FF8C00">Cygwin Terminal:  </font> 
<img src="/img/post/2023-10-11-Cygwin.png"/>  

<font color="#FF8C00">Cygwin Console:  </font> 
<img src="/img/post/2023-10-11-Cygwin-Console"/>  

Static compile iperf steps:  

	./configure CFLAGS=-static CXXFLAGS=-static --prefix=/usr/local/iperf3.1.3
	make   
	make install 

<font color="#FF8C00">Generated iperf.exe:  </font> 
<img src="/img/post/2023-10-11-Generated-Iperf-Bin.png"/> 


### 3.Compile Error

#### 3.1.conflicting types for 'iprintf'

	iperf_api.h:227:5: error: conflicting types for 'iprintf'
	solution is remove one of the conflicting define.  
	
#### 3.2.unable to create a new stream: No such file or directory 

	unable to create a new stream: No such file or directory
	change src\iperf_api.c code as below:
	char template[] = "/tmp/iperf3.XXXXXX";  -->  char template[] = "./iperf3.XXXXXX";

### 4.Run test  

Copy iperf.exe, cygwin1.dll and run for test.  

<font color="#FF8C00">Generated iperf.exe test:  </font> 
<img src="/img/post/2023-10-11-Generated-Iperf-Test.png"/> 

	usage for iperf
	iperf3.exe -c 192.168.1.191 -u -b 60M -i 1 -l 1470 --cport 52723 --bind 192.168.1.99 -t 20  
	# make 5-tuple info same for acceleration test
	iperf3.exe -c 192.168.1.191 -u -b 60M -i 1 -l 1470 -k 15000 
	# send 15000 packets for packets loss debug
	iperf3.exe -c 192.168.1.191 -u -b 60M -i 1 -l 1470 -t 20 -O 2
	# ignore 2s by '-O' option, avoid the influence from create acceleration rule 
	iperf3.exe -s -i 1 --debug 
	# enable iperf log output for debugging


### 5.Reference  

[Install Cygwin details](https://blog.csdn.net/lvsehaiyang1993/article/details/81027399?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163576264216780265436642%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=163576264216780265436642&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-2-81027399.first_rank_v2_pc_rank_v29&utm_term=cygwin%E5%AE%89%E8%A3%85&spm=1018.2226.3001.4187)  
[Compile iperf](https://blog.csdn.net/C112336/article/details/121085370)  
[download iperf](https://files.budman.pw/) 