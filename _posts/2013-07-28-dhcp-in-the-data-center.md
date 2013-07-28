---
layout: post
title: "DHCP In The Data Center"
description: ""
category: 
tags: [automation, datacenter]
---
{% include JB/setup %}

I've had and 
[heard](http://serverfault.com/questions/99280/dhcp-for-data-center) debates 
about using DHCP for large data center deployments. There is no doubt that DHCP 
is necessary for PXE install of servers but what about manual static addressing 
vs dynamic allocation?

Well, my automation mantra rejects anything that has some sort of manual 
process. When you have a pile of hundreds or thousands bare metal servers to set 
up, the last thing you want is decide manually all the IPs for each server. How 
would you even do that? Some places have a wiki page where they reference all 
IPs, which is probably a nightmare. Other places have static IPs but add a DNS 
entry manually to centralize the IPs in a zone file. This helps determine "free" 
addresses from one place. You can improve this slightly by using your 
configuration manager (Chef, Puppet...) to set the static IP and the DNS zone 
and have them in sync.

But, in the end, what are we trying to do here? We try to have a central place 
where we list IPs and network configurations in order to determine which is used 
and which is free. Them we manually assign a free address to our newly installed 
server. Guess what? Some thing invented years ago can automatically do that for 
you through a well documented and robust protocol. It's called DHCP and removes 
the burden of doing the network configuration from you and negotiating all the 
mundane things that you don't really want to repeat for each install like 
updating the DNS entries through DDNS updates.

So why not use DHCP? Excessive automation is often perceived as dangerous from 
many sysadmins and it is a significant concern. Yes, you have one more service 
to maintain but DHCP servers are generally one of the most stable and mature 
services around, requiring close to zero maintenance and have a simple failover 
mechanism. Some quirks can happen but here are some ways to alleviates issues 
(Note: I mostly know of the [ISC DHCP](http://www.isc.org/downloads/dhcp/) 
server implementation packaged in most systems):

* Make sure dhclient is in persistent mode (See PERSISTENT_DHCLIENT option in 
RedHat/CentOS systems, it's off by default) so that your server won't stop 
requesting a network configuration if an error occur
* Increase the lease time to days, weeks or more. Small lease times are for places 
with high client turnaround and a limited address pool. Needless to say that 
this is not the case on a data center. You should also make sure that you have 
enough time to react on case of a server failure.
* Monitor lease expiration dates on your servers so that you are alerted before 
your server lease expire and loose connectivity.

Now you can spend time doing useful stuff like making your systems reliable, 
improve efficiency and monitoring...etc instead of assigning IPs every other 
day. Note that this has to be done in combination with automatic VLAN 
assignment, automatic DNS configuration and probably automated install and 
provisioning.
