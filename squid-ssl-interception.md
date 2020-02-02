```
SQUID SSL interception
 
http://www.faqs.org/docs/Linux-mini/TransparentProxy.html#ss2.3

However, with every single web site moving towards SSL/HTTPS nowadays, and 
google penalized those who don't, if squid is only caching the tradition HTTP traffics, 
then there will be less and less to cache, and the caching will become more and
more useless, with the majority of the sites being severed under HTTPS now.


The SQUID proxy will basically act as a man in the middle. 

The motivation behind setting this up is to decrypt HTTPS connections to apply content filtering and so on.

There are some concerns that transparently intercepting HTTPS traffic is unethical and 
can cause legality issues. 

So, on to the technical details of setting the proxy up. When compiling 
SQUID from scratch, the --enable-ssl switch is a prerequisite for SslBump,
which squid uses to intercept SSL traffic transparently.

Once SQUID has been installed, a very important step is to create the certificate that 
SQUID will present to the end client. In a test environment, 
you can easily create a self-signed certificate using OpenSSL by using the following:

$ openssl req -new -newkey rsa:1024 -days 365 -nodes -x509 \
-keyout http://www.sample.com.pem  -out http://www.sample.com.pem

Then edit the /etc/squid.conf file to change the “cert=” and the “key=” to
point to the correct file in your environment.

To save each user from configuring their proxy each by themselves, configure 
iptables to perform destination NAT, basically to redirect the traffic to the proxy:

iptables -t nat -A PREROUTING -i eth0 -p tcp –dport 80 -j DNAT –to-destination 192.9.200.32:3128
iptables -t nat -A PREROUTING -i eth0 -p tcp –dport 443 -j DNAT –to-destination 192.9.200.32:3129

This will of course cause the client browser to choke and display a security error.
I.e., the client will surely know about it unless their computer was preconfigured
to accept self-signed certs (which are not signed by 'well-known' root CAs). 
They need to import the above Certificate CA for Squid into their Browser to continue.
