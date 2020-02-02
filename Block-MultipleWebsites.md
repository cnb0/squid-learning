```
How to block multiple website with single acl, creating a time based acl & 
also speeding up our browsing by enabling cache.



Blocking Multiple websites
Firstly, we will create a file named blacksites (or bad-domains or whatever )

vi /etc/squid/blacksites

and add the websites we need blocked & save the file

.facebook.com
.youtube.com
.twitter.com

Now, open main configuration file

vi /etc/squid/squid.conf

and create a new acl

acl blacksites dstdomain /etc/squid/blacksites

then, we deny access to the created acl

http_access deny blacksites

lastly, restart proxy server to apply changes.

service squid restart

Note you can also use squid -k reconfigure to apply changes to server 
without actually restarting the server.

Time based acl

Sometimes, we might require access to a blocked website for a certain period of
time or we might need to block certain websites for certain time. 
This can be achieved using a time based acl

Firstly, open configuration file

vi /etc/squid/squid.conf

then create a new acl and allow access to the acl

acl timebased time MTW 10:30-11:30
http_access allow blacksites

lastly, restart your server to implement changes. & we now have access of 
blocked sites on Monday, Tuesday & Wednesday between 10:30AM to 11:30AM .

Enabling cache to speed up browsing
So, by enabling cache in our server we can speed up our browsing speed for frequently visited pages.

By adding just one line in our configuration file, we can enable cache.

To enable cache , open configuration file

vi /etc/squid/squid.conf

and add following line to bottom of the file

cache_dir ufs /var/cache/squid 2000 16 256

where ufs is squid storage format,

/var/cache/squid is path for cache storage,

2000 is size in MB can be used for cache,

and, 16 is number of 1st level sub-directories & 256 is 2nd level sub directories in cache folder.
