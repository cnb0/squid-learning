https://wiki.squid-cache.org/Features/eCAP


eCAP is a software interface that allows a network application, such as an
HTTP proxy or an ICAP server, to outsource content analysis and adaptation to
a loadable module. For each applicable protocol message being processed, an
eCAP-enabled host application supplies the message details to the adaptation
module and gets back an adapted message, a "not interested" response, or a
"block this message now!" instruction. These exchanges often include message
bodies.

Install libecap and build Squid with the --enable-ecap configure option. Install an adapter module.

http://www.e-cap.org/downloads/
http://www.e-cap.org/market/


https://github.com/maxpmaxp/ecap-exif
http://www.measurement-factory.com/


1. ecap modules :
   1. eCAP adapter for HTTP compression with GZIP.
          https://github.com/c-rack/squid-ecap-gzip
   2.an eCAP adapter for HTTP compression with GZIP and DEFLATE. This is fully re-worked in C++11, improved 
          https://github.com/yvoinov/squid-ecap-gzip
 eCap adapter gives the possibility to load a gzip module that would perform compression, hence


2. Securepoint eCAP antivirus adapter : https://github.com/Securepoint/squid-ecap-av

3. ziproxy the HTTP traffic compressor :http://ziproxy.sourceforge.net/download.html
setup a web proxy that will compress the data and will also perfrorm some sort of content filtering,
a) ziproxy --> Does the compression but no content filtering

b) squid with squidguard --> Does the content filtering but not compression.

eCap adapter is used for just http compression.
Ziproxy compresses far more elements , such as jpg, flash e.t.c compare to the particular squid module.
  Chain squid with ziproxy

Squid(with guard) --> ziproxy -->

In that way, clients requests from squid the websitea, squid checks with the aid of squid-guard if it should process the reuquest, and if yest it will ask from ziproxy to fetch the webpage.
Ziproxy in turn will fetch the webpage, compress it and turn it to squid where then will be delivered to the client.
Hence, we have content filtering and compression


https://stackoverflow.com/questions/21592427/squid-content-manipulation

