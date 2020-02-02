```

Disable SELinux or put it in permissive mode.  

Install Squid and Apache. We want Apache to help share the client-side certificate:

yum install squid httpd
Backup and empty your “/etc/squid/squid.conf” file and replace the contents with the following:

#Enable quick shutdown
shutdown_lifetime 0 seconds

#Enable transparent proxy with SSL bump
http_port  3120 intercept
https_port 3128 intercept ssl-bump generate-host-certificates=on dynamic_cert_mem_cache_size=16MB cert=/etc/squid/ssl_cert/myCA.pem
http_port  3129 ssl-bump generate-host-certificates=on dynamic_cert_mem_cache_size=16MB cert=/etc/squid/ssl_cert/myCA.pem

# Log stuff
#log_fqdn on
logformat common     %>a %[ui %[un [%tl] "%rm %ru HTTP/%rv" %>Hs %<st %Ss:%Sh
access_log /var/log/squid/access.log.simple common

#Configure SSL Bump for all sites
acl broken_sites dstdom_regex icicibank.com hdfcbank.com
acl monitor_domains dstdom_regex youtube.com facebook.com ytimg.com googlevideo.com ggpht.com
acl monitor_domains2 dst 216.58.196.110 216.58.199.174  #youtube connect works over IP
ssl_bump none localhost
ssl_bump none broken_sites   #Avoid bumping financial sites such as banks
ssl_bump server-first monitor_domains  #Bump facebook and youtube
ssl_bump server-first monitor_domains2  #Since youtube bump fails with just domain also add youtube serverIP

#Configure hostname
visible_hostname proxy.agix.local

#Configure logging of query terms
strip_query_terms off    #This will allow checking which youtube URLs were visited by user

http_access allow all
Create your server certificate:

cd /etc/squid
mkdir ssl_cert
chown squid:squid ssl_cert
chmod 700 ssl_cert
cd ssl_cert
openssl req -new -newkey rsa:2048 -sha256 -days 365 -nodes -x509 -keyout myCA.pem  -out myCA.pem
Create your client side (web browser) certificate:

cd /etc/squid
openssl x509 -in ssl_cert/myCA.pem -outform DER -out myCA.der
I’m not sure what this does but you need it:

openssl dhparam -outform PEM -out dhparam.pem 2048
Initialize the SSL certificate directory. The following error’d for me but still worked:

/usr/lib64/squid/ssl_crtd -c -s /var/lib/ssl_db
chown -R squid:squid /var/lib/ssl_db/
Start Squid:

systemctl enable squid
systemctl restart squid
systemctl status squid
Enable routing (ip forwarding). Create the file “/etc/sysctl.d/ip_forward.conf” and put the following contents into it:

net.ipv4.ip_forward=1
And run:

sysctl -p /etc/sysctl.d/ip_forward.conf
Add the firewall rules. In my case the default zone is “public” and the only zone i’m using.
But you will need to apply the rules to whatever is the appropriate zone:

firewall-cmd --zone=public --add-port=3128/tcp --permanent
firewall-cmd --zone=public --add-port=3129/tcp --permanent
firewall-cmd --zone=public --add-port=81/tcp --permanent

firewall-cmd --add-service http --permanent
firewall-cmd --add-service https --permanent

#For masquerading:
firewall-cmd --permanent --zone=public --add-masquerade
Note: the above ports allow squid to listen on both 3128 and 3129 and Apache (httpd) to listen on port 81. 
We want Apache listening on port 81 because 80 is reserved for Squid. 
We will use Apache to share the client side (web browser) certificate. Ie, 
tell your users to visit “http://proxy.agix.local/myCA.der” to get the certificate.

Note: The masquerading line above: If you had two interfaces (eth0 and eth1 for example) 
you might make eth0 in the “internal” zone, and eth1 in the “external” zone. For this example, 
we’re using just the one interface.

Add the following content to the file “/etc/firewalld/direct.xml”. Update the ethernet
devices as appropriate. Mine are “enp0s3” but yours might be “eth0” for example:

<?xml version="1.0" encoding="utf-8"?>
<direct>
<rule ipv="ipv4" table="nat" chain="PREROUTING" priority="0">-i enp0s3 -p tcp --dport 80 -j REDIRECT --to-ports 3128</rule>
<rule ipv="ipv4" table="nat" chain="PREROUTING" priority="0">-i ens0s3 -p tcp --dport 443 -j REDIRECT --to-ports 3129</rule>
</direct>
Reload the firewall:

firewall-cmd --reload
Copy the client side certificate to the Apache document root:

cp /etc/squid/myCA.der /var/www/html/
Configure Apache to listen on port 81. Edit the file “/etc/httpd/conf/httpd.conf” and change the line:

Listen 80
To this:

Listen 81
Restart Apache.

systemctl enable httpd
systemctl restart httpd
```
