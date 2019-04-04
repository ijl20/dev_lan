# Create a NAT network on second Linux ethernet port

Configuration instructions for an Ubuntu workstation with two ethernet ports, one 
connected to the local network / WAN as usual and the other providing WAN
connectivity to a NAT / DHCP ethernet segment suitable for development use.

## Terminology

*workstation*: the Linux box providing the connectivity to the development subnet
and the WAN NAT routing / DHCP support.

*development subnet*: the ethernet segment connected to the *secondary* ethernet interface
using local IP addresses, i.e. 192.168.1.X (or whatever you choose).

*WAN subnet*: the existing network the workstation is connecting to via its *primary*
ethernet interface, typically with a static or dynamic IP address issued by the local
network infrastructure.

*primary network interface*: this will typically be a 'built-in' ethernet port on the
*workstation* that is already connecting to the local network which provides access to
the WAN.

*secondary network interface*: in our example, this will be a USB-to-Ethernet dongle
adding a second ethernet port to the *workstation*. If you already have an additional
ethernet port on your workstation (or server) then congratulations, just use that.

*client*: we'll use this term to refer to any ethernet device connected to the 
*development subnet* given a local IP address. I.e. the whole point of this
project is to give these devices acccess to the internet (or WAN) with the
minimum hassle of getting them connected.

*gateway*: the workstation will already be configured (either manually or via DHCP) with
an address of a gateway through which it will route its traffic to the WAN.  Also, for
our purposes, the workstation will be configured as a gateway and the *clients* will be
provided with the address of the workstation on the *development subnet* such that
they use it as their gateway.

*dhcp server*: the workstation will be configured as a *dhcp server* to issue IP
addresses to *clients* on the *development subnet* (e.g. 192.168.1.X). If you intend
to manually configure the network settings of *clients* then you can skip installing
the dhcp server.

## Network diagram

![network diagram](https://github.com/ijl20/dev_lan/blob/master/network_diagram.png "Network Diagram")

## USB-Ethernet dongle

## Secondary ethernet configuration

```
ls /sys/class/net
> enp0s31f6  lo
```

So the *primary network interface* is `enp0s31f6` (yours will be different)

Plug in the USB-Ethernet dongle.

```
ls /sys/class/net
> enp0s31f6  enxa0cec8c234c7  lo
```
The *secondary network interface* is `enxa0cec8c234c7` (yours will be different)

Stop the Ubuntu user-friendly NetworkManager from managing your network devices:

```
sudo vim /etc/Networkmanager/Networkmanager.conf
```
```
[main]
plugins=ifupdown,keyfile

[ifupdown]
managed=false

[device]
wifi.scan-rand-mac-address=no
```

```
sudo vim /etc/network/interfaces
```
```
# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo
iface lo inet loopback

auto enp0s31f6
iface enp0s31f6 inet dhcp

auto enxa0cec8c234c7
iface enxa0cec8c234c7 inet static
    address 192.168.1.1
    netmask 255.255.255.0
    post-up /etc/init.d/isc-dhcp-server start
    post-down /etc/init.d/isc-dhcp-server stop
```

```
netstat -a
ifconfig -a
sudo sudo systemctl restart networking
journalctl -xe
```

Use the ping command to check your secondary network interface
```
ping 192.168.1.1
```

## DHCP server configuration

```
sudo apt install isc-dhcp-server
```

```
sudo vim /etc/default/isc-dhcp-server
```

```
INTERFACESv4="enxa0cec8c234c7"
INTERFACESv6=""
```

```
sudo vim /etc/dhcp/dhcpd.conf
```
```
# dhcpd.conf
#
# Sample configuration file for ISC dhcpd
#
# Attention: If /etc/ltsp/dhcpd.conf exists, that will be used as
# configuration file instead of this file.
#

# option definitions common to all supported networks...
option domain-name "cl.cam.ac.uk";
option domain-name-servers 128.232.1.1, 128.232.1.2, 128.232.1.3;

default-lease-time 600;
max-lease-time 7200;

# The ddns-updates-style parameter controls whether or not the server will
# attempt to do a DNS update when a lease is confirmed. We default to the
# behavior of the version 2 packages ('none', since DHCP v2 didn't
# have support for DDNS.)
ddns-update-style none;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
#authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
#log-facility local7;

# No service will be given on this subnet, but declaring it helps the 
# DHCP server to understand the network topology.

#subnet 10.152.187.0 netmask 255.255.255.0 {
#}

# This is a very basic subnet declaration.

#subnet 10.254.239.0 netmask 255.255.255.224 {
#  range 10.254.239.10 10.254.239.20;
#  option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;
#}

# This declaration allows BOOTP clients to get dynamic addresses,
# which we don't really recommend.

#subnet 10.254.239.32 netmask 255.255.255.224 {
#  range dynamic-bootp 10.254.239.40 10.254.239.60;
#  option broadcast-address 10.254.239.31;
#  option routers rtr-239-32-1.example.org;
#}

# A slightly different configuration for an internal subnet.
#subnet 10.5.5.0 netmask 255.255.255.224 {
#  range 10.5.5.26 10.5.5.30;
#  option domain-name-servers ns1.internal.example.org;
#  option domain-name "internal.example.org";
#  option subnet-mask 255.255.255.224;
#  option routers 10.5.5.1;
#  option broadcast-address 10.5.5.31;
#  default-lease-time 600;
#  max-lease-time 7200;
#}

# Hosts which require special configuration options can be listed in
# host statements.   If no address is specified, the address will be
# allocated dynamically (if possible), but the host-specific information
# will still come from the host declaration.

#host passacaglia {
#  hardware ethernet 0:0:c0:5d:bd:95;
#  filename "vmunix.passacaglia";
#  server-name "toccata.example.com";
#}

# Fixed IP addresses can also be specified for hosts.   These addresses
# should not also be listed as being available for dynamic assignment.
# Hosts for which fixed IP addresses have been specified can boot using
# BOOTP or DHCP.   Hosts for which no fixed address is specified can only
# be booted with DHCP, unless there is an address range on the subnet
# to which a BOOTP client is connected which has the dynamic-bootp flag
# set.
#host fantasia {
#  hardware ethernet 08:00:07:26:c0:a5;
#  fixed-address fantasia.example.com;
#}

# You can declare a class of clients and then do address allocation
# based on that.   The example below shows a case where all clients
# in a certain class get addresses on the 10.17.224/24 subnet, and all
# other clients get addresses on the 10.0.29/24 subnet.

#class "foo" {
#  match if substring (option vendor-class-identifier, 0, 4) = "SUNW";
#}

#shared-network 224-29 {
#  subnet 10.17.224.0 netmask 255.255.255.0 {
#    option routers rtr-224.example.org;
#  }
#  subnet 10.0.29.0 netmask 255.255.255.0 {
#    option routers rtr-29.example.org;
#  }
#  pool {
#    allow members of "foo";
#    range 10.17.224.10 10.17.224.250;
#  }
#  pool {
#    deny members of "foo";
#    range 10.0.29.10 10.0.29.230;
#  }
#}
#
option subnet-mask 255.255.255.0;

option broadcast-address 192.168.1.255;

subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
}

```

```
sudo service isc-dhcp-server restart
```

```
sudo service isc-dhcp-server-status
```

```
● isc-dhcp-server.service - ISC DHCP IPv4 server
   Loaded: loaded (/lib/systemd/system/isc-dhcp-server.service; disabled; vendor preset: enabled)
   Active: active (running) since Thu 2019-04-04 10:23:20 BST; 3min 8s ago
     Docs: man:dhcpd(8)
 Main PID: 32704 (dhcpd)
    Tasks: 1 (limit: 4915)
   CGroup: /system.slice/isc-dhcp-server.service
           └─32704 dhcpd -user dhcpd -group dhcpd -f -4 -pf /run/dhcp-server/dhcpd.pid -cf /etc/dhcp/dhcpd.conf

Apr 04 10:23:20 ijl20-dell7040 dhcpd[32704]: No subnet declaration for enp0s31f6 (128.232.60.94).
Apr 04 10:23:20 ijl20-dell7040 dhcpd[32704]: ** Ignoring requests on enp0s31f6.  If this is not what
Apr 04 10:23:20 ijl20-dell7040 dhcpd[32704]:    you want, please write a subnet declaration
Apr 04 10:23:20 ijl20-dell7040 dhcpd[32704]:    in your dhcpd.conf file for the network segment
Apr 04 10:23:20 ijl20-dell7040 dhcpd[32704]:    to which interface enp0s31f6 is attached. **
Apr 04 10:23:20 ijl20-dell7040 dhcpd[32704]: 
Apr 04 10:23:20 ijl20-dell7040 dhcpd[32704]: Sending on   Socket/fallback/fallback-net
Apr 04 10:23:20 ijl20-dell7040 dhcpd[32704]: Server starting service.
Apr 04 10:23:27 ijl20-dell7040 dhcpd[32704]: DHCPREQUEST for 192.168.1.125 from 00:08:00:4a:78:48 via enxa0cec8c234c7
Apr 04 10:23:27 ijl20-dell7040 dhcpd[32704]: DHCPACK on 192.168.1.125 to 00:08:00:4a:78:48 via enxa0cec8c234c7
```

Connect a laptop (or any device) as a *client* on your development subnet.

Check it has been issued with a DHCP address in the range 192.168.1.X, with a netmask of 255.255.255.0, and
a gateway of 192.168.1.1.

From the *client*, test by pinging the gateway:
```
ping 192.168.1.1
```
If the ping fails then the first thing to do is look at the clients ethernet configuration e.g. with
```
ip a
```

Let's say for example the test *client* has an IP of 192.168.1.123.

From the *workstation* test by pinging the *client*:
```
ping 192.168.1.123
```

## NAT routing configuration

```
sudo apt install iptables-persistent
sudo apt install traceroute
```

In `/etc/sysctl.conf` set `net.ipv4.ip_forward=1` (by default the line will be commented out).

``` 
sudo vim /etc/sysctl.conf
```

```
 #
# /etc/sysctl.conf - Configuration file for setting system variables
# See /etc/sysctl.d/ for additional system variables.
# See sysctl.conf (5) for information.
#

#kernel.domainname = example.com

# Uncomment the following to stop low-level messages on console
#kernel.printk = 3 4 1 3

##############################################################3
# Functions previously found in netbase
#

# Uncomment the next two lines to enable Spoof protection (reverse-path filter)
# Turn on Source Address Verification in all interfaces to
# prevent some spoofing attacks
#net.ipv4.conf.default.rp_filter=1
#net.ipv4.conf.all.rp_filter=1

# Uncomment the next line to enable TCP/IP SYN cookies
# See http://lwn.net/Articles/277146/
# Note: This may impact IPv6 TCP sessions too
#net.ipv4.tcp_syncookies=1

# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1

# Uncomment the next line to enable packet forwarding for IPv6
#  Enabling this option disables Stateless Address Autoconfiguration
#  based on Router Advertisements for this host
#net.ipv6.conf.all.forwarding=1


###################################################################
# Additional settings - these settings can improve the network
# security of the host and prevent against some network attacks
# including spoofing attacks and man in the middle attacks through
# redirection. Some network environments, however, require that these
# settings are disabled so review and enable them as needed.
#
# Do not accept ICMP redirects (prevent MITM attacks)
#net.ipv4.conf.all.accept_redirects = 0
#net.ipv6.conf.all.accept_redirects = 0
# _or_
# Accept ICMP redirects only for gateways listed in our default
# gateway list (enabled by default)
# net.ipv4.conf.all.secure_redirects = 1
#
# Do not send ICMP redirects (we are not a router)
#net.ipv4.conf.all.send_redirects = 0
#
# Do not accept IP source route packets (we are not a router)
#net.ipv4.conf.all.accept_source_route = 0
#net.ipv6.conf.all.accept_source_route = 0
#
# Log Martian Packets
#net.ipv4.conf.all.log_martians = 1
#

###################################################################
# Magic system request Key
# 0=disable, 1=enable all
# Debian kernels have this set to 0 (disable the key)
# See https://www.kernel.org/doc/Documentation/sysrq.txt
# for what other values do
#kernel.sysrq=1

###################################################################
# Protected links
#
# Protects against creating or following links under certain conditions
# Debian kernels have both set to 1 (restricted) 
# See https://www.kernel.org/doc/Documentation/sysctl/fs.txt
#fs.protected_hardlinks=0
#fs.protected_symlinks=0
#

# Disable IPV6
net.ipv6.conf.all.disable_ipv6 = 1

net.ipv6.conf.default.disable_ipv6 = 1

net.ipv6.conf.lo.disable_ipv6 = 1

```

Check the setting is live with the command:
```
sysctl net.ipv4.ip_forward
```

```
 4332  2019-03-28 14:18:27 sudo systemctl restart networking
 4333  2019-03-28 14:18:30 sysctl net.ipv4.ip_forward
 4334  2019-03-28 14:18:50 sudo vim /etc/sysctl.conf
 4335  2019-03-28 14:19:04 sudo systemctl restart networking
 4336  2019-03-28 14:19:08 sysctl net.ipv4.ip_forward
 4337  2019-03-28 14:19:53 shutdown -r now
 4338  2019-03-28 14:23:31 sysctl net.ipv4.ip_forward
 4339  2019-03-28 14:23:54 ping 192.168.1.123
 4340  2019-03-28 14:24:03 ssh 192.168.1.123
 4341  2019-03-28 14:24:26 ssh 192.168.1.10
 
 shutdown -r now
 4338  2019-03-28 14:23:31 sysctl net.ipv4.ip_forward
 4339  2019-03-28 14:23:54 ping 192.168.1.123
 4340  2019-03-28 14:24:03 ssh 192.168.1.123
 4341  2019-03-28 14:24:26 ssh 192.168.1.10
 4342  2019-03-28 14:25:48 sudo iptables -t nat -A POSTROUTING -o enp0s31f6 -j MASQUERADE
 4343  2019-03-28 14:26:10 sudo apt install iptables-persistent
 4344  2019-03-28 14:27:25 cat /etc/iptables/rules.v4
 4345  2019-03-28 14:27:52 vim /etc/rc.local
 4346  2019-03-28 14:29:35 sudo vim /etc/rc.local
 4347  2019-03-28 14:30:08 sudo /etc/rc.local
 4348  2019-03-28 14:30:21 sudo sh /etc/rc.local
 4349  2019-03-28 14:30:34 ping 192.168.1.123
 4350  2019-03-28 14:30:53 shutdown -r now
 4351  2019-03-28 14:34:16 ping tfc-app2.cl.cam.ac.uk
 4352  2019-03-28 14:34:32 ping 192.168.1.10
 4353  2019-03-28 14:34:54 sudo service isc-dhcp-server status
 4354  2019-03-28 14:35:18 ip a
 4355  2019-03-28 14:36:11 cat /etc/dhcp/dhcpd.conf 
 4356  2019-03-28 14:37:09 ip a
 4357  2019-03-28 14:37:30 cat /etc/default/isc-dhcp-server 
 4358  2019-03-28 14:37:34 ip a
 4359  2019-03-28 14:40:08 sudo service isc-dhcp-server status
 4360  2019-03-28 14:40:17 sudo service isc-dhcp-server start
 4361  2019-03-28 14:40:22 sudo service isc-dhcp-server status
 4362  2019-03-28 14:42:18 ll /etc/init.d
 4363  2019-03-28 14:43:37 cat /etc/rc.local
 4364  2019-03-28 14:43:47 ping 192.168.1.123
 4365  2019-03-28 14:47:15 sudo systemctl disable isc-dhcp-server
 4366  2019-03-28 14:47:55 sudo vim /etc/network/interfaces
 4367  2019-03-28 14:50:08 shutdown -r now
 4368  2019-03-28 14:53:25 sudo service isc-dhcp-server status
 4369  2019-03-28 14:53:55 ping 192.168.1.123
 4370  2019-03-28 15:07:25 ping 128.232.1.1
 4371  2019-03-28 15:08:01 ping tfc-app2.cl.cam.ac.uk
 4372  2019-03-28 15:08:46 ping 128.232.60.1
 4373  2019-03-28 15:09:02 ping 128.232.1.1
 4374  2019-03-28 15:09:06 ping 128.232.60.1
 4375  2019-03-28 15:12:03 ip a
 4376  2019-03-28 15:13:13 sysctl net.ipv4.ip_forward
 4377  2019-03-28 15:13:58 sudo /sbin/iptables-restore </etc/iptables/rules.v4
 4378  2019-03-28 15:15:37 ping 128.232.60.1
 4379  2019-03-28 15:15:49 ping tfc-app2.cl.cam.ac.uk
 4380  2019-03-28 15:15:54 ip a
 4381  2019-03-28 15:16:31 ping tfc-app2.cl.cam.ac.uk
 4382  2019-03-28 15:17:09 cat /var/lib/dhcp/dhcpd.leases
 4383  2019-03-28 15:17:34 date
 4384  2019-03-28 15:18:18 ping 128.232.60.1
 4385  2019-03-28 15:18:28 ping tfc-app2.cl.cam.ac.uk
 4386  2019-03-28 15:19:55 sudo /etc/init.d/networking restart
 4387  2019-03-28 15:20:11 ping 192.168.1.10
 4388  2019-03-28 15:20:15 ping 192.168.1.123
 4389  2019-03-28 15:20:30 ping 128.232.60.1
 4390  2019-03-28 15:20:42 ping tfc-app2.cl.cam.ac.uk
 4391  2019-03-28 15:22:01 nslookup
 4392  2019-03-28 15:22:51 nslookup 128.232.1.1
 4393  2019-03-28 15:23:07 nslookup
 4394  2019-03-28 15:24:15 ping 8.8.8.8
 4395  2019-03-28 15:25:05 ping 128.232.98.198
 4396  2019-03-28 15:26:52 sudo vim /etc/NetworkManager/NetworkManager.conf 
 4397  2019-03-28 15:28:25 ip route list
 4398  2019-03-28 15:29:49 sudo vim /etc/network/interfaces
 4399  2019-03-28 15:34:54 sudo /etc/init.d/networking restart
 4400  2019-03-28 15:35:35 systemctl status networking.service
 4401  2019-03-28 15:36:29 shutdown -r now
 4402  2019-03-28 15:40:41 ping 192.168.1.10
 4403  2019-03-28 15:40:46 ping 192.168.1.123
 4404  2019-03-28 15:41:11 ip route list
 4405  2019-03-28 15:41:32 ping tfc-app2.cl.cam.ac.uk
 4406  2019-03-28 15:41:52 ping 128.232.60.1
 4407  2019-03-28 15:42:48 sudo vim /etc/network/interfaces
 4408  2019-03-28 15:43:33 sudo /etc/init.d/networking restart
 4409  2019-03-28 15:43:52 ip route list
 4410  2019-03-28 15:44:04 ping tfc-app2.cl.cam.ac.uk
 4411  2019-03-28 15:44:41 ping 192.168.1.123
 4412  2019-03-28 15:52:03 cat /etc/dhcp/dhcpd.conf 
 4413  2019-03-28 15:53:20 cat /etc/resolv.conf
 4414  2019-03-28 15:53:34 ping tfc-app2
 4415  2019-03-28 15:54:30 nmcli dev show
 4416  2019-03-28 15:55:49 nslookup google.com
 4417  2019-03-28 15:56:24 systemd-resolve --status
 4418  2019-03-28 15:58:04 ping 128.232.1.1
 4419  2019-03-28 15:59:41 sudo route -n
 4420  2019-03-28 15:59:54 netstat -rn
 4421  2019-03-28 16:07:14 iptables -t nat -L
 4422  2019-03-28 16:07:21 sudo iptables -t nat -L
 4423  2019-03-28 16:07:59 sysctl net.ipv4.ip_forward
 4424  2019-03-28 16:08:18 cat /etc/iptables/rules.v4
 4425  2019-03-28 16:09:40 sudo vim /etc/iptables/rules.v4
 4426  2019-03-28 16:10:36 sudo /sbin/iptables-restore </etc/iptables/rules.v4
 4427  2019-03-28 16:10:43 sudo iptables -t nat -L
 4428  2019-03-28 16:12:37 sudo iptables -t nat -D POSTROUTING
 4429  2019-03-28 16:13:14 sudo iptables -t nat -F POSTROUTING
 4430  2019-03-28 16:13:39 sudo iptables -t nat -A POSTROUTING -o enp0s31f6 -j MASQUERADE
 4431  2019-03-28 16:14:05 sudo iptables-save > /etc/iptables/rules.v4
 4432  2019-03-28 16:14:41 ll /etc/iptables
 4433  2019-03-28 16:14:50 su
 4434  2019-03-28 16:16:24 shutdown -r now
 4435  2019-03-28 16:20:22 netstat -nr
 4436  2019-03-28 16:20:33 ping 192.168.1.123
 4437  2019-03-28 16:20:44 ping tfc-app2.cl.cam.ac.uk
 4438  2019-03-28 17:09:21 ll p*
 4439  2019-04-02 08:47:39 netstat -nr
 4440  2019-04-02 08:48:18 ping analytics.twitter.com
 4441  2019-04-02 08:48:33 tracert analytics.twitter.com
 4442  2019-04-02 08:49:06 traceroute analytics.twitter.com
 4443  2019-04-02 08:50:14 sudo apt install traceroute
 4444  2019-04-02 09:04:47 tracetroute analytics.twitter.com
 4445  2019-04-02 09:04:52 traceroute analytics.twitter.com

```
