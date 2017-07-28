

# isc-dhcp-server
[Source](https://help.ubuntu.com/community/isc-dhcp-server "Permalink to Ubuntu Community Page on Ubuntu Official Documentation Site")

The Dynamic Host Configuration Protocol (DHCP) is a network service that enables host computers to be automatically assigned settings from a server as opposed to manually configuring each network host. Computers configured to be DHCP clients have no control over the settings they receive from the DHCP server, and the configuration is transparent to the computer's user.



# Installation

At a terminal prompt, enter the following command to install dhcpd: 
```
sudo apt-get install isc-dhcp-server
```

You will probably need to change the default configuration by editing /etc/dhcp3/dhcpd.conf to suit your needs and particular configuration.

You also need to edit /etc/default/isc-dhcp-server to specify the interfaces dhcpd should listen to. By default it listens to eth0. 

Also, you have to assign a static ip to the interface that you will use for dhcp. If you will use eth0 for providing addresses in the 192.168.1.x subnet then you should assign for instance ip 192.168.1.1 to the eth0 interface using NetworkManager. Without this step you will get an error from dhcpd when starting the service.

[Linux academy link to Introduction to DHCP](https://linuxacademy.com/howtoguides/posts/show/topic/13613-introduction-to-dhcp)


# Configuration

Most commonly, what you want to do is assign an IP address randomly. This can be done with settings as follows: 

```
nano -w /etc/dhcp/dhcpd.conf
```


```
# Sample /etc/dhcpd.conf
# (add your comments here) 
default-lease-time 600;
max-lease-time 7200;
option subnet-mask 255.255.255.0;
option broadcast-address 192.168.1.255;
option routers 192.168.1.254;
option domain-name-servers 192.168.1.1, 192.168.1.2;
option domain-name "mydomain.example";

subnet 192.168.1.0 netmask 255.255.255.0 {
range 192.168.1.10 192.168.1.100;
range 192.168.1.150 192.168.1.200;

```

This will result in the DHCP server giving a client an IP address from the range 192.168.1.10-192.168.1.100 or 192.168.1.150-192.168.1.200. It will lease an IP address for 600 seconds if the client doesn't ask for a specific time frame. Otherwise the maximum (allowed) lease will be 7200 seconds. The server will also "advise" the client that it should use 255.255.255.0 as its subnet mask, 192.168.1.255 as its broadcast address, 192.168.1.254 as the router/gateway and 192.168.1.1 and 192.168.1.2 as its DNS servers. Here 192.168.1.1 is primary server as it is first in the list and in case the client is not able to reach the primary server it will send its query to the secondary server which is 192.168.1.2, further dns servers can be added and client will query the latter ones if former ones are down.


## Start and stop service

```
 sudo service isc-dhcp-server restart
 sudo service isc-dhcp-server start
 sudo service isc-dhcp-server stop 
```
We need to restart the server for changes to take effect. It can be done by a single restart command or a combination of start\stop.



# isc-dhcp-server and multiple interfaces

multiple interfaces example



### Interface

```
nano -w /etc/network/interfaces
```

Here we edit the interface files for 2 different subnets over 2 Network cards, using this following example. 

```
auto lo
iface lo inet loopback

mapping hotplug
        script grep
        map eth1

iface eth1 inet dhcp

auto eth0
iface eth0 inet static
    address 10.152.187.1
    netmask 255.255.255.0

auto wlan0
  iface wlan0 inet static
    address 192.168.1.1
    netmask 255.255.255.0
    up     /sbin/iwconfig wlan0 mode TTTTTT && /sbin/iwconfig wlan0 enc
restricted && /sbin/iwconfig wlan0 key [Y] XXXXXXXX && /sbin/iwconfig
wlan0 essid SSSSSSSS ; these are wireless lan parameters which describe SSID name, password and transmission mode

auto eth1

```

### Select Interface card

```
nano -w /etc/default/isc-dhcp-server
```

We edit this defaults file to edit the default NIC's which we want to use, here we use 2 NIC's in the following example

```
INTERFACES="wlan0 eth0"

```


### Configure Subnet

```
nano -w /etc/dhcp/dhcpd.conf
```

This example explains use of 2 different subnets and how we configure them, it is similar to our single subnet configuration except here we have added static host ip for bla1, bla2 on first subnet and bla3 for second subnet. Also added hardware ethernet address for the server to know which client to assign the static ip.  

```
ddns-update-style none;
log-facility local7;

subnet 192.168.1.0 netmask 255.255.255.0 {

        option routers                  192.168.1.1;
        option subnet-mask              255.255.255.0;
        option broadcast-address        192.168.1.255;
        option domain-name-servers      194.168.4.100;
        option ntp-servers              192.168.1.1;
        option netbios-name-servers     192.168.1.1;
        option netbios-node-type 2;
        default-lease-time 86400;
        max-lease-time 86400;

        host bla1 {
                hardware ethernet DD:GH:DF:E5:F7:D7;
                fixed-address 192.168.1.2;
        }
        host bla2 {
                hardware ethernet 00:JJ:YU:38:AC:45;
                fixed-address 192.168.1.20;
        }
}

subnet  10.152.187.0 netmask 255.255.255.0 {

        option routers                  10.152.187.1;
        option subnet-mask              255.255.255.0;
        option broadcast-address        10.152.187.255;
        option domain-name-servers      194.168.4.100;
        option ntp-servers              10.152.187.1;
        option netbios-name-servers     10.152.187.1;
        option netbios-node-type 2;

        default-lease-time 86400;
        max-lease-time 86400;

        host bla3 {
                hardware ethernet 00:KK:HD:66:55:9B;
                fixed-address 10.152.187.2;
        }
}
```



### Additional Links for Reference

[Tldp foundation link for DHCP server set up](http://www.tldp.org/HOWTO/DHCP/x369.html)
[Link for dhcp3 server for reference only it is now obselete and has been replaced by isc-dhcp-server](https://help.ubuntu.com/community/dhcp3-server)
[FreeBSD page for detailed description of config options](https://www.freebsd.org/cgi/man.cgi?query=dhcpd.conf&sektion=5&apropos=0&manpath=FreeBSD+9.0-RELEASE+and+Ports)

