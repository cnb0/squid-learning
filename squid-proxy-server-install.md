```
Squid Proxy Server- Installation & Configuration 
 
Squid Proxy Server, which is a widely used Open Source web proxy.
 

Web Proxy
A proxy is an intermediary/middle-agent between computer/computers & other resources, mostly internet. 
It seeks requests from client & transfer them to internet.

Benefits of a Web Proxy
It can be used to accelerate the internet as a proxy can build up a cache of frequently used websites,
which makes it easier & faster to load up after,
Can be used to block/allow websites as required,
also can be used to bypass another web proxy . For example in many organizations Social networking websites
like Facebook, Twitter , Youtube etc are not allowed. So a web proxy can be used to bypass 
those restrictions & provide access to restricted websites.

Squid proxy server

Its a caching proxy server which supports HTTP, HTTPS, FTP . 
It can be used as an accelerating server, thereby decreasing response time & 
reducing bandwidth. It can also be used for the purpose of Web filtering 
due to availability of extensive access controls.

And we will be exploring web filtering part 

Scenario Setup
----------------------------
Firstly, to test or create a squid proxy setup, we will need a squid server & a client machine.

Squid server                                                              Client’s Machine

OS : Centos/RHEL 6 or 7                                       OS: Centos/RHEL 6 or 7

Hostname : server.test.com                                   Hostname: client1.test.com

IP Address :192.168.1.100                                      IP Address : 192.168.1.101

Important
Configuration file       /etc/squid/squid.conf

Default port                 3128

  
 

Installation
 

yum install squid -y

 

Configuration
We need to create an ACL rule (Access Control List), 
which is the list or rule with list of access control entries.
Some acl rules are already written in configuration file by default in the configuration file,

acl localhost src 127.0.0.1/32
http_access allow localhost                                            

So, this is what an acl rule look like. Lets see what these means,

firstly,acl this is declaring that a new acl is starting

then,localhost is the name of acl created

src is used in case acl is for local Ipadress , srcdomain is used for declaring Localdomain, 
dst for public IP & dstdomain for publlic domain name
and lastly,127.0.01/32 declares the IP Address on which the acl is to be applied, 
in this case its localhost or 127.0.0.1

 

Next line i.e. http_access allow localhost, means

http_access will initiate an action based on next word

allow/deny will either allow or deny access

and,localhost again is the name of acl as declared above.

 

So, basically that how we create a ACL/rule in squid proxy server.

Now, lets restart our server (with default config file) & configure the client machine
to see if proxy is working properly.
service squid restart
chkconfig squid on

Note  Its always wise to have a backup of original configuration file when 
starting to make changes. So, create a backup a backup of before starting.

 Configuration on Client Side
Open Firefox Browser &

Open Edit menu —> Preferences —> Advanced —-> Settings
Check the box ‘ Manual proxy configuration’ & enter IP Address & Port Number of squid proxy server.
In our case its 192.168.1.100 & 3128.

Click OK
& that’s all we need to configure on Client’s side.

Then we check out if its works. Open a website (example Facebook.com),
if proxy server is working properly you will be greeted with an error ‘ Access Denied’. 
That’s because by default internet access is denied for all in server.

Now, lets check logs in server, to see if a request was received by proxy server or not,

tail -f /var/log/squid/access.log

and it should show you all the received requests from client to server.

Restricting access to websites
In order to restrict access to a website, open configuration file & then create a new acl
acl blacksite .facebook.com

and deny access to the acl

Note Also set “http_access deny all” to “http_access allow all “, 
otherwise we wont be able to access internet.

Now, restart your squid proxy server to apply changes or 
we can also use squid -k reconfigure to implement changes to server without restarting the server.

then, we will access client’s machine and open Facebook but you wont be able to
access it at all. As for other websites you can access them just fine.
