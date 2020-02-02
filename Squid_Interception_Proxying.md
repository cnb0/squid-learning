```
Ordinarily, when using Squid on a network to cache Web traffic, 
browsers must be configured to use the Squid system as a proxy.
This type of configuration is known as traditional proxying. In many environments,
this is simply not an acceptable method of implementation. 
Therefore Squid provides a method to operate as an interception proxy, or 
transparently, which means users do not even need to be aware that a proxy is in place. 
Web traffic is redirected from port 80 to the port where Squid resides, and
Squid acts like a standard Web server for the browser.

Using Squid transparently is a two part process, 
requiring first that Squid be configured properly to accept non-proxy requests, and 
second that Web traffic gets redirected to the Squid port. 
The first part of configuration is performed in the Squid module, 
while the second part can be performed in the Linux Firewall module. 
That is, assuming you are using Linux, otherwise you should consult the
Squid FAQ Transparent Caching/Proxying entry.

Configuring Squid for Transparency
In order for Squid to operate as a transparent proxy, 
it must be configured to accept normal Web requests rather than (or in addition to) proxy requests.
Here, you'll learn about this part of the process, examining both the console configuration and
the Webmin configuration. Console configuration is explained, and Webmin configuration is shown in the figure below.

As root, open the squid.conf file in your favorite text editor. 
This file will be located in one of a few different locations depending on your operating system and 
the method of installation. Usually it is found in either /usr/local/squid/etc, 
when installed from source, or /etc/squid, on Red Hat style systems. 
First you'll notice the http_port option. This tells you what port Squid will listen on. 
By default, this is port 3128, but you may change it if you need to for some reason. 
Next you should configure the following options, as shown in Figure 12.19, 
"Transparent Configuration of Squid".

        httpd_accel_host virtual
        httpd_accel_port 80
        httpd_accel_with_proxy  on
        httpd_accel_uses_host_header on
        
These options, as described in the Miscellaneous Options section of this document,
configures Squid as follows. httpd_accel_host virtual causes Squid to act as an accelerator for any number 
of Web servers, meaning that Squid will use the request header information to figure
out what server the user wants to access, and that Squid will behave as a Web server 
when dealing with the client. httpd_accel_port 80 configures Squid to send out requests 
to origin servers on port 80, even though it may be receiving requests on another port, 3128 for example. 
httpd_accel_with_proxy on allows you to continue using Squid as a traditional proxy as well as a transparent proxy. 
This isn't always necessary, but it does make testing a lot easier when you are trying to get transparency working,
which is discussed a bit more later in the troubleshooting section. Finally, 
httpd_accel_uses_host_header on tells Squid that it should figure out what server to fetch content  
from based on the host name found in the header. This option must be configured this way for transparency.

Linux Firewall Configuration For Transparent Proxying
The iptables portion of your transparent configuration is equally simple. The goal is to 
hijack all outgoing network traffic that is on the HTTP port (that's port 80, to be numerical about it).
iptables, in its incredible power and flexibility allows you to do this with a single command line or a 
single rule. Again, the configuration is shown and discussed for both the Webmin interface and the 
console configuration.

Note:	The Linux Firewall module is new in Webmin version 1.00. All previous revisions lack this module, 
so to follow these steps, you'll need to have a recent Webmin revision.
When first entering the Linux Firewall module, the Packet filtering rules will be displayed. 
For your purposes you need to edit the Network address translation rules. So, select it from 
the drop-down list beside the Showing IPtable button, and click the button to display the NAT rules.

Now, add a new rule to the PREROUTING chain by clicking the Add rule button to the right of the 
PREROUTING section of the page.

Fill in the following fields. The Action to take should be Redirect, and the 
Target ports for redirect set to 3128. Next you'll need to specify what clients 
should be redirected to the Squid port. If you know all port 80 traffic on a single 
interface should be redirected, it is simplest to specify an Incoming interface, but you
could instead specify a Source address or network. Next, set the Network protocol to Equals TCP. 
Finally, set the Destination TCP or UDP port to 80. Click Create to add the new rule to the configuration. 
Once on the main page again, click the Apply Configuration button to make the new rule take effect. 
Finally, set the firewall to be activated at boot so that redirection will continue to be in effect on reboots.

 # iptables -t nat -I PREROUTING 1 -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 3128
While a detailed description of the iptables tool is beyond the scope of this section, 
it should briefly be explained what is happening in this configuration. First, 
you are inserting a rule into the first PREROUTING chain of the NAT routing path, 
with the -t nat -I PREROUTING 1 portion of the command. Next you're defining whose 
requests will be acted upon, in this case iptables will work on all packets originating from the network 
attached to device eth0. This is defined by the -i eth0 portion of the rule. Then comes the 
choice of protocol to act upon; here you've chosen TCP with the -p tcp section. Then, the last 
match rule specifies the destination port you would like for your redirect to act upon 
with the --dport 80 section. Finally, iptables is told what to do with packets that match 
the prior defined criteria, specifically, it will REDIRECT the packets --to-port 3128.
