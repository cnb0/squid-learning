 
### Setup Squid proxy as a transparent server.

Main benefit of setting transparent proxy is you do not have to setup up 
individual browsers to work with proxies.
http://proxy.ccu.edu.tw/squidfaq/FAQ-20.html

My Setup:
i) System: HP dual Xeon CPU system with 8 GB RAM (good for squid).
ii) Eth0: IP:192.168.1.1
iii) Eth1: IP: 192.168.2.1 (192.168.2.0/24 network (around 150 windows XP systems))
iv) OS: Red Hat Enterprise Linux 4.0 (Following instruction should work with Debian and 
all other Linux distros)

Eth0 connected to internet and eth1 connected to local lan i.e. system act as router.

Server Configuration
Step #1 : Squid configuration so that it will act as a transparent proxy
Step #2 : Iptables configuration
a) Configure system as router
b) Forward all http requests to 3128 (DNAT)
Step #3: Run scripts and start squid service
First, Squid server installed (use up2date squid) and configured by adding following directives to file:
# vi /etc/squid/squid.conf

Modify or add following squid directives:
httpd_accel_host virtual
httpd_accel_port 80
httpd_accel_with_proxy on
httpd_accel_uses_host_header on
acl lan src 192.168.1.1 192.168.2.0/24
http_access allow localhost
http_access allow lan

Where,

httpd_accel_host virtual: Squid as an httpd accelerator
httpd_accel_port 80: 80 is port you want to act as a proxy
httpd_accel_with_proxy on: Squid act as both a local httpd accelerator and as a proxy.
httpd_accel_uses_host_header on: Header is turned on which is the hostname from the URL.
acl lan src 192.168.1.1 192.168.2.0/24: Access control list, only allow LAN computers to use squid
http_access allow localhost: Squid access to LAN and localhost ACL only
http_access allow lan: — same as above —
Here is the complete listing of squid.conf for your reference (grep will remove all
comments and sed will remove all empty lines, thanks to David Klein for quick hint ):
# grep -v "^#" /etc/squid/squid.conf | sed -e '/^$/d'

OR, try out sed (thanks to kotnik for small sed trick)
# cat /etc/squid/squid.conf | sed '/ *#/d; /^ *$/d'

Output:
hierarchy_stoplist cgi-bin ?
acl QUERY urlpath_regex cgi-bin \?
no_cache deny QUERY
hosts_file /etc/hosts
refresh_pattern ^ftp: 1440 20% 10080
refresh_pattern ^gopher: 1440 0% 1440
refresh_pattern . 0 20% 4320
acl all src 0.0.0.0/0.0.0.0
acl manager proto cache_object
acl localhost src 127.0.0.1/255.255.255.255
acl to_localhost dst 127.0.0.0/8
acl purge method PURGE
acl CONNECT method CONNECT
cache_mem 1024 MB
http_access allow manager localhost
http_access deny manager
http_access allow purge localhost
http_access deny purge
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
acl lan src 192.168.1.1 192.168.2.0/24
http_access allow localhost
http_access allow lan
http_access deny all
http_reply_access allow all
icp_access allow all
visible_hostname myclient.hostname.com
httpd_accel_host virtual
httpd_accel_port 80
httpd_accel_with_proxy on
httpd_accel_uses_host_header on
coredump_dir /var/spool/squid

Iptables configuration
Next, I had added following rules to forward all http requests (coming to port 80) to the Squid server port 3128 :
iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 80 -j DNAT --to 192.168.1.1:3128
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 3128

Here is complete shell script. Script first configure Linux system as router and
forwards all http request to port 3128 (Download the fw.proxy shell script):
#!/bin/sh
# squid server IP
SQUID_SERVER="192.168.1.1"
# Interface connected to Internet
INTERNET="eth0"
# Interface connected to LAN
LAN_IN="eth1"
# Squid port
SQUID_PORT="3128"
# DO NOT MODIFY BELOW
# Clean old firewall
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X
# Load IPTABLES modules for NAT and IP conntrack support
modprobe ip_conntrack
modprobe ip_conntrack_ftp
# For win xp ftp client
#modprobe ip_nat_ftp
echo 1 > /proc/sys/net/ipv4/ip_forward
# Setting default filter policy
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
# Unlimited access to loop back
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
# Allow UDP, DNS and Passive FTP
iptables -A INPUT -i $INTERNET -m state --state ESTABLISHED,RELATED -j ACCEPT
# set this system as a router for Rest of LAN
iptables --table nat --append POSTROUTING --out-interface $INTERNET -j MASQUERADE
iptables --append FORWARD --in-interface $LAN_IN -j ACCEPT
# unlimited access to LAN
iptables -A INPUT -i $LAN_IN -j ACCEPT
iptables -A OUTPUT -o $LAN_IN -j ACCEPT
# DNAT port 80 request comming from LAN systems to squid 3128 ($SQUID_PORT) aka transparent proxy
iptables -t nat -A PREROUTING -i $LAN_IN -p tcp --dport 80 -j DNAT --to $SQUID_SERVER:$SQUID_PORT
# if it is same system
iptables -t nat -A PREROUTING -i $INTERNET -p tcp --dport 80 -j REDIRECT --to-port $SQUID_PORT
# DROP everything and Log it
iptables -A INPUT -j LOG
iptables -A INPUT -j DROP

Save shell script. Execute script so that system will act as a router and forward the ports:
# chmod +x /etc/fw.proxy
# /etc/fw.proxy
# service iptables save
# chkconfig iptables on

Start or Restart the squid:
# /etc/init.d/squid restart
# chkconfig squid on

Desktop / Client computer configuration
Point all desktop clients to your eth1 IP address (192.168.2.1) as 
Router/Gateway (use DHCP to distribute this information).
You do not have to setup up individual browsers to work with proxies.

How do I test my squid proxy is working correctly?
See access log file /var/log/squid/access.log:
# tail -f /var/log/squid/access.log

Above command will monitor all incoming request and log them
to /var/log/squid/access_log file. Now if somebody accessing a website through browser, 
squid will log information.

Problems and solutions
(A) WINDOWS XP FTP CLIENT
All Desktop client FTP session request ended with an error:
Illegal PORT command.

I had loaded the ip_nat_ftp kernel module. Just type the following command press Enter and voila!
# modprobe ip_nat_ftp

Please note that modprobe command is already added to a shell script (above).

(B) PORT 443 REDIRECTION
I had block out all connection request from our router settings except for our proxy (192.168.1.1) server. 
So all ports including 443 (https/ssl) request denied. You cannot redirect port 443, 
from debian mailing list, “Long answer: SSL is specifically designed to prevent “man in the middle” attacks,
and setting up squid in such a way would be the same as such a “man in the middle” attack. 
You might be able to successfully achive this, but not without breaking the encryption and 
certification that is the point behind SSL“.

Therefore, I had quickly reopen port 443 (router firewall) for all my LAN computers and problem was solved.

(C) SQUID PROXY AUTHENTICATION IN A TRANSPARENT MODE
You cannot use Squid authentication with a transparently intercepting proxy.

```
