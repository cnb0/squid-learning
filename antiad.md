``` 
Squid "Anti-Ad" Server Blocker
The proxy server Squid (Squid Web Proxy Cache) has the ability to 
read a list of ips from a text file and 
block those ips from clients using the proxy. 
This is perfect for blocking ad servers for your internal clients. 
Your clients in turn will not have to be bothered with ads, 
they will save bandwidth and you wont have to worry as much about
that user that will click on any shiny animated object in front of them. 

This script works similarly to SafeSquid, but gives you full 
control of the list and allows for increased flexability.
https://www.safesquid.com/content-filtering/overview

Getting Started

The following three(3) lines need to be added anywhere in your squid.conf file. 
We are going to assume your squid.conf file is in /etc/squid/ and you will be 
putting your list of ad servers called ad_block.txt in the same directory.

NOTE: If you need assistance with Squid check out the Calomel.org Squid proxy "how to". 
You can also setup a proxy auto configuration (PAC) file in the browser using 
our Proxy Auto Config for Firefox (PAC) "how to".
The first line below is a comment and reminder where you are getting your list from. 
The second line is the regular expression that reads the "/etc/squid/ad_block.txt" file 
when the squid daemon loads or when you reconfigure the daemon with "squid -k reconfigure". 
The next line instructs squid to deny access to those ips in the list from clients using the squid proxy. 
The last line (deny_info) is optional, 
it just sends back a tcp rest to the client instead of sending an infomational error page. 
You may want this option if you do not want clients provided with 
any ifo about your proxy or why the error was triggered.

## disable ads ( http://pgl.yoyo.org/adservers/ )
acl ads dstdom_regex "/etc/squid/ad_block.txt"
http_access deny ads
#deny_info TCP_RESET ads
Fetching the list of ad servers
The next step is to fetch the list of known advertising hostnames and
save them to a file so squid can read it. 
The following script uses curl to download the list from pgl.yoyo.org and 
save the list to a file in /etc/squid/ad_block.txt. 
The last line in the script tells squid to re-read the ad_block.txt list after the 
file is downloaded to load in any new ad servers.

#### Calomel.org  ad_servers_newlist.sh 
#
## get new ad server list
curl -sS -L --compressed
"http://pgl.yoyo.org/adservers/serverlist.php?hostformat=nohtml&showintro=0&mimetype=plaintext" > 
/etc/squid/ad_block.txt 

## refresh squid
/usr/local/sbin/squid -k reconfigure

Automating with cron

Lastly, you may want to setup and cron job to get the latest list every few days. 
The site you get the ad list from (pgl.yoyo.org) updates their ips
every 3 days or so on average. 
With a cron job running you can make sure you have the latest list.
Below is a cron job line to get the ad servers list every 3 days at 5:35am (0535).

#minute (0-59)
#|   hour (0-23)
#|   |    day of the month (1-31)
#|   |    |   month of the year (1-12 or Jan-Dec)
#|   |    |   |   day of the week (0-6 with 0=Sun or Sun-Sat)
#|   |    |   |   |   commands
#|   |    |   |   |   |
#### refresh squid's anti-ad server list
35   5    *   *   */3 /scripts_dir/ad_servers_newlist.sh >> /dev/null 2>&1

Questions?
Where an ad was supposed to be, it now says "error". Whats wrong? Nothing. 
What you are seeing is the squid error page. When the browser asks 
for an ip listed in the ad_block.txt file squid servers the error page saying this 
ip can not be reached because it had been blocked. If you had an ad box large enough 
you would be able to see the entire squid error page in the ad space.
