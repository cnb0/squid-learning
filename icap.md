The Internet Content Adaptation Protocol (ICAP) enables 
enterprises to choose the best of breed applications and infrastructure. 
ICAP enables a new class of services by allowing site owners to offer Web applications closer to the user. 
ICAP is a lightweight and extensible point-to-point protocol that effectively adapts content for user needs. 
Since its introduction in 1999 ICAP became the standard protocol for communication between proxy servers and 
callout servers.

http://www.icap-forum.org/icap?do=resources&subdo=faq

http://www.icap-forum.org/icap?do=products


Language translation
Virus checking
Family ( PG/R/X ) content filtering
Local real-time ad insertion
Wireless protocol translation
Anonymous Web usage profiling (perhaps for a dating service)
Transcoding or image enhancement
Image magnification for the elderly
Digest production/batch download of Web content

ICAP provides a very simple operation to vector content between caches and network-based applications servers. 
Because no proprietary APIs or software developer toolkits are required, ICAP will dramatically reduce implementation cost
and time-to-market for software developers and service providers. 
By being standards-based, ICAP has the potential to be adopted by range of vendors, 
as indicated by the number of companies presently supporting its development. ICAP also leverages 
the latest Internet infrastructure developments including the cache and Internet content delivery convergence.

The principal advantages of ICAP include:

It is scalable, allowing several ICAP servers to service requests & responses from a single cache.
It is open: anyone can implement, which extends a caching server to provide new services at the edge of the network.
It is more efficient than adding one Web proxy per service.

Today, applications are centralized within the confines of a single enterprise or Web site.
ICAP "externalizes" applications and enables them to be network-based.
The end result is that ISPs, Web site owners, and Web visitors can dynamically utilize
applications on an as-needed basis.

ICAP offers special features such as "Preview" and "204 responses", especially designed to reduce the traffic 
that has to be exchanged between an ICAP client and an ICAP server. 
Not only they reduce the bandwidth that an ICAP service needs to handle but also help
to keep the latency introduced by filtering to a minimum.

ICAP/1.0 has been designed to encapsulate HTTP messages. By translating other protocols first to HTTP and 
then encapsulating into ICAP, some vendors have found a way to use ICAP also for other protocols such as FTP;
in this case implementation remains interoperable with other ICAP services. ICAP has also been used 
to encapsulate messages of other protocols natively (without translating to HTTP first); 
those implementations are usually no standard implementations and are not interoperable with other solutions.
