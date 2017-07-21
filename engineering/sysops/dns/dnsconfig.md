
# Dns config and setting forward look up addresses




### Editing config options

Opening the config file
```
nano /etc/bind/named.conf.options
```

Add allow-query option with any parameter so anyone can query this dns server.

```
options {
	directory "/var/cache/bind";
        allow-query { any; };
	auth-nxdomain no;    # conform to RFC1035
	listen-on-v6 { any; };
};
```








## Adding dns entries manually 
[source][3]


#### Define the zones for the local domain:

Editing local zone file
```
sudo nano /etc/bind/named.conf.local
```

#### Add a zone for the local domain in the file opened

```
zone "home.lan" IN {
    type master;
    file "/etc/bind/zones/home.lan.db";
};
```





#### Create the zones directory:
```
sudo mkdir /etc/bind/zones
```

#### Configure the local domain:
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




### Restart services to use the new settings:

Restart bind for changes to take effect.
We can restart the whole docker or go inside docker and restart the bind service only.






## Adding dns entries through WEBMIN log in using browser

Login to webmin thru browser using ip:port, where ip is local network ip where webmin docker/process is running and the port at which it is broadcasted. 

```
example:
192.168.0.50:10000
```

This will open a login window, the default user id and password are as configured, in our case they are root and password, logged in shows the GUI panel. It has Dashboard for stats and webmin panel for controlling the System and applications running in webmin. The Webmin panel is for configuring webmin default options and interface. THe System panel is for starting stopping processes, editing configurations and log file parameters, as well as backing up filesystems. It can also be used to update software packages. The third panel is Server panel, which has all the current running server applications BIND is in this tab under the name BIND DNS Server, clicking on it we entrr the BIND DNS server page, here we can change all the BIND configurations as well as add\edit\delete dns entries directly through GUI.

Bind server should be pre configured for initial use till here, we can start with adding dns entries for a domain name. To start with that, we go to Existing DNS zones and click on Create Master zone. This takes us to a page for Creating Master Zone. 


* Zone type:  Forward
* Domain name/ Network:  firstdomainexample.com
* Records file: Automatic
* Master server: (leave default)
* Email address: ``` default@email.com```
* Use zone template?: no
* Add reverses for template addresses?: yes
* Refresh time: default
* Expiry time: default

After selecting options as described above, click create for making a new masterzone for domain name "firstdomainexample.com". This takes us to the editing page of the newly created master zone. To add ip addresses to this domain name, we select the Address option, which takes us to Address records page, this will display any previous records if any, in our case will not show any old records, only option to add new ones.

Adding new records

* Name: (leave blank for default address in our case firstdomainexample.com)
* Address: 192.168.10.10
* Update reverse: (default)
* Time-to-live: (default)

Click Create.
This creates a ip address for domain name "firstdomainexample.com" as 192.168.10.10
We add another ip address now.

* Name: www
* Address: 192.168.10.15
* Update reverse: (default)
* Time-to-live: (default)

Click Create.
This creates a ip address for domain name ``` "www.firstdomainexample.com" ``` as 192.168.10.15
And so on we can create more ip addresses for subdomains if needed. Click on apply zone and apply configuration for configuration changes to take effect. We can also restart bind from system services for the changes to take effect.



# Testing the dns entries

Now from any other client machine which is using the dns service which we configured, we can check if dns resolution is working.

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




### Additional Reference Resources for info

* [Digital Ocean link on how to configure BIND on ubuntu][1]
* [Debian wiki for BIND][2]
* [Permalink to lani78.com/2012/07/22/setting-up-a-dns-for-the-local-network-on-the-ubuntu-12-04-precise-pangolin-server][3]


[1]: https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-an-authoritative-only-dns-server-on-ubuntu-14-04
[2]: https://wiki.debian.org/Bind9
[3]: https://lani78.com/2012/07/22/setting-up-a-dns-for-the-local-network-on-the-ubuntu-12-04-precise-pangolin-server
