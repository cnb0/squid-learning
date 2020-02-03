### Squid has built-in ICAP (Internet Content Adaptation Protocol)  support.

https://wiki.squid-cache.org/SquidFaq/ContentAdaptation#eCAP

Content Adaptation
A proxy may analyze, capture, block, replace, or modify the messages it proxies. Such actions are often called content adaptation even though some of them do not alter anything.

Squid can be configured or modified to perform some forms of content adaptation. This page highlights content adaptation approaches supported by Squid.
Use cases
The following are typical content adaptation needs. Virtually all of the adaptations listed below have been implemented using one or more mechanisms described in this document.

Add, remove, or modify an HTTP header field (e.g., Cookie)
Block messages based on request URLs
Block messages based on content
Redirect certain requests to a custom page or server
Respond to certain requests with a custom page
Modify a page to insert new content (e.g., warnings or ads)
Modify a page to remove existing content (e.g., images or ads)
Scale an embedded image (e.g., for mobile devices)


ICAP is a poorly documented and inconsistently implemented protocol.

ICAP Server: no

ICAP Client: yes

   ICAP Features:
   - ICAP for HTTP
   - ICAP for HTTPS
   - ICAP for FTP
   - Support for any Preview size
   - Allow 204 response at end

   Proxy Features:
   - User Authentication
   - Web Cache
   - WAN acceleration
   - Non-ICAP Content Adaptation
   
    

ICAP specifies how an HTTP proxy (an ICAP client) can outsource content adaptation to an external ICAP server.
No proxy code modifications are necessary for most content adaptations using ICAP.
If your adaptation algorithm resides in an ICAP server, 

Pro:
- Proxy-independent,
- adaptation-focused API, 
- no Squid modifications, 
- supports remote adaptation servers, 
- scalable. 

Cons: 
- Communication delays, 
- protocol functionality limitations, 
- needs a stand-alone ICAP server process or box.
- it will be able to work in a variety of environments and 
- will not depend on a single proxy project or vendor


One proxy may access many ICAP servers, and
one ICAP server may be accessed by many proxies. 
An ICAP server may reside on the same physical machine as Squid or run on a remote host. 
Depending on configuration and context, some ICAP failures can be bypassed, making them invisible to proxy end-users.


Scripting :
``` 
C API : http://c-icap.sourceforge.net/
C++ API (Traffic Spicer - paid ): http://spicer.measurement-factory.com/
      With direct access to ICAP experts, Factory clients benefit from years of Spicer compatibility work as well as Spicer APIs that isolate client-specific adaptation logic from ICAP idiosyncrasies.

PYTHON API :http://icap-server.sourceforge.net/
Java : http://www.poesia-filter.org/
WebFlowAdapter ICAP server - http://www.l3ws.com/
                             HTTP anonymization,
                             Advertisement Insertion,
                             Tracking,
                             Access Control,
                             Web Application firewall,

```



Squid Conf :
--------------
``` 
icap_enable on

icap_service service_req reqmod_precache 1 icap://127.0.0.1:1344/request
icap_class class_req service_req
icap_access class_req allow all

icap_service service_resp respmod_precache 0 icap://127.0.0.1:1344/response
icap_class class_resp service_resp
icap_access class_resp allow all
```





### C API 
https://wiki.squid-cache.org/ConfigExamples/ContentAdaptation/C-ICAP

- Ubuntu 19:04 :
                 - https://zoomadmin.com/HowToInstall/UbuntuPackage/c-icap
                 - http://manpages.ubuntu.com/manpages/disco/man8/c-icap.8.html
- CVE Details  : https://www.cvedetails.com/vendor/15032/C-icap-Project.html

github(server and modules )
- https://github.com/c-icap/c-icap-server
- https://github.com/c-icap/c-icap-modules

Implements basic services for the c-icap server. Currently the following
services are implemented:
  - virus_scan, an antivirus ICAP service
  - url_check, an URL blacklist/whitelist icap service
  - srv_content_filtering, a score based content filtering icap service

c-icap is an implementation of an ICAP server. It can be used with HTTP proxies that support the ICAP protocol to implement content adaptation and filtering services.

Most of the commercial HTTP proxies must support the ICAP protocol. The open source Squid 3.x proxy server supports it.

Instructions : https://sourceforge.net/p/c-icap/wiki/faqcicap/

Major features:

- basic C API for developing custom content adaptation and filtering services
- plugins interface
- LDAP integration
- simple ICAP client API

Currently the following services have been implemented for the c-icap server:
- Web antivirus service, using the clamav open-source antivirus engine
- basic URL filtering service

#### c-icap Install : https://sourceforge.net/p/c-icap/wiki/c-icapInstall/
config : https://sourceforge.net/p/c-icap/wiki/configuration/

```
acl infoaccess dstdomain icap.info

icap_service service_info reqmod_precache 1 icap://192.168.1.2:1344/info
adaptation_service_set class_info service_info
adaptation_access class_info allow infoaccess
adaptation_access class_info deny all
```
To view c-icap statistics use the http://icap.info url in your browser

Examples :
https://serverfault.com/questions/482552/how-to-enable-url-filtering-with-just-squid-c-icap
