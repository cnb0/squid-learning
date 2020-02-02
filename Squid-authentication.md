```
Squid is a proxy web server that uses caching to optimizes website operation
so that the web pages load more quickly, thereby improving the response time for 
the pages that are accessed by users most frequently. Using Squid as a proxy server,
it decreases the load on the server, increases the capacity and decreases the costs.
Squid is not only a caching proxy but we can also use Squid for managing access 
to internet websites it provides us controls to limit user 
access to specific pages with the help of its ACLs.

To setup squid  authentication . 


SQUID AUTHENTICATION CAN NOT BE USED WHEN CONFIGURED AS TRANSPARENT PROXY. 

Now that we got it out of the way, let’s start our configurations 
for setting up squid with authentication.

 

Pre-requisites for Squid Authentication

We need to have both squid & http packages installed on our system.

$ sudo yum install httpd

$ sudo yum install squid

To detailed squid installation, refer to the articles mentioned above.



Configuration
We are going to use a module called ‘ncsa_auth’ for squid authentication. 
It’s located at ‘/usr/lib/squid/ncsa_auth’ for 32 bit systems & 
for 64 bit system, it’s located at ‘/usr/lib64/squid’ directory.

So firstly, we need to makes changes in squid’s main configuration file i.e. 
‘/etc/squid/squid.conf’,  

$ sudo vim /etc/squid/squid.conf

firstly we will assign the squid authentication method, so add the following line in squid.conf

auth_param basic program /usr/lib64/squid/ncsa_auth /etc/squid/users_passwd

auth_param basic realm proxy

here, /etc/squid/users_passwd is the files with user information. 
Next, we will set up an acl named ‘auth_users’ for the authentication,

acl auth_users proxy_auth REQUIRED

Now we will apply the created acl

http_access allow auth_users

http_access deny all

Now save the file & exit but make sure that all these acl should be entered above all 
other acls otherwise they might not work.
Now we only need to create users for authentication. 
To create the password table, we need to execute the following command,

$ sudo htpasswd -c /etc/squid/users_passwd squid

here ‘-c’ option is used to create the file & will not be used for adding other users,
‘squid’ is the name user. Now we need to restart the squid server to implement all the changes.

$ sudo service squid restart

We can also use PAM, RADIUS or DIGEST for setting up squid authentication, 
but for this tutorial it’s NCSA_AUTH, maybe in some future tutorials, we will discuss those.
```
