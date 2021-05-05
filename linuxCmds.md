
## Network ##


### 1. ip command  
ip link  
ip route   
ip --help  
ip address   


### 2. traceroute
If ping shows missing packets, you should use traceroute to see what route the packets are taking.  Traceroute shows the sequence of gateways through which the packets travel to reach their destination. 
```
$ traceroute google.com
traceroute to google.com (172.217.167.46), 64 hops max, 52 byte packets
 1  dlinkrouter.dlink (192.168.0.1)  5.376 ms  2.076 ms  1.932 ms
 2  10.194.0.1 (10.194.0.1)  5.190 ms  5.125 ms  4.989 ms
 3  broadband.actcorp.in (49.207.47.201)  7.165 ms  5.749 ms  5.755 ms
 4  broadband.actcorp.in (49.207.47.225)  5.918 ms *  8.483 ms
...
 9  108.170.251.97 (108.170.251.97)  6.359 ms
    del03s16-in-f14.1e100.net (172.217.167.46)  5.448 ms
    108.170.251.97 (108.170.251.97)  6.400 ms
```

Line 4 in this output shows a * in the round trip times. This indicates no response was received. This can be due to many reasons – as the traceroute ICMP packets are low-priority, these may be dropped by a router. Or there could be simply congestion.  If you see a * in all the time fields for a given gateway, then possibly the gateway is down.

### 3. telnet  
telnet connect destination’s host and port via a telnet protocol if a connection establishes means connectivity between two hosts is working fine  
```
[root@lab ~]# telnet gf.dev 443
Trying 104.27.153.44...
Connected to gf.dev.
```


### 4. nslookup  
nslookup is a program to query domain name servers and resolving IP.  
```
nslookup google.com
```


### 5. ss - moving from netstat  
You can see what services are running with the netstat command. While netstat is still available, most Linux distributions are transitioning to ss command.  
use ss command with -t and -a flags to list all TCP sockets. This displays both listening and non-listening sockets.  
To display only TCP connections with state established:  
 > ss -a -t -o state established  
 
### 6. 
### 7. 
