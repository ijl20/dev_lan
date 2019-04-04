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

