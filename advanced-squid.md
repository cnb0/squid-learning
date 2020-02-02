``` 
Squid has one primary configuration file, squid.conf. This file is generally located in /etc/squid/, 
or if you compiled Squid from source, the default location is /usr/local/squid/etc/. 
You’ll be editing this file, so it’s wise to make a backup copy of it before you make any changes.

Access control lists

The primary use of ACLs is to control access,
but they can also be used to route requests through a hierarchy, 
control request rewriting, and manage quality of service.

Access controls are divided into two parts: elements and rules.
ACL elements are things such as IP addresses, port numbers, hostnames, and 
URL patterns. Each ACL element has a name, which you refer to when writing the access list rules. 
The basic syntax of an ACL element is:

ACLname type value1 value2

Squid has more than 20 ACL types, including types for source and destination IP addresses, 
time, URLs, port numbers, and transfer protocols. See the Squid Configuration Manual for a full list of types.

After defining the ACL elements, the next step is to combine them with Access list rules. 
Rules combine elements to allow or deny certain actions. The syntax for an access control rule is:

access_list allow|deny [!]ACLname

For example, the rule:

http_access allow MyClients

tells Squid to allow access from the host or hosts defined under the name MyClients. 
The optional exclamation point is a standard negation operator, 
used to reverse the logic of the ACL. If this seems confusing, the following examples should help.

Restricting access to local network users

You should always limit access to your proxy server to local IP addresses, 
unless you have a specific need to allow external users. This can save you large bandwidth bills,
from outsiders using your machine as a proxy. 
A simple way to do this is to write an ACL that contains your IP address space and 
then allow HTTP requests for that ACL and deny all others:


acl All src 0/0

acl PrivateNet src 192.168.0.0/24 192.168.1.0/24

http_access allow PrivateNet

http_access deny All

Squid makes one pass through the configuration file, reading the ACLs and rules in order. 
This means that you must define an ACL before you make a rule applying it, and the order
of the http_access rules is important. Incoming requests are checked in the order 
in which the rules are written. If the first rule allows the request, the remaining 
requests are not read. If the first rule blocks the request, Squid passes on to 
the next one, and so on. Your last http_access line should always be a deny All, 
so that a request which is not permitted by any of the previous rules is blocked by default. 
If you change this to allow All, all your rules become meaningless, since Squid will allow
the request at the end. The default squid.conf configuration file contains 
some important access controls. Try not to change these before you understand what they do.
When you edit squid.conf for the first time, look for this comment:


#

# INSERT YOUR OWN RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS

#

Insert your new rules below this comment, and before the http_access deny All line.

Blocking specific computers

It is often necessary to block a particular IP address. At our university, 
for example, if a student uses excessive bandwidth, we block his computer 
for a few days. Until you can solve the problem at the source, 
you can block requests coming to Squid with this configuration:


acl All src 0/0

acl PrivateNet src 192.168.0.0/24 192.168.1.0/24

acl ProblemHost src 192.168.0.15

http_access deny ProblemHost

http_access allow PrivateNet

http_access deny All

This will block requests from the IP address 192.168.0.15. 
You can also block an IP range, such as 192.168.0.0/24.

Restricting usage to specified Web sites during working hours

You can set up a simple ACL to restrict Internet usage to work-related 
sites during working hours. To do this, you need to make a list of 
allowed sites and save it as a file with the domain names on each line. For example:


#Allowed Sites

www.cnn.com
www.news.google.com
www.bbc.co.uk
www.newsforge.com

and other allowed sites...
Once you have your allow list ready, use the following ACL to restrict usage:


acl All src 0/0
acl PrivateNet src 192.168.0.0/24 192.168.1.0/24
acl AllowedSites dstdomain "/usr/local/squid/etc/allowed-sites"
acl WorkingHours time D 08:00-17:30
http_access allow WorkingHours AllowedSites
http_access deny All

Blocking porn

Pornographic sites are quite a headache for many organizations. 
While many specialized free and commercial packages exist for filtering content, 
you can use Squid to block pornography as well.
The hardest part about using Squid to deny access to pornography
is coming up with the list of sites that should be blocked. 
If you want a ready-made list, the Access Controls section 
of the Squid FAQ has links to freely available lists.

The ACL you have to write for such a list depends on the content of the list. 
If the list contains regular expressions, you’ll need to use the following ACL:


acl PornSites url_regex "/usr/local/squid/etc/pornlist"

http_access deny PornSites

If the list contains hostnames, the url_regex will have to be changed to dstdomain, 
which tells Squid to match the entire hostname instead of the words in the hostname:


acl PornSites dstdomain "/usr/local/squid/etc/pornlist"

http_access deny PornSites

These methods are fine for casual use. If you are really 
serious about blocking such sites, you might want to look at specialized software,
such as SquidGuard or Dansguardian.

Proxy authentication

Proxy authentication is a complex subject, due to the various types 
of proxy authentication schemes available. 
I describe a simple user authentication scheme below, 
but there are many more schemes available, and the best one will vary according to your specific needs.

Squid currently supports three techniques for receiving user credentials:
HTTP Basic and Digest and NTLM. Basic authentication has been around for a long time. 
Though this is what I use in this example, you should know that it is a very insecure protocol,
since the usernames and passwords are sent over the network in clear text. 
Anyone who runs a packet analyzer on your network can get the passwords. 
Still, it’s a good place to start, and for smaller networks, where security is not a major problem, it works well.

To use proxy authentication, Squid needs to be configured 
to spawn a number of external helper processes.
The Squid source code includes some programs that 
authenticate against a number of standard databases. 
The auth_param directive controls the configuration of all helper programs.

The order of the auth_param directive and proxy_auth 
ACL is extremely important. Remember that Squid reads the 
config file in one pass, and in order. If you don’t put 
the proxy authentication ACLs in the proper order, 
you could end up allowing (or denying) all access. To use proxy authentication, 
you must define at least one authentication helper before any proxy_auth ACLs. 
If you don’t, Squid will print an error message to the logs and start up anyway, 
and all user requests may be denied. If you try to set up proxy authentication and 
find that it’s not working, look at the logs to make sure that the problem does not lie in the order of the ACLs.

HTTP Basic authentication supports the following auth_param parameters:


l auth_param basic program command

l auth_param basic children number

l auth_param basic realm string

l auth_param basic credentialsttl time-specification

The program parameter specifies the command, including arguments,
for the helper program. This is generally the pathname to one of the 
authentication helper programs. By default, the path is /usr/local/squid/libexec.

The children parameter tells Squid how many helper processes to use. 
The default value is 5, which is a good starting point if you don’t 
know how many helpers Squid needs to handle the load. For a 400-user network, 
I use a value of 25. You should check your cache.log to make sure that there are
no warning messages about too few helper processes, and increase the number of helper 
processes if there are warnings.

The realm parameter is the authentication realm string that the 
proxy server should present to the user when prompting for a username and password. 
Use something simple, such as “Orgname Proxy Server.”

The credentialsttl parameter specifies the amount of time that 
Squid internally caches authentication results.
A larger “time to live” value reduces the load on the external authenticator processes, 
but increases the amount of time until Squid detects changes to the authentication database. 
If you have a relatively fixed user base, set this high, but if the user base is transient, 
as in a public library, use a lower value. The default TTL value is two hours.

A complete setup would look like this:


auth_param basic program /usr/local/squid/libexec/ncsa_auth /usr/local/squid/etc/passwd

auth_param basic children 10

auth_param basic realm NLU Proxy Server

auth_param basic credentialsttl 3 hour

acl Students proxy_auth REQUIRED

http_access allow Students

For this example I have used the NCSA authentication helper, which is a simple authentication method that stores usernames and passwords in a single text file, similar to the /etc/passwd file. You pass the path to the password file as the program’s single command-line argument in Squid.conf:

auth_param basic program /usr/local/squid/libexec/ncsa_auth /usr/local/squid/etc/passwd

To create and update the file, you can use the htpasswd program. If you have the Apache Web server installed, htpasswd should also be installed; if not, download it from the Squid Web site. To create a file, the command is htpasswd -c passwdfile user.

To add users and change their passwords, the command is htpasswd passwdfile username.

htpasswd will prompt you for a password. If you want to allow users to change their own passwords, you can use the chpasswd CGI script, which is also available on the Squid Web site.

There are several other authentication helpers you can use with Basic authentication. For example, you can authenticate against a LDAP server, Windows Domain, or Samba domain.
