# KodeKloud Linux Basics: Networking

## Networking


[KodeKloud Linux Basics Course Notes Table of Contents](https://github.com/pslucas0212/LinuxBasics).  

#### DNS overview  
You can add hostname and ip address to the /etc/hosts file.  This is the source of truth for the host where this configed.  The destination system may have a different name.  This is known as host resolution.  You can ping, but ping maybe disabled on the target system, so consider using nslookup or dig.  
  
hosts file is ok for a small network.  It's easier to move all the hostnames and ip addressess to a DNS server and point all servers to a DNS server for name resolution.   
  
Every system has a dns hostname file to resolve the DNS location in the /etc/resolv.conf file.  Add 'nameserver 192.168.1.100' to the /etc/resolve.conf file.    

If you add information to the /ect/hosts file, the system looks for hostname resolution there and then the DNS server.    
  
You can change the order can be modified modifying the /etc/nsswitch.conf  
  
What if you ping an address not in the /etc/hosts or DNS server.  You can add a second (multiple) name servers in the /etc/hosts file and your system will look there if the systems are not listed in /etc/hosts or your DNS server.
 
##### Domain names  
.com .net .edu .org .io define the domain "purpose"  
  
Root -> . ; .com -> top level domain ; subdomaind -> www or maps or mail    
  
When asking for resolution, your system first goes to internal DNS.  If not found there then you are forwarded to an external DNS server and it is was forwared to different DNS serviers until the domain is resolved.   

Enable shortcut in /etc/resolv.conf
```
nameserver   192.168.1.100
search       mycompany.com prod.mycompmy.com
```
  
DNS Record Types   
How records are stored in DNS:  
A record types map hostname to an ip addresses 
AAAA record types map hostname to ipv6 addresses
CNAME Record types maps one hostname to another  hostname  - name to name mapping   

Other look up options:  
- nslookup - doesnt look at /etc/hsts
- dig 
  
  
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
  

