```
This article demonstrates how to configure a Squid transparent proxy. 
We’re using CentOS or Redhat here but the configuration its self will work on any distribution. 
Note that Debian related distributions call it “squid3” while Redhat related distributions just call it “squid”.

A few extra notes. We’re going to be logging in a simplified way to “/var/log/squid/access.log.simple”. 
We’re using the network “192.168.0.0/24”. We’re not touching SSL/HTTPS. 
This server is both the router and proxy. It must be able to resolve DNS names and get to the Internet.
Clients should use this proxy/router as their default gateway. 
The IP address of this server (in my examples) is “192.168.0.2”.

Install squid:

yum install squid
Enable Squid:

# CentOS/Redhat 6
chkconfig squid on
# CentOS/Redhat 7
systemctl enable squid
Make a backup of the config file:

cp /etc/squid/squid.conf /etc/squid/squid.conf.original
Remove the contents of the config file “/etc/squid/squid.conf” and replace it with the following:

http_port 3128 transparent

logformat agix %>a %>A %ul %ru %>Hs
access_log /var/log/squid/access.log.simple agix

acl mylan src 192.168.0.0/255.255.0.0

http_access allow mylan
http_access deny all

coredump_dir /var/spool/squid
Restart Squid with the following:

# CentOS/Redhat 6
service squid restart
# CentOS/Redhat 7
systemctl restart squid
Make sure it’s running with either of these commands:

# On any distribution
ps aux | grep squid
# CentOS/Redhat 6
service squid restart
# CentOS/Redhat 7
systemctl restart squid
Now we need to set IPTables to redirect packets passing through on port 80 to the proxy port 3128.
Here’s a sample that you can use to modify your IPTables. In the IPTables example below, 
the IP address of the server that’s running Squid (and IPTables) is “192.168.0.2”.
Squid is listening on its default port of “3128” and we’re using interface “eth0” only. 
We’re not doing anything with SSL (port 443) with Squid. To do that you need to either use 
a normal non-transparent proxy or do some fancy things with certificates.

*nat
:PREROUTING ACCEPT [14904813:1139520557]
:OUTPUT ACCEPT [433364:32375497]
:POSTROUTING ACCEPT [87113:5963902]
-A PREROUTING -s 192.168.0.0/24 -p tcp -m tcp --dport 80 -j DNAT --to-destination 192.168.0.2:3128
-A PREROUTING -s 192.168.0.0/24 -p tcp -m tcp --dport 80 -j REDIRECT --to-ports 3128
-A POSTROUTING -o eth0 -j MASQUERADE
COMMIT
*filter
:INPUT ACCEPT [641269:51209011]
:FORWARD ACCEPT [40492249:23342309940]
:OUTPUT ACCEPT [19321861:14820738577]
-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
-A INPUT -s 192.168.0.0/24 -p tcp -j ACCEPT
-A INPUT -s 127.0.0.1/32 -p tcp -j ACCEPT
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -j ACCEPT
COMMIT
Restart IPTables:

# CentOS/Redhat 6
service iptables status
# CentOS/Redhat 7
systemctl status iptables
Now we need to make sure this server is configured to route. Add the following text to the “/etc/sysctl.conf” file:

net.ipv4.ip_forward = 1
And apply it:

sysctl -p /etc/sysctl.conf
And we’re done. If you have trouble check if SELinux is causing an issue by monitoring the “/var/log/audit/audit.log” file.

tail -f /var/log/audit/audit.log
```
