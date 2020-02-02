```
Squid Transparent proxy server : How to configure
 
 we have learned to install & configure squid proxy server. We will now, 
 we learn to configure Squid transparent proxy server. 
 With setting up Squid transparent proxy server, we have a 
 major advantage of not configuring proxy setting on every user’s machine. 
 Being transparent means that users will have no idea 
 that there requests are being passed through a proxy server.

Squid as transparent proxy acts as a gateway between internet and users. 
It redirects all the internet traffic from port 80 to squid proxy’s port i.e. 3128. 

So now let’s start with the setting squid as transparent proxy…

 

Installation


$ sudo yum install squid -y

 

Configuring squid

Next we need to enable IP Packet Forwarding on the machine, to do this

$ sudo vim /etc/sysctl.conf

then change the following parameter to ‘1’, i.e.

net.ipv4.ip_forward = 1

Save file & exit. Now execute the following command to implement the changes made,

$ sudo sysctl -p

Next, we will configure the squid proxy using it’s main configuration file i.e. ‘/etc/squid/squid.conf’,

$ sudo vim /etc/squid/squid.conf

& make changes as follows to the options mentioned,

http_access allow all
http_port 3128 intercept
visible_hostname squid.proxy

Now save the file & exit. Next to implement the changes restart the squid service,

$ sudo service squid restart

$ chkconfig squid on
 

Configuring firewall rules

All that remains in the configuration for setting up squid transparent proxy is firewall rules configuration. 
Firewalld rules for RHEL/CentOS 7 are ,

$ sudo firewall-cmd –permanent –zone=public –add-forward- port=port=80:proto=tcp:toport=3128:toaddr=192.168.1.10

$ sudo firewall-cmd –permanent –zone=public –add-port=3128/tcp

$ sudo firewall-cmd –permanent –add-masquerade

$ sudo firewall-cmd –reload

here, 192.168.1.10 is the LAN IP address of the squid proxy server.

For RHEL/CentOS 6, the Iptables rules are

$ sudo iptables -t nat -A PREROUTING -i eth0 -p tcp –dport 80 -j DNAT –to 192.168.1.10:3128
$ sudo iptables -t nat -A PREROUTING -i eth1 -p tcp –dport 80 -j REDIRECT –to-port 3128

$ sudo iptables –t nat -A POSTROUTING –out-interface eth1 -j MASQUERADE

After the changes have been made to firewall rules, our server is now ready to work as a squid transparent proxy.
All we have to do test this, is to change the gateway of the client machine 
to the IP address of squid server i.e. 192.168.1.10. When we access any website from client machine, 
it will first arrive at squid proxy server on port 80 & will then be redirected to port 3128, 
then after analysing the ACLs, traffic will be forward to WAN or internet.
