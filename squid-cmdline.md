``` Squid Command-Line Options
Before getting too far into other things, let's look at Squid's command-line options. 
Many of these you will never use and some are useful only when debugging problems:


-a port
Specifies a new http_port value. This option always overrides the value from squid.conf.
Note, however, that you can specify multiple values in squid.conf. The -a option overrides only the 
first value from the config file. (This option uses the letter "a" because in the Harvest cache, 
the HTTP port was called the ASCII port.)


-d level
Makes Squid write its debugging messages to stderr (as well as cache.log and syslog, if configured). 
The level argument specifies the maximum level for messages that should be shown on stderr. 
In most cases -d1 works well. See Section 16.2 for a description of debugging levels.


-f file
Specifies an alternate configuration file.


-h
Displays the usage information.


-k function
Signals Squid to perform various administrative functions. 
The function argument may be one of the following: 
reconfigure, rotate, shutdown, interrupt, kill, debug, check, or parse. 
reconfigure causes the running Squid process to reread its configuration file. 
rotate causes Squid to rotate its log files, which involves closing them, possibly renaming them, and 
opening them again. shutdown sends the signal to shut down the Squid process. 
interrupt also shuts down Squid but does so immediately, without waiting for active transactions to finish. 
kill sends the unstoppable KILL signal to Squid, which should only be used as a last resort. 
debug puts Squid into full debugging mode. It can quickly fill up your disk space if your cache is busy. 
check simply checks for a running Squid process. The process return value indicates 
whether Squid is running or not. 
Finally, parse simply parses the squid.conf file. 
The process return value is non-zero if the configuration file contains errors.


-s
Enables logging to the syslog daemon. Squid uses the LOCAL4 syslog facility. 
Level 0 debug messages are logged with priority LOG_WARNING, and level 1 messages are logged with LOG_NOTICE. 
Higher level debugging messages aren't sent to syslogd. You might use an entry like this in /etc/syslogd.conf:

local4.warning                /var/log/squid.log

-u port
Specifies an alternate ICP port number, overriding icp_port in squid.conf.


-v
Prints the version string.


-z
Initializes cache, or swap, directories. You must use this option when running Squid for the first time or
whenever you add a new cache directory.


-C
Prevents the installation of signal handlers that trap certain fatal signals such as SIGBUS and SIGSEGV. 
Normally, the signals are trapped by Squid so that it can attempt a clean shutdown. 
However, trapping the signal may make it harder to debug the problem afterwards. 
With this option, the fatal signals cause their default actions, which is usually to dump core.


-D
Disables initial DNS tests. Normally, Squid won't start until it verifies that its DNS server is working. 
This option prevents that check. You can also alter or remove the dns_testnames option in squid.conf.


-F
Makes Squid refuse all requests until it rebuilds the storage metadata. 
If your cache is busy, this option may shorten the time required to rebuild the metadata.
If your cache is large, however, the rebuild procedure may take a long time anyway.


-N
Prevents Squid from becoming a background daemon process.


-R
Prevents Squid from using the SO_REUSEADDR option before binding to the HTTP port.


-V
Enables virtual host surrogate mode. Similar to entering httpd_accel_host virtual in squid.conf.


-X
Forces full debugging, as though you had specified debug_options ALL,9 in squid.conf.


-Y
Returns ICP_MISS_NOFETCH instead of ICP_MISS when rebuilding store metadata. For busy parent caches,
this option may result in less load while the cache is rebuilding. See Section 10.6.1.2.
