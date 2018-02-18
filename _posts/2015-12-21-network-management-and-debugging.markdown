---
layout: post
title: "Network management and debugging"
date: 2015-12-21 22:51:26 +0530
comments: true
categories: [ Linux, C, Tutorial ]
---

In this post, I am gonna discuss few simple network troubleshooting tools and how it can be used in day to day work. 

Network management is the essential understanding needed for any sysadmin and unix guy. It mainly used in the below scenarios,

* failure detection of networks,gateways and other devices 
* monitoring and reporting the failure to system administrators 
* ensuring the availabilty of network connectivity among machines. 

In a large enterprise network, due to increase in the number of machines connected it is essential to automate the network management through shell scripts and/or any automation framework. The efficiency of any system/network administrator is determined how fast he solves the problem which largely depends of how well his troubleshooting skills and awareness about network debugging tools. Here we are going to see few quick tips on how to use network debugging tools

* ping 
* traceroute 
* netstat 
* tcpdump

## ping
One of the simple command which most of unix users be aware of and of course the first command be used if anything goes wrong in machines connected to the network. It sends an ICMP_ECHO_REQUEST packet to a target machine and will wait for the response back.

### purpose
To check whether the target system is reachable or can say alive or not. If the ping command doesn't succeed means something went wrong in target machine or anyone of the interfacting devices ( like gateway , routers). If the firewall which blocks the ICMP requests , come in between then can decide based on ping results.

### options
Most of the times, `ping` is used without any options but with host name / ip address.

* `t` - timeout - time in secs before ping exits
* `-c` - count - stop sending and receiving number ECHO_RESPONSE packets
### examples

{% highlight bash %}
$ ping google.com
PING google.com (216.58.192.14): 56 data bytes
64 bytes from 216.58.192.14: icmp_seq=0 ttl=48 time=307.088 ms
64 bytes from 216.58.192.14: icmp_seq=1 ttl=48 time=328.524 ms
64 bytes from 216.58.192.14: icmp_seq=2 ttl=48 time=351.347 ms
64 bytes from 216.58.192.14: icmp_seq=3 ttl=48 time=374.660 ms
64 bytes from 216.58.192.14: icmp_seq=4 ttl=48 time=397.763 ms
64 bytes from 216.58.192.14: icmp_seq=5 ttl=48 time=318.065 ms
{% endhighlight %}

{% highlight bash %}
$ ping -c5  google.com
PING google.com (216.58.192.14): 56 data bytes
64 bytes from 216.58.192.14: icmp_seq=0 ttl=48 time=374.565 ms
64 bytes from 216.58.192.14: icmp_seq=1 ttl=48 time=396.424 ms
64 bytes from 216.58.192.14: icmp_seq=2 ttl=48 time=317.762 ms
64 bytes from 216.58.192.14: icmp_seq=3 ttl=48 time=443.040 ms
64 bytes from 216.58.192.14: icmp_seq=4 ttl=48 time=363.656 ms

--- google.com ping statistics ---
5 packets transmitted, 5 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 317.762/379.089/443.040/41.012 ms
{% endhighlight %}

## traceroute
This is second tool which is used after checking the ping results. It gives the list/sequence of gateways involved when an IP packets travels through to reach the target machine.( `tracert` - windows version )

### purpose
If something went wrong in any of the gateway device in the travel path, traceroute will give which device went wrong. Also based on the results, one can find how far ( how many hops ) the target machine is from source machine.

### options
* `-S` - prints the summary
* `-v` - verbose

### examples

{% highlight bash %}
$ traceroute google.com
traceroute to google.com (216.58.192.46), 64 hops max, 52 byte packets
 1  10.228.129.13 (10.228.129.13)  62.556 ms  55.818 ms  40.850 ms
 2  10.228.149.14 (10.228.149.14)  49.270 ms  45.479 ms  35.500 ms
 3  116.202.226.145 (116.202.226.145)  69.431 ms  36.812 ms  50.864 ms
 4  10.228.158.82 (10.228.158.82)  38.921 ms  46.168 ms  56.392 ms
 5  * * *
 6  * 116.202.226.29 (116.202.226.29)  69.303 ms  45.366 ms
 7  72.14.205.145 (72.14.205.145)  62.391 ms  279.589 ms  60.562 ms
 8  72.14.233.204 (72.14.233.204)  70.989 ms
    72.14.232.110 (72.14.232.110)  65.159 ms  51.378 ms
 9  72.14.238.178 (72.14.238.178)  91.255 ms *  88.974 ms
10  209.85.255.129 (209.85.255.129)  156.642 ms  164.059 ms  146.132 ms
11  209.85.142.51 (209.85.142.51)  303.842 ms  306.960 ms  307.133 ms
12  216.239.49.199 (216.239.49.199)  409.659 ms  308.340 ms  307.171 ms
13  209.85.246.39 (209.85.246.39)  307.114 ms  354.074 ms  512.001 ms
14  * 74.125.37.43 (74.125.37.43)  424.770 ms  306.891 ms
15  nuq04s30-in-f14.1e100.net (216.58.192.46)  307.194 ms  283.052 ms  277.967 ms
{% endhighlight %}

{% highlight bash %}
$ traceroute -S  google.com
traceroute to google.com (216.58.192.46), 64 hops max, 52 byte packets
 1  10.228.129.13 (10.228.129.13)  44.543 ms  45.019 ms  49.097 ms (0% loss)
 2  10.228.149.14 (10.228.149.14)  47.689 ms  36.681 ms  45.811 ms (0% loss)
 3  116.202.226.145 (116.202.226.145)  42.370 ms  58.391 ms  69.763 ms (0% loss)
 4  10.228.158.82 (10.228.158.82)  52.306 ms  47.104 ms  42.364 ms (0% loss)
 5  * * * (100% loss)
 6  116.202.226.29 (116.202.226.29)  286.176 ms  60.681 ms  75.139 ms (0% loss)
 7  72.14.205.145 (72.14.205.145)  97.671 ms  52.065 ms  64.311 ms (0% loss)
 8  72.14.233.204 (72.14.233.204)  81.570 ms
    72.14.232.110 (72.14.232.110)  35.136 ms
    72.14.233.204 (72.14.233.204)  55.917 ms (0% loss)
 9  72.14.238.178 (72.14.238.178)  111.890 ms
    216.239.48.215 (216.239.48.215)  93.550 ms
    72.14.238.178 (72.14.238.178)  102.708 ms (0% loss)
10  209.85.255.129 (209.85.255.129)  188.965 ms  164.314 ms  150.868 ms (0% loss)
11  * 209.85.142.51 (209.85.142.51)  315.784 ms  365.749 ms (33% loss)
12  216.239.49.199 (216.239.49.199)  307.158 ms  305.867 ms  305.937 ms (0% loss)
13  209.85.246.39 (209.85.246.39)  306.018 ms  511.760 ms  307.158 ms (0% loss)
14  74.125.37.43 (74.125.37.43)  307.454 ms  307.037 ms * (33% loss)
15  nuq04s30-in-f14.1e100.net (216.58.192.46)  323.977 ms  306.656 ms  307.299 ms (0% loss)
{% endhighlight %}

## netstat
`netstat` Network Statistics tool, provides the rich information about the network connections, interface details and routing tables.
### purpose

* List network connections
* Interface configuration details
* Routing table details
* Statistics for all n/w protocols

### options
`netstat` without any options display only the active connections of TCP/UDP protocols.

* `-a` - lists all connections including the listening ports
* `-n` - shows n/w address as numbers
* `-p` - display protocol specific details ( valid proto listed in /etc/protocols )
* `-i` - state of auto-configured interface details
* `-r` - shows routing tables
* `-s` - display per protocol statistics
* `-v` - verbose

### examples

{% highlight bash %}
$ netstat -a|more
Active Internet connections (including servers)
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)    
tcp4       0      0  192.168.1.106.50892    103.245.222.133.http   LAST_ACK   
tcp4       0      0  localhost.terabase     *.*                    LISTEN     
tcp6       0      0  localhost.terabase     *.*                    LISTEN     
tcp6       0      0  localhost.terabase     *.*                    LISTEN     
tcp4       0      0  192.168.1.106.50891    17.172.239.131.5223    ESTABLISHED
tcp4       0      0  *.*                    *.*                    CLOSED     
tcp46      0      0  *.http                 *.*                    LISTEN     
tcp4       0      0  localhost.ipp          *.*                    LISTEN     
tcp6       0      0  localhost.ipp          *.*                    LISTEN     
tcp4       0      0  *.ftp                  *.*                    LISTEN     
tcp6       0      0  *.ftp                  *.*                    LISTEN     
udp6       0      0  *.61052                *.*                               
udp4       0      0  *.61052                *.*                               
udp4       0      0  192.168.1.106.ntp      *.*            
{% endhighlight %}

{% highlight bash %}
$ netstat -an|more
Active Internet connections (including servers)
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)    
tcp4       0      0  192.168.1.106.50892    103.245.222.133.80     LAST_ACK   
tcp4       0      0  127.0.0.1.4000         *.*                    LISTEN     
tcp6       0      0  ::1.4000               *.*                    LISTEN     
tcp4       0      0  192.168.1.106.50891    17.172.239.131.5223    ESTABLISHED
tcp4       0      0  *.*                    *.*                    CLOSED     
tcp46      0      0  *.80                   *.*                    LISTEN     
tcp4       0      0  127.0.0.1.631          *.*                    LISTEN     
tcp6       0      0  ::1.631                *.*                    LISTEN     
tcp4       0      0  *.21                   *.*                    LISTEN     
tcp6       0      0  *.21                   *.*                    LISTEN     
udp6       0      0  2406:5600:65:e90.123   *.*                               
udp6       0      0  *.61052                *.*                               
udp4       0      0  *.61052                *.*              
{% endhighlight %}

{% highlight bash %}
$ netstat -p tcp     
Active Internet connections
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)    
tcp4       0      0  192.168.1.106.50958    17.110.228.152.5223    ESTABLISHED
{% endhighlight %}

{% highlight bash %}
$ netstat -s  
tcp:
	48491 packets sent
		9893 data packets (30309378 bytes)
		109 data packets (65356 bytes) retransmitted
		0 resends initiated by MTU discovery
		30041 ack-only packets (560 delayed)
		0 URG only packets
		0 window probe packets
		4761 window update packets
		3699 control packets
		0 data packets sent after flow control
		43446 checksummed in software
			39970 segments (3356695 bytes) over IPv4
			3476 segments (195663 bytes) over IPv6
	50096 packets received
		11810 acks (for 30196532 bytes)
		4175 duplicate acks
		0 acks for unsent data
		29320 packets (54768822 bytes) received in-sequence
		2017 completely duplicate packets (81500 bytes)
		0 old duplicate packets
		324 packets with some dup. data (738 bytes duped)
		2221 out-of-order packets (2089821 bytes)
		0 packets (0 bytes) of data after window
		0 window probes
		1237 window update packets
		347 packets received after close
		0 bad resets
		0 discarded for bad checksums
		42574 checksummed in software
			40459 segments (30219462 bytes) over IPv4
			2115 segments (1509589 bytes) over IPv6
		0 discarded for bad header offset fields
		0 discarded because packet too short
	1807 connection requests
	175 connection accepts
	0 bad connection attempts
	0 listen queue overflows
..............
{% endhighlight %}

## packet sniffers
Packet sniffers which is widely used by network security admins for debugging and ensuring the safety of the network.  Its useful to give the solutions and sometimes to find the problems in the network as well. `tcpdump` - king of network sniffers,  is widely used for this purpose. Other n/w sniffer tools are wireshark in windows, nettl in HP-UNIX, snoop in solaris
### purpose
Listen to the network traffic and record/print the network packets sent/received to/from other destination machines.
### options

* `-i any` - listen to all interfaces
* `-i <intf>` - specific to an interface
* `-n` - Not to resolve hostnames
* `-nn` - Not to resolve hostnames and ports
* `-X` - display contents in ascii and hex
* `-S` - show absolute sequence number
* `-e` - display ethernet header
* `-s <size>` -  sanplength size for capturing the packets
* `-w` - writing to a file
* `-r` - reading from a file

`tcpdump` provides features to specify expression to filter various of n/w traffic. There are three types of expression as below

* `type` : host(based on ip addr) , net ( capture entire n/w based on CIDR ) and port
* `dir` : src, dst, src or dst, src and dst
* `proto` : tcp , udp

Packet size filter is also possible based on expression `less <size>` , `greater <size>` , `>` and `<=`. `tcpdump` provides some advanced features like logical and grouping in the expression.

### examples

{% highlight bash %}
$ sudo tcpdump -nS
Password:
tcpdump: data link type PKTAP
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on pktap, link-type PKTAP (Packet Tap), capture size 65535 bytes
00:42:30.498758 IP6 fe80::fedd:55ff:fe08:7319 > ff02::1: ICMP6, router advertisement, length 64
00:42:35.511677 IP6 fe80::fedd:55ff:fe08:7319 > ff02::1: ICMP6, router advertisement, length 64
00:42:40.529430 IP6 fe80::fedd:55ff:fe08:7319 > ff02::1: ICMP6, router advertisement, length 64
00:42:45.542351 IP6 fe80::fedd:55ff:fe08:7319 > ff02::1: ICMP6, router advertisement, length 64
00:42:50.555216 IP6 fe80::fedd:55ff:fe08:7319 > ff02::1: ICMP6, router advertisement, length 64
00:42:55.323157 IP 192.168.1.106.51215 > 54.243.65.90.443: Flags [P.], seq 2315517496:2315518173, ack 2049518988, win 16384, length 677
00:42:55.470470 IP6 fe80::fedd:55ff:fe08:7319 > ff02::1: ICMP6, router advertisement, length 64
00:42:55.984674 IP 54.243.65.90.443 > 192.168.1.106.51215: Flags [.], ack 2315518173, win 247, length 0
00:42:56.291941 IP 54.243.65.90.443 > 192.168.1.106.51215: Flags [P.], seq 2049518988:2049519265, ack 2315518173, win 250, length 277
00:42:56.291987 IP 192.168.1.106.51215 > 54.243.65.90.443: Flags [.], ack 2049519265, win 16366, length 0
00:43:00.483332 IP6 fe80::fedd:55ff:fe08:7319 > ff02::1: ICMP6, router advertisement, length 64
00:43:10.313961 IP 192.168.1.106.51215 > 54.243.65.90.443: Flags [P.], seq 2315518173:2315518866, ack 2049519265, win 16384, length 693
00:43:10.462048 IP6 fe80::fedd:55ff:fe08:7319 > ff02::1: ICMP6, router advertisement, length 64
00:43:11.438018 IP 54.243.65.90.443 > 192.168.1.106.51215: Flags [.], ack 2315518866, win 247, length 0
00:43:11.586302 IP 54.243.65.90.443 > 192.168.1.106.51215: Flags [P.], seq 2049519265:2049519542, ack 2315518866, win 250, length 277
00:43:11.586382 IP 192.168.1.106.51215 > 54.243.65.90.443: Flags [.], ack 2049519542, win 16366, length 0
00:43:15.531813 IP6 fe80::fedd:55ff:fe08:7319 > ff02::1: ICMP6, router advertisement, length 64
00:43:20.549672 IP6 fe80::fedd:55ff:fe08:7319 > ff02::1: ICMP6, router advertisement, length 64
{% endhighlight %}
