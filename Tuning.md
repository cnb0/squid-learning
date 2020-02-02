 
``` 
Tuning Squid will speed things up a little bit. 

directives 

pipeline_prefetch on
shutdown_lifetime 1 second

While pipeline_prefetch will boost the performance of pipelined requests to closer 
match that of a non-proxied environment, the second directive shutdown_lifetime saves 
you a lot of time waiting for Squid to shut down. The latter comes in very handy
if you’re tweaking Squid and need to restart it a lot.

Even though Squid is meant as a cache there are reasons 
running it without a cache, i.e. as a pure forwarding proxy: 
you might want to use it as a load balancer with some parent proxies, 
simply as a transparent proxy or you don’t have particularly fast hardware. 
There are two methods to circumvent caching:

Deny caching for all connections:
acl all src 0.0.0.0/0.0.0.0
no_cache deny all
This way neither a request will be satisfied from the cache nor the reply will be cached. 
Note that the first line might already be in your configuration.

If you use a parent proxy you can specify the proxy-only 
option to prevent that retrieved data from the remote cache is stored locally. An example:
cache_peer proxy.isp.com parent 8080 0 proxy-only
Finally you might want to turn off logging. On a Debian based system 
it’s sufficient to turn of cache_access_log and cache_store_log:

cache_access_log none
cache_store_log none
Hardening
When talking about hardening I think about turning off features that aren’t used and

restricting access to the proxy. Features that aren’t used might be ICP and HTCP: 
they are used to communicate with other caches in a hierarchy. 
In most cases we don’t need this:

icp_port 0
htcp_port 0
icp_access deny all
htcp_access deny all

If you don’t wish to use SNMP we can disable this too. 
This is already the default for systems running Debian.

snmp_port 0
snmp_access deny all
At last you definitely want to restrict access to your proxy: define an access control list (acl) and
either allow or deny access with http_access. 

Lets say your LAN is 172.16.0.0/24 and 172.16.1.0/24. Then you would put the following into squid.conf:

acl LAN src 172.16.0.0/24 172.16.1.0/24
http_access allow LAN
 
If somebody outside your network tries to access your proxy 
he’ll get an error message that he isn’t allowed to do so.
```
