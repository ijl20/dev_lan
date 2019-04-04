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


sudo apt install isc-dhcp-server
 4314  2019-03-28 13:38:49 sudo apt install isc-dhcp-server
 4343  2019-03-28 14:26:10 sudo apt install iptables-persistent
 4443  2019-04-02 08:50:14 sudo apt install traceroute



ls /sys/class/net
 4284  2019-03-28 13:10:10 sudo vim /etc/network/interfaces
 4285  2019-03-28 13:10:51 netstat -a
 4286  2019-03-28 13:11:08 ifconfig -a
 4287  2019-03-28 13:12:03 sudo vim /etc/network/interfaces
 4288  2019-03-28 13:17:26 sudo systemctl restart networking
 4289  2019-03-28 13:17:47 journalctl -xe
 4290  2019-03-28 13:18:18 sudo vim /etc/network/interfaces
 4291  2019-03-28 13:19:43 sudo systemctl restart networking
 4292  2019-03-28 13:21:33 top
 4293  2019-03-28 13:20:29 sudo apt install isc-dhcp-server
 4294  2019-03-28 13:21:58 ping smartcambridge.org
 4295  2019-03-28 13:22:03 ping localhost
 4296  2019-03-28 13:22:12 ifconfig -a
 4297  2019-03-28 13:22:24 ping 128.232.60.94
 4298  2019-03-28 13:23:19 ping smartcambridge.org
 4299  2019-03-28 13:23:46 shutdown -r now
 4300  2019-03-28 13:27:21 ifconfig -a
 4301  2019-03-28 13:27:28 ping smartcambridge.org
 4302  2019-03-28 13:27:44 ping 182.232.60.94
 4303  2019-03-28 13:27:57 ping 128.232.60.94
 4304  2019-03-28 13:28:08 ping tfc-app2.cl.cam.ac.uk
 4305  2019-03-28 13:28:23 sudo vim /etc/network/interfaces
 4306  2019-03-28 13:28:58 sudo systemctl restart networking
 4307  2019-03-28 13:29:04 ping tfc-app2.cl.cam.ac.uk
 4308  2019-03-28 13:29:49 ping 128.232.60.1
 4309  2019-03-28 13:30:06 ping 128.232.1.1
 4310  2019-03-28 13:30:15 ping 128.232.1.2
 4311  2019-03-28 13:32:16 shutdown -r now
 4312  2019-03-28 13:35:51 ifconfig -a
 4313  2019-03-28 13:36:31 ping 128.232.1.1
 4314  2019-03-28 13:38:49 sudo apt install isc-dhcp-server
 4315  2019-03-28 13:39:34 sudo vim /etc/default/isc-dhcp-server 
 4316  2019-03-28 13:40:47 sudo vim /etc/dhcp/dhcpd.conf
 4317  2019-03-28 13:45:12 sudo service isc-dhcp-server-restart
 4318  2019-03-28 13:47:09 sudo service isc-dhcp-server restart
 4319  2019-03-28 13:52:23 ll /etc/netplan
 4320  2019-03-28 13:53:37 ip a
 4321  2019-03-28 13:57:54 ping 192.168.1.10
 4322  2019-03-28 14:01:01 ip a
 4323  2019-03-28 14:04:20 sudo server isc-dhcp-server status
 4324  2019-03-28 14:04:34 sudo service isc-dhcp-server status
 4325  2019-03-28 14:06:18 sudo vim /etc/dhcp/dhcpd.conf
 4326  2019-03-28 14:12:33 sudo systemctl restart networking
 4327  2019-03-28 14:13:02 sudo service isc-dhcp-server status
 4328  2019-03-28 14:14:08 sudo service isc-dhcp-server restart
 4329  2019-03-28 14:14:13 sudo service isc-dhcp-server status
 4330  2019-03-28 14:16:49 sudo vim /etc/sysctl.conf
 4331  2019-03-28 14:18:13 sysctl net.ipv4.ip_forward
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

