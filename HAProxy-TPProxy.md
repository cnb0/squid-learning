```
If you use HaProxy as the load balancer then all of the backend servers 
see the traffic coming from the IP address of the load balancer. 
TPROXY allows you to make sure the backend servers see the true client IP address in the logs. NB.
Standard Kernel builds didn't support TPROXY but as of 2.6.28 they all now do support it.

Ps. A MUCH easier alternative is inserting the clients ip in the x-forwarded-for header, 
follow the instructions here to achieve this:
How to implement x-forwarded-for with Microsoft IIS <-> How to implement x-forwarded-for with Apache

BUT if you want REAL COMPLETE TRANSPARENCY then you need to implement TPROXY:

For TPROXY to work you need three things:

TPROXY compiled into the linux kernel
TPROXY/Socket compiled into netfilter/iptables (due in v1.4.3?)
HaProxy compiled with the USE_LINUX_TPROXY option
The TPROXY patch for Linux Kernel 2.6.25.11 is here:
https://my.balabit.com/login

The following is a guide how to install on Centos 5.1:

Heavily borrowed from: http://howtoforge.com/kernel_compilation_centos

Download The Kernel Sources
First we download our desired kernel to /usr/src. Go to www.kernel.org and select the kernel you want 
to install, e.g. linux-2.6.25.11.tar.bz2 (you can find all 2.6 kernels here: 
http://www.kernel.org/pub/linux/kernel/v2.6/).
Then you can download it to /usr/src like this:

cd /usr/src
wget http://www.kernel.org/pub/linux/kernel/v2.6/linux-2.6.25.11.tar.bz2
Then we unpack the kernel sources and create a symlink linux to the kernel sources directory:

tar xjf linux-2.6.25.11.tar.bz2
ln -s linux-2.6.25.11 linux
cd /usr/src/
wget http://www.balabit.com/downloads/files/tproxy/tproxy-kernel-2.6.25-20080519-165031-1211208631.tar.bz2
tar -xjf tproxy-kernel-2.6.25-20080519-165031-1211208631.tar.bz2
cd linux
cat ../tproxy-kernel-2.6.25-20080519-165031-1211208631/00*.patch | patch -p1 --dry-run
cat ../tproxy-kernel-2.6.25-20080519-165031-1211208631/00*.patch | patch -p1
Configure The Kernel
It's a good idea to use the configuration of your current working kernel as a basis for your new kernel.
Therefore we copy the existing configuration to /usr/src/linux:

make clean &amp;&amp; make mrproper
cp /boot/config-`uname -r` ./.config
I needed to do a:

yum install ncurses-devel gcc gcc-c++ make rpm-build
Then we run

make menuconfig
which brings up the kernel configuration menu. Go to Load an Alternate Configuration File and 
choose .config(which contains the configuration of your current working kernel) as the configuration file

Then browse through the kernel configuration menu and make your choices.

Make sure you enable tproxy support, 'socket' and 'TPROXY' modules (with optional conntrack support if you need SNAT)

Make sure you specify a kernel version identification string under General Setup --> () 
Local version - append to kernel release. I use CS3 so our kernel rpm package will be named 
kernel-2.6.25.11CS3.x86_64.rpm. You can leave the string empty or specify a different one which 
helps you identify the kernel (e.g. -custom or whatever you like).

Please note: After you have installed kernel-2.6.25.11CS3.x86_64.rpm and decide to compile another 
2.6.25 kernel rpm package, it is important to use a different version string, e.g. -default1, -default2, etc., 
because otherwise you can't install your new kernel because rpm complains that kernel-2.6.25.11CS3.x86_64.rpm
is already installed!

Once you are happy with the kernel configuration, save & exit menuconfig then simply:

make rpm
This may take quite a long time....
Once it has finished:

Source RPM is here:
ls -l /usr/src/redhat/SRPMS/
Binary RPM is here:
ls -l /usr/src/redhat/RPMS/x86_64/
now install the new kernel:

cd /usr/src/redhat/RPMS/x86_64/
rpm -ivh --nodeps kernel-2.6.25CS-1.x86_64.rpm
Now you can either run the following command:

/sbin/new-kernel-pkg --package kernel --mkinitrd --depmod --install 2.6.25CS
Or you can do the usual manual steps i.e.

Make sure you create a new initrd file:

mkinitrd /boot/initrd-2.6.25.11CS3.img 2.6.25.11CS3
Now configure the boot loader:

vi /boot/grub/menu.lst
default=0
timeout=5
splashimage=(hd0,0)/boot/grub/splash.xpm.gz
hiddenmenu
title CentOS (2.6.25.11CS3)
root (hd0,0)
kernel /boot/vmlinuz-2.6.25.11CS3 ro root=LABEL=/
initrd /boot/initrd-2.6.25.11CS3.img
That's it, so reboot and do a:

uname -a
To check that we are using the new kernel:

Linux lbmaster 2.6.25.11CS3 #6 SMP Mon Jul 28 13:10:43 GMT 2008 x86_64 x86_64 x86_64 GNU/Linux
Compiling iptables with TPROXY support:
First download the current iptables source code:

cd /usr/src/
wget http://www.netfilter.org/projects/iptables/files/iptables-1.4.0.tar.bz2
tar -xjf iptables-1.4.0.tar.bz2
Then download the tproxy patch:

wget http://www.balabit.com/downloads/files/tproxy/tproxy-iptables-1.4.0-20080521-113954-1211362794.patch
cd /usr/src/iptables-1.4.0/
cat ../tproxy-iptables*.patch | patch -p1
make
make install
Compiling HAProxy with TPROXY support:
Download the latest version of the HAProxy source code:

wget http://haproxy.1wt.eu/download/1.3/src/haproxy-1.3.15.7.tar.gz
tar -xvf haproxy-1.3.15.7.tar.gz
cd haproxy-1.3.15.7/
Then compile making sure to enable TPROXY

make TARGET=linux26 CPU=x86_64 USE_STATIC_PCRE=1 USE_LINUX_TPROXY=1
make install target=linux26
If you have got this far then great, that's the hard part done!

Now before Haproxy can utilize TPROXY we need to set up some firewall marks:
You can put this script in a start up file such as rc.local etc.

#!/bin/bash
iptables -t mangle -N DIVERT
iptables -t mangle -A PREROUTING -p tcp -m socket -j DIVERT
iptables -t mangle -A DIVERT -j MARK --set-mark 111
iptables -t mangle -A DIVERT -j ACCEPT
ip rule add fwmark 111 lookup 100
ip route add local 0.0.0.0/0 dev lo table 100
We also need to ensure that we have the correct architecture for the TPROXY trick to work. 
Using the normal HAProxy you can have real servers anywhere on the internet because the 
source address always points back at the HAProxy units IP address. However if the 
clients source IP address is going to be used then the HAProxy server MUST BE IN THE PATH of the return traffic.

The easiest way to do this is to put the backend servers in a different subnet to the front end clients
and make sure that the default gateway points back at the HAProxy load balancer.

NB. With clever routing this should be possible on the same subnet but I haven't tried that yet!

So here is an example configuration that I used for HAProxy:
# HAProxy configuration file
global
# uid 99
# gid 99
daemon
stats socket /var/run/haproxy.stat mode 600
log 127.0.0.1 local4
maxconn 40000
ulimit-n 80013
pidfile /var/run/haproxy.pid
defaults
log global
mode http
contimeout 4000
clitimeout 42000
srvtimeout 43000
balance roundrobin
listen VIP_Name 192.168.2.87:80
mode http
option forwardfor
source 0.0.0.0 usesrc clientip
cookie SERVERID insert nocache indirect
server server1 10.0.0.60:80 weight 1 cookie server1 check
server server2 10.0.0.61:80 weight 1 cookie server2 check
server backup 127.0.0.1:80 backup
option redispatch
The most important line is this one:

source 0.0.0.0 usesrc clientip
If your test setup doesn't work then remove this line to check if a standard configuration does work.

Check your backend server logs to ensure that the client source IP address is correctly showing.

NB. One gotcha (of the many) is that you can no long use any local (i.e. 127.0.0.1) 
backup servers due to routing issues.
To resolve this change the backup server definition as follows:

server    backup 127.0.0.1:80 backup source 0.0.0.0
Ps. Many thanks to John Lauro for his help with the firewall marks stuff

Oh and forgot to say change your sysctrls to allow redirects...i.e.

echo 1 > /proc/sys/net/ipv4/conf/all/forwarding
echo 1 > /proc/sys/net/ipv4/conf/all/send_redirects
echo 1 > /proc/sys/net/ipv4/conf/eth0/send_redirects
```
