
# Dns config and setting forward look up addresses




## Editing config options


##### Editing manually the config options using nano and command line

Opening the config file
```
nano /etc/bind/named.conf.options
```

Add allow-query option with any parameter so anyone can query this dns server.

```
options {
	directory "/var/cache/bind";
    allow-query { any; };  # this line added to allow query from any client
	auth-nxdomain no;    # conform to RFC1035
	listen-on-v6 { any; };
};
```

##### Editing config options using webmin for Dockerized container running BIND server and webmin panel

Login to webmin thru browser using ip:port, where ip is local network ip where webmin docker/process is running and the port at which it is broadcasted. 

```
example:
192.168.0.50:10000
```

This will open a window as follows(My default login id is root and password is password):
![img][login]

After we log in successfully, the dashboard opens by default, as shown below:
![img][dashboard]

To edit the config file we need to go to the webmin server panel, and select BIND DNS server panel as shown below:
![img][BIND DNS server panel]


In the BIND DNS server, we need to click on edit config file option, which will bring us to a Edit config file window, as shown below:
![img][config1]

In this window it shows us config file is subdivided into 3 sections, we need to edit the named.conf.options file only for changes in options that we added, so we click on the drop down, and select on /etc/bind/named.conf.options and click on edit, as shown below:
![img][config2]

This opens up the /etc/bind/named.conf.options file for editing in the window, we added the highlighted line 
```
allow-query { any; }; 
```
after that we click on Save, also we click on apply configuration after save, as shown below:
![img][config3]

All done we have succesfully edited and applied the configuration to our BIND server on docker container. 






## Adding dns entries 

We can add dns entries using cmdline and a text editor like nano, as well as using a GUI based application. This article describes 2 methods to add how to add entries, 

1. Manually using cmdline and nano
2. Using GUI webmin running in browser for our dockerized container which is running webmin and DNS server 

### Manual CMDline method 
[source][3]


##### Step 1 

Define the zones for the local domain, we add zone name, zone type and zone database file location in "/etc/bind/named.conf.local" as per syntax shown in below example.

Editing local zone file
```
sudo nano /etc/bind/named.conf.local
```

Add a zone, zone type and zone file database location for the local domain in the file opened

```
zone "home.lan" IN {
    type master;
    file "/etc/bind/zones/home.lan.db";
};
```




##### Step 2

Now since we already added the zone file database location, we also need to create the same file and directory, after we create the file we edit it as described below 


Create the zones directory:
```
sudo mkdir /etc/bind/zones
```

Configure the local domain:
```
sudo nano /etc/bind/zones/home.lan.db
```

Example settings, change to match your host names and ip-addresses:
```
; Use semicolons to add comments.
; Host-to-IP Address DNS Pointers for home.lan
; Note: The extra “.” at the end of the domain names are important.

; The following parameters set when DNS records will expire, etc.
; Importantly, the serial number must always be iterated upward to prevent
; undesirable consequences. A good format to use is YYYYMMDDII where
; the II index is in case you make more that one change in the same day.
$ORIGIN .
$TTL 86400      ; 1 day
home.lan. IN SOA ubuntu.home.lan. hostmaster.home.lan. (
    2008080901 ; serial
    8H ; refresh
    4H ; retry
    4W ; expire
    1D ; minimum
)

; NS indicates that ubuntu is the name server on home.lan
; MX indicates that ubuntu is (also) the mail server on home.lan
home.lan. IN NS ubuntu.home.lan.
home.lan. IN MX 10 ubuntu.home.lan.

$ORIGIN home.lan.

; Set the address for localhost.home.lan
localhost    IN A 127.0.0.1

; Set the hostnames in alphabetical order
print-srv    IN A 192.168.0.9
router       IN A 192.168.0.1
server       IN A 192.168.0.5
ubuntu       IN A 192.168.0.2
xbox         IN A 192.168.0.3
```



##### Step 3

Now for the new zone settings to take effect we need to restart the BIND services.
We can restart the whole docker or go inside docker and restart the bind service only. Since docker is only running webmin and BIND, we can restart the docker using (dockername is bind here, replace it with your docker name):

```
sudo docker restart bind
```





## Adding dns entries through WEBMIN log in using browser

##### Step 1
Login to webmin thru browser using ip:port, where ip is local network ip where webmin docker/process is running and the port at which it is broadcasted. 

```
example:
192.168.0.50:10000
```

![img][login]


This will open a login window, the default user id and password are as configured, in our case they are root and password, logged in shows the GUI panel. It has Dashboard for stats and webmin panel for controlling the System and applications running in webmin. The Webmin panel is for configuring webmin default options and interface. The System panel is for starting stopping processes, editing configurations and log file parameters, as well as backing up filesystems. It can also be used to update software packages. 

![img][webmin panel]
![img][system panel]

##### Step 2

The third panel is Server panel, which has all the current running server applications BIND is in this tab under the name BIND DNS Server, clicking on it we entrr the BIND DNS server page, here we can change all the BIND configurations as well as add\edit\delete dns entries directly through GUI.

![img][BIND DNS server panel]

##### Step 3

Bind server should be pre configured for initial use till here, we can start with adding dns entries for a domain name. To start with that, we go to Existing DNS zones and click on Create Master zone. This takes us to a page for Creating Master Zone. 

![img][add master zone]


After selecting options as described above, click create for making a new masterzone for domain name "firstdomainexample.com".

* Zone type:  Forward
* Domain name/ Network:  firstdomainexample.com
* Records file: Automatic
* Master server: (leave default)
* Email address: ``` default@email.com```
* Use zone template?: no
* Add reverses for template addresses?: yes
* Refresh time: default
* Expiry time: default

##### Step 4

This takes us to the editing page of the newly created master zone. To add ip addresses to this domain name, we select the Address option, which takes us to Address records page, this will display any previous records if any, in our case will not show any old records, only option to add new ones.

![img][edit master zone1]
![img][edit master zone2]


##### Step 5

Adding new records

![img][add address1]

* Name: (leave blank for default address in our case firstdomainexample.com)
* Address: 192.168.10.10
* Update reverse: (default)
* Time-to-live: (default)

Click Create.
This creates a ip address for domain name "firstdomainexample.com" as 192.168.10.10


##### Step 6

We add another ip address now.

![img][add address2]

* Name: www
* Address: 192.168.10.15
* Update reverse: (default)
* Time-to-live: (default)

Click Create.
This creates a ip address for domain name ``` "www.firstdomainexample.com" ``` as 192.168.10.15


##### Step 7

We add our last ip address now.

![img][add address3]

* Name: host1
* Address: 192.168.10.21
* Update reverse: (default)
* Time-to-live: (default)

Click Create.
This creates a ip address for domain name ``` "host1.firstdomainexample.com" ``` as 192.168.10.21
And so on we can create more ip addresses for subdomains if needed. Click on apply zone and apply configuration for configuration changes to take effect. We can also restart bind from system services for the changes to take effect. 


##### Step 8

We apply configuration from BINS DNS server page.
![img][Apply Configuration]



# Testing the dns entries

We have two ways of testing using host command, using a specific dns server on the network or using the default dns server allocated to client by dhcp server.


##### Using a dns server on the network which is not given by dhcp

To use a random dns server, we use the syntax " host domainname dnsserveraddress"

test command


```
host firstdomainexample.com 192.168.0.50
```
output
```
Using domain server:
Name: 192.168.0.50
Address: 192.168.0.50#53
Aliases:

firstdomainexample.com has address 192.168.10.10

```

##### Using a dns server assigned to client by dhcp server

In this case our dhcp server allocated the dns server 192.168.0.50 as default primary dns server. Now from any other client machine which is using the dns service which we configured, we can check if dns resolution is working. In this case we use the syntax " host domainname"

test command

```
host www.firstdomainexample.com
```

output
```
Using domain server:
Name: 192.168.0.50
Address: 192.168.0.50#53
Aliases:

www.firstdomainexample.com has address 192.168.10.15

```



to do  

add link for dhcp how to change dns



### Additional Reference Resources for info

* [Digital Ocean link on how to configure BIND on ubuntu][1]
* [Debian wiki for BIND][2]
* [Permalink to lani78.com/2012/07/22/setting-up-a-dns-for-the-local-network-on-the-ubuntu-12-04-precise-pangolin-server][3]


[1]: https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-an-authoritative-only-dns-server-on-ubuntu-14-04
[2]: https://wiki.debian.org/Bind9
[3]: https://lani78.com/2012/07/22/setting-up-a-dns-for-the-local-network-on-the-ubuntu-12-04-precise-pangolin-server










[Apply Configuration]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/Apply%20configuration.png
[BIND DNS server panel]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/Bind%20DNS%20Server%20panel%20in%20Servers.png
[Configuring bootup shutdown]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/Configuring%20application%20bootup%20or%20shutdown.png
[add master zone]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/Create%20New%20master%20zone.png
[dashboard]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/Dashboard.png
[config1]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/Editing%20config%20file%201.png
[config2]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/Editing%20config%20file%202.png
[config3]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/Editing%20config%20file%203.png
[lower panel]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/Lower%20panel%20option%20of%20BIND%20DNS%20server.png
[system panel]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/System%20panel%20options.png
[view process]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/Viewing%20running%20processes.png
[webmin panel]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/Webmin%20panel%20options.png
[login]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/login.png
[add address1]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/add%20address1.png
[add address2]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/add%20address2.png
[add address3]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/add%20address3.png
[edit master zone1]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/edit%20master%20zone1.png
[edit master zone2]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/edit%20master%20zone2.png
[create master zone]: https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/Images/create%20master%20zone.png
