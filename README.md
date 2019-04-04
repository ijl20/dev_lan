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
sudo vim /etc/network/interfaces
netstat -a
ifconfig -a
sudo vim /etc/network/interfaces
sudo systemctl restart networking
journalctl -xe
```

## DHCP server configuration

```
sudo apt install isc-dhcp-server
```

## NAT routing configuration

```
sudo apt install iptables-persistent
sudo apt install traceroute
```

```
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
```

