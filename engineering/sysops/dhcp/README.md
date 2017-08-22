# Introduction to DHCP basics and Terminology
[source][1]

## DHCP introduction

DHCP (Dynamic Host Configuration Protocol) is a standardized networking protocol used on IP networks that dynamically configures IP addresses and other information that is needed for Internet communication.

DHCP allows computers and other devices to receive an IP address automatically from a central DHCP server, reducing the need for a network administrator or a user from having to configure these settings manually.

DHCP servers maintain a database of available IP addresses and other kinds of addresses, such as a default route, and one or more DNS server addresses.

DHCP may be used to configure some of these settings and the remaining settings may be manually configured.

DHCP servers typically grant IP addresses to devices only for a limited interval. Devices are responsible for renewing their IP address lease before that interval expires, and must stop using the address once the interval has expired, if they have not been able to renew it.



## DHCP process

There are a surprising number of steps which take place before your computer or mobile device can “talk” or “access” network / internet resources.

Lets look at the steps which take place from the moment you either plug in a network cable or connect to a wireless network..

##### 1. DHCPDISCOVER

It is a DHCP message that marks the beginning of a DHCP interaction between the client (your home PC or mobile phone/tablet) and server (most likely your home router (BT homehub for example)).

This message is sent by the client that is connected to the local subnet. It’s a broadcast message that uses 255.255.255.255 as destination IP address while the source IP address is 0.0.0.0

##### 2. DHCPOFFER

It is DHCP message that is sent in response to DHCPDISCOVER by a DHCP server (BT homehub) to DHCP client (your mobile phone/PC).

This message contains the network configuration settings for the client that sent the DHCPDISCOVER message.

##### 3. DHCPREQUEST

This DHCP message is sent in response to DHCPOFFER indicating that the client has accepted the network configuration sent in DHCPOFFER message from the server.

##### 4. DHCPACK

This message is sent by the DHCP server in response to DHCPREQUEST received from the client. This message marks the end of the process that started with DHCPDISCOVER. The DHCPACK message is nothing but an acknowledgement by the DHCP server that authorizes the DHCP client to start using the network configuration it received from the DHCP server earlier.

##### 5. DHCPNAK

This message is the exact opposite to DHCPACK described above. This message is sent by the DHCP server when it is not able to satisfy the DHCPREQUEST message from the client.

##### 6. DHCPDECLINE

This message is sent from the DHCP client to the server in case the client finds that the IP address assigned by DHCP server is already in use.

##### 7. DHCPINFORM

This message is sent from the DHCP client in case the IP address is statically configured on the client and only other network settings or configurations are desired to be dynamically acquired from DHCP server.

##### 8. DHCPRELEASE

This message is sent by the DHCP client in case it wants to terminate the lease of network address it has be provided by DHCP server.

Now we’ve looked at the steps involved, below shows a simple diagram using the steps above as to how DHCP commonly works.

![dhcp1](https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dhcp/Images/dhcp1.png)

 

## DHCP Terminology
[source][2]


#### Pool/Scope

The IP addresses available for clients to request. Many home networks today use a 192.168.x.0/24, like 192.168.1.0/24. This provides an address range of 192.168.1.1 – 192.168.1.254. Any address that is not taken in this range is available to be “leased” to clients on the network.

#### Lease

A lease is an IP address given to a client. This is called a lease because it expires after a certain period of time and can return to the address pool if necessary. Typically, a client will continue requesting the same leased IP address at half the configured lease time. For instance, if an address is leased to a device for 24 hours, it will request the address again at 12 hours to prevent the address from returning to the pool and being used by another device. If this request does not happen (perhaps if the device has left the network), the IP address will return to the pool and be reassigned to another device if one requests.

#### IP Reservation

With an IP reservation, you can instruct the DHCP server to always assign the same address to a device using that device’s MAC address. A MAC address is the hardcoded or virtual hardware address used by a device’s network interface card (NIC).

#### IP Conflict

An IP conflict occurs when 2 or more devices on a network attempt to request the same IP address. This typically happens when one device has a static IP address set that is within the address pool and another device attempts to request that IP address. To avoid this, it is best to configure an IP reservation for the static client or modify the address scope such that they do not contain the IP address statically assigned to the client. For instance, if a client is given 192.168.1.10, set the scope to only provide IP addresses from 192.168.1.11 – 192.168.1.253. This will allow all of the addresses below 192.168.1.11 to be statically assigned without conflicting with a DHCP host. 





## The Concept of Lease

As mentioned above, your device is simply assigned an IP address (leased) by the DHCP server. This means if the lease expires, the DHCP server is free to assign the same IP address to any other host or device requesting for the same.

This is why a lot of networks which see frequent client connections and disconnects (for example wifi hot spots) will have very short lease times (2-4 hours up to 8-10 hours).

Where as a lot of physical networks, which have desktops connected, you tend to find the lease is set for a longer time period (7-10 days).

It’s also worth noting that the DHCP client tries to renew the lease after half of the lease time has expired. This is done by the exchange of DHCPREQUEST and DHCPACK messages. While doing all this, the client enters the renewing stage.




## Additional Links for Info

* [Wiki page for dhcp server][3]


[1]: http://www.michaelriccioni.com/introduction-to-dhcp-the-basics/ "permalink to http://www.michaelriccioni.com/introduction-to-dhcp-the-basics/"
[2]: https://linuxacademy.com/howtoguides/posts/show/topic/13613-introduction-to-dhcp "Linux Academy link to DHCP"
[3]: https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol