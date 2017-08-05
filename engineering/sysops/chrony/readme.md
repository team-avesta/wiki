

# Network Time Protocol (NTP)

Network Time Protocol (NTP) is a networking protocol for clock synchronization between computer systems over packet-switched, variable-latency data networks. In operation since before 1985, NTP is one of the oldest Internet protocols in current use. NTP was designed by David L. Mills of the University of Delaware. NTP is intended to synchronize all participating computers to within a few milliseconds of Coordinated Universal Time (UTC).






## Chrony 

Chrony is a versatile implementation of the Network Time Protocol (NTP). It can synchronize the system clock with NTP servers, reference clocks (e.g. GPS receiver), and manual input using wristwatch and keyboard. It can also operate as an NTPv4 (RFC 5905) server and peer to provide a time service to other computers in the network.

It is designed to perform well in a wide range of conditions, including intermittent network connections, heavily congested networks, changing temperatures (ordinary computer clocks are sensitive to temperature), and systems that do not run continuosly, or run on a virtual machine.

Typical accuracy between two machines synchronised over the Internet is within a few milliseconds; on a LAN, accuracy is typically in tens of microseconds. With hardware timestamping, or a hardware reference clock, sub-microsecond accuracy may be possible.

Two programs are included in chrony, chronyd is a daemon that can be started at boot time and chronyc is a command-line interface program which can be used to monitor chronyd’s performance and to change various operating parameters whilst it is running.

###### Supported systems

The software is supported on Linux, FreeBSD, NetBSD, macOS, and Solaris. Closely related systems may work too. Any other system will likely require a porting exercise.


#### Installation 

```
sudo apt-get install chrony
```

Editing configuration file for chrony, changes in here reflect if chrony is used as a client only or as a server and a client.

```
sudo nano /etc/chrony/chrony.conf
```


#### Client config file changes
[source](https://chrony.tuxfamily.org/doc/1.31/manual.html)

	(our client uses chrony version 1.30)
We edited the chrony.conf to add the changes below, in which server to poll we have specified a domainname which we get resolved by our dns to get a local ip, where chrony server is running. We can also use direct ip or different hostname as our need be.
```
server 0.debian.pool.ntp.org offline iburst minpoll 8
server 1.debian.pool.ntp.org offline iburst minpoll 8
server 2.debian.pool.ntp.org offline iburst minpoll 8
server 3.debian.pool.ntp.org offline iburst minpoll 8
makestep 1 -1
```

#### Server config file changes
[source](https://chrony.tuxfamily.org/doc/2.1/manual.html)

	(Our server uses chrony version 2.1.1)
We need to add the allow directive to let the chrony behave as server, rest configurations are same as client for its client behavior, except for a server we will need a different domainname as higher level server from which this system polls time, it can not poll time from itself.
```
allow foo.example.net
allow 1.2
allow 3.4.5
allow 6.7.8/22
allow 6.7.8.9/22
allow 2001:db8::/32
allow 0/0
allow ::/0
allow
```

#### Add DNS entries 

We create a domainname ```debian.pool.ntp.org offline``` and add address entries for 0 1 2 3 which all refer to our private ip address\es
You can refer [here](https://github.com/team-avesta/wiki/blob/master/engineering/sysops/dns/dnsconfig.md) to add domain name addresses to DNS server.
```
0.debian.pool.ntp.org resolves to 192.168.1.101
```

### Chrony server

To use a system as a chrony server, we need to use allow directive to allow other systems to poll it. 




##### allow
[source](https://chrony.tuxfamily.org/manual.html#allow-directive "chrony documentation for allow-directive")

The allow command is used to designate a particular subnet from which NTP clients are allowed to access the computer as an NTP server.

The default is that no clients are allowed access, i.e. chronyd operates purely as an NTP client. If the allow directive is used, chronyd will be both a client of its servers, and a server to other clients.

Examples of use of the command are as follows:
```
allow foo.example.net
allow 1.2
allow 3.4.5
allow 6.7.8/22
allow 6.7.8.9/22
allow 2001:db8::/32
allow 0/0
allow ::/0
allow
```
The first command allows the named node to be an NTP client of this computer. The second command allows any node with an IPv4 address of the form 1.2.x.y (with x and y arbitrary) to be an NTP client of this computer. Likewise, the third command allows any node with an IPv4 address of the form 3.4.5.x to have client NTP access. The fourth and fifth forms allow access from any node with an IPv4 address of the form 6.7.8.x, 6.7.9.x, 6.7.10.x or 6.7.11.x (with x arbitrary), i.e. the value 22 is the number of bits defining the specified subnet. (In the fifth form, the final byte is ignored). The sixth form is used for IPv6 addresses. The seventh and eighth forms allow access by any IPv4 and IPv6 node respectively. The ninth forms allows access by any node (IPv4 or IPv6).

A second form of the directive, allow all, has a greater effect, depending on the ordering of directives in the configuration file. To illustrate the effect, consider the two examples

```
allow 1.2.3.4
deny 1.2.3
allow 1.2
and

allow 1.2.3.4
deny 1.2.3
allow all 1.2
```
In the first example, the effect is the same regardles of what order the three directives are given in. So the 1.2.x.y subnet is allowed access, except for the 1.2.3.x subnet, which is denied access, however the host 1.2.3.4 is allowed access.

In the second example, the allow all 1.2 directives overrides the effect of any previous directive relating to a subnet within the specified subnet. Within a configuration file this capability is probably rather moot; however, it is of greater use for reconfiguration at run-time via chronyc (see section allow all).

Note, if the initstepslew directive (see section initstepslew) is used in the configuration file, each of the computers listed in that directive must allow client access by this computer for it to work.






### Chrony client 
[source](https://chrony.tuxfamily.org/doc/3.1/chrony.conf.html)

We use a system as a chrony client by default, and the servers which we use for time syncronization should be specified as our network needs, it can be a domainname that our dns can resolve in a network, or can be direct ip of the server. We can use 1 or multiple servers for polling, as well as a pool of servers. The server config parameters are as following syntax.

```
server hostname [option]
```

The server directive allows NTP servers to be specified. The client/server relationship is strictly hierarchical : a client may synchronise its system time to that of the server, but the server’s system time will never be influenced by that of a client.

The server directive is immediately followed by either the name of the server, or its IP address. The server command also supports a number of subfields (which may be defined in any order):

* port

   This option allows the UDP port on which the server understands NTP requests to be specified. For normal servers this option should not be required (the default is 123, the standard NTP port).

* minpoll

   Although chronyd will trim the rate at which it samples the server during normal operation, the user may wish to constrain the minimum polling interval. This is always defined as a power of 2, so minpoll 5 would mean that the polling interval cannot drop below 32 seconds. The default is 6 (64 seconds).

* maxpoll

   In a similar way, the user may wish to constrain the maximum polling interval. Again this is specified as a power of 2, maxpoll 9 indicates that the polling interval must stay at or below 512 seconds. The default is 10 (1024 seconds).

* maxdelay

   chronyd uses the network round-trip delay to the server to determine how accurate a particular measurement is likely to be. Long round-trip delays indicate that the request, or the response, or both were delayed. If only one of the messages was delayed the measurement error is likely to be substantial.

   For small variations in round trip delay, chronyd uses a weighting scheme when processing the measurements. However, beyond a certain level of delay the measurements are likely to be so corrupted as to be useless. (This is particularly so on dial-up or other slow links, where a long delay probably indicates a highly asymmetric delay caused by the response waiting behind a lot of packets related to a download of some sort).

   If the user knows that round trip delays above a certain level should cause the measurement to be ignored, this level can be defined with the maxdelay command. For example, maxdelay 0.3 would indicate that measurements with a round-trip delay of 0.3 seconds or more should be ignored. The default value is 3 seconds.

* maxdelayratio

   This option is similar to the maxdelay option above. chronyd keeps a record of the minimum round-trip delay amongst the previous measurements that it has buffered. If a measurement has a round trip delay that is greater than the maxdelayratio times the minimum delay, it will be rejected.

* maxdelaydevratio

   If a measurement has ratio of the increase in round-trip delay from the minimum delay amongst the previous measurements to the standard deviation of the previous measurements that is greater than maxdelaydevratio, it will be rejected. The default is 10.0.

* presend

   If the timing measurements being made by chronyd are the only network data passing between two computers, you may find that some measurements are badly skewed due to either the client or the server having to do an ARP lookup on the other party prior to transmitting a packet. This is more of a problem with long sampling intervals, which may be similar in duration to the lifetime of entries in the ARP caches of the machines.

   In order to avoid this problem, the presend option may be used. It takes a single integer argument, which is the smallest polling interval for which an extra pair of NTP packets will be exchanged between the client and the server prior to the actual measurement. For example, with the following option included in a server directive :

* presend 9

   when the polling interval is 512 seconds or more, an extra NTP client packet will be sent to the server a short time (currently 4 seconds) before making the actual measurement.

* key

   The NTP protocol supports the inclusion of checksums in the packets, to prevent computers having their system time upset by rogue packets being sent to them. The checksums are generated as a function of a password, using the cryptographic hash function set in the key file.

   The association between key numbers and passwords is contained in the keys file, defined by the keyfile command.

   If the key option is present, chronyd will attempt to use authenticated packets when communicating with this server. The key number used will be the single argument to the key option (an unsigned integer in the range 1 through 2**32-1). The server must have the same password for this key number configured, otherwise no relationship between the computers will be possible.

* offline

   If the server will not be reachable when chronyd is started, the offline option may be specified. chronyd will not try to poll the server until it is enabled to do so (by using the online option of chronyc).

* auto_offline

   If this option is set, the server will be assumed to have gone offline when 2 requests have been sent to it without receiving a response. This option avoids the need to run the offline (see section offline) command from chrony when disconnecting the dial-up link. (It will still be necessary to use chronyc’s online (see section online) command when the link has been established, to enable measurements to start.)

* iburst

   On start, make four measurements over a short duration (rather than the usual periodic measurements). If this option is set, the interval between the first four polls will be 2 seconds instead of minpoll. This is useful to quickly get the first update of the clock after chronyd is started.


* minstratum

   When the synchronisation source is selected from available sources, sources with lower stratum are normally preferred. This option can be used to increase stratum of the source to the specified minimum, so chronyd will avoid selecting that source. This is useful with low stratum sources that are known to be unrealiable or inaccurate and which should be used only when other sources are unreachable.

* polltarget

   Target number of measurements to use for the regression algorithm which chronyd will try to maintain by adjusting polling interval between minpoll and maxpoll. A higher target makes chronyd prefer shorter polling intervals. The default is 6 and a useful range is 6 to 60.

* version

   This option sets the NTP version number used in packets sent to the server. This can be useful when the server runs an old NTP implementation that doesn’t respond to newer versions. The default version number is 4.

* prefer

   Prefer this source over sources without prefer option.

* noselect

   Never select this source. This is particularly useful for monitoring.

* trust

   Assume time from this source is always true. It can be rejected as a falseticker in the source selection only if another source with this option doesn’t agree with it.

* require

   Require that at least one of the sources specified with this option is selectable (i.e. recently reachable and not a falseticker) before updating the clock. Together with the trust option this may be useful to allow a trusted authenticated source to be safely combined with unauthenticated sources in order to improve the accuracy of the clock. They can be selected and used for synchronisation only if they agree with the trusted and required source.

* minsamples

   Set the minimum number of samples kept for this source. This overrides the minsamples directive (see section minsamples).

* maxsamples

   Set the maximum number of samples kept for this source. This overrides the maxsamples directive (see section maxsamples).





##### Makestep directive
[source](https://chrony.tuxfamily.org/manual.html#makestep-command "chrony documentation for makestep-command")


Normally chronyd will cause the system to gradually correct any time offset, by slowing down or speeding up the clock as required. In certain situations, the system clock may be so far adrift that this slewing process would take a very long time to correct the system clock.

The makestep command can be used in this situation. There are two forms of the command. The first form has no parameters. It tells chronyd to cancel any remaining correction that was being slewed and jump the system clock by the equivalent amount, making it correct immediately.

The second form configures the automatic stepping, similarly to the makestep directive (see section makestep). It has two parameters, stepping threshold (in seconds) and number of future clock updates for which will be the threshold active. This can be used with the burst command to quickly make a new measurement and correct the clock by stepping if needed, without waiting for chronyd to complete the measurement and update the clock.
```
makestep 0.1 1
burst 1/2
```
BE WARNED - certain software will be seriously affected by such jumps to the system time. (That is the reason why chronyd uses slewing normally.)







## chronyc - command-line interface for chrony daemon

```
chronyc [OPTION]…​ [COMMAND]…​
```


##### sources
[source](https://chrony.tuxfamily.org/manual.html#sources-command "chrony documentation for sources-command")

Usage
```
chroncy sources
```

This command displays information about the current time sources that chronyd is accessing.

The optional argument -v can be specified, meaning verbose. In this case, extra caption lines are shown as a reminder of the meanings of the columns.
```

210 Number of sources = 3
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
#* GPS0                          0   4   377    11   -479ns[ -621ns] +/-  134ns
^? foo.example.net               2   6   377    23   -923us[ -924us] +/-   43ms
^+ bar.example.net               1   6   377    21  -2629us[-2619us] +/-   86ms
The columns are as follows:
```


* M
   
   This indicates the mode of the source. ^ means a server, = means a peer and # indicates a locally connected reference clock.

* S

   This column indicates the state of the sources. * indicates the source to which chronyd is currently synchronised. + indicates acceptable sources which are combined with the selected source. - indicates acceptable sources which are excluded by the combining algorithm. ? indicates sources to which connectivity has been lost or whose packets don’t pass all tests. x indicates a clock which chronyd thinks is is a falseticker (i.e. its time is inconsistent with a majority of other sources). ~ indicates a source whose time appears to have too much variability. The ? condition is also shown at start-up, until at least 3 samples have been gathered from it.

* Name/IP address
  
   This shows the name or the IP address of the source, or refid for reference clocks.

* Stratum

   This shows the stratum of the source, as reported in its most recently received sample. Stratum 1 indicates a computer with a locally attached reference clock. A computer that is synchronised to a stratum 1 computer is at stratum 2. A computer that is synchronised to a stratum 2 computer is at stratum 3, and so on.

* Poll
   
   This shows the rate at which the source is being polled, as a base-2 logarithm of the interval in seconds. Thus, a value of 6 would indicate that a measurement is being made every 64 seconds.

   chronyd automatically varies the polling rate in response to prevailing conditions.

 * Reach
   
   This shows the source’s reachability register printed as octal number. The register has 8 bits and is updated on every received or missed packet from the source. A value of 377 indicates that a valid reply was received for all from the last eight transmissions.

* LastRx

   This column shows how long ago the last sample was received from the source. This is normally in seconds. The letters m, h, d or y indicate minutes, hours, days or years. A value of 10 years indicates there were no samples received from this source yet.

* Last sample

   This column shows the offset between the local clock and the source at the last measurement. The number in the square brackets shows the actual measured offset. This may be suffixed by ns (indicating nanoseconds), us (indicating microseconds), ms (indicating milliseconds), or s (indicating seconds). The number to the left of the square brackets shows the original measurement, adjusted to allow for any slews applied to the local clock since. The number following the +/- indicator shows the margin of error in the measurement.

   Positive offsets indicate that the local clock is fast of the source.








##### tracking
[source](https://chrony.tuxfamily.org/manual.html#tracking-command "chrony documentation for tracking-command")

Usage
```
chroncy tracking
```



The tracking command displays parameters about the system’s clock performance. An example of the output is shown below.

```
Reference ID    : 1.2.3.4 (foo.example.net)
Stratum         : 3
Ref time (UTC)  : Fri Feb  3 15:00:29 2012
System time     : 0.000001501 seconds slow of NTP time
Last offset     : -0.000001632 seconds
RMS offset      : 0.000002360 seconds
Frequency       : 331.898 ppm fast
Residual freq   : 0.004 ppm
Skew            : 0.154 ppm
Root delay      : 0.373169 seconds
Root dispersion : 0.024780 seconds
Update interval : 64.2 seconds
Leap status     : Normal
```

The fields are explained as follows.

* Reference ID
   
   This is the refid and name (or IP address) if available, of the server to which the computer is currently synchronised. If this is 127.127.1.1 it means the computer is not synchronised to any external source and that you have the ‘local’ mode operating (via the local command in chronyc (see section local), or the local directive in the ‘/etc/chrony.conf’ file (see section local)).

* Stratum
   
   The stratum indicates how many hops away from a computer with an attached reference clock we are. Such a computer is a stratum-1 computer, so the computer in the example is two hops away (i.e. foo.example.net is a stratum-2 and is synchronised from a stratum-1).

* Ref time
   
   This is the time (UTC) at which the last measurement from the reference source was processed.

* System time

   In normal operation, chronyd never steps the system clock, because any jump in the timescale can have adverse consequences for certain application programs. Instead, any error in the system clock is corrected by slightly speeding up or slowing down the system clock until the error has been removed, and then returning to the system clock’s normal speed. A consequence of this is that there will be a period when the system clock (as read by other programs using the gettimeofday() system call, or by the date command in the shell) will be different from chronyd's estimate of the current true time (which it reports to NTP clients when it is operating in server mode). The value reported on this line is the difference due to this effect.

* Last offset

   This is the estimated local offset on the last clock update.

* RMS offset

   This is a long-term average of the offset value.

* Frequency
   The ‘frequency’ is the rate by which the system’s clock would be would be wrong if chronyd was not correcting it. It is expressed in ppm (parts per million). For example, a value of 1ppm would mean that when the system’s clock thinks it has advanced 1 second, it has actually advanced by 1.000001 seconds relative to true time.

   As you can see in the example, the clock in the computer is not a very good one - it gains about 30 seconds per day!

* Residual freq

   This shows the ‘residual frequency’ for the currently selected reference source. This reflects any difference between what the measurements from the reference source indicate the frequency should be and the frequency currently being used.

   The reason this is not always zero is that a smoothing procedure is applied to the frequency. Each time a measurement from the reference source is obtained and a new residual frequency computed, the estimated accuracy of this residual is compared with the estimated accuracy (see ‘skew’ next) of the existing frequency value. A weighted average is computed for the new frequency, with weights depending on these accuracies. If the measurements from the reference source follow a consistent trend, the residual will be driven to zero over time.

* Skew

   This is the estimated error bound on the the frequency.

* Root delay

   This is the total of the network path delays to the stratum-1 computer from which the computer is ultimately synchronised.

* Root dispersion

   This is the total dispersion accumulated through all the computers back to the stratum-1 computer from which the computer is ultimately synchronised. Dispersion is due to system clock resolution, statistical measurement variations etc.

   An absolute bound on the computer’s clock accuracy (assuming the stratum-1 computer is correct) is given by
   clock_error <= root_dispersion + (0.5 * |root_delay|)

* Update interval
  
   This is the interval between the last two clock updates.

* Leap status

   This is the leap status, which can be Normal, Insert second, Delete second or Not synchronised.





## Additional Links

* [UBUNTU NTP](https://ubuntu-mate.community/t/ubuntumate-15-10-reset-rtc-chip-date-time-during-bootup/2750/3 )
* [Fedora system administration guide for chrony](https://docs.fedoraproject.org/en-US/Fedora/18/html/System_Administrators_Guide/sect-Checking_if_chrony_is_synchronized.html)
* [Chrony comparision to NTPD](https://chrony.tuxfamily.org/comparison.html)
* [Chrony documentation](https://chrony.tuxfamily.org)




























































