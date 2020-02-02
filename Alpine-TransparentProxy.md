Setting up Transparent Squid Proxy 
This document covers how to set up squid as a transparent proxy server.

The following is assumed:

You have already install a server running Alpine Linux as a base, with Alpine 1.10.6 or later.
Your proxy server will reside in a DMZ zone, separated from the network segment your clients are in.
Other implementations are covered here.
In order to transparently redirect web traffic from your clients to the proxy server in the DMZ, 
you will need to configure your intercepting router to DNAT traffic.
Note: If you're looking to setup an explicit squid proxy, or don't know what the differences are, 
please see this wiki page

Install Squid
apk add acf-squid

Configure squid with at least the following configuration:

Note: substitute proxy.example.com and the mynet acl for your own desired hostname 
and network subnet respectively
## This makes squid transparent in versions before squid 3.1
#http_port 8080 transparent
## For squid 3.1 and later, use this instead
http_port 8080 intercept
## Note that you need Squid 3.4 or above to support IPv6 for intercept mode. Requires ip6tables support

visible_hostname proxy.example.com
cache_mem 8 MB
cache_dir aufs /var/cache/squid 900 16 256

# Even though we only use one proxy, this line is recommended
# More info: http://www.squid-cache.org/Versions/v2/2.7/cfgman/hierarchy_stoplist.html
hierarchy_stoplist cgi-bin ?

# Keep 7 days of logs
logfile_rotate 7

access_log /var/log/squid/access.log squid
cache_store_log none
pid_filename /var/run/squid.pid

# Web auditors want to see the full uri, even with the query terms
strip_query_terms off

refresh_pattern ^ftp:		1440	20%	10080
refresh_pattern ^gopher:	1440	0%	1440
refresh_pattern -i (/cgi-bin/|\?) 0	0%	0
refresh_pattern .		0	20%	4320

coredump_dir /var/cache/squid

# 
# Authentication
#

# Optional authentication methods (NTLM, etc) can go here

#
# Access Control Lists (ACL's)
#

# These settings are recommended by squid
acl shoutcast rep_header X-HTTP09-First-Line ^ICY.[0-9]
upgrade_http0.9 deny shoutcast
acl apache rep_header Server ^Apache
broken_vary_encoding allow apache

# Standard ACL settings
acl QUERY urlpath_regex cgi-bin \? asp aspx jsp
acl all src all
acl manager proto cache_object
acl localhost src 127.0.0.1/32
acl to_localhost dst 127.0.0.0/8
acl SSL_ports port 443 563 8004 9000
acl Safe_ports port 21 70 80 81 210 280 443 563 499 591 777 1024 1022 1025-65535
acl purge method PURGE
acl CONNECT method CONNECT

# Require authentication
#acl userlist  proxy_auth REQUIRED
acl userlist  src 0.0.0.0/0.0.0.0

# Definition of network subnets
acl mynet src 192.168.0.0/24

#
# Access restrictions
#

cache deny QUERY

# Only allow cachemgr access from localhost
http_access allow manager localhost
http_access deny manager

# Only allow purge requests from localhost
http_access allow purge localhost
http_access deny purge

# Deny requests to unknown ports
http_access deny !Safe_ports

# Deny CONNECT to other than SSL ports
http_access deny CONNECT !SSL_ports

# Allow hosts in mynet subnet to access the entire Internet without being
# authenticated
http_access allow mynet

# Denying all access not explicitly allowed
http_access deny all

http_reply_access allow all
icp_access allow all
Check squid with:

squid -k reconfigure

Start squid:

/etc/init.d/squid start

Add squid to boot-up sequence:

rc-update add squid default

Remember to add port 8080 to the permitted ports clients can connect on to any
firewalls on your proxy server or in-between the proxy and the clients.

If you are running an Alpine Linux firewall on the firewall separating the Proxy 
from the clients, you will need to redirect all traffic from your client subnet on port 80 
to the proxy server on port 8080 to allow web traffic to be proxied.

If you are running shorewall, add this to your /etc/shorewall/rules file:

Note: Substitute loc and dmz:172.16.1.2:8080 for your client subnet zone and proxy 
server zone and IP as defined in /etc/shorewall/zones.
## This forces all web traffic to be redirected to the proxy on port 8080
DNAT    loc      dmz:172.16.1.2:8080     tcp    80
And restart shorewall with:

shorewall check

shorewall restart

Alternatively, you can configure Squid to listen on port 80. With this method,
it is usual for either the proxy to be configured 'in-line' so that due to physical 
cabling traffic must pass through the proxy (in this instance Squid will usually run 
on the same physical box as a router, which will be the clients' default gateway), or
alternatively (as described above) traffic on port 80 is re-directed by a router or 
firewall to the proxy (but remains on port 80). WCCP on a Cisco router is one method 
to redirect the traffic in this way.
