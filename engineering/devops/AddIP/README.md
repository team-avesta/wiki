# Adding additional IP Addresses to a Network Interface Card on Ubuntu
[Source](https://www.garron.me/en/linux/add-secondary-ip-linux.html "Permalink to https://www.garron.me/en/linux/add-secondary-ip-linux.html")



## Add a second temporary IP address


Using ifconfig

If you want to add a secondary IP address to a NIC already in use in Linux, and have that change only temporary. Enter this command:
```
ifconfig [nic]:0 [IP-Address] netmask [mask] up
```

An example is shown below
```
ifconfig eth0:0 192.168.1.2 netmask 255.255.255.0 up
```
You need to be root in order to execute that command.





## Add a permanent IP address

Ubuntu

For Ubuntu systems, edit the /etc/network/interfaces file
```
vim /etc/network/interfaces
```

Add this for one extra IP
```
auto [NIC]:[n]
iface [NIC]:[n] inet static
address [ip.add.rr.ss]
gateway [gw.ip.ad.rs]
netmask [ne.tm.as.kk]
```

Here an example:
```
auto eth0:1
iface eth0:1 inet static
address 192.168.0.1
gateway 192.168.0.254
netmask 255.255.255.0
```

You can add as many blocks as you want. Just change eth0:1 for eth0:2, eth0:3 and so on.
If you are adding additional IPs to eth1, or eth2 also modify that on the example.
