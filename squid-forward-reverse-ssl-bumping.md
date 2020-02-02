```
About
Squid is a very powerful proxy server app with very little and awful documentation.

Some steps to get started:

Official reference - mostly for setting config directives, knowing the latest version options etc.



 

See more links at the end of this guide!

Installation
Just install through your package manager. Check your version e.g. 3.5, 4. 
Enable with systemctl enable --now squid. Check if it crashed with systemctl status 
squid since it won't display any info in the command line. Check logs in /var/log/squid - 
they only show up once squid has started.

Your magic command will be squid -k reconfigure which reloads the config file after any changes.

Setting up

The key is getting the config file /etc/squid/squid.conf right.  .

The config file consists of directives. They do 3 things:

Who and how can access the proxy, and what they can access:

acl: define access control lists (source, destination, protocol etc.),

http_access: control access for ACLs (checked in order, first rejection rejects request)

How the proxy can be reached and what it actually does to incoming requests:

http_port, https_port: where the proxy listens

ssl_bump: splice, peek and bump (intercept/inspect) some SSL connections

cache_peer: forward some requests to another (caching) proxy

Misc I/O, caching & debugging stuff:

logformat, access_log: specify logging

refresh_pattern, cache_dir: configure caching

debug_options: additional debug logging

Modes of use
Normal forward proxy: clients connect to the internet through this. Squid: http_port

Reverse / acceleator proxy: sits in front of servers to cache and route data. Squid: http_port accel

Transparent / intercepting proxy: requests are routed to this with a firewall / iptables without the client knowing. Squid: http_port intercept, https_port ssl_bump intercept

Examples
Example 1: simple forward proxy for web crawlers
acl SSL_ports port 443
# Ports where clients can connect to.
acl Safe_ports port 80		# http
acl Safe_ports port 21		# ftp
acl Safe_ports port 443		# https
acl Safe_ports port 70		# gopher
acl Safe_ports port 210		# wais
acl Safe_ports port 1025-65535	# unregistered ports
acl Safe_ports port 280		# http-mgmt
acl Safe_ports port 488		# gss-http
acl Safe_ports port 591		# filemaker
acl Safe_ports port 777		# multiling http
acl CONNECT method CONNECT

# if connection is not to any of this port, Sqiud rejects. otherwise check the next rule.
http_access deny !Safe_ports

# Squid cache manager app
http_access allow localhost manager
http_access deny manager

# localhost is allowed. if source is not localhost, squid checks the next rule
http_access allow localhost
# only allow these destination domains (1 domain per line)
acl allowed_domains dstdomain "/etc/squid/domain_whitelist.txt"
# deny picures, videos etc. to save bandwidth
acl notallowed_resources urlpath_regex -i \.(avi|mp4|mov|m4v|mkv|flv|css|jpg|png|gif|eps)(\?.*)?$
# only some IPs can use the proxy
acl allowed_clients src "/etc/squid/allowed_clients.txt"
http_access deny notallowed_resources
http_access allow allowed_clients allowed_domains

# IMPORTANT LINE: deny anything that's not allowed above
http_access deny all

# listen on this port as a proxy
http_port 3128

# memory settings
cache_mem 512 MB
coredump_dir /var/spool/squid3

refresh_pattern ^ftp:		1440	20%	10080
refresh_pattern ^gopher:	1440	0%	1440
refresh_pattern -i (/cgi-bin/|\?) 0	0%	0\
# refresh_pattern [-i] regex min percent max [options]
# here, . means 'any link'. Cache for at least 0, at most 20160 minutes, ot 50% of its age since 'last-modified' header.
refresh_pattern .		0	50%	20160

# delete x-forwarded-for header in requests (anonymize them)
forwarded_for delete
Example 2: intercepting proxy with SSL bumping
HTTPS requests are:

Safe and anonymous, impossible to listen to.

The proxy or anybody in between only knows the destination domain for helping with routing.

Proxies cannot intercept or cache it, they can only tunnel (proxy) them to the destination.

But they can be bumped: instead of creating a secure tunnel (like most proxies), the proxy may intercept the connections between client and server, creating 2 connections and forwarding data. The client will know about it, since the proxy's certificate authority (CA) will not be the same as the server's.

Bumping SSL (HTTPS):

Allows intercepting and caching individual HTTPS requests

May not work for all requests due to TLS compatibility issues

May be illegal, since technically this is the same as a man-in-the-middle attack!

Violates web standards

The client will know about it unless their computer was preconfigured to accept self-signed certs which are not signed by 'well-known' root CAs. (You can get a well-known certificate from e.g. Let's Encrypt, but only for your own domains!)

Obtaining SSL key
Install openssl. Then:

mkdir -p /etc/squid/cert/
cd /etc/squid/cert/
# This puts the private key and the self-signed certificate in the same file
openssl req -new -newkey rsa:4096 -sha256 -days 365 -nodes -x509 -keyout myCA.pem -out myCA.pem
# This can be added to browsers
openssl x509 -in myCA.pem -outform DER -out myCA.der
Then assign the above files and folders to the squid user.

Initialize SSL database
With the below config, Squid will generate a new 'fake' self-signed certificate for each bumped SSL connection (that the clients will hate). These will be cached in a folder.

On Fedora 29, it can be done with:

sudo -u squid /usr/lib64/squid/security_file_certgen -c -s /var/spool/squid/ssl_db -M 4MB
(This is the default directory. If you try to start Squid with SSL signing without initializing this folder, it will crash, and you can get some guidance with systemctl status squid)

Config file
acl localnet src 0.0.0.1-0.255.255.255	# RFC 1122 "this" network (LAN)
acl localnet src 10.0.0.0/8		# RFC 1918 local private network (LAN)
acl localnet src 100.64.0.0/10		# RFC 6598 shared address space (CGN)
acl localnet src 169.254.0.0/16 	# RFC 3927 link-local (directly plugged) machines
acl localnet src 172.16.0.0/12		# RFC 1918 local private network (LAN)
acl localnet src 192.168.0.0/16		# RFC 1918 local private network (LAN)
acl localnet src fc00::/7       	# RFC 4193 local private network range
acl localnet src fe80::/10      	# RFC 4291 link-local (directly plugged) machines

acl SSL_ports port 443
acl SSL_ports port 22225
acl Safe_ports port 80		# http
acl Safe_ports port 21		# ftp
acl Safe_ports port 443		# https
acl Safe_ports port 70		# gopher
acl Safe_ports port 210		# wais
acl Safe_ports port 1025-65535	# unregistered ports
acl Safe_ports port 280		# http-mgmt
acl Safe_ports port 488		# gss-http
acl Safe_ports port 591		# filemaker
acl Safe_ports port 777		# multiling http
acl CONNECT method CONNECT

sslproxy_cert_error allow all

# Deny requests to certain unsafe ports
http_access deny !Safe_ports

# Deny CONNECT to other than secure SSL ports
http_access deny CONNECT !SSL_ports

# Only allow cachemgr access from localhost
http_access allow localhost manager
http_access deny manager

# Adapt localnet in the ACL section to list your (internal) IP networks
# from where browsing should be allowed
http_access allow localnet
http_access allow localhost

# And finally deny all other access to this proxy
http_access deny all

# Squid normally listens to port 3128
# Needs at least 1 regular port to listen (otherwise Squid will crash -_-)
http_port 3128
# Listen on this HTTP port, intercepting requests
http_port 3129 intercept
# intercept & bump SSL connections
https_port 3130 intercept ssl-bump \
 generate-host-certificates=on \
 dynamic_cert_mem_cache_size=4MB \
 cert=/etc/squid/cert/myCA.pem \
 key=/etc/squid/cert/myCA.pem

# SSL bump instructions
# Define SSL connections steps
acl step1 at_step SslBump1
acl step2 at_step SslBump2
acl step3 at_step SslBump3

#ssl_bump peek step1    # <- enabling this breaks it
ssl_bump stare step2
ssl_bump bump step3
# Uncommenting this may also break bumping.
#ssl_bump bump all

# Usually you don't need to set these, but in case you want to tweak defaults:
#sslcrtd_program /usr/lib64/squid/security_file_certgen \
#  -s /var/cache/ssl_db/db \
#  -M 4MB
#sslcrtd_children 8 startup=1 idle=1

# Uncomment and adjust the following to add a disk cache directory.
cache_dir ufs /var/spool/squid 100 16 256

# Leave coredumps in the first cache dir
coredump_dir /var/spool/squid

refresh_pattern ^ftp:		1440	20%	10080
refresh_pattern ^gopher:	1440	0%	1440
refresh_pattern -i (/cgi-bin/|\?) 0	0%	0
refresh_pattern .		0	20%	4320

# Logging with our own logformat
logformat agix %>a %>A %ul %ru %>Hs
access_log /var/log/squid/access.log.simple agix

# Remove forwarded-for header
forwarded_for delete
Fix clients
Clients hate self-signed certs for good reasons.

Browsers: use the myCA.der file to import the certificate.

Linux: copy the cert files to /etc/pki/ca-trust/source/anchors and 
then run update-ca-trust or whatever you have on your distribution, YMMV.

Node.js: Node will hate self-signed certs and you will have to modify 
the default HTTPS agent (locally or globally) or each request's options. You have 2 options:

Read the PEM file and add the Buffer to the ca option.

Set rejectUnauthorized to false - this is unsafe, allows man-in-the-middle attacks.

If using the request library, set strictSSL to false.

Headless Chrome (Puppeteer): start Chromium with --ignore-certificate-errors, 
start Puppeteer with ignoreHTTPSErrors set to true.

Divert traffic to the transparent proxy with iptables
From other computers, we use the PREROUTING chain, specifying the source with -s:

iptables -t nat -A PREROUTING -s 192.168.0.0/2 -p tcp --dport 80 -j REDIRECT --to-port 3129
iptables -t nat -A PREROUTING -s 192.168.0.0/2 -p tcp --dport 443 -j REDIRECT --to-port 3130
On localhost this is a tougher issue since we want to avoid forwarding loops 
(packet is diverted to Squid but it should be sent to the Internet when 
Squid done its thing). 
Fortunately iptables can differentiate between packet owner users. 
We need to use the OUTPUT chain for locally-generated packets. 
So we allow packets by root and squid through and divert everything else to Squid.

iptables -t nat -A OUTPUT -p tcp -m tcp --dport 80 -m owner --uid-owner root -j RETURN
iptables -t nat -A OUTPUT -p tcp -m tcp --dport 80 -m owner --uid-owner squid -j RETURN
iptables -t nat -A OUTPUT -p tcp -m tcp --dport 80 -j REDIRECT --to-ports 3129
iptables -t nat -A OUTPUT -p tcp -m tcp --dport 443 -m owner --uid-owner root -j RETURN
iptables -t nat -A OUTPUT -p tcp -m tcp --dport 443 -m owner --uid-owner squid -j RETURN
iptables -t nat -A OUTPUT -p tcp -m tcp --dport 443 -j REDIRECT --to-ports 3130
Debugging
In the config, set debug_options. E.g. some general debug logs and detailed ACL info:

debug_options ALL,2 28,9



