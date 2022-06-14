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
How does server a reach server b.  We need a switch and an interface on both systems.  
```
$ ip link
```
Assign ip address to computer:  
```
$ ip addre add 192.168.1.10/24 dev eth0
```
 
Say wwe have another network with server c and server d, with a subnet 192.168.2.0 how do we reach the other network.  We use an intelligent (computer like device) called a router.  We connect the two networks via the router: 192.168.1.1 and 192.168.2.1.  We configure a gateway to route between the networks.   
  
Run route command to see routing options for your Linux machine
```
$ route
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

To persist ip link, ip addr, ip addr, add these to the network interface file.
  
  
 #### Network Troubleshooting
 - time out error
    - run $ ip link to make sure interface link is up
    - Check to resolve that hostname is resolving - $ nslookup <server hostname>
    - Ping remote server - $ ping <host-name> - not the best tool to check connectivity as some devices may have it disabled
    - Run traceroute command to troubleshoot the routing - $ traceroute 192.168.2.5
    - On the server you can if a process is running on a particular port like port 80 - $ netstat -an | grep 80 | grep -i LISTEN
    - If web server for example is running try running the ip link command on the server
    - Bring link up with - $ ip link set dev <interface name> up
  

