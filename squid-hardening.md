```
cache_effective_user and cache_effective_group: even though the user and group are default to nobody. 
It is highly recommended to execute Squid under a dedicated (sandbox) user, with no shell privileges.

cache_effective_user squid_user
cache_effective_group squid_group

http_port: Port where Squid listens for incoming requests. 
By default, 3128, as we did for SSH, we should change it. 
If the server where Squid is hosted has several network interfaces, 
it is also possible to restrict to an IP address on which Squid listens to.
http_port [ip_adress:]port

ACLs (this is the most critical part) will define whether a particular HTTP request is permitted or denied. 
They are defined by two lines: ACL definition line, which usually follows the format acl name type 
decission_string and ACL rule line, which usually follows the format rule deny|allow [!]acl_name [[!]acl_name ...]. 
The rules are processed as a short-circuit evaluation, first match and stops processing. 
It is worth mentioning that if none of the rules are matched, then the default action is
the opposite of the last rule in the list. For further information, I highly recommend to read the Squid FAQ - ACL,
the configuration manual entry and this Web site (you can find it in the references as well.)
Let's introduce the most basic ones:

Restrict the access to your network:

acl mynetwork src 192.168.1.0/24
http_access allow mynetwork


Define the trusted ports:
acl trusted_ports port 21 80 443
http_access deny !trusted_ports

To avoid weird behaviour on the rules processing, remember that if none of the rules are matched, 
then the default action is the opposite of the last rule in the list. Add as a last rule line:

http_access deny all

Authentication: (you can skip this part if you won't use user authentication) 
Squid supports various authentication schemes (NTLM, Digest and Basic). 
However, the user authentication is performed via external modules (NCSA, MS Windows, RADIUS, etc.) 
Let's see a basic example (Digest with NCSA for "mynetwork", defined above), 
where the first rule indicates the scheme and authentication module, 
the second the number of authenticatior processes spawn, the third the realm:

acl trusted_users proxy_auth REQUIRED
http_access allow mynetwork trusted_users
auth_param digest /usr/lib/squid3/ncsa_auth /etc/squid3/i_just_put_here_the_passwd
auth_param digest children 5
auth_param digest realm Squid proxy-caching web server

Session expiration (or TTL). 
There are two options: authenticate_ttl and authenticate_ip_ttl. 
The first will set how long Squid will remember client authentication information, and 
the second how long a client’s authentication should be bound to a particular IP address.
The purpose of this parameter is to discourage people from sharing their passwords among themselves.
Be careful with last parameter and roadworkers, it could block legitimate users.

authenticate_ttl 1 hour
authenticate_ip_ttl 0 seconds

FTP gatewaying: ftp_user, ftp_passive and ftp_sanitycheck. 
If you want the anonymous login password to be more informative, 
set ftp_user to something reasonable for your domain. 
This email address will be used to report any abuse coming from your proxy server. 
The passive mode is considered to be more secure because it uses 2 fixed ports; 
one for connection and one for data transfer. 
Sanity check option uses an extensive mechanism to ensure the connection 
is established with the requested server.

ftp_user my_email@adress.com
ftp_passive on
ftp_sanitycheck on

client_lifetime sets the maximum time a client is allowed to be bound to a Squid process. 
Set this option to less than 24h.
client_lifetime 1 day

pconn_timeout sets the maximum time an idle client is allowed to be bound to a squid process. 
Set this option to less or equal to 120s.

pconn_timeout 1 minute

Limit the size of acceptable HTTP headers.  
Most denial of service attacks against proxy servers are attempted by sending them headers 
larger than what they can handle. This option should therefore be left to its default value and never be disabled.

request_header_max_size 64 KB

forwarded_for: Enabling this option allows to Squid adds its own HTTP header: 
HTTP_FORWARD_FOR to add the client address. Unless it is needed, because certain
Web sites require the client address, it is recommended to turn it off. 
This practice gives you the advantage of hiding the client’s IP address on the internet.

forwarded_for off


ignore_unknown_nameservers: This option verifies if the nameserver 
answering the lookup has the same IP address than the one the lookup
was sent to. This makes it much more difficult for an attacker to falsify DNS queries.
This option is on by default and it is recommended to leave as it is.
ignore_unknown_nameservers on

Cache configuration, we need to take into account the following options:
Limits on the size of cached objects. The primary goal of this option is to 
optimize the cache/HIT ratio. This option is also interesting in a security 
standpoint since it helps to protect against a denial of service.

maximum_object_size value
minimum_object_size value

Cachemgr.cgi: It is a CGI utility for displaying statistics about the Squid 
HTTP proxy process as it runs. We need to configure our Web server (Apache configuration, IIS, etc.) 
along with Squid to limit its access:

acl manager proto cache_object
acl localhost src 127.0.0.1/255.255.255.255
acl webserver src 192.168.1.X/255.255.255.255 # webserver IP
http_access allow manager localhost
http_access allow manager webserver
http_access deny manager

Disable it completely.
acl all src 0.0.0.0/0.0.0.0
cache deny all

ICP and HTCP ports are used to communicate with other caches in a hierarchy.
In most cases we don’t need these ports open, specifically if we've disabled the cache.
icp_port 0
icp_access deny all
htcp_port 0
htcp_access deny all

SNMP port in the same way we did with ICP and HTCP ports, if it is not required, disable it.
snmp_port 0
snmp_access deny all
