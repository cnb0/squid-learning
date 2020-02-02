```
The goal was to build two proxy node to have distributed workload, the two proxy exchange 
their cache informations using the ICP protocol (Internet Cache Protocol), 
this is an UDP based protocol that use UDP port 3130 to exchange data availability 
between the two or more proxy.

The same Squid proxy configuration is deployed on two host,  

So here is the how to:
 

First thing is to specify our access list, this include any authorized host
that is allowed to use the Squid proxy server.

   acl localnet src 192.168.60.0/24
   acl partner src 192.168.60.3/32
   acl SSL_ports port 443
   acl Safe_ports port 80          # http
   acl CONNECT method CONNECT
 
    http_access deny !Safe_ports
    http_access deny CONNECT !SSL_ports
    http_access deny all
     “partner” is the second proxy host that is participating in the ICP exchange.

Specify the cache directory, the file system type, the size and the structure. 
In this case it’s an 256GB LV partition.

   cache_dir aufs /cache 262144 16 256

we use aufs to avoid I/O locking.

Reserve 1GB of memory to Squid internal usage, this include, frequently used and object in transit

cache_mem 1024 MB

This parameter allow the proxy server to continue downloading the file even
if the user canceled it’s request. 
This allow the proxy to have a partial copy in the case the user requested the object again.

quick_abort_min 1024 KB

Allow to read ahead of the user request, so the proxy have a ready copy of the object.

read_ahead_gap 512 KB

For anonymity, we can use this parameter in the configuration file.
iptables -I INPUT 4  -p tcp -m state --state NEW -m tcp --dport 3128 -j ACCEPT
61
iptables -I INPUT 4  -p udp -s 192.168.60.4 -d 192.168.60.3 --sport 3130 --dport 3130 -j ACCEPT
62
​
63
This delete the “X-Forwarded-For:” in the HTTP header request.

forwarded_for delete

Specifying this parameter in the Squid configuration file, will allow it to run under the following user.

cache_effective_user squid

Here we specify the ICP port, the allowed partner which we have configured using access list and 
then the cache peer that participate in the ICP exchange.

icp_port 3130
icp_access  allow  partner
cache_peer 192.168.60.4 sibling 3128 3130

Note that if you change the cache storage directory or the log storage location, 
you should also update the SElinux context file, permission and ownership for those directories.

# chcon –R –t squid_cache_t /cache
# chown –R squid:squid /cache

I also modified the iptables configuration to allow only the proxy peer to
iptables -I INPUT 4  -p tcp -m state --state NEW -m tcp --dport 3128 -j ACCEPT
61
iptables -I INPUT 4  -p udp -s 192.168.60.4 -d 192.168.60.3 --sport 3130 --dport 3130 -j ACCEPT
62
​
63
communicate with each other using the ICP protocol.

iptables -I INPUT 4  -p tcp -m state --state NEW -m tcp --dport 3128 -j ACCEPT
iptables -I INPUT 4  -p udp -s 192.168.60.4 -d 192.168.60.3 --sport 3130 --dport 3130 -j ACCEPT


Save the iptable configuration

save iptabes configuration

After the change in the Squid proxy server configuration file, restart the service for change to take effect.

restart squid proxy server

The Squid documentation section offer a complete explanation of all the parameters you can use in your Squid proxy configuration file.
