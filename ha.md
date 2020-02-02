```
What is the recommended approach to perform load balancing and
high availability between N squid servers?
I have the following list of requirements to fullfil:

1) Manage N squid servers that share cache (as far as i understand is done using cache_peer). Desirable.
2) Availability: if any of the N servers fails the clients are redirected to the rest N-1. Prefferable.
3) Scalability: The load is distributed (round-roubin or some other algorithm) between N servers,
if a new server is added (N + 1) new clients will be able to use it reducing the load on the rest N. 
Prefferable.
4) I need to be able to identify client IP addresses on the squid side and/or
perform Squid authentication. The client IP and User Name are later to be passed 
to the ICAP server to scan the HTTP(S) request/response contents using 
icap_send_client_ip and icap_send_client_username. Very important requirement.

5) I need to support both HTTP and HTTPS connections with support of selective SSL Bump.
I.e. for some web sites I do not want to look inside SSL so that original site's 
certificates are used for encryption. Very important requirement too.

I know that strictly for HTTP I could use HAProxy with Forward-For or something similar, 
but the 5th requirement is very important and I could not find a way to handle SSL with HAProxy properly.

The only idea that comes to my mind is to use some form of round-robin load balancing on the level of DNS,
but it has its own drawbacks (should be able to check availability of my N servers + 
not real balancing, more like distribution).


Solution :

1 - you are likely going to want to create a config for peers:
acl peers src 192.168.1.1/32
acl peers src 192.168.1.2/32
...
http_access allow peers
...
cache_peer 192.168.1.1 sibling 3128 4827 htcp=no-clr
cache_peer 192.168.1.2 sibling 3128 4827 htcp=no-clr
...

remember to make sure all peers are properly represented

2 - use a load balancing device or software, and this is part-and-parcel to that functionality

3 - load balancers give you several options.  i use "least connections" which is a bit more
intelligent that straight round-robin.

4 - depending on how you setup your load balancer, you might be able to get the client IP without playing games.  
if you cant see the client IP as the source of the connection, you will have to work with the "X-Forwarded-For" header, 
like so:
...
follow_x_forwarded_for allow svc_chk
follow_x_forwarded_for deny all
...
acl_uses_indirect_client on
...
log_uses_indirect_client on
...

Also, your auth methods may have nuances that need to be accounted for 
(Kerberos and load balancing requires some extra steps).

 5 - HAProxy will work for HTTP and HTTPS.  remember, your clients arent talking to HAProxy for HTTP proxying.  
 the port you load balance on with HAProxy is not the HTTP proxy process.  
 All HAProxy has to do is hand the connection off to Squid which will handle the 
 HTTP or HTTPS or HTTPS-with-SSL-Bump, independent of anything HAProxy sees or cares about.

```
