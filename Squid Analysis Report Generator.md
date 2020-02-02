``` 
SARG (Squid Analysis Report Generator).
It is an Open-Source tool, which helps us analyze Squid Proxy logs & generates reports in HTML format with all the information from logs presented in nice & easy to understand format.

& It gives information about User’s IP addresses , total & individually used bandwidth etc with access to Daily, Weekly & Monthly reports.

( Also read : Setting up SQUID AUTHENTICATION )

 

Installation
The process for installing sarg on Centos/Redhat is a bit complicated, as it needs to be compiled from source. To do that, firstly we need to install required packages to download & compile the package

yum install -y gcc gd gd-devel make perl-GD wget httpd

Secondly, download ppackage from the link mentioned in below

wget http://sourceforge.net/projects/sarg/files/sarg/sarg-2.3.10/sarg-2.3.10.tar.gz

now, we will extract the package & will than compile the package

tar -xvzf sarg-2.3.10.tar.gz
cd sarg-2.3.10
./configure
make
make install

Configuration
Now that’s the installation is complete, we will configure it as per our needs by making changes in configuration file

vi /usr/local/etc/sarg.conf

Firstly, uncomment the line starting with access_log & add path for squid access log. Next, provide output directory for reports next to line starting with output_dir & also select your desired time format, change  line with date_format

# TAG: access_log file
# Where is the access.log file
#
#
access_log /var/log/squid/access.log
Add output directory
# TAG: output_dir
# The reports will be saved in that directory
#
#
output_dir /var/www/html/squid
Set the correct date format
# TAG: date_format
# Date format in reports: e (European=dd/mm/yy), u (American=mm/dd/yy), w (Weekly=yy.ww)
#
date_format e

& lastly , set overwrite report to yes

# # TAG: overwrite_report yes|no
# yes – if report date already exist then will be overwritten.
# no – if report date already exist then will be renamed to filename.n, filename.n+1
#
overwrite_report yes

Generating report
To create squid analysis report, we have to enter following command

sarg -x

Note: It may take a while depending on number of users accessing squid proxy.

Accessing report

To access the report, enter below mentioned URL in web-browser

http://IP-Address of server/squid

Now, we have all the squid analyzed logs in nice, sorted &easy to understand format

Note : you can also create a cron–job to schedule a report being generated automatically at the time of your choosing.

For Example

 * */4 * * * /usr/local/bin/sarg -x

This will generate a report every 4th hour.
