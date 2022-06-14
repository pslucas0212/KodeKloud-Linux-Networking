# KodeKloud Linux Basics: Networking

[KodeKloud Linux Basics Course Notes Table of Contents](https://github.com/pslucas0212/LinuxBasics)    

## Networking

#### DNS overview  
You can add hostname and ip address to the /etc/hosts file.  This is the source of truth for the host where this is configured.  The destination system may have a different name.  This is known as host resolution.  You can ping, but ping maybe disabled on the target system, so consider using nslookup or dig.  
  
Usings hosts file is ok for a small network.  It's easier to move all the hostnames and ip addressess to a DNS server and point all servers to a DNS server for name resolution.   
  
Every system has a dns hostname file to resolve the DNS location in the /etc/resolv.conf file.  Add 'nameserver 192.168.1.100' to the /etc/resolve.conf file.    

If you add information to the /ect/hosts file, the system looks for hostname resolution there and then the DNS server.    
  
You can change the order can be modified modifying the /etc/nsswitch.conf  
  
What if you ping an address not in the /etc/hosts or DNS server.  You can add a second (multiple) name servers in the /etc/hosts file and your system will look there if the systems are not listed in /etc/hosts or your DNS server.

You can change the order in whicn server names are resolved in the /etc/nsswitch.conf file
```
cat /etc/nsswitch.conf
--
hosts:    files dns
--
```
 
This states that /etc/hosts is looked at first for a dns entry and the dns server

You can also add a second dns server in your /etc/resolv.conf file to look at other dns servers
```
nameserver   192.168.1.100
nameserver   8.8.8.8
```

You can also configure the DNS server to forward requests it can resolve to another DNS server

### Domain names  
.com .net .edu .org .io define the top leve domain and describe the intent or  "purpose"  
  
Root -> . ; Top Level Domain name .com -> top level domain ; subdomain -> www or maps or mail    
  
When asking for resolution, your system first goes to internal DNS.  If not found there then you are forwarded to an external DNS server and it is was forwared to different DNS serviers until the domain is resolved.   

Your organization DNS server may point to a Root DNS server which points to .com DNS server which points a specific companies DNS like Google
Speed up future requests, your company's DNS server may cache this informaion for a period of time.

Adding search domains allows you to ping web and have mycompany.com appended to web -> web.mycompany.com
Enable shortcut in /etc/resolv.conf
```
nameserver   192.168.1.100
search       mycompany.com prod.mycompmy.com
```
  
DNS Record Types   
How records are stored in DNS:  
A record type map hostname to an ip addresses 
AAAA record types map hostname to ipv6 addresses
CNAME Record types maps one hostname to another  hostname  - name to name mapping   
Record | Example name | IP Address
-------|--------------|-----------
A | web-server | 192.168.1.1
AAAA | web-server | 2001:0db8:...7334
CNAME | food.web-server | eat.web-server, hungry.web-server

  
Ping nay not be the right tool to check name resolution
- nslookup doesn't use the /etc/hosts file. In this example my local dns server forwards the request toa public dns server
```
$ nslookup www.google.com
Server:		10.1.1.254
Address:	10.1.1.254#53

Non-authoritative answer:
Name:	www.google.com
Address: 142.250.191.132
```
- dig same for dig as nslookup.  it ignores the /etc/hosts file
```
$ dig www.google.com

; <<>> DiG 9.10.6 <<>> www.google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10766
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.google.com.			IN	A

;; ANSWER SECTION:
www.google.com.		130	IN	A	142.250.191.132

;; Query time: 1069 msec
;; SERVER: 10.1.1.254#53(10.1.1.254)
;; WHEN: Tue Jun 14 17:32:17 CDT 2022
;; MSG SIZE  rcvd: 59
```
  
  
#### Switching and Routing
How does server A reach server B.  We need a switch and an interface on both systems.  
To see interface names
```
$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:ab:d4:22 brd ff:ff:ff:ff:ff:ff
3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:d8:45:21 brd ff:ff:ff:ff:ff:ff
4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:d8:45:21 brd ff:ff:ff:ff:ff:ff
```
Assign ip address to computers to allow them to communicate on the switch
```
$ ip addr add 192.168.1.10/24 dev eth0
```
 
Say wwe have another network with server C and server D, with a subnet 192.168.2.0 how do we reach the other network.  We use an intelligent (computer like device) called a router.  We connect the two networks via the router and assign to IP address on the router: 192.168.1.1 and 192.168.2.1.  We configure a gateway to route between the networks (like a door to another room.   
  
Run route command to see the kernel's routing table for your Linux machine
```
$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         10.1.1.1        0.0.0.0         UG    202    0        0 eth0
10.1.1.0        0.0.0.0         255.255.255.0   U     202    0        0 eth0
10.1.10.0       edge.example.co 255.255.255.0   UG    0      0        0 eth0
```
To add a route:
```
$ ip route add 192.168.2.0/24 via 192.168.1.1
```
For any network where you want an unknown ip address sent use the default router setting:
```
$ ip route add default via 192.168.1.1
```
You could also 0.0.0.0 in the gateway field.  However if you have a public and private internet you will need a second router and gateway "link"

Key Network commands
To list and modify interfaces on the host;
```
$ ip link
```
To see ip addresses assigned to interfaces
```
$ ip addr
```
To add an ip address to an interface.  This examples assignes the ip address 192.168.1.10 to the device eth0 (ethernet nic)
```
$ ip addr add 192.168.1.10/24 dev eth0
```
These are only valid until the system is rebooted.  To make the settings persistent add a static ip address to the etc/sysconfig/network-scripts/ifcfg-ens192 in RHEL on Raspberry pi its in the /etc/dhcpcd.conf file
  
  
 #### Network Troubleshooting
 - time out error
    - run $ ip link to make sure interface link is up
    - Check to resolve that hostname is resolving - $ nslookup <server hostname>
    - Ping remote server - $ ping <host-name> - not the best tool to check connectivity as some devices may have it disabled
    - Run traceroute command to troubleshoot the routing - $ traceroute 192.168.2.5
    - On the server you can if a process is running on a particular port like port 80 - $ netstat -an | grep 80 | grep -i LISTEN
    - If web server for example is running try running the ip link command on the server
    - Bring link up with - $ ip link set dev <interface name> up
  

